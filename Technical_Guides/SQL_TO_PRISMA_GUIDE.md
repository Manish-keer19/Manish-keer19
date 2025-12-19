# 📚 Complete Guide: Raw SQL to Prisma Schema Mapping

## Table of Contents
1. [Understanding Database Tables vs Prisma Models](#understanding-database-tables-vs-prisma-models)
2. [SQL Syntax Fundamentals](#sql-syntax-fundamentals)
3. [Prisma Schema Syntax](#prisma-schema-syntax)
4. [Mapping SQL to Prisma](#mapping-sql-to-prisma)
5. [Real Examples from Your Schema](#real-examples-from-your-schema)
6. [Advanced Queries](#advanced-queries)

---

## Understanding Database Tables vs Prisma Models

### Visual Representation

```
┌─────────────────────────────────────────────────────────────┐
│                    DATABASE LAYER                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  PostgreSQL Database                                  │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│  │  │  users   │  │  posts   │  │ comments │           │   │
│  │  │  table   │  │  table   │  │  table   │           │   │
│  │  └──────────┘  └──────────┘  └──────────┘           │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕
                    (Prisma Schema)
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                   APPLICATION LAYER                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Prisma Client (TypeScript)                          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│  │  │   User   │  │   Post   │  │ Comment  │           │   │
│  │  │  model   │  │  model   │  │  model   │           │   │
│  │  └──────────┘  └──────────┘  └──────────┘           │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Key Concept**: 
- **SQL** = Direct database operations (raw queries)
- **Prisma Schema** = Blueprint that generates type-safe client code
- **Prisma Client** = Auto-generated TypeScript/JavaScript API

---

## SQL Syntax Fundamentals

### 1. CREATE TABLE (Raw SQL)

```sql
-- Creating a users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    avatar_url TEXT,
    bio TEXT,
    role VARCHAR(50) DEFAULT 'USER',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Creating indexes
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_deleted_at ON users(deleted_at);
```

**SQL Syntax Breakdown**:
```
CREATE TABLE table_name (
    column_name DATA_TYPE [CONSTRAINTS],
    column_name DATA_TYPE [CONSTRAINTS],
    ...
    [TABLE_CONSTRAINTS]
);
```

### 2. Common SQL Data Types

| SQL Type | Description | Example |
|----------|-------------|---------|
| `VARCHAR(n)` | Variable-length string | `VARCHAR(255)` |
| `TEXT` | Unlimited text | `TEXT` |
| `INTEGER` | Whole numbers | `INTEGER` |
| `BIGINT` | Large integers | `BIGINT` |
| `BOOLEAN` | True/False | `BOOLEAN` |
| `TIMESTAMP` | Date and time | `TIMESTAMP` |
| `UUID` | Universal unique ID | `UUID` |
| `ENUM` | Predefined values | `ENUM('USER', 'ADMIN')` |

### 3. SQL Constraints

```sql
-- PRIMARY KEY: Unique identifier for each row
id UUID PRIMARY KEY

-- UNIQUE: No duplicate values allowed
email VARCHAR(255) UNIQUE

-- NOT NULL: Must have a value
password VARCHAR(255) NOT NULL

-- DEFAULT: Default value if none provided
role VARCHAR(50) DEFAULT 'USER'

-- FOREIGN KEY: Reference to another table
author_id UUID REFERENCES users(id)

-- CHECK: Custom validation
age INTEGER CHECK (age >= 18)
```

### 4. Relationships in SQL

```sql
-- One-to-Many: One user has many posts
CREATE TABLE posts (
    id UUID PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    author_id UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    -- Foreign key constraint
    CONSTRAINT fk_author 
        FOREIGN KEY (author_id) 
        REFERENCES users(id)
        ON DELETE CASCADE
);

-- Many-to-Many: Users follow users (junction table)
CREATE TABLE follows (
    follower_id UUID NOT NULL,
    following_id UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(id),
    FOREIGN KEY (following_id) REFERENCES users(id)
);
```

**Relationship Diagram**:
```
┌──────────────┐         ┌──────────────┐
│    users     │         │    posts     │
├──────────────┤         ├──────────────┤
│ id (PK)      │◄────────│ id (PK)      │
│ username     │  1    * │ title        │
│ email        │         │ content      │
│ password     │         │ author_id(FK)│
└──────────────┘         └──────────────┘
   One User              Many Posts
```

---

## Prisma Schema Syntax

### Basic Structure

```prisma
// 1. Generator: What code to generate
generator client {
  provider = "prisma-client-js"
}

// 2. Datasource: Which database to use
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 3. Enums: Predefined values
enum Role {
  USER
  ADMIN
}

// 4. Models: Database tables
model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  
  // Relations
  posts Post[]
}
```

### Prisma Field Attributes

| Attribute | SQL Equivalent | Description |
|-----------|----------------|-------------|
| `@id` | `PRIMARY KEY` | Primary key |
| `@unique` | `UNIQUE` | Unique constraint |
| `@default(value)` | `DEFAULT value` | Default value |
| `@updatedAt` | Trigger/Function | Auto-update timestamp |
| `@relation` | `FOREIGN KEY` | Define relationships |
| `@map("name")` | Column alias | Map to different DB name |
| `@@index([field])` | `CREATE INDEX` | Create index |

### Prisma Data Types

| Prisma Type | SQL Type | Description |
|-------------|----------|-------------|
| `String` | `VARCHAR/TEXT` | Text data |
| `Int` | `INTEGER` | Whole numbers |
| `BigInt` | `BIGINT` | Large integers |
| `Float` | `DECIMAL/NUMERIC` | Decimal numbers |
| `Boolean` | `BOOLEAN` | True/False |
| `DateTime` | `TIMESTAMP` | Date and time |
| `Json` | `JSON/JSONB` | JSON data |
| `Bytes` | `BYTEA` | Binary data |

### Optional vs Required Fields

```prisma
model User {
  id       String   @id @default(uuid())
  username String   // Required (NOT NULL)
  bio      String?  // Optional (NULL allowed) - notice the ?
  email    String   @unique
}
```

**SQL Equivalent**:
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(255) NOT NULL,  -- Required
    bio TEXT,                         -- Optional (no NOT NULL)
    email VARCHAR(255) UNIQUE NOT NULL
);
```

---

## Mapping SQL to Prisma

### Example 1: Simple Table

**SQL**:
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    avatar_url TEXT,
    bio TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_username ON users(username);
```

**Prisma**:
```prisma
model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  avatarUrl String?  // Note: camelCase in Prisma, snake_case in DB
  bio       String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([username])
}
```

**Mapping Diagram**:
```
SQL Column          Prisma Field         Notes
───────────────────────────────────────────────────────────
id                  id                   UUID → String
username            username             VARCHAR → String
email               email                VARCHAR → String
password            password             VARCHAR → String
avatar_url          avatarUrl           TEXT → String? (optional)
bio                 bio                  TEXT → String? (optional)
created_at          createdAt           TIMESTAMP → DateTime
updated_at          updatedAt           Auto-managed by @updatedAt
```

### Example 2: One-to-Many Relationship

**SQL**:
```sql
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    author_id UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_posts_author_created ON posts(author_id, created_at);
```

**Prisma**:
```prisma
model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  authorId  String   // Foreign key field
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // Relation field (not in database, only in Prisma)
  author    User     @relation(fields: [authorId], references: [id])
  
  @@index([authorId, createdAt])
}

model User {
  id        String   @id @default(uuid())
  username  String   @unique
  // ... other fields
  
  // Relation field (one user has many posts)
  posts     Post[]
}
```

**Relationship Diagram**:
```
┌─────────────────────┐              ┌─────────────────────┐
│       User          │              │       Post          │
├─────────────────────┤              ├─────────────────────┤
│ id: String          │◄─────────────│ id: String          │
│ username: String    │   1      *   │ title: String       │
│ email: String       │              │ content: String     │
│                     │              │ authorId: String ───┤
│ posts: Post[]       │              │ author: User        │
└─────────────────────┘              └─────────────────────┘
   (Relation field)                     (Relation field)
   (Not in DB)                          (Not in DB)
```

### Example 3: Many-to-Many Relationship

**SQL**:
```sql
-- Junction table for many-to-many
CREATE TABLE follows (
    follower_id UUID NOT NULL,
    following_id UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (following_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
```

**Prisma**:
```prisma
model Follow {
  followerId  String
  followingId String
  createdAt   DateTime @default(now())
  
  // Relations
  follower    User     @relation("UserFollowers", fields: [followerId], references: [id])
  following   User     @relation("UserFollowing", fields: [followingId], references: [id])
  
  // Composite primary key
  @@id([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}

model User {
  id        String   @id @default(uuid())
  username  String   @unique
  
  // Many-to-many relations
  followers Follow[] @relation("UserFollowers")
  following Follow[] @relation("UserFollowing")
}
```

**Many-to-Many Diagram**:
```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    User      │         │    Follow    │         │    User      │
│  (Follower)  │         │  (Junction)  │         │ (Following)  │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ id           │◄────────│ followerId   │         │ id           │
│ username     │  1    * │ followingId  │*    1   │ username     │
│              │         │ createdAt    │─────────►│              │
│ following[]  │         │              │         │ followers[]  │
└──────────────┘         └──────────────┘         └──────────────┘
```

### Example 4: Enums

**SQL**:
```sql
-- Create enum type
CREATE TYPE role_enum AS ENUM ('USER', 'ADMIN');

CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    role role_enum DEFAULT 'USER'
);
```

**Prisma**:
```prisma
enum Role {
  USER
  ADMIN
}

model User {
  id       String @id @default(uuid())
  username String
  role     Role   @default(USER)
}
```

---

## Real Examples from Your Schema

### Example 1: User Model

**Your Prisma Schema**:
```prisma
model User {
  id        String    @id @default(uuid())
  username  String    @unique
  email     String    @unique
  password  String
  avatarUrl String?
  bio       String?
  role      Role      @default(USER)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  posts    Post[]
  comments Comment[]
  likes    Like[]
  
  followers Follow[] @relation("UserFollowers")
  following Follow[] @relation("UserFollowing")

  @@index([username])
  @@index([deletedAt])
}
```

**Equivalent SQL**:
```sql
-- Create enum first
CREATE TYPE role_enum AS ENUM ('USER', 'ADMIN');

-- Create users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    avatar_url TEXT,
    bio TEXT,
    role role_enum DEFAULT 'USER',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_deleted_at ON users(deleted_at);

-- Note: Relation fields (posts, comments, etc.) are NOT columns in SQL
-- They are virtual fields in Prisma that represent relationships
```

### Example 2: Message & MessageRead (Composite Key)

**Your Prisma Schema**:
```prisma
model Message {
  id             String    @id @default(uuid())
  conversationId String
  senderId       String
  content        String?
  createdAt      DateTime  @default(now())
  deletedAt      DateTime?

  conversation Conversation  @relation("ChatMessages", fields: [conversationId], references: [id])
  sender       User          @relation(fields: [senderId], references: [id])
  reads        MessageRead[]
  attachments  Attachment[]

  @@index([conversationId, createdAt])
}

model MessageRead {
  messageId String
  userId    String
  readAt    DateTime @default(now())

  message Message @relation(fields: [messageId], references: [id])
  user    User    @relation(fields: [userId], references: [id])

  @@id([messageId, userId])
}
```

**Equivalent SQL**:
```sql
-- Messages table
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL,
    sender_id UUID NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP,
    
    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
    FOREIGN KEY (sender_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_messages_conversation_created ON messages(conversation_id, created_at);

-- Message reads table (composite primary key)
CREATE TABLE message_reads (
    message_id UUID NOT NULL,
    user_id UUID NOT NULL,
    read_at TIMESTAMP DEFAULT NOW(),
    
    PRIMARY KEY (message_id, user_id),  -- Composite key
    FOREIGN KEY (message_id) REFERENCES messages(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Composite Key Diagram**:
```
MessageRead Table
┌─────────────┬─────────┬────────────────┐
│ message_id  │ user_id │ read_at        │
│   (PK)      │  (PK)   │                │
├─────────────┼─────────┼────────────────┤
│ msg-001     │ user-1  │ 2024-01-01     │ ✓ Valid
│ msg-001     │ user-2  │ 2024-01-02     │ ✓ Valid
│ msg-001     │ user-1  │ 2024-01-03     │ ✗ Duplicate (same PK combo)
└─────────────┴─────────┴────────────────┘
```

### Example 3: Self-Referencing (Comment Tree)

**Your Prisma Schema**:
```prisma
model Comment {
  id        String    @id @default(uuid())
  text      String
  postId    String
  userId    String
  parentId  String?   // Self-reference for nested comments
  createdAt DateTime  @default(now())
  deletedAt DateTime?

  post Post @relation(fields: [postId], references: [id])
  user User @relation(fields: [userId], references: [id])

  parent  Comment?  @relation("CommentTree", fields: [parentId], references: [id])
  replies Comment[] @relation("CommentTree")

  @@index([postId, createdAt])
  @@index([parentId])
}
```

**Equivalent SQL**:
```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    text TEXT NOT NULL,
    post_id UUID NOT NULL,
    user_id UUID NOT NULL,
    parent_id UUID,  -- Self-referencing foreign key
    created_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP,
    
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (parent_id) REFERENCES comments(id) ON DELETE CASCADE
);

CREATE INDEX idx_comments_post_created ON comments(post_id, created_at);
CREATE INDEX idx_comments_parent ON comments(parent_id);
```

**Comment Tree Diagram**:
```
┌─────────────────────────────────────────────┐
│ Post: "How to learn Prisma?"                │
└─────────────────────────────────────────────┘
         │
         ├─► Comment 1 (parentId: null)
         │   "Great question!"
         │        │
         │        ├─► Comment 2 (parentId: comment-1)
         │        │   "I agree!"
         │        │
         │        └─► Comment 3 (parentId: comment-1)
         │            "Me too!"
         │
         └─► Comment 4 (parentId: null)
             "Check the docs"
                  │
                  └─► Comment 5 (parentId: comment-4)
                      "Thanks!"
```

---

## Advanced Queries

### 1. JOIN Queries

**SQL - Get all posts with author info**:
```sql
SELECT 
    posts.id,
    posts.title,
    posts.content,
    posts.created_at,
    users.username,
    users.email
FROM posts
INNER JOIN users ON posts.author_id = users.id
WHERE posts.deleted_at IS NULL
ORDER BY posts.created_at DESC
LIMIT 10;
```

**Prisma Equivalent**:
```typescript
const posts = await prisma.post.findMany({
  where: {
    deletedAt: null
  },
  include: {
    author: {
      select: {
        username: true,
        email: true
      }
    }
  },
  orderBy: {
    createdAt: 'desc'
  },
  take: 10
});
```

### 2. Aggregation Queries

**SQL - Count posts per user**:
```sql
SELECT 
    users.id,
    users.username,
    COUNT(posts.id) as post_count
FROM users
LEFT JOIN posts ON users.id = posts.author_id
GROUP BY users.id, users.username
HAVING COUNT(posts.id) > 5
ORDER BY post_count DESC;
```

**Prisma Equivalent**:
```typescript
const usersWithPostCount = await prisma.user.findMany({
  select: {
    id: true,
    username: true,
    _count: {
      select: {
        posts: true
      }
    }
  },
  where: {
    posts: {
      some: {}  // Has at least one post
    }
  },
  orderBy: {
    posts: {
      _count: 'desc'
    }
  }
});

// Filter in application code for count > 5
const filtered = usersWithPostCount.filter(u => u._count.posts > 5);
```

### 3. Complex Nested Query

**SQL - Get conversation with messages and read status**:
```sql
SELECT 
    c.id as conversation_id,
    c.name,
    m.id as message_id,
    m.content,
    m.created_at,
    u.username as sender_name,
    mr.read_at
FROM conversations c
LEFT JOIN messages m ON c.id = m.conversation_id
LEFT JOIN users u ON m.sender_id = u.id
LEFT JOIN message_reads mr ON m.id = mr.message_id AND mr.user_id = 'current-user-id'
WHERE c.id = 'conversation-id'
ORDER BY m.created_at DESC;
```

**Prisma Equivalent**:
```typescript
const conversation = await prisma.conversation.findUnique({
  where: {
    id: 'conversation-id'
  },
  include: {
    messages: {
      include: {
        sender: {
          select: {
            username: true
          }
        },
        reads: {
          where: {
            userId: 'current-user-id'
          }
        }
      },
      orderBy: {
        createdAt: 'desc'
      }
    }
  }
});
```

### 4. Transaction Example

**SQL**:
```sql
BEGIN;

-- Create a post
INSERT INTO posts (id, title, content, author_id, created_at)
VALUES (gen_random_uuid(), 'My Post', 'Content here', 'user-id', NOW())
RETURNING id;

-- Update user's post count
UPDATE post_stats 
SET post_count = post_count + 1 
WHERE user_id = 'user-id';

COMMIT;
```

**Prisma Equivalent**:
```typescript
const result = await prisma.$transaction(async (tx) => {
  // Create post
  const post = await tx.post.create({
    data: {
      title: 'My Post',
      content: 'Content here',
      authorId: 'user-id'
    }
  });

  // Update stats
  await tx.postStats.update({
    where: { postId: post.id },
    data: {
      commentCount: { increment: 1 }
    }
  });

  return post;
});
```

---

## Quick Reference: SQL to Prisma Cheat Sheet

| Operation | SQL | Prisma |
|-----------|-----|--------|
| **Create** | `INSERT INTO users (...) VALUES (...)` | `prisma.user.create({ data: {...} })` |
| **Read One** | `SELECT * FROM users WHERE id = ?` | `prisma.user.findUnique({ where: { id } })` |
| **Read Many** | `SELECT * FROM users WHERE role = ?` | `prisma.user.findMany({ where: { role } })` |
| **Update** | `UPDATE users SET ... WHERE id = ?` | `prisma.user.update({ where: { id }, data: {...} })` |
| **Delete** | `DELETE FROM users WHERE id = ?` | `prisma.user.delete({ where: { id } })` |
| **Join** | `SELECT * FROM posts JOIN users ON ...` | `prisma.post.findMany({ include: { author: true } })` |
| **Count** | `SELECT COUNT(*) FROM users` | `prisma.user.count()` |
| **Order** | `ORDER BY created_at DESC` | `orderBy: { createdAt: 'desc' }` |
| **Limit** | `LIMIT 10` | `take: 10` |
| **Offset** | `OFFSET 20` | `skip: 20` |

---

## Summary

### Key Takeaways:

1. **SQL is the foundation** - Understanding SQL helps you understand what Prisma does under the hood
2. **Prisma is type-safe** - It generates TypeScript types from your schema
3. **Relations are different** - In SQL, you use foreign keys. In Prisma, you define relation fields
4. **Naming conventions** - SQL uses `snake_case`, Prisma uses `camelCase`
5. **Virtual fields** - Relation fields like `posts Post[]` don't exist in the database

### Workflow:

```
1. Design Database Schema (SQL thinking)
   ↓
2. Write Prisma Schema (schema.prisma)
   ↓
3. Run: npx prisma migrate dev
   ↓
4. Prisma generates SQL migrations
   ↓
5. Database tables created
   ↓
6. Run: npx prisma generate
   ↓
7. Prisma Client generated (TypeScript)
   ↓
8. Use Prisma Client in your code
```

### Next Steps:

1. ✅ Understand this guide
2. ✅ Practice writing simple models
3. ✅ Learn to map SQL queries to Prisma queries
4. ✅ Master relationships (1:1, 1:N, N:M)
5. ✅ Learn transactions and advanced queries
6. ✅ Build real-world features

---

**Remember**: Prisma is an abstraction over SQL. The better you understand SQL, the better you'll use Prisma! 🚀
