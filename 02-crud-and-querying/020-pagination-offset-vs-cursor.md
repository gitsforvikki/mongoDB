# Lesson 020 — Pagination: Offset vs Cursor

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why paginate?
Returning 100,000 products in one API response wastes bandwidth and memory. Pagination returns a small page of results.

## Dataset
```js
use mongoCourse
db.paginationDemo.insertMany([
 {_id:1,name:"A",price:100},
 {_id:2,name:"B",price:100},
 {_id:3,name:"C",price:200},
 {_id:4,name:"D",price:300},
 {_id:5,name:"E",price:300},
 {_id:6,name:"F",price:400}
])
```

## 1. Offset pagination: skip and limit
```js
db.paginationDemo.find().sort({price:1,_id:1}).skip(0).limit(2) // A, B
db.paginationDemo.find().sort({price:1,_id:1}).skip(2).limit(2) // C, D
```
Formula: `skip = (page - 1) * pageSize`.

**Advantages:** Easy page numbers and jumping to a specific page.

**Problems:** Large offsets require MongoDB to advance past many results; inserts/deletes between requests can shift page boundaries and cause duplicates or missed records.

## 2. Cursor/keyset pagination: continue after last item
Use a stable sort on **price, then unique _id**:
```js
db.paginationDemo.find().sort({price:1,_id:1}).limit(2) // A, B
```
The last record has `price:100, _id:2`. The next page should include documents **after that pair**:

```js
db.paginationDemo.find({
 $or:[
  {price:{$gt:100}},
  {price:100,_id:{$gt:2}}
 ]
}).sort({price:1,_id:1}).limit(2) // C, D
```

If the last record on page 2 is D (`price:300,_id:4`), continue with `price > 300 OR (price == 300 AND _id > 4)`. The tie-breaker prevents skipping records with equal prices.

## 3. Why indexes matter
```js
db.paginationDemo.createIndex({price:1,_id:1})
```
A compound index matching filter/sort patterns can make keyset pagination efficient. Inspect `explain("executionStats")` to validate actual query behavior.

## 4. Mongoose example
```js
const pageSize = 2;
const lastPrice = 100;
const lastId = 2; // this example uses numeric _id; real apps often use ObjectId
const products = await Product.find({
 $or:[
  {price:{$gt:lastPrice}},
  {price:lastPrice,_id:{$gt:lastId}}
 ]
}).sort({price:1,_id:1}).limit(pageSize).lean();
```
In a real API, use the schema's actual ID type, validate cursor input, and encode cursor values into an opaque token rather than trusting raw query parameters.

## 5. API response design
```json
{
 "data":[{"_id":3,"name":"C","price":200},{"_id":4,"name":"D","price":300}],
 "nextCursor":"opaque-token-for-price-300-id-4",
 "hasMore":true
}
```
The cursor string above is illustrative, not a working token. To determine `hasMore`, fetch `pageSize + 1` items and return only `pageSize`; encode the last returned item's sort keys. Cursor tokens should be validated and, where appropriate, signed.

## 6. Which should you choose?
| Requirement | Offset | Cursor/keyset |
| --- | --- | --- |
| Jump to page 20 | Easy | Not directly |
| Deep pages | Often expensive | Usually more efficient with index |
| Frequent inserts/deletes | Pages can shift | More stable continuation |
| Implementation | Simpler | Requires stable sort and cursor |
| Common use | Small admin tables | Infinite scroll, feeds, large datasets |

**Important:** Cursor pagination is not a snapshot. If documents change sort keys or are deleted between requests, results can still shift. Stable immutable sort keys are often preferable for feeds.

## 7. Common mistakes
- Sorting only by nonunique price and losing stable ordering.
- Using `skip(100000)` without considering performance.
- Forgetting the same filter and sort must apply across pages.
- Treating client-supplied cursors as trusted database filters.
- Assuming cursor pagination always prevents duplicates under concurrent edits.

## Practice
Fetch three pages of two products with offset pagination. Then repeat using the last `price,_id` pair. Explain why `_id` is needed when prices tie.

## Interview check
**Why keyset pagination?** Efficient continuation without deep skipping. **Why unique tie-breaker?** Stable order. **Is it perfectly consistent under updates?** No, not automatically.

[Cursor skip](https://www.mongodb.com/docs/manual/reference/method/cursor.skip/) · [Compound indexes](https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/)
