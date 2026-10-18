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

## 5. Detailed walkthrough — Unique, Sparse and Partial Indexes

### 5.1 Unique: stop duplicate email addresses

Imagine an e-commerce application with users Rahul, Amit and Priya. Each user should have a different email:

```js
db.users.createIndex({ email: 1 }, { unique: true });

db.users.insertOne({ name: "Rahul", email: "rahul@gmail.com" });
db.users.insertOne({ name: "Vikas", email: "rahul@gmail.com" });
// E11000 duplicate key error
```

**Why useful?** Even if two registration requests arrive simultaneously, the database enforces uniqueness. Checking with `findOne()` before inserting is not enough to prevent races.

**Mongoose:**

```js
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true }
});
```

`unique: true` asks Mongoose to define an index; it is **not a validator**. The index must actually exist in MongoDB. Handle duplicate-key error code `11000` in your API.

### 5.2 Sparse: index only documents with the field

Suppose phone numbers are optional:

| User | phone field |
| --- | --- |
| Rahul | "9876543210" |
| Amit | Missing |
| Priya | "9123456780" |
| Vikas | Missing |

```js
db.users.createIndex({ phone: 1 }, { sparse: true });
```

**Conceptual diagram:**

```text
Users collection             Sparse phone index
Rahul  phone: 9876543210  -->  9876543210 -> Rahul
Amit   (no phone)         -X-  not indexed
Priya  phone: 9123456780  -->  9123456780 -> Priya
Vikas  (no phone)         -X-  not indexed
```

Combine `sparse` and `unique` if present phone values must not repeat:

```js
db.users.createIndex(
  { phone: 1 },
  { unique: true, sparse: true }
);
```

**Important difference:** A **missing field** is excluded from a single-field sparse index, but an explicitly present `phone: null` **is included**. So a unique sparse index may reject a second explicit `null`.

### 5.3 Partial: choose indexed documents with a filter

Imagine a product catalog where most searches target active products:

| Product | Price | Status |
| --- | ---: | --- |
| Mouse | 500 | active |
| Keyboard | 1200 | inactive |
| Laptop | 50000 | active |
| Monitor | 15000 | inactive |

```js
db.products.createIndex(
  { price: 1 },
  { partialFilterExpression: { status: "active" } }
);
```

Only **active** products get entries in this index; inactive products remain in the collection.

```text
Products collection               Partial price index
Mouse     500   active        -->  500 -> Mouse
Keyboard 1200   inactive      -X-  excluded
Laptop  50000   active        -->  50000 -> Laptop
Monitor 15000   inactive      -X-  excluded
```

This query can benefit from the index:

```js
db.products.find({
  status: "active",
  price: { $gte: 1000 }
});
```

But a query filtering **only** by price generally cannot use that partial index to get complete results, because inactive products are absent from it.

### 5.4 Partial + unique: enforce a rule only for selected documents

Example: usernames must be unique **among active users**:

```js
db.users.createIndex(
  { username: 1 },
  {
    unique: true,
    partialFilterExpression: { isActive: true }
  }
);
```

Duplicate usernames among indexed active users are rejected. Inactive users are outside this particular unique constraint.

For optional email, a partial unique index can include only string values:

```js
db.users.createIndex(
  { email: 1 },
  {
    unique: true,
    partialFilterExpression: { email: { $type: "string" } }
  }
);
```

Now multiple documents may omit email or have `email: null`, while string email values must be unique.

### 5.5 All three in a Mongoose schema

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  phone: String,
  username: String,
  isActive: Boolean
});

// Unique email values, but only when email is a string
userSchema.index(
  { email: 1 },
  { unique: true, partialFilterExpression: { email: { $type: "string" } } }
);

// Optional phone; unique when the field is present
userSchema.index({ phone: 1 }, { unique: true, sparse: true });

// Unique usernames for active users only
userSchema.index(
  { username: 1 },
  { unique: true, partialFilterExpression: { isActive: true } }
);

export const User = mongoose.model("User", userSchema);
```

**Production tip:** Create and manage indexes through a controlled deployment/migration process. Index creation can fail if existing data already violates a uniqueness rule.

### 5.6 Comparison

| Feature | Unique | Sparse | Partial |
| --- | --- | --- | --- |
| Main purpose | Reject duplicate indexed values | Exclude documents missing indexed field | Include documents matching a filter |
| Custom condition | No | Field existence | Yes |
| Uniqueness enforced? | Yes | Only with `unique: true` | Only with `unique: true` |
| Example | Email | Optional phone | Active products |

**Interview answer:** Unique controls whether indexed keys may repeat; sparse excludes missing fields; partial selects indexed documents using an explicit condition.

**Practice:** Try inserting two users without `phone`, then two with `phone: null` under a unique sparse index. Explain why their outcomes differ. Then try the partial unique email index.

## Practice
Allow optional email but require uniqueness when present. Compare a plain unique index, unique sparse index, and unique partial index.

**Interview:** **Sparse vs partial?** Sparse skips missing fields; partial uses an explicit filter for which documents are indexed.

[Unique](https://www.mongodb.com/docs/manual/core/index-unique/) · [Partial](https://www.mongodb.com/docs/manual/core/index-partial/)
