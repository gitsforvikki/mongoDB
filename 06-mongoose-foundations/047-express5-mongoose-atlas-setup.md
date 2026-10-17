# Lesson 047 — Set Up Express 5 + Mongoose + Atlas

[← Section 6](./README.md) · [Full curriculum](../README.md)

## Deep theory: connection lifecycle and production reasoning

A connection string is **not the connection itself**. It tells the driver where and how to connect. `mongoose.connect(uri)` initializes a Mongoose connection backed by the MongoDB driver. The driver discovers suitable servers, authenticates, and maintains pools of reusable sockets.

### Why one shared connection?

Imagine 100 HTTP requests arriving together. A naive implementation calls `mongoose.connect()` inside each controller. That repeatedly performs connection work and makes application behavior difficult to reason about. A better design initializes the database connection **once during startup**, then each controller executes queries through compiled models. The underlying driver uses pooled connections, subject to pool size and server availability.

```mermaid
flowchart TD
  A["Start Node process"] --> B["Load configuration"]
  B --> C["Connect Mongoose once"]
  C --> D["Driver connection pool"]
  D --> DB["MongoDB"]
  C --> E["Start Express listener"]
  E --> R1["Request 1"]
  E --> R2["Request 2"]
  E --> R3["Request 3"]
  R1 --> M["Shared models"]
  R2 --> M
  R3 --> M
  M --> D
```

A pool of size 10 does **not** mean the application can handle only 10 requests. Most requests spend time in different parts of the application, and database operations reuse available sockets. When more operations need sockets than are available, some wait in the driver's queue. The right pool size depends on workload, deployment replicas, MongoDB limits, and query duration; increasing it blindly is not an optimization.

### Why connect before app.listen()?

Suppose `app.listen()` starts first. A user requests `GET /api/users` while the database is still unavailable. By default, Mongoose may buffer model operations and eventually time them out. From the user's perspective, the API is alive but not ready. Waiting for the **initial successful connection** before listening avoids that startup race.

Once the process is running, a temporary disconnection is different from a failed initial connection. The driver may attempt to recover connectivity, but individual operations can still fail. Production services should expose **liveness** (process responding) and **readiness** (dependencies sufficiently available) separately.

### Environment and Atlas: what each setting means

- **Cluster URI:** where the MongoDB deployment lives.
- **Database user:** identity MongoDB authenticates; it is not your application end-user.
- **Network Access:** which network sources can reach Atlas. A correct password cannot fix blocked network access.
- **Database name:** selects the database used for your collections; it is not a username.
- **TLS:** protects traffic in transit when correctly configured.
- **Environment variable:** keeps credentials out of source control, but deployment secrets must still be protected.

**Security note:** Avoid printing the full URI in logs because it can contain credentials. Use a least-privilege database user. For local learning, `127.0.0.1` is often more predictable than `localhost` when IPv6 resolution causes connection surprises.

### How an async controller behaves

```js
app.get("/api/users", async (req, res) => {
  const users = await User.find().limit(20).lean();
  res.json({ success: true, data: users });
});
```

The `await` suspends **this async handler**, not the entire Node.js process. While MongoDB performs the query, Node.js can continue handling other events and requests. When the query settles, the handler resumes. If the promise rejects, **Express 5** forwards the error to error-handling middleware. This does not mean errors should be ignored: translate expected errors into appropriate HTTP statuses and avoid leaking internals.

### Operational failure scenarios

**Wrong password:** authentication fails; do not retry forever with the same invalid secret. **Cluster unavailable:** server selection may time out; report startup failure and use appropriate deployment restart policies. **Slow query:** a healthy connection does not imply a fast endpoint; inspect indexes and explain plans. **Process termination:** stop accepting new requests, allow in-flight work to complete within a bounded grace period, then close the database connection.

### A small readiness endpoint

```js
import mongoose from "mongoose";

app.get("/ready", (req, res) => {
  const ready = mongoose.connection.readyState === 1;
  res.status(ready ? 200 : 503).json({ ready });
});
```

This is a **basic indicator**, not a complete end-to-end database health test. `readyState === 1` indicates Mongoose considers the connection connected; it does not guarantee that every query will succeed.

**Checkpoint:** Explain in your own words why a driver pool is different from a thread pool, and why an API might return 503 for readiness while still responding to a liveness check.

---
## Learning goals
Build a minimal Express 5 API with ES modules, environment variables, one shared database connection, health check, clean startup and error handling.

## 1. How startup works
```mermaid
flowchart TD
 A["Load .env"] --> B["Create Express app"]
 B --> C["Connect to MongoDB"]
 C --> D{"Connection succeeded?"}
 D -->|Yes| E["app.listen()"]
 D -->|No| F["Log error and exit"]
 E --> G["Incoming HTTP requests"]
 G --> H["Controllers use shared Mongoose connection"]
```
**Why connect before listening?** Otherwise your API may accept requests before the database is ready. Mongoose also buffers some operations by default, which can hide missing-connection problems until timeout.

## 2. Prerequisites
- Node.js LTS, npm, MongoDB Atlas cluster or local MongoDB.
- For Atlas: create a database user with appropriate permissions and allow the app's outbound IP in Network Access. Do not use broad IP access casually in production.
- Copy the connection URI from Atlas. Encode special characters in credentials if constructing a URI.
- Never commit credentials to Git.

## 3. Create project
```bash
mkdir mongoose-express-course && cd mongoose-express-course
npm init -y
npm pkg set type=module
npm pkg set scripts.dev="node --watch src/server.js"
npm pkg set scripts.start="node src/server.js"
npm install express@^5 mongoose dotenv
mkdir -p src/config src/models
```
Use compatible supported versions; install output resolves concrete versions.

**.gitignore**
```gitignore
node_modules/
.env
.env.*
!.env.example
```

**.env.example** (safe template to commit)
```dotenv
PORT=5000
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@YOUR-CLUSTER.mongodb.net/learning?retryWrites=true&w=majority
```
Create a real `.env` locally and replace placeholders. Use `mongodb://127.0.0.1:27017/learning` for local MongoDB. Never publish the real URI.

## 4. src/config/database.js
```js
import mongoose from "mongoose";

export async function connectDatabase() {
  mongoose.connection.on("disconnected", () => {
    console.warn("MongoDB disconnected");
  });
  mongoose.connection.on("error", (error) => {
    console.error("MongoDB connection error:", error.message);
  });

  await mongoose.connect(process.env.MONGODB_URI, {
    serverSelectionTimeoutMS: 10000,
    maxPoolSize: 10
  });

  console.log("MongoDB connected:", mongoose.connection.name);
}
```
`mongoose.connect()` returns a promise that resolves after an initial successful connection. **maxPoolSize:10** sets an upper bound for the driver's connection pool; it does not mean 10 new connections are opened for every HTTP request. MongoDB drivers may maintain additional monitoring connections.

## 5. src/app.js
```js
import express from "express";
const app = express();

app.use(express.json({limit:"100kb"}));

app.get("/health", (req,res) => {
  res.status(200).json({ok:true,service:"mongoose-course"});
});

app.get("/api/demo", async (req,res) => {
  // In the next lesson, replace this with a Mongoose model query.
  res.json({message:"Express 5 is running"});
});

// Error middleware must have four parameters.
app.use((error,req,res,next) => {
  console.error(error);
  res.status(500).json({message:"Internal server error"});
});

export default app;
```
Express 5 automatically forwards **rejected promises/throwing async route handlers** to error middleware. You usually do not need a `try/catch(next)` wrapper for every async controller. Keep the error handler after your routes.

## 6. src/server.js
```js
import "dotenv/config";
import mongoose from "mongoose";
import app from "./app.js";
import {connectDatabase} from "./config/database.js";

let server;
async function startServer() {
  try {
    if (!process.env.MONGODB_URI) {
      throw new Error("MONGODB_URI is missing");
    }
    await connectDatabase();
    server = app.listen(Number(process.env.PORT) || 5000, () => {
      console.log("API listening");
    });
  } catch (error) {
    console.error("Startup failed:", error.message);
    process.exitCode = 1;
    await mongoose.disconnect();
  }
}
startServer();

async function shutdown(signal) {
  console.log(`Received ${signal}; shutting down`);
  if (server) await new Promise((resolve,reject) =>
    server.close(err => err ? reject(err) : resolve())
  );
  await mongoose.disconnect();
  process.exitCode = 0;
}
process.once("SIGINT", () => shutdown("SIGINT").catch(console.error));
process.once("SIGTERM", () => shutdown("SIGTERM").catch(console.error));
```
This is a teaching baseline. Production shutdown should have a bounded timeout and explicit failure exit handling; orchestrators may terminate the process after a grace period.

## 7. Run and verify
```bash
npm run dev
curl http://localhost:5000/health
curl http://localhost:5000/api/demo
```
Expected health response:
```json
{"ok":true,"service":"mongoose-course"}
```

## 8. How request handling uses the connection
```mermaid
sequenceDiagram
 participant Client
 participant Express
 participant Model as Mongoose Model
 participant Pool as MongoDB Driver Pool
 participant DB as MongoDB
 Client->>Express: GET /api/users
 Express->>Model: User.find()
 Model->>Pool: execute query
 Pool->>DB: MongoDB command
 DB-->>Pool: BSON results
 Pool-->>Model: result data
 Model-->>Express: Mongoose documents
 Express-->>Client: JSON response
```
A **single shared Mongoose connection** manages a pool of reusable network connections. Do **not** call `mongoose.connect()` in each route or for each request.

## 9. Troubleshooting
| Symptom | Likely cause | Check |
| --- | --- | --- |
| Server selection timeout | Cluster unreachable / IP not allowed / DNS | Atlas Network Access, cluster status, network |
| Authentication failed | Incorrect DB user/password | Credentials and URL encoding |
| `MONGODB_URI is missing` | `.env` not loaded | File location and variable name |
| Query buffers then times out | Model queried without working connection | Startup order and connection state |
| Works locally but not deployed | Wrong env/IP/TLS settings | Hosting environment and Atlas access |

## Practice
Add a `/health/db` route that checks `mongoose.connection.readyState === 1` and returns 503 if disconnected. Explain why the health endpoint and readiness endpoint answer different questions.

## Interview check
**Q: Why one connection per app?** Mongoose's shared connection uses a pool to reuse sockets and avoid per-request connection overhead.

**Q: What does await mongoose.connect() do?** Waits for initial connection before accepting traffic.

**Q: Does Express 5 handle rejected async routes?** Yes, it forwards errors to error middleware.

**Official references:** [Mongoose connections](https://mongoosejs.com/docs/connections.html) · [Express error handling](https://expressjs.com/en/guide/error-handling.html) · [Atlas connections](https://www.mongodb.com/docs/atlas/connect-to-database-deployment/)
