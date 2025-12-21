# Prisma ORM – Beginner to Advanced (Production-Ready Guide)

## 1. What is Prisma?

### What Problem Does Prisma Solve?
Prisma solves the "disconnect" between your application code (JavaScript/TypeScript) and your database.
*   **Type Safety**: It auto-generates a type-safe client based on your database schema. If your database has a user with an `email` field, Typescript knows about it.
*   **Productivity**: Autocompletion (IntelliSense) for database queries.
*   **Maintenance**: Schema migrations are declarative and version-controlled.

### Prisma vs Traditional ORM (TypeORM, Sequelize)
*   **Traditional ORMs**: Map classes to database tables. Often suffer from "impedance mismatch" and complex internal object state.
*   **Prisma**: Doesn't map classes. It generates a custom client based on your `schema.prisma` file. It's more like a query builder with superpowers than a traditional ORM.

### Prisma vs Raw SQL
*   **Raw SQL**: Maximum control, but prone to typos, SQL injection (if not careful), and lack of type safety. Refactoring is painful.
*   **Prisma**: slightly less control than raw SQL for complex edge cases, but provides safety, speed, and cleaner code for 99% of operations.

### When NOT to use Prisma?
*   If you need to use database features not yet supported by Prisma (e.g., complex PostGIS geography queries, though support is improving).
*   If you are building a ultra-low-latency application where the slight overhead of the Query Engine is unacceptable (rare).

---

## 2. Prisma Architecture (Internal Working)

Prisma is not just a library; it has a running engine.

### The Components
1.  **Prisma Client**: The auto-generated Node.js library you import (`@prisma/client`).
2.  **Query Engine**: A binary (written in Rust) that runs alongside your app. It manages connections and translates Prisma queries into optimized SQL.

### Query Translation Flow
When you run `prisma.user.findMany()`, the following happens:

```text
+------------------+       +-------------------+       +---------------------+       +--------------+
|  Node.js Code    | ----> |   Prisma Client   | ----> | Rust Query Engine   | ----> |  PostgreSQL  |
| (prisma.find...) |       | (Validates Types) |       | (Generates SQL)     |       | ( Executes ) |
+------------------+       +-------------------+       +---------------------+       +--------------+
```

1.  **Request**: JS calls the Client.
2.  **Validate**: Client validates input against the generated schema types.
3.  **Serialize**: Request is sent to the Rust Query Engine.
4.  **SQL Generation**: Engine generates optimized SQL.
5.  **Execution**: SQL runs on Postgres.
6.  **Formatting**: Results are formatted back to plain JS objects (POJOs) and returned.

---

## 3. Installing Prisma (Beginner)

Assumes you have Node.js installed.

### 1. Setup Node.js Project
```bash
mkdir prisma-guide
cd prisma-guide
npm init -y
npm install typescript ts-node @types/node --save-dev
```

### 2. Install Prisma CLI & Client
```bash
npm install prisma --save-dev
npm install @prisma/client
```

### 3. Initialize Prisma
This creates a `prisma` folder and `.env` file.
```bash
npx prisma init
```

### 4. Configure Database
In `.env`:
```bash
DATABASE_URL="postgresql://user:password@localhost:5432/mydb?schema=public"
```

### 5. Generate & Migrate
After writing your schema (see below):
```bash
npx prisma migrate dev --name init  # Applies SQL to DB
npx prisma generate                 # Updates Node.js Client
```

---

## 4. Prisma Schema Fundamentals

The `schema.prisma` file is the source of truth.

### Basic Structure
```prisma
// 1. Data Source: Where is the DB?
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 2. Generator: build the client for Node.js
generator client {
  provider = "prisma-client-js"
}

// 3. Models: Your database tables
model User {
  id        Int      @id @default(autoincrement()) // Primary Key
  email     String   @unique                       // Unique Constraint
  name      String?                                // Optional (Nullable)
  role      Role     @default(USER)                // Enum with Default
  createdAt DateTime @default(now())               // Auto timestamp
  updatedAt DateTime @updatedAt                    // Auto update on change
}

// 4. Enums
enum Role {
  USER
  ADMIN
}
```

*   `@id`: Marks the primary key.
*   `@default(autoincrement())`: DB handles ID generation (1, 2, 3...).
*   `String?`: The `?` means the column allows `NULL`.
*   `@updatedAt`: Prisma automatically sets this field to the current time whenever you update the row.

---

## 5. Relations in Prisma (Beginner → Intermediate)

### One-to-Many (1-n)
A User has many Posts.

```prisma
model User {
  id    Int    @id @default(autoincrement())
  posts Post[] // "Virtual" field, essentially usage helper
}

model Post {
  id        Int  @id @default(autoincrement())
  authorId  Int  // Foreign Key (Actual column in DB)
  author    User @relation(fields: [authorId], references: [id])
}
```

**Internal Diagram:**
```text
User 1 ───────────< * Post
(id) <---------- (authorId)
```

### One-to-One (1-1)
A User has one Profile.

```prisma
model User {
  id      Int      @id @default(autoincrement())
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  userId Int  @unique // Must be unique for 1-1
  user   User @relation(fields: [userId], references: [id])
}
```

### Many-to-Many (m-n)
Posts have Tags, Tags have Posts.

**Implicit (Easier)**:
Prisma handles the join table for you.
```prisma
model Post {
  id   Int   @id @default(autoincrement())
  tags Tag[]
}

model Tag {
  id    Int    @id @default(autoincrement())
  posts Post[]
}
```

**Explicit (Production Preferred)**:
You define the join table to add extra fields (like `assignedAt`).
```prisma
model Post {
  id   Int @id @default(autoincrement())
  tags TagsOnPosts[]
}

model Tag {
  id    Int @id @default(autoincrement())
  posts TagsOnPosts[]
}

model TagsOnPosts {
  postId Int
  tagId  Int
  post   Post @relation(fields: [postId], references: [id])
  tag    Tag  @relation(fields: [tagId], references: [id])

  @@id([postId, tagId]) // Composite Key
}
```

---

## 6. Prisma CRUD Operations (Core Section)

Assume `const prisma = new PrismaClient();`

### CREATE

**Prisma Client:**
```typescript
// Create single
const user = await prisma.user.create({
  data: {
    email: 'alice@prisma.io',
    name: 'Alice',
    posts: { // Nested Create
      create: { title: 'Hello World' }
    }
  },
});
```

**Equivalent SQL:**
```sql
BEGIN;
INSERT INTO "User" ("email", "name", "updatedAt") 
VALUES ('alice@prisma.io', 'Alice', NOW()) 
RETURNING "id";

INSERT INTO "Post" ("title", "authorId", "updatedAt") 
VALUES ('Hello World', 1, NOW()); -- 1 comes from previous query
COMMIT;
```
*Note: Prisma handles the transaction for nested creates automatically.*

---

### READ

**Prisma Client:**
```typescript
const users = await prisma.user.findMany({
  where: {
    email: { endsWith: '@prisma.io' },
    active: true
  },
  select: { id: true, name: true }, // Only return these fields
  take: 10,                         // LIMIT
  skip: 0,                          // OFFSET
  orderBy: { createdAt: 'desc' }
});
```

**Equivalent SQL:**
```sql
SELECT "id", "name" 
FROM "User" 
WHERE "email" LIKE '%@prisma.io' 
  AND "active" = true
ORDER BY "createdAt" DESC 
LIMIT 10 OFFSET 0;
```

---

### UPDATE

**Prisma Client:**
```typescript
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { 
    name: 'Alice Wonderland',
    visits: { increment: 1 } // Atomic increment
  }
});
```

**Equivalent SQL:**
```sql
UPDATE "User" 
SET "name" = 'Alice Wonderland', 
    "visits" = "visits" + 1,
    "updatedAt" = NOW()
WHERE "id" = 1 
RETURNING "id", "email", "name", ...;
```

---

### DELETE

**Prisma Client:**
```typescript
const deleted = await prisma.user.delete({
  where: { email: 'alice@prisma.io' }
});
```

**Equivalent SQL:**
```sql
DELETE FROM "User" 
WHERE "email" = 'alice@prisma.io'
RETURNING id, email...;
```

**Soft Delete Pattern (Best Practice):**
Don't actually delete; just mark as deleted.
```typescript
// Update instead of Delete
await prisma.user.update({
  where: { id: 1 },
  data: { isDeleted: true }
});
```

---

## 7. Advanced Queries

### Filtering Operators
```typescript
await prisma.product.findMany({
  where: {
    price: { 
      gte: 10,   // Greater than or equal (>=)
      lte: 50    // Less than or equal (<=)
    },
    OR: [        // OR condition
      { category: 'Books' },
      { category: 'Movies' }
    ],
    NOT: {       // NOT condition
      stock: 0
    }
  }
});
```

### Case Sensitivity & Full Text
PostgreSQL ILIKE (Case Insensitive):
```typescript
await prisma.user.findMany({
  where: {
    name: { 
      contains: 'alice', 
      mode: 'insensitive' // Uses ILIKE instead of LIKE
    } 
  }
});
```

### IN / NOT IN
```typescript
await prisma.user.findMany({
  where: {
    id: { in: [1, 2, 3, 5] }
  }
});
```
**SQL**: `WHERE "id" IN (1, 2, 3, 5)`

---

## 8. Aggregations & Analytics

Prisma executes aggregation queries efficiently.

### Count, Sum, Avg, Min, Max

**Prisma Client:**
```typescript
const aggregations = await prisma.order.aggregate({
  _sum: { amount: true },
  _avg: { amount: true },
  _count: { id: true },
  where: { status: 'PAID' }
});
// Result: { _sum: { amount: 5000 }, _avg: { amount: 50 }, ... }
```

**Equivalent SQL:**
```sql
SELECT 
  SUM("amount"), 
  AVG("amount"), 
  COUNT("id") 
FROM "Order" 
WHERE "status" = 'PAID';
```

### Group By

**Prisma Client:**
```typescript
const group = await prisma.order.groupBy({
  by: ['country'],
  _sum: { amount: true },
  having: {
    amount: { _sum: { gt: 1000 } }
  }
});
```

**Equivalent SQL:**
```sql
SELECT "country", SUM("amount") 
FROM "Order" 
GROUP BY "country" 
HAVING SUM("amount") > 1000;
```

---

## 9. Transactions

ACID: Atomicity, Consistency, Isolation, Durability.

### 1. Sequential ($transaction array)
Pass an array of promises. If one fails, ALL fail (Rollback).
```typescript
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { ... } }),
  prisma.post.create({ data: { ... } })
]);
```
**SQL**: `BEGIN` -> Exec 1 -> Exec 2 -> `COMMIT` (or `ROLLBACK` on error).

### 2. Interactive ($transaction callback)
Use this when result of Query A is needed for Query B.
```typescript
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ ... });
  
  // Logic inside transaction
  if (user.balance < 10) throw new Error('Not enough funds');

  await tx.auditLog.create({ data: { userId: user.id } });
});
```
*Note: Use `tx` (the transaction client) inside the function, NOT `prisma`.*

---

## 10. Raw Queries

Use when Prisma's API doesn't support a specific SQL feature.

### Safe Query ($queryRaw)
Uses Prepared Statements. **Safe from SQL Injection**.
```typescript
const email = 'alice@example.com';
const users = await prisma.$queryRaw`SELECT * FROM "User" WHERE email = ${email}`;
```

### Unsafe Query ($queryRawUnsafe)
Executes a raw string. **Danger: Vulnerable to Injection**.
```typescript
// DO NOT DO THIS WITH USER INPUT
const users = await prisma.$queryRawUnsafe(`SELECT * FROM "User" WHERE email = '${input}'`);
```

---

## 11. Performance Optimization

### 1. The N+1 Problem
**Bad:**
```typescript
const users = await prisma.user.findMany();
for (const user of users) {
  // Runs 1 query per user! 100 users = 101 queries.
  const posts = await prisma.post.findMany({ where: { authorId: user.id } });
}
```

**Good (Eager Loading):**
```typescript
// Runs exactly 2 queries (one for users, one for posts) join is done in memory or DB level
const users = await prisma.user.findMany({
  include: { posts: true }
});
```

### 2. Select vs Include
Only fetch what you need. `include` fetches ALL fields of the relation.
```typescript
// Heavy
const user = prisma.user.findFirst({ include: { posts: true } });

// Optimized
const user = prisma.user.findFirst({
  select: {
    name: true,
    posts: {
      select: { title: true } // Don't fetch post content, just title
    }
  }
});
```

### 3. Indexes
Always add indexes for fields you filter by often.
```prisma
model User {
  email String @unique
  region String

  @@index([region]) // Index for faster searching by region
}
```

---

## 12. Prisma Migrations (Production)

### Development
```bash
npx prisma migrate dev --name add_profile
```
*   Creates SQL file in `prisma/migrations`.
*   Applies it to the dev DB.
*   Regenerates Client.

### Production
```bash
npx prisma migrate deploy
```
*   Does NOT look at schema.prisma.
*   Runs pending SQL migration files purely.
*   Safe for CI/CD pipelines.

### Shadow Database
Prisma uses a temporary second database to detect schema drift. Ensure your cloud user has `CREATE DATABASE` privileges.

---

## 13. Prisma in Production

### Environment Variables
Never hardcode credentials.
```env
# .env
DATABASE_URL="postgresql://user:pass@AWS_RDS_HOST:5432/db?connection_limit=5"
```

### Connection Pooling
Serverless functions (AWS Lambda, Vercel) can exhaust DB connections.
*   **Solution**: Use **PgBouncer** or **Prisma Accelerate**.
*   Set `connection_limit` in URL to avoid OOM.

### Logging
Debug slow queries.
```typescript
const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],
});
```

---

## 14. Prisma vs SQL Command Mapping

| SQL Command | Prisma Method | Notes |
| :--- | :--- | :--- |
| `SELECT *` | `findMany()` | Returns all fields. |
| `SELECT col` | `findMany({ select: ... })` | More efficient. |
| `WHERE` | `findMany({ where: ... })` | Supports complex filters. |
| `LIMIT / OFFSET` | `take / skip` | For pagination. |
| `ORDER BY` | `orderBy` | Sort results. |
| `INSERT` | `create` | Single row. |
| `INSERT (Bulk)` | `createMany` | Not supported on all DBs (Supported on Postgres). |
| `UPDATE` | `update` | Update by unique identifier. |
| `UPDATE (Bulk)` | `updateMany` | Update based on filter. |
| `DELETE` | `delete` | Delete by unique identifier. |
| `JOIN` | `include` | Fetches related data. |
| `GROUP BY` | `groupBy` | Aggregations. |

---

## 15. Common Mistakes & Anti-Patterns

1.  **Over-fetching**: Using `include` everywhere. Start with `select` or default, only include relation if used on UI.
2.  **Logic in Loop**: Running database calls inside a `.map()` or `for` loop. Use `Promise.all` or `createMany`.
3.  **Missing DB Constraints**: Relying only on Prisma validation. Always use `@unique` and `@relation` in schema to enforce referential integrity at the DB level.
4.  **Implicit Many-to-Many with Metadata**: Trying to add data to an implicit m-n table. You *must* switch to explicit m-n if you need extra fields on the relation.

---

## 16. Real-World Mini Project: Chat App

### Schema
```prisma
model User {
  id            String         @id @default(uuid())
  username      String         @unique
  conversations Conversation[]
  messages      Message[]
}

model Conversation {
  id        String    @id @default(uuid())
  createdAt DateTime  @default(now())
  users     User[]    // Implicit m-n
  messages  Message[]
}

model Message {
  id             String       @id @default(uuid())
  text           String
  createdAt      DateTime     @default(now())
  senderId       String
  sender         User         @relation(fields: [senderId], references: [id])
  conversationId String
  conversation   Conversation @relation(fields: [conversationId], references: [id])
}
```

### Business Logic: Send Message
We ensure the user is part of the conversation before sending.

```typescript
async function sendMessage(userId: string, conversationId: string, text: string) {
  return await prisma.$transaction(async (tx) => {
    // 1. Verify user is in conversation
    const conversation = await tx.conversation.findFirst({
      where: {
        id: conversationId,
        users: { some: { id: userId } } // Relation filter
      }
    });

    if (!conversation) throw new Error("User not in conversation");

    // 2. Create message
    return await tx.message.create({
      data: {
        text,
        senderId: userId,
        conversationId
      }
    });
  });
}
```

**SQL Equivalent (Conceptual):**
```sql
BEGIN;
-- Check membership
SELECT id FROM "Conversation" c
JOIN "_ConversationToUser" ctu ON c.id = ctu."A"
WHERE c.id = 'conv_id' AND ctu."B" = 'user_id';

-- If exists, Insert
INSERT INTO "Message" ... VALUES ...;
COMMIT;
```

---

## 17. Prisma Command Cheat Sheet

| Command | Description | Context |
| :--- | :--- | :--- |
| `npx prisma init` | Setup Prisma in project. | Initial setup |
| `npx prisma generate` | Generate TS Client based on schema. | Run after every schema change |
| `npx prisma migrate dev` | Create migration & apply to local DB. | Development |
| `npx prisma migrate deploy` | Apply pending migrations. | **Production / CI** |
| `npx prisma db push` | Push schema to DB without migrations. | Prototyping (Avoid in Prod) |
| `npx prisma studio` | Open GUI to view/edit data. | Debugging |
| `npx prisma format` | Auto-format `schema.prisma`. | Linting |
| `npx prisma validate` | Check schema for errors. | CI Check |
