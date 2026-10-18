# Lesson 020 — Pagination: Offset vs Cursor

[← Section 2](./README.md) · [Full curriculum](../README.md)

## Why paginate?
Returning 100,000 products in one API response wastes bandwidth and memory. Pagination returns a small page of results.

## Dataset
```js
use mongoCourse
db.paginationDemo.insertMany([
 {_id:1,name:"A",price:100},
 {_id:2,name:"B",price:100},
 {_id:3,name:"C",price:200},
 {_id:4,name:"D",price:300},
 {_id:5,name:"E",price:300},
 {_id:6,name:"F",price:400}
])
```

## 1. Offset pagination: skip and limit
```js
db.paginationDemo.find().sort({price:1,_id:1}).skip(0).limit(2) // A, B
db.paginationDemo.find().sort({price:1,_id:1}).skip(2).limit(2) // C, D
```
Formula: `skip = (page - 1) * pageSize`.

**Advantages:** Easy page numbers and jumping to a specific page.

**Problems:** Large offsets require MongoDB to advance past many results; inserts/deletes between requests can shift page boundaries and cause duplicates or missed records.

## 2. Cursor/keyset pagination: continue after last item
Use a stable sort on **price, then unique _id**:
```js
db.paginationDemo.find().sort({price:1,_id:1}).limit(2) // A, B
```
The last record has `price:100, _id:2`. The next page should include documents **after that pair**:

```js
db.paginationDemo.find({
 $or:[
  {price:{$gt:100}},
  {price:100,_id:{$gt:2}}
 ]
}).sort({price:1,_id:1}).limit(2) // C, D
```

If the last record on page 2 is D (`price:300,_id:4`), continue with `price > 300 OR (price == 300 AND _id > 4)`. The tie-breaker prevents skipping records with equal prices.

## 3. Why indexes matter
```js
db.paginationDemo.createIndex({price:1,_id:1})
```
A compound index matching filter/sort patterns can make keyset pagination efficient. Inspect `explain("executionStats")` to validate actual query behavior.

## 4. Mongoose example
```js
const pageSize = 2;
const lastPrice = 100;
const lastId = 2; // this example uses numeric _id; real apps often use ObjectId
const products = await Product.find({
 $or:[
  {price:{$gt:lastPrice}},
  {price:lastPrice,_id:{$gt:lastId}}
 ]
}).sort({price:1,_id:1}).limit(pageSize).lean();
```
In a real API, use the schema's actual ID type, validate cursor input, and encode cursor values into an opaque token rather than trusting raw query parameters.

### 4.1 Where do `lastPrice` and `lastId` come from?

**Simple answer:** They come from the **last product returned on the previous page**. We do **not** store them separately in MongoDB.

Example sorted data (page size = 2):

| _id | Product | Price |
| --- | --- | --- |
| 1 | Mouse | 100 |
| 2 | Keyboard | 100 |
| 3 | Monitor | 200 |
| 4 | Laptop | 300 |
| 5 | Webcam | 400 |

1. First request: no cursor → return Mouse and Keyboard.
2. The **last product is Keyboard** → `lastPrice = 100`, `lastId = 2`.
3. Backend encodes these two values into a `nextCursor` and returns it.
4. Frontend saves that token in React state and sends it on **Load More**.
5. Backend decodes it and queries for products **after** that pair.
6. Page 2 returns Monitor and Laptop; its last product becomes the next cursor.

```mermaid
sequenceDiagram
    participant UI as React Frontend
    participant API as Express + Mongoose
    participant DB as MongoDB
    UI->>API: GET /api/products (no cursor)
    API->>DB: Sort by price, _id; limit 3
    DB-->>API: Mouse, Keyboard, Monitor
    API-->>UI: [Mouse, Keyboard] + nextCursor(100, 2)
    UI->>API: GET /api/products?cursor=...
    API->>DB: Fetch after (100, 2); limit 3
    DB-->>API: Monitor, Laptop, Webcam
    API-->>UI: [Monitor, Laptop] + nextCursor(300, 4)
```

### 4.2 Express + Mongoose implementation (numeric `_id` demo)

The following example uses **numeric `_id` values to match this lesson's dataset**. In typical Mongoose apps, `_id` is an ObjectId; convert and validate the decoded ID according to your actual schema.

```js
import express from "express";
import { Product } from "./models/Product.js";

const app = express();

app.get("/api/products", async (req, res) => {
  const pageSize = 2;
  let cursor = null;

  if (req.query.cursor !== undefined) {
    try {
      if (typeof req.query.cursor !== "string" ||
          req.query.cursor.length > 1024) {
        return res.status(400).json({ message: "Invalid cursor" });
      }

      cursor = JSON.parse(
        Buffer.from(req.query.cursor, "base64url").toString("utf8")
      );

      if (!cursor || typeof cursor !== "object" ||
          !Number.isFinite(cursor.lastPrice) ||
          !Number.isSafeInteger(cursor.lastId)) {
        return res.status(400).json({ message: "Invalid cursor values" });
      }
    } catch {
      return res.status(400).json({ message: "Malformed cursor" });
    }
  }

  const filter = cursor
    ? {
        $or: [
          { price: { $gt: cursor.lastPrice } },
          { price: cursor.lastPrice, _id: { $gt: cursor.lastId } }
        ]
      }
    : {};

  // Fetch one extra document to detect whether another page exists.
  const rows = await Product.find(filter)
    .sort({ price: 1, _id: 1 })
    .limit(pageSize + 1)
    .lean();

  const hasMore = rows.length > pageSize;
  const data = rows.slice(0, pageSize);
  const lastProduct = data.at(-1);

  const nextCursor = hasMore && lastProduct
    ? Buffer.from(JSON.stringify({
        lastPrice: lastProduct.price,
        lastId: lastProduct._id
      })).toString("base64url")
    : null;

  return res.json({ data, nextCursor, hasMore });
});
```

**Why `pageSize + 1`?** If you want two products, fetch three. The third tells you that a further page exists, but return only the first two.

**Production note:** Base64url makes a cursor opaque-looking, **not secure or tamper-proof**. Validate the entire payload and consider HMAC-signed cursors; apply the same search filters on every page. Use a suitable compound index and stable sort keys.

### 4.3 React frontend: maintain the cursor in state

```jsx
import { useState } from "react";

export default function Products() {
  const [products, setProducts] = useState([]);
  const [cursor, setCursor] = useState(null);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);

  async function loadMore() {
    if (loading || !hasMore) return;
    setLoading(true);
    try {
      const url = cursor
        ? `/api/products?cursor=${encodeURIComponent(cursor)}`
        : "/api/products";

      const response = await fetch(url);
      if (!response.ok) throw new Error("Failed to load products");

      const result = await response.json();
      setProducts(prev => [...prev, ...result.data]);
      setCursor(result.nextCursor);
      setHasMore(result.hasMore);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      {products.map(product => (
        <p key={product._id}>{product.name} - ₹{product.price}</p>
      ))}
      <button disabled={loading || !hasMore} onClick={loadMore}>
        {loading ? "Loading..." : hasMore ? "Load More" : "No more products"}
      </button>
    </div>
  );
}
```

The frontend **does not need to calculate** `lastPrice` or `lastId`. It only stores the backend's `nextCursor` and sends it with the next request. Reset the products and cursor when changing sorting or filters.

## 5. API response design
```json
{
 "data":[{"_id":3,"name":"C","price":200},{"_id":4,"name":"D","price":300}],
 "nextCursor":"opaque-token-for-price-300-id-4",
 "hasMore":true
}
```
The cursor string above is illustrative, not a working token. To determine `hasMore`, fetch `pageSize + 1` items and return only `pageSize`; encode the last returned item's sort keys. Cursor tokens should be validated and, where appropriate, signed.

## 6. Which should you choose?
| Requirement | Offset | Cursor/keyset |
| --- | --- | --- |
| Jump to page 20 | Easy | Not directly |
| Deep pages | Often expensive | Usually more efficient with index |
| Frequent inserts/deletes | Pages can shift | More stable continuation |
| Implementation | Simpler | Requires stable sort and cursor |
| Common use | Small admin tables | Infinite scroll, feeds, large datasets |

**Important:** Cursor pagination is not a snapshot. If documents change sort keys or are deleted between requests, results can still shift. Stable immutable sort keys are often preferable for feeds.

## 7. Common mistakes
- Sorting only by nonunique price and losing stable ordering.
- Using `skip(100000)` without considering performance.
- Forgetting the same filter and sort must apply across pages.
- Treating client-supplied cursors as trusted database filters.
- Assuming cursor pagination always prevents duplicates under concurrent edits.

## Practice
Fetch three pages of two products with offset pagination. Then repeat using the last `price,_id` pair. Explain why `_id` is needed when prices tie.

## Interview check
**Why keyset pagination?** Efficient continuation without deep skipping. **Why unique tie-breaker?** Stable order. **Is it perfectly consistent under updates?** No, not automatically.

[Cursor skip](https://www.mongodb.com/docs/manual/reference/method/cursor.skip/) · [Compound indexes](https://www.mongodb.com/docs/manual/core/indexes/index-types/index-compound/)
