# Lesson 018 — Upsert and Atomic Single-Document Updates

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why upsert?
An **upsert** means **update if found, insert if missing**. Useful for user preferences, counters, and syncing external data.

## 1. Basic upsert
```js
use mongoCourse
db.settingsDemo.updateOne(
 {userId:"u101"},
 {$set:{theme:"dark"},$setOnInsert:{createdAt:new Date()}},
 {upsert:true}
)
```
First run inserts a document. Later runs update `theme` and preserve the original `createdAt`. With an acknowledged result, an insert reports `upsertedId`; an existing match reports `matchedCount`.

## 2. Why a unique index matters
Two requests can both see "no matching document" and attempt inserts. Without a suitable unique constraint, duplicate logical records may result.

```js
db.settingsDemo.createIndex({userId:1},{unique:true})
```
Now the database enforces uniqueness of `userId`. Under races, one request may still receive a duplicate-key error; handle it appropriately rather than assuming every upsert succeeds.

## 3. Atomicity of one document
Suppose stock is 5 and two customers buy an item. Avoid reading stock into application memory, subtracting, and saving without a concurrency guard.

Safer conditional update:
```js
db.inventoryDemo.insertOne({_id:"sku-1",stock:5})
db.inventoryDemo.updateOne(
 {_id:"sku-1",stock:{$gte:2}},
 {$inc:{stock:-2}}
)
```
MongoDB checks the filter and changes the document atomically. If stock is below 2, `matchedCount:0`; no stock is deducted. Two concurrent requests cannot both successfully deduct beyond the available quantity using this conditional single-document update.

## 4. Return the updated document
```js
db.inventoryDemo.findOneAndUpdate(
 {_id:"sku-1",stock:{$gte:1}},
 {$inc:{stock:-1}},
 {returnDocument:"after"}
)
```
Returns the updated document, or `null` if no match. This can be useful when the API needs the new stock immediately.

## 5. Multi-document transactions are different
Updating inventory and creating an order are **two document writes**. A single-document atomic update does not make both operations atomic together. Transactions may be appropriate when the workflow requires all-or-nothing behavior.

## Mongoose equivalent
```js
const updated = await Inventory.findOneAndUpdate(
 {_id:skuId,stock:{$gte:quantity}},
 {$inc:{stock:-quantity}},
 {new:true}
);
if (!updated) throw new Error("Insufficient stock or missing item");
```
In Mongoose, `new:true` returns the document after the update. MongoDB's underlying atomicity rules still apply. Mongoose `findOneAndUpdate` has different validation/middleware behavior from `save()`.

## Common mistakes
- Assuming upsert means "always insert".
- Assuming upsert alone prevents duplicate logical records.
- Checking stock in a separate read without an atomic conditional update.
- Assuming one document's atomicity automatically covers an order and payment.
- Treating `matchedCount:0` as a successful inventory deduction.

## Practice
Upsert the same user settings twice; create a unique index; simulate stock decrements until the conditional update fails.

## Interview check
**Upsert?** Update or insert. **Atomic single-document update?** Filter and modification happen as one indivisible write for that document.

[Upsert](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateOne/) · [Atomicity](https://www.mongodb.com/docs/manual/core/write-operations-atomicity/)
