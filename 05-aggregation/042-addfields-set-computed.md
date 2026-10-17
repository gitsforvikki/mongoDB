# Lesson 042 — $addFields, $set and Computed Values

[← Section 5](./README.md) · [Full curriculum](../README.md)

## Why computed fields?
Your database stores quantity and unit price, but a report needs **line total**, **order total**, and a readable status label. Aggregation expressions calculate values while the pipeline runs.

## Shared practice dataset
Run this once in **mongosh** on a disposable learning database. All lessons use these same IDs and amounts, so you can follow the data from stage to stage.

```js
use mongoCourse
db.aggCustomers.deleteMany({_id:{$in:["u1","u2","u3"]}})
db.aggProducts.deleteMany({_id:{$in:["p1","p2","p3"]}})
db.aggOrders.deleteMany({_id:{$in:["o1","o2","o3","o4"]}})
db.aggCustomers.insertMany([
 {_id:"u1",name:"Asha",city:"Patna"},
 {_id:"u2",name:"Ravi",city:"Pune"},
 {_id:"u3",name:"Neha",city:"Patna"}
])
db.aggProducts.insertMany([
 {_id:"p1",name:"Mouse",category:"Accessories"},
 {_id:"p2",name:"Keyboard",category:"Accessories"},
 {_id:"p3",name:"Monitor",category:"Electronics"}
])
db.aggOrders.insertMany([
 {_id:"o1",customerId:"u1",status:"PAID",createdAt:ISODate("2026-01-10T10:00:00Z"),items:[{productId:"p1",qty:2,unitPrice:500},{productId:"p2",qty:1,unitPrice:1500}]},
 {_id:"o2",customerId:"u2",status:"PAID",createdAt:ISODate("2026-01-20T10:00:00Z"),items:[{productId:"p1",qty:1,unitPrice:500}]},
 {_id:"o3",customerId:"u1",status:"CANCELLED",createdAt:ISODate("2026-02-03T10:00:00Z"),items:[{productId:"p3",qty:1,unitPrice:12000}]},
 {_id:"o4",customerId:"u3",status:"PAID",createdAt:ISODate("2026-02-15T10:00:00Z"),items:[{productId:"p3",qty:1,unitPrice:12000}]}
])
```
Prices here are **illustrative whole currency units** for easy arithmetic. In a production shop, store money consistently as integer minor units or Decimal128 and handle currency and refunds explicitly.


## 1. $addFields and $set are aliases
```js
db.aggOrders.aggregate([
 {$match:{_id:"o1"}},
 {$addFields:{itemCount:{$size:"$items"}}}
])
```
This retains original fields and adds `itemCount:2`. The aggregation stage `$set` is an alias for `$addFields`—**different** from the update operator `$set` used by `updateOne()`.

## 2. Calculate each line total with $map
```js
db.aggOrders.aggregate([
 {$match:{_id:"o1"}},
 {$set:{
  lineTotals:{$map:{
   input:"$items",as:"item",
   in:{$multiply:["$$item.qty","$$item.unitPrice"]}
  }}
 }}
])
```
Expected `lineTotals:[1000,1500]`. `$map` processes each array element; `$$item` refers to the current item.

## 3. Sum the calculated array
```js
db.aggOrders.aggregate([
 {$match:{_id:"o1"}},
 {$set:{lineTotals:{$map:{
  input:"$items",as:"item",
  in:{$multiply:["$$item.qty","$$item.unitPrice"]}
 }}}},
 {$set:{orderTotal:{$sum:"$lineTotals"}}},
 {$project:{_id:1,lineTotals:1,orderTotal:1}}
])
```
Expected `{_id:"o1",lineTotals:[1000,1500],orderTotal:2500}`. This example intentionally uses **two $set stages** because the second references the field computed by the first.

## 4. Conditional computed fields
```js
db.aggOrders.aggregate([
 {$set:{statusLabel:{$cond:{
  if:{$eq:["$status","PAID"]},
  then:"Completed payment",
  else:"Not paid"
 }}}},
 {$project:{_id:1,statusLabel:1}}
])
```
`$cond` behaves like if/else for each document.

## 5. $project vs $addFields
- `$project`: control the **output shape**, include/exclude/compute.
- `$addFields` / aggregation `$set`: preserve existing fields while adding/replacing specified ones.
- `$unset` aggregation stage: remove named fields.

## 6. Handling missing data
```js
db.aggOrders.aggregate([
 {$set:{discount:{$ifNull:["$discount",0]}}}
])
```
`$ifNull` supplies a fallback for null/missing values. Validate types before arithmetic: a string price is not a trustworthy numeric amount.

## Mongoose
```js
const totals = await Order.aggregate([
 {$set:{itemCount:{$size:"$items"}}},
 {$project:{customerId:1,itemCount:1}}
]);
```

## Practice
Calculate itemCount for every order, line totals for o4, and a `isPaid` boolean using `$eq`. Predict the result before running it.

**Interview:** **Difference between $project and $addFields?** Reshape fields vs add/overwrite while retaining others.

[$addFields](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addFields/) · [$map](https://www.mongodb.com/docs/manual/reference/operator/aggregation/map/)
