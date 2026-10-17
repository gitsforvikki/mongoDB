# Lesson 001 — What Is MongoDB and Why Use It?

[← Section 1](./README.md) · [Next: Lesson 002 →](./002-mongodb-vs-mongoose-vs-atlas.md)

## 1. What is MongoDB?

**MongoDB is a NoSQL document database.** We use it to store, retrieve, update, and delete application data.

In simple words, imagine an e-commerce application. You need to remember users, products, carts, and orders even after the server restarts. MongoDB can store that information permanently.

Instead of storing data in traditional rows and tables, MongoDB stores **documents** (BSON objects, similar to JavaScript objects) in **collections**.

```text
MongoDB Server
└── shopDB                 ← database
    ├── users              ← collection
    │   ├── { name: "Asha", email: "asha@example.com" }   ← document
    │   └── { name: "Ravi", email: "ravi@example.com" }   ← document
    └── products           ← collection
        └── { name: "Laptop", price: 55000 }              ← document
```

A MongoDB deployment can contain multiple databases. Each database contains collections, and each collection contains documents.

## 2. Why do we need a database?

Without a database, storing users in an in-memory JavaScript array is not a durable solution:

```js
const users = [{ name: "Asha", email: "asha@example.com" }];
```

When the Node.js process restarts, that array is recreated and any newly added users are lost. A database stores application information separately from the Node.js process so it can be read again after restarts.

MongoDB also lets multiple API requests work with the same stored data.

## 3. What does a MongoDB document look like?

A document can contain nested objects and arrays:

```js
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Asha",
  email: "asha@example.com",
  address: {
    city: "Patna",
    state: "Bihar"
  },
  skills: ["React", "Node.js"],
  isActive: true
}
```

- `_id` uniquely identifies a document within its collection. MongoDB automatically generates an ObjectId if you insert a document without `_id`.
- `address` is an **embedded document**.
- `skills` is an **array**.
- Documents are stored in **BSON** (Binary JSON), which supports types such as ObjectId and Date.

You will study BSON and ObjectId in Lesson 006.

## 4. MongoDB vs SQL databases

| Concept | MongoDB | Relational database such as PostgreSQL |
| --- | --- | --- |
| Main structure | Documents in collections | Rows in tables |
| Data representation | BSON documents | Table columns and rows |
| Schema | Flexible by default; validation is possible | Usually predefined table schema |
| Relationships | Embed documents or reference other documents | Foreign keys and joins |
| Query language | MongoDB Query Language and aggregation | SQL |

Example SQL-style row:

```text
users table
id | name | city
1  | Asha | Patna
```

Equivalent MongoDB document:

```js
{ _id: 1, name: "Asha", city: "Patna" }
```

**Important:** MongoDB is not automatically better or faster than SQL. Both can support complex queries and transactions. The correct choice depends on your data, relationships, and access patterns.

## 5. What does "flexible schema" mean?

Two documents in the same collection may have different fields:

```js
{ name: "Asha", phone: "9999999999" }
{ name: "Ravi", email: "ravi@example.com", skills: ["React"] }
```

That is allowed by default.

**But flexible does not mean structure is unnecessary.** In production, define consistent fields and validate data using MongoDB schema validation, Mongoose, or both.

## 6. Practical example in mongosh

These commands run in the MongoDB shell (`mongosh`), not in ordinary Node.js code:

```js
use shopDB

db.products.insertOne({
  name: "Laptop",
  price: 55000,
  category: "Electronics",
  inStock: true
})

db.products.find({ category: "Electronics" })
```

Execution flow:

1. `use shopDB` selects the database; it may not be physically created until data is written.
2. `db.products.insertOne(...)` inserts a product document.
3. MongoDB adds `_id` automatically if you did not provide one.
4. `find(...)` returns the matching product documents.

You will set up `mongosh` in Lesson 003 and learn CRUD deeply in Section 2.

## 7. When should you use MongoDB?

MongoDB is a reasonable option when:
- Your application naturally works with document-like data (profiles, catalogs, chats).
- You frequently read a related group of fields together.
- You want flexible modeling while still enforcing application data rules.
- Your team understands the indexing and query patterns your application needs.

Consider a relational database too when complex joins, rigid relational integrity, or a heavily relational model are central to your application.

For example, a chat app might store each message as a document:

```js
{
  conversationId: "chat_101",
  senderId: "user_1",
  text: "Hello!",
  createdAt: ISODate("2026-10-17T10:00:00Z")
}
```

This is an illustrative document; a real schema would typically use consistent ID types and indexes.

## 8. Common mistakes

- Thinking MongoDB stores plain JSON: it stores **BSON**.
- Thinking one document equals one database: documents live inside collections.
- Assuming "NoSQL" means no querying or no schema rules.
- Assuming MongoDB is always faster than PostgreSQL.
- Adding arbitrary fields without validation or planning your queries.

## 9. Practice

1. Describe database → collection → document in your own words.
2. Design a `users` document with `name`, `email`, `address`, and `skills`.
3. Explain why an in-memory array is not a reliable primary database.
4. Predict what changes when you insert a document without `_id`.

## 10. Interview questions

**Q: What is MongoDB?** A NoSQL document database that stores BSON documents in collections.

**Q: Is MongoDB schema-less?** It has a flexible schema by default, but you can and should enforce validation rules as needed.

**Q: Can MongoDB handle relationships?** Yes. You can embed related data or store references between documents.

**Q: Is MongoDB always better than SQL?** No. Pick based on the data model, workload, and requirements.

## Quick revision

**MongoDB = database system → database → collection → BSON document.**

**Next:** [Lesson 002 — MongoDB vs Mongoose vs Atlas](./002-mongodb-vs-mongoose-vs-atlas.md)

**Official reference:** [MongoDB Documents](https://www.mongodb.com/docs/manual/core/document/)
