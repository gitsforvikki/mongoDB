# Lesson 007 — mongosh Essentials and Basic Commands

[← Section 1](./README.md) · [Full curriculum](../README.md)

## 1. What is mongosh?
**mongosh** is the MongoDB Shell: an interactive command-line client for working with MongoDB. It is **not** the MongoDB server.

```text
You type command → mongosh sends request → MongoDB server executes → mongosh prints result
```

## 2. Connect
```bash
mongosh "mongodb://127.0.0.1:27017"
```
For Atlas, copy the shell connection instructions from your deployment and authenticate with your database user. Never commit or share a connection string containing a password.

## 3. Essential commands
Run these **inside mongosh**:
```js
help                    // shell help
show dbs                // list visible databases
use learningDB          // switch current database
db                      // display current database name
show collections        // list collections in current database
db.runCommand({ ping: 1 }) // verify server responds
db.students.insertOne({ name: "Asha", age: 24 })
db.students.find()      // cursor of documents
db.students.findOne({ name: "Asha" })
db.students.countDocuments({ age: { $gte: 18 } })
db.getName()            // current database name
```
`show dbs` and `show collections` are shell helpers, not ordinary JavaScript function calls.

## 4. Understand a query
```js
db.students.find({ age: { $gte: 18 } }, { name: 1, _id: 0 })
```
- `db`: current database.
- `students`: collection.
- `find`: query documents.
- `{ age: { $gte: 18 } }`: filter ages 18 or above.
- `{ name: 1, _id: 0 }`: show only `name`, excluding `_id`.

`find()` returns a cursor; `findOne()` returns a document or `null`. In interactive mongosh, cursors can be displayed automatically.

## 5. Useful shell workflow
```js
use learningDB
db.students.insertMany([
  { name: "Asha", age: 24 },
  { name: "Ravi", age: 17 },
  { name: "Neha", age: 21 }
])
db.students.find({ age: { $gte: 18 } }).sort({ age: -1 }).limit(2)
```
Expected names in order: Asha, Neha. `sort({ age: -1 })` sorts descending; `limit(2)` returns at most two.

## 6. Safety and common mistakes
- `use` only selects a database; it does not guarantee the database exists on disk.
- Do not run destructive commands such as `dropDatabase()` on real data.
- Do not confuse `mongosh` commands with Mongoose methods inside Node.js.
- The shell can run JavaScript, but shell helpers such as `show dbs` are special syntax.
- `exit` or Ctrl+D leaves the shell; it does not stop MongoDB Server.

## Practice
Create three products; find products priced above 1000; sort by price descending; display only names and prices. Check the results in Compass.

## Interview check
**What is mongosh?** An interactive MongoDB client.
**find vs findOne?** `find` returns a cursor; `findOne` returns one document or null.
**Does exiting mongosh stop MongoDB?** No.

**Reference:** [MongoDB Shell](https://www.mongodb.com/docs/mongodb-shell/)
