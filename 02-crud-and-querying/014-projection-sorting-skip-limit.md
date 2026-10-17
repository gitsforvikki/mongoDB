# Lesson 014 — Projection, Sorting, Skip and Limit

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why do these matter?
A product listing may contain thousands of products, but the UI needs only **10 items per page**, sorted by price, with just name and price fields.

## Dataset
```js
use mongoCourse
db.pageDemo.insertMany([
 {_id:1,name:"Pen",price:20,category:"Stationery",internalNote:"supplier-A"},
 {_id:2,name:"Notebook",price:80,category:"Stationery",internalNote:"supplier-B"},
 {_id:3,name:"Bag",price:900,category:"Accessories",internalNote:"supplier-C"},
 {_id:4,name:"Bottle",price:250,category:"Accessories",internalNote:"supplier-D"},
 {_id:5,name:"Pencil",price:10,category:"Stationery",internalNote:"supplier-E"}
])
```

## 1. Projection: return only needed fields
```js
db.pageDemo.find({}, {name:1,price:1,_id:0})
db.pageDemo.find({}, {internalNote:0})
```
Projection controls which fields are returned. `1` includes, `0` excludes; mixing inclusion and exclusion is generally not allowed except `_id`. Projection helps reduce data transferred but **is not a substitute for authorization**.

## 2. Sorting: order the results
```js
db.pageDemo.find().sort({price:1,_id:1}) // cheapest first
db.pageDemo.find().sort({price:-1,_id:1}) // expensive first
```
`1` = ascending; `-1` = descending. Add a unique tie-breaker such as `_id` for stable pagination when sort values repeat. Without a sort, MongoDB does not guarantee result order.

## 3. Skip and limit: basic pagination
```js
db.pageDemo.find({}, {name:1,price:1,_id:0})
  .sort({price:1,_id:1})
  .skip(0)
  .limit(2) // page 1: Pencil, Pen

db.pageDemo.find({}, {name:1,price:1,_id:0})
  .sort({price:1,_id:1})
  .skip(2)
  .limit(2) // page 2: Notebook, Bottle
```
`skip(n)` skips the first n results in the sorted result set; `limit(n)` caps the count.

**Page formula:** `skip = (page - 1) * pageSize`. Example: page 3 at 10 items/page → skip 20.

## 4. Full product listing query
```js
db.pageDemo.find(
  {category:"Stationery"},
  {name:1,price:1,_id:0}
).sort({price:1,_id:1}).skip(0).limit(2)
```
Flow: filter matches → sort establishes order → skip advances past earlier results → limit returns page size → projection determines fields sent back. The database optimizer may execute internal steps differently while preserving query semantics.

## 5. Mongoose equivalent
```js
const page = 2;
const pageSize = 10;
const products = await Product.find({category:"Stationery"})
  .select("name price")
  .sort({price:1,_id:1})
  .skip((page - 1) * pageSize)
  .limit(pageSize)
  .lean();
```
Validate and cap `page` and `pageSize` in a real API. `select()` chooses fields; `lean()` avoids Mongoose document hydration when you only need plain data.

## 6. Performance and correctness
- Large `skip()` values can be expensive because MongoDB must traverse skipped results. Learn **cursor/keyset pagination** in Lesson 020.
- Sorting can require significant work without a suitable index.
- Concurrent inserts/deletes between page requests can cause duplicates or omissions in offset pagination.
- Do not expose internal/sensitive fields just because they exist in the database.
- A projection that hides a password is helpful, but access control and safe schema design are still necessary.

## Practice
Show the two cheapest products; show the second page; return only name and price; filter Stationery and sort expensive-first.

## Interview checks
**Why projection?** Reduce returned fields. **Why stable sorting?** Consistent pagination. **Why can skip be slow?** Large offsets still require traversing results.

[Projection](https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/) · [Sort](https://www.mongodb.com/docs/manual/reference/method/cursor.sort/) · [Skip](https://www.mongodb.com/docs/manual/reference/method/cursor.skip/)
