# Lesson 017 — Delete Documents and Bulk Writes

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why delete and bulk write?
Deleting removes unwanted records. Bulk writing combines many insert/update/replace/delete operations into fewer client requests, useful for imports and synchronization.

## Dataset
```js
use mongoCourse
db.bulkDemo.insertMany([
 {_id:1,name:"Pen",stock:10,archived:false},
 {_id:2,name:"Notebook",stock:0,archived:false},
 {_id:3,name:"Bag",stock:4,archived:true}
])
```

## deleteOne and deleteMany
```js
db.bulkDemo.deleteOne({_id:2}) // at most one
db.bulkDemo.deleteMany({archived:true}) // all matching
```
Acknowledged results include `deletedCount`. A missing match returns zero; it is not normally an exception.

**Warning:** `deleteMany({})` removes all documents in the collection. Always preview the filter with `find()`.

## Soft delete vs hard delete
Hard delete physically removes the document:
```js
db.bulkDemo.deleteOne({_id:1})
```
Soft delete retains it but hides it from normal queries:
```js
db.bulkDemo.updateOne({_id:1},{$set:{deletedAt:new Date()}})
db.bulkDemo.find({deletedAt:{$exists:false}})
```
Soft delete supports recovery/audit but requires consistent filtering and a retention policy.

## bulkWrite: mixed operations
Use a **fresh** collection for this independent example:
```js
db.bulkWriteDemo.bulkWrite([
 {insertOne:{document:{_id:1,name:"Mouse",stock:10}}},
 {insertOne:{document:{_id:2,name:"Keyboard",stock:5}}},
 {updateOne:{filter:{_id:1},update:{$inc:{stock:3}}}},
 {replaceOne:{filter:{_id:2},replacement:{name:"Mechanical Keyboard",stock:7}}},
 {deleteOne:{filter:{_id:99}}}
])
db.bulkWriteDemo.find()
```
Expected: Mouse stock 13; Mechanical Keyboard stock 7. Deleting nonexistent `_id:99` changes nothing.

Bulk results expose counts for inserted, matched, modified, deleted, and upserted documents. Bulk operations may partially succeed if errors occur.

## Ordered vs unordered
```js
db.bulkWriteDemo.bulkWrite([
 {insertOne:{document:{_id:3,name:"A"}}},
 {insertOne:{document:{_id:3,name:"Duplicate"}}},
 {insertOne:{document:{_id:4,name:"B"}}}
],{ordered:false})
```
The duplicate `_id` triggers an error, but unordered mode still attempts independent operations. Do not rely on their execution order. Ordered mode stops attempting later operations on an error. Neither is an all-or-nothing transaction.

## Mongoose equivalent
```js
await Product.deleteOne({_id:id});
await Product.deleteMany({archived:true});
await Product.bulkWrite([
 {updateOne:{filter:{_id:id},update:{$inc:{stock:1}}}}
]);
```
Mongoose bulk write behavior and middleware differ from individual `save()` operations; validate assumptions for your installed version.

## Safety and performance
- Preview destructive filters.
- Require appropriate authorization and backups.
- For large operations, batch sensibly and monitor load.
- A bulk request reduces round trips but does not make every write atomic as a group.
- Avoid unbounded deletions during user-facing requests when background processing is more appropriate.

## Practice
Seed a fresh collection, delete one ID, soft-delete another, then bulk-insert and update records. Verify counts and surviving documents.

## Interview check
**Hard vs soft delete?** Remove document vs mark it inactive/deleted. **Is bulkWrite transactional?** Not automatically.

[Delete documents](https://www.mongodb.com/docs/manual/tutorial/remove-documents/) · [bulkWrite](https://www.mongodb.com/docs/manual/reference/method/db.collection.bulkWrite/)
