# Lesson 002 — MongoDB vs Mongoose vs Atlas

[← Lesson 001](./001-what-is-mongodb.md) · [Section 1](./README.md)

## 1. First understand the three roles

**MongoDB, Mongoose, and MongoDB Atlas are different things.** They are often used together, which is why beginners confuse them.

| Tool | What it is | What it does |
| --- | --- | --- |
| **MongoDB** | Database system | Stores data and runs queries |
| **Mongoose** | Node.js ODM (Object Document Mapper) library | Helps define schemas/models, validate data, and interact with MongoDB from JavaScript |
| **MongoDB Atlas** | Managed cloud database service | Hosts and manages MongoDB deployments in the cloud |

**Important:** Mongoose is optional. You can connect Node.js to MongoDB using the official MongoDB Node.js driver without Mongoose. Atlas is also optional: MongoDB can run locally or on your own servers.

## 2. Understand with a Node.js application

Imagine a user sends a request to create an account:

```text
React Frontend
      |
      | POST /api/users
      v
Node.js / Express 5
      |
      | User.create(...) using Mongoose
      v
Mongoose ODM
      |
      | uses MongoDB Node.js driver
      v
MongoDB Database  ← hosted locally OR on MongoDB Atlas
      |
      v
users collection → user document
```

**Who stores the data?** MongoDB.

**Who helps your Node.js code work with schemas and models?** Mongoose.

**Who can run the MongoDB database in the cloud?** Atlas.

## 3. MongoDB: the actual database

MongoDB stores documents, for example:

```js
{
  name: "Asha",
  email: "asha@example.com",
  age: 24
}
```

In `mongosh`, you can work with MongoDB directly:

```js
use learningDB

db.users.insertOne({
  name: "Asha",
  email: "asha@example.com",
  age: 24
})

db.users.find({ age: { $gte: 18 } })
```

No Mongoose is required for these shell commands.

## 4. Mongoose: an ODM for Node.js

Mongoose adds a useful application-level structure on top of the official MongoDB driver:

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true },
  age: { type: Number, min: 0 }
});

const User = mongoose.model("User", userSchema);

// After connecting Mongoose to MongoDB:
await User.create({
  name: "Asha",
  email: "asha@example.com",
  age: 24
});
```

What happens here?

1. `new mongoose.Schema(...)` defines the expected fields and validation.
2. `mongoose.model("User", userSchema)` creates a model.
3. `User.create(...)` validates/casts the data and writes via the MongoDB driver.
4. MongoDB stores it in the model's collection (by default, `users`).

**Important distinction:** A Mongoose `required` validator is not automatically a MongoDB server-side constraint. Other clients accessing the database can bypass it unless you also enforce database-level validation.

## 5. Atlas: managed hosting for MongoDB

Atlas operates a MongoDB deployment for you.

```text
MongoDB Atlas
└── Project
    └── Cluster / Deployment
        └── learningDB
            ├── users
            └── products
```

An **Atlas project** organizes deployments and access settings. A **cluster/deployment** runs MongoDB. Within the deployment, you create your application databases and collections.

Atlas can help manage hosting, operations, monitoring, and deployment configuration; available features depend on the service tier.

**Atlas is not Mongoose**, and **the Atlas project is not itself a MongoDB database**.

## 6. Complete Express-free Node.js example using Mongoose

Install Mongoose in a Node.js project:

```bash
npm init -y
npm pkg set type=module
npm install mongoose dotenv
```

Create a `.env` file (keep it out of Git):

```dotenv
MONGODB_URI=mongodb://127.0.0.1:27017/learningDB
```

This example assumes MongoDB is running locally. With Atlas, `MONGODB_URI` would instead be your Atlas connection string (with the credentials handled securely).

Create `index.js`:

```js
import "dotenv/config";
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true }
});

const User = mongoose.model("User", userSchema);

async function main() {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log("MongoDB connected");

    const user = await User.create({
      name: "Asha",
      email: "asha@example.com"
    });

    console.log("Saved user:", user);
  } catch (error) {
    console.error("Database operation failed:", error.message);
    process.exitCode = 1;
  } finally {
    await mongoose.disconnect();
  }
}

main();
```

Run:

```bash
node index.js
```

What happens in order?

1. Node loads the connection string from `.env`.
2. `mongoose.connect(...)` connects to the MongoDB server (local or Atlas).
3. `User.create(...)` inserts one validated document.
4. `mongoose.disconnect()` closes the connection when this short script ends.

For a long-running Express server, you usually keep one shared Mongoose connection open and start listening after the initial connection succeeds. Do not connect separately inside each route handler.

## 7. Can we use MongoDB without Mongoose?

**Yes.** With the official Node.js driver:

```bash
npm install mongodb
```

```js
import { MongoClient } from "mongodb";

const client = new MongoClient("mongodb://127.0.0.1:27017");

try {
  await client.connect();

  const users = client.db("learningDB").collection("users");

  await users.insertOne({ name: "Ravi", age: 22 });
  const result = await users.findOne({ name: "Ravi" });

  console.log(result);
} finally {
  await client.close();
}
```

This talks to the same MongoDB database without Mongoose models or automatic Mongoose validation.

## 8. Which option should you choose?

- **MongoDB + `mongosh`:** best for learning database commands and understanding how data is stored.
- **MongoDB + official Node.js driver:** direct, lightweight database access from Node.js.
- **MongoDB + Mongoose:** helpful when you want model-based data organization, casting, validation, hooks and `populate()`.
- **MongoDB Atlas:** choose when you want a managed cloud MongoDB deployment instead of managing your own server.

For this course: **learn MongoDB first (Sections 1–5), then Mongoose with Express 5 (Sections 6–8).**

## 9. Common mistakes

- Calling Atlas the database library used in your Node.js code.
- Believing Mongoose runs or hosts MongoDB.
- Believing Mongoose is required for MongoDB.
- Calling an Atlas cluster and a MongoDB collection the same thing.
- Hardcoding Atlas credentials in code or committing `.env`.
- Opening a new Mongoose connection for every HTTP request.
- Assuming Mongoose validation automatically protects writes made outside Mongoose.

## 10. Practice and interview questions

**Practice**

1. Draw the flow from Express → Mongoose → MongoDB.
2. Explain where your `users` collection lives when using Atlas.
3. Explain whether the Mongoose `User` model or MongoDB actually stores a new user.
4. Run the local example after setting up MongoDB in Lesson 003.

**Interview**

**Q: What is the difference between MongoDB and Mongoose?** MongoDB is the database; Mongoose is an optional ODM library for Node.js.

**Q: Is MongoDB Atlas another database engine?** No. Atlas is a managed cloud service that runs MongoDB.

**Q: Can Mongoose connect to local MongoDB?** Yes, it can connect to local or hosted MongoDB deployments.

**Q: Do you need Atlas to use Mongoose?** No.

## Quick revision

**MongoDB stores. Mongoose models and validates in Node.js. Atlas hosts MongoDB in the cloud.**

**Official references:** [MongoDB Node.js Driver](https://www.mongodb.com/docs/drivers/node/current/) · [Mongoose Guide](https://mongoosejs.com/docs/guide.html) · [MongoDB Atlas](https://www.mongodb.com/docs/atlas/)
