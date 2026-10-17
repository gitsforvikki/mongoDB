# Lesson 033 — TTL and Specialized Indexes

[← Section 4](./README.md) · [Full curriculum](../README.md)

## Goal
Learn when MongoDB should automatically expire documents and recognize specialized index types.

## 1. TTL index: expire old documents
A temporary verification code can expire after 15 minutes.

```js
use mongoCourse
db.verificationCodes.insertOne({
 email:"asha@example.test",
 codeHash:"example-hash-not-a-real-code",
 createdAt:new Date()
})
db.verificationCodes.createIndex(
 {createdAt:1},
 {expireAfterSeconds:900}
)
```
TTL removal is **asynchronous**; expired documents are not necessarily deleted at the exact second. Application logic must still check expiry before accepting a code. TTL indexes have restrictions: typically single-field date indexes, not general compound indexes.

## 2. Per-document expiration
```js
db.sessionsTTL.insertOne({
 tokenHash:"example-token-hash",
 expiresAt:new Date(Date.now()+3600_000)
})
db.sessionsTTL.createIndex({expiresAt:1},{expireAfterSeconds:0})
```
With `expireAfterSeconds:0`, each document's date controls when it becomes eligible for cleanup. Still check expiration in auth logic.

## 3. Hashed indexes
```js
db.hashUsers.createIndex({userId:"hashed"})
```
Hashed indexes index a hash of a field value, useful in some hashed-sharding strategies and equality-style lookups. They do not preserve original value order for range queries.

## 4. Geospatial indexes
```js
db.geoStores.insertOne({
 name:"Store A",
 location:{type:"Point",coordinates:[77.5946,12.9716]}
})
db.geoStores.createIndex({location:"2dsphere"})
db.geoStores.find({
 location:{$near:{$geometry:{type:"Point",coordinates:[77.60,12.97]},$maxDistance:5000}}
})
```
GeoJSON point coordinates are **[longitude, latitude]**, not the other way around. `$maxDistance` for GeoJSON near queries is in meters.

## 5. Wildcard indexes
```js
db.dynamicCatalog.createIndex({"attributes.$**":1})
```
Wildcard indexes can help with varying attribute keys such as `attributes.color` and `attributes.size`. They have limitations and can cost storage/write performance; don't use them instead of known targeted indexes by default.

## 6. Mongoose example
```js
sessionSchema.index({expiresAt:1},{expireAfterSeconds:0});
storeSchema.index({location:"2dsphere"});
```

## Practice
Design TTL for password reset requests and explain why TTL cleanup alone cannot enforce token expiry. Describe when you'd choose hashed, 2dsphere, or wildcard indexes.

**Interview:** **Is TTL immediate?** No, background cleanup is eventual.

[TTL](https://www.mongodb.com/docs/manual/core/index-ttl/) · [Geospatial](https://www.mongodb.com/docs/manual/geospatial-queries/)
