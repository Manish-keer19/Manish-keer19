# 🏗️ Enterprise Survey Engine Architecture
### Production-Grade Design Reference for Scalable Survey Systems

> **Reference Guide for:** Google Forms · SurveyMonkey · Typeform · Qualtrics · Microsoft Forms

---

## Table of Contents

1. [Overview](#1-overview)
2. [Core Design Principles](#2-core-design-principles)
3. [High-Level Architecture Flow](#3-high-level-architecture-flow)
4. [Supported Question Types](#4-supported-question-types)
5. [Database Architecture](#5-database-architecture)
6. [Full Prisma Schema](#6-full-prisma-schema)
7. [How Each Question Type Stores Data](#7-how-each-question-type-stores-data)
8. [Sample Data Tables](#8-sample-data-tables)
9. [Analytics Design & Queries](#9-analytics-design--queries)
10. [NPS Calculation](#10-nps-calculation)
11. [Index Strategy](#11-index-strategy)
12. [Scalability Considerations](#12-scalability-considerations)
13. [Future Extensions](#13-future-extensions)
14. [Production Best Practices](#14-production-best-practices)
15. [Final Architecture Diagram](#15-final-architecture-diagram)

---

## 1. Overview

### What is Survey Engine Architecture?

A **Survey Engine** is the backend data system that powers dynamic surveys. It must:

- Support many question types (radio, checkbox, grids, NPS, etc.)
- Store millions of responses efficiently
- Enable real-time analytics without redesigning tables
- Be extensible without schema migrations for new question types

### Why Scalable Design Matters

| Problem | Naive Approach | Scalable Approach |
|---|---|---|
| New question type | Add new table | Use `config` JSON |
| Analytics query | Complex JOINs | Single `answers` table |
| Grid questions | Separate grid table | Reuse `options` + `gridRows` |
| 10M+ responses | Slow queries | Partition + index strategy |

### Analytics-First Architecture

The schema is designed so that **every analytics query** follows the same pattern:

```sql
SELECT optionId, COUNT(*) FROM answers
WHERE questionId = ?
GROUP BY optionId;
```

No matter the question type — radio, checkbox, dropdown — the query is identical.

---

## 2. Core Design Principles

### Principle 1: Flexible Question Types via Config JSON

Instead of creating a separate table for every question type, a `config` JSON field on the `Question` table stores type-specific settings.

```json
// Rating question config
{ "min": 1, "max": 5, "labels": { "1": "Poor", "5": "Excellent" } }

// NPS question config
{ "min": 0, "max": 10 }

// Dropdown question config
{ "searchable": true, "placeholder": "Select one..." }
```

### Principle 2: Universal Answer Table

**One `answers` table stores everything.** Each row uses only the relevant columns:

| Question Type | Column Used |
|---|---|
| Radio / Checkbox / Dropdown | `optionId` |
| Rating / NPS | `numberValue` |
| Short / Long Text | `textValue` |
| Grid Questions | `gridRowId` + `optionId` |
| Location / Media | `jsonValue` |

### Principle 3: Minimal Tables, Maximum Flexibility

Only **4 core domain tables** needed:

```
Question → QuestionOption → QuestionGridRow → Answer
```

### Principle 4: Grid-Based System

Grid questions (Radio Grid, Checkbox Grid, Dropdown Grid, NPS Grid) are not special — they reuse the same `options` table for **columns** and `gridRows` table for **rows**.

---

## 3. High-Level Architecture Flow

```
┌─────────────────────────────────────────────────────┐
│                      SURVEY                         │
│  id · title · description · status · settings       │
└──────────────────────┬──────────────────────────────┘
                       │ 1:N
                       ▼
┌─────────────────────────────────────────────────────┐
│                    QUESTION                         │
│  id · text · type · config (JSON) · orderIndex      │
└───────┬───────────────────────┬─────────────────────┘
        │ 1:N                   │ 1:N
        ▼                       ▼
┌───────────────┐     ┌──────────────────────┐
│ QUESTION      │     │  QUESTION GRID ROW   │
│ OPTION        │     │                      │
│               │     │  id · text           │
│ id · text     │     │  orderIndex          │
│ orderIndex    │     │  (rows in grid)      │
│ isOtherOption │     └──────────────────────┘
└───────────────┘               │
        │                       │
        └──────────┬────────────┘
                   │ N:N via Answer
                   ▼
┌─────────────────────────────────────────────────────┐
│                     ANSWER                          │
│  sessionId · questionId · optionId · gridRowId      │
│  textValue · numberValue · jsonValue                │
└──────────────────────┬──────────────────────────────┘
                       │ N:1
                       ▼
┌─────────────────────────────────────────────────────┐
│                  SURVEY SESSION                     │
│  id · surveyId · userId · status · completedAt      │
└─────────────────────────────────────────────────────┘
```

---

## 4. Supported Question Types

### Simple (Single Value) Questions

| Type | Storage Column | Description |
|---|---|---|
| `SHORT_TEXT` | `textValue` | One-line text input |
| `LONG_TEXT` | `textValue` | Multi-line textarea |
| `NUMBER` | `numberValue` | Numeric input |
| `RATING` | `numberValue` | Star/slider rating (e.g., 1–5) |
| `NPS` | `numberValue` | Net Promoter Score (0–10) |
| `DATE` | `textValue` | Date picker |
| `EMAIL` | `textValue` | Email input with validation |
| `PHONE` | `textValue` | Phone number |

### Choice Questions (Use Options Table)

| Type | Storage Column | Multi-Select? |
|---|---|---|
| `RADIO` | `optionId` | ❌ Single |
| `RADIO_WITH_OTHER` | `optionId` or `textValue` | ❌ Single |
| `CHECKBOX` | `optionId` (multiple rows) | ✅ Multiple rows |
| `CHECKBOX_WITH_OTHER` | `optionId` or `textValue` | ✅ Multiple rows |
| `DROPDOWN` | `optionId` | ❌ Single |
| `DROPDOWN_WITH_OTHER` | `optionId` or `textValue` | ❌ Single |

### Grid Questions (Use Options + GridRows)

| Type | Columns From | Rows From | Multi-Select Per Row? |
|---|---|---|---|
| `RADIO_GRID` | `QuestionOption` | `QuestionGridRow` | ❌ |
| `CHECKBOX_GRID` | `QuestionOption` | `QuestionGridRow` | ✅ |
| `DROPDOWN_GRID` | `QuestionOption` | `QuestionGridRow` | ❌ |
| `RATING_GRID` | `QuestionOption` (labels) | `QuestionGridRow` | ❌ |
| `NPS_GRID` | N/A (0–10 scale) | `QuestionGridRow` | ❌ |

### Rich Input Questions

| Type | Storage Column | Config Keys |
|---|---|---|
| `MEDIA_UPLOAD` | `jsonValue` | `{ allowedTypes, maxSizeMB }` |
| `LOCATION` | `jsonValue` | `{ lat, lng, address }` |
| `SIGNATURE` | `jsonValue` | `{ format: "base64" }` |
| `FILE_UPLOAD` | `jsonValue` | `{ allowedExtensions }` |

---

## 5. Database Architecture

### Entity Relationship Overview

```
Survey (1) ──────── (N) Question
Question (1) ─────── (N) QuestionOption
Question (1) ─────── (N) QuestionGridRow
Question (1) ─────── (N) Answer
QuestionOption (1) ── (N) Answer
QuestionGridRow (1) ─ (N) Answer
SurveySession (1) ─── (N) Answer
```

### Why This Structure Works

**For a Radio question:**
- `Question` stores the question text + type = `RADIO`
- `QuestionOption` stores each choice (BJP, Congress, AAP)
- `Answer` stores `optionId` of the selected choice

**For a Radio Grid question:**
- `Question` stores the question text + type = `RADIO_GRID`
- `QuestionOption` stores **column headers** (Poor, Good, Excellent)
- `QuestionGridRow` stores **row labels** (Service, Price, Support)
- `Answer` stores `gridRowId` + `optionId` for each row's selection

**The beauty:** Analytics queries are identical regardless of type.

---

## 6. Full Prisma Schema

```prisma
// ============================================================
// ENTERPRISE SURVEY ENGINE - PRISMA SCHEMA
// Production-ready · Analytics-first · Extensible
// ============================================================

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─────────────────────────────────────────────
// SURVEY
// ─────────────────────────────────────────────
model Survey {
  id          String   @id @default(cuid())
  title       String
  description String?
  status      String   @default("DRAFT") // DRAFT | ACTIVE | CLOSED | ARCHIVED
  settings    Json?    // { allowAnonymous, maxResponses, redirectUrl, theme }
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  questions Question[]
  sessions  SurveySession[]

  @@map("surveys")
}

// ─────────────────────────────────────────────
// QUESTION
// ─────────────────────────────────────────────
model Question {
  id          String   @id @default(cuid())
  surveyId    String
  text        String
  description String?
  type        String
  // Examples of config by type:
  // RATING:   { "min": 1, "max": 5, "shape": "star" }
  // NPS:      { "min": 0, "max": 10, "lowLabel": "Never", "highLabel": "Definitely" }
  // TEXT:     { "maxLength": 500, "placeholder": "Your answer..." }
  // DROPDOWN: { "searchable": true, "placeholder": "Select..." }
  config      Json?
  // Examples of validation:
  // { "required": true, "minLength": 10, "maxLength": 500 }
  // { "required": true, "min": 1, "max": 10 }
  validation  Json?
  orderIndex  Int
  isRequired  Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  survey   Survey           @relation(fields: [surveyId], references: [id], onDelete: Cascade)
  options  QuestionOption[]
  gridRows QuestionGridRow[]
  answers  Answer[]

  @@index([surveyId])
  @@map("questions")
}

// ─────────────────────────────────────────────
// QUESTION OPTION
// Used by: Radio, Checkbox, Dropdown, and Grid columns
// ─────────────────────────────────────────────
model QuestionOption {
  id            String  @id @default(cuid())
  questionId    String
  text          String
  orderIndex    Int
  isOtherOption Boolean @default(false) // For "Radio with Other" etc.

  question Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  answers  Answer[]

  @@index([questionId])
  @@map("question_options")
}

// ─────────────────────────────────────────────
// QUESTION GRID ROW
// Used by: Radio Grid, Checkbox Grid, Dropdown Grid, NPS Grid
// ─────────────────────────────────────────────
model QuestionGridRow {
  id         String @id @default(cuid())
  questionId String
  text       String
  orderIndex Int

  question Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  answers  Answer[]

  @@index([questionId])
  @@map("question_grid_rows")
}

// ─────────────────────────────────────────────
// ANSWER
// Universal answer store — one table for ALL question types
// ─────────────────────────────────────────────
model Answer {
  id          String   @id @default(cuid())
  sessionId   String
  questionId  String
  optionId    String?  // Set for Radio, Checkbox, Dropdown, Grid (column)
  gridRowId   String?  // Set for Grid questions (row)
  textValue   String?  // Set for Text, Email, Phone, Date, "Other" text
  numberValue Float?   // Set for Rating, NPS, Number
  jsonValue   Json?    // Set for Location, Media, File, Signature
  createdAt   DateTime @default(now())

  session   SurveySession   @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  question  Question        @relation(fields: [questionId], references: [id], onDelete: Cascade)
  option    QuestionOption? @relation(fields: [optionId], references: [id])
  gridRow   QuestionGridRow? @relation(fields: [gridRowId], references: [id])

  @@index([questionId])
  @@index([sessionId])
  @@index([optionId])
  @@index([gridRowId])
  @@index([createdAt])
  @@index([questionId, optionId])
  @@index([questionId, gridRowId, optionId])
  @@map("answers")
}

// ─────────────────────────────────────────────
// SURVEY SESSION
// One session = one respondent's attempt at a survey
// ─────────────────────────────────────────────
model SurveySession {
  id          String    @id @default(cuid())
  surveyId    String
  userId      String?   // Null for anonymous responses
  startedAt   DateTime  @default(now())
  completedAt DateTime?
  status      String    @default("IN_PROGRESS") // IN_PROGRESS | COMPLETED | ABANDONED

  survey  Survey   @relation(fields: [surveyId], references: [id], onDelete: Cascade)
  answers Answer[]

  @@index([surveyId])
  @@index([userId])
  @@index([status])
  @@map("survey_sessions")
}
```

---

## 7. How Each Question Type Stores Data

### 7.1 Radio Question

**Question:** "Which party do you support?"

**Setup:**
```
Question Table:
id=Q1 | text="Which party do you support?" | type=RADIO

QuestionOption Table:
id=O1 | questionId=Q1 | text="BJP"      | orderIndex=1
id=O2 | questionId=Q1 | text="Congress" | orderIndex=2
id=O3 | questionId=Q1 | text="AAP"      | orderIndex=3
```

**User selects: Congress**
```
Answer Table:
sessionId=S1 | questionId=Q1 | optionId=O2
```

**Analytics:**
```sql
SELECT o.text, COUNT(*) as votes
FROM answers a
JOIN question_options o ON a.optionId = o.id
WHERE a.questionId = 'Q1'
GROUP BY o.id, o.text
ORDER BY votes DESC;
```

---

### 7.2 Checkbox Question (Multi-Select)

**Question:** "Which problems exist in your area?"

**Setup:**
```
Options: O1=Water | O2=Road | O3=Electricity
```

**User selects: Water + Electricity → 2 rows inserted**

```
Answer Table:
sessionId=S1 | questionId=Q2 | optionId=O1  (Water)
sessionId=S1 | questionId=Q2 | optionId=O3  (Electricity)
```

> ⚠️ Key insight: One session produces **multiple answer rows** for checkbox. This is intentional and correct.

---

### 7.3 Radio With Other

**User selects "Other" and types custom text:**

```
Answer Table:
sessionId=S1 | questionId=Q3 | optionId=O_OTHER | textValue="Green Party"
```

The `isOtherOption=true` flag on the option tells the UI to render a text input.

---

### 7.4 Rating Question

**Question:** "Rate our service quality (1–5)"

**Config:** `{ "min": 1, "max": 5, "shape": "star" }`

```
Answer Table:
sessionId=S1 | questionId=Q4 | numberValue=4
```

**Analytics:**
```sql
SELECT AVG(numberValue) as avg_rating,
       COUNT(*) as total_responses
FROM answers
WHERE questionId = 'Q4';
```

---

### 7.5 NPS Question

**Question:** "How likely are you to recommend us? (0–10)"

```
Answer Table:
sessionId=S1 | questionId=Q5 | numberValue=9
sessionId=S2 | questionId=Q5 | numberValue=3
sessionId=S3 | questionId=Q5 | numberValue=7
```

NPS logic is applied at **query time**, not storage time (see Section 10).

---

### 7.6 Radio Grid Question ⭐ (Most Complex)

**Question:** "Please rate each aspect of our service"

**Grid Structure:**

|           | Poor | Good | Excellent |
|-----------|------|------|-----------|
| Service   | ○    | ○    | ○         |
| Price     | ○    | ○    | ○         |
| Support   | ○    | ○    | ○         |

**Setup:**
```
Question: id=Q6 | type=RADIO_GRID

QuestionOption (COLUMNS):
id=C1 | questionId=Q6 | text="Poor"      | orderIndex=1
id=C2 | questionId=Q6 | text="Good"      | orderIndex=2
id=C3 | questionId=Q6 | text="Excellent" | orderIndex=3

QuestionGridRow (ROWS):
id=R1 | questionId=Q6 | text="Service" | orderIndex=1
id=R2 | questionId=Q6 | text="Price"   | orderIndex=2
id=R3 | questionId=Q6 | text="Support" | orderIndex=3
```

**User selects: Service→Good, Price→Excellent, Support→Poor**

```
Answer Table (3 rows, one per grid row):
sessionId=S1 | questionId=Q6 | gridRowId=R1 | optionId=C2  (Service → Good)
sessionId=S1 | questionId=Q6 | gridRowId=R2 | optionId=C3  (Price → Excellent)
sessionId=S1 | questionId=Q6 | gridRowId=R3 | optionId=C1  (Support → Poor)
```

**Analytics:**
```sql
SELECT
  gr.text as row_label,
  o.text  as column_label,
  COUNT(*) as count
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
JOIN question_options   o  ON a.optionId  = o.id
WHERE a.questionId = 'Q6'
GROUP BY gr.text, o.text
ORDER BY gr.orderIndex, o.orderIndex;
```

---

### 7.7 Checkbox Grid Question

Same structure as Radio Grid, but users can select **multiple columns per row**.

```
Answer Table (multiple rows per gridRowId allowed):
sessionId=S1 | questionId=Q7 | gridRowId=R1 | optionId=C1
sessionId=S1 | questionId=Q7 | gridRowId=R1 | optionId=C3  ← same row, different column
sessionId=S1 | questionId=Q7 | gridRowId=R2 | optionId=C2
```

---

### 7.8 NPS Grid Question

**Question:** "Rate your likelihood to recommend each product (0–10)"

```
QuestionGridRow: iPhone, MacBook, iPad

Answer Table:
sessionId=S1 | questionId=Q8 | gridRowId=R1 (iPhone)  | numberValue=9
sessionId=S1 | questionId=Q8 | gridRowId=R2 (MacBook) | numberValue=7
sessionId=S1 | questionId=Q8 | gridRowId=R3 (iPad)    | numberValue=4
```

No options needed — NPS range (0–10) comes from config.

---

### 7.9 Location Question

```
Answer Table:
sessionId=S1 | questionId=Q9 | jsonValue={"lat": 22.7196, "lng": 75.8577, "address": "Indore, MP"}
```

---

## 8. Sample Data Tables

### surveys

| id | title | status |
|---|---|---|
| SV1 | Customer Satisfaction Q4 | ACTIVE |
| SV2 | Employee Feedback 2024 | ACTIVE |

### questions

| id | surveyId | text | type | orderIndex | isRequired |
|---|---|---|---|---|---|
| Q1 | SV1 | Which party do you support? | RADIO | 1 | true |
| Q2 | SV1 | Which problems exist in your area? | CHECKBOX | 2 | false |
| Q3 | SV1 | Rate our service quality | RATING | 3 | true |
| Q4 | SV1 | How likely to recommend? | NPS | 4 | true |
| Q5 | SV1 | Rate each aspect of our service | RADIO_GRID | 5 | false |

### question_options

| id | questionId | text | orderIndex | isOtherOption |
|---|---|---|---|---|
| O1 | Q1 | BJP | 1 | false |
| O2 | Q1 | Congress | 2 | false |
| O3 | Q1 | AAP | 3 | false |
| O4 | Q2 | Water | 1 | false |
| O5 | Q2 | Road | 2 | false |
| O6 | Q2 | Electricity | 3 | false |
| O7 | Q5 | Poor | 1 | false |
| O8 | Q5 | Good | 2 | false |
| O9 | Q5 | Excellent | 3 | false |

### question_grid_rows

| id | questionId | text | orderIndex |
|---|---|---|---|
| R1 | Q5 | Service | 1 |
| R2 | Q5 | Price | 2 |
| R3 | Q5 | Support | 3 |

### answers

| id | sessionId | questionId | optionId | gridRowId | numberValue | textValue |
|---|---|---|---|---|---|---|
| A1 | S1 | Q1 | O2 | — | — | — |
| A2 | S1 | Q2 | O4 | — | — | — |
| A3 | S1 | Q2 | O6 | — | — | — |
| A4 | S1 | Q3 | — | — | 4 | — |
| A5 | S1 | Q4 | — | — | 9 | — |
| A6 | S1 | Q5 | O8 | R1 | — | — |
| A7 | S1 | Q5 | O9 | R2 | — | — |
| A8 | S1 | Q5 | O7 | R3 | — | — |

### survey_sessions

| id | surveyId | userId | status | completedAt |
|---|---|---|---|---|
| S1 | SV1 | U1 | COMPLETED | 2024-12-01 |
| S2 | SV1 | U2 | COMPLETED | 2024-12-02 |
| S3 | SV1 | null | COMPLETED | 2024-12-02 |

---

## 9. Analytics Design & Queries

### 9.1 Radio / Dropdown Analytics

```sql
-- Option distribution for a radio/dropdown question
SELECT
  o.text        AS option,
  COUNT(a.id)   AS responses,
  ROUND(COUNT(a.id) * 100.0 / SUM(COUNT(a.id)) OVER (), 2) AS percentage
FROM answers a
JOIN question_options o ON a.optionId = o.id
WHERE a.questionId = 'Q1'
GROUP BY o.id, o.text
ORDER BY responses DESC;
```

**Result:**
| option | responses | percentage |
|---|---|---|
| BJP | 450 | 45.00 |
| Congress | 320 | 32.00 |
| AAP | 230 | 23.00 |

---

### 9.2 Checkbox Analytics

```sql
-- Checkbox: Count each option selected (one respondent = multiple rows)
SELECT
  o.text       AS option,
  COUNT(a.id)  AS times_selected
FROM answers a
JOIN question_options o ON a.optionId = o.id
WHERE a.questionId = 'Q2'
GROUP BY o.id, o.text
ORDER BY times_selected DESC;
```

---

### 9.3 Rating Analytics

```sql
-- Rating distribution + average
SELECT
  numberValue AS rating,
  COUNT(*)    AS count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct
FROM answers
WHERE questionId = 'Q3'
GROUP BY numberValue
ORDER BY numberValue;

-- Average
SELECT AVG(numberValue) AS avg_rating FROM answers WHERE questionId = 'Q3';
```

---

### 9.4 Grid Analytics (Radio Grid / Checkbox Grid)

```sql
-- Full grid heatmap
SELECT
  gr.text   AS row_label,
  o.text    AS col_label,
  COUNT(*)  AS count
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
JOIN question_options   o  ON a.optionId  = o.id
WHERE a.questionId = 'Q5'
GROUP BY gr.id, gr.text, gr.orderIndex, o.id, o.text, o.orderIndex
ORDER BY gr.orderIndex, o.orderIndex;
```

**Result:**
| row_label | col_label | count |
|---|---|---|
| Service | Poor | 45 |
| Service | Good | 312 |
| Service | Excellent | 643 |
| Price | Poor | 120 |
| Price | Good | 450 |
| Price | Excellent | 430 |

---

### 9.5 Trend Query (Responses Over Time)

```sql
-- Daily response volume for last 30 days
SELECT
  DATE(ss.completedAt) AS date,
  COUNT(DISTINCT ss.id) AS completions
FROM survey_sessions ss
WHERE ss.surveyId = 'SV1'
  AND ss.completedAt >= NOW() - INTERVAL '30 days'
GROUP BY DATE(ss.completedAt)
ORDER BY date;
```

---

## 10. NPS Calculation

### Score Segments

| Score | Segment | Label |
|---|---|---|
| 9–10 | Promoters | Love you, will recommend |
| 7–8 | Passives | Neutral, won't promote |
| 0–6 | Detractors | Unhappy, risk of churn |

### Formula

```
NPS = ((Promoters - Detractors) / Total Respondents) × 100
```

### SQL

```sql
WITH nps_data AS (
  SELECT
    numberValue,
    CASE
      WHEN numberValue >= 9 THEN 'promoter'
      WHEN numberValue >= 7 THEN 'passive'
      ELSE 'detractor'
    END AS segment
  FROM answers
  WHERE questionId = 'Q4'
    AND numberValue IS NOT NULL
),
counts AS (
  SELECT
    COUNT(*) FILTER (WHERE segment = 'promoter')  AS promoters,
    COUNT(*) FILTER (WHERE segment = 'passive')   AS passives,
    COUNT(*) FILTER (WHERE segment = 'detractor') AS detractors,
    COUNT(*) AS total
  FROM nps_data
)
SELECT
  promoters,
  passives,
  detractors,
  total,
  ROUND((promoters - detractors) * 100.0 / total, 1) AS nps_score
FROM counts;
```

**Example Output:**

| promoters | passives | detractors | total | nps_score |
|---|---|---|---|---|
| 600 | 250 | 150 | 1000 | +45.0 |

### NPS Score Benchmarks

| NPS Score | Interpretation |
|---|---|
| > 70 | World-class (Apple, Netflix) |
| 50–70 | Excellent |
| 30–50 | Good |
| 0–30 | Okay, room to improve |
| < 0 | Needs urgent attention |

---

## 11. Index Strategy

### Core Indexes (Already in Schema)

```sql
-- answers table - most queried
CREATE INDEX idx_answers_questionId    ON answers (questionId);
CREATE INDEX idx_answers_sessionId     ON answers (sessionId);
CREATE INDEX idx_answers_optionId      ON answers (optionId);
CREATE INDEX idx_answers_gridRowId     ON answers (gridRowId);
CREATE INDEX idx_answers_createdAt     ON answers (createdAt);

-- Composite indexes for analytics queries
CREATE INDEX idx_answers_q_option      ON answers (questionId, optionId);
CREATE INDEX idx_answers_q_grid_option ON answers (questionId, gridRowId, optionId);

-- sessions table
CREATE INDEX idx_sessions_surveyId    ON survey_sessions (surveyId);
CREATE INDEX idx_sessions_status      ON survey_sessions (status);
CREATE INDEX idx_sessions_completed   ON survey_sessions (completedAt);

-- questions table
CREATE INDEX idx_questions_surveyId   ON questions (surveyId, orderIndex);
```

---

## 12. Scalability Considerations

### Handling Millions of Responses

| Strategy | Implementation |
|---|---|
| **Partitioning** | Partition `answers` by `createdAt` (monthly) |
| **Read Replicas** | Route analytics queries to read replicas |
| **Materialized Views** | Pre-compute option counts every 5 minutes |
| **Caching** | Cache analytics results in Redis (TTL: 60s) |
| **Async Processing** | Write answers to queue (Kafka/SQS), process async |

### Table Partitioning

```sql
-- Partition answers by month for large-scale deployments
CREATE TABLE answers (
  id          TEXT,
  sessionId   TEXT NOT NULL,
  questionId  TEXT NOT NULL,
  optionId    TEXT,
  gridRowId   TEXT,
  textValue   TEXT,
  numberValue FLOAT,
  jsonValue   JSONB,
  createdAt   TIMESTAMPTZ NOT NULL DEFAULT NOW()
) PARTITION BY RANGE (createdAt);

CREATE TABLE answers_2024_01 PARTITION OF answers
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE answers_2024_02 PARTITION OF answers
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
-- etc.
```

### Materialized View for Live Dashboards

```sql
CREATE MATERIALIZED VIEW mv_option_counts AS
SELECT
  questionId,
  optionId,
  COUNT(*) AS count
FROM answers
WHERE optionId IS NOT NULL
GROUP BY questionId, optionId;

-- Refresh every 5 minutes
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_option_counts;
```

---

## 13. Future Extensions

### Conditional Logic / Skip Logic

```json
// On Question table, in config field:
{
  "conditions": [
    {
      "if": { "questionId": "Q1", "optionId": "O2" },
      "then": "SHOW_QUESTION",
      "targetQuestionId": "Q5"
    }
  ]
}
```

### Survey Scoring

```json
// On QuestionOption, add a score field
{ "text": "Strongly Agree", "score": 5 }
{ "text": "Disagree", "score": 1 }
```

```sql
-- Total score per session
SELECT sessionId, SUM(o.score) AS total_score
FROM answers a
JOIN question_options o ON a.optionId = o.id
WHERE a.questionId IN ('Q1','Q2','Q3')
GROUP BY sessionId;
```

### Survey Versioning

```prisma
model SurveyVersion {
  id        String   @id @default(cuid())
  surveyId  String
  version   Int
  snapshot  Json     // Full survey JSON snapshot
  createdAt DateTime @default(now())
}
```

### AI Analytics Integration

```
Answer Data → Vector Embeddings (for text answers)
           → Sentiment Analysis
           → Topic Clustering
           → Anomaly Detection
```

### Other Future Features

| Feature | Approach |
|---|---|
| A/B Testing | `variant` field on Survey |
| Offline Surveys | Queue answers locally, sync on connect |
| Multi-language | `translations` JSON on Question |
| Survey Templates | Seed data + template flag on Survey |
| Webhooks | Trigger on session completion |
| Export (CSV/PDF) | Background job reads answers + formats |

---

## 14. Production Best Practices

### ✅ Do This

| Practice | Reason |
|---|---|
| Use `config` JSON for type-specific settings | Avoid schema migrations for new types |
| Use `type` as a string (not enum) | Add new types without DB migration |
| Keep answers generic (4 value columns) | One table handles all question types |
| Index `(questionId, optionId)` composite | Fast analytics GROUP BY |
| Store NPS/Rating as `numberValue` | Enables AVG(), percentile queries |
| Use `SurveySession` to group answers | Enables completion rate analytics |

### ❌ Avoid This

| Anti-Pattern | Problem |
|---|---|
| Separate table per question type | Schema explosion, hard to maintain |
| Enum for question types | Can't add types without migration |
| Storing answers in a single JSON blob | Can't query/aggregate individual answers |
| No indexing on answers table | Queries timeout at 1M+ rows |
| Hardcoding NPS segments | Business may want to adjust thresholds |

---

## 15. Final Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                         SURVEY ENGINE                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Survey                                                          │
│   ├── id, title, status, settings (JSON)                         │
│   │                                                              │
│   └── Questions []                                               │
│        ├── id, text, type, config (JSON), orderIndex             │
│        │                                                         │
│        ├── Options []         ← COLUMNS (for choice + grid)      │
│        │    ├── "BJP"                                            │
│        │    ├── "Congress"                                       │
│        │    └── "AAP"                                            │
│        │                                                         │
│        ├── GridRows []        ← ROWS (for grid questions only)   │
│        │    ├── "Service"                                        │
│        │    ├── "Price"                                          │
│        │    └── "Support"                                        │
│        │                                                         │
│        └── Answers []         ← ALL responses stored here        │
│             ├── optionId      → Radio, Checkbox, Dropdown        │
│             ├── numberValue   → Rating, NPS, Number              │
│             ├── textValue     → Text, Email, "Other" input       │
│             ├── gridRowId     → Grid row reference               │
│             └── jsonValue     → Location, Media, File            │
│                                                                  │
│  SurveySession                                                   │
│   ├── id, userId, status                                         │
│   ├── startedAt, completedAt                                     │
│   └── answers []              ← Groups all answers per session   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

ANALYTICS FLOW:
  answers
    ├── GROUP BY optionId    → Radio / Checkbox / Dropdown
    ├── AVG(numberValue)     → Rating / NPS
    ├── GROUP BY gridRowId + optionId → Grid Questions
    └── Segment numberValue  → NPS (Promoter/Passive/Detractor)
```

### The Single Most Important Mental Model

```
Every answer — regardless of question type — is just:

  WHO answered? ────────── sessionId
  WHAT question? ────────── questionId
  WHICH option chosen? ──── optionId      (nullable)
  WHICH grid row? ────────── gridRowId     (nullable)
  WHAT text did they type? ─ textValue     (nullable)
  WHAT number did they give? numberValue   (nullable)
  WHAT rich data? ────────── jsonValue     (nullable)

One row per answer. Multiple rows per session for multi-selects.
```

---

*Generated for Engineering Teams · Enterprise Survey Architecture Reference · Version 1.0*
