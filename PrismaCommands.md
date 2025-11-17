# Complete Prisma CLI Commands Cheatsheet

A comprehensive guide to all Prisma CLI commands from beginner to advanced usage with real-world examples and best practices.

---

## Table of Contents

1. [Installation & Setup](#installation--setup)
2. [Initialization Commands](#initialization-commands)
3. [Schema Management](#schema-management)
4. [Database Commands](#database-commands)
5. [Migration Commands](#migration-commands)
6. [Prisma Client](#prisma-client)
7. [Prisma Studio](#prisma-studio)
8. [Complete Workflows](#complete-workflows)
9. [Quick Reference](#quick-reference)
10. [Best Practices](#best-practices)

---

## Installation & Setup

### Install Prisma CLI

```bash
# Install as dev dependency (recommended)
npm install prisma --save-dev

# Install Prisma Client
npm install @prisma/client

# Install globally (optional)
npm install -g prisma

# Using Yarn
yarn add prisma --dev
yarn add @prisma/client

# Using pnpm
pnpm add -D prisma
pnpm add @prisma/client
```

### Check Prisma Version

```bash
prisma --version
# or
npx prisma --version
```

**Output Example:**
```
prisma                  : 5.22.0
@prisma/client          : 5.22.0
Computed binaryTarget   : darwin-arm64
Operating System        : darwin
Node.js                 : v20.11.0
```

---

## Initialization Commands

### `prisma init`

Initialize a new Prisma project with default configuration.

| Command | Description | Use Case |
|---------|-------------|----------|
| `prisma init` | Initialize with PostgreSQL (default) | Starting new project |
| `prisma init --datasource-provider postgresql` | Initialize with PostgreSQL | Production apps |
| `prisma init --datasource-provider mysql` | Initialize with MySQL | MySQL databases |
| `prisma init --datasource-provider sqlite` | Initialize with SQLite | Local development, testing |
| `prisma init --datasource-provider mongodb` | Initialize with MongoDB | NoSQL applications |
| `prisma init --datasource-provider sqlserver` | Initialize with SQL Server | Microsoft environments |
| `prisma init --datasource-provider cockroachdb` | Initialize with CockroachDB | Distributed databases |

**What it creates:**
- `prisma/schema.prisma` - Main Prisma schema file
- `.env` - Environment variables file with `DATABASE_URL`

**Example Project Structure:**
```
project/
├── prisma/
│   └── schema.prisma
├── .env
└── package.json
```

**Generated schema.prisma:**

```prisma
// This is your Prisma schema file

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

**Generated .env:**

```env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb?schema=public"
```

---

## Schema Management

### `prisma format`

Auto-format the Prisma schema file according to Prisma's style guidelines.

```bash
# Format default schema
prisma format

# Format custom schema path
prisma format --schema=./custom/path/schema.prisma
```

**Use Cases:**
- Format before committing code
- Maintain consistent code style
- Automatically fix indentation and spacing

**Before formatting:**
```prisma
model User{
id Int @id @default(autoincrement())
email String @unique
  name String?
posts Post[]
}
```

**After formatting:**
```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

---

### `prisma validate`

Validate the Prisma schema for syntax errors and logical issues.

```bash
# Validate default schema
prisma validate

# Validate custom schema
prisma validate --schema=./prisma/schema.prisma
```

**Use Cases:**
- Run in CI/CD pipelines to catch schema errors
- Validate before running migrations
- Check for breaking changes before deployment

**Example Validation Errors:**

```bash
# Missing @id attribute
Error: Field `id` must be marked with @id attribute

# Invalid relation
Error: The relation field `author` on Model `Post` is missing an opposite relation field on model `User`

# Invalid type
Error: Type `Strings` is neither a built-in type, nor refers to another model, custom type, or enum
```

---

### `prisma generate`

Generate Prisma Client based on the schema. **Required after schema changes.**

```bash
# Generate Prisma Client
prisma generate

# Custom schema path
prisma generate --schema=./custom/schema.prisma

# Watch mode (regenerate on changes)
prisma generate --watch
```

**What it generates:**

```
node_modules/
└── .prisma/
    └── client/
        ├── index.js
        ├── index.d.ts
        └── ... (generated files)
```

**When to run:**
- After every schema change
- After pulling code with schema updates
- Before building for production
- In CI/CD pipelines before deployment

**Note:** Commands like `migrate dev` and `db push` automatically run `generate`.

---

## Database Commands

### `prisma db push`

Push schema changes directly to the database **without creating migrations**. Perfect for rapid prototyping.

```bash
# Push schema to database
prisma db push

# Skip generating Prisma Client
prisma db push --skip-generate

# Accept data loss warnings automatically
prisma db push --accept-data-loss

# Force reset database
prisma db push --force-reset
```

**Example Schema:**

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  role      String   @default("USER")
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
}
```

**Use Cases:**
- ✅ Local development and rapid prototyping
- ✅ Quick schema iterations without migration history
- ✅ Testing schema designs
- ✅ Setting up test/seed databases
- ❌ Production environments (use migrations instead)

**⚠️ Warning:** Does not create migration history. Use `prisma migrate dev` for production workflows.

---

### `prisma db pull` (formerly introspect)

Introspect an existing database and generate a Prisma schema from it.

```bash
# Pull schema from database
prisma db pull

# Force overwrite existing schema
prisma db pull --force

# Print schema to console without writing
prisma db pull --print
```

**Use Cases:**
- Adding Prisma to existing projects with databases
- Reverse-engineering legacy databases
- Syncing schema after manual database changes
- Importing from existing SQL databases

**Example - Existing Database:**

```sql
-- Existing PostgreSQL tables
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  published BOOLEAN DEFAULT false,
  author_id INTEGER REFERENCES users(id)
);
```

**After `prisma db pull`:**

```prisma
model users {
  id         Int       @id @default(autoincrement())
  email      String    @unique @db.VarChar(255)
  name       String?   @db.VarChar(255)
  created_at DateTime? @default(now()) @db.Timestamp(6)
  posts      posts[]
}

model posts {
  id         Int      @id @default(autoincrement())
  title      String   @db.VarChar(255)
  content    String?
  published  Boolean? @default(false)
  author_id  Int?
  users      users?   @relation(fields: [author_id], references: [id])
}
```

---

### `prisma db seed`

Seed the database with initial data using a custom seed script.

```bash
# Run seed script
prisma db seed

# Seed is also run after:
# - prisma migrate reset
# - prisma migrate dev (first time)
```

**Setup in package.json:**

```json
{
  "name": "my-project",
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "prisma": "^5.0.0",
    "ts-node": "^10.9.0",
    "typescript": "^5.0.0"
  }
}
```

**Example seed.ts:**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Clean existing data
  await prisma.post.deleteMany();
  await prisma.user.deleteMany();

  // Create users with posts
  const alice = await prisma.user.create({
    data: {
      email: 'alice@prisma.io',
      name: 'Alice',
      role: 'ADMIN',
      posts: {
        create: [
          {
            title: 'Getting Started with Prisma',
            content: 'Prisma is a next-generation ORM...',
            published: true
          },
          {
            title: 'Advanced Prisma Patterns',
            content: 'Learn advanced techniques...',
            published: false
          }
        ]
      }
    },
    include: { posts: true }
  });

  const bob = await prisma.user.create({
    data: {
      email: 'bob@prisma.io',
      name: 'Bob',
      role: 'USER',
      posts: {
        create: [
          {
            title: 'Database Design Best Practices',
            content: 'Learn how to design efficient databases...',
            published: true
          }
        ]
      }
    },
    include: { posts: true }
  });

  console.log({ alice, bob });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**Use Cases:**
- Populate development databases
- Create test data
- Set up demo environments
- Initialize reference data

---

### `prisma db execute`

Execute raw SQL statements against the database.

```bash
# Execute SQL from file
prisma db execute --file=./scripts/seed.sql

# Execute from stdin
prisma db execute --stdin < ./scripts/migration.sql

# Custom schema location
prisma db execute --file=./reset.sql --schema=./prisma/schema.prisma
```

**Example SQL file (seed.sql):**

```sql
-- Insert sample data
INSERT INTO "User" (email, name, role) VALUES
  ('john@example.com', 'John Doe', 'USER'),
  ('jane@example.com', 'Jane Smith', 'ADMIN'),
  ('bob@example.com', 'Bob Johnson', 'USER');

INSERT INTO "Post" (title, content, published, "authorId") VALUES
  ('First Post', 'Hello World!', true, 1),
  ('Second Post', 'More content here', false, 1),
  ('Another Post', 'Even more content', true, 2);

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_user_email ON "User"(email);
CREATE INDEX IF NOT EXISTS idx_post_author ON "Post"("authorId");
CREATE INDEX IF NOT EXISTS idx_post_published ON "Post"(published);
```

**Execute:**

```bash
prisma db execute --file=./seed.sql
```

**Use Cases:**
- Run custom SQL scripts
- Database maintenance tasks
- Complex queries not supported by Prisma
- Bulk data operations
- Custom migrations with advanced SQL

---

## Migration Commands

### `prisma migrate dev`

Create and apply migrations in development. **Primary command for local development.**

```bash
# Create and apply migration
prisma migrate dev

# Name the migration
prisma migrate dev --name add_user_role

# Create migration without applying
prisma migrate dev --create-only

# Skip seed script
prisma migrate dev --skip-seed

# Skip generating Prisma Client
prisma migrate dev --skip-generate
```

**Development Workflow:**

```prisma
// 1. Update your schema
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  role      String   @default("USER") // ← New field added
  createdAt DateTime @default(now())
}
```

```bash
# 2. Run migrate dev
prisma migrate dev --name add_user_role
```

**Creates migration structure:**

```
prisma/migrations/
└── 20250117120000_add_user_role/
    └── migration.sql
```

**Generated migration.sql:**

```sql
-- AlterTable
ALTER TABLE "User" ADD COLUMN "role" TEXT NOT NULL DEFAULT 'USER';
```

**What `migrate dev` does:**
1. ✅ Creates migration file in `prisma/migrations/`
2. ✅ Applies migration to database
3. ✅ Generates Prisma Client
4. ✅ Runs seed script (if configured)
5. ✅ Updates `_prisma_migrations` table

**Use Cases:**
- Daily development workflow
- Adding new models/fields
- Modifying existing schema
- Team collaboration with version-controlled migrations

---

### `prisma migrate deploy`

Apply pending migrations in production/staging environments. **Production-safe command.**

```bash
# Deploy all pending migrations
prisma migrate deploy

# Preview without executing
prisma migrate deploy --preview
```

**Critical Rules:**
- ✅ **Always use in production/staging**
- ❌ **Never use `migrate dev` in production**
- ✅ **Test in staging first**
- ✅ **Backup database before deployment**

**Use Cases:**
- CI/CD pipeline deployments
- Production releases
- Staging environment updates
- Automated deployment scripts

**Example CI/CD Workflow (.github/workflows/deploy.yml):**

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Generate Prisma Client
        run: npx prisma generate
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
      
      - name: Run database migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
      
      - name: Build application
        run: npm run build
      
      - name: Deploy to server
        run: npm run deploy
```

---

### `prisma migrate reset`

Reset the database and re-apply all migrations. **Destructive operation - deletes all data!**

```bash
# Reset database completely
prisma migrate reset

# Skip confirmation prompt
prisma migrate reset --force

# Skip seed script
prisma migrate reset --skip-seed

# Skip generating Prisma Client
prisma migrate reset --skip-generate
```

**What it does:**
1. 🗑️ Drops the database (or all tables)
2. 🆕 Creates a fresh database
3. 📝 Applies all migrations from scratch
4. 🌱 Runs seed script
5. ⚙️ Generates Prisma Client

**Use Cases:**
- Resetting local development database
- Starting fresh with clean data
- Fixing corrupted migration history
- Testing full migration flow

**⚠️ WARNING:** Deletes ALL data. **Never use in production!**

---

### `prisma migrate status`

Check the status of migrations (applied vs pending).

```bash
# Check migration status
prisma migrate status
```

**Example Output (Up to date):**

```
Database schema is up to date!

The following migrations are applied:

20250101120000_init
20250105093000_add_user_profile
20250110140000_add_post_categories
20250115180000_add_comments
```

**Example Output (Pending migrations):**

```
Your database is not up to date!

The following migrations have not yet been applied:

20250117120000_add_user_roles
20250117150000_add_tags

To apply migrations in development, run:
  prisma migrate dev

To apply migrations in production/staging, run:
  prisma migrate deploy
```

**Use Cases:**
- Verify migration state before deployment
- Debug migration issues
- Check database sync with codebase
- CI/CD health checks

---

### `prisma migrate diff`

Compare two database schemas and generate a SQL diff.

```bash
# Compare schema file to actual database
prisma migrate diff \
  --from-schema-datamodel=./prisma/schema.prisma \
  --to-schema-datasource=./prisma/schema.prisma \
  --script

# Compare two different databases
prisma migrate diff \
  --from-url="postgresql://user:pass@localhost:5432/db1" \
  --to-url="postgresql://user:pass@localhost:5432/db2" \
  --script

# Compare migrations folder to database
prisma migrate diff \
  --from-migrations=./prisma/migrations \
  --to-schema-datasource=./prisma/schema.prisma \
  --script

# Output as JSON instead of SQL
prisma migrate diff \
  --from-schema-datamodel=./prisma/schema.prisma \
  --to-schema-datasource=./prisma/schema.prisma
```

**Use Cases:**
- Detect schema drift between environments
- Preview changes before creating migrations
- Compare production vs staging schemas
- Debug migration issues
- Verify database consistency

**Example Output:**

```sql
-- CreateTable
CREATE TABLE "Comment" (
  "id" SERIAL PRIMARY KEY,
  "content" TEXT NOT NULL,
  "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "postId" INTEGER NOT NULL,
  "userId" INTEGER NOT NULL
);

-- CreateIndex
CREATE INDEX "Comment_postId_idx" ON "Comment"("postId");

-- AddForeignKey
ALTER TABLE "Comment" ADD CONSTRAINT "Comment_postId_fkey" 
  FOREIGN KEY ("postId") REFERENCES "Post"("id") ON DELETE CASCADE ON UPDATE CASCADE;

-- AddForeignKey
ALTER TABLE "Comment" ADD CONSTRAINT "Comment_userId_fkey" 
  FOREIGN KEY ("userId") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE;
```

---

### `prisma migrate resolve`

Mark a migration as applied or rolled back without actually executing it. **Advanced troubleshooting command.**

```bash
# Mark migration as applied
prisma migrate resolve --applied 20250117120000_migration_name

# Mark migration as rolled back
prisma migrate resolve --rolled-back 20250117120000_migration_name
```

**Use Cases:**
- Fix migration history inconsistencies
- Handle failed migrations that were fixed manually
- Sync migration state after manual database changes
- Recovery from corrupted migration state

**Example Scenario:**

```bash
# Scenario: Migration failed midway, you fixed it manually in the database
# Now sync the migration state:

prisma migrate resolve --applied 20250117120000_add_indexes

# Output:
# Migration 20250117120000_add_indexes marked as applied.
```

**⚠️ Use with caution:** Only use when you're certain the database state matches the migration.

---

## Prisma Client

### Prisma Client Generation

Prisma Client is auto-generated based on your schema and provides type-safe database access.

**Basic Usage:**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Use prisma throughout your application
```

---

### CRUD Operations Examples

#### Create

```typescript
// Create single user
const user = await prisma.user.create({
  data: {
    email: 'alice@prisma.io',
    name: 'Alice'
  }
});

// Create user with related posts
const userWithPosts = await prisma.user.create({
  data: {
    email: 'bob@prisma.io',
    name: 'Bob',
    posts: {
      create: [
        { title: 'First Post', published: true },
        { title: 'Second Post', published: false }
      ]
    }
  },
  include: {
    posts: true
  }
});

// Create many
await prisma.user.createMany({
  data: [
    { email: 'user1@example.com', name: 'User 1' },
    { email: 'user2@example.com', name: 'User 2' },
    { email: 'user3@example.com', name: 'User 3' }
  ]
});
```

#### Read

```typescript
// Find unique
const user = await prisma.user.findUnique({
  where: { email: 'alice@prisma.io' }
});

// Find first
const firstUser = await prisma.user.findFirst({
  where: { name: 'Alice' }
});

// Find many with filters
const users = await prisma.user.findMany({
  where: {
    email: { contains: 'prisma' },
    posts: { some: { published: true } }
  },
  include: {
    posts: true
  },
  orderBy: {
    createdAt: 'desc'
  },
  take: 10, // Limit
  skip: 0   // Offset
});

// Count records
const userCount = await prisma.user.count({
  where: { role: 'USER' }
});
```

#### Update

```typescript
// Update single
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: { 
    name: 'Alice Updated',
    role: 'ADMIN'
  }
});

// Update many
await prisma.user.updateMany({
  where: { role: 'USER' },
  data: { role: 'MEMBER' }
});

// Upsert (update or create)
const upsertedUser = await prisma.user.upsert({
  where: { email: 'charlie@prisma.io' },
  update: { name: 'Charlie Updated' },
  create: { 
    email: 'charlie@prisma.io',
    name: 'Charlie'
  }
});
```

#### Delete

```typescript
// Delete single
await prisma.user.delete({
  where: { id: 1 }
});

// Delete many
await prisma.user.deleteMany({
  where: { role: 'USER' }
});

// Delete all
await prisma.user.deleteMany();
```

---

### Advanced Queries

#### Transactions

```typescript
// Interactive transactions
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'user@example.com', name: 'User' }
  });
  
  const post = await tx.post.create({
    data: {
      title: 'New Post',
      authorId: user.id
    }
  });
  
  return { user, post };
});

// Sequential operations
await prisma.$transaction([
  prisma.user.create({ data: { email: 'user1@example.com' } }),
  prisma.user.create({ data: { email: 'user2@example.com' } }),
  prisma.post.create({ data: { title: 'Post', authorId: 1 } })
]);
```

#### Raw SQL

```typescript
// Raw query
const users = await prisma.$queryRaw`
  SELECT * FROM "User" 
  WHERE email LIKE ${`%prisma%`}
`;

// Raw execute
await prisma.$executeRaw`
  UPDATE "User" 
  SET role = 'ADMIN' 
  WHERE email = ${'admin@example.com'}
`;

// Unsafe raw queries (use with caution)
const result = await prisma.$queryRawUnsafe(
  'SELECT * FROM "User" WHERE id = $1',
  userId
);
```

#### Aggregations

```typescript
// Aggregate
const aggregations = await prisma.user.aggregate({
  _count: { id: true },
  _avg: { id: true },
  _sum: { id: true },
  _min: { id: true },
  _max: { id: true }
});

// Group by
const groupedUsers = await prisma.user.groupBy({
  by: ['role'],
  _count: { id: true },
  having: {
    id: { _count: { gt: 10 } }
  }
});
```

---

## Prisma Studio

Visual database browser and editor for easy data management.

```bash
# Start Prisma Studio (default: http://localhost:5555)
prisma studio

# Custom port
prisma studio --port 3000

# Custom browser
prisma studio --browser chrome
prisma studio --browser firefox

# Custom schema location
prisma studio --schema=./custom/schema.prisma
```

**Features:**
- 📊 Visual data browser for all tables
- ✏️ CRUD operations via GUI
- 🔗 Navigate through relationships easily
- 🔍 Filter, sort, and search data
- 📝 Edit records without writing SQL
- 👥 Great for non-technical team members

**Use Cases:**
- Quick data inspection during development
- Manual data entry and editing
- Testing relationships and constraints
- Debugging data issues
- Demo environments
- Content management

---

## Complete Workflows

### 1. Starting a New Project from Scratch

```bash
# Step 1: Create project and install dependencies
mkdir my-prisma-app
cd my-prisma-app
npm init -y
npm install prisma --save-dev
npm install @prisma/client

# Step 2: Initialize Prisma
npx prisma init --datasource-provider postgresql

# Step 3: Configure database URL in .env
# DATABASE_URL="postgresql://user:password@localhost:5432/mydb"

# Step 4: Define your schema in prisma/schema.prisma
# (Add your models here)

# Step 5: Create initial migration
npx prisma migrate dev --name init

# Step 6: (Optional) Create seed file
# Create prisma/seed.ts and configure in package.json

# Step 7: Seed the database
npx prisma db seed

# Step 8: Start using Prisma in your code
```

**Example schema.prisma for new project:**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]
  profile   Profile?
}

model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  userId Int     @unique
  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  USER
  ADMIN
  MODERATOR
}
```

---

### 2. Adding Prisma to Existing Database

```bash
# Step 1: Initialize Prisma
npx prisma init

# Step 2: Configure DATABASE_URL in .env
# Point to your existing database

# Step 3: Pull existing schema from database
npx prisma db pull

# Step 4: Review generated schema in prisma/schema.prisma
# Make any necessary adjustments (rename models, add relations, etc.)

# Step 5: Generate Prisma Client
npx prisma generate

# Step 6: Create baseline migration (optional but recommended)
npx prisma migrate dev --name baseline

# Step 7: Start using Prisma in your application
```

---

### 3. Daily Development Workflow

```bash
# Step 1: Update your schema
# Edit prisma/schema.prisma (add/modify models, fields, relations)

# Step 2: Format schema
npx prisma format

# Step 3: Validate schema
npx prisma validate

# Step 4: Create and apply migration
npx prisma migrate dev --name descriptive_migration_name

# Step 5: Prisma Client is auto-generated

# Step 6: Use new fields/models in your code

# Step 7: Test your changes

# Step 8: Commit schema and migration files
git add prisma/
git commit -m "Add new feature to database schema"
git push
```

---

### 4. Production Deployment Workflow

```bash
# Step 1: Pull latest code
git pull origin main

# Step 2: Install dependencies
npm ci

# Step 3: Generate Prisma Client
npx prisma generate

# Step 4: Check migration status
npx prisma migrate status

# Step 5: Apply pending migrations
npx prisma migrate deploy

# Step 6: Build application
npm run build

# Step 7: Start application
npm start
```

**Production Deployment Checklist:**
- ✅ Backup database before deployment
- ✅ Test migrations in staging first
- ✅ Use `migrate deploy`, never `migrate dev`
- ✅ Monitor application logs during deployment
- ✅ Have rollback plan ready
- ✅ Verify data integrity after migration

---

### 5. Team Collaboration Workflow

**Developer A (Creating migration):**

```bash
# 1. Pull latest changes
git pull origin main

# 2. Update schema
# Edit prisma/schema.prisma

# 3. Create migration
npx prisma migrate dev --name add_comment_system

# 4. Test changes locally

# 5. Commit and push
git add prisma/
git commit -m "Add comment system"
git push origin feature/comments
```

**Developer B (Applying teammate's migration):**

```bash
# 1. Pull changes
git pull origin main

# 2. Install dependencies (if package.json changed)
npm install

# 3. Apply new migrations
npx prisma migrate dev

# 4. Prisma Client is auto-generated

# 5. Continue development
```

---

## Quick Reference

### Command Cheat Sheet

| Command | Purpose | Environment | Safety |
|---------|---------|-------------|--------|
| `prisma init` | Initialize Prisma | Development | ✅ Safe |
| `prisma generate` | Generate Prisma Client | All | ✅ Safe |
| `prisma format` | Format schema | Development | ✅ Safe |
| `prisma validate` | Validate schema | All | ✅ Safe |
| `prisma db push` | Sync schema (no migrations) | Development only | ⚠️ Can lose data |
| `prisma db pull` | Pull schema from DB | Development | ✅ Safe |
| `prisma db seed` | Populate database | Development | ✅ Safe |
| `prisma db execute` | Run raw SQL | All | ⚠️ Use carefully |
| `prisma migrate dev` | Create + apply migration | Development only | ✅ Safe (dev) |
| `prisma migrate deploy` | Apply migrations | Production/Staging |