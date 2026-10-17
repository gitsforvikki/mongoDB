# Lesson 048 — Schemas and Models

[← Section 6](./README.md) · [Full curriculum](../README.md)

## Deep theory: schema, model, document and the full save lifecycle

These terms sound similar because they all refer to the same kind of data, but they exist at **different levels**:

- A **schema** is a configuration object in Node.js describing paths and behaviors.
- A **model** is a constructor compiled from that schema, associated with a MongoDB collection.
- A **document** is one Mongoose instance with concrete values and internal change tracking.
- A **MongoDB document** is the persisted BSON record stored on the server. It is not literally the same JavaScript object as the Mongoose document instance.

### Why a schema if MongoDB is flexible?

MongoDB can accept `{name:"Asha"}` and `{name:17}` in the same collection unless database validation restricts it. Your app may not want that. The schema centralizes rules for **writes performed through that model**. A different application using the raw driver can still write incompatible data unless MongoDB itself has collection validation.

### What happens when each line executes?

```js
const schema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  age: { type: Number, min: 18 },
  status: { type: String, default: "ACTIVE" }
});
const User = mongoose.model("CourseUserLifecycle", schema);
const user = new User({ name: " Asha ", age: "24" });
await user.save();
```

**Line 1 — `new Schema()`:** registers field paths, their casting rules, defaults, and validators. It does **not** contact MongoDB or create a collection.

**Line 2 — `mongoose.model()`:** compiles a Model tied to a connection and collection. The model exposes static operations such as `find()`, `create()`, and `updateOne()`. Compiling a model does not insert a document.

**Line 3 — `new User()`:** constructs an in-memory document. Mongoose applies appropriate setters, casts input where possible, and assigns defaults to undefined paths. It usually also generates an `_id` before saving. `user.isNew` is `true`.

**Line 4 — `save()`:** runs document validation and configured middleware, then sends an insert for a new document or an update for a previously saved, modified document. If the write succeeds, the promise resolves. If validation or persistence fails, it rejects.

```mermaid
flowchart TD
  A["Schema declaration"] --> B["Compile Model"]
  B --> C["Construct document in memory"]
  C --> D["Cast + defaults + setters"]
  D --> E["user.save()"]
  E --> F["Pre-validation middleware"]
  F --> G["Validation"]
  G --> H["Post-validation / pre-save hooks"]
  H --> I["MongoDB insert or update"]
  I --> J["Post-save hooks / result"]
```

This is a **conceptual ordering** for ordinary document saves. The precise middleware order and behavior depend on which hooks are registered; Mongoose's built-in validation runs as the first `pre('save')` hook. In particular, modifications made by later pre-save hooks are not automatically revalidated in the same way. Learn middleware ordering in its dedicated lesson before depending on it.

### `new + save` vs `create` vs `insertMany`

```js
const one = new User({ name: "Asha", age: 24 });
await one.save();

const two = await User.create({ name: "Ravi", age: 26 });

await User.insertMany([
  { name: "Neha", age: 23 },
  { name: "Kabir", age: 30 }
]);
```

`new + save` lets you modify a document before persistence. `create()` is a convenient create-and-save API that triggers save middleware. `insertMany()` is designed for batch inserts and **does not run save middleware**; it has its own middleware and validation behavior. Do not replace one with another without checking hooks and error semantics.

### Model queries vs document methods

```js
const found = await User.findOne({ name: "Asha" }); // Model query
if (found) {
  found.status = "INACTIVE";
  await found.save(); // Document method; tracks changed paths
}

await User.updateOne(
  { name: "Ravi" },
  { $set: { status: "INACTIVE" } }
); // Query update; does not use save middleware
```

**Why does this distinction matter?** Document validation and save hooks are tied to document saves. Query updates can be more efficient for targeted modifications, but have different middleware and validation rules. A query such as `find()` builds a query object that executes when awaited or explicitly executed with `.exec()`.

### Common misconception: `ref` is not a foreign key

```js
const orderSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
});
```

This tells Mongoose which model to use when you explicitly call `populate()`. MongoDB does not automatically reject a nonexistent referenced user, and deleting a user does not automatically delete their orders. Handle referential integrity in application workflows.

### A debugging checklist

If a field is missing from saved data, check **strict mode**, projections, setters, and whether you ever called `save()`. If a model is compiled twice during hot reload, inspect `mongoose.models`. If a validator passes but an insert fails, check unique indexes, server-side validation, permissions and connectivity. If a modified nested `Mixed` value does not persist, examine Mongoose change tracking.

**Checkpoint:** Predict `isNew` before and after `save()`; explain why `await user.validate()` does not create a MongoDB document; explain why `updateOne()` does not trigger document save middleware.

---
## Learning goals
Understand **Schema vs Model vs Document**, compilation, collection naming, document creation, model reuse and schema design options.

## 1. Core mental model
```mermaid
flowchart LR
 S["new Schema(): rules and shape"] --> M["mongoose.model(): compiled Model"]
 M --> D["new User(): document instance"]
 D --> V["validate / save"]
 V --> C["MongoDB users collection"]
```
A **Schema** defines structure and behavior. A **Model** is the class-like interface created from a schema, used for CRUD and queries. A **Document** is one instance with actual field values.

## 2. A real schema
```js
import mongoose from "mongoose";
const {Schema} = mongoose;

const userSchema = new Schema({
  name:{type:String,required:true,trim:true},
  email:{type:String,required:true,lowercase:true},
  age:{type:Number,min:0},
  isActive:{type:Boolean,default:true},
  address:{
    city:{type:String,default:"Unknown"},
    pincode:String
  }
},{timestamps:true});

const User = mongoose.model("User",userSchema);
export default User;
```
`timestamps:true` manages `createdAt` and `updatedAt` for supported Mongoose operations. `mongoose.model("User", ...)` typically uses the pluralized `users` collection. Collection names can be specified explicitly if required.

## 3. Creating a document: new + save
```js
const user = new User({name:" Asha ",email:"ASHA@EXAMPLE.TEST"});
console.log(user.isNew); // true
console.log(user.name); // "Asha" (trim setter)
console.log(user.isActive); // true
await user.save();
console.log(user.isNew); // false
console.log(user._id); // ObjectId assigned by Mongoose
```
**Important:** `new User(...)` creates an in-memory Mongoose document; **it does not insert into MongoDB** until you save. `save()` validates the document before writing, then uses insert/update operations depending on whether the document is new.

## 4. Model.create shortcut
```js
const ravi = await User.create({
 name:"Ravi",email:"ravi@example.test",age:25
});
```
`Model.create()` creates and saves document(s), invoking save-related validation and middleware.

## 5. Query with the model
```js
const all = await User.find({isActive:true}).sort({createdAt:-1});
const one = await User.findOne({email:"asha@example.test"});
const byId = await User.findById(ravi._id);
```
`find()` returns an array; `findOne()` and `findById()` return a document or `null`. Query methods build **Query objects** that execute when awaited or when `.exec()` is called.

## 6. Schema does not equal collection
A schema lives in Node.js code. The collection lives in MongoDB. Schema fields are not automatically enforced on direct `mongosh` inserts unless the database also has its own validator. MongoDB remains schema-flexible.

## 7. Compile models once
```js
// Standard server-side model module:
export const User = mongoose.models.User || mongoose.model("User",userSchema);
```
This guard is useful in hot-reload environments where the same model module may be evaluated repeatedly. In a stable Node.js process, normal module caching already avoids repeated compilation. Calling `mongoose.model("User", differentSchema)` again can throw `OverwriteModelError`.

## 8. Explicit collection name
```js
const auditSchema = new Schema({action:String},{collection:"audit_events"});
const AuditEvent = mongoose.model("AuditEvent",auditSchema);
```
Now the collection is `audit_events`, not the default pluralized name.

## 9. Document vs plain object
```js
const doc = await User.findOne();
if (doc) {
  console.log(typeof doc.save); // "function"
  const plain = doc.toObject();
  console.log(typeof plain.save); // "undefined"
}
```
Hydrated documents have Mongoose methods and change tracking. `.lean()` queries skip hydration for read-only results (Lesson 055).

## 10. Relationships and indexes
```js
const orderSchema = new Schema({
 userId:{type:Schema.Types.ObjectId,ref:"User",required:true},
 totalMinor:{type:Number,required:true,min:0}
});
orderSchema.index({userId:1,createdAt:-1});
```
The `ref` enables Mongoose population; it is **not** a database foreign key. An index is created in MongoDB when the index-management workflow runs; the schema declaration alone is not a uniqueness guarantee.

## 11. Common errors
- Thinking `new User()` immediately saves data.
- Confusing `User.find()` (model query) with `user.save()` (document method).
- Compiling the same model name with different schemas.
- Expecting `required:true` to protect direct driver writes.
- Defining a `ref` and assuming automatic cascade deletes.

## Practice
Create `Product` schema with name, price, tags, timestamps. Insert using `new Product().save()` and `Product.create()`, then query and compare `toObject()`.

## Interview check
**Q: Schema vs Model?** Blueprint vs compiled database-facing constructor/query API.

**Q: Model vs Document?** Model represents a collection interface; document represents one record.

**Q: Does creating a schema create a collection immediately?** Not necessarily; collections are typically created on first write or through explicit collection creation/index operations.

**Official references:** [Schemas](https://mongoosejs.com/docs/guide.html) · [Models](https://mongoosejs.com/docs/models.html) · [Documents](https://mongoosejs.com/docs/documents.html)
