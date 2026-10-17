# Lesson 013 — Nested Fields and Arrays

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why nested data?
A product can contain an `address` object, `tags` array, and multiple `reviews`. MongoDB lets you query fields inside these structures without splitting every value into separate collections.

## Dataset
```js
use mongoCourse
db.nestedDemo.insertMany([
 {_id:1,name:"Laptop",seller:{city:"Patna",rating:4.8},tags:["electronics","work"],reviews:[{user:"A",stars:5},{user:"B",stars:2}]},
 {_id:2,name:"Mouse",seller:{city:"Pune",rating:4.2},tags:["electronics","accessory"],reviews:[{user:"C",stars:4}]},
 {_id:3,name:"Notebook",seller:{city:"Patna",rating:4.5},tags:["stationery"],reviews:[]}
])
```

## Query embedded fields (dot notation)
```js
db.nestedDemo.find({"seller.city":"Patna"}) // Laptop, Notebook
db.nestedDemo.find({"seller.rating":{$gte:4.5}}) // Laptop, Notebook
```
**Why quotes?** A field path with a dot must be represented as a string key in JavaScript object syntax.

## Query arrays
```js
db.nestedDemo.find({tags:"electronics"}) // Laptop, Mouse
db.nestedDemo.find({tags:{$all:["electronics","work"]}}) // Laptop
db.nestedDemo.find({tags:{$in:["work","stationery"]}}) // Laptop, Notebook
db.nestedDemo.find({tags:{$size:1}}) // Notebook
```
`{tags:"electronics"}` matches an array containing that value. `$all` requires all listed elements; `$in` requires any listed value. `$size` matches an exact array length.

## Array of embedded documents: $elemMatch
```js
db.nestedDemo.find({
  reviews:{$elemMatch:{user:"A",stars:{$gte:4}}}
}) // Laptop
```
`$elemMatch` requires **the same array element** to satisfy all conditions.

Contrast:
```js
db.nestedDemo.find({
  "reviews.user":"A",
  "reviews.stars":{$lte:2}
}) // Laptop, but different review elements can satisfy each condition
```
The user A review has 5 stars; user B has 2. Dot-notation conditions without `$elemMatch` can match different array elements.

## Exact embedded document match
```js
db.nestedDemo.find({seller:{city:"Patna",rating:4.8}})
```
An exact embedded-document match is sensitive to field order and complete structure. Prefer dot notation for individual fields.

## Mongoose equivalent
```js
const products = await Product.find({
  "seller.city":"Patna",
  tags:"electronics"
});
const reviewed = await Product.find({
  reviews:{$elemMatch:{user:"A",stars:{$gte:4}}}
});
```

## Common mistakes
- Assuming dot-notation conditions on an array always refer to the same element.
- Forgetting to quote keys containing dots.
- Assuming `$size` accepts a numeric range; it matches an exact size.
- Embedding endlessly growing arrays inside a document (16 MiB BSON document limit).

## Practice
Find sellers in Patna, products tagged electronics, products with both electronics and work tags, and reviews where one reviewer gives at least 4 stars.

## Interview checks
**Why $elemMatch?** Multiple conditions must match one array element. **Embedding vs referencing?** A data-modeling decision based on access and growth.

[Nested documents](https://www.mongodb.com/docs/manual/tutorial/query-embedded-documents/) · [Arrays](https://www.mongodb.com/docs/manual/tutorial/query-arrays/) · [Arrays of documents](https://www.mongodb.com/docs/manual/tutorial/query-array-of-documents/)
