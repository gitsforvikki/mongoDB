# Lesson 005 — Databases, Collections and Documents

[← Section 1](./README.md) · [Full curriculum](../README.md)

## 1. Understand the hierarchy
```text
MongoDB deployment
└── shopDB (database)
    ├── products (collection)
    │   ├── { _id: ..., name: "Phone", price: 12000 } (document)
    │   └── { _id: ..., name: "Laptop", price: 55000 }
    └── orders (collection)
        └── { _id: ..., total: 12000 }
```
A **database** groups collections. A **collection** groups documents. A **document** is one record represented as BSON.

## 2. Try it in mongosh
```js
show dbs
use shopDB
db
db.products.insertOne({
  name: "Phone",
  price: 12000,
  category: "Electronics",
  specifications: { ram: "8 GB", storage: "128 GB" },
  tags: ["mobile", "android"]
})
show collections
db.products.find()
db.products.findOne({ name: "Phone" })
```
`use shopDB` selects the database; the database is not necessarily created on disk until data is written. `db.products` selects the collection. The first insert can implicitly create that collection.

## 3. Why documents are useful
An order can keep related item details together:
```js
db.orders.insertOne({
  customerName: "Ravi",
  items: [
    { productName: "Phone", quantity: 1, price: 12000 },
    { productName: "Case", quantity: 2, price: 300 }
  ],
  total: 12600,
  status: "placed"
})
```
The `items` array and its embedded objects are part of the same order document. This makes reading one order convenient, but not every relationship should be embedded; you will study referencing and trade-offs in Section 3.

## 4. Important behaviors
- Every stored document has a unique `_id` within its collection. If omitted, MongoDB generates one (usually an ObjectId).
- Collections are schema-flexible by default; MongoDB supports schema validation.
- Documents in different collections may have similar fields without being the same record.
- Collections and databases can be created explicitly, but normal inserts often create them implicitly.
- A BSON document has a size limit of **16 MiB**. Use other approaches such as GridFS for larger files.

## 5. Avoid these mistakes
- Treating `use databaseName` as proof the database already exists.
- Thinking `db.products` means the products collection exists before writing.
- Using a separate collection for every user or every individual order without a strong reason.
- Assuming MongoDB automatically enforces relationships like relational foreign keys.

## Practice
Create `shopDB.users` with two documents, then list collections. Insert a document with a nested address and an array of skills. Query one user by name.

## Interview check
**Collection vs document?** A collection is a group; a document is an individual BSON record.
**Why `_id`?** It identifies each document uniquely within its collection.

**Reference:** [Databases and collections](https://www.mongodb.com/docs/manual/core/databases-and-collections/)
