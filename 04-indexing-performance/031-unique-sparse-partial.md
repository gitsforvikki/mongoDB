# Lesson 031 — Unique, Sparse and Partial Indexes

[← Section 4](./README.md) · [Full curriculum](../README.md)

## Goal
Enforce uniqueness and index only relevant subsets.

## 1. Unique indexes
```js
use mongoCourse
db.uniqueUsers.createIndex({email:1},{unique:true})
db.uniqueUsers.insertOne({_id:1,email:"a@example.test"})
db.uniqueUsers.insertOne({_id:2,email:"a@example.test"}) // E11000 duplicate key
```
A unique index enforces uniqueness at the **database**, including writes outside Mongoose. Creating it fails if existing data violates the rule.

**Important:** A normal single-field unique index treats missing/null values as a null index entry, so only one document with a missing or null indexed value can be present.

## 2. Sparse indexes
```js
db.sparseUsers.createIndex({phone:1},{unique:true,sparse:true})
db.sparseUsers.insertMany([{_id:1,name:"A"},{_id:2,name:"B"}])
db.sparseUsers.insertOne({_id:3,phone:"123"})
```
Sparse indexes omit documents that **lack** the indexed field. They still index explicit `null` values, so a unique sparse index can reject multiple explicit nulls.

## 3. Partial indexes
```js
db.partialUsers.createIndex(
 {email:1},
 {unique:true,partialFilterExpression:{email:{$type:"string"}}}
)
db.partialUsers.insertMany([{_id:1},{_id:2},{_id:3,email:"a@example.test"}])
```
The partial index includes only documents with a string email, allowing multiple missing or null emails. Two documents with the same string email still conflict.

Another use: index only active products:
```js
db.partialProducts.createIndex(
 {category:1,price:1},
 {partialFilterExpression:{active:true}}
)
```
Queries generally must include or imply `active:true` for MongoDB to safely use that partial index.

## 4. Mongoose equivalent
```js
userSchema.index(
 {email:1},
 {unique:true,partialFilterExpression:{email:{$type:"string"}}}
);
```
`unique:true` is **not** a Mongoose validator; duplicate keys produce a database error (commonly code 11000).

## Practice
Allow optional email but require uniqueness when present. Compare a plain unique index, unique sparse index, and unique partial index.

**Interview:** **Sparse vs partial?** Sparse skips missing fields; partial uses an explicit filter for which documents are indexed.

[Unique](https://www.mongodb.com/docs/manual/core/index-unique/) · [Partial](https://www.mongodb.com/docs/manual/core/index-partial/)
