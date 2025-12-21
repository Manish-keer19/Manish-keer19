# Full Backend Mastery Roadmap: ChatApp & Social Platform

This roadmap is designed so you **actually understand what you are building**, not just make CRUD APIs. Follow the phases strictly in order. Do NOT jump to Prisma or NestJS early.

**Project Context**: You are building a Social Media App with Chat (Private/Group), Feed, Recursive Comments, and Media.

---

## Phase 0 – Foundation & Engineering Mindset

### Goals
* Think like a backend engineer, not a framework user.
* Understand data before code.
* **Mantra**: "The database is the source of truth. The API is just a gatekeeper."

### What to Set Up
* **PostgreSQL** (v14+ is recommended).
* **pgAdmin** or **DBeaver** (for visualizing ERDs and running raw queries).
* **Node.js LTS**.
* **Git** (Commit often: "feat: implemented user schema sql").

### Concepts to Understand
* **Database-first development**: Define schemas -> Write SQL -> Then write API.
* **Why schemas matter**: A bad schema requires 10x more code to fix in the backend.
* **ORM Reality**: Prisma is great, but it generates SQL. If you don't know the SQL it generates, you can't optimize it.

---

## Phase 1 – Database Fundamentals (RAW SQL ONLY)

### What You Learn
* Relational database theory.
* Data integrity and constraints.

### Concepts (Mapped to Your Schema)
* **Primary Keys**:
    * **UUID** (`User.id`, `Post.id`): Why? Security (no enumeration), distributed systems compatible.
    * **Composite Keys** (`Follow`, `ConversationUser`): Why? A user can only follow another user once. `(followerId, followingId)` MUST be unique together.
* **Foreign Keys**:
    * `User` -> `Post` (One-to-Many).
    * `User` -> `Follow` -> `User` (Self-referential Many-to-Many).
    * `Conversation` -> `Message` (One-to-Many).
* **ENUMs**: `Role` ('USER', 'ADMIN'), `ConversationType`, `MediaType`.
* **Soft Deletes**: usage of `deletedAt`. Records are never actually removed, just hidden.
* **Normalization**:
    * **1NF**: No array of `userIds` in `Conversation`. Used `ConversationUser` join table instead.

### Exercises
1.  Draw the **Entity Relationship Diagram (ERD)** on paper.
2.  List every Foreign Key constraint needed (e.g., `ON DELETE CASCADE` for Messages when a Conversation is deleted).
3.  Explain why `ConversationUser` needs a composite Primary Key `(conversationId, userId)`.

---

## Phase 2 – Build the Entire Database in RAW SQL

**Goal**: Write the `schema.sql` file manually.

### Step-by-Step

#### 1. Create ENUMs
```sql
CREATE TYPE "Role" AS ENUM ('USER', 'ADMIN');
CREATE TYPE "ConversationType" AS ENUM ('PRIVATE', 'GROUP');
CREATE TYPE "MediaType" AS ENUM ('IMAGE', 'VIDEO', 'FILE', 'AUDIO');
```

#### 2. Core Tables (Write CREATE TABLE statements)
*   **Users**: UUID PK, Unique Email/Username.
*   **Follows**: Composite PK `(followerId, followingId)`.
*   **Conversations**: `lastMessageId` (optimization).
*   **ConversationUsers**: The join table `(conversationId, userId)`.
*   **Messages**: `senderId`, `conversationId`.
*   **MessageReads**: Track who read what.
*   **Posts & Comments**: Comments need `parentId` (Self-relation) for threads.
*   **Likes**: `(postId, userId)` Composite PK.
*   **PostStats**: `(postId)` PK, counts for caching.

#### 3. Constraints
*   **UNIQUE**: `email`, `username`.
*   **CHECK**: Ensure `followerId != followingId` (Prevent self-follow).
*   **FK Cascades**: If User is deleted, should their Likes be deleted? (Yes, `ON DELETE CASCADE`).

#### 4. Indexing (Critical for Speed)
*   `CREATE INDEX ON "Message" ("conversationId", "createdAt" DESC);` (Get chat history fast).
*   `CREATE INDEX ON "Post" ("authorId", "createdAt" DESC);` (Get user profile feed).
*   `CREATE INDEX ON "Comment" ("postId");` (Load comments for a post).

### Mandatory Practice
*   Write 5 `INSERT` statements for Users.
*   Write a `DELETE` that fails because of a Foreign Key constraint. Correct it.

---

## Phase 3 – Advanced SQL (Critical Phase)

### Real Queries You Must Write (Save these as .sql files)

#### 1. The "News Feed" Query
Fetch posts from users I follow, sorted by time.
```sql
SELECT p.*, u.username, u."avatarUrl"
FROM "Post" p
JOIN "Follow" f ON p."authorId" = f."followingId"
JOIN "User" u ON p."authorId" = u.id
WHERE f."followerId" = :myUserId
ORDER BY p."createdAt" DESC
LIMIT 20;
```

#### 2. Recursive Comment Tree (CTEs)
Fetch a comment + its replies + replies of replies.
```sql
WITH RECURSIVE CommentTree AS (
    SELECT *, 0 as depth FROM "Comment" WHERE "postId" = :postId AND "parentId" IS NULL
    UNION ALL
    SELECT c.*, ct.depth + 1
    FROM "Comment" c
    INNER JOIN CommentTree ct ON c."parentId" = ct.id
)
SELECT * FROM CommentTree ORDER BY "createdAt" ASC;
```

#### 3. Chat Analytics
Unread message count per conversation.
```sql
SELECT m."conversationId", COUNT(m.id) as unread_count
FROM "Message" m
LEFT JOIN "MessageRead" mr ON m.id = mr."messageId" AND mr."userId" = :myUserId
WHERE mr."readAt" IS NULL
GROUP BY m."conversationId";
```

### Advanced Concepts
*   **Transactions**: Wrap message sending + updating `lastMessageId` in `BEGIN ... COMMIT`.
*   **EXPLAIN ANALYZE**: Run it on the Feed query. Did it use the Index?

---

## Phase 4 – Prisma (AFTER SQL MASTERY)

### What You Learn
*   How Prisma acts as a bridge between your App and SQL.
*   The `schema.prisma` file is your new source of truth.

### Tasks
1.  **Introspection**: If you built the DB in Phase 2, try `npx prisma db pull`.
2.  **Relations**: Define `@relation` fields. Understand `fields` vs `references`.
3.  **Migrations**: Run `npx prisma migrate dev --name init`. Compare the generated SQL folder with your hand-written SQL.
4.  **Prisma Client**:
    *   `findUnique` vs `findFirst`.
    *   **Transactions**: `prisma.$transaction([ ... ])`.
    *   **Raw SQL**: Use `prisma.$queryRaw` for the Recursive Comment CTE (Prisma doesn't support recursive queries well natively yet).

### Rules
*   Do NOT use Prisma filters for everything. If the query helps Analytics (e.g., "Top 10 posts past hour"), use Raw SQL or Views.

---

## Phase 5 – NestJS Architecture (Enterprise Level)

### Folder Structure
Each feature gets a Module, Controller, Service.
```
src/
├── common/ (Guards, Interceptors, Decorators)
├── modules/
│   ├── auth/ (JWT, Login, Register)
│   ├── users/ (Profile, Search)
│   ├── social/ (Follows, Block)
│   ├── feed/ (Posts aggregation)
│   ├── content/ (Posts, Comments management)
│   ├── chat/ (Conversations, Gateway, Messages)
│   ├── analytics/ (PostStats)
│   └── media/ (S3/Cloudinary upload)
```

### Core Concepts
*   **Guards**: `JwtAuthGuard` (Global), `RolesGuard` (Admin only).
*   **Interceptors**: `TransformInterceptor` (Standardize API response `{ data: ... }`).
*   **PrismaService**: Create a singleton module.
*   **DTOs**: Strict validation using `class-validator` (e.g., `@IsUUID()`, `@Length(1, 500)`).

---

## Phase 6 – Business Logic Enforcement

### Must-Have Rules
*   **Duplicate Likes**: A user cannot like the same Post twice. Handle unique constraint violation errors from Prisma (`P2002`).
*   **Chat Security**: Use a Guard or Service check. `if (!conversation.users.includes(currentUser))` -> Throw 403.
*   **Group Permissions**: Only the `ownerId` can delete the Group.
*   **Message Read Receipts**: When a user opens a chat, bulk insert into `MessageRead`.

---

## Phase 7 – Performance & Scaling

### Topics
*   **N+1 Problem**:
    *   *Bad*: Loop through posts, find author for each.
    *   *Good*: `prisma.post.findMany({ include: { author: true } })`.
*   **Pagination**:
    *   Implement **Cursor-based pagination** for Chat Messages (`take: 20, cursor: { id: lastMsgId }`). Offset is too slow for millions of messages.
*   **Query Operations**: Separate Read logic (Feed) from Write logic (Posting).

---

## Phase 8 – Testing & Validation

### Strategy
1.  **Unit Tests**: Mock `PrismaService`. Test business logic (e.g., "Calculate Post Score").
2.  **Integration Tests**: Use a **Dockerized Test DB**. E2E test the "Signup -> Login -> Create Post" flow.
3.  **Edge Cases**:
    *   What if a user is deleted? (Check `deletedAt`).
    *   What if 1000 users like a post at the exact same second? (Test Transaction locks).

---

## Phase 9 – Production Readiness

### Must Know
*   **Migrations**: Never `prisma db push` in prod. Use `prisma migrate deploy`.
*   **Health Checks**: Add a `/health` endpoint.
*   **Logging**: Use `nestjs-pino` or functional logging tailored to track strict request IDs.
*   **Backup**: How to `pg_dump` and restore.

---

## Phase 10 – Mastery Checklist

You are DONE only if you can:
- [ ] Write the `schema.sql` for this app from memory.
- [ ] Explain the difference between B-Tree and Hash indexes.
- [ ] Write a Recursive SQL query for comments.
- [ ] Explain how NestJS Dependency Injection works.
- [ ] Debug a slow Query Plan using `EXPLAIN ANALYZE`.
- [ ] Implement Cursor Pagination without a library.

---

## Final Rule
**If you skip RAW SQL → you will never master backend engineering.**
Follow the phases exactly.
