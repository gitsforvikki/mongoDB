# Lesson 69 — Environment Configuration and DB Connection

[← Section 8](./README.md) · [Full curriculum](../README.md)

## 1. Why configuration deserves its own lesson

A backend runs in development, tests, staging and production. Each environment needs different database credentials, ports, signing secrets and connection settings. Hardcoding values leads to leaked credentials, accidental writes to production, and difficult deployments.

**Environment variables are configuration inputs, not inherently secure storage.** Production secrets should come from a trusted secrets mechanism, and should never appear in Git commits, logs or client-side bundles.

## 2. Dependencies and package scripts

```bash
npm init -y
npm pkg set type=module
npm install express@5 mongoose zod dotenv
npm pkg set scripts.dev="node --watch src/server.js"
npm pkg set scripts.start="node src/server.js"
```

Node.js versions vary in native env-file support; here we use `dotenv` explicitly.

```dotenv
# .env.example — placeholders, safe to commit
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb+srv://USER:PASSWORD@CLUSTER_HOST/shop_api?retryWrites=true&w=majority
```

```gitignore
node_modules/
.env
.env.*
!.env.example
```

Never commit real `.env` values. If a secret is committed, deleting the line is not sufficient: rotate the secret and address Git history.

## 3. Parse configuration once with Zod

```js
// src/config/env.js
import "dotenv/config";
import { z } from "zod";

const schema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]).default("development"),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
  MONGODB_URI: z.string().min(1).refine(
    value => value.startsWith("mongodb://") ||
             value.startsWith("mongodb+srv://"),
    "MONGODB_URI must be a MongoDB URI"
  )
});

export const env = schema.parse(process.env);
```

**Why fail fast?** If `MONGODB_URI` is missing, the server should stop immediately with a clear startup error, not fail on the first customer request. Zod checks the shape of a URI, but it does **not** prove credentials, TLS, DNS or network permissions are valid.

## 4. Understand the connection string

```text
mongodb+srv://USER:PASSWORD@cluster.example.mongodb.net/shop_api
│              │    │        │                         │
scheme         user password  cluster host              database
```

- `mongodb+srv://` uses DNS SRV discovery and is common for Atlas.
- Database users are separate from Atlas dashboard users.
- Atlas network access rules must permit the application's network.
- Special characters in username/password may need URL encoding.
- The database name in the URI or `dbName` option chooses the target database.
- Never display the full URI in an error response or log.

## 5. Connect once at startup

```js
// src/config/database.js
import mongoose from "mongoose";
import { env } from "./env.js";

export async function connectDatabase() {
  await mongoose.connect(env.MONGODB_URI, {
    maxPoolSize: 20,
    serverSelectionTimeoutMS: 5000
  });
  console.log("MongoDB connected");
}

export async function disconnectDatabase() {
  await mongoose.disconnect();
}
```

A Mongoose connection uses the MongoDB driver's **connection pool**. Many concurrent HTTP requests can share pooled connections. Pool size is not equal to maximum supported users; a request may wait for an available connection while other requests are processing.

```mermaid
flowchart TD
 A["Node.js process starts"] --> B["Load and validate environment"]
 B --> C["mongoose.connect()"]
 C --> D{"Connected?"}
 D -->|No| E["Fail startup / alert"]
 D -->|Yes| F["Start HTTP listener"]
 F --> G["Concurrent HTTP requests"]
 G --> H["Shared MongoDB driver connection pool"]
 H --> I["MongoDB"]
```

## 6. Connection states and buffering

Mongoose has connection states such as disconnected, connecting, connected and disconnecting. If commands are issued while disconnected, Mongoose can **buffer** certain operations by default, sometimes making errors appear delayed. Understand `bufferCommands`, `bufferTimeoutMS` and driver `serverSelectionTimeoutMS`: they affect different waiting behaviors. Do not assume a database connection is permanently healthy after startup; monitor disconnects and readiness.

```js
import mongoose from "mongoose";
mongoose.connection.on("disconnected", () => console.warn("MongoDB disconnected"));
mongoose.connection.on("error", error => console.error("MongoDB connection error:", error.message));
```

Avoid printing full connection strings or authentication details.

## 7. Graceful shutdown

```js
// src/server.js
import { app } from "./app.js";
import { env } from "./config/env.js";
import { connectDatabase, disconnectDatabase } from "./config/database.js";

let server;
try {
  await connectDatabase();
  server = app.listen(env.PORT, () => console.log(`Listening on \${env.PORT}`));
} catch (error) {
  console.error("Startup failure:", error.message);
  process.exitCode = 1;
}

async function shutdown(signal) {
  console.log(`Received \${signal}`);
  if (server) {
    await new Promise((resolve, reject) => {
      server.close(error => error ? reject(error) : resolve());
    });
  }
  await disconnectDatabase();
}
process.once("SIGTERM", () => { shutdown("SIGTERM").catch(console.error); });
process.once("SIGINT", () => { shutdown("SIGINT").catch(console.error); });
```

**Theory:** Stop accepting new HTTP connections, let in-flight requests finish as appropriate, then close database resources. Production servers need shutdown deadlines and error handling to avoid hanging indefinitely. A SIGTERM handler should not assume all external systems remain reachable.

## 8. Atlas troubleshooting by symptom

| Symptom | Likely area |
| --- | --- |
| DNS/SRV resolution error | Hostname, DNS, network |
| Authentication failed | Database username/password/roles |
| Server selection timed out | Atlas access list, network, TLS, cluster availability |
| Operations buffering timed out | Query issued before connection or after disconnect |
| Duplicate index errors | Existing data/index definitions |
| Works locally, fails in cloud | Cloud egress IP, secrets injection, firewall |

Do not “fix” production by opening database network access to the entire internet without considering the security consequences.

## 9. Exercises

Create `.env.example`, deliberately omit `MONGODB_URI` and observe fail-fast validation, connect to a practice Atlas database, log connection state transitions, and test graceful SIGTERM shutdown.

## 10. Interview preparation

**Why connect before listen?** Fail fast rather than accept unusable requests. **Why use a pool?** Reuse network connections under concurrency. **Is a valid URI enough?** No; DNS, authentication and network permissions must work. **Why graceful shutdown?** Avoid abruptly terminating active work.

**References:** [Mongoose connections](https://mongoosejs.com/docs/connections.html) · [MongoDB connection options](https://www.mongodb.com/docs/drivers/node/current/connect/connection-options/)
