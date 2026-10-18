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

Mongoose mein **Model**, **Query**, aur **Document** related hain, lekin same cheez nahi hain:

- **Model (`User`)**: schema se compiled class-like interface. Collection ke documents find, create, update, delete karne ke methods provide karta hai.
- **Model query (`User.findOne(...)`)**: Model se query build hoti hai jo MongoDB mein matching records search karti hai. `find()`, `findOne()`, `findById()`, `updateOne()` etc. Mongoose **Query** objects return karte hain.
- **Document (`user`)**: `findOne()` se mila hydrated record ya `new User(...)` se banaya instance. Is par `save()`, `validate()` aur custom instance methods call kar sakte ho.

#### 1. Model queries: collection mein records find/update/delete karo

Assume a `User` model with fields `name`, `age`, `isActive`, `status`.

```js
// Read documents
const users = await User.find({ isActive: true }); // array
const user = await User.findOne({ name: "Asha" }); // document or null
const byId = await User.findById(userId);         // document or null

// Update without loading the document first
await User.updateOne(
  { name: "Asha" },
  { $set: { status: "INACTIVE" } }
);

// Delete a matching record
await User.deleteOne({ name: "Ravi" });
```

**Why useful?** Jab database mein matching records search karne hain, multiple records read karne hain, ya direct targeted update chahiye. `User.find()` query object build karta hai; `await` ya `.exec()` par execute hota hai.

#### 2. Document methods: ek particular record par kaam karo

```js
const user = await User.findOne({ name: "Asha" }); // Model query

if (user) {
  user.age = 26;      // change in the Mongoose document
  await user.save();  // Document method: persist the change
}
```

Yahan `User.findOne()` **Model query** hai; `user.save()` **Document method** hai. `save()` document validation, change tracking aur save middleware ke saath kaam karta hai.

#### 3. Custom document (instance) methods

Business logic ko ek document ke saath attach kar sakte hain:

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  isActive: { type: Boolean, default: true }
});

userSchema.methods.isAdult = function () {
  return this.age >= 18;
};

const User = mongoose.model("User", userSchema);
const user = await User.findOne({ name: "Asha" });

if (user) {
  console.log(user.isAdult()); // true when age >= 18
}
```

`isAdult()` ek **custom document method** hai. Normal `function` use kiya gaya hai kyunki `this` current Mongoose document ko refer karta hai. `isAdult()` apne aap database query nahi chalata; loaded document ki age check karta hai.

#### 4. Same profile update: model query vs document save

**Approach A — direct model query:**

```js
await User.findByIdAndUpdate(
  userId,
  { $set: { name: "Rahul" } },
  { runValidators: true }
);
```

Ismein pehle document fetch karna necessary nahi. Query update middleware apply ho sakta hai; **document `save` middleware nahi**. Update validators ko `runValidators:true` se enable karna padta hai, aur inki coverage document validation se differ kar sakti hai.

**Approach B — document method:**

```js
const user = await User.findById(userId);
if (!user) throw new Error("User not found");

user.name = "Rahul";
await user.save();
```

Ismein document load hota hai, field change hoti hai, aur `save()` document validation/save hooks run karta hai. Extra read ki cost hai, aur concurrent updates ke liye separate consistency considerations ho sakti hain.

| Feature | Model query | Document method |
| --- | --- | --- |
| Called on | `User` Model | `user` document |
| Example | `User.find()`, `User.updateOne()` | `user.save()`, `user.validate()` |
| Purpose | Search/modify matching DB records | One hydrated document par operations |
| Can target many? | Yes, e.g. `find()`, `updateMany()` | Usually one document |
| Middleware | Query middleware for query operations | Document/save middleware for `save()` |
| Validation | Update validators have separate rules | `save()` validates document |

#### 5. Bonus: Query helpers are NOT document methods

```js
userSchema.query.onlyActive = function () {
  return this.where({ isActive: true });
};

// After model compilation:
const activeUsers = await User.find().onlyActive();
```

`onlyActive()` ek **query helper** hai: ye query ko modify karta hai, individual user document ko nahi. Iske opposite `user.isAdult()` document instance method hai.

**Remember:**
- `User.find()` → database mein users **dhundo**.
- `user.save()` → ek particular user's changes **save karo**.
- `user.isAdult()` → loaded user ki business condition **check karo**.
- `User.find().onlyActive()` → query par reusable filter **lagao**.

**Quick check:** `const user = await User.findById(id);` ke baad `user.save()` **document method** hai, model query nahi. Null result ko pehle check karna zaroori hai.

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
