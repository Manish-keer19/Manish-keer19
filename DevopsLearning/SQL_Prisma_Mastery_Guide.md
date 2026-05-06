# 🧠 Mastering SQL & Prisma ORM
### A Complete Production Guide Using a Real-World Survey System

> **Goal:** After reading this guide, you will be able to write ANY SQL query and convert it into a Prisma ORM query confidently — including complex analytics, grouping, reporting, and data transformations.

---

## Table of Contents

1. [Core SQL Mental Model](#1-core-sql-mental-model)
2. [Survey System Schema](#survey-system-schema)
3. [Basic SQL Commands](#3-basic-sql-commands)
4. [Filtering & Conditions](#4-filtering--conditions)
5. [Joins](#5-joins)
6. [Aggregations](#6-aggregations)
7. [GROUP BY (Critical)](#7-group-by-critical)
8. [Advanced Queries](#8-advanced-queries)
9. [Real-World Analytics Scenarios](#9-real-world-analytics-scenarios)
10. [Data Transformation Layer](#10-data-transformation-layer)
11. [Performance & Indexing](#11-performance--indexing)
12. [Common Mistakes](#12-common-mistakes)
13. [Practice Section (10 Problems)](#13-practice-section-10-problems)

---

## 1. Core SQL Mental Model

### How Relational Databases Work

Think of a relational database as a collection of **Excel-like spreadsheets** (tables) that can **talk to each other** through shared identifiers (foreign keys).

```
DATABASE
├── surveys          ← Each row = one survey
├── survey_sessions  ← Each row = one user attempt
├── answers          ← Each row = one answer to one question
├── questions        ← Each row = one question in a survey
├── users            ← Each row = one registered user
├── locations        ← Each row = a geographic location
└── media            ← Each row = an uploaded file/image
```

### Tables, Rows, Columns

| Concept | Analogy | SQL Term |
|---|---|---|
| Spreadsheet | Table | `TABLE` |
| Row | One record | `ROW / TUPLE` |
| Column | Field/Attribute | `COLUMN / FIELD` |
| Unique ID | Serial number | `PRIMARY KEY` |
| Link to another table | Reference | `FOREIGN KEY` |

### Relationships

#### 1:1 (One-to-One)
One user has one profile. One survey has one creator.

```
User (1) ──── (1) Profile
```

#### 1:N (One-to-Many) — Most Common
One survey has many sessions. One question has many answers.

```
Survey (1) ──── (N) SurveySession
Question (1) ──── (N) Answer
```

#### N:N (Many-to-Many)
A survey can have many tags. A tag can belong to many surveys. Requires a **junction table**.

```
Survey (N) ──── SurveyTag (junction) ──── (N) Tag
```

### Foreign Keys

A foreign key is a column in Table B that points to the primary key of Table A. It enforces **referential integrity** — you can't have an answer without a valid question.

```sql
-- answers.question_id must match an existing questions.id
ALTER TABLE answers
ADD CONSTRAINT fk_question
FOREIGN KEY (question_id) REFERENCES questions(id);
```

### How to Think in SQL (The Retrieval Mindset)

Always answer these 5 questions before writing any query:

```
1. WHAT do I want?          → SELECT columns
2. FROM where?              → FROM table
3. HOW are tables connected? → JOIN conditions
4. WHICH rows do I want?    → WHERE filters
5. HOW should it be grouped/sorted? → GROUP BY / ORDER BY
```

---

## Survey System Schema

### Prisma Schema Definition

```prisma
// schema.prisma

model User {
  id            String          @id @default(cuid())
  name          String
  email         String          @unique
  role          Role            @default(RESPONDENT)
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
  lastActiveAt  DateTime?

  surveys        Survey[]
  sessions       SurveySession[]

  @@index([email])
  @@index([createdAt])
}

model Survey {
  id          String          @id @default(cuid())
  title       String
  description String?
  status      SurveyStatus    @default(DRAFT)
  createdById String
  locationId  String?
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  publishedAt DateTime?

  createdBy  User            @relation(fields: [createdById], references: [id])
  location   Location?       @relation(fields: [locationId], references: [id])
  questions  Question[]
  sessions   SurveySession[]

  @@index([createdById])
  @@index([status])
  @@index([createdAt])
}

model Question {
  id         String       @id @default(cuid())
  surveyId   String
  text       String
  type       QuestionType
  required   Boolean      @default(true)
  order      Int
  createdAt  DateTime     @default(now())

  survey   Survey   @relation(fields: [surveyId], references: [id])
  answers  Answer[]

  @@index([surveyId])
  @@index([surveyId, order])
}

model SurveySession {
  id          String        @id @default(cuid())
  surveyId    String
  userId      String?
  status      SessionStatus @default(IN_PROGRESS)
  startedAt   DateTime      @default(now())
  completedAt DateTime?
  durationSec Int?
  locationId  String?

  survey   Survey    @relation(fields: [surveyId], references: [id])
  user     User?     @relation(fields: [userId], references: [id])
  location Location? @relation(fields: [locationId], references: [id])
  answers  Answer[]

  @@index([surveyId])
  @@index([userId])
  @@index([status])
  @@index([startedAt])
}

model Answer {
  id         String   @id @default(cuid())
  sessionId  String
  questionId String
  value      String
  createdAt  DateTime @default(now())

  session  SurveySession @relation(fields: [sessionId], references: [id])
  question Question      @relation(fields: [questionId], references: [id])
  media    Media[]

  @@index([sessionId])
  @@index([questionId])
  @@unique([sessionId, questionId])
}

model Location {
  id        String  @id @default(cuid())
  name      String
  city      String
  state     String?
  country   String
  latitude  Float?
  longitude Float?

  surveys  Survey[]
  sessions SurveySession[]

  @@index([country])
  @@index([city])
}

model Media {
  id        String    @id @default(cuid())
  answerId  String
  url       String
  type      MediaType
  size      Int
  createdAt DateTime  @default(now())

  answer Answer @relation(fields: [answerId], references: [id])

  @@index([answerId])
}

enum Role {
  ADMIN
  CREATOR
  RESPONDENT
}

enum SurveyStatus {
  DRAFT
  PUBLISHED
  CLOSED
  ARCHIVED
}

enum SessionStatus {
  IN_PROGRESS
  COMPLETED
  ABANDONED
}

enum QuestionType {
  TEXT
  MULTIPLE_CHOICE
  RATING
  YES_NO
  DATE
}

enum MediaType {
  IMAGE
  VIDEO
  DOCUMENT
}
```

### Visual Schema Map

```
User ──────────────────────────────────────────┐
 │                                              │
 │ (creates)                                    │ (fills)
 ▼                                              ▼
Survey ──── Question ──── Answer ◄──── SurveySession
 │              │            │              │
 │              └────────────┘         (has media)
 │                                          ▼
 └──── Location ◄──────────────────────── Media
```

---

## 3. Basic SQL Commands

### SELECT — Retrieve Data

**Syntax:**
```sql
SELECT column1, column2, ...
FROM table_name;
```

**Example — Get all surveys:**
```sql
SELECT id, title, status, created_at
FROM surveys;
```

**Select all columns (avoid in production):**
```sql
SELECT * FROM surveys;  -- Never do this in production
```

**Prisma Equivalent:**
```typescript
// Get specific fields
const surveys = await prisma.survey.findMany({
  select: {
    id: true,
    title: true,
    status: true,
    createdAt: true,
  },
});

// Get all fields (equivalent to SELECT *)
const surveys = await prisma.survey.findMany();
```

---

### WHERE — Filter Rows

**Syntax:**
```sql
SELECT columns
FROM table
WHERE condition;
```

**Example — Get only PUBLISHED surveys:**
```sql
SELECT id, title, published_at
FROM surveys
WHERE status = 'PUBLISHED';
```

**Prisma Equivalent:**
```typescript
const surveys = await prisma.survey.findMany({
  where: {
    status: 'PUBLISHED',
  },
  select: {
    id: true,
    title: true,
    publishedAt: true,
  },
});
```

---

### ORDER BY — Sort Results

**Syntax:**
```sql
SELECT columns
FROM table
ORDER BY column [ASC | DESC];
```

**Example — Get surveys newest first:**
```sql
SELECT id, title, created_at
FROM surveys
ORDER BY created_at DESC;
```

**Multi-column sort:**
```sql
SELECT id, title, status, created_at
FROM surveys
ORDER BY status ASC, created_at DESC;
```

**Prisma Equivalent:**
```typescript
// Single sort
const surveys = await prisma.survey.findMany({
  orderBy: {
    createdAt: 'desc',
  },
});

// Multi-column sort
const surveys = await prisma.survey.findMany({
  orderBy: [
    { status: 'asc' },
    { createdAt: 'desc' },
  ],
});
```

---

### LIMIT & OFFSET — Pagination

**Syntax:**
```sql
SELECT columns
FROM table
LIMIT n OFFSET m;
```

**Example — Get page 2 of surveys (10 per page):**
```sql
SELECT id, title, status
FROM surveys
ORDER BY created_at DESC
LIMIT 10 OFFSET 10;  -- Skip first 10, take next 10
```

**Prisma Equivalent:**
```typescript
const page = 2;
const pageSize = 10;

const surveys = await prisma.survey.findMany({
  orderBy: { createdAt: 'desc' },
  take: pageSize,
  skip: (page - 1) * pageSize,  // skip = 10
});
```

---

### DISTINCT — Remove Duplicates

**Syntax:**
```sql
SELECT DISTINCT column
FROM table;
```

**Example — Get unique countries from locations:**
```sql
SELECT DISTINCT country
FROM locations
ORDER BY country ASC;
```

**Example — Get unique users who have submitted at least one session:**
```sql
SELECT DISTINCT user_id
FROM survey_sessions
WHERE status = 'COMPLETED';
```

**Prisma Equivalent:**
```typescript
// Prisma doesn't have a direct DISTINCT, but groupBy achieves this
const countries = await prisma.location.findMany({
  select: { country: true },
  distinct: ['country'],
  orderBy: { country: 'asc' },
});

// For unique users with completed sessions
const users = await prisma.surveySession.findMany({
  where: { status: 'COMPLETED' },
  select: { userId: true },
  distinct: ['userId'],
});
```

---

## 4. Filtering & Conditions

### AND / OR / NOT

```sql
-- AND: Both conditions must be true
SELECT id, title, status
FROM surveys
WHERE status = 'PUBLISHED'
  AND created_by_id = 'user_abc123';

-- OR: At least one condition must be true
SELECT id, title, status
FROM surveys
WHERE status = 'PUBLISHED'
   OR status = 'CLOSED';

-- NOT: Condition must be false
SELECT id, title
FROM surveys
WHERE NOT status = 'ARCHIVED';
-- Equivalent: WHERE status != 'ARCHIVED'
```

**Prisma Equivalent:**
```typescript
// AND (implicit when multiple keys in where)
const surveys = await prisma.survey.findMany({
  where: {
    status: 'PUBLISHED',
    createdById: 'user_abc123',
  },
});

// Explicit AND
const surveys = await prisma.survey.findMany({
  where: {
    AND: [
      { status: 'PUBLISHED' },
      { createdById: 'user_abc123' },
    ],
  },
});

// OR
const surveys = await prisma.survey.findMany({
  where: {
    OR: [
      { status: 'PUBLISHED' },
      { status: 'CLOSED' },
    ],
  },
});

// NOT
const surveys = await prisma.survey.findMany({
  where: {
    NOT: { status: 'ARCHIVED' },
  },
});
```

---

### IN — Match Multiple Values

```sql
-- Get surveys with specific statuses
SELECT id, title, status
FROM surveys
WHERE status IN ('PUBLISHED', 'CLOSED');

-- NOT IN
SELECT id, title, status
FROM surveys
WHERE status NOT IN ('DRAFT', 'ARCHIVED');
```

**Prisma Equivalent:**
```typescript
// IN
const surveys = await prisma.survey.findMany({
  where: {
    status: { in: ['PUBLISHED', 'CLOSED'] },
  },
});

// NOT IN
const surveys = await prisma.survey.findMany({
  where: {
    status: { notIn: ['DRAFT', 'ARCHIVED'] },
  },
});
```

---

### BETWEEN — Range Filtering

```sql
-- Sessions started in the last 7 days
SELECT id, survey_id, started_at
FROM survey_sessions
WHERE started_at BETWEEN '2024-01-01' AND '2024-01-31';

-- Sessions with duration between 30 and 300 seconds
SELECT id, duration_sec
FROM survey_sessions
WHERE duration_sec BETWEEN 30 AND 300;
```

**Prisma Equivalent:**
```typescript
// Date range
const sessions = await prisma.surveySession.findMany({
  where: {
    startedAt: {
      gte: new Date('2024-01-01'),
      lte: new Date('2024-01-31'),
    },
  },
});

// Number range
const sessions = await prisma.surveySession.findMany({
  where: {
    durationSec: {
      gte: 30,
      lte: 300,
    },
  },
});
```

---

### LIKE — Pattern Matching

```sql
-- Surveys whose title starts with "Customer"
SELECT id, title
FROM surveys
WHERE title LIKE 'Customer%';

-- Surveys containing "feedback" anywhere in title (case-insensitive)
SELECT id, title
FROM surveys
WHERE LOWER(title) LIKE '%feedback%';

-- Questions that start with "How" and end with "?"
SELECT id, text
FROM questions
WHERE text LIKE 'How%?';
```

**Prisma Equivalent:**
```typescript
// Starts with
const surveys = await prisma.survey.findMany({
  where: {
    title: { startsWith: 'Customer' },
  },
});

// Contains (case-insensitive in PostgreSQL with mode)
const surveys = await prisma.survey.findMany({
  where: {
    title: {
      contains: 'feedback',
      mode: 'insensitive',
    },
  },
});

// Ends with
const questions = await prisma.question.findMany({
  where: {
    text: { endsWith: '?' },
  },
});
```

---

### NULL Handling

```sql
-- Sessions that are NOT completed (completedAt is NULL)
SELECT id, survey_id, status
FROM survey_sessions
WHERE completed_at IS NULL;

-- Sessions that ARE completed
SELECT id, survey_id, completed_at
FROM survey_sessions
WHERE completed_at IS NOT NULL;

-- Anonymous sessions (no user)
SELECT id, survey_id
FROM survey_sessions
WHERE user_id IS NULL;

-- COALESCE: Use fallback when NULL
SELECT id, COALESCE(duration_sec, 0) AS duration
FROM survey_sessions;
```

**Prisma Equivalent:**
```typescript
// IS NULL
const incompleteSessions = await prisma.surveySession.findMany({
  where: {
    completedAt: null,
  },
});

// IS NOT NULL
const completedSessions = await prisma.surveySession.findMany({
  where: {
    completedAt: { not: null },
  },
});

// Anonymous sessions
const anonSessions = await prisma.surveySession.findMany({
  where: {
    userId: null,
  },
});
```

---

## 5. Joins

> Joins are the most powerful concept in SQL. They let you combine data from multiple tables into a single result.

### INNER JOIN — Only Matching Rows

Returns rows where the condition matches in **both** tables.

```
Table A    INNER JOIN    Table B
   ●●●          ●●●
     ↑ Only the overlap
```

**Example — Get all sessions with user info:**
```sql
SELECT
  ss.id          AS session_id,
  ss.status,
  ss.started_at,
  u.name         AS user_name,
  u.email        AS user_email
FROM survey_sessions ss
INNER JOIN users u ON ss.user_id = u.id;
-- NOTE: This excludes anonymous sessions (where user_id IS NULL)
```

**Example — Get answers with their question text:**
```sql
SELECT
  a.id           AS answer_id,
  a.value        AS answer_value,
  q.text         AS question_text,
  q.type         AS question_type
FROM answers a
INNER JOIN questions q ON a.question_id = q.id;
```

**Prisma Equivalent:**
```typescript
// Sessions with user (include)
const sessions = await prisma.surveySession.findMany({
  where: {
    userId: { not: null },  // Simulate INNER JOIN behavior
  },
  include: {
    user: true,
  },
});

// Answers with question text (select specific fields)
const answers = await prisma.answer.findMany({
  select: {
    id: true,
    value: true,
    question: {
      select: {
        text: true,
        type: true,
      },
    },
  },
});
```

---

### LEFT JOIN — All Left Rows, Matching Right Rows

Returns **all rows from the left table**, plus matching rows from right (NULL if no match).

```
Table A    LEFT JOIN    Table B
   ●●●●●●
     ↑ All of A, plus matching B (or NULL)
```

**Example — All sessions, including anonymous ones:**
```sql
SELECT
  ss.id          AS session_id,
  ss.status,
  ss.started_at,
  u.name         AS user_name,   -- NULL for anonymous sessions
  u.email        AS user_email   -- NULL for anonymous sessions
FROM survey_sessions ss
LEFT JOIN users u ON ss.user_id = u.id;
```

**Example — All surveys with their location (even if no location set):**
```sql
SELECT
  s.id           AS survey_id,
  s.title,
  l.name         AS location_name,  -- NULL if no location
  l.city,
  l.country
FROM surveys s
LEFT JOIN locations l ON s.location_id = l.id;
```

**Prisma Equivalent:**
```typescript
// Prisma's include/select always does LEFT JOIN behavior
const sessions = await prisma.surveySession.findMany({
  include: {
    user: true,  // null if userId is null
  },
});

// Surveys with optional location
const surveys = await prisma.survey.findMany({
  include: {
    location: true,  // null if locationId is null
  },
});
```

---

### RIGHT JOIN (Theory)

Returns **all rows from the right table**, plus matching rows from left. Rare in practice — just swap table order and use LEFT JOIN.

```sql
-- This RIGHT JOIN...
SELECT * FROM survey_sessions ss
RIGHT JOIN users u ON ss.user_id = u.id;

-- ...is equivalent to this LEFT JOIN:
SELECT * FROM users u
LEFT JOIN survey_sessions ss ON ss.user_id = u.id;
```

> 💡 **Tip:** Always prefer LEFT JOIN over RIGHT JOIN for readability. Reorder your tables instead.

---

### Multiple Joins

**Example — Full answer details with session, user, and question info:**
```sql
SELECT
  a.id              AS answer_id,
  a.value,
  q.text            AS question_text,
  q.type            AS question_type,
  ss.status         AS session_status,
  ss.started_at,
  u.name            AS user_name,
  u.email           AS user_email,
  s.title           AS survey_title
FROM answers a
INNER JOIN questions  q  ON a.question_id  = q.id
INNER JOIN survey_sessions ss ON a.session_id = ss.id
LEFT  JOIN users      u  ON ss.user_id     = u.id
INNER JOIN surveys    s  ON ss.survey_id   = s.id
WHERE ss.status = 'COMPLETED'
ORDER BY ss.started_at DESC;
```

**Prisma Equivalent:**
```typescript
const answers = await prisma.answer.findMany({
  where: {
    session: {
      status: 'COMPLETED',
    },
  },
  select: {
    id: true,
    value: true,
    question: {
      select: {
        text: true,
        type: true,
      },
    },
    session: {
      select: {
        status: true,
        startedAt: true,
        user: {
          select: {
            name: true,
            email: true,
          },
        },
        survey: {
          select: {
            title: true,
          },
        },
      },
    },
  },
  orderBy: {
    session: {
      startedAt: 'desc',
    },
  },
});
```

---

## 6. Aggregations

Aggregation functions collapse many rows into a single value.

### COUNT

```sql
-- Total number of surveys
SELECT COUNT(*) AS total_surveys
FROM surveys;

-- Total PUBLISHED surveys
SELECT COUNT(*) AS published_count
FROM surveys
WHERE status = 'PUBLISHED';

-- Count distinct users who submitted at least one session
SELECT COUNT(DISTINCT user_id) AS unique_respondents
FROM survey_sessions
WHERE status = 'COMPLETED';
```

**Prisma Equivalent:**
```typescript
// Total surveys
const total = await prisma.survey.count();

// Published surveys
const publishedCount = await prisma.survey.count({
  where: { status: 'PUBLISHED' },
});

// Unique respondents
const uniqueRespondents = await prisma.surveySession.findMany({
  where: { status: 'COMPLETED' },
  select: { userId: true },
  distinct: ['userId'],
});
const count = uniqueRespondents.length;

// Using _count in a relation context
const surveyWithCount = await prisma.survey.findUnique({
  where: { id: 'survey_id_here' },
  include: {
    _count: {
      select: { sessions: true, questions: true },
    },
  },
});
// surveyWithCount._count.sessions = 42
```

---

### SUM

```sql
-- Total duration of all completed sessions (in seconds)
SELECT SUM(duration_sec) AS total_duration_sec
FROM survey_sessions
WHERE status = 'COMPLETED';

-- Total media file size per answer
SELECT
  answer_id,
  SUM(size) AS total_size_bytes
FROM media
GROUP BY answer_id;
```

**Prisma Equivalent:**
```typescript
// Total duration
const result = await prisma.surveySession.aggregate({
  where: { status: 'COMPLETED' },
  _sum: {
    durationSec: true,
  },
});
console.log(result._sum.durationSec); // e.g., 84600

// Media size per answer (requires groupBy)
const mediaSizes = await prisma.media.groupBy({
  by: ['answerId'],
  _sum: {
    size: true,
  },
});
```

---

### AVG

```sql
-- Average session duration
SELECT AVG(duration_sec) AS avg_duration_sec
FROM survey_sessions
WHERE status = 'COMPLETED'
  AND duration_sec IS NOT NULL;

-- Average duration per survey
SELECT
  survey_id,
  AVG(duration_sec) AS avg_duration,
  COUNT(*) AS session_count
FROM survey_sessions
WHERE status = 'COMPLETED'
GROUP BY survey_id;
```

**Prisma Equivalent:**
```typescript
// Global average
const result = await prisma.surveySession.aggregate({
  where: {
    status: 'COMPLETED',
    durationSec: { not: null },
  },
  _avg: {
    durationSec: true,
  },
});
console.log(result._avg.durationSec); // e.g., 187.5

// Average per survey (groupBy)
const avgPerSurvey = await prisma.surveySession.groupBy({
  by: ['surveyId'],
  where: { status: 'COMPLETED' },
  _avg: {
    durationSec: true,
  },
  _count: {
    id: true,
  },
});
```

---

### MIN / MAX

```sql
-- Fastest and slowest completed sessions
SELECT
  MIN(duration_sec) AS fastest_sec,
  MAX(duration_sec) AS slowest_sec,
  AVG(duration_sec) AS avg_sec
FROM survey_sessions
WHERE status = 'COMPLETED';

-- Earliest and latest session per survey
SELECT
  survey_id,
  MIN(started_at) AS first_response,
  MAX(started_at) AS latest_response
FROM survey_sessions
GROUP BY survey_id;
```

**Prisma Equivalent:**
```typescript
// Global min/max
const stats = await prisma.surveySession.aggregate({
  where: { status: 'COMPLETED' },
  _min: { durationSec: true },
  _max: { durationSec: true },
  _avg: { durationSec: true },
});

// Per survey
const perSurvey = await prisma.surveySession.groupBy({
  by: ['surveyId'],
  _min: { startedAt: true },
  _max: { startedAt: true },
});
```

---

## 7. GROUP BY (Critical)

### The Core Concept

`GROUP BY` collapses multiple rows that share the same value(s) into a single row, allowing aggregate functions to operate on each group.

**Rule:** Every column in SELECT must either be in GROUP BY **or** be inside an aggregate function.

```sql
-- WRONG ❌ — 'title' is not in GROUP BY and not aggregated
SELECT survey_id, title, COUNT(*)
FROM survey_sessions
GROUP BY survey_id;

-- CORRECT ✅
SELECT survey_id, COUNT(*) AS session_count
FROM survey_sessions
GROUP BY survey_id;

-- ALSO CORRECT ✅ — join to get title
SELECT
  ss.survey_id,
  s.title,
  COUNT(*) AS session_count
FROM survey_sessions ss
JOIN surveys s ON ss.survey_id = s.id
GROUP BY ss.survey_id, s.title;
```

---

### HAVING vs WHERE

| | WHERE | HAVING |
|---|---|---|
| Filters | Individual rows | Groups (after GROUP BY) |
| Runs | Before aggregation | After aggregation |
| Can use aggregates? | ❌ No | ✅ Yes |

```sql
-- WHERE filters rows BEFORE grouping
SELECT survey_id, COUNT(*) AS count
FROM survey_sessions
WHERE status = 'COMPLETED'        -- ← Runs first, on rows
GROUP BY survey_id
HAVING COUNT(*) > 10;             -- ← Runs last, on groups

-- Real example: Surveys with more than 10 completed responses
SELECT
  ss.survey_id,
  s.title,
  COUNT(*) AS completed_count
FROM survey_sessions ss
JOIN surveys s ON ss.survey_id = s.id
WHERE ss.status = 'COMPLETED'
GROUP BY ss.survey_id, s.title
HAVING COUNT(*) > 10
ORDER BY completed_count DESC;
```

---

### Responses per User

```sql
SELECT
  u.id,
  u.name,
  u.email,
  COUNT(ss.id) AS total_sessions,
  SUM(CASE WHEN ss.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed,
  SUM(CASE WHEN ss.status = 'ABANDONED' THEN 1 ELSE 0 END) AS abandoned
FROM users u
LEFT JOIN survey_sessions ss ON ss.user_id = u.id
GROUP BY u.id, u.name, u.email
ORDER BY completed DESC;
```

**Prisma Equivalent:**
```typescript
const userStats = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
    _count: {
      select: { sessions: true },
    },
  },
  orderBy: {
    sessions: { _count: 'desc' },
  },
});

// For completed/abandoned breakdown, use groupBy
const sessionBreakdown = await prisma.surveySession.groupBy({
  by: ['userId', 'status'],
  _count: { id: true },
  where: { userId: { not: null } },
});
```

---

### Answers per Question

```sql
SELECT
  q.id,
  q.text,
  q.type,
  COUNT(a.id) AS answer_count
FROM questions q
LEFT JOIN answers a ON a.question_id = q.id
WHERE q.survey_id = 'survey_abc123'
GROUP BY q.id, q.text, q.type
ORDER BY q.order ASC;
```

**Prisma Equivalent:**
```typescript
const questionsWithCounts = await prisma.question.findMany({
  where: { surveyId: 'survey_abc123' },
  select: {
    id: true,
    text: true,
    type: true,
    order: true,
    _count: {
      select: { answers: true },
    },
  },
  orderBy: { order: 'asc' },
});
```

---

### Sessions per Day (Time-based Grouping)

```sql
-- PostgreSQL: Use DATE() to truncate to day
SELECT
  DATE(started_at)   AS submission_date,
  COUNT(*)           AS session_count,
  COUNT(DISTINCT user_id) AS unique_users
FROM survey_sessions
WHERE started_at >= NOW() - INTERVAL '30 days'
  AND status = 'COMPLETED'
GROUP BY DATE(started_at)
ORDER BY submission_date ASC;
```

**Prisma Equivalent (requires $queryRaw for date truncation):**
```typescript
// Option 1: Raw query for date grouping
const dailyStats = await prisma.$queryRaw<
  Array<{ submission_date: Date; session_count: bigint; unique_users: bigint }>
>`
  SELECT
    DATE(started_at)        AS submission_date,
    COUNT(*)                AS session_count,
    COUNT(DISTINCT user_id) AS unique_users
  FROM survey_sessions
  WHERE started_at >= NOW() - INTERVAL '30 days'
    AND status = 'COMPLETED'
  GROUP BY DATE(started_at)
  ORDER BY submission_date ASC
`;

// Option 2: Fetch sessions and group in application code
const sessions = await prisma.surveySession.findMany({
  where: {
    startedAt: { gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
    status: 'COMPLETED',
  },
  select: { startedAt: true, userId: true },
});

// Group in JS
const grouped = sessions.reduce((acc, s) => {
  const date = s.startedAt.toISOString().split('T')[0];
  if (!acc[date]) acc[date] = { count: 0, users: new Set() };
  acc[date].count++;
  if (s.userId) acc[date].users.add(s.userId);
  return acc;
}, {} as Record<string, { count: number; users: Set<string> }>);
```

---

## 8. Advanced Queries

### Subqueries

A subquery is a query nested inside another query.

```sql
-- Surveys that have at least one completed session (using subquery)
SELECT id, title
FROM surveys
WHERE id IN (
  SELECT DISTINCT survey_id
  FROM survey_sessions
  WHERE status = 'COMPLETED'
);

-- Users who have NEVER submitted a session
SELECT id, name, email
FROM users
WHERE id NOT IN (
  SELECT DISTINCT user_id
  FROM survey_sessions
  WHERE user_id IS NOT NULL
);

-- Sessions that took longer than the average duration
SELECT id, survey_id, duration_sec
FROM survey_sessions
WHERE duration_sec > (
  SELECT AVG(duration_sec)
  FROM survey_sessions
  WHERE status = 'COMPLETED'
)
AND status = 'COMPLETED'
ORDER BY duration_sec DESC;
```

**Prisma Equivalent:**
```typescript
// Surveys with completed sessions
const surveysWithResponses = await prisma.survey.findMany({
  where: {
    sessions: {
      some: {
        status: 'COMPLETED',
      },
    },
  },
});

// Users who have NEVER submitted
const inactiveUsers = await prisma.user.findMany({
  where: {
    sessions: {
      none: {},
    },
  },
});

// Sessions longer than average (requires $queryRaw)
const aboveAvgSessions = await prisma.$queryRaw`
  SELECT id, survey_id, duration_sec
  FROM survey_sessions
  WHERE duration_sec > (
    SELECT AVG(duration_sec) FROM survey_sessions WHERE status = 'COMPLETED'
  )
  AND status = 'COMPLETED'
  ORDER BY duration_sec DESC
`;
```

---

### CASE Statements (Conditional Logic)

```sql
-- Classify session duration
SELECT
  id,
  duration_sec,
  CASE
    WHEN duration_sec < 60  THEN 'Quick (< 1 min)'
    WHEN duration_sec < 300 THEN 'Normal (1-5 min)'
    WHEN duration_sec < 600 THEN 'Long (5-10 min)'
    ELSE 'Very Long (10+ min)'
  END AS duration_label
FROM survey_sessions
WHERE status = 'COMPLETED';

-- Count sessions by engagement tier
SELECT
  survey_id,
  COUNT(*) AS total,
  SUM(CASE WHEN duration_sec < 60  THEN 1 ELSE 0 END) AS quick,
  SUM(CASE WHEN duration_sec BETWEEN 60 AND 299 THEN 1 ELSE 0 END) AS normal,
  SUM(CASE WHEN duration_sec >= 300 THEN 1 ELSE 0 END) AS deep
FROM survey_sessions
WHERE status = 'COMPLETED'
GROUP BY survey_id;
```

**Prisma Equivalent (post-processing in JS):**
```typescript
const sessions = await prisma.surveySession.findMany({
  where: { status: 'COMPLETED' },
  select: { id: true, durationSec: true, surveyId: true },
});

// Add label in application layer
const labeled = sessions.map(s => ({
  ...s,
  durationLabel:
    (s.durationSec ?? 0) < 60  ? 'Quick (< 1 min)' :
    (s.durationSec ?? 0) < 300 ? 'Normal (1-5 min)' :
    (s.durationSec ?? 0) < 600 ? 'Long (5-10 min)' :
    'Very Long (10+ min)',
}));
```

---

### EXISTS

```sql
-- Surveys that have at least one question
SELECT id, title
FROM surveys s
WHERE EXISTS (
  SELECT 1
  FROM questions q
  WHERE q.survey_id = s.id
);

-- Users who have answered at least one rating question
SELECT DISTINCT u.id, u.name
FROM users u
WHERE EXISTS (
  SELECT 1
  FROM survey_sessions ss
  JOIN answers a ON a.session_id = ss.id
  JOIN questions q ON a.question_id = q.id
  WHERE ss.user_id = u.id
    AND q.type = 'RATING'
);
```

**Prisma Equivalent:**
```typescript
// Surveys with at least one question
const surveysWithQuestions = await prisma.survey.findMany({
  where: {
    questions: { some: {} },
  },
});

// Users who answered a RATING question
const usersWithRatingAnswers = await prisma.user.findMany({
  where: {
    sessions: {
      some: {
        answers: {
          some: {
            question: { type: 'RATING' },
          },
        },
      },
    },
  },
});
```

---

### Prisma Limitations & When to Use $queryRaw

| Feature | Prisma Support | Use $queryRaw? |
|---|---|---|
| DATE/time truncation for GROUP BY | ❌ | ✅ |
| CASE WHEN in SELECT | ❌ | ✅ |
| Window functions (ROW_NUMBER, RANK) | ❌ | ✅ |
| Complex subqueries | Limited | Sometimes |
| UNION / UNION ALL | ❌ | ✅ |
| CTEs (WITH clause) | ❌ | ✅ |
| Full-text search (native) | Partial | Sometimes |

```typescript
// $queryRaw — always returns typed results
const results = await prisma.$queryRaw<
  Array<{ survey_id: string; count: bigint }>
>`
  SELECT survey_id, COUNT(*) as count
  FROM survey_sessions
  WHERE status = 'COMPLETED'
  GROUP BY survey_id
`;

// IMPORTANT: BigInt conversion for JSON serialization
const serializable = results.map(r => ({
  surveyId: r.survey_id,
  count: Number(r.count),  // Convert BigInt → number
}));
```

---

## 9. Real-World Analytics Scenarios

### Scenario 1: Total Responses per Survey

```sql
SELECT
  s.id            AS survey_id,
  s.title,
  s.status,
  COUNT(ss.id)    AS total_sessions,
  SUM(CASE WHEN ss.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed,
  SUM(CASE WHEN ss.status = 'ABANDONED' THEN 1 ELSE 0 END) AS abandoned,
  ROUND(
    100.0 * SUM(CASE WHEN ss.status = 'COMPLETED' THEN 1 ELSE 0 END)
    / NULLIF(COUNT(ss.id), 0), 2
  ) AS completion_rate_pct
FROM surveys s
LEFT JOIN survey_sessions ss ON ss.survey_id = s.id
GROUP BY s.id, s.title, s.status
ORDER BY completed DESC;
```

**Prisma Equivalent:**
```typescript
const surveys = await prisma.survey.findMany({
  select: {
    id: true,
    title: true,
    status: true,
    _count: {
      select: { sessions: true },
    },
  },
});

// Get breakdown per status
const sessionBreakdown = await prisma.surveySession.groupBy({
  by: ['surveyId', 'status'],
  _count: { id: true },
});

// Merge in application layer
const result = surveys.map(survey => {
  const breakdown = sessionBreakdown.filter(b => b.surveyId === survey.id);
  const completed = breakdown.find(b => b.status === 'COMPLETED')?._count.id ?? 0;
  const abandoned = breakdown.find(b => b.status === 'ABANDONED')?._count.id ?? 0;
  const total = survey._count.sessions;

  return {
    ...survey,
    completed,
    abandoned,
    completionRate: total > 0 ? ((completed / total) * 100).toFixed(2) + '%' : '0%',
  };
});
```

---

### Scenario 2: Top Users by Submissions

```sql
SELECT
  u.id,
  u.name,
  u.email,
  COUNT(ss.id)                       AS total_submissions,
  COUNT(DISTINCT ss.survey_id)       AS unique_surveys,
  MAX(ss.started_at)                 AS last_submission,
  ROUND(AVG(ss.duration_sec), 0)     AS avg_duration_sec
FROM users u
INNER JOIN survey_sessions ss ON ss.user_id = u.id
WHERE ss.status = 'COMPLETED'
GROUP BY u.id, u.name, u.email
ORDER BY total_submissions DESC
LIMIT 10;
```

**Prisma Equivalent:**
```typescript
const topUsers = await prisma.user.findMany({
  where: {
    sessions: {
      some: { status: 'COMPLETED' },
    },
  },
  select: {
    id: true,
    name: true,
    email: true,
    sessions: {
      where: { status: 'COMPLETED' },
      select: {
        surveyId: true,
        startedAt: true,
        durationSec: true,
      },
    },
  },
});

// Transform in application layer
const ranked = topUsers
  .map(user => ({
    id: user.id,
    name: user.name,
    email: user.email,
    totalSubmissions: user.sessions.length,
    uniqueSurveys: new Set(user.sessions.map(s => s.surveyId)).size,
    lastSubmission: user.sessions.reduce(
      (max, s) => s.startedAt > max ? s.startedAt : max,
      new Date(0)
    ),
    avgDurationSec: Math.round(
      user.sessions.reduce((sum, s) => sum + (s.durationSec ?? 0), 0)
      / user.sessions.length
    ),
  }))
  .sort((a, b) => b.totalSubmissions - a.totalSubmissions)
  .slice(0, 10);
```

---

### Scenario 3: Question-wise Answer Count

```sql
SELECT
  q.id,
  q.text,
  q.type,
  q.order,
  COUNT(a.id)          AS answer_count,
  COUNT(DISTINCT a.session_id) AS unique_sessions
FROM questions q
LEFT JOIN answers a ON a.question_id = q.id
LEFT JOIN survey_sessions ss ON a.session_id = ss.id
  AND ss.status = 'COMPLETED'
WHERE q.survey_id = 'survey_abc123'
GROUP BY q.id, q.text, q.type, q.order
ORDER BY q.order ASC;
```

**Prisma Equivalent:**
```typescript
const questions = await prisma.question.findMany({
  where: { surveyId: 'survey_abc123' },
  orderBy: { order: 'asc' },
  select: {
    id: true,
    text: true,
    type: true,
    order: true,
    answers: {
      where: {
        session: { status: 'COMPLETED' },
      },
      select: { id: true, sessionId: true },
    },
  },
});

const result = questions.map(q => ({
  id: q.id,
  text: q.text,
  type: q.type,
  order: q.order,
  answerCount: q.answers.length,
  uniqueSessions: new Set(q.answers.map(a => a.sessionId)).size,
}));
```

---

### Scenario 4: Daily Submission Trends

```sql
WITH daily AS (
  SELECT
    DATE(started_at)                AS day,
    COUNT(*)                        AS submissions,
    COUNT(DISTINCT user_id)         AS unique_users,
    SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed,
    ROUND(AVG(duration_sec), 0)     AS avg_duration
  FROM survey_sessions
  WHERE started_at >= NOW() - INTERVAL '30 days'
  GROUP BY DATE(started_at)
)
SELECT
  day,
  submissions,
  unique_users,
  completed,
  avg_duration,
  SUM(submissions) OVER (ORDER BY day) AS running_total
FROM daily
ORDER BY day ASC;
```

**Prisma Equivalent ($queryRaw for CTE + window function):**
```typescript
interface DailyTrend {
  day: Date;
  submissions: bigint;
  unique_users: bigint;
  completed: bigint;
  avg_duration: number;
  running_total: bigint;
}

const trends = await prisma.$queryRaw<DailyTrend[]>`
  WITH daily AS (
    SELECT
      DATE(started_at)                AS day,
      COUNT(*)                        AS submissions,
      COUNT(DISTINCT user_id)         AS unique_users,
      SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed,
      ROUND(AVG(duration_sec), 0)     AS avg_duration
    FROM survey_sessions
    WHERE started_at >= NOW() - INTERVAL '30 days'
    GROUP BY DATE(started_at)
  )
  SELECT
    day,
    submissions,
    unique_users,
    completed,
    avg_duration,
    SUM(submissions) OVER (ORDER BY day) AS running_total
  FROM daily
  ORDER BY day ASC
`;

// Convert BigInts to numbers for JSON
const serialized = trends.map(t => ({
  day: t.day.toISOString().split('T')[0],
  submissions: Number(t.submissions),
  uniqueUsers: Number(t.unique_users),
  completed: Number(t.completed),
  avgDuration: t.avg_duration,
  runningTotal: Number(t.running_total),
}));
```

---

### Scenario 5: Location-based Responses

```sql
SELECT
  l.id,
  l.name            AS location_name,
  l.city,
  l.country,
  COUNT(DISTINCT ss.id)         AS total_sessions,
  COUNT(DISTINCT ss.user_id)    AS unique_respondents,
  COUNT(DISTINCT ss.survey_id)  AS surveys_taken,
  ROUND(AVG(ss.duration_sec), 0) AS avg_duration
FROM locations l
LEFT JOIN survey_sessions ss ON ss.location_id = l.id
  AND ss.status = 'COMPLETED'
GROUP BY l.id, l.name, l.city, l.country
ORDER BY total_sessions DESC;
```

**Prisma Equivalent:**
```typescript
const locationStats = await prisma.location.findMany({
  select: {
    id: true,
    name: true,
    city: true,
    country: true,
    sessions: {
      where: { status: 'COMPLETED' },
      select: {
        id: true,
        userId: true,
        surveyId: true,
        durationSec: true,
      },
    },
  },
});

const result = locationStats
  .map(loc => ({
    id: loc.id,
    locationName: loc.name,
    city: loc.city,
    country: loc.country,
    totalSessions: loc.sessions.length,
    uniqueRespondents: new Set(loc.sessions.map(s => s.userId).filter(Boolean)).size,
    surveysTaken: new Set(loc.sessions.map(s => s.surveyId)).size,
    avgDuration: loc.sessions.length > 0
      ? Math.round(
          loc.sessions.reduce((sum, s) => sum + (s.durationSec ?? 0), 0)
          / loc.sessions.length
        )
      : 0,
  }))
  .sort((a, b) => b.totalSessions - a.totalSessions);
```

---

## 10. Data Transformation Layer

### Converting DB Results → API Response

```typescript
// Raw Prisma result (nested, verbose)
const rawSession = await prisma.surveySession.findUnique({
  where: { id: sessionId },
  include: {
    survey: { select: { id: true, title: true } },
    user: { select: { id: true, name: true, email: true } },
    answers: {
      include: {
        question: { select: { text: true, type: true } },
        media: { select: { url: true, type: true } },
      },
    },
  },
});

// Transform → Clean API Response
function transformSession(raw: typeof rawSession) {
  if (!raw) return null;
  return {
    id: raw.id,
    status: raw.status,
    startedAt: raw.startedAt.toISOString(),
    completedAt: raw.completedAt?.toISOString() ?? null,
    durationSeconds: raw.durationSec,
    survey: {
      id: raw.survey.id,
      title: raw.survey.title,
    },
    respondent: raw.user
      ? { id: raw.user.id, name: raw.user.name, email: raw.user.email }
      : { id: null, name: 'Anonymous', email: null },
    answers: raw.answers.map(a => ({
      question: a.question.text,
      questionType: a.question.type,
      value: a.value,
      attachments: a.media.map(m => ({ url: m.url, type: m.type })),
    })),
  };
}
```

---

### Prepare Data for Excel Export

```typescript
import ExcelJS from 'exceljs';

async function exportSurveyResults(surveyId: string) {
  // 1. Fetch data
  const sessions = await prisma.surveySession.findMany({
    where: { surveyId, status: 'COMPLETED' },
    include: {
      user: { select: { name: true, email: true } },
      answers: {
        include: { question: { select: { text: true, order: true } } },
        orderBy: { question: { order: 'asc' } },
      },
    },
    orderBy: { completedAt: 'desc' },
  });

  // 2. Get question headers (from first session or query directly)
  const questions = await prisma.question.findMany({
    where: { surveyId },
    orderBy: { order: 'asc' },
    select: { id: true, text: true },
  });

  // 3. Build flat rows for Excel
  const rows = sessions.map(session => {
    const row: Record<string, string | number | Date | null> = {
      'Session ID':    session.id,
      'User Name':     session.user?.name ?? 'Anonymous',
      'User Email':    session.user?.email ?? '',
      'Started At':    session.startedAt,
      'Completed At':  session.completedAt ?? '',
      'Duration (sec)': session.durationSec ?? 0,
    };

    // Map each question answer into its own column
    questions.forEach(q => {
      const answer = session.answers.find(a => a.question.text === q.text);
      row[q.text] = answer?.value ?? '';
    });

    return row;
  });

  // 4. Write to Excel
  const workbook = new ExcelJS.Workbook();
  const sheet = workbook.addWorksheet('Responses');

  if (rows.length > 0) {
    sheet.columns = Object.keys(rows[0]).map(key => ({
      header: key,
      key,
      width: Math.max(key.length + 4, 15),
    }));
    sheet.addRows(rows);

    // Style header row
    sheet.getRow(1).font = { bold: true };
    sheet.getRow(1).fill = {
      type: 'pattern', pattern: 'solid',
      fgColor: { argb: 'FF4F81BD' },
    };
  }

  return workbook.xlsx.writeBuffer();
}
```

---

### Prepare Chart Data

```typescript
// Prepare data for a bar chart: responses per survey (last 30 days)
async function getChartData() {
  const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);

  const grouped = await prisma.surveySession.groupBy({
    by: ['surveyId'],
    where: {
      status: 'COMPLETED',
      startedAt: { gte: thirtyDaysAgo },
    },
    _count: { id: true },
    orderBy: { _count: { id: 'desc' } },
    take: 10,
  });

  // Get survey titles
  const surveyIds = grouped.map(g => g.surveyId);
  const surveys = await prisma.survey.findMany({
    where: { id: { in: surveyIds } },
    select: { id: true, title: true },
  });

  const titleMap = Object.fromEntries(surveys.map(s => [s.id, s.title]));

  // Format for Chart.js / Recharts
  return {
    labels: grouped.map(g => titleMap[g.surveyId] ?? g.surveyId),
    datasets: [{
      label: 'Completed Responses',
      data: grouped.map(g => g._count.id),
      backgroundColor: '#4F81BD',
    }],
  };
}
```

---

## 11. Performance & Indexing

### What Indexes Do

An index is like a **book's table of contents** — without it, the database reads every row (full table scan). With it, it jumps directly to the matching rows.

```
WITHOUT INDEX on survey_sessions.status:
  → DB reads ALL 1,000,000 rows to find 'COMPLETED' ones
  → O(n) time

WITH INDEX on survey_sessions.status:
  → DB jumps directly to 'COMPLETED' rows
  → O(log n) time
```

### When Queries Become Slow

1. **No index on WHERE/JOIN columns** — full table scans
2. **SELECT \*** — fetches unnecessary data over the network
3. **N+1 problem** — 1 query to get 100 records, then 100 more queries for each related record
4. **Missing compound index** — index on `(a)` doesn't help `WHERE a = ? AND b = ?`
5. **Wildcard LIKE at start** — `LIKE '%feedback'` can't use an index

### How the Survey Schema Indexes Help

```prisma
// From our schema — these are CRITICAL:

model SurveySession {
  @@index([surveyId])    // Fast: find all sessions for a survey
  @@index([userId])      // Fast: find all sessions by a user
  @@index([status])      // Fast: filter by COMPLETED/ABANDONED
  @@index([startedAt])   // Fast: date range queries & sorting
}

model Answer {
  @@index([sessionId])    // Fast: get all answers in a session
  @@index([questionId])   // Fast: get all answers to a question
  @@unique([sessionId, questionId])  // Fast: prevent duplicates + lookup
}
```

### Adding Compound Indexes for Common Query Patterns

```prisma
// If you often query: WHERE survey_id = ? AND status = ?
model SurveySession {
  @@index([surveyId, status])     // Compound index for this pattern
  @@index([surveyId, startedAt])  // For date-range per survey
}

// If you often query answers for a completed session:
model Answer {
  @@index([sessionId, questionId])  // Already unique — covers this
}
```

### EXPLAIN ANALYZE (PostgreSQL)

```sql
-- See how PostgreSQL executes your query
EXPLAIN ANALYZE
SELECT ss.id, ss.status, u.name
FROM survey_sessions ss
JOIN users u ON ss.user_id = u.id
WHERE ss.survey_id = 'survey_abc123'
  AND ss.status = 'COMPLETED';

-- Look for: "Seq Scan" (bad) vs "Index Scan" (good)
-- Look for: actual time and row estimates
```

---

## 12. Common Mistakes

### 1. Wrong Joins (Missing Rows)

```sql
-- ❌ WRONG: Using INNER JOIN when you want all surveys
SELECT s.title, COUNT(ss.id) as responses
FROM surveys s
INNER JOIN survey_sessions ss ON ss.survey_id = s.id
GROUP BY s.title;
-- Problem: Surveys with ZERO responses are excluded!

-- ✅ CORRECT: Use LEFT JOIN
SELECT s.title, COUNT(ss.id) as responses
FROM surveys s
LEFT JOIN survey_sessions ss ON ss.survey_id = s.id
GROUP BY s.title;
-- Now surveys with 0 responses show count = 0
```

### 2. The N+1 Problem

```typescript
// ❌ N+1: 1 query for surveys + N queries for sessions
const surveys = await prisma.survey.findMany();
for (const survey of surveys) {
  // This runs 1 query PER survey = N+1 total queries!
  const sessions = await prisma.surveySession.findMany({
    where: { surveyId: survey.id },
  });
}

// ✅ CORRECT: Single query with include
const surveys = await prisma.survey.findMany({
  include: {
    sessions: true,  // Fetched in 1 query (or 2 at most)
  },
});
```

### 3. Over-fetching in Prisma

```typescript
// ❌ Over-fetch: Getting entire user object when you need only name
const sessions = await prisma.surveySession.findMany({
  include: {
    user: true,           // Gets ALL user fields including sensitive ones
    survey: true,         // Gets ALL survey fields
    answers: {
      include: {
        question: true,   // Gets ALL question fields
        media: true,      // Gets ALL media fields
      },
    },
  },
});

// ✅ CORRECT: Select only what you need
const sessions = await prisma.surveySession.findMany({
  select: {
    id: true,
    status: true,
    startedAt: true,
    user: { select: { name: true, email: true } },
    survey: { select: { title: true } },
    answers: {
      select: {
        value: true,
        question: { select: { text: true } },
      },
    },
  },
});
```

### 4. Misusing groupBy

```typescript
// ❌ WRONG: groupBy without aggregate — just use findMany + distinct
const result = await prisma.surveySession.groupBy({
  by: ['surveyId'],
  // No _count, _sum, etc. — you just wanted unique survey IDs
});

// ✅ CORRECT for unique IDs:
const uniqueSurveys = await prisma.surveySession.findMany({
  select: { surveyId: true },
  distinct: ['surveyId'],
});

// ✅ CORRECT for actual groupBy with aggregates:
const stats = await prisma.surveySession.groupBy({
  by: ['surveyId', 'status'],
  _count: { id: true },
  _avg: { durationSec: true },
});
```

### 5. Forgetting NULL Safety

```typescript
// ❌ WRONG: Can crash if completedAt is null
const duration = session.completedAt.getTime() - session.startedAt.getTime();

// ✅ CORRECT: Always guard nullables
const duration = session.completedAt
  ? session.completedAt.getTime() - session.startedAt.getTime()
  : null;
```

### 6. Not Converting BigInt from $queryRaw

```typescript
// ❌ WRONG: JSON.stringify will throw on BigInt
const result = await prisma.$queryRaw<{ count: bigint }[]>`
  SELECT COUNT(*) as count FROM survey_sessions
`;
return res.json(result); // TypeError: Do not know how to serialize a BigInt

// ✅ CORRECT: Convert to Number
return res.json(result.map(r => ({ count: Number(r.count) })));
```

---

## 13. Practice Section (10 Problems)

> For each problem: try writing the query yourself first, then check the solution.

---

### Problem 1: Get All Completed Sessions of a Survey

**Goal:** Return all completed sessions for survey ID `'survey_abc123'`, including the user's name and email, sorted by completion date.

```sql
-- SQL Solution
SELECT
  ss.id,
  ss.started_at,
  ss.completed_at,
  ss.duration_sec,
  u.name   AS user_name,
  u.email  AS user_email
FROM survey_sessions ss
LEFT JOIN users u ON ss.user_id = u.id
WHERE ss.survey_id = 'survey_abc123'
  AND ss.status = 'COMPLETED'
ORDER BY ss.completed_at DESC;
```

```typescript
// Prisma Solution
const completedSessions = await prisma.surveySession.findMany({
  where: {
    surveyId: 'survey_abc123',
    status: 'COMPLETED',
  },
  select: {
    id: true,
    startedAt: true,
    completedAt: true,
    durationSec: true,
    user: {
      select: { name: true, email: true },
    },
  },
  orderBy: { completedAt: 'desc' },
});
```

---

### Problem 2: Count Answers per Question for a Survey

**Goal:** For each question in survey `'survey_abc123'`, count how many answers it received (from completed sessions only).

```sql
-- SQL Solution
SELECT
  q.id,
  q.text,
  q.type,
  COUNT(a.id) AS answer_count
FROM questions q
LEFT JOIN answers a ON a.question_id = q.id
LEFT JOIN survey_sessions ss ON a.session_id = ss.id
  AND ss.status = 'COMPLETED'
WHERE q.survey_id = 'survey_abc123'
GROUP BY q.id, q.text, q.type, q.order
ORDER BY q.order ASC;
```

```typescript
// Prisma Solution
const questions = await prisma.question.findMany({
  where: { surveyId: 'survey_abc123' },
  orderBy: { order: 'asc' },
  select: {
    id: true,
    text: true,
    type: true,
    _count: {
      select: {
        answers: {
          where: {
            session: { status: 'COMPLETED' },
          },
        },
      },
    },
  },
});
```

---

### Problem 3: Find Inactive Users (No Sessions in 30 Days)

**Goal:** Get users who haven't submitted any session in the last 30 days.

```sql
-- SQL Solution
SELECT
  u.id,
  u.name,
  u.email,
  MAX(ss.started_at) AS last_session_date
FROM users u
LEFT JOIN survey_sessions ss ON ss.user_id = u.id
GROUP BY u.id, u.name, u.email
HAVING MAX(ss.started_at) < NOW() - INTERVAL '30 days'
    OR MAX(ss.started_at) IS NULL
ORDER BY last_session_date ASC NULLS FIRST;
```

```typescript
// Prisma Solution
const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);

const inactiveUsers = await prisma.user.findMany({
  where: {
    OR: [
      // No sessions at all
      { sessions: { none: {} } },
      // Last session older than 30 days
      {
        sessions: {
          every: {
            startedAt: { lt: thirtyDaysAgo },
          },
        },
      },
    ],
  },
  select: {
    id: true,
    name: true,
    email: true,
    sessions: {
      select: { startedAt: true },
      orderBy: { startedAt: 'desc' },
      take: 1,
    },
  },
});
```

---

### Problem 4: Get Top 5 Surveys by Completed Responses

**Goal:** Return the 5 surveys with the most completed sessions.

```sql
-- SQL Solution
SELECT
  s.id,
  s.title,
  s.status,
  COUNT(ss.id) AS completed_count
FROM surveys s
INNER JOIN survey_sessions ss ON ss.survey_id = s.id
WHERE ss.status = 'COMPLETED'
GROUP BY s.id, s.title, s.status
ORDER BY completed_count DESC
LIMIT 5;
```

```typescript
// Prisma Solution
const topSurveys = await prisma.survey.findMany({
  where: {
    sessions: {
      some: { status: 'COMPLETED' },
    },
  },
  select: {
    id: true,
    title: true,
    status: true,
    _count: {
      select: {
        sessions: {
          where: { status: 'COMPLETED' },
        },
      },
    },
  },
  orderBy: {
    sessions: { _count: 'desc' },
  },
  take: 5,
});
```

---

### Problem 5: Average Completion Time per Survey

**Goal:** For each survey, compute the average session duration (in seconds) for completed sessions.

```sql
-- SQL Solution
SELECT
  s.id,
  s.title,
  ROUND(AVG(ss.duration_sec), 2)  AS avg_duration_sec,
  MIN(ss.duration_sec)            AS min_duration_sec,
  MAX(ss.duration_sec)            AS max_duration_sec,
  COUNT(ss.id)                    AS sample_size
FROM surveys s
INNER JOIN survey_sessions ss ON ss.survey_id = s.id
WHERE ss.status = 'COMPLETED'
  AND ss.duration_sec IS NOT NULL
GROUP BY s.id, s.title
ORDER BY avg_duration_sec ASC;
```

```typescript
// Prisma Solution
const durationStats = await prisma.surveySession.groupBy({
  by: ['surveyId'],
  where: {
    status: 'COMPLETED',
    durationSec: { not: null },
  },
  _avg: { durationSec: true },
  _min: { durationSec: true },
  _max: { durationSec: true },
  _count: { id: true },
  orderBy: { _avg: { durationSec: 'asc' } },
});

// Enrich with survey titles
const surveyIds = durationStats.map(d => d.surveyId);
const surveys = await prisma.survey.findMany({
  where: { id: { in: surveyIds } },
  select: { id: true, title: true },
});
const titleMap = Object.fromEntries(surveys.map(s => [s.id, s.title]));

const result = durationStats.map(d => ({
  surveyId: d.surveyId,
  title: titleMap[d.surveyId],
  avgDurationSec: d._avg.durationSec,
  minDurationSec: d._min.durationSec,
  maxDurationSec: d._max.durationSec,
  sampleSize: d._count.id,
}));
```

---

### Problem 6: Find All Surveys a User Has Participated In

**Goal:** Get all distinct surveys a specific user has submitted a session for.

```sql
-- SQL Solution
SELECT DISTINCT
  s.id,
  s.title,
  s.status,
  MIN(ss.started_at)  AS first_attempt,
  MAX(ss.started_at)  AS latest_attempt,
  COUNT(ss.id)        AS attempt_count
FROM surveys s
INNER JOIN survey_sessions ss ON ss.survey_id = s.id
WHERE ss.user_id = 'user_xyz789'
GROUP BY s.id, s.title, s.status
ORDER BY latest_attempt DESC;
```

```typescript
// Prisma Solution
const userSurveys = await prisma.survey.findMany({
  where: {
    sessions: {
      some: { userId: 'user_xyz789' },
    },
  },
  select: {
    id: true,
    title: true,
    status: true,
    sessions: {
      where: { userId: 'user_xyz789' },
      select: { startedAt: true },
      orderBy: { startedAt: 'desc' },
    },
  },
});

const result = userSurveys.map(s => ({
  id: s.id,
  title: s.title,
  status: s.status,
  firstAttempt: s.sessions[s.sessions.length - 1]?.startedAt,
  latestAttempt: s.sessions[0]?.startedAt,
  attemptCount: s.sessions.length,
}));
```

---

### Problem 7: Abandoned Session Rate by Survey

**Goal:** For each survey, calculate the percentage of sessions that were abandoned.

```sql
-- SQL Solution
SELECT
  s.id,
  s.title,
  COUNT(ss.id)                                               AS total,
  SUM(CASE WHEN ss.status = 'ABANDONED' THEN 1 ELSE 0 END)  AS abandoned,
  ROUND(
    100.0 * SUM(CASE WHEN ss.status = 'ABANDONED' THEN 1 ELSE 0 END)
    / NULLIF(COUNT(ss.id), 0), 2
  ) AS abandonment_rate_pct
FROM surveys s
LEFT JOIN survey_sessions ss ON ss.survey_id = s.id
GROUP BY s.id, s.title
ORDER BY abandonment_rate_pct DESC NULLS LAST;
```

```typescript
// Prisma Solution
const surveys = await prisma.survey.findMany({
  select: {
    id: true,
    title: true,
  },
});

const breakdown = await prisma.surveySession.groupBy({
  by: ['surveyId', 'status'],
  _count: { id: true },
});

const result = surveys.map(survey => {
  const stats = breakdown.filter(b => b.surveyId === survey.id);
  const total = stats.reduce((sum, s) => sum + s._count.id, 0);
  const abandoned = stats.find(s => s.status === 'ABANDONED')?._count.id ?? 0;

  return {
    id: survey.id,
    title: survey.title,
    total,
    abandoned,
    abandonmentRate: total > 0
      ? `${((abandoned / total) * 100).toFixed(2)}%`
      : 'N/A',
  };
});
```

---

### Problem 8: Get All Media Files for a Survey's Completed Sessions

**Goal:** List all media files uploaded as part of a specific survey's completed sessions.

```sql
-- SQL Solution
SELECT
  m.id          AS media_id,
  m.url,
  m.type        AS media_type,
  m.size,
  a.value       AS answer_value,
  q.text        AS question_text,
  u.name        AS respondent_name,
  ss.completed_at
FROM media m
INNER JOIN answers a     ON m.answer_id      = a.id
INNER JOIN questions q   ON a.question_id    = q.id
INNER JOIN survey_sessions ss ON a.session_id = ss.id
LEFT  JOIN users u       ON ss.user_id       = u.id
WHERE ss.survey_id = 'survey_abc123'
  AND ss.status = 'COMPLETED'
ORDER BY ss.completed_at DESC, q.order ASC;
```

```typescript
// Prisma Solution
const mediaFiles = await prisma.media.findMany({
  where: {
    answer: {
      session: {
        surveyId: 'survey_abc123',
        status: 'COMPLETED',
      },
    },
  },
  select: {
    id: true,
    url: true,
    type: true,
    size: true,
    answer: {
      select: {
        value: true,
        question: { select: { text: true, order: true } },
        session: {
          select: {
            completedAt: true,
            user: { select: { name: true } },
          },
        },
      },
    },
  },
  orderBy: {
    answer: { session: { completedAt: 'desc' } },
  },
});
```

---

### Problem 9: Questions with Zero Responses

**Goal:** Find questions in a survey that no one has answered yet.

```sql
-- SQL Solution
SELECT
  q.id,
  q.text,
  q.type,
  q.required
FROM questions q
WHERE q.survey_id = 'survey_abc123'
  AND NOT EXISTS (
    SELECT 1
    FROM answers a
    WHERE a.question_id = q.id
  )
ORDER BY q.order ASC;
```

```typescript
// Prisma Solution
const unansweredQuestions = await prisma.question.findMany({
  where: {
    surveyId: 'survey_abc123',
    answers: { none: {} },
  },
  select: {
    id: true,
    text: true,
    type: true,
    required: true,
    order: true,
  },
  orderBy: { order: 'asc' },
});
```

---

### Problem 10: Surveys Created in the Last 7 Days with Their Stats

**Goal:** Get surveys created in the last 7 days with session counts broken down by status.

```sql
-- SQL Solution
SELECT
  s.id,
  s.title,
  s.status          AS survey_status,
  s.created_at,
  COUNT(ss.id)      AS total_sessions,
  SUM(CASE WHEN ss.status = 'COMPLETED'   THEN 1 ELSE 0 END) AS completed,
  SUM(CASE WHEN ss.status = 'IN_PROGRESS' THEN 1 ELSE 0 END) AS in_progress,
  SUM(CASE WHEN ss.status = 'ABANDONED'   THEN 1 ELSE 0 END) AS abandoned
FROM surveys s
LEFT JOIN survey_sessions ss ON ss.survey_id = s.id
WHERE s.created_at >= NOW() - INTERVAL '7 days'
GROUP BY s.id, s.title, s.status, s.created_at
ORDER BY s.created_at DESC;
```

```typescript
// Prisma Solution
const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);

const recentSurveys = await prisma.survey.findMany({
  where: {
    createdAt: { gte: sevenDaysAgo },
  },
  select: {
    id: true,
    title: true,
    status: true,
    createdAt: true,
    _count: {
      select: { sessions: true },
    },
  },
  orderBy: { createdAt: 'desc' },
});

// Get breakdown
const sessionBreakdown = await prisma.surveySession.groupBy({
  by: ['surveyId', 'status'],
  where: {
    survey: { createdAt: { gte: sevenDaysAgo } },
  },
  _count: { id: true },
});

const result = recentSurveys.map(survey => {
  const stats = sessionBreakdown.filter(b => b.surveyId === survey.id);
  const getCount = (status: string) =>
    stats.find(s => s.status === status)?._count.id ?? 0;

  return {
    id: survey.id,
    title: survey.title,
    surveyStatus: survey.status,
    createdAt: survey.createdAt,
    totalSessions: survey._count.sessions,
    completed: getCount('COMPLETED'),
    inProgress: getCount('IN_PROGRESS'),
    abandoned: getCount('ABANDONED'),
  };
});
```

---

## Quick Reference Cheat Sheet

### SQL → Prisma Mapping

| SQL | Prisma |
|---|---|
| `SELECT col1, col2` | `select: { col1: true, col2: true }` |
| `WHERE col = val` | `where: { col: val }` |
| `WHERE col IN (a, b)` | `where: { col: { in: [a, b] } }` |
| `WHERE col IS NULL` | `where: { col: null }` |
| `WHERE col IS NOT NULL` | `where: { col: { not: null } }` |
| `WHERE col LIKE '%x%'` | `where: { col: { contains: 'x' } }` |
| `ORDER BY col DESC` | `orderBy: { col: 'desc' }` |
| `LIMIT n OFFSET m` | `take: n, skip: m` |
| `INNER JOIN` | `include` / `select` (with non-null filter) |
| `LEFT JOIN` | `include` / `select` (default behavior) |
| `COUNT(*)` | `_count: { select: { field: true } }` |
| `AVG(col)` | `_avg: { col: true }` |
| `SUM(col)` | `_sum: { col: true }` |
| `MIN(col)` | `_min: { col: true }` |
| `MAX(col)` | `_max: { col: true }` |
| `GROUP BY col` | `groupBy: { by: ['col'] }` |
| `HAVING COUNT(*) > n` | `having: { field: { _count: { gt: n } } }` |
| `DISTINCT col` | `distinct: ['col']` |
| Complex SQL | `$queryRaw` |

### Prisma Relation Filters

```typescript
// At least one matching related record
where: { sessions: { some: { status: 'COMPLETED' } } }

// All related records match
where: { sessions: { every: { status: 'COMPLETED' } } }

// No related records match
where: { sessions: { none: { status: 'ABANDONED' } } }

// Filter on related record existence
where: { sessions: { some: {} } }        // has any sessions
where: { sessions: { none: {} } }        // has no sessions
```

---

*This guide was designed to take you from zero to confidently writing production-grade SQL and Prisma queries. The survey system schema used throughout is representative of real enterprise applications. Keep this as a reference — the more you use it, the more intuitive these patterns become.*
