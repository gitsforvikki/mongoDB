# Lesson 65 — Optimistic Concurrency and Version Keys

[← Section 7](./README.md) · [Full curriculum](../README.md)

## 1. The lost-update problem

Two recruiters open the same job application. Both see status `APPLIED`. Recruiter A changes it to `INTERVIEW`; recruiter B, working from an older copy, changes it to `REJECTED`. If both save without conflict detection, the later write may silently overwrite the earlier business decision.

**Optimistic concurrency control (OCC)** assumes conflicts are relatively rare. Instead of locking a document for the entire editing period, it detects whether the document changed between **read** and **write**.

## 2. Version key: what __v means

By default, Mongoose uses a version key named `__v`. New documents typically begin with `__v:0`. **Crucial distinction:** the existence of `__v` alone does **not** mean every scalar field update is protected. Mongoose's default versioning behavior has historically focused on certain array changes. For broad document-save conflict detection, enable `optimisticConcurrency:true`.

## 3. Runnable example

```js
import mongoose from "mongoose";
await mongoose.connect(process.env.MONGODB_URI, { dbName: "mongoose_course" });

const applicationSchema = new mongoose.Schema({
  company: { type: String, required: true },
  status: {
    type: String, required: true,
    enum: ["APPLIED", "INTERVIEW", "OFFER", "REJECTED"]
  }
}, { optimisticConcurrency: true });

const Application = mongoose.model("CourseOCCApplication", applicationSchema);

const created = await Application.create({
  company: "Acme", status: "APPLIED"
});

const recruiterA = await Application.findById(created._id);
const recruiterB = await Application.findById(created._id);

recruiterA.status = "INTERVIEW";
await recruiterA.save(); // succeeds, version advances

recruiterB.status = "REJECTED";
try {
  await recruiterB.save();
} catch (error) {
  console.log(error.name); // "VersionError"
}
```

The second save fails rather than silently overwriting the first. After a conflict, re-read the document and decide whether to retry, merge, or ask the user to resolve it.

## 4. Internal working

Conceptually, Mongoose uses the previously read version as part of the write condition and increments the version on a successful document save.

```js
// Conceptual illustration, not the exact generated driver command:
db.applications.updateOne(
  { _id: id, __v: 0 },
  { $set: { status: "INTERVIEW" }, $inc: { __v: 1 } }
);
```

When the second editor attempts a write using `__v:0`, the stored document is now version 1. The condition matches no document, so Mongoose reports a version conflict.

```mermaid
sequenceDiagram
 participant A as Editor A
 participant B as Editor B
 participant DB as MongoDB
 A->>DB: Read status APPLIED, version 0
 B->>DB: Read status APPLIED, version 0
 A->>DB: Save INTERVIEW where version 0
 DB-->>A: Success; version 1
 B->>DB: Save REJECTED where version 0
 DB-->>B: No match; VersionError
```

## 5. What optimisticConcurrency does NOT automatically cover

```js
await Application.updateOne(
  { _id: created._id },
  { $set: { status: "OFFER" } }
);
```

Query updates like `updateOne()` and `findOneAndUpdate()` do not automatically use the same document-save optimistic concurrency mechanism. If an API uses query updates, implement explicit version checks when needed:

```js
const expectedVersion = 1; // read from a trusted, validated request precondition
const result = await Application.updateOne(
  { _id: created._id, __v: expectedVersion },
  {
    $set: { status: "OFFER" },
    $inc: { __v: 1 }
  },
  { runValidators: true }
);
if (result.matchedCount === 0) {
  throw new Error("Not found or version conflict");
}
```

This is an **atomic compare-and-swap** style update. A real API should distinguish not-found from conflict, validate version inputs, and coordinate all writers so they do not bypass the version policy.

## 6. OCC vs atomic updates vs transactions

| Technique | Solves | Does not automatically solve |
| --- | --- | --- |
| `$inc` with conditional filter | Atomic single-document counter update | Multi-document consistency |
| Optimistic document `save()` | Detect stale document saves | Query updates that bypass version checks |
| Transaction | Commit several DB writes as a unit | External API side effects or all business races |

For inventory reservation, an atomic predicate such as `{stock:{$gte:quantity}}` plus `$inc` is often simpler than read-modify-save. For multi-document money transfers, a transaction may be required.

## 7. HTTP API design

When a client edits a resource, it can send the version it originally read. The server can reject stale changes with **409 Conflict** or **412 Precondition Failed** when using HTTP conditional request semantics. Do not silently retry a conflicting status change without checking business intent; a retry could overwrite a legitimate concurrent decision.

## 8. Common mistakes

- Assuming `__v` protects every update by default.
- Assuming `optimisticConcurrency:true` protects raw MongoDB driver writes.
- Retrying stale edits indefinitely.
- Disabling `versionKey` without understanding consequences.
- Expecting OCC to replace transaction handling.
- Returning a generic 500 for `VersionError` rather than a useful conflict response.

## 9. Exercises

1. Open the same document twice and reproduce a `VersionError`.
2. Repeat with a query update and explain why behavior differs.
3. Build an Express PATCH endpoint that accepts a version precondition.
4. Compare a guarded stock decrement with OCC for inventory.
5. Explain how a client should respond to a stale edit.

**Interview:** **What is OCC?** Detecting conflicts at write time. **What is __v?** Mongoose's version key. **Does it cover query updates automatically?** No.

**Official:** [Schema optimisticConcurrency](https://mongoosejs.com/docs/guide.html#optimisticConcurrency) · [Version key](https://mongoosejs.com/docs/guide.html#versionKey)
