# Lesson 004 — MongoDB Atlas Setup and Connection Strings

[← Section 1](./README.md) · [Full curriculum](../README.md)

## Why Atlas?
A local MongoDB server runs on your machine. **Atlas** hosts MongoDB in a managed cloud deployment, so your backend can connect to it remotely.

```text
Express API → MongoDB driver / Mongoose → encrypted connection → Atlas deployment → database → collection
```

## 1. Create a deployment
1. Sign in at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
2. Create or select an **organization** and **project**.
3. Create a cluster/deployment using a suitable available tier (free options depend on current availability).
4. Create a **database user** with a strong password and only the permissions your app needs.
5. In **Network Access**, allow only the IP addresses that should connect.
6. Select **Connect → Drivers** to get your connection URI.

A **project** groups resources and permissions; a **cluster** runs the database; a **database** holds collections. They are not interchangeable.

## 2. Understand the URI
```text
mongodb+srv://USERNAME:PASSWORD@cluster.example.mongodb.net/learningDB?retryWrites=true&w=majority
             │        │            │                      │
         DB user   password      host                 database
```
This is an illustrative URI, **not** a working hostname. `mongodb+srv` uses DNS records to discover hosts and connection settings. Connection options can vary; copy the actual URI from Atlas. Percent-encode reserved characters in usernames/passwords when used in a URI.

## 3. Connect from Node.js with Mongoose
```bash
npm init -y
npm pkg set type=module
npm install mongoose dotenv
```
`.env`:
```dotenv
MONGODB_URI=mongodb+srv://YOUR_USER:YOUR_ENCODED_PASSWORD@YOUR_REAL_CLUSTER_HOST/learningDB?retryWrites=true&w=majority
```
`index.js`:
```js
import "dotenv/config";
import mongoose from "mongoose";

async function main() {
  try {
    if (!process.env.MONGODB_URI) throw new Error("MONGODB_URI missing");
    await mongoose.connect(process.env.MONGODB_URI);
    console.log("Connected to:", mongoose.connection.name);
  } catch (error) {
    console.error("Connection failed:", error.message);
    process.exitCode = 1;
  } finally {
    await mongoose.disconnect();
  }
}
main();
```
Run `node index.js`. Use the URI from your own Atlas deployment.

## 4. Security and common problems
- Put `.env` in `.gitignore`; never commit credentials or paste them into logs.
- **Authentication failed:** Check the **database user** (not your Atlas website login), password and URI encoding.
- **Server selection timeout:** Check the allowed client IP, DNS, firewall and cluster availability.
- **TLS/hostname errors:** Verify the URI and system/network configuration.
- Avoid `0.0.0.0/0` for production network access. If temporarily used during development, understand it allows connections from any IP (authentication is still required) and restrict it afterward.
- Never create a new connection on each API request; reuse a connection/pool.

## Practice
Connect with Compass and with Mongoose to the same Atlas deployment. Explain where the database user, IP allowlist, and database name are configured.

## Interview check
**Atlas vs MongoDB?** Atlas is managed hosting; MongoDB is the database system.
**Why IP allowlisting?** It restricts which client network addresses can reach the deployment.

**Reference:** [Atlas connect](https://www.mongodb.com/docs/atlas/connect-to-cluster/)
