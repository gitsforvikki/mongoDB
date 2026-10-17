# Lesson 008 — MongoDB CRUD Mental Model

[← Lesson 007](./007-mongosh-essentials-basic-commands.md) · [Section 1](./README.md) · [Next: Section 2 →](../02-crud-and-querying/README.md)

## 1. What is CRUD and why do we need it?

**CRUD** means **Create, Read, Update, Delete**. These are the four basic actions almost every application performs on stored data.

Imagine a shopping application:
- **Create:** An admin adds a new product.
- **Read:** A customer opens the product listing.
- **Update:** The admin changes the price or stock.
- **Delete:** The admin removes a discontinued product.

```text
Frontend (React)
      |
      v
Backend API (Express)
      |
      v
MongoDB collection
  ├─ Create → insert a document
  ├─ Read   → find documents
  ├─ Update → change matching documents
  └─ Delete → remove matching documents
```

MongoDB stores data; the backend controls which operations users are allowed to perform.

## 2. Start with one real example

Run the following in **mongosh**, connected to a local MongoDB server or an Atlas deployment.

```js
use shopDB

db.products.insertMany([
  { name: "Laptop", price: 55000, stock: 5, category: "Electronics" },
  { name: "Mouse", price: 800, stock: 20, category: "Accessories" },
  { name: "Keyboard", price: 1500, stock: 10, category: "Accessories" }
])
```

We now have three product documents. MongoDB generates a unique `_id` for each document if it is omitted.

## 3. C — Create: insert a document

```js
db.products.insertOne({
  name: "Monitor",
  price: 12000,
  stock: 8,
  category: "Electronics"
})
```

**Execution flow:** mongosh sends an insert command → MongoDB writes the document → the result contains `acknowledged: true` and `insertedId` if the write is acknowledged.

Use `insertOne()` for one document and `insertMany()` for multiple documents. In production, validate data and handle possible duplicate-key errors.

## 4. R — Read: retrieve documents

```js
db.products.find()                                // all products
db.products.findOne({ name: "Laptop" })           // one matching product
db.products.find({ category: "Accessories" })     // matching products
db.products.find({ price: { $gt: 1000 } })        // price greater than 1000
```

**Execution flow:** MongoDB evaluates the **filter** and returns matching documents. `find()` returns a cursor; `findOne()` returns one document or `null`.

For `price: { $gt: 1000 }`, the matching initial products are Laptop and Keyboard; Monitor also matches after you insert it.

**Important:** If multiple documents match `findOne()`, do not rely on which one is chosen unless your query and sorting make the selection deterministic.

## 5. U — Update: change stored data

```js
db.products.updateOne(
  { name: "Laptop" },
  { $set: { price: 52000 } }
)
```

**Execution flow:**
1. The first object is the **filter**: find a document whose name is Laptop.
2. The second object is the **update**: set its price to 52000.
3. MongoDB returns fields such as `matchedCount` and `modifiedCount`.

To increase stock without reading and rewriting the old value:

```js
db.products.updateOne(
  { name: "Mouse" },
  { $inc: { stock: 5 } }
)
```

Now Mouse stock changes from 20 to 25 (assuming this example's initial data and no concurrent changes).

`updateOne()` affects at most one matching document; `updateMany()` affects all matching documents. MongoDB guarantees atomicity for a single-document write, including updates of multiple fields in that document.

**Why `$set`?** It changes the specified field without replacing the whole document. `replaceOne()` is the separate operation for replacing an entire document (except its immutable `_id`).

## 6. D — Delete: remove data

```js
db.products.deleteOne({ name: "Keyboard" })
```

MongoDB removes at most one matching document and returns a result with `deletedCount`.

```js
db.products.deleteMany({ category: "Accessories" })
```

This removes **all** remaining products matching that category. Always check your filter: `deleteMany({})` removes every document in the collection!

**Production note:** Sometimes a business prefers a soft delete such as `isDeleted: true` instead of physically deleting data, for audit/history reasons. This is an application design choice, not MongoDB's default delete behavior.

## 7. How CRUD maps to REST APIs

| User action | Typical HTTP request | MongoDB operation |
| --- | --- | --- |
| Add product | `POST /api/products` | `insertOne()` |
| List products | `GET /api/products` | `find()` |
| Get one product | `GET /api/products/:id` | `findOne()` |
| Edit product | `PATCH /api/products/:id` | `updateOne()` |
| Remove product | `DELETE /api/products/:id` | `deleteOne()` |

This is a **typical API design**, not an automatic MongoDB mapping. The Express controller decides which query to execute, handles validation, and sends the HTTP response.

```text
PATCH /api/products/123
        |
        v
Express controller validates request and authorization
        |
        v
MongoDB updateOne({ _id: ... }, { $set: ... })
        |
        v
Controller checks matchedCount and returns HTTP response
```

You will implement real Express 5 + Mongoose controllers in Section 8.

## 8. MongoDB vs Mongoose CRUD

The same idea looks slightly different with Mongoose:

| MongoDB shell | Mongoose model |
| --- | --- |
| `db.products.insertOne(data)` | `Product.create(data)` |
| `db.products.find(filter)` | `Product.find(filter)` |
| `db.products.findOne(filter)` | `Product.findOne(filter)` |
| `db.products.updateOne(filter, update)` | `Product.updateOne(filter, update)` |
| `db.products.deleteOne(filter)` | `Product.deleteOne(filter)` |

The database still performs the actual storage operations. Mongoose adds schema/model features, casting, validation behavior, and middleware. **Do not assume all Mongoose update methods run the same validation or hooks as `save()`**; you will study those differences later.

## 9. Key concepts you must remember

- **Filter:** Determines which documents match, e.g. `{ name: "Laptop" }`.
- **Update operator:** Describes how a field changes, e.g. `$set` or `$inc`.
- **Single vs many:** `updateOne`/`deleteOne` affect at most one; `updateMany`/`deleteMany` can affect multiple.
- **Acknowledged result:** Confirms the server acknowledged a write; it does not mean a matching document was necessarily changed.
- **No match:** Reads can return nothing; updates/deletes can report zero matches without throwing an error.
- **Atomicity:** A single-document write is atomic. Multiple separate writes are not automatically one all-or-nothing transaction.
- **Indexes:** Can make frequently used queries faster, but they have write/storage costs; see Section 4.
- **Safety:** Validate inputs, check authorization, and avoid untrusted filters and accidental mass updates/deletes.

## 10. Hands-on practice

Use a fresh practice collection so repeated exercises do not depend on old data:

```js
use shopDB
db.practiceProducts.insertMany([
  { name: "Pen", price: 20, stock: 100 },
  { name: "Notebook", price: 80, stock: 30 },
  { name: "Bag", price: 900, stock: 12 }
])
```

Try these tasks **without copying a solution**:
1. Insert one product called Bottle with price 250 and stock 15.
2. Find products whose price is greater than 50.
3. Find one product named Notebook.
4. Increase Pen stock by 10 using `$inc`.
5. Change Bag price to 850 using `$set`.
6. Delete the Bottle document.
7. Verify the final collection using `find()`.

Run the setup once, or clear/recreate only your **practice** collection before retrying.

## 11. Interview questions

**Q: What is CRUD?** The four basic database operations: Create, Read, Update and Delete.

**Q: Difference between `find()` and `findOne()`?** `find()` returns a cursor for matching documents; `findOne()` returns one document or null.

**Q: Difference between `$set` and `$inc`?** `$set` assigns a value; `$inc` increases or decreases a numeric field by an amount.

**Q: Does `updateOne()` throw if no document matches?** Usually no. Inspect `matchedCount` and `modifiedCount`.

**Q: Are multi-document updates atomic as a group?** No. Each individual document update is atomic; use a transaction when you need an appropriate all-or-nothing operation across documents.

## Quick revision

**Create → insert. Read → find. Update → modify matched documents. Delete → remove matched documents.**

**Next:** [Section 2 — MongoDB CRUD & Querying](../02-crud-and-querying/README.md), where each operation, filter, and update operator is explored in depth.

**Official references:** [MongoDB CRUD Operations](https://www.mongodb.com/docs/manual/crud/) · [MongoDB Query Documents](https://www.mongodb.com/docs/manual/tutorial/query-documents/)
