# Lesson 016 — updateOne, updateMany and replaceOne

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why three methods?
Sometimes you want to edit one product, update every discontinued product, or replace an entire document. These operations have different risks.

## Dataset
```js
use mongoCourse
db.editDemo.insertMany([
 {_id:1,name:"Mouse",price:700,category:"Accessories",active:true},
 {_id:2,name:"Keyboard",price:1500,category:"Accessories",active:true},
 {_id:3,name:"Laptop",price:55000,category:"Electronics",active:true}
])
```

## updateOne: at most one match
```js
db.editDemo.updateOne({_id:1},{$set:{price:650}})
```
MongoDB finds one matching document and changes price only. Use a unique `_id` for predictable targeting. If a nonunique filter matches several documents, do not assume which one is chosen.

## updateMany: all matches
```js
db.editDemo.updateMany({category:"Accessories"},{$set:{onSale:true}})
```
Both Mouse and Keyboard receive `onSale:true`. **Each document update is atomic**, but the entire multi-document operation is not automatically one atomic transaction.

## replaceOne: replace whole document
```js
db.editDemo.replaceOne(
 {_id:2},
 {name:"Mechanical Keyboard",price:2400,category:"Accessories"}
)
```
The old `active` and `onSale` fields disappear. `_id` stays unchanged. Replacement documents cannot use update operators like `$set` as the replacement body.

## Read the result
Typical acknowledged result:
```js
{ acknowledged:true, matchedCount:1, modifiedCount:1, upsertedCount:0 }
```
`matchedCount` tells how many matched; `modifiedCount` how many actually changed. If the new value equals the old value, a matched update can have `modifiedCount:0`.

## Upsert option
```js
db.editDemo.updateOne(
 {_id:4},
 {$set:{name:"Monitor",price:12000}},
 {upsert:true}
)
```
If no match exists, MongoDB inserts a document and reports `upsertedId`. See Lesson 018 for concurrency and upsert details.

## Mongoose equivalent
```js
await Product.updateOne({_id:id},{$set:{price:650}});
await Product.updateMany({category:"Accessories"},{$set:{onSale:true}});
await Product.replaceOne({_id:id},{name:"New Keyboard",price:2400});
```
Mongoose update validation and middleware differ from `document.save()`. Be careful not to unintentionally erase fields with replacement operations.

## Safety
- Never run `updateMany({})` on real data without deliberately intending to update everything.
- Verify filters with `find()` first.
- Enforce authorization before updating records.
- Use specific filters, and check matched counts for not-found responses.
- Transactions are needed for some multi-document all-or-nothing workflows; not for every update.

## Practice
Change one price; mark all Accessories on sale; replace one test document and inspect which fields disappeared; repeat an unchanged update and inspect `modifiedCount`.

## Interview check
**updateOne vs updateMany?** At most one vs all matches. **updateOne vs replaceOne?** Targeted field update vs full document replacement.

[Official updateOne](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateOne/) · [replaceOne](https://www.mongodb.com/docs/manual/reference/method/db.collection.replaceOne/)
