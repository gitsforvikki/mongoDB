# Lesson 034 — Explain Plans and Query Execution

[← Section 4](./README.md) · [Full curriculum](../README.md)

## Goal
Stop guessing whether an index helps. Measure actual query plans and work performed.

## 1. Seed enough data to observe differences
Use a disposable collection:
```js
use mongoCourse
db.explainDemo.drop()
const docs=Array.from({length:10000},(_,i)=>({
  email:`user${i}@example.test`,
  age:18+(i%50),
  status:i%2===0?"active":"inactive"
}))
db.explainDemo.insertMany(docs)
```
This deliberately drops **only** the disposable `explainDemo` collection. Never copy a drop into production scripts.

## 2. Without an index
```js
db.explainDemo.find({email:"user9999@example.test"}).explain("executionStats")
```
Inspect `queryPlanner.winningPlan` (structure varies by MongoDB version) and `executionStats`. You may see COLLSCAN and thousands of documents examined.

## 3. Add index and measure again
```js
db.explainDemo.createIndex({email:1})
db.explainDemo.find({email:"user9999@example.test"}).explain("executionStats")
```
Often IXSCAN + FETCH, with far fewer examined keys/documents.

## 4. Metrics explained
| Metric | Meaning |
| --- | --- |
| `nReturned` | Documents returned |
| `totalDocsExamined` | Documents inspected during execution |
| `totalKeysExamined` | Index entries examined |
| `executionTimeMillis` | Reported execution time; noisy for small tests |
| `winningPlan` | Selected plan; representation varies by server engine/version |

`totalDocsExamined` can be **0** for a covered query. Large `docsExamined / nReturned` often signals extra work, but zero returned documents need separate interpretation.

## 5. Hint for controlled comparison
```js
db.explainDemo.find({email:"user9999@example.test"})
 .hint({email:1}).explain("executionStats")
```
`hint()` forces an index for experimentation. Avoid forcing a bad index in production; MongoDB's planner usually chooses.

## 6. Covered query concept
```js
db.explainDemo.find(
 {email:"user9999@example.test"},
 {_id:0,email:1}
).explain("executionStats")
```
If the index contains all fields needed by the filter and projection, the database may answer without fetching full documents.

## 7. Mongoose equivalent
```js
const plan = await User.find({email:"user9999@example.test"})
  .explain("executionStats");
console.log(plan.executionStats?.totalDocsExamined);
```
Explain returns plan information rather than ordinary model documents.

## Mistakes
- Relying only on milliseconds from one run.
- Assuming IXSCAN is always the fastest plan.
- Forgetting sort and projection when measuring the real endpoint.
- Ignoring that explain plans may differ across server versions and dataset sizes.

## Practice
Measure before and after indexing; compare `nReturned`, `totalDocsExamined`, and `totalKeysExamined`; test a covered projection.

**Interview:** **How do you detect a collection scan?** Inspect explain's winning plan and execution statistics.

[Explain results](https://www.mongodb.com/docs/manual/reference/explain-results/)
