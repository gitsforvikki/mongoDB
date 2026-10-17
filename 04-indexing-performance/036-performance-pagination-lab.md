# Lesson 036 — Performance and Pagination Lab

[← Section 4](./README.md) · [Full curriculum](../README.md)

## Goal
Measure a real query, add a suitable index, and compare offset with cursor pagination. Run these commands only on a **disposable local/practice database**.

## 1. Generate test data
```js
use mongoCourse
db.performanceLab.drop()
const batch=[]
for(let i=0;i<20000;i++){
 batch.push({
  _id:i+1,
  userId:"user-"+(i%100),
  status:["APPLIED","SCREENING","INTERVIEW"][i%3],
  appliedAt:new Date(Date.UTC(2026,0,1)+(i*60000))
 })
}
db.performanceLab.insertMany(batch)
```
20,000 documents is small enough for local practice but large enough to compare patterns. Don't reuse this drop statement on a real collection.

## 2. Measure without a custom index
```js
db.performanceLab.find({userId:"user-42"})
 .sort({appliedAt:-1,_id:-1}).limit(10)
 .explain("executionStats")
```
Record `nReturned`, `totalDocsExamined`, `totalKeysExamined`, and whether a blocking sort appears. Exact numbers depend on the planner.

## 3. Add compound index
```js
db.performanceLab.createIndex({userId:1,appliedAt:-1,_id:-1})
db.performanceLab.find({userId:"user-42"})
 .sort({appliedAt:-1,_id:-1}).limit(10)
 .explain("executionStats")
```
Compare work performed. A matching compound index can support both filtering and sorted retrieval.

## 4. Offset pagination
```js
db.performanceLab.find({userId:"user-42"})
 .sort({appliedAt:-1,_id:-1}).skip(100).limit(10)
```
Offset is easy but requires traversing skipped results. With 20,000 documents and 100 users, each user has only about 200 records; this is a demonstration, not proof of production-scale cost.

## 5. Cursor/keyset pagination (descending)
Get page 1:
```js
const page1=db.performanceLab.find({userId:"user-42"})
 .sort({appliedAt:-1,_id:-1}).limit(10).toArray()
const last=page1[page1.length-1]
```
For descending date + ID, the next page contains records **less than** the last key:
```js
const page2=db.performanceLab.find({
 userId:"user-42",
 $or:[
  {appliedAt:{$lt:last.appliedAt}},
  {appliedAt:last.appliedAt,_id:{$lt:last._id}}
 ]
}).sort({appliedAt:-1,_id:-1}).limit(10).toArray()
```
Using both date and unique `_id` ensures a deterministic continuation when dates tie.

## 6. Explain pagination
```js
db.performanceLab.find({userId:"user-42"})
 .sort({appliedAt:-1,_id:-1}).skip(100).limit(10)
 .explain("executionStats")
```
Compare to an equivalent cursor query. Keyset can be more efficient for deep pages with a matching index, but changing sort keys or concurrent writes still affect consistency.

## 7. Mongoose equivalent
```js
const applications=await Application.find({userId})
 .sort({appliedAt:-1,_id:-1})
 .limit(10)
 .lean();
```
For the next page, apply the cursor predicate shown above using your schema's actual ObjectId types. Validate user ownership and cursor input.

## 8. Lab report template
| Query | Index | nReturned | Docs examined | Keys examined | Observation |
| --- | --- | --- | --- | --- | --- |
| Recent applications | No custom index | Record | Record | Record | Before |
| Recent applications | Compound index | Record | Record | Record | After |
| Offset page | Compound index | Record | Record | Record | Deep skip |
| Cursor page | Compound index | Record | Record | Record | Keyset |

## Review
- **Indexes optimize particular queries**, not entire collections automatically.
- Use `explain("executionStats")` to verify.
- Indexes trade read speed for write/storage cost.
- Cursor pagination is useful for deep scrolling; offset is simpler for small admin tables.

**Interview:** Explain how you'd diagnose a slow endpoint: capture query shape → run explain → design index → measure again → monitor production.

[Explain](https://www.mongodb.com/docs/manual/reference/explain-results/) · [Indexes](https://www.mongodb.com/docs/manual/indexes/)
