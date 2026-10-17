# Lesson 012 — Logical and Element Operators

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why these operators?
You may need a product that is **in stock AND inexpensive**, or a user who is **admin OR moderator**. You may also need to detect missing fields or check stored BSON types.

## Dataset
```js
use mongoCourse
db.logicDemo.insertMany([
 {_id:1,name:"Asha",age:24,role:"admin",active:true,phone:"111"},
 {_id:2,name:"Ravi",age:17,role:"member",active:true},
 {_id:3,name:"Neha",age:30,role:"moderator",active:false,phone:null},
 {_id:4,name:"Dev",age:22,role:"member",active:true,phone:"444"}
])
```

## Logical operators
```js
// AND: adult AND active (Asha, Dev)
db.logicDemo.find({$and:[{age:{$gte:18}},{active:true}]})

// OR: admin OR moderator (Asha, Neha)
db.logicDemo.find({$or:[{role:"admin"},{role:"moderator"}]})

// NOR: neither admin nor moderator (Ravi, Dev)
db.logicDemo.find({$nor:[{role:"admin"},{role:"moderator"}]})

// NOT: age not greater than 20 (Ravi)
db.logicDemo.find({age:{$not:{$gt:20}}})
```
Simple different-field conditions already imply AND: `{active:true,age:{$gte:18}}`. Use explicit `$and` when conditions need grouping or repeated operators.

**Important:** `$not` applies to an operator expression on a **field**, not to an arbitrary entire query. It may also match missing fields.

## Element operators
```js
db.logicDemo.find({phone:{$exists:true}}) // Asha, Neha, Dev
db.logicDemo.find({phone:{$exists:false}}) // Ravi
db.logicDemo.find({phone:{$type:"string"}}) // Asha, Dev
```
**Key distinction:** `phone:null` exists as a field, but is not a string. `{phone:null}` matches both explicit null **and missing** fields; combine `$exists` or `$type` when you need to distinguish them.

## Real example: moderation dashboard
```js
db.logicDemo.find({
  $or:[{role:"admin"},{role:"moderator"}],
  active:true
})
```
Only Asha matches: role must be privileged **and** active.

## Mongoose equivalent
```js
const people = await User.find({
  $or:[{role:"admin"},{role:"moderator"}],
  active:true
});
```
Mongoose passes MongoDB logical operators to the database, with schema-aware casting where applicable.

## Common mistakes
- Writing `$not` as if it accepts a top-level query document.
- Assuming `$exists:true` means the value is not null.
- Forgetting that `$ne`, `$nin`, and `$not` can match missing fields.
- Building filters directly from arbitrary user-supplied operator objects (NoSQL injection risk).

## Practice
Find active adults; users without phone; users whose phone is a string; users who are neither admin nor moderator.

## Interview checks
**$and vs $or?** All conditions versus any. **$exists vs $type?** Presence versus BSON type.

[Logical operators](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/logical/) · [Element operators](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/data-type/)
