# Lesson 011 — Comparison Query Operators

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why comparison operators?
In real applications we filter by price ranges, age, stock, status, and membership rather than reading everything. MongoDB query operators begin with `$`.

## Dataset
```js
use mongoCourse
db.comparisonDemo.insertMany([
 {_id:1,name:"Pen",price:20,stock:100,status:"active"},
 {_id:2,name:"Notebook",price:80,stock:30,status:"active"},
 {_id:3,name:"Bag",price:900,stock:0,status:"inactive"},
 {_id:4,name:"Bottle",price:250,stock:15,status:"active"}
])
```

## Operators with results
| Operator | Meaning | Example |
| --- | --- | --- |
| `$eq` | Equal | `{price:{$eq:80}}` → Notebook |
| `$ne` | Not equal | `{status:{$ne:"inactive"}}` → Pen, Notebook, Bottle |
| `$gt` | Greater than | `{price:{$gt:250}}` → Bag |
| `$gte` | Greater or equal | `{price:{$gte:250}}` → Bag, Bottle |
| `$lt` | Less than | `{price:{$lt:80}}` → Pen |
| `$lte` | Less or equal | `{price:{$lte:80}}` → Pen, Notebook |
| `$in` | In a list | `{price:{$in:[20,250]}}` → Pen, Bottle |
| `$nin` | Not in a list | `{status:{$nin:["inactive"]}}` → Pen, Notebook, Bottle |

```js
db.comparisonDemo.find({price:{$gte:80,$lte:250}})
db.comparisonDemo.find({status:{$in:["active","pending"]}})
db.comparisonDemo.find({stock:{$gt:0}})
```
The first query returns Notebook and Bottle: both constraints on `price` must hold.

## Important subtleties
- `{price:80}` is shorthand for `{price:{$eq:80}}`.
- `$ne` and `$nin` can also match documents where the field is **missing**. Add `$exists:true` if needed.
- Comparisons are type-aware (MongoDB's **type bracketing**); a string `"80"` is not the same as number `80`.
- For arrays, membership and comparisons have additional element-wise behavior; study Lesson 013.
- A range query is often faster with a suitable index; query plans matter.

## Mongoose comparison
```js
const items = await Product.find({
  price:{$gte:80,$lte:250},
  status:{$in:["active","pending"]}
});
```
The `$gte`, `$lte`, and `$in` operators come from **MongoDB**, not Mongoose.

## Practice
Find prices above 100; items with zero stock; all active items; products outside the prices 20 and 80.

## Interview checks
**$gt vs $gte?** Exclusive versus inclusive boundary. **$in vs $nin?** Membership versus non-membership. **Why care about field types?** They change match behavior.

[Official comparison operators](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/comparison/)
