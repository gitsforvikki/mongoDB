## **What is MongoDB?**

**MongoDB** is a **NoSQL, document-oriented database** used to store data in a flexible JSON-like format called **BSON (Binary JSON)**.

Instead of storing data in **tables and rows** like traditional relational databases, MongoDB stores data in **collections and documents**.

Example document:
```javascript
{  
 "_id": "101",  
 "name": "Vikash Kumar",  
 "role": "Full Stack Developer",  
 "skills": \["React", "Next.js", "Node.js", "MongoDB"\],  
 "experience": 2  
}
```

* **Database** → contains collections

* **Collection** → group of documents (similar to a table)

* **Document** → JSON-like data (similar to a row)

# **Why MongoDB is Better (in Many Cases)**

MongoDB is not always better than relational databases, but it has advantages in **modern web applications, scalability, and flexibility**.

Let's look at the important reasons interviewers expect.

# **1\. Flexible Schema**

Traditional SQL databases require a **fixed schema**.

### **MongoDB Solution**

MongoDB allows **dynamic schema**.
```javascript

{ "name": "Vikash", "age": 25 }

{ "name": "Rahul", "age": 24, "phone": "9876543210" }

```
# **2\. JSON-like Data (Perfect for JavaScript)**

MongoDB uses **BSON**, which is similar to JSON.

Example:
```javascript
const user \= {  
 name: "Vikash",  
 skills: \["React", "Node.js"\]  
};

```
This can be stored directly in MongoDB.

# **3\. Better for Hierarchical Data**

MongoDB supports **nested documents**.

In SQL this would require **multiple tables and joins**.

MongoDB does not require an explicit command to create a database. We use the `use databaseName` command to switch to a database, and the database is actually created when we insert the first document into a collection.

# **CRUD Operation**

# **1️⃣ CREATE (Insert Data)**

Create operations are used to **add new documents** to a collection.

### **1\. `insertOne()`**

Inserts **one document** into a collection.
```javascript
db.users.insertOne({  
 name: "Vikash",  
 role: "Full Stack Developer",  
 experience: 2  
})
```
### **2\. `insertMany()`**

Inserts **multiple documents** at once.
```javascript
db.users.insertMany([  
 {  
   name: "Vikash",  
   role: "Developer",  
   experience: 2  
 },  
 {  
   name: "Rahul",  
   role: "Backend Developer",  
   experience: 3  
 }  
])
```
# **2️⃣ READ (Fetch Data)**

Read operations retrieve documents from the database.

### **1. `find()`**

Returns **multiple documents**.

db.users.find()

Example with filter:

db.users.find({ role: "Developer" })  
---

### **2. `findOne()`**

Returns **only one document**.

db.users.findOne({ name: "Vikash" })

Output example:
```javascript
{  
 "\_id": "...",  
 "name": "Vikash",  
 "role": "Developer"  
}
```
# **3️⃣ UPDATE (Modify Data)**

Update operations modify existing documents.

---

### **1. `updateOne()`**

Updates **one matching document**.
```javascript
db.users.updateOne(  
 { name: "Vikash" },  
 { $set: { experience: 3 } }  
)
```
Explanation:

* `{ name: "Vikash" }` → filter

* `$set` → update operator

### **2. `updateMany()`**

Updates **multiple documents**.
```javascript
db.users.updateMany(  
 { role: "Developer" },  
 { $set: { department: "Engineering" } }  
)  
```

### **3\. `replaceOne()`**

Replaces the entire document.
```javascript
db.users.replaceOne(  
 { name: "Vikash" },  
 {  
   name: "Vikash",  
   role: "Senior Developer",  
   experience: 4  
 }  
)
```
⚠️ Old fields will be removed.

# **4️⃣ DELETE (Remove Data)**

Delete operations remove documents.

---

### **1. `deleteOne()`**

Deletes **one matching document**.
```javascript
db.users.deleteOne({ name: "Rahul" })  
---
```

### **2. `deleteMany()`**

Deletes **multiple documents**.
```javascript
db.users.deleteMany({ role: "Intern" })  
```

# **1️⃣ What `find()` Returns**

The `find()` method returns a **cursor**.

A **cursor** is an **iterator (pointer)** that allows MongoDB to return documents **one by one instead of loading everything into memory at once**.

Example:
```javascript
db.users.find()

Return type:

Cursor → points to the result documents

Example output:

[  
 { "\_id": 1, "name": "Vikash", "role": "Developer" },  
 { "\_id": 2, "name": "Rahul", "role": "Backend Developer" }  
]
```
Internally MongoDB sends results **in batches using the cursor**.

# **Cursor Methods (Important)**

Since `find()` returns a cursor, we can use:

db.users.find().limit(5)  
db.users.find().sort({ age: 1 })  
db.users.find().skip(10)

These methods work **on the cursor**.

# **Using `find()` in Node.js (Real Full-Stack Example)**

Example with **Node.js \+ MongoDB**.

const users \= await db.collection("users").find({ role: "Developer" }).toArray();

Explanation:

* `find()` returns a **cursor**

* `.toArray()` converts cursor to **array**

Result:

\[  
 { name: "Vikash", role: "Developer" },  
 { name: "Rahul", role: "Developer" }  
\]

**`Comparison` Operator**

In **MongoDB**, **conditional operators** (also called **comparison operators**) are used to **filter documents based on conditions on field values**.  
 They are commonly used with the **`find()`** method to search specific data from a collection.

These operators compare a field’s value with a specified value and return documents that **1️⃣ `$eq` — Equal To**

Finds documents where the field value is **equal to** a given value.

Example:

db.users.find({ role: { $eq: "Developer" } })

This returns users where:

role \= Developer

# **2️⃣ `$ne` — Not Equal**

Finds documents where the field value is **not equal** to the given value.

Example:

db.users.find({ role: { $ne: "Intern" } })

Returns users where:

role ≠ Intern

# **3️⃣ `$gt` — Greater Than**

Finds documents where the field value is **greater than** a specified value.

Example:

db.users.find({ experience: { $gt: 2 } })

Meaning:

experience \> 2

# **4️⃣ `$gte` — Greater Than or Equal**

Finds documents where the value is **greater than or equal to** a value.

Example:

db.users.find({ experience: { $gte: 3 } })

Meaning:

experience ≥ 3

# **5️⃣ `$lt` — Less Than**

Finds documents where the value is **less than** a given value.

Example:

db.users.find({ salary: { $lt: 70000 } })

Meaning:

salary \< 70000

# **6️⃣ `$lte` — Less Than or Equal**

Finds documents where the value is **less than or equal to** a value.

Example:

db.users.find({ salary: { $lte: 80000 } })

Meaning:

salary ≤ 80000

# **7️⃣ `$in` — Match Any Value in a List**

Finds documents where the field value matches **any value in an array**.

Example:

db.users.find({

 role: { $in: \["Developer", "Designer"\] }

})

Meaning:

role \= Developer OR Designer

# **8️⃣ `$nin` — Not in List**

Finds documents where the field value **is not in a specified list**.

Example:

db.users.find({

 role: { $nin: \["Intern", "Tester"\] }

})

Meaning:

role ≠ Intern AND role ≠ Tester

# 

# **Logical Operators**

In **MongoDB**, **logical operators** are used to **combine multiple conditions in a query**.  
 They allow you to perform complex searches using **AND, OR, NOT type logic**.

Logical operators are commonly used with the **`find()`** method.

---

# **Logical Operators in MongoDB**

The main logical operators are:

1. `$and`  
2. `$or`  
3. `$not`  
4. `$nor`

# **1️⃣ `$and` Operator**

The **`$and`** operator returns documents where **all conditions must be true**.

### **Example**

db.users.find({

 $and: \[

   { role: "Developer" },

   { experience: { $gt: 2 } }

 \]

})

Meaning:

role \= Developer AND experience \> 2

---

### **Important Interview Point**

MongoDB **implicitly uses AND** when multiple conditions are written.

Example:

db.users.find({

 role: "Developer",

 experience: { $gt: 2 }

})

This automatically works like `$and`.

# **2️⃣ `$or` Operator**

The **`$or`** operator returns documents where **at least one condition is true**.

### **Example**

db.users.find({

 $or: \[

   { role: "Developer" },

   { role: "Designer" }

 \]

})

Meaning:

role \= Developer OR role \= Designer

# **3️⃣ `$not` Operator**

The **`$not`** operator **negates a condition**.

### **Example**

db.users.find({

 experience: { $not: { $gt: 5 } }

})

Meaning:

experience NOT greater than 5

# **4️⃣ `$nor` Operator**

The **`$nor`** operator returns documents that **do not match any of the given conditions**.

### **Example**

db.users.find({

 $nor: \[

   { role: "Intern" },

   { experience: { $lt: 2 } }

 \]

})

Meaning:

role ≠ Intern

AND

experience ≥ 2

## **What is Projection in MongoDB?**

In **MongoDB**, **projection** is used to **select which fields should be returned in the query result**.

Instead of returning the **entire document**, projection allows you to return **only the fields you need**.

This helps:

* Reduce data transfer

* Improve performance

* Return cleaner API responses

Example:

db.users.find(

 { role: "Developer" },

 { name: 1, role: 1 }

)

Here:

* `{ role: "Developer" }` → query condition  
* `{ name: 1, role: 1, salary: 0 }` → projection

Here name and role field comes in output but salary excluded. \_id included by default. You can also exclude \_id by adding it manually in the projection.

# **Why Projection is Useful**

### **1️⃣ Performance Improvement**

**2️⃣ Better API Responses**  
**3️⃣ Security**

A strong interview answer:

Projection in MongoDB is used to control which fields are returned in the query result. It is specified as the second parameter in the `find()` method. Fields can be included using `1` or excluded using `0`, and it helps improve performance by reducing the amount of data returned.

**`ordered` Option**

In MongoDB, the `ordered` option is used with `insertMany()` (and bulk operations) to control how MongoDB inserts multiple documents when an error occurs.

It determines whether MongoDB should stop or continue inserting documents if one insert fails.

# **Where `ordered` is Used**

It is mainly used with:

insertMany()

Example:

db.users.insertMany(

 \[

   { \_id: 1, name: "Vikash" },

   { \_id: 1, name: "Rahul" },

   { \_id: 3, name: "Amit" }

 \],

 { ordered: true }

)

# **1️⃣ `ordered: true` (Default)**

This means MongoDB will **insert documents in order** and **stop if an error occurs**.

Example:

db.users.insertMany(

 \[

   { \_id: 1, name: "Vikash" },

   { \_id: 1, name: "Rahul" },

   { \_id: 3, name: "Amit" }

 \],

 { ordered: true }

)

What happens:

1️⃣ First document inserted  
 2️⃣ Second document causes **duplicate `_id` error**  
 3️⃣ MongoDB **stops execution**  
 4️⃣ Third document **will NOT be inserted**

Result:

Inserted: 1 document

Stopped because of error

# 

# **2️⃣ `ordered: false`**

This means MongoDB will **continue inserting remaining documents even if one fails**.

Example:

db.users.insertMany(

 \[

   { \_id: 1, name: "Vikash" },

   { \_id: 1, name: "Rahul" },

   { \_id: 3, name: "Amit" }

 \],

 { ordered: false }

)

What happens:

1️⃣ First document inserted  
2️⃣ Second document fails (duplicate `_id`)  
3️⃣ MongoDB **continues execution**  
4️⃣ Third document **will still be inserted**

Result:

Inserted: 2 documents

1 document failed

# **Comparison**

| Option | Behavior |
| ----- | ----- |
| `ordered: true` | Stops when first error occurs |
| `ordered: false` | Continues inserting remaining documents  |

## **1️⃣ What is a Schema in MongoDB?**

In **MongoDB**, a **schema** defines the **structure of documents in a collection**.  
 It describes:

* What **fields** a document should have  
* The **data types** of those fields  
* Optional **rules or constraints**

Unlike relational databases, MongoDB is **schema-flexible (schema-less)**, meaning documents in the same collection **do not need to have exactly the same structure**.

### **Example Collection Without Schema**

Documents in `users` collection:

{  
 "name": "Vikash",  
 "age": 25  
}

Another document:

{  
 "name": "Rahul",  
 "email": "rahul@gmail.com"  
}

Both documents are valid because MongoDB allows **flexible schemas**.

### **Why Schema is Still Important**

Even though MongoDB is flexible, using a schema helps:

* Maintain **consistent data structure**  
* Prevent **invalid data**  
* Improve **data quality**  
* Make backend logic easier

Example expected structure:

{  
 "name": "string",  
 "email": "string",  
 "age": "number"  
}

# **2️⃣ What is Schema Validation?**

**Schema validation** is a feature that allows MongoDB to **enforce rules on documents before they are inserted or updated**.

It ensures documents follow a **defined structure**.

MongoDB uses **JSON Schema** for validation.

### **Example Schema Validation**

Create a collection with validation rules:

db.createCollection("users", {  
  validator: {  
    $jsonSchema: {  
      bsonType: "object",  
      required: \["name", "email"\],  
      properties: {  
        name: {  
          bsonType: "string",  
          description: "Name must be a string"  
        },  
        email: {  
          bsonType: "string",  
          description: "Email must be a string"  
        },  
        age: {  
          bsonType: "int",  
          minimum: 18  
        }  
      }  
    }  
  }  
})

### **Valid Document**

db.users.insertOne({  
 name: "Vikash",  
 email: "vikash@gmail.com",  
 age: 25  
})

This works because it follows the schema.

# **Schema vs Schema Validation**

| Feature | Schema | Schema Validation |
| ----- | ----- | ----- |
| Definition | Structure of documents | Rules enforcing the structure |
| Purpose | Design data model | Prevent invalid data |
| Mandatory | Not required | Optional but recommended |

A strong interview answer:

A schema in MongoDB defines the structure of documents in a collection, including fields and their data types. Since MongoDB is schema-flexible, documents can have different structures. Schema validation allows developers to enforce rules using JSON Schema to ensure documents follow a specific structure before being inserted or updated.

Even though **MongoDB** is **schemaless**, developers usually define schemas in applications using **Mongoose** to maintain **structure, validation, and consistency**.

**"If MongoDB is schemaless, why do we define schemas in Mongoose?"**

Good answer:

MongoDB is schemaless, meaning documents in a collection can have different structures. However, in real applications we use Mongoose schemas to enforce a consistent data structure, perform validation, define data types, and manage business logic at the application level. This helps maintain data integrity and makes the backend code more maintainable.

 **How to update Schema**

In **MongoDB**, schema modification is different from relational databases because MongoDB is **schema-flexible (schemaless)**.  
 This means you usually **modify the structure of documents directly**, rather than altering a fixed table schema.

There are **three common ways to modify schema in MongoDB**.

 **1️⃣ Add a New Field to Existing Documents**

You can modify documents by **adding a new field** using `updateMany()` with `$set`.

### **Example**

Suppose existing document:

{  
 "name": "Vikash",  
 "role": "Developer"  
}

Now you want to add `age`.

db.users.updateMany(  
 {},  
 { $set: { age: 25 } }  
)

# **2️⃣ Remove a Field**

Use `$unset` to remove a field.

### **Example**

db.users.updateMany(  
 {},  
 { $unset: { age: "" } }  
)

# **3️⃣ Rename a Field**

Use `$rename` to change field names.

Example:

db.users.updateMany(  
 {},  
 { $rename: { "role": "position" } }  
)

# **4️⃣ Modify Schema Validation Rules**

If the collection uses **schema validation**, you can update it using `collMod`.

Example:

db.runCommand({  
 collMod: "users",  
 validator: {  
   $jsonSchema: {  
     bsonType: "object",  
     required: \["name", "email"\],  
     properties: {  
       name: { bsonType: "string" },  
       email: { bsonType: "string" },  
       age: { bsonType: "int" }  
     }  
   }  
 }  
})

Now MongoDB enforces the updated schema rules.

## **What is Write Concern in MongoDB?**

In **MongoDB**, **Write Concern** controls **how MongoDB confirms that a write operation (insert, update, delete) has been successfully completed**.

It defines the **level of acknowledgment requested from MongoDB for write operations**.

In simple terms:

Write concern determines **how safely MongoDB writes data and when it confirms the write to the client**.

# **Why Write Concern is Important**

When you insert or update data, MongoDB must decide:

* Should it **acknowledge immediately**?  
* Should it wait until **data is written to disk**?  
* Should it wait until **multiple replicas confirm the write**?

Write concern controls this behavior.

# **Write Concern Levels**

### **1️⃣ `w: 0` (No Acknowledgment):-** MongoDB **does not confirm** if the write was successful.

### **2️⃣ `w: 1` (Default):-** MongoDB **confirm** if the write was successful.

### **3️⃣ `w: "majority" :-`** MongoDB waits until **most replica set nodes confirm the write**.

Example:

db.users.insertOne(  
 { name: "Vikash" },  
 { writeConcern: { w: 0 } }  
)

Behavior:

* Client sends request  
* MongoDB does not return confirmation

Used when:

* speed is more important than reliability

# **What is the MongoDB Journal?**

MongoDB maintains a **journal file** that records write operations before they are permanently written to the database files.

The journal acts like a **write-ahead log**.

Workflow:

Client → MongoDB → Journal File → Database File

If the server crashes, MongoDB can **recover data using the journal**.

# **Basic Syntax**

db.users.insertOne(  
 { name: "Vikash" },  
 { writeConcern: { j: true } }  
)

Meaning:

Write operation must be recorded in the journal before acknowledgment.  
---

# **Example Without `j`**

db.users.insertOne(  
 { name: "Vikash" },  
 { writeConcern: { w: 1 } }  
)

Process:

1\. Primary node writes data to memory  
2\. MongoDB sends acknowledgment  
3\. Data may not yet be written to disk

If a crash happens immediately → **data might be lost**.

### **What is `wtimeout` in MongoDB Write Concern?**

In MongoDB, `wtimeout` specifies the maximum amount of time (in milliseconds) that MongoDB should wait for the requested write concern to be satisfied.

# **Why `wtimeout` is Needed**

When using higher write concerns like:

w: "majority"

MongoDB must wait for **multiple replica nodes** to confirm the write.

But sometimes:

* secondary nodes may be **slow**

* network latency may occur

* nodes may be **temporarily unavailable**

Without `wtimeout`, the client might **wait too long**.

So `wtimeout` sets a **time limit**.

Example:

db.orders.insertOne(  
 { product: "Laptop" },  
 {  
   writeConcern: {  
     w: "majority",  
     wtimeout: 3000  
   }  
 }  
)

Process:

1. Primary writes the data.  
2. MongoDB waits for **majority nodes** to acknowledge.  
3. If confirmation arrives **within 3 seconds → success**.  
4. If not → **timeout error returned**.

Important:

The write may still succeed later,  
but the client receives a timeout warning.

## **What is Atomicity in MongoDB?**

In **MongoDB**, **Atomicity** means that a database operation is treated as a **single, indivisible unit of work**.

In simple words:

**Atomicity means either the entire operation succeeds or nothing happens.**

There is **no partial update**.

# **Atomicity in MongoDB**

MongoDB guarantees **atomic operations at the document level**.

Meaning:

One document update \= Atomic

Multiple document updates \= Not atomic (unless using transactions)

# **Example of Atomic Operation**

Document:

{  
 "\_id": 1,  
 "name": "Vikash",  
 "balance": 5000  
}

**Update query:**

db.users.updateOne(  
 { \_id: 1 },  
 { $set: { balance: 3000 } }  
)

MongoDB ensures:

Either balance becomes 3000  
OR it remains 5000

There is **no half update**.

# **Multi-Document Atomicity**

By default:

MongoDB atomicity \= single document only

For multiple documents we use **transactions**.

Example:

session.startTransaction()

db.accounts.updateOne(...)  
db.accounts.updateOne(...)

session.commitTransaction()

Now both operations become **atomic together**.

# **Why Atomicity is Important**

Without atomicity:

System crash during update  
Data becomes inconsistent

Example problem:

balance deducted but transaction not recorded

Atomicity prevents such issues.

**Query Helper methods**

In **MongoDB**, **query helper methods** are functions used with `find()` to **modify, control, or refine the results returned by a query**.

They help with things like:

* limiting results  
* pagination  
* sorting  
* selecting specific fields

These methods are commonly used in **backend APIs and full-stack applications**.

# **1️⃣ `limit()` – Restrict Number of Documents**

`limit()` controls **how many documents are returned**.

### **Syntax**

db.collection.find().limit(number)

# **2️⃣ `skip()` – Skip Documents**

`skip()` tells MongoDB to **skip a number of documents before returning results**.

### **Syntax**

db.collection.find().skip(number)

# **3️⃣ `sort()` – Sort Results**

`sort()` arranges documents **ascending or descending**.

### **Syntax**

db.collection.find().sort({ field: order })

Order values:

1  → Ascending

\-1 → Descending

### **Example**

db.users.find().sort({ age: 1 })

# **4️⃣ Pagination (Common Interview Example)**

Pagination uses **limit \+ skip**.

Example:

db.users.find().skip(10).limit(5)

Meaning:

Skip first 10 users

Return next 5 users

Example in API:

Page 1 → skip(0) limit(10)

Page 2 → skip(10) limit(10)

Page 3 → skip(20) limit(10)

# **5️⃣ `count()` / `countDocuments()`**

Counts the number of documents.

### **Example**

db.users.countDocuments()

With condition:

db.users.countDocuments({ age: { $gt: 25 } })

# **1️⃣ Query Evaluation Operators**

Query evaluation operators are used to **evaluate expressions or conditions during query execution**.

Common ones include:

$expr  
$regex  
$text  
$mod

They allow MongoDB to perform **advanced filtering logic**.

# **2️⃣ `$expr` Operator**

`$expr` allows **aggregation expressions inside queries**.

This means you can **compare fields within the same document**.

### **Example Document**

{  
 "item": "Laptop",  
 "price": 1000,  
 "discount": 200  
}

### **Query**

db.products.find({  
 $expr: { $gt: \["$price", "$discount"\] }  
})

Meaning:

Return documents where price \> discount

Important:

$price and $discount refer to fields inside the document

### **Use Case**

Comparing two fields in the same document.

# **What is `$regex` in MongoDB?**

`$regex` is used to **search for patterns inside string fields using regular expressions**.

In simple words:

`$regex` allows MongoDB to perform **pattern-based text matching** instead of exact matching.

Example:

Instead of searching only:

name \= "Vikash"

You can search:

names containing "vik"  
names starting with "vik"  
names ending with "sh"  
---

# **Basic Syntax**

db.collection.find({  
 field: { $regex: "pattern" }  
})

Example:

db.users.find({  
 name: { $regex: "vik" }  
})

This returns documents like:

{ "name": "vikash" }  
{ "name": "vikram" }  
{ "name": "vicky" }

# **1️⃣ Case Insensitive Search**

By default `$regex` is **case-sensitive**.

Example:

db.users.find({  
 name: { $regex: "vik", $options: "i" }  
})

`i` means **ignore case**.

This will match:

vikash  
Vikash  
VIKASH  
---

# **2️⃣ Start With Pattern (`^`)**

`^` means **string starts with**.

Example:

db.users.find({  
 name: { $regex: "^vik" }  
})

Matches:

vikash  
vikram

Does NOT match:

ravikash  
---

# **3️⃣ End With Pattern (`$`)**

`$` means **string ends with**.

Example:

db.users.find({  
 name: { $regex: "ash$" }  
})

Matches:

vikash  
akash  
---

# **4️⃣ Contains Pattern**

Example:

db.users.find({  
 name: { $regex: "ika" }  
})

Matches:

vikash  
nikan

Because **"ika" appears inside the string**.

# **What is `$text` in MongoDB?**

`$text` is used to perform **full-text search on string fields**.

In simple terms:

`$text` allows MongoDB to search **words or phrases inside large text fields efficiently using a text index**.

It is commonly used in:

* blog search

* product description search

* article search

* documentation search

---

# **Important Requirement**

Before using `$text`, you must create a **text index**.

Example collection:

{  
 "title": "MongoDB Tutorial",  
 "content": "Learn MongoDB database for backend development"  
}

Create a text index:

db.articles.createIndex({ content: "text" })

Now MongoDB can perform **text search on the `content` field**.

---

# **Basic Syntax**

db.collection.find({  
 $text: { $search: "keyword" }  
})

Example:

db.articles.find({  
 $text: { $search: "mongodb" }  
})

Result:

Returns all documents containing the word "mongodb"

# **Searching Multiple Fields**

You can create a text index on multiple fields.

Example:

db.articles.createIndex({  
 title: "text",  
 content: "text"  
})

Query:

db.articles.find({  
 $text: { $search: "mongodb" }  
})

Now MongoDB searches in:

title \+ content  
---

The `$text` operator in MongoDB is used for full-text search. It allows searching words or phrases inside text fields that have a text index. MongoDB tokenizes the text and returns documents matching the search terms, optionally ranking them by relevance using text scores.

# **5️⃣ `$mod` Operator**

`$mod` performs **modulus operations** (remainder).

Syntax:

{ field: { $mod: \[divisor, remainder\] } }

### **Example**

db.numbers.find({  
 value: { $mod: \[2, 0\] }  
})

Meaning:

Return numbers divisible by 2

In **MongoDB**, **Element Query Operators** are used to check the **existence or data type of a field in a document**.

The two main element operators are:

$exists  
$type

They are useful when documents in a collection **do not have the same structure**, which is common in MongoDB because it is **schemaless**.

---

# **1️⃣ `$exists` Operator**

## **What it Does**

`$exists` checks **whether a field exists or not in a document**.

Syntax:

db.collection.find({  
 field: { $exists: true | false }  
})  
---

## **Example Dataset**

{ "name": "Vikash", "age": 25 }

{ "name": "Rahul" }

{ "name": "Anita", "age": 30 }  
---

## **Example 1 – Find Documents Where Field Exists**

db.users.find({  
 age: { $exists: true }  
})

Result:

{ "name": "Vikash", "age": 25 }

{ "name": "Anita", "age": 30 }

MongoDB returns documents **where the field `age` is present**.

## **Example 2 – Find Documents Where Field Does Not Exist**

db.users.find({  
 age: { $exists: false }  
})

Result:

{ "name": "Rahul" }

This returns documents **missing the `age` field**.

---

# **2️⃣ `$type` Operator**

## **What it Does**

`$type` checks the **data type of a field**.

Syntax:

db.collection.find({  
 field: { $type: "datatype" }  
})  
---

## **Example Dataset**

{ "name": "Vikash", "age": 25 }

{ "name": "Rahul", "age": "30" }

{ "name": "Anita", "age": 28 }

Notice:

age \= number in some documents  
age \= string in one document  
---

## **Example Dataset**

{ "name": "Vikash", "age": 25 }

{ "name": "Rahul", "age": "30" }

{ "name": "Anita", "age": 28 }

Notice:

age \= number in some documents  
age \= string in one document  
---

# **Example Combining `$exists` and `$type`**

db.users.find({  
 age: { $exists: true, $type: "int" }  
})

Meaning:

Return documents where:  
1\. age field exists  
2\. age is integer

Element query operators in MongoDB check the structure of documents rather than their values. `$exists` verifies whether a field is present in a document, while `$type` checks the data type of a field. These operators are useful in schemaless databases where documents may have different structures.

In **MongoDB**, operators like **`$size`**, **`$all`**, and **`$in`** are used to **query arrays or match values against multiple options**. These are commonly used when documents contain **array fields**, which is very common in MongoDB applications.

# **1️⃣ `$size` Operator**

## **What it Does**

`$size` is used to **find documents where an array has a specific number of elements**.

### **Syntax**

db.collection.find({  
 field: { $size: number }  
})

## **Example Dataset**

{ "name": "Vikash", "skills": \["JavaScript", "Node.js", "MongoDB"\] }

{ "name": "Rahul", "skills": \["Java", "Spring"\] }

{ "name": "Anita", "skills": \["Python", "Django", "React"\] }  
---

## **Query Example**

db.users.find({  
 skills: { $size: 3 }  
})

Result:

{ "name": "Vikash", "skills": \["JavaScript", "Node.js", "MongoDB"\] }

{ "name": "Anita", "skills": \["Python", "Django", "React"\] }

Meaning:

Find users having exactly 3 skills  
---

## **Real Use Case**

* Finding users with **exact number of roles/skills**

* Checking **array length conditions**

# **2️⃣ `$all` Operator**

## **What it Does**

`$all` finds documents where an array **contains all the specified elements**.

### **Syntax**

db.collection.find({  
 field: { $all: \[value1, value2\] }  
})

## **Query Example**

db.users.find({  
 skills: { $all: \["JavaScript", "MongoDB"\] }  
})

Meaning:

Return users who have BOTH JavaScript AND MongoDB skills

## **`$in` with Arrays**

Example:

db.users.find({  
 skills: { $in: \["MongoDB", "Python"\] }  
})

Meaning:

Find users who know MongoDB OR Python

# **Example Combining Operators**

db.users.find({  
 skills: { $all: \["JavaScript", "MongoDB"\] },  
 city: { $in: \["Delhi", "Pune"\] }  
})

Meaning:

Users who know JavaScript and MongoDB  
AND live in Delhi or Pune

**How to apply search on a array of object with multiple fields**

In **MongoDB**, when you want to **search inside an array of objects using multiple fields**, the correct operator to use is **`$elemMatch`**.

Without `$elemMatch`, MongoDB may match **different elements of the array**, which can give incorrect results.

---

# **1️⃣ Example Dataset**

Suppose we have a `students` collection:

{  
 "name": "Vikash",  
 "marks": \[  
   { "subject": "Math", "score": 80 },  
   { "subject": "English", "score": 60 }  
 \]  
}

Another document:

{  
 "name": "Rahul",  
 "marks": \[  
   { "subject": "Math", "score": 50 },  
   { "subject": "English", "score": 90 }  
 \]  
}

Here **`marks` is an array of objects**.

---

# **2️⃣ Problem Without `$elemMatch`**

If you write:

db.students.find({  
 "marks.subject": "Math",  
 "marks.score": { $gt: 70 }  
})

MongoDB may return incorrect results because:

subject may match one element  
score may match another element

Example:

subject \= Math (first element)  
score \> 70 (second element)

So MongoDB might return the document even though **both conditions are not satisfied in the same object**.

# **3️⃣ Correct Solution: `$elemMatch`**

Use `$elemMatch` to ensure **both conditions apply to the same array element**.

### **Query**

db.students.find({  
 marks: {  
   $elemMatch: {  
     subject: "Math",  
     score: { $gt: 70 }  
   }  
 }  
})

Meaning:

Find students who scored \>70 in Math

Now MongoDB checks **both fields inside the same object**.

Result:

{  
 "name": "Vikash",  
 "marks": \[  
   { "subject": "Math", "score": 80 },  
   { "subject": "English", "score": 60 }  
 \]  
}

Rahul is not returned because:

Math score \= 50

# **7️⃣ Real Backend Use Cases**

Searching inside arrays of objects is common in:

### **E-commerce**

products → reviews array

Example:

db.products.find({  
 reviews: {  
   $elemMatch: {  
     rating: { $gte: 4 },  
     verified: true  
   }  
 }  
})

We use the `$elemMatch` operator to apply multiple conditions to the same element of an array. It ensures that all conditions match within the same array object instead of matching different elements.

In **MongoDB**, operators like **`$inc`, `$min`, `$max`, `$mul`, `$unset`, and `$rename`** are **update operators**.  
 They are used with update operations such as `updateOne()` or `updateMany()` to **modify specific fields in documents without replacing the entire document**.

---

# **1️⃣ `$inc` Operator**

## **Purpose**

`$inc` is used to **increment (increase) or decrement numeric values**.

### **Example Document**

{ "name": "Vikash", "score": 50 }

### **Query**

db.users.updateOne(  
 { name: "Vikash" },  
 { $inc: { score: 10 } }  
)

Result:

{ "name": "Vikash", "score": 60 }

You can also decrease values:

{ $inc: { score: \-5 } }

# **2️⃣ `$min` Operator**

## **Purpose**

`$min` updates a field **only if the new value is smaller than the current value**.

### **Example**

{ "name": "Vikash", "age": 30 }

Query:

db.users.updateOne(  
 { name: "Vikash" },  
 { $min: { age: 25 } }  
)

Result:

{ "name": "Vikash", "age": 25 }

If the value is larger, MongoDB **does not update**.

# **3️⃣ `$max` Operator**

## **Purpose**

`$max` updates a field **only if the new value is greater than the current value**.

Example:

{ "name": "Vikash", "age": 25 }

Query:

db.users.updateOne(  
 { name: "Vikash" },  
 { $max: { age: 30 } }  
)

Result:

{ "name": "Vikash", "age": 30 }

### **Real Use Case**

Tracking **highest score** or **maximum salary**.

# **4️⃣ `$mul` Operator**

## **Purpose**

`$mul` multiplies the value of a field by a specified number.

### **Example**

Document:

{ "product": "Laptop", "price": 1000 }

Query:

db.products.updateOne(  
 { product: "Laptop" },  
 { $mul: { price: 2 } }

# **5️⃣ `$unset` Operator**

## **Purpose**

`$unset` removes a field from a document.

### **Example**

Document:

{ "name": "Vikash", "age": 25, "city": "Delhi" }

Query:

db.users.updateOne(  
 { name: "Vikash" },  
 { $unset: { city: "" } }  
)

# **6️⃣ `$rename` Operator**

## **Purpose**

`$rename` changes the **name of a field**.

### **Example**

Document:

{ "name": "Vikash", "city": "Delhi" }

Query:

db.users.updateOne(  
 { name: "Vikash" },  
 { $rename: { "city": "location" } }  
)

In **MongoDB**, operators like **`$push`**, **`$pull`**, **`$pop`**, and **`$addToSet`** are **array update operators**.  
 They are used to **add, remove, or modify elements inside arrays**, including **nested arrays inside objects**.

These operators are very common in real applications such as:

* comments in posts

* product reviews

* tags in blogs

* user roles or permissions

---

# **Example Dataset (Nested Array)**

In **MongoDB**, operators like **`$push`**, **`$pull`**, **`$pop`**, and **`$addToSet`** are **array update operators**.  
 They are used to **add, remove, or modify elements inside arrays**, including **nested arrays inside objects**.

These operators are very common in real applications such as:

* comments in posts

* product reviews

* tags in blogs

* user roles or permissions

---

# **Example Dataset (Nested Array)**

{  
 "name": "Vikash",  
 "skills": \["JavaScript", "Node.js"\],  
 "projects": \[  
   {  
     "title": "Shop App",  
     "technologies": \["React", "MongoDB"\]  
   },  
   {  
     "title": "Chat App",  
     "technologies": \["Node.js", "Socket.io"\]  
   }  
 \]  
}

Here we have:

skills → array  
projects → array of objects  
technologies → nested array

## **`$push` with Nested Array**

Example: add technology to a project.

db.users.updateOne(  
 { "projects.title": "Shop App" },  
 { $push: { "projects.$.technologies": "Node.js" } }  
)

Result:

"technologies": \["React", "MongoDB", "Node.js"\]

`$` refers to the **matched array element**.

# **2️⃣ `$pull` (Remove element from array)**

## **Purpose**

Removes elements that **match a condition**.

Example:

db.users.updateOne(  
 { name: "Vikash" },  
 { $pull: { skills: "Node.js" } }  
)

Result:

"skills": \["JavaScript"\]  
---

## **`$pull` with Nested Array**

db.users.updateOne(  
 { "projects.title": "Shop App" },  
 { $pull: { "projects.$.technologies": "MongoDB" } }  
)

Result:

"technologies": \["React"\]

# **3️⃣ `$pop` (Remove first or last element)**

## **Purpose**

Removes **first or last element** of an array.

Values:

1  → remove last element  
\-1 → remove first element

Example:

db.users.updateOne(  
 { name: "Vikash" },  
 { $pop: { skills: 1 } }  
)

Result:

"skills": \["JavaScript"\]

# **4️⃣ `$addToSet` (Add unique value)**

## **Purpose**

Adds a value **only if it does not already exist**.

Example:

db.users.updateOne(  
 { name: "Vikash" },  
 { $addToSet: { skills: "MongoDB" } }  
)

If MongoDB already exists:

Nothing changes

Result:

"skills": \["JavaScript", "MongoDB"\]

# **`$addToSet` with `$each`**

To add multiple unique values:

db.users.updateOne(  
 { name: "Vikash" },  
 {  
   $addToSet: {  
     skills: { $each: \["MongoDB", "Express"\] }  
   }  
 }  
)

Result:

"skills": \["JavaScript", "MongoDB", "Express"\]

Duplicates are automatically ignored.

# **Example with Nested Array Object**

Dataset:

{  
 "title": "MongoDB Guide",  
 "comments": \[  
   { "user": "Vikash", "text": "Great article" },  
   { "user": "Rahul", "text": "Very helpful" }  
 \]  
}

### **Add comment**

db.posts.updateOne(  
 { title: "MongoDB Guide" },  
 {  
   $push: {  
     comments: { user: "Anita", text: "Nice tutorial" }  
   }  
 }  
)

# **Quick Comparison**

| Operator | Purpose |
| ----- | ----- |
| `$push` | Add element to array |
| `$pull` | Remove matching element |
| `$pop` | Remove first/last element |
| `$addToSet` | Add unique value |

**What is Indexing**  
Indexing in MongoDB is used to improve query performance by allowing the database to quickly locate documents without scanning the entire collection. It works like an index in a book. Indexes are useful for frequently queried fields, sorting, and large datasets, but they come with trade-offs like increased storage and slower write operations.

In **MongoDB**, **indexing** is a way to **store a small portion of data in a structured format** so that MongoDB can **quickly locate documents without scanning the entire collection**.

👉 Think of it like a **book index**:

* Without index → read whole book 📖

* With index → jump directly to the page 🚀

# 

# 

# **Basic Syntax**

db.collection.createIndex({ field: 1 })

* `1` → ascending order  
* `-1` → descending order  
* order

---

# **🔹 1\. Single Field Index**

db.users.createIndex({ email: 1 })

👉 Useful when you frequently search:

db.users.find({ email: "test@gmail.com" })

# **🔹 2\. Compound Index (Multiple Fields)**

db.users.createIndex({ name: 1, age: \-1 })

👉 Helps for queries like:

db.users.find({ name: "vikash", age: { $gt: 25 } })

⚠️ Order matters:

* `{ name: 1, age: -1 }` works for:  
  * `name`  
  * `name + age`  
* ❌ Not efficient for `age` alone

# **🔹 3\. Unique Index**

db.users.createIndex({ email: 1 }, { unique: true })

👉 Prevents duplicate values (like emails)

---

# **🔹 4\. Text Index (for search)**

db.posts.createIndex({ title: "text", description: "text" })

👉 Used with:

db.posts.find({ $text: { $search: "mongodb" } })

# **How MongoDB Works Without Index**

db.users.find({ age: 25 })

Without index:

MongoDB scans ALL documents → slow (O(n))

This is called:

COLLSCAN (Collection Scan)

# **With Index**

Create index:

db.users.createIndex({ age: 1 })

Now query:

db.users.find({ age: 25 })

MongoDB:

Uses index → directly finds matching documents → fast (O(log n))

# **Why We Need Indexing**

## **1️⃣ Faster Queries**

Without index:

1 million records → scan all

With index:

Search directly → milliseconds  
---

## **2️⃣ Efficient Filtering**

Example:

db.users.find({ email: "test@gmail.com" })

Index on `email` → super fast lookup.

## **3️⃣ Sorting Optimization**

db.users.find().sort({ age: 1 })

With index → no extra sorting needed.

---

## **4️⃣ Better Performance for APIs**

In full-stack apps:

Backend API → DB query → Response

Indexing reduces:

Response time → improves UX

# **Types of Indexes (Important for Interviews)**

### **1️⃣ Single Field Index**

db.users.createIndex({ age: 1 })  
---

### **2️⃣ Compound Index**

db.users.createIndex({ age: 1, name: 1 })

Used when querying multiple fields.

---

### **3️⃣ Text Index**

Used with `$text`:

db.posts.createIndex({ content: "text" })  
---

### **4️⃣ Unique Index**

db.users.createIndex({ email: 1 }, { unique: true })

Prevents duplicate values.

# **🔹 8\. Check Existing Indexes**

db.users.getIndexes()

---

# **🔹 9\. Drop Index**

db.users.dropIndex("email\_1")

# **When Should You Use Indexing?**

## **✅ Use Index When:**

### **1\. Frequent Search Fields**

2\. Sorting Fields  
3\. Filtering Large Data  
4\. High Read Operations

# **When NOT to Use Indexing**

## **❌ Avoid Index When:**

### **1\. Small Collections**

### **2\. Too Many Indexes**

3\. Frequently Changing Fields  
4\. HIgh Write operaions

# **🔹 What is `explain()`?**

It shows:

* ✅ Which index is used  
* ✅ How many documents are scanned  
* ✅ How MongoDB executes your query internally

👉 Think of it as a **query performance debugger**

The `explain()` method in MongoDB is used to **analyze how a query is executed**. It helps you understand whether your query is efficient or not.

# **🔹 Example**

db.users.find({ email: "test@gmail.com" }).explain()

# **🔹 Types of Explain Modes**

### **1\. Default (`queryPlanner`)**

db.users.find({ email: "test@gmail.com" }).explain("queryPlanner")

👉 Shows:

* Which index MongoDB *plans* to use

---

### **2\. Execution Stats (Most Important 🔥)**

db.users.find({ email: "test@gmail.com" }).explain("executionStats")

👉 Shows:

* Actual execution details

* Documents scanned vs returned

---

### **3\. All Plans Execution**

db.users.find({ email: "test@gmail.com" }).explain("allPlansExecution")

👉 Shows:

* All possible query plans MongoDB considered

# **🔹 When to Use `explain()`**

✔ Performance tuning  
 ✔ Debugging slow queries  
 ✔ Checking index usage  
 ✔ Interview questions

# **🔹 What is a Covered Query?**

A query is called **covered** when:

✅ **All fields in the query (filter \+ projection)** are satisfied using **only the index**  
 ❌ MongoDB does **NOT** need to look at the actual documents

👉 Means: **data is returned directly from index**

---

# **🔥 Simple Definition**

A covered query is a query where MongoDB can answer using only the index without scanning the collection.

---

# **🔹 Example**

### **Step 1: Create Index**

db.users.createIndex({ name: 1, age: 1 })

---

### **Step 2: Covered Query**

db.users.find(

 { name: "vikash" },

 { name: 1, age: 1, \_id: 0 }

)

👉 This is a **covered query** because:

* `name` → used in filter ✅ (in index)

* `name, age` → used in projection ✅ (in index)

* `_id: 0` → excluded (important ⚠️)

---

# **🔹 What is a Winning Plan?**

When you run a query, MongoDB:

1. Generates **multiple possible plans** (using different indexes or scans)  
2. Tests them internally  
3. Picks the **most efficient one**

👉 That selected plan is called the **winning plan**

In MongoDB, the **winning plan** is the **final query execution plan that MongoDB chooses** after evaluating multiple possible ways to run your query.

The winning plan is the best execution strategy chosen by MongoDB to run a query efficiently.

# **🔹 Where You See It**

You can see it using `explain()`:

db.users.find({ age: 25 }).explain()

👉 Inside output:

{

 "queryPlanner": {

   "winningPlan": {

     "stage": "IXSCAN",

     ...

   }

 }

}

# **🔹 What is Text Indexing?**

A **text index** allows MongoDB to perform **full-text search** on string fields instead of simple exact matching.

👉 It breaks text into **words (tokens)** and indexes them.

---

# **🔥 Simple Definition**

Text indexing enables fast and flexible searching of words inside string fields.

---

# **🔹 Why Normal Index is Not Enough?**

### **Without text index:**

db.posts.find({ title: "mongodb tutorial" })

❌ Only matches exact string  
 ❌ Slow for partial search

### **With text index:**

db.posts.find({ $text: { $search: "mongodb tutorial" } })

✅ Matches:

* "MongoDB tutorial for beginners"  
* "Advanced MongoDB guide"

---

# **🔹 How to Create Text Index**

db.posts.createIndex({ title: "text", description: "text" })

👉 You can create text index on:

* Single field  
* Multiple fields

# **🔹 How It Works Internally**

MongoDB:

1. Breaks text into words (tokens)  
2. Removes stop words (like "the", "is")  
3. Stores them in index  
4. Performs fast lookup

# **🔴 Limitations**

❌ Only one text index per collection  
 ❌ Not good for:

* Exact filtering (use normal index)

* Complex search (use Atlas Search)

# **🔹 1\. What is Score? (Text Score)**

### **👉 Definition**

**Score** is a number that tells **how relevant a document is to the search query**.

### **🔥 Simple Meaning**

Higher score \= better match

* partial

---

# **🔹 2\. What is Weight?**

### **👉 Definition**

**Weight** decides **importance of fields** in text search.

---

### **🔥 Simple Meaning**

Higher weight \= more impact on score

# **🔹 Big Picture (1 line)**

* **Weight → decides importance of fields**  
* **Score → decides ranking of results**

---

# **🔹 Think of It Like Google Search 🔍**

When you search something:

* Results where keyword is in **title** → come first

* Results where keyword is in **description** → come later

👉 That’s exactly:

* **Weight \= importance (title \> description)**

* **Score \= ranking (who comes first)**

# **🔹 1\. Weight (Set at Index Time)**

### **👉 What it does**

Controls **which field matters more**

---

### **🔹 Example**

db.posts.createIndex(  
 { title: "text", description: "text" },  
 {  
   weights: {  
     title: 5,  
     description: 1  
   }  
 }  
)

### **👉 Meaning**

* `title` is **5x more important**

* `description` is less important

# **🔹 2\. Score (Calculated at Query Time)**

### **👉 What it does**

Gives each document a **relevance value**

---

### **🔹 Example Query**

db.posts.find(  
 { $text: { $search: "mongodb tutorial" } },  
 { score: { $meta: "textScore" } }  
).sort({ score: { $meta: "textScore" } })  
---

# **🔹 Simple Real Example**

### **Data:**

Doc1: { title: "MongoDB tutorial", description: "Learn database" }  
Doc2: { title: "Learn database", description: "MongoDB tutorial" }

### **What Happens?**

Both documents match `"mongodb tutorial"` ✅

But:

* Doc1 → match in **title (high weight)**  
* Doc2 → match in **description (low weight)**

---

### **Final Result:**

* Doc1 → 🔥 Higher score → comes first  
* Doc2 → Lower score → comes later

# **🔹 What is Aggregation?**

### **👉 Simple Definition**

Aggregation is used to **process data and transform it into meaningful results** (like sum, average, grouping, filtering).

---

# **🔥 Real-Life Analogy**

Think of it like:

* Excel operations (SUM, GROUP BY, FILTER)  
* SQL:

SELECT category, SUM(price) FROM products GROUP BY category;

👉 MongoDB does this using **Aggregation Pipeline**

---

# **🔹 What is Aggregation Pipeline?**

It is a **sequence of stages**, where each stage processes data and passes it to the next.

db.collection.aggregate(\[  
 { stage1 },  
 { stage2 },  
 { stage3 }  
\])

👉 Data flows like a pipeline 🚿

# **🔹 What is `$match` in MongoDB?**

### **👉 Definition**

`$match` is used to **filter documents** in an aggregation pipeline.

# **🔥 Simple Understanding**

👉 It works exactly like:

* `find()` in MongoDB  
* `WHERE` in SQL

---

# **🔹 Basic Syntax**

db.collection.aggregate(\[  
 { $match: { field: value } }  
\])

# **🔹 Example**

### **📌 Data (orders)**

\[  
 { "item": "laptop", "price": 1000, "status": "completed" },  
 { "item": "phone", "price": 500, "status": "pending" }  
\]  
---

### **🎯 Goal:**

Get only **completed orders**

---

### **✅ Query**

db.orders.aggregate(\[  
 { $match: { status: "completed" } }  
\])

### **✅ Output**

\[  
 { "item": "laptop", "price": 1000, "status": "completed" }  
\]

# **🔹 Using Conditions**

### **👉 Example: price \> 500**

db.orders.aggregate(\[  
 { $match: { price: { $gt: 500 } } }  
\])  
---

# **🔹 Multiple Conditions**

db.orders.aggregate(\[  
 {  
   $match: {  
     status: "completed",  
     price: { $gt: 500 }  
   }  
 }  
\])

# **🔹 Using `$and`, `$or`**

db.orders.aggregate(\[  
 {  
   $match: {  
     $or: \[  
       { price: { $lt: 100 } },  
       { status: "pending" }  
     \]  
   }  
 }  
\])  
---

# **🔹 Important Rule (Very Important 🔥)**

👉 Always place `$match` **as early as possible** in pipeline

### **❓ Why?**

* Reduces number of documents

* Improves performance 🚀

# **🔹 Example with Pipeline**

db.orders.aggregate(\[  
 { $match: { status: "completed" } }, // filter first  
 {  
   $group: {  
     \_id: "$item",  
     total: { $sum: "$price" }  
   }  
 }  
\])

# **🔹 What is `$group`?**

### **👉 Definition**

`$group` is used to **group documents based on a field** and perform **calculations (like sum, avg, count, etc.)** on each group.

# **🔹 Basic Syntax**

db.collection.aggregate(\[  
 {  
   $group: {  
     \_id: \<group\_by\_field\>,  
     fieldName: { \<operator\>: \<expression\> }  
   }  
 }  
\])

# **🔹 Example 1 (Basic)**

### **📌 Data:**

\[  
 { "item": "laptop", "category": "electronics", "price": 1000 },  
 { "item": "phone", "category": "electronics", "price": 500 },  
 { "item": "shirt", "category": "clothing", "price": 50 }  
\]  
---

### **🎯 Goal:**

Get **total price per category**

### **✅ Query:**

db.orders.aggregate(\[  
 {  
   $group: {  
     \_id: "$category",  
     totalSales: { $sum: "$price" }  
   }  
 }  
\])

### **✅ Output:**

\[  
 { "\_id": "electronics", "totalSales": 1500 },  
 { "\_id": "clothing", "totalSales": 50 }  
\]  
---

# **🔹 Key Concept: `_id`**

👉 `_id` in `$group` \= **grouping key**

* `_id: "$category"` → group by category

* `_id: null` → group all documents together

# **🔹 Example 3 (Average Price)**

db.orders.aggregate(\[  
 {  
   $group: {  
     \_id: "$category",  
     avgPrice: { $avg: "$price" }  
   }  
 }  
\])

# **🔹 Example 4 (Using `$push`)**

db.orders.aggregate(\[  
 {  
   $group: {  
     \_id: "$category",  
     items: { $push: "$item" }  
   }  
 }  
\])

### **Output:**

{  
 "\_id": "electronics",  
 "items": \["laptop", "phone"\]  
}

# **🔹 Combine with `$match` (Best Practice)**

db.orders.aggregate(\[  
 { $match: { status: "completed" } },  
 {  
   $group: {  
     \_id: "$category",  
     total: { $sum: "$price" }  
   }  
 }  
\])

# **🔹 What is `$project`?**

### **👉 Definition**

`$project` is used to **select, include, exclude, or reshape fields** in documents.

---

# **🔥 Simple Understanding**

👉 Think of it like:

* **SQL → SELECT name, age**

* Choosing what fields you want in output

---

# **🔹 Basic Syntax**

db.collection.aggregate(\[  
 {  
   $project: {  
     field1: 1,  // include  
     field2: 1,  
     field3: 0   // exclude  
   }  
 }  
\])

# **🔹 Example 1 (Select Fields)**

### **📌 Data:**

{  
 "name": "vikash",  
 "age": 25,  
 "city": "patna"  
}  
---

### **🎯 Goal:**

Show only `name` and `age`

---

### **✅ Query:**

db.users.aggregate(\[  
 {  
   $project: {  
     name: 1,  
     age: 1,  
     \_id: 0  
   }  
 }  
\])  
---

### **✅ Output:**

{  
 "name": "vikash",  
 "age": 25  
}

# **🔹 Example 2 (Rename Fields)**

db.users.aggregate(\[  
 {  
   $project: {  
     username: "$name",  
     userAge: "$age",  
     \_id: 0  
   }  
 }  
\])

👉 Output:

{  
 "username": "vikash",  
 "userAge": 25  
}

# **🔥 Key Uses of `$project`**

✅ Select specific fields  
 ✅ Rename fields  
 ✅ Remove unwanted fields  
 ✅ Create new computed fields  
 ✅ Reshape document structure

# **🔹 1\. `$unwind` (Very Important 🔥)**

### **👉 Definition**

`$unwind` is used to **break an array into multiple documents** (one document per element).

---

# **🔥 Simple Understanding**

👉 Converts:

{ name: "vikash", skills: \["js", "react", "node"\] }

👉 Into:

{ name: "vikash", skills: "js" }  
{ name: "vikash", skills: "react" }  
{ name: "vikash", skills: "node" }  
---

# **🔹 Syntax**

{ $unwind: "$arrayField" }  
---

# **🔹 Example**

### **📌 Data:**

{  
 "name": "vikash",  
 "skills": \["js", "react"\]  
}  
---

### **✅ Query**

db.users.aggregate(\[  
 { $unwind: "$skills" }  
\])  
---

### **✅ Output**

{ "name": "vikash", "skills": "js" }  
{ "name": "vikash", "skills": "react" }  
---

# **🔹 Why Use `$unwind`?**

✔ When working with arrays  
 ✔ For grouping or counting array values  
 ✔ For analytics

# **🔹 2\. `$ifNull`**

### **👉 Definition**

`$ifNull` is used to **handle null or missing values** by providing a default value.

---

# **🔥 Simple Understanding**

👉 Like:

value ?? "default"

# **🔹 Example**

### **📌 Data:**

{ "name": "vikash", "city": null }  
---

### **✅ Query**

db.users.aggregate(\[  
 {  
   $project: {  
     name: 1,  
     city: { $ifNull: \["$city", "Not Provided"\] }  
   }  
 }  
\])

### **✅ Output**

{  
 "name": "vikash",  
 "city": "Not Provided"  
}

# **🔹 What is `$filter`?**

### **👉 Definition**

`$filter` is used to **filter elements inside an array** based on a condition.

# **🔹 Example 1 (Basic)**

### **📌 Data:**

{  
 "name": "vikash",  
 "marks": \[40, 60, 80, 30\]  
}  
---

### **🎯 Goal:**

Get marks **greater than 50**

### **✅ Query**

db.students.aggregate(\[  
 {  
   $project: {  
     name: 1,  
     passedMarks: {  
       $filter: {  
         input: "$marks",  
         as: "mark",  
         cond: { $gt: \["$$mark", 50\] }  
       }  
     }  
   }  
 }  
\])  
---

### **✅ Output**

{  
 "name": "vikash",  
 "passedMarks": \[60, 80\]  
}

# **🔹 Example 2 (Array of Objects 🔥)**

### **📌 Data:**

{  
 "name": "vikash",  
 "orders": \[  
   { "item": "laptop", "price": 1000 },  
   { "item": "phone", "price": 500 }  
 \]  
}  
---

### **🎯 Goal:**

Get orders with **price \> 600**

### **✅ Query**

db.users.aggregate(\[  
 {  
   $project: {  
     name: 1,  
     expensiveOrders: {  
       $filter: {  
         input: "$orders",  
         as: "order",  
         cond: { $gt: \["$$order.price", 600\] }  
       }  
     }  
   }  
 }  
\])  
---

### **✅ Output**

{  
 "name": "vikash",  
 "expensiveOrders": \[  
   { "item": "laptop", "price": 1000 }  
 \]  
}

# **🔹 What is `$bucket`?**

### **👉 Definition**

`$bucket` is used to **group documents into ranges (buckets)** based on a field value.

---

# **🔥 Simple Understanding**

👉 Instead of grouping by exact value (`$group`),  
 👉 `$bucket` groups data into **ranges**

# **🔹 Real-Life Example 🧠**

Think of:

* Age groups:

  * 0–18 → Teen

  * 19–40 → Adult

  * 41+ → Senior

👉 This is exactly what `$bucket` does

# **🔹 Syntax**

{  
 $bucket: {  
   groupBy: \<field\>,  
   boundaries: \[ \<lower1\>, \<lower2\>, \<lower3\>, ... \],  
   default: "Other", // optional  
   output: {  
     \<field\>: { \<operator\>: \<expression\> }  
   }  
 }  
}

# **🔹 Example 1 (Basic)**

### **📌 Data:**

\[  
 { "name": "A", "age": 15 },  
 { "name": "B", "age": 25 },  
 { "name": "C", "age": 45 }  
\]  
---

### **🎯 Goal:**

Group users by age ranges

---

### **✅ Query**

db.users.aggregate(\[  
 {  
   $bucket: {  
     groupBy: "$age",  
     boundaries: \[0, 18, 40, 60\],  
     default: "Other",  
     output: {  
       count: { $sum: 1 }  
     }  
   }  
 }  
\])  
---

### **✅ Output**

\[  
 { "\_id": 0, "count": 1 },   // age 0–18  
 { "\_id": 18, "count": 1 },  // age 18–40  
 { "\_id": 40, "count": 1 }   // age 40–60  
\]

# **🔹 How Boundaries Work (Important ⚠️)**

\[0, 18, 40, 60\]

👉 Buckets:

* 0 ≤ age \< 18

* 18 ≤ age \< 40

* 40 ≤ age \< 60

---

# **🔹 Example 2 (With Sum)**

### **🎯 Total salary per range**

db.employees.aggregate(\[  
 {  
   $bucket: {  
     groupBy: "$salary",  
     boundaries: \[0, 50000, 100000, 200000\],  
     output: {  
       totalSalary: { $sum: "$salary" },  
       count: { $sum: 1 }  
     }  
   }  
 }  
\])

# **🔹 What is `$lookup`?**

### **👉 Definition**

`$lookup` is used to **join data from another collection**.

---

# **🔥 Simple Understanding**

👉 It works like:

* **SQL → JOIN**

* Combines data from two collections

---

# **🔹 Why Do We Need `$lookup`?**

MongoDB is **NoSQL (no joins by default)**, but sometimes:

* Data is stored in separate collections

* You need related data together

👉 `$lookup` solves this problem

---

# **🔹 Syntax**

{  
 $lookup: {  
   from: "\<collection\_name\>",  
   localField: "\<field\_in\_current\_collection\>",  
   foreignField: "\<field\_in\_other\_collection\>",  
   as: "\<output\_array\_field\>"  
 }  
}  
---

# **🔹 Example (Most Important 🔥)**

### **📌 Collections:**

#### **users**

{ "\_id": 1, "name": "vikash" }

#### **orders**

{ "item": "laptop", "userId": 1 }  
---

### **🎯 Goal:**

Get user details with orders

---

### **✅ Query**

db.orders.aggregate(\[  
 {  
   $lookup: {  
     from: "users",  
     localField: "userId",  
     foreignField: "\_id",  
     as: "userDetails"  
   }  
 }  
\])  
---

### **✅ Output**

{  
 "item": "laptop",  
 "userId": 1,  
 "userDetails": \[  
   { "\_id": 1, "name": "vikash" }  
 \]  
}

# **🔹 When to Use `$lookup`?**

✔ When data is in multiple collections  
 ✔ When you need related data together  
 ✔ For reporting and APIs

# 

# **🔥 Real-World Use Cases**

* E-commerce:  
  * Orders \+ user details  
* Blog:  
  * Posts \+ author info  
* Social media:  
  * Posts \+ comments

"`$lookup` is used in MongoDB aggregation to perform left outer joins between collections and combine related data."

---

# **🔹 Memory Trick 🧠**

* `$lookup` \= **look into another collection**

# **🔹 Types of Joins (Conceptual)**

---

# **🔥 1\. Inner Join**

### **👉 Definition**

Returns only matching records from both collections

# **🔥 2\. Left Outer Join (Default in MongoDB)**

### **👉 Definition**

Returns all records from left collection \+ matched from right

# **🔥 3\. Right Join**

### **👉 Definition**

Returns all records from right collection \+ matched from left

# **🔥 5\. Cross Join (Cartesian Product)**

### **👉 Definition**

Combines every record from both collections

# **🔹 What is Replication?**

### **👉 Definition**

Replication is the process of **copying data across multiple servers** to ensure availability and reliability.

---

# **🔥 Simple Understanding**

👉 Instead of storing data on one server:

* You store **same data on multiple servers**  
* If one fails → others still work

---

# **🔹 MongoDB Uses → Replica Set**

👉 A group of MongoDB servers is called a **Replica Set**

---

# **🔹 Components of Replica Set**

### **1\. Primary Node 🟢**

* Handles **read & write operations**  
* Main database

---

### **2\. Secondary Nodes 🔵**

* Copies data from primary  
* Used for **read scaling & backup**

---

### **3\. Arbiter (Optional) ⚪**

* No data stored  
* Helps in **election voting**

---

# **🔹 How It Works**

1. Client writes data → **Primary**  
2. Primary updates data  
3. Secondaries copy data (replication)  
4. If primary fails → new primary is elected automatically

---

# **🔥 Visual Flow**

Client → Primary → Secondary1  
                 → Secondary2  
---

# **🔹 Example Scenario**

👉 You have 3 servers:

* Server A (Primary)  
* Server B (Secondary)  
* Server C (Secondary)

If Server A crashes ❌  
👉 Server B becomes new Primary automatically ✅

---

# **🔹 Why Replication is Useful? (Very Important 🔥)**

### **✅ 1\. High Availability**

* System never goes down  
* Automatic failover

---

### **✅ 2\. Data Backup**

* Copies exist on multiple servers  
* Prevents data loss

---

### **✅ 3\. Read Scalability**

* Read from secondary nodes  
* Improves performance

---

### **✅ 4\. Disaster Recovery**

* If one server crashes → data safe

---

# **🔹 Read Preference (Advanced 🔥)**

You can control reads:

* `primary` → strong consistency  
* `secondary` → faster reads

---

# **🔴 Important Points**

❌ Writes only happen on primary  
❌ Replication is asynchronous (slight delay possible)

---

# **🔥 Real-World Use Case**

* Banking systems 🏦  
* E-commerce 🛒  
* Social media 📱

👉 Anywhere downtime is not acceptable

"Replication in MongoDB uses replica sets to maintain multiple copies of data, ensuring high availability, fault tolerance, and read scalability."

# **🔹 What is Sharding?**

### **👉 Definition**

Sharding is the process of **splitting large data across multiple servers**.

---

# **🔥 Simple Understanding**

👉 Instead of storing all data in one server:

* You **divide data into smaller chunks**  
* Store each chunk on different servers

---

# **🔹 Real-Life Analogy 🧠**

Imagine:

* One library 📚 (too many books → slow)  
* You split books into:  
  * Library A → Science  
  * Library B → History  
  * Library C → Tech

👉 This is **sharding**

---

# **🔹 Why Do We Need Sharding?**

When:

* Data becomes very large 📈  
* Single server can't handle load ❌

👉 Sharding helps to:

* Scale horizontally (add more servers)

---

# **🔹 Components of Sharded Cluster**

### **1\. Shards 🟢**

* Store actual data  
* Each shard \= part of dataset

---

### **2\. Config Server 🔵**

* Stores metadata (which data is where)

---

### **3\. Query Router (`mongos`) 🟡**

* Receives client requests  
* Sends query to correct shard

---

# **🔹 How It Works**

1. Data is divided using a **shard key**  
2. MongoDB distributes data across shards  
3. Query router sends request to correct shard  
4. Results are combined and returned

---

# **🔥 Visual Flow**

Client → mongos → Shard1  
                → Shard2  
                → Shard3  
---

# **🔹 What is Shard Key? (Very Important 🔥)**

👉 A field used to **distribute data**

Example:

{ userId: 1 }

👉 MongoDB uses this to decide:

* Which shard stores which data

---

# **🔹 Types of Sharding**

### **1\. Range-Based Sharding**

* Data split by ranges

👉 Example:

* 1–1000 → Shard1  
* 1001–2000 → Shard2

---

### **2\. Hash-Based Sharding**

* Data distributed randomly using hash

👉 Better distribution (avoids hotspots)

---

# **🔹 Example**

### **📌 Data:**

{ userId: 101, name: "vikash" }

👉 Based on shard key (`userId`):

* Stored in a specific shard

---

# **🔹 Why Sharding is Useful? (Very Important 🔥)**

### **✅ 1\. Horizontal Scaling**

* Add more servers easily

---

### **✅ 2\. Handle Big Data**

* Supports huge datasets

---

### **✅ 3\. Better Performance**

* Queries run on smaller datasets

---

### **✅ 4\. Load Distribution**

* Traffic distributed across servers

---

# **🔴 Challenges**

❌ Choosing wrong shard key → bad performance  
❌ Complex queries across shards  
❌ More system complexity

---

# **🔹 Sharding vs Replication**

| Feature | Sharding | Replication |
| ----- | ----- | ----- |
| Purpose | Scale data | Backup data |
| Data | Split | Same copy |
| Goal | Performance | Availability |

---

# **🔥 Real-World Use Cases**

* E-commerce (millions of users)  
* Social media apps  
* Big data platforms

---

# **🔥 Interview Tip**

Say this:

"Sharding distributes data across multiple servers using a shard key to achieve horizontal scalability and handle large datasets efficiently."

---

# **🔹 Memory Trick 🧠**

* Sharding \= **split data**  
* Replication \= **copy data**
