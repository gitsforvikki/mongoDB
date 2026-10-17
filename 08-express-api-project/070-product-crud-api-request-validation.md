# Lesson 70 — Product CRUD API with Request Validation

[← Section 8](./README.md) · [Full curriculum](../README.md)

## 1. What makes a CRUD API production-ready?

CRUD means **Create, Read, Update and Delete**. But a production API also needs stable routes, input validation, safe update fields, meaningful HTTP statuses, pagination, concurrency awareness, and consistent errors. A Mongoose schema is not a replacement for validating the public HTTP contract.

We continue the Shop API from Lessons 068–069.

## 2. Define a Product model

```js
// src/models/product.model.js
import mongoose from "mongoose";

const productSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true, minlength: 2 },
  sku: { type: String, required: true, trim: true, uppercase: true },
  priceMinor: { type: Number, required: true, min: 0 },
  stock: { type: Number, required: true, min: 0, default: 0 },
  active: { type: Boolean, default: true }
}, { timestamps: true });

productSchema.index({ sku: 1 }, { unique: true });
export const Product = mongoose.model("Product", productSchema);
```

Store money in **integer minor units** (for example 49900 paise = ₹499). `unique` defines a MongoDB index, not a Mongoose validator. Do not rely on Mongoose `min:0` alone to prevent negative stock under concurrent `$inc` updates.

## 3. Define separate request schemas

```js
// src/validators/product.validator.js
import { z } from "zod";

export const createProductSchema = z.strictObject({
  name: z.string().trim().min(2).max(100),
  sku: z.string().trim().toUpperCase().regex(/^[A-Z0-9-]+$/),
  priceMinor: z.number().int().nonnegative(),
  stock: z.number().int().nonnegative().default(0)
});

export const updateProductSchema = createProductSchema
  .partial()
  .refine(data => Object.keys(data).length > 0, "At least one field required");

export const productIdSchema = z.string().regex(/^[a-fA-F0-9]{24}$/);
```

**Why separate create and update?** Creation requires core fields; PATCH should allow a subset but reject an empty body. `strictObject` rejects unexpected fields such as `active` or `role`, reducing accidental mass assignment. The ObjectId format check only validates shape, not whether a document exists.

## 4. Reusable validation middleware

```js
// src/middlewares/validate.middleware.js
export const validateBody = schema => (req, res, next) => {
  const parsed = schema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({
      success: false, message: "Invalid request body",
      issues: parsed.error.issues.map(issue => ({
        path: issue.path.join("."), message: issue.message
      }))
    });
  }
  req.validatedBody = parsed.data;
  next();
};
```

Parsing into `req.validatedBody` ensures controllers use the **validated and transformed** values rather than raw input. This pattern is one possible architecture; TypeScript projects may type this property explicitly.

## 5. Service operations

```js
// src/services/product.service.js
import { Product } from "../models/product.model.js";

export const createProduct = input => Product.create(input);

export const listProducts = () => Product.find({ active: true })
  .select("name sku priceMinor stock")
  .sort({ _id: 1 }).limit(20).lean();

export const getProduct = id => Product.findById(id)
  .select("name sku priceMinor stock").lean();

export const updateProduct = (id, patch) => Product.findByIdAndUpdate(
  id, { $set: patch }, { new: true, runValidators: true }
);

export const deleteProduct = id => Product.findByIdAndUpdate(
  id, { $set: { active: false } }, { new: true }
);
```

We use **soft delete** (`active:false`) rather than physical deletion because existing orders may refer to historical products. Soft delete requires consistent query filters and explicit recovery rules. In this example `getProduct` and `updateProduct` can still access inactive products by ID; production services should enforce the intended visibility and authorization policy.

## 6. Controllers: HTTP semantics

```js
// src/controllers/product.controller.js
import * as service from "../services/product.service.js";

export async function create(req, res) {
  const product = await service.createProduct(req.validatedBody);
  return res.status(201).json({ success: true, data: product });
}

export async function list(req, res) {
  return res.status(200).json({ success: true, data: await service.listProducts() });
}

export async function get(req, res) {
  const product = await service.getProduct(req.params.id);
  if (!product) return res.status(404).json({ message: "Product not found" });
  return res.status(200).json({ success: true, data: product });
}

export async function update(req, res) {
  const product = await service.updateProduct(req.params.id, req.validatedBody);
  if (!product) return res.status(404).json({ message: "Product not found" });
  return res.status(200).json({ success: true, data: product });
}

export async function remove(req, res) {
  const product = await service.deleteProduct(req.params.id);
  if (!product) return res.status(404).json({ message: "Product not found" });
  return res.status(204).send();
}
```

Express 5 forwards rejected async handlers to error middleware. It does not automatically decide whether a Mongoose error should be 400, 409 or 500; we'll centralize that in Lesson 077.

## 7. Routes and authorization boundary

```js
// src/routes/product.routes.js
import { Router } from "express";
import * as controller from "../controllers/product.controller.js";
import { validateBody } from "../middlewares/validate.middleware.js";
import {
  createProductSchema, updateProductSchema, productIdSchema
} from "../validators/product.validator.js";

const router = Router();
const validateId = (req, res, next) => {
  if (!productIdSchema.safeParse(req.params.id).success) {
    return res.status(400).json({ message: "Invalid product id" });
  }
  next();
};

router.get("/", controller.list);
router.get("/:id", validateId, controller.get);
router.post("/", validateBody(createProductSchema), controller.create);
router.patch("/:id", validateId, validateBody(updateProductSchema), controller.update);
router.delete("/:id", validateId, controller.remove);
export default router;
```

**Security warning:** This route example is a teaching scaffold. **Do not deploy POST/PATCH/DELETE publicly without authentication and admin authorization.** Lesson 073 adds those checks.

## 8. Expected HTTP behavior

| Method | Path | Success | Error |
| --- | --- | --- | --- |
| POST | `/api/products` | 201 Created | 400 invalid; 409 duplicate SKU |
| GET | `/api/products` | 200 array | 500 unexpected |
| GET | `/api/products/:id` | 200 product | 400 malformed ID; 404 missing |
| PATCH | `/api/products/:id` | 200 updated product | 400 invalid; 404 missing |
| DELETE | `/api/products/:id` | 204 no body | 404 missing |

```bash
curl -X POST http://localhost:3000/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Wireless Mouse","sku":"MOUSE-001","priceMinor":49900,"stock":10}'
```

## 9. Why query update validation differs

`findByIdAndUpdate()` uses **query update semantics**, not document `save()`. `runValidators:true` enables supported update validators but does not run `pre("save")` hooks or perform complete document validation. For stock reservation, use a conditional atomic update; do not simply read stock, subtract and save under concurrency.

## 10. Exercises and interview questions

Test missing name, negative price, unknown body fields, duplicate SKU, malformed ID and valid missing ID. Add a test that PATCH cannot set `active:false` or an arbitrary `isAdmin` property. Explain **why HTTP validation and Mongoose validation are both useful**, and why `unique` is a database constraint rather than a validator.

**References:** [Express 5 routing](https://expressjs.com/en/guide/routing.html) · [Mongoose validation](https://mongoosejs.com/docs/validation.html) · [Zod](https://zod.dev/)
