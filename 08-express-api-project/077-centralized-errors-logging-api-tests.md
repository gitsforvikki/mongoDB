# Lesson 77 — Centralized Errors, Logging and API Tests

[← Section 8](./README.md) · [Full curriculum](../README.md)

## 1. Why the final layer matters

A successful request is only half of a production API. Real clients send invalid input, expired tokens and nonexistent IDs; databases may disconnect or reject duplicate keys; services can throw unexpected errors. The API needs **consistent errors**, useful logs and repeatable tests.

Express 5 forwards errors from rejected async route handlers automatically, but it does not decide the correct HTTP status or response shape. That is our job.

## 2. Expected vs unexpected errors

**Expected operational errors** include malformed input, missing resources, duplicate SKU, expired sessions and forbidden operations. These should become predictable 4xx responses. **Unexpected errors** include programming bugs, broken invariants and unavailable dependencies. They should be logged for investigation while clients receive a safe generic 5xx response.

## 3. AppError for intentional failures

```js
// src/utils/app-error.js
export class AppError extends Error {
  constructor(message, statusCode = 500, code = "INTERNAL_ERROR") {
    super(message);
    this.name = "AppError";
    this.statusCode = statusCode;
    this.code = code;
  }
}
```

Usage in a service:

```js
if (!product) {
  throw new AppError("Product not found", 404, "PRODUCT_NOT_FOUND");
}
```

Keep business services independent of Express `res`; a service can throw a domain-specific error and an HTTP boundary can translate it.

## 4. Centralized error middleware

```js
// src/middlewares/error.middleware.js
import mongoose from "mongoose";
import { AppError } from "../utils/app-error.js";

export function errorHandler(err, req, res, next) {
  if (res.headersSent) return next(err);

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: { code: err.code, message: err.message }
    });
  }

  if (err instanceof mongoose.Error.CastError) {
    return res.status(400).json({
      success: false,
      error: { code: "INVALID_ID_OR_VALUE", message: "Invalid input value" }
    });
  }

  if (err instanceof mongoose.Error.ValidationError) {
    return res.status(400).json({
      success: false,
      error: {
        code: "VALIDATION_FAILED",
        message: "Validation failed",
        fields: Object.entries(err.errors).map(([field, issue]) => ({
          field, message: issue.message
        }))
      }
    });
  }

  if (err?.code === 11000) {
    return res.status(409).json({
      success: false,
      error: { code: "DUPLICATE_RESOURCE", message: "Resource already exists" }
    });
  }

  console.error("Unexpected request error", {
    method: req.method, path: req.path, error: err
  });

  return res.status(500).json({
    success: false,
    error: { code: "INTERNAL_ERROR", message: "Internal server error" }
  });
}
```

In production, review validation messages before exposing them; custom validators may include sensitive details. Do not log passwords, JWTs, cookie values, full database URIs or private request bodies. Some database outages may deserve 503 rather than 500 under an explicit availability policy.

## 5. Express 5 async error propagation

```js
app.get("/demo", async (req, res) => {
  throw new AppError("Example missing", 404, "NOT_FOUND");
});
// Express 5 forwards the rejection to error middleware.
```

You generally do not need a `try/catch` around every async controller just to call `next(error)`. However, Express cannot automatically catch errors thrown later in unrelated timers or callbacks that are outside the returned Promise chain; handle those explicitly.

Place `app.use(errorHandler)` **after** all routes. An Express error handler has four arguments `(err,req,res,next)` even when `next` is not used.

## 6. Request logging and correlation IDs

Logs should help answer: which endpoint failed, when, for which request, and why? Use a structured logger such as Pino in production. Include a generated request ID to correlate application and downstream logs, while avoiding personal data.

```js
import { randomUUID } from "node:crypto";

app.use((req, res, next) => {
  req.requestId = randomUUID();
  res.setHeader("X-Request-ID", req.requestId);
  next();
});
```

A reverse proxy may supply a request ID; only trust it under a controlled boundary. Metrics such as request latency, error rate and DB query duration complement logs.

## 7. API tests with Supertest

```bash
npm install -D supertest
```

Supertest can send HTTP-like requests to an imported Express `app` without binding a production port.

```js
// tests/health.test.js
import test from "node:test";
import assert from "node:assert/strict";
import request from "supertest";
import { app } from "../src/app.js";

test("unknown route is not successful", async () => {
  const res = await request(app).get("/does-not-exist");
  assert.equal(res.status, 404);
});
```

Add `"test": "node --test"` to package scripts. For MongoDB-dependent integration tests, use an isolated test database or a properly configured MongoDB memory server/replica set, especially for transaction tests. Never point destructive tests at production.

## 8. Integration test for product validation

```js
import test from "node:test";
import assert from "node:assert/strict";
import request from "supertest";
import { app } from "../src/app.js";

test("rejects invalid product input", async () => {
  const response = await request(app)
    .post("/api/products")
    .send({ name: "A", priceMinor: -1, sku: "BAD SKU", stock: -5 });

  assert.equal(response.status, 400);
});
```

This example assumes the product route and validation middleware from Lesson 070 are registered. When Lesson 073 admin authorization is active, authenticate as an admin before testing validation, or the request will correctly fail at the authorization layer first.

## 9. What to test beyond happy paths

| Scenario | Expected behavior |
| --- | --- |
| Invalid ObjectId | 400 |
| Valid missing ObjectId | 404 |
| Duplicate SKU | 409 after index exists |
| Missing authentication | 401 |
| USER attempts admin action | 403 |
| User A requests User B order | 404 or policy-defined denial |
| Stale version update | 409/412 when version checks implemented |
| Invalid stock reservation | No negative stock |
| Database unavailable | Controlled failure; no secret leakage |

Test concurrent checkout behavior and transaction rollback with a real replica-set-capable test environment. Unit tests are useful for service logic; integration tests verify routing, middleware, validation, Mongoose and MongoDB behavior together.

## 10. Shutdown and unhandled failures

Handle process termination gracefully, close the HTTP listener, and disconnect Mongoose. Log unhandled rejections and uncaught exceptions using a controlled fatal-error policy; do not continue serving indefinitely after an unknown corrupted application state. Use your deployment's process manager to restart safely.

## 11. Final Section 8 exercise

Build the full Shop API: user registration/login, admin product CRUD, cart updates, guarded checkout, user-scoped order history, paginated product browsing and an admin dashboard. Document the API routes and provide tests for invalid input, ownership, duplicate keys, transaction rollback and authentication. Explain the complete request path from browser → Express middleware → controller → service → Mongoose → MongoDB → HTTP response.

## 12. Interview questions

**Does Express 5 automatically map a duplicate-key error to 409?** No. **Why keep error responses generic for 500s?** Prevent internal detail leakage. **What is the benefit of Supertest?** Test HTTP behavior without manually starting a listening server. **Why integration tests?** They verify interactions across layers.

**References:** [Express error handling](https://expressjs.com/en/guide/error-handling.html) · [Node.js test runner](https://nodejs.org/api/test.html) · [Supertest](https://github.com/ladjs/supertest)
