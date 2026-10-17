# Lesson 043 — Dates, Strings and Conditional Expressions

[← Section 5](./README.md) · [Full curriculum](../README.md)

## Goal
Turn raw timestamps and values into useful report dimensions such as **month**, **display label**, and **revenue category**. Understand date types and time zones.

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


## 1. Group orders by month
```js
db.aggOrders.aggregate([
 {$project:{
  status:1,
  month:{$dateToString:{format:"%Y-%m",date:"$createdAt",timezone:"UTC"}}
 }},
 {$group:{_id:"$month",orders:{$sum:1}}},
 {$sort:{_id:1}}
])
```
Expected January → 2 orders; February → 2. The `timezone` choice affects which month a timestamp belongs to near midnight.

## 2. $dateTrunc: bucket by calendar month
```js
db.aggOrders.aggregate([
 {$group:{
  _id:{$dateTrunc:{date:"$createdAt",unit:"month",timezone:"UTC"}},
  count:{$sum:1}
 }},
 {$sort:{_id:1}}
])
```
`$dateTrunc` returns date values representing the start of each bucket, useful for time-series grouping (requires a MongoDB version that supports it).

## 3. String expressions
```js
db.aggCustomers.aggregate([
 {$project:{
  _id:0,
  displayName:{$toUpper:"$name"},
  label:{$concat:["$name"," (","$city",")"]}
 }}
])
```
Expected for u1: `ASHA` and `Asha (Patna)`. `$concat` may return null if an input is null/missing; use `$ifNull` for fallbacks.

## 4. $cond and $switch
```js
db.aggOrders.aggregate([
 {$project:{
  _id:1,
  paymentLabel:{$cond:[{$eq:["$status","PAID"]},"Paid","Not paid"]},
  category:{$switch:{
   branches:[
    {case:{$eq:["$status","PAID"]},then:"Revenue candidate"},
    {case:{$eq:["$status","CANCELLED"]},then:"Exclude"}
   ],
   default:"Review"
  }}
 }}
])
```
`$cond` is simple if/else. `$switch` is useful for multiple ordered cases.

## 5. Type conversion
```js
db.aggCustomers.aggregate([
 {$project:{name:1,idAsText:{$toString:"$_id"}}}
])
```
Use conversion operators when a report needs a different type. For untrusted or inconsistent values, `$convert` with `onError` and `onNull` can avoid pipeline failures; **do not silently convert bad monetary data to zero** without a clear policy.

## 6. Time zone pitfalls
A timestamp is stored as a date/time value, not a formatted local date. Use an explicit business timezone when computing daily or monthly metrics. For example, a late-night UTC order may fall on the next day in Asia/Kolkata.

## Mongoose
```js
const monthly = await Order.aggregate([
 {$group:{
  _id:{$dateToString:{format:"%Y-%m",date:"$createdAt",timezone:"Asia/Kolkata"}},
  orders:{$sum:1}
 }}
]);
```

## Practice
Produce a report of paid orders by month, uppercase customer names, and status labels. Explain why date bucketing needs a timezone decision.

**Interview:** **Why not store formatted date strings?** BSON Date supports proper date operations; format at query/report boundaries.

[Date expressions](https://www.mongodb.com/docs/manual/reference/operator/aggregation/dateToString/) · [Conditional expressions](https://www.mongodb.com/docs/manual/reference/operator/aggregation/cond/)
