# Lesson 010 — Read Documents: find and findOne

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why read data?
When a customer opens a catalog, your API must retrieve stored products. A **filter** selects matching documents; a **projection** chooses which fields to return.

## Practice dataset
Run once in `mongosh`:
```js
use mongoCourse
db.readDemo.insertMany([
 {name:"Laptop",category:"Electronics",price:55000,stock:5},
 {name:"Mouse",category:"Accessories",price:700,stock:20},
 {name:"Keyboard",category:"Accessories",price:1500,stock:0},
 {name:"Monitor",category:"Electronics",price:12000,stock:8}
])
```
For repeatable exercises, use a new practice collection or clear only this disposable dataset before seeding.

## find() and findOne()
```js
db.readDemo.find() // cursor over all documents
db.readDemo.find({category:"Accessories"}) // Mouse and Keyboard
db.readDemo.findOne({name:"Laptop"}) // one document
db.readDemo.findOne({name:"Missing"}) // null
```
Execution: mongosh sends a query → MongoDB matches filters (using an index if suitable) → results are returned. **find() returns a cursor**, not a plain array; `findOne()` returns a document or null. No sort means **no guaranteed order**.

## Filtering and projections
```js
db.readDemo.find({price:{$gte:1000}}, {name:1,price:1,_id:0})
db.readDemo.find({category:"Electronics",stock:{$gt:0}})
db.readDemo.find({}, {stock:0})
```
`{name:1,price:1,_id:0}` includes only name and price; `_id` is included by default unless excluded. You generally cannot mix inclusion and exclusion except for `_id`.

## Cursors, sorting, and limiting
```js
db.readDemo.find().sort({price:-1,_id:1}).limit(2)
db.readDemo.find().toArray() // shell: materialize all results, use only for small datasets
```
Sort `-1` is descending; `1` is ascending. `limit(2)` avoids returning more than two records. Never load a huge collection into memory unnecessarily.

## Mongoose equivalent
After defining a Product model and connecting:
```js
const products = await Product.find({price:{$gte:1000}})
  .select("name price -_id")
  .sort({price:-1})
  .limit(10)
  .lean();
const one = await Product.findOne({name:"Laptop"});
```
Mongoose queries are thenables, and `await` executes them. `lean()` returns plain objects instead of hydrated Mongoose documents.

## Common mistakes
- Expecting `find()` to return one document rather than a cursor.
- Assuming `findOne()` selects the newest record without sorting.
- Returning sensitive fields by default.
- Fetching an unbounded dataset for an API.
- Treating a missing match as an exception instead of an empty cursor or null.

## Practice
Find in-stock Electronics, find a missing product, return only product names, and sort prices descending.

## Interview checks
**find vs findOne?** Cursor versus single document/null. **What is projection?** Selecting returned fields. **Why index?** To avoid scanning too many documents for common queries.

[Official MongoDB reference](https://www.mongodb.com/docs/manual/tutorial/query-documents/)
