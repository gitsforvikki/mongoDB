# MongoDB + Mongoose ODM — Beginner to Production

> A hands-on learning roadmap for JavaScript / Node.js developers: learn **MongoDB itself first**, then build real APIs using **Mongoose**. The curriculum is published now; individual lesson content will be added progressively.

## What you will learn

MongoDB databases, documents, BSON, CRUD, query operators, indexing, aggregation, modeling, MongoDB Atlas, Mongoose schemas/models/hooks/transactions, Express 5 integration, security, testing, performance and production deployment.

## Study order and priority

**Phase A — MongoDB database skills:** Sections 1–5. Practice in `mongosh` and MongoDB Compass so you understand what Mongoose runs against.

**Phase B — Mongoose ODM + backend API:** Sections 6–8. Implement MongoDB-backed models and controllers in Express 5.

**Phase C — Correctness and production:** Sections 9–11. Focus on transactions, security, indexes, pooling and testing.

**Phase D — Portfolio and interviews:** Section 12. Build projects and practice questions.

## Curriculum (click any lesson)

### [Section 1 — MongoDB Foundations](./01-mongodb-foundations/README.md)

- [Lesson 001 — What Is MongoDB and Why Use It?](./01-mongodb-foundations/001-what-is-mongodb.md)
- [Lesson 002 — MongoDB vs Mongoose vs Atlas](./01-mongodb-foundations/002-mongodb-vs-mongoose-vs-atlas.md)
- [Lesson 003 — Installation: Local MongoDB, Compass and mongosh](./01-mongodb-foundations/003-installation-local-mongodb-compass-mongosh.md)
- [Lesson 004 — MongoDB Atlas Setup and Connection Strings](./01-mongodb-foundations/004-mongodb-atlas-setup-connection-strings.md)
- [Lesson 005 — Databases, Collections and Documents](./01-mongodb-foundations/005-databases-collections-documents.md)
- [Lesson 006 — BSON, ObjectId and MongoDB Data Types](./01-mongodb-foundations/006-bson-objectid-data-types.md)
- [Lesson 007 — mongosh Essentials and Basic Commands](./01-mongodb-foundations/007-mongosh-essentials-basic-commands.md)
- [Lesson 008 — MongoDB CRUD Mental Model](./01-mongodb-foundations/008-mongodb-crud-mental-model.md)

### [Section 2 — MongoDB CRUD & Querying](./02-crud-and-querying/README.md)

- [Lesson 009 — Insert Documents: insertOne and insertMany](./02-crud-and-querying/009-insert-documents-insertone-insertmany.md)
- [Lesson 010 — Read Documents: find and findOne](./02-crud-and-querying/010-read-documents-find-findone.md)
- [Lesson 011 — Comparison Query Operators](./02-crud-and-querying/011-comparison-query-operators.md)
- [Lesson 012 — Logical and Element Operators](./02-crud-and-querying/012-logical-element-operators.md)
- [Lesson 013 — Nested Fields and Arrays](./02-crud-and-querying/013-nested-fields-arrays.md)
- [Lesson 014 — Projection, Sorting, Skip and Limit](./02-crud-and-querying/014-projection-sorting-skip-limit.md)
- [Lesson 015 — Update Operators](./02-crud-and-querying/README.md#lesson-015-update-operators)
- [Lesson 016 — updateOne, updateMany and replaceOne](./02-crud-and-querying/README.md#lesson-016-updateone-updatemany-and-replaceone)
- [Lesson 017 — Delete Documents and Bulk Writes](./02-crud-and-querying/README.md#lesson-017-delete-documents-and-bulk-writes)
- [Lesson 018 — Upsert and Atomic Single-Document Updates](./02-crud-and-querying/README.md#lesson-018-upsert-and-atomic-single-document-updates)
- [Lesson 019 — Regex, Text and Search Options](./02-crud-and-querying/README.md#lesson-019-regex-text-and-search-options)
- [Lesson 020 — Pagination: Offset vs Cursor](./02-crud-and-querying/README.md#lesson-020-pagination-offset-vs-cursor)

### [Section 3 — Document Modeling & Schema Design](./03-data-modeling/README.md)

- [Lesson 021 — Embedding vs Referencing](./03-data-modeling/README.md#lesson-021-embedding-vs-referencing)
- [Lesson 022 — One-to-One and One-to-Many Relationships](./03-data-modeling/README.md#lesson-022-one-to-one-and-one-to-many-relationships)
- [Lesson 023 — Many-to-Many Relationships](./03-data-modeling/README.md#lesson-023-many-to-many-relationships)
- [Lesson 024 — Designing E-commerce Schemas](./03-data-modeling/README.md#lesson-024-designing-e-commerce-schemas)
- [Lesson 025 — Schema Design for Queries and Growth](./03-data-modeling/README.md#lesson-025-schema-design-for-queries-and-growth)
- [Lesson 026 — Document Limits and Modeling Anti-patterns](./03-data-modeling/README.md#lesson-026-document-limits-and-modeling-anti-patterns)
- [Lesson 027 — MongoDB Schema Validation](./03-data-modeling/README.md#lesson-027-mongodb-schema-validation)
- [Lesson 028 — Data Modeling Exercises and Review](./03-data-modeling/README.md#lesson-028-data-modeling-exercises-and-review)

### [Section 4 — Indexes & Query Performance](./04-indexing-performance/README.md)

- [Lesson 029 — What Is an Index and Why Is It Needed?](./04-indexing-performance/README.md#lesson-029-what-is-an-index-and-why-is-it-needed)
- [Lesson 030 — Single-Field and Compound Indexes](./04-indexing-performance/README.md#lesson-030-single-field-and-compound-indexes)
- [Lesson 031 — Unique, Sparse and Partial Indexes](./04-indexing-performance/README.md#lesson-031-unique-sparse-and-partial-indexes)
- [Lesson 032 — Multikey and Text Indexes](./04-indexing-performance/README.md#lesson-032-multikey-and-text-indexes)
- [Lesson 033 — TTL and Specialized Indexes](./04-indexing-performance/README.md#lesson-033-ttl-and-specialized-indexes)
- [Lesson 034 — Explain Plans and Query Execution](./04-indexing-performance/README.md#lesson-034-explain-plans-and-query-execution)
- [Lesson 035 — Index Design and Trade-offs](./04-indexing-performance/README.md#lesson-035-index-design-and-trade-offs)
- [Lesson 036 — Performance and Pagination Lab](./04-indexing-performance/README.md#lesson-036-performance-and-pagination-lab)

### [Section 5 — Aggregation Pipeline](./05-aggregation/README.md)

- [Lesson 037 — Aggregation Concepts and Pipeline Flow](./05-aggregation/README.md#lesson-037-aggregation-concepts-and-pipeline-flow)
- [Lesson 038 — $match, $project, $sort and $limit](./05-aggregation/README.md#lesson-038-match-project-sort-and-limit)
- [Lesson 039 — $group and Accumulators](./05-aggregation/README.md#lesson-039-group-and-accumulators)
- [Lesson 040 — $unwind and Array Processing](./05-aggregation/README.md#lesson-040-unwind-and-array-processing)
- [Lesson 041 — $lookup and $facet](./05-aggregation/README.md#lesson-041-lookup-and-facet)
- [Lesson 042 — $addFields, $set and Computed Values](./05-aggregation/README.md#lesson-042-addfields-set-and-computed-values)
- [Lesson 043 — Dates, Strings and Conditional Expressions](./05-aggregation/README.md#lesson-043-dates-strings-and-conditional-expressions)
- [Lesson 044 — Aggregations for E-commerce Analytics](./05-aggregation/README.md#lesson-044-aggregations-for-e-commerce-analytics)
- [Lesson 045 — Aggregation Optimization](./05-aggregation/README.md#lesson-045-aggregation-optimization)

### [Section 6 — Mongoose ODM Fundamentals](./06-mongoose-foundations/README.md)

- [Lesson 046 — Why Mongoose? Driver vs ODM](./06-mongoose-foundations/README.md#lesson-046-why-mongoose-driver-vs-odm)
- [Lesson 047 — Set Up Express 5 + Mongoose + Atlas](./06-mongoose-foundations/README.md#lesson-047-set-up-express-5-mongoose-atlas)
- [Lesson 048 — Schemas and Models](./06-mongoose-foundations/README.md#lesson-048-schemas-and-models)
- [Lesson 049 — Schema Types, Defaults and Required Fields](./06-mongoose-foundations/README.md#lesson-049-schema-types-defaults-and-required-fields)
- [Lesson 050 — Creating and Reading Documents](./06-mongoose-foundations/README.md#lesson-050-creating-and-reading-documents)
- [Lesson 051 — Updating and Deleting Documents](./06-mongoose-foundations/README.md#lesson-051-updating-and-deleting-documents)
- [Lesson 052 — Document save vs Query Updates](./06-mongoose-foundations/README.md#lesson-052-document-save-vs-query-updates)
- [Lesson 053 — Casting, Validation and Error Types](./06-mongoose-foundations/README.md#lesson-053-casting-validation-and-error-types)
- [Lesson 054 — Strict Mode, strictQuery and Schema Options](./06-mongoose-foundations/README.md#lesson-054-strict-mode-strictquery-and-schema-options)
- [Lesson 055 — Lean Queries and Projections](./06-mongoose-foundations/README.md#lesson-055-lean-queries-and-projections)

### [Section 7 — Mongoose ODM Advanced Patterns](./07-mongoose-advanced/README.md)

- [Lesson 056 — Schema Validation: Built-in and Custom](./07-mongoose-advanced/README.md#lesson-056-schema-validation-built-in-and-custom)
- [Lesson 057 — Nested Schemas and Subdocuments](./07-mongoose-advanced/README.md#lesson-057-nested-schemas-and-subdocuments)
- [Lesson 058 — populate() and ref](./07-mongoose-advanced/README.md#lesson-058-populate-and-ref)
- [Lesson 059 — Virtuals and Getters/Setters](./07-mongoose-advanced/README.md#lesson-059-virtuals-and-getterssetters)
- [Lesson 060 — Instance Methods, Static Methods and Query Helpers](./07-mongoose-advanced/README.md#lesson-060-instance-methods-static-methods-and-query-helpers)
- [Lesson 061 — Document, Query, Model and Aggregate Middleware](./07-mongoose-advanced/README.md#lesson-061-document-query-model-and-aggregate-middleware)
- [Lesson 062 — Mongoose Transactions and Sessions](./07-mongoose-advanced/README.md#lesson-062-mongoose-transactions-and-sessions)
- [Lesson 063 — Indexes and Unique Constraints in Mongoose](./07-mongoose-advanced/README.md#lesson-063-indexes-and-unique-constraints-in-mongoose)
- [Lesson 064 — Discriminators and Polymorphic Data](./07-mongoose-advanced/README.md#lesson-064-discriminators-and-polymorphic-data)
- [Lesson 065 — Optimistic Concurrency and Version Keys](./07-mongoose-advanced/README.md#lesson-065-optimistic-concurrency-and-version-keys)
- [Lesson 066 — Plugins and Reusable Schema Logic](./07-mongoose-advanced/README.md#lesson-066-plugins-and-reusable-schema-logic)
- [Lesson 067 — Mongoose Anti-patterns and Performance](./07-mongoose-advanced/README.md#lesson-067-mongoose-anti-patterns-and-performance)

### [Section 8 — Build a Real Express + Mongoose API](./08-express-api-project/README.md)

- [Lesson 068 — Project Structure: Routes, Controllers and Services](./08-express-api-project/README.md#lesson-068-project-structure-routes-controllers-and-services)
- [Lesson 069 — Environment Configuration and DB Connection](./08-express-api-project/README.md#lesson-069-environment-configuration-and-db-connection)
- [Lesson 070 — Product CRUD API with Request Validation](./08-express-api-project/README.md#lesson-070-product-crud-api-with-request-validation)
- [Lesson 071 — User Registration and Password Hashing](./08-express-api-project/README.md#lesson-071-user-registration-and-password-hashing)
- [Lesson 072 — JWT / Cookie Authentication](./08-express-api-project/README.md#lesson-072-jwt-cookie-authentication)
- [Lesson 073 — Authorization and Ownership Checks](./08-express-api-project/README.md#lesson-073-authorization-and-ownership-checks)
- [Lesson 074 — Products, Cart and Orders Modeling](./08-express-api-project/README.md#lesson-074-products-cart-and-orders-modeling)
- [Lesson 075 — Filtering, Sorting and Cursor Pagination API](./08-express-api-project/README.md#lesson-075-filtering-sorting-and-cursor-pagination-api)
- [Lesson 076 — Aggregation-based Dashboard Endpoints](./08-express-api-project/README.md#lesson-076-aggregation-based-dashboard-endpoints)
- [Lesson 077 — Centralized Errors, Logging and API Tests](./08-express-api-project/README.md#lesson-077-centralized-errors-logging-and-api-tests)

### [Section 9 — Transactions, Concurrency & Consistency](./09-transactions-consistency/README.md)

- [Lesson 078 — MongoDB Atomicity and Isolation Basics](./09-transactions-consistency/README.md#lesson-078-mongodb-atomicity-and-isolation-basics)
- [Lesson 079 — Race Conditions and Conditional Updates](./09-transactions-consistency/README.md#lesson-079-race-conditions-and-conditional-updates)
- [Lesson 080 — Transactions with Mongoose Sessions](./09-transactions-consistency/README.md#lesson-080-transactions-with-mongoose-sessions)
- [Lesson 081 — Retryable Writes and Transaction Retries](./09-transactions-consistency/README.md#lesson-081-retryable-writes-and-transaction-retries)
- [Lesson 082 — Idempotency Keys and Unique Constraints](./09-transactions-consistency/README.md#lesson-082-idempotency-keys-and-unique-constraints)
- [Lesson 083 — Consistency and Read/Write Concerns](./09-transactions-consistency/README.md#lesson-083-consistency-and-readwrite-concerns)
- [Lesson 084 — Change Streams Introduction](./09-transactions-consistency/README.md#lesson-084-change-streams-introduction)

### [Section 10 — Security, Backup & Operations](./10-security-operations/README.md)

- [Lesson 085 — MongoDB Users, Roles and Least Privilege](./10-security-operations/README.md#lesson-085-mongodb-users-roles-and-least-privilege)
- [Lesson 086 — Network Security, TLS and Atlas Access](./10-security-operations/README.md#lesson-086-network-security-tls-and-atlas-access)
- [Lesson 087 — Secrets and Connection String Management](./10-security-operations/README.md#lesson-087-secrets-and-connection-string-management)
- [Lesson 088 — NoSQL Injection and Query Sanitization](./10-security-operations/README.md#lesson-088-nosql-injection-and-query-sanitization)
- [Lesson 089 — Backups, Restore and Atlas Recovery](./10-security-operations/README.md#lesson-089-backups-restore-and-atlas-recovery)
- [Lesson 090 — Replica Sets and Failover Basics](./10-security-operations/README.md#lesson-090-replica-sets-and-failover-basics)
- [Lesson 091 — Sharding and Horizontal Scaling Overview](./10-security-operations/README.md#lesson-091-sharding-and-horizontal-scaling-overview)
- [Lesson 092 — Monitoring MongoDB and Connection Pools](./10-security-operations/README.md#lesson-092-monitoring-mongodb-and-connection-pools)
- [Lesson 093 — Database Migrations and Safe Deployment](./10-security-operations/README.md#lesson-093-database-migrations-and-safe-deployment)

### [Section 11 — Testing, Production & Optimization](./11-testing-and-production/README.md)

- [Lesson 094 — Unit Testing Mongoose-dependent Services](./11-testing-and-production/README.md#lesson-094-unit-testing-mongoose-dependent-services)
- [Lesson 095 — Integration Tests with Test MongoDB](./11-testing-and-production/README.md#lesson-095-integration-tests-with-test-mongodb)
- [Lesson 096 — Testing Indexes, Hooks and Transactions](./11-testing-and-production/README.md#lesson-096-testing-indexes-hooks-and-transactions)
- [Lesson 097 — Mongoose Connection Pooling](./11-testing-and-production/README.md#lesson-097-mongoose-connection-pooling)
- [Lesson 098 — Caching with Redis](./11-testing-and-production/README.md#lesson-098-caching-with-redis)
- [Lesson 099 — Bulk Operations and Streaming Large Results](./11-testing-and-production/README.md#lesson-099-bulk-operations-and-streaming-large-results)
- [Lesson 100 — Production Performance Checklist](./11-testing-and-production/README.md#lesson-100-production-performance-checklist)
- [Lesson 101 — Deploy Express + MongoDB to Production](./11-testing-and-production/README.md#lesson-101-deploy-express-mongodb-to-production)

### [Section 12 — Projects, Practice & Interview Preparation](./12-real-world-projects/README.md)

- [Lesson 102 — Project 1 — Product Catalog API](./12-real-world-projects/README.md#lesson-102-project-1-product-catalog-api)
- [Lesson 103 — Project 2 — Order Management Backend](./12-real-world-projects/README.md#lesson-103-project-2-order-management-backend)
- [Lesson 104 — Project 3 — Job Application Tracker](./12-real-world-projects/README.md#lesson-104-project-3-job-application-tracker)
- [Lesson 105 — Project 4 — Real-Time Chat Data Modeling](./12-real-world-projects/README.md#lesson-105-project-4-real-time-chat-data-modeling)
- [Lesson 106 — MongoDB Interview Questions and Query Challenges](./12-real-world-projects/README.md#lesson-106-mongodb-interview-questions-and-query-challenges)
- [Lesson 107 — Mongoose Interview Questions and Debugging](./12-real-world-projects/README.md#lesson-107-mongoose-interview-questions-and-debugging)
- [Lesson 108 — Final Production Review and Capstone Checklist](./12-real-world-projects/README.md#lesson-108-final-production-review-and-capstone-checklist)

## Recommended learning workflow

- Read one lesson, run its MongoDB commands or Node.js code, then write a short explanation in your own words.
- Maintain a local test database and never use a production database for exercises.
- Keep lessons focused: definition → purpose → execution flow → example → common mistakes → practice → interview questions.
- Use official references alongside exercises: [MongoDB Manual](https://www.mongodb.com/docs/manual/), [MongoDB Node.js Driver](https://www.mongodb.com/docs/drivers/node/current/), [Mongoose Documentation](https://mongoosejs.com/docs/), and [MongoDB Atlas Documentation](https://www.mongodb.com/docs/atlas/).

## Tooling

MongoDB Community Server or Atlas Free cluster, `mongosh`, MongoDB Compass, Node.js LTS, Express 5, Mongoose, and a test client such as curl/Postman/Supertest. MongoDB CLI commands and Mongoose code serve different purposes—learn both.

## Course conventions

- Examples use JavaScript ES modules (`import/export`) and modern Express 5.
- MongoDB shell query syntax is labeled `js` but executed in `mongosh`; Mongoose examples run from Node.js.
- Query and Mongoose behavior can vary by server/library version; follow official docs for the installed versions.
- MongoDB indexes/unique constraints and real DB behavior must be tested; Mongoose schema validation alone is not a database constraint.
- All eight lessons in Section 1 have dedicated Markdown pages. Remaining section links point to outlines until their content is written.

**Total: 12 sections · 108 lessons.**
