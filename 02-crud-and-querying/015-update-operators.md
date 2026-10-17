# Lesson 015 — Update Operators

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why update operators?
A product's price, stock, and tags change over time. MongoDB update operators change **specific fields** without replacing the whole document.

## Prepare data in mongosh
```js
use mongoCourse
db.updateDemo.insertOne({
 _id:1,name:"Laptop",price:55000,stock:5,
 tags:["electronics"],specs:{ram:"8GB"},discontinued:false
})
```

## $set: assign a value
```js
db.updateDemo.updateOne({_id:1},{$set:{price:52000,"specs.ram":"16GB"}})
```
Sets price and nested RAM. Creates the field if absent.

## $unset: remove a field
```js
db.updateDemo.updateOne({_id:1},{$unset:{discontinued:""}})
```
Removes the field; it does not set it to null.

## $inc and $mul: numeric changes
```js
db.updateDemo.updateOne({_id:1},{$inc:{stock:3}}) // 5 → 8
db.updateDemo.updateOne({_id:1},{$mul:{price:0.9}}) // 52000 → 46800
```
`$inc` is atomic within the document, useful for counters and stock adjustments. A missing field can be created by `$inc`; applying it to a null or nonnumeric field fails. For monetary values, plan numeric precision deliberately.

## $min, $max, $rename, $currentDate
```js
db.updateDemo.updateOne({_id:1},{$min:{price:45000}}) // smaller value wins
db.updateDemo.updateOne({_id:1},{$max:{stock:10}}) // larger value wins
db.updateDemo.updateOne({_id:1},{$rename:{"specs.ram":"specs.memory"}})
db.updateDemo.updateOne({_id:1},{$currentDate:{updatedAt:true}})
```
`$currentDate` writes the current date; `$rename` changes a field's name. Test renames carefully with nested data.

## Array operators: $push vs $addToSet
```js
db.updateDemo.updateOne({_id:1},{$push:{tags:"sale"}})
db.updateDemo.updateOne({_id:1},{$push:{tags:"sale"}}) // duplicate allowed
db.updateDemo.updateOne({_id:1},{$addToSet:{tags:"featured"}})
db.updateDemo.updateOne({_id:1},{$addToSet:{tags:"featured"}}) // no new duplicate
```
`$addToSet` prevents adding another matching value, but does not clean up duplicates already present. For multiple additions:
```js
db.updateDemo.updateOne({_id:1},{$addToSet:{tags:{$each:["new","popular"]}}})
```
`$push` supports `$each`, `$position`, `$sort`, and `$slice` modifiers for ordered/bounded arrays.

## $pull and $pop: remove array elements
```js
db.updateDemo.updateOne({_id:1},{$pull:{tags:"sale"}}) // removes all matching "sale" entries
db.updateDemo.updateOne({_id:1},{$pop:{tags:1}}) // removes last element; -1 removes first
```
Use `$pull` with a condition to remove matching embedded array elements.

## $setOnInsert: only when upserting
```js
db.updateDemo.updateOne(
 {_id:2},
 {$set:{name:"Mouse"},$setOnInsert:{stock:0,createdAt:new Date()}},
 {upsert:true}
)
```
`$setOnInsert` only applies when the upsert inserts a new document.

## Execution flow and response
MongoDB finds the matching document → applies update operators atomically to that document → returns `matchedCount`, `modifiedCount`, and sometimes `upsertedId`. A match may not change any value (`modifiedCount:0`).

## Mongoose equivalent
```js
await Product.updateOne({_id:productId},{$inc:{stock:1},$set:{price:50000}});
```
MongoDB defines the operators. Mongoose provides model-based casting; update validators and hooks have their own rules (later lessons).

## Common mistakes
- Writing `{price:500}` instead of `{$set:{price:500}}` in operator-style `updateOne`.
- Assuming `$push` removes duplicates.
- Using read → modify → write for counters when `$inc` is safer.
- Allowing arbitrary update paths from user input.
- Assuming array updates always target one element; positional operators and array filters are separate concepts.

## Practice
Increase stock by 2, remove an obsolete field, add a tag only once, remove all matching tags, and update a nested field. Verify each result using `findOne()`.

## Interview check
**$set vs $unset?** Assign vs remove. **$push vs $addToSet?** Append vs avoid adding a duplicate. **Why $inc?** Atomic numeric change.

[Official update operators](https://www.mongodb.com/docs/manual/reference/operator/update/)
