# Lesson 009 — Insert Documents: insertOne and insertMany

[← Section 2](./README.md) · [Full curriculum](../README.md)

## 1. What does inserting mean?

**Insert** means saving a new document inside a MongoDB collection. Example: when an admin adds a product, the application must store it so customers can see it later.

```text
Product form → Express API → MongoDB insert → products collection
```

This lesson uses **mongosh** to learn MongoDB behavior. A short Mongoose comparison appears at the end.

## 2. Prepare a practice database

Connect using `mongosh`, then run:

```js
use shopLearning
db.products.find()
```

If the collection does not exist yet, that is fine: the first successful insert can create it automatically.

## 3. insertOne() — save one document

```js
db.products.insertOne({
  name: "Laptop",
  price: 55000,
  stock: 5,
  category: "Electronics",
  specifications: { ram: "16 GB", storage: "512 GB" },
  tags: ["computer", "work"]
})
```

**What happens?**
1. `db` refers to `shopLearning`; `products` is the collection.
2. `insertOne()` receives **one document** (a BSON object).
3. MongoDB stores the document and creates the collection if needed.
4. If `_id` is missing, the driver or server normally generates an ObjectId.
5. With acknowledged writes, the result includes `acknowledged: true` and `insertedId`.

Example response (your ObjectId will differ):

```js
{
  acknowledged: true,
  insertedId: ObjectId("507f1f77bcf86cd799439011")
}
```

Verify:

```js
db.products.findOne({ name: "Laptop" })
```

**Why useful?** Creating one user, one order, or one product at a time.

## 4. insertMany() — save several documents

```js
db.products.insertMany([
  { name: "Mouse", price: 700, stock: 25 },
  { name: "Keyboard", price: 1600, stock: 10 },
  { name: "Monitor", price: 12000, stock: 8 }
])
```

The argument is an **array of documents**, not one document. The acknowledged result contains `acknowledged` and `insertedIds` (IDs keyed by input position).

**Why useful?** Importing a product catalog, seeding test data, or inserting several records in a batch. It generally avoids making a separate client-to-server call for each document.

## 5. insertOne vs insertMany

| Feature | insertOne | insertMany |
| --- | --- | --- |
| Input | One document | Array of documents |
| Result ID | `insertedId` | `insertedIds` |
| Common use | Registration, single product | Imports, initial data |
| Failure | One insert fails | Some inserts may succeed before a failure |
| Options | Write concern | Ordered/unordered, write concern |

**Important:** `insertMany()` is not automatically an all-or-nothing transaction.

## 6. How _id and uniqueness work

Every document has an `_id`, unique **within that collection**.

```js
db.products.insertOne({ _id: "product-101", name: "Desk" })
```

Trying the same `_id` again fails:

```js
db.products.insertOne({ _id: "product-101", name: "Chair" })
// E11000 duplicate key error
```

The `_id` can be a supported BSON type, but using automatically generated ObjectIds is common. A duplicate `email` or `sku` is **not** automatically rejected: create a unique index if the business requires uniqueness (covered in Section 4).

## 7. Ordered vs unordered bulk insertion

**Ordered** is the default. If an insert fails, later documents are not attempted.

Use a fresh practice collection for this example:

```js
db.orderedDemo.insertMany([
  { _id: 1, name: "A" },
  { _id: 1, name: "Duplicate" },
  { _id: 2, name: "B" }
], { ordered: true })
```

Expected: A is inserted, the second document causes a duplicate-key error, and B is not attempted. Inspect:

```js
db.orderedDemo.find()
```

**Unordered** continues attempting other documents even when one fails:

```js
db.unorderedDemo.insertMany([
  { _id: 1, name: "A" },
  { _id: 1, name: "Duplicate" },
  { _id: 2, name: "B" }
], { ordered: false })
```

Expected: a duplicate-key error is still reported, but A and B can be inserted. The server may reorder unordered writes; do not rely on execution order.

```js
db.unorderedDemo.find()
```

**When to choose?** Ordered when later writes should stop after a failure; unordered when independent valid records should still be attempted. Neither option provides transaction-wide atomicity.

## 8. What is write concern?

**Write concern** specifies the level of acknowledgment requested for a write. For example, `w: "majority"` asks for acknowledgment from a majority of data-bearing voting replica-set members according to MongoDB's write-concern rules.

```js
db.products.insertOne(
  { name: "Webcam", price: 2200 },
  { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

Write concern affects acknowledgment/durability guarantees, not whether the document has valid business fields. A write concern timeout does not always prove the write was rolled back; handle uncertain outcomes carefully. For ordinary learning examples, use your deployment's default write concern.

## 9. Validation, errors and safety

MongoDB collections are flexible by default. That means this may be accepted without schema validation:

```js
db.products.insertOne({ name: "Invalid product", price: "not a number" })
```

In production, validate incoming data and use appropriate database schema validation or Mongoose schemas. Common errors include duplicate keys (code `11000`), schema validation failures, connection errors, and write concern errors.

Other important details:
- A document has a **16 MiB BSON size limit**.
- `insertMany()` may be split into internal batches for large input.
- Do not assume a failed batch inserted zero documents.
- Avoid inserting passwords, secrets, or untrusted input without proper security handling.
- Do not repeatedly rerun examples with manually fixed `_id` values unless you expect duplicate-key errors.

## 10. Equivalent Mongoose operations

After connecting Mongoose and defining a `Product` model:

```js
// One product
const product = await Product.create({
  name: "Laptop",
  price: 55000
});

// Multiple products
const products = await Product.insertMany([
  { name: "Mouse", price: 700 },
  { name: "Keyboard", price: 1600 }
]);
```

MongoDB still performs the database write. **Mongoose adds model-based casting and validation**, and Mongoose `create()` and `insertMany()` have different middleware behavior (covered in the Mongoose section). Do not treat their return values as identical to mongosh's insert result objects.

## 11. Practice exercises

Create a new collection called `practiceBooks`:
1. Insert one book with title, author, price and categories.
2. Insert three books with `insertMany()`.
3. Read the documents with `find()` and identify their `_id` values.
4. Try a duplicate `_id` using `insertOne()`. Explain the error.
5. Use **separate new collections** to test `ordered: true` and `ordered: false`. Compare which records remain.

## 12. Interview questions

**Q: What is the difference between insertOne and insertMany?** One inserts a single document; the other inserts an array of documents.

**Q: Does insertMany roll back every document if one fails?** No, not by default.

**Q: What happens if _id is omitted?** The driver or server typically generates an ObjectId.

**Q: What is ordered: false?** MongoDB continues attempting other documents after an individual insert error, potentially in a different order.

**Q: Does MongoDB enforce required fields automatically?** No, not unless configured through schema validation or enforced by application code.

## Quick revision

**insertOne = one document. insertMany = array of documents. _id = unique identifier. Ordered = stop on error; unordered = continue attempts.**

**Next:** Lesson 010 — Read Documents: find and findOne (not yet published).

### Official references
- [insertOne](https://www.mongodb.com/docs/manual/reference/method/db.collection.insertOne/)
- [insertMany](https://www.mongodb.com/docs/manual/reference/method/db.collection.insertMany/)
- [Write Concern](https://www.mongodb.com/docs/manual/reference/write-concern/)
