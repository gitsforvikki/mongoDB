# Lesson 027 — MongoDB Schema Validation

[← Section 3](./README.md) · [Full curriculum](../README.md)

## Learning goal
MongoDB allows flexible documents by default. Database-level schema validation ensures important data rules apply even if the application bypasses Mongoose.

## 1. Why validate at the database layer?
Imagine a product API accidentally sends `price:"free"`. Without constraints, MongoDB may store it. You can require field types and valid ranges using `$jsonSchema` validators.

**Note:** BSON types matter: `int`, `long`, `double`, `decimal` are distinct. For this example we use positive integer amounts in paise.

## 2. Create a validated collection
Run in `mongosh` on a **practice database**:

```js
use mongoCourse
db.createCollection("validatedProductsDemo",{
 validator:{
  $jsonSchema:{
   bsonType:"object",
   required:["name","priceMinor","stock"],
   properties:{
    name:{bsonType:"string",minLength:1,description:"Name is required"},
    priceMinor:{bsonType:"int",minimum:0,description:"Nonnegative integer paise"},
    stock:{bsonType:"int",minimum:0},
    active:{bsonType:"bool"}
   }
  }
 },
 validationLevel:"strict",
 validationAction:"error"
})
```

This creates a collection once. If it already exists, use a fresh name or `collMod` rather than rerunning creation.

## 3. Try valid and invalid writes
```js
db.validatedProductsDemo.insertOne({
 name:"Mouse",priceMinor:NumberInt(69900),stock:NumberInt(12),active:true
}) // accepted

db.validatedProductsDemo.insertOne({
 name:"Keyboard",priceMinor:"700",stock:NumberInt(5)
}) // rejected: priceMinor must be BSON int

db.validatedProductsDemo.insertOne({
 name:"",priceMinor:NumberInt(500),stock:NumberInt(1)
}) // rejected: minLength is 1
```
The exact error format varies by server/version, but invalid documents are rejected with validation error information.

## 4. Change validation with collMod
```js
db.runCommand({
 collMod:"validatedProductsDemo",
 validator:{$jsonSchema:{
  bsonType:"object",
  required:["name","priceMinor","stock"],
  properties:{
   name:{bsonType:"string",minLength:1},
   priceMinor:{bsonType:"int",minimum:0},
   stock:{bsonType:"int",minimum:0},
   active:{bsonType:"bool"},
   category:{enum:["Electronics","Accessories"]}
  }
 }},
 validationLevel:"strict",
 validationAction:"error"
})
```
A new validator applies to subsequent validated writes; changing it does not automatically rewrite all existing documents.

## 5. validationLevel and validationAction
- **strict**: validate inserts and updates.
- **moderate**: validate inserts and updates to existing **valid** documents; updates to already-invalid documents are not checked in the same way. Useful during staged adoption, but not a long-term excuse for dirty data.
- **error**: reject invalid writes.
- **warn**: permit invalid writes but log validation violations. Use it to observe before enforcing.
Some MongoDB versions support additional action options; check your deployed server documentation.

## 6. Database validation vs Mongoose schema
| | MongoDB validator | Mongoose schema |
| --- | --- | --- |
| Runs where? | Database server | Node.js application |
| Applies to mongosh/direct writes? | Yes | No |
| Can shape/cast application input? | Not like Mongoose casting | Yes |
| Replaces unique indexes? | No | No |
| Supports application-specific validators/hooks? | Limited server expressions | Richer model features |

Mongoose example:
```js
const productSchema=new mongoose.Schema({
 name:{type:String,required:true,minlength:1},
 priceMinor:{type:Number,required:true,min:0},
 stock:{type:Number,required:true,min:0}
});
```
A Mongoose `Number` isn't automatically a BSON `int` in every context. When combining the example validator with Mongoose, verify that the BSON numeric representation emitted by your actual writes satisfies the database validator. Prefer choosing one deliberate numeric storage representation for your application.

## 7. Important limitations
- A JSON Schema validator is **not** a foreign-key constraint.
- It doesn't protect against all application-level business rules or authorization mistakes.
- Indexes handle uniqueness; validator handles document shape/types.
- Some privileged operations can intentionally bypass document validation; protect those permissions.
- Plan deployment of new validators carefully for existing data.

## Practice
Create a validated `applications` collection requiring a string company and enum status. Try three invalid inserts, then set validationAction to warn on a disposable collection and compare behavior.

**Interview:** **Why validate in MongoDB if Mongoose validates?** Direct clients and migration scripts can bypass Mongoose.

[Official schema validation](https://www.mongodb.com/docs/manual/core/schema-validation/) · [$jsonSchema operator](https://www.mongodb.com/docs/manual/reference/operator/query/jsonSchema/)
