# Lesson 019 — Regex, Text and Search Options

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why search?
Customers type partial product names, look for keywords in descriptions, or request relevant search results. Different search methods serve different needs.

## Dataset
```js
use mongoCourse
db.searchDemo.insertMany([
 {_id:1,name:"Wireless Mouse",description:"Ergonomic bluetooth mouse"},
 {_id:2,name:"Gaming Keyboard",description:"Mechanical RGB keyboard"},
 {_id:3,name:"Mouse Pad",description:"Large desk accessory"},
 {_id:4,name:"USB Cable",description:"Fast charging cable"}
])
```

## 1. Regex: match text patterns
```js
db.searchDemo.find({name:{$regex:"Mouse"}}) // Wireless Mouse, Mouse Pad
db.searchDemo.find({name:{$regex:"^Mouse"}}) // Mouse Pad
db.searchDemo.find({name:{$regex:"mouse",$options:"i"}}) // case-insensitive
```
`^Mouse` means the name **starts** with Mouse; `i` makes regex matching case-insensitive. Regex is not automatically a full-text search engine.

**Performance:** Anchored, case-sensitive prefix regex may benefit from an appropriate index. Unanchored patterns and case-insensitive regex can be expensive. Escape or validate user-provided patterns and cap their length; arbitrary regex can cause resource exhaustion.

## 2. Basic MongoDB text index and $text
```js
db.searchDemo.createIndex({name:"text",description:"text"})
db.searchDemo.find({$text:{$search:"keyboard"}})
db.searchDemo.find(
 {$text:{$search:"mouse"}},
 {name:1,score:{$meta:"textScore"}}
).sort({score:{$meta:"textScore"}})
```
A text index enables the classic `$text` query. It uses tokenization and language rules; it is **not** substring matching. Search relevance scores can help rank matches. Index rules and supported language behavior matter.

## 3. Atlas Search is a different capability
**MongoDB Search (Atlas Search)** supports richer search features such as autocomplete, analyzers, fuzzy matching, and ranking through a dedicated search index and aggregation stages such as `$search`. Availability and setup depend on your Atlas deployment and tier. A classic text index is not the same as an Atlas Search index.

Conceptual pipeline (requires a configured Atlas Search index):
```js
db.products.aggregate([
 {$search:{index:"productSearch",text:{query:"wireless mouse",path:["name","description"]}}},
 {$limit:10},
 {$project:{name:1,score:{$meta:"searchScore"}}}
])
```
This **will not run** on the local `searchDemo` collection without an appropriate supported deployment and search index.

## 4. Mongoose equivalents
```js
const matching = await Product.find({name:{$regex:"^Mouse"}});
const fullText = await Product.find({$text:{$search:"keyboard"}});
```
Mongoose does not automatically create a search engine; the correct MongoDB indexes/features must exist.

## Choosing the right approach
- **Exact value:** equality query with a suitable index.
- **Starts-with:** anchored regex if its limitations fit.
- **Keywords:** classic `$text` with a text index.
- **Autocomplete, fuzzy/relevance-heavy search:** MongoDB Search or another dedicated search solution.

## Common mistakes
- Expecting `$text` to match arbitrary substrings.
- Expecting regex `/mouse/i` to scale like indexed exact equality.
- Passing unchecked user input directly as regex.
- Assuming Atlas Search works without creating its own search index.

## Practice
Search names containing Mouse; search names beginning Mouse; create a text index and search keyboard; explain when autocomplete requires a different tool.

## Interview check
**Regex vs text search?** Pattern matching vs tokenized keyword search. **Why index?** Avoid expensive collection scans when supported by the query shape.

[Regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/) · [Text search](https://www.mongodb.com/docs/manual/core/text-search/) · [MongoDB Search](https://www.mongodb.com/docs/atlas/atlas-search/)
