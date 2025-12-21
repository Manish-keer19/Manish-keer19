# Prisma Mastery Roadmap: Beginner to Production Expert

## PHASE 0 — Prerequisites (Databases & SQL)
**Goal:** Establish the foundational knowledge required to use an ORM effectively. You cannot master Prisma without understanding the underlying database.

*   **Mandatory SQL Knowledge**:
    *   `SELECT`, `INSERT`, `UPDATE`, `DELETE` syntax.
    *   `WHERE`, `ORDER BY`, `LIMIT`, `OFFSET`.
    *   `JOIN` types (INNER, LEFT, RIGHT) and how they merge data.
    *   **Transactions**: `BEGIN`, `COMMIT`, `ROLLBACK`.
    *   **Constraints**: Primary Keys, Foreign Keys, Unique, Not Null.
*   **PostgreSQL Specifics**:
    *   Data types (`TEXT` vs `VARCHAR`, `JSONB`, `TIMESTAMP`).
    *   Schemas (`public` namespace).
    *   Connection strings (Structure: `postgresql://user:pass@host:port/db`).
*   **Why This Matters**: Prisma is a layer *over* SQL. Debugging requires reading raw SQL to understand why a query is slow or failing.

---

## PHASE 1 — Prisma Fundamentals
**Goal:** Understand what Prisma actually is, how it fits into the stack, and its internal architecture.

*   **Core Concepts**:
    *   **Prisma Client**: The auto-generated, type-safe Node.js/TypeScript query builder.
    *   **Prisma Schema (`schema.prisma`)**: The single source of truth for your data model.
    *   **Prisma Studio**: The GUI for viewing database content.
*   **Architecture & Internals**:
    *   **The Query Engine**: Understanding that Prisma runs a Rust binary sidecar.
    *   **The Translation Layer**: How `prisma.user.findMany()` becomes `SELECT * FROM "User"`.
    *   **Code Generation**: How `prisma generate` builds `node_modules/@prisma/client`.
*   **Production Relevance**: Understanding the binary size and execution flow (serialization/deserialization) between JS and Rust.

---

## PHASE 2 — Schema Design & Data Modeling
**Goal:** Learn to translate business requirements into a rigid, efficient database schema using Prisma's DSL (Domain Specific Language).

*   **Field Definitions**:
    *   Scalar types (`String`, `Int`, `Boolean`, `DateTime`, `Json`).
    *   Modifiers (`?` for optional/NULL, `[]` for arrays).
    *   Attributes (`@id`, `@unique`, `@default()`, `@updatedAt`).
*   **Relation Modeling**:
    *   **1-to-1**: `User` <-> `Profile` (Usage of `@unique` on foreign key).
    *   **1-to-Many**: `User` <-> `Post` (The most common relation).
    *   **Many-to-Many**: `Post` <-> `Tag` (Implicit vs Explicit join tables).
    *   **Self-Relations**: `User` (manager) <-> `User` (employee).
*   **Enums**: Modeling fixed sets of values (e.g., `Role`, `Status`) for type safety.
*   **Indexes**: Using `@@index` for performance on non-unique fields.
*   **Mapping**: Using `@map` and `@@map` to decouple Prisma names from DB table/column names.

---

## PHASE 3 — CRUD Mastery
**Goal:** Master the bread-and-butter operations. Stop writing repetitive boilerplate and use Prisma's fluent API.

*   **Create**:
    *   `create`: Single record insertion.
    *   `createMany`: Bulk insertion (Critical for seeding/imports).
    *   **Nested Writes**: Creating a User AND their Profile in one command (Transactional by default).
*   **Read**:
    *   `findUnique`: Fetch by ID or unique field (Optimized).
    *   `findMany`: Fetch lists with pagination (`take`, `skip`, `cursor`).
    *   `findFirst`: Reading the "latest" item.
*   **Update**:
    *   `update`: Modifying by unique identifier.
    *   `updateMany`: Updating multiple records based on a filter context (Warning: does not trigger `@updatedAt` in some versions, check docs).
    *   **Upsert**: The "Create if not exists, otherwise Update" pattern (`upsert`).
*   **Delete**:
    *   `delete`: Remove by ID.
    *   `deleteMany`: Bulk deletion.
    *   **Cascading Deletes**: Configuring `onDelete: Cascade` in schema vs manual handling.

---

## PHASE 4 — Advanced Querying
**Goal:** Move beyond simple lookups. Handle complex filtering logic matching real-world search requirements.

*   **Logical Operators**:
    *   `AND`, `OR`, `NOT`: constructing complex boolean logic trees.
*   **Comparison Operators**:
    *   `lt`, `lte`, `gt`, `gte` (Numbers/Dates).
    *   `in`, `notIn` (Array containment).
*   **String Filters**:
    *   `contains`, `startsWith`, `endsWith`.
    *   **Mode**: `insensitive` (PostgreSQL `ILIKE`).
*   **JSON Filtering**: Querying deep inside `JSONB` columns (PostgreSQL specific superpower).
*   **Deep Filtering**: Filtering a User based on the properties of their Posts (e.g., "Find users who have posted about 'Tech'").

---

## PHASE 5 — Relations & Joins
**Goal:** Master efficient data retrieval across tables. This is where ORMs often kill performance.

*   **Data Retrieval**:
    *   **`include`**: Eager loading related data (The SQL `JOIN` equivalent).
    *   **`select`**: Fetching ONLY specific fields (Projection).
    *   **Nested Selects**: Selecting specific fields of a related model.
*   **The N+1 Problem**:
    *   Validating when Prisma optimizes queries vs when it fires 100+ queries.
    *   Using `relationLoadStrategy: 'join'` (Prisma 5.10+ features) for database-level joins.
*   **Fluent API Traversal**: navigating from a record to its relations internally.

---

## PHASE 6 — Aggregations & Analytics
**Goal:** Perform calculations ON the database, not in the application (Node.js is single-threaded, don't block it).

*   **Aggregate Functions**:
    *   `_count`, `_sum`, `_avg`, `_min`, `_max`.
*   **Grouping**:
    *   `groupBy`: Aggregating data by specific columns (e.g., "Sales by Region").
    *   `having`: Filtering the result of a grouping (e.g., "Regions with > 1000 sales").
*   **Count Relations**: Fast counting of related records (e.g., `_count` of posts for a user) without fetching the simpler data.

---

## PHASE 7 — Transactions & Concurrency
**Goal:** Ensure data integrity. Handling money, inventory, or critical state changes safely.

*   **Sequential Transactions** (`$transaction([...])`):
    *   Running an array of independent queries together. All succeed or all fail.
*   **Interactive Transactions** (`$transaction(async (tx) => { ... })`):
    *   Using the output of Query A to determine Query B inside a lock.
    *   **Timeout Handling**: Configuring max wait times.
*   **Isolation Levels**: Understanding Read Committed vs Serializable (mostly relevant for Interactive Transactions).
*   **Optimistic Concurrency Control**: Handling specialized versioning collisions (using `@version` fields).

---

## PHASE 8 — Raw SQL & Security
**Goal:** Knowing when to escape the ORM. Writing high-performance raw SQL safely.

*   **Raw Methods**:
    *   `$queryRaw`: Returns strongly typed results (if mapped).
    *   `$executeRaw`: Returns number of affected rows.
*   **The Danger Zone**:
    *   `$queryRawUnsafe`: Executing dynamic strings (Extreme Risk).
*   **Security**:
    *   **SQL Injection**: Implementation of Parameterized Queries (Prisma's tagged template literals).
    *   Why string concatenation (`SELECT * FROM table WHERE id = ` + id) is strictly forbidden.
*   **Use Cases**:
    *   Complex Window Functions.
    *   Recursive CTEs.
    *   PostGIS/Geospatial queries.

---

## PHASE 9 — Performance Optimization
**Goal:** Making queries fast at scale.

*   **Indexing Strategy**:
    *   Single column indexes (`@@index`).
    *   Composite indexes (Multi-column) for specific filter combinations (`@@index([a, b])`).
*   **Projection**: Use `select` aggressively. Never do `findMany()` (SELECT *) on tables with huge Text/JSON blobs if you don't need them.
*   **Pagination**:
    *   `offset`/`limit` (Slow for large datasets).
    *   **Cursor-based Pagination**: Infinite scroll scaling (`cursor: { id: ... }`).
*   **Batching**: Using `createMany` and `updateMany` instead of loops in JavaScript.
*   **Connection Pooling**:
    *   Why Postgres connections are expensive.
    *   Using **PgBouncer** or Prisma Accelerate to multiplex connections.

---

## PHASE 10 — Migrations & Versioning
**Goal:** Managing database schema changes over time in a team environment.

*   **The Migration Flow**:
    *   `prisma migrate dev`: For local iteration (Generates SQL + applies it).
    *   `prisma migrate deploy`: For CI/CD and Production (Applies pending SQL only).
*   **Schema Drift**:
    *   `prisma db pull`: Introspecting an existing DB to generate schema (Reverse engineering).
    *   **Shadow Database**: Why Prisma needs a second empty DB to validate migrations safely.
*   **Production Safety**:
    *   Zero-downtime migrations (Expand and Contract pattern).
    *   Why `prisma db push` is forbidden in production.

---

## PHASE 11 — Prisma in Production
**Goal:** Running robustly in the real world.

*   **Environment Management**:
    *   Handling `DATABASE_URL` securely.
    *   Secrets management (AWS Secrets Manager / Vault).
*   **Logging & Observability**:
    *   Configuring Log Levels (`query`, `info`, `warn`).
    *   Long-running query logging.
    *   Tracing Prisma requests with OpenTelemetry.
*   **Serverless Considerations**:
    *   Cold starts and connection limits.
    *   Instantiating `PrismaClient` outside the handler scope.
*   **Error Handling**:
    *   Catching `PrismaClientKnownRequestError`.
    *   Handling Unique Constraint violations (P2002) and Record Not Found (P2025) gracefully.

---

## PHASE 12 — Real-World System Design
**Goal:** Putting it all together. Design a "WhatsApp" clone backend logic.

*   **Schema**:
    *   `User`, `Conversation` (Group/Direct), `Message`, `Attachment`.
    *   `MessageStatus` (Sent, Delivered, Read).
*   **Complex Operations**:
    *   Fetch "My Conversations" with "Last Message" and "Unread Count" (Complex Relation + Aggregation).
    *   Send Message (Transaction: Insert Message + Update Conversation `lastMessageAt`).
*   **Optimization**:
    *   Index `senderId` and `conversationId`.
    *   Cursor pagination for scrolling back in chat history.
