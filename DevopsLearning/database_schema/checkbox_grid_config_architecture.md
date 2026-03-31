# Checkbox Grid Survey Architecture — Using Config JSON
### Engineering Reference: Dynamic UI, Schema Design, Data Flow & Analytics

---

## Table of Contents

1. [Overview](#1-overview)
2. [Example Survey](#2-example-survey)
3. [Question Table Schema](#3-question-table-schema)
4. [Config JSON — Checkbox Grid](#4-config-json--checkbox-grid)
5. [Grid Rows Table](#5-grid-rows-table)
6. [Options Table](#6-options-table)
7. [UI Generation Flow](#7-ui-generation-flow)
8. [User Response JSON](#8-user-response-json)
9. [Answer Table Schema](#9-answer-table-schema)
10. [Stored Answer Example](#10-stored-answer-example)
11. [Full Data Flow](#11-full-data-flow)
12. [Analytics Queries](#12-analytics-queries)
13. [Analytics Output](#13-analytics-output)
14. [Chart Generation](#14-chart-generation)
15. [Indexing](#15-indexing)
16. [Why Config JSON Is Powerful](#16-why-config-json-is-powerful)
17. [Final Architecture](#17-final-architecture)

---

## 1. Overview

### What Is a Checkbox Grid?

A **Checkbox Grid** is a two-dimensional survey question where:

- **Rows** represent subjects being evaluated (e.g., Village A, Village B)
- **Columns** represent selectable options (e.g., Water, Road, Electricity)
- Each cell contains a **checkbox** — users may tick **any number of columns per row**

Unlike Radio Grid (one selection per row), Checkbox Grid allows multiple selections per row, making it suitable for multi-issue, multi-attribute, or multi-factor questions.

### Why Is Checkbox Grid Complex?

| Complexity | Reason |
|---|---|
| Multi-select per row | Each row independently stores 0 to N selected options |
| "Other" support | One column is a special free-text input, not a fixed option |
| Storage volume | 10 rows × 5 options = up to 50 answer rows per respondent |
| Dynamic rendering | The grid must be built at runtime from database records |
| Analytics dimension | Queries must aggregate across two axes: row + column |

### Why Use Config JSON?

The `config` field on the `Question` table stores **type-specific behavior settings** as a JSON object. This means:

- No new database columns are needed when a new variant is introduced
- The same `Question` model serves every question type in the system
- UI rendering behavior, validation rules, and feature flags live in one portable JSON blob
- Survey designers can toggle features (like "Other" input) without schema migrations

### Why Scalable Schema Is Required

A naive approach — one table per question type, or storing answer arrays in a JSON column — breaks at scale:

- Array storage cannot be indexed or aggregated in SQL
- Type-specific tables multiply maintenance burden
- Analytics requires parsing stored JSON at query time = catastrophic performance

The correct approach: **one universal `answers` table**, one row per selection, all analytics done via standard SQL `GROUP BY`.

---

## 2. Example Survey

### Question

> **"Which issues exist in each village?"**

### Grid Layout

| Village | Water | Road | Electricity | Other |
|---|---|---|---|---|
| Village A | ☐ | ☐ | ☐ | ☐ |
| Village B | ☐ | ☐ | ☐ | ☐ |

### Respondent Fills the Grid

**Village A** checks:
- ✅ Water
- ✅ Electricity
- ✅ Other → types: `"Internet"`

**Village B** checks:
- ✅ Road

### Storage Implication

| Selection | gridRowId | optionId | textValue |
|---|---|---|---|
| Village A → Water | R1 | O1 | null |
| Village A → Electricity | R1 | O3 | null |
| Village A → Other | R1 | O4 | "Internet" |
| Village B → Road | R2 | O2 | null |

**4 answer rows** are inserted for this single respondent answering this single question.

---

## 3. Question Table Schema

```prisma
model Question {
  id          Int          @id @default(autoincrement())
  surveyId    Int
  text        String       @db.Text
  type        QuestionType
  config      Json?
  isRequired  Boolean      @default(false)
  orderIndex  Int          @default(0)
  validation  Json?
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
  description String?      @db.Text

  options  QuestionOption[]
  gridRows QuestionGridRow[]
  answers  Answer[]

  @@index([surveyId])
  @@map("questions")
}

enum QuestionType {
  SHORT_TEXT
  LONG_TEXT
  RADIO
  CHECKBOX
  DROPDOWN
  RATING
  NPS
  RADIO_GRID
  CHECKBOX_GRID
  DROPDOWN_GRID
  RATING_GRID
  NPS_GRID
  MEDIA
  LOCATION
}
```

### Purpose of Each Key Field

| Field | Purpose |
|---|---|
| `type` | Tells the frontend which component to render and the backend how to validate |
| `config` | Stores type-specific settings as JSON — no extra columns needed |
| `validation` | JSON rules like `{ "minSelections": 1, "maxSelections": 3 }` |
| `orderIndex` | Controls question display order within the survey |
| `isRequired` | Frontend and backend validation — must have at least one answer row |

### The `config` Field Is the Key Innovation

Without `config`, every new question variant requires a schema migration. With `config`, the database row stays identical — only the JSON payload changes. The frontend reads `config` to decide exactly how to render the question.

---

## 4. Config JSON — Checkbox Grid

### Full Config Example

```json
{
  "variant": "checkbox-grid",
  "allowOther": true,
  "otherPlaceholder": "Enter other issue",
  "minSelectionsPerRow": 0,
  "maxSelectionsPerRow": null,
  "shuffleOptions": false,
  "shuffleRows": false,
  "displayStyle": "standard"
}
```

### Field Explanations

| Field | Type | Description |
|---|---|---|
| `variant` | string | Identifies the exact rendering variant. The `type` field says `CHECKBOX_GRID`; `variant` refines it further (e.g., `checkbox-grid`, `checkbox-grid-compact`, `checkbox-grid-matrix`) |
| `allowOther` | boolean | When `true`, the "Other" option renders with an attached text input. When `false`, "Other" is treated as a plain checkbox even if `isOther=true` in the options table |
| `otherPlaceholder` | string | The placeholder text shown inside the "Other" free-text input. Kept in config so survey designers can customize it without modifying the options table |
| `minSelectionsPerRow` | int \| null | Minimum checkboxes a respondent must tick per row. `0` = optional. Used alongside `validation` JSON for backend enforcement |
| `maxSelectionsPerRow` | int \| null | Maximum checkboxes allowed per row. `null` = unlimited |
| `shuffleOptions` | boolean | If `true`, the frontend randomizes column order per respondent (reduces bias) |
| `shuffleRows` | boolean | If `true`, row order is randomized |
| `displayStyle` | string | `"standard"` \| `"compact"` \| `"scrollable"` — controls CSS layout class applied by the frontend |

### Minimal Config (Most Common)

```json
{
  "variant": "checkbox-grid",
  "allowOther": true,
  "otherPlaceholder": "Describe the issue..."
}
```

Unspecified fields fall back to defaults in the frontend rendering logic.

### Config for "No Other" Variant

```json
{
  "variant": "checkbox-grid",
  "allowOther": false
}
```

Even if the options table contains a row with `isOther=true`, the frontend will not render the text input when `allowOther: false`. Config overrides table data for UI behavior.

---

## 5. Grid Rows Table

```prisma
model QuestionGridRow {
  id         Int      @id @default(autoincrement())
  questionId Int
  text       String
  orderIndex Int      @default(0)
  createdAt  DateTime @default(now())

  question Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  answers  Answer[]

  @@index([questionId])
  @@map("question_grid_rows")
}
```

### Example Data

| id | questionId | text | orderIndex |
|---|---|---|---|
| 1 | 10 | Village A | 1 |
| 2 | 10 | Village B | 2 |

### Design Notes

- Each row is an independent entity. Answering Row 1 has no coupling to Row 2.
- `orderIndex` governs top-to-bottom display order in the rendered grid.
- Adding a new village = one `INSERT` into this table. No migration, no code change.
- `onDelete: Cascade` ensures orphaned rows are cleaned up when the question is deleted.
- If `shuffleRows: true` is in config, the frontend ignores `orderIndex` and randomizes display.

---

## 6. Options Table

```prisma
model QuestionOption {
  id         Int      @id @default(autoincrement())
  questionId Int
  text       String
  orderIndex Int      @default(0)
  isOther    Boolean  @default(false)
  createdAt  DateTime @default(now())

  question Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  answers  Answer[]

  @@index([questionId])
  @@map("question_options")
}
```

### Example Data

| id | questionId | text | orderIndex | isOther |
|---|---|---|---|---|
| 1 | 10 | Water | 1 | false |
| 2 | 10 | Road | 2 | false |
| 3 | 10 | Electricity | 3 | false |
| 4 | 10 | Other | 4 | **true** |

### Why `isOther` Is Needed

The `isOther` flag is the bridge between the structured options system and the free-text capture system. It serves three distinct purposes:

**1. UI rendering signal**
When the frontend iterates over options to render columns, it checks `isOther`. For `isOther=true`, it renders a checkbox AND an adjacent text input. For `isOther=false`, it renders a checkbox only.

```
isOther=false → [☐] Water
isOther=true  → [☐] Other: [________________]
```

**2. API validation signal**
When the backend receives an answer with `optionId=4`, it knows to also expect and validate a `textValue`. If `allowOther: true` is in config but `textValue` is empty, the backend can optionally warn or reject.

**3. Analytics separation signal**
Analytics queries use `isOther` to separate structured counts (aggregatable) from free-text responses (qualitative). The heatmap query filters `WHERE isOther=false`. The word-cloud query filters `WHERE isOther=true`.

---

## 7. UI Generation Flow

### How the Frontend Builds the Grid

The frontend makes a single API call to fetch the question and receives this response:

```json
{
  "question": {
    "id": 10,
    "text": "Which issues exist in each village?",
    "type": "CHECKBOX_GRID",
    "config": {
      "variant": "checkbox-grid",
      "allowOther": true,
      "otherPlaceholder": "Describe the issue..."
    }
  },
  "gridRows": [
    { "id": 1, "text": "Village A", "orderIndex": 1 },
    { "id": 2, "text": "Village B", "orderIndex": 2 }
  ],
  "options": [
    { "id": 1, "text": "Water",       "orderIndex": 1, "isOther": false },
    { "id": 2, "text": "Road",        "orderIndex": 2, "isOther": false },
    { "id": 3, "text": "Electricity", "orderIndex": 3, "isOther": false },
    { "id": 4, "text": "Other",       "orderIndex": 4, "isOther": true  }
  ]
}
```

### Frontend Rendering Logic

```javascript
function renderCheckboxGrid(question, gridRows, options, config) {
  // 1. Build table header from options
  const headers = options.map(o => o.text);

  // 2. Build table body — one row per gridRow
  gridRows.forEach(row => {
    options.forEach(option => {
      if (option.isOther && config.allowOther) {
        // Render: checkbox + text input
        renderCheckboxWithTextInput(row, option, config.otherPlaceholder);
      } else {
        // Render: plain checkbox
        renderCheckbox(row, option);
      }
    });
  });
}
```

### What the Grid Looks Like (Rendered)

```
                 Water    Road    Electricity    Other
Village A        [☐]      [☐]     [☐]           [☐] [_____________]
Village B        [☐]      [☐]     [☐]           [☐] [_____________]
```

The text input next to "Other" appears because `config.allowOther = true` AND `option.isOther = true`. If either is false, the text input is hidden.

---

## 8. User Response JSON

After the respondent submits, the frontend sends this payload to `POST /api/answers`:

```json
{
  "sessionId": 101,
  "questionId": 10,
  "answers": [
    {
      "gridRowId": 1,
      "optionId": 1,
      "textValue": null
    },
    {
      "gridRowId": 1,
      "optionId": 3,
      "textValue": null
    },
    {
      "gridRowId": 1,
      "optionId": 4,
      "textValue": "Internet"
    },
    {
      "gridRowId": 2,
      "optionId": 2,
      "textValue": null
    }
  ]
}
```

### Payload Rules

| Rule | Reason |
|---|---|
| One object per checkbox ticked | Mirrors the one-row-per-answer storage model |
| `textValue` is always present in payload | `null` for normal options, string for "Other" |
| Unticked checkboxes are NOT sent | Absence = not selected. No need to store negative answers |
| `gridRowId` on every answer | Required — otherwise the backend cannot tell which row the answer belongs to |

### Backend Validation Steps

```
1. Confirm sessionId exists and belongs to this survey
2. Confirm questionId belongs to this survey
3. For each answer:
   a. Confirm gridRowId belongs to this questionId
   b. Confirm optionId belongs to this questionId
   c. If option.isOther=true AND config.allowOther=true → accept textValue
   d. If option.isOther=false → reject textValue (ignore or error)
4. Check minSelectionsPerRow from config.validation
5. Bulk INSERT all valid answers
```

---

## 9. Answer Table Schema

```prisma
model Answer {
  id         Int      @id @default(autoincrement())
  sessionId  Int
  questionId Int

  // Set for all choice-based questions (radio, checkbox, dropdown, grid)
  optionId   Int?

  // Set for all grid-type questions (identifies which row)
  gridRowId  Int?

  // Set when isOther=true — stores the free-text input
  textValue  String?  @db.Text

  // Set for location, file upload, signature questions
  jsonValue  Json?

  createdAt  DateTime @default(now())

  session  SurveySession    @relation(fields: [sessionId], references: [id])
  question Question         @relation(fields: [questionId], references: [id])
  option   QuestionOption?  @relation(fields: [optionId], references: [id])
  gridRow  QuestionGridRow? @relation(fields: [gridRowId], references: [id])

  @@index([questionId])
  @@index([sessionId])
  @@index([optionId])
  @@index([gridRowId])
  @@index([questionId, gridRowId, optionId])
  @@map("answers")
}
```

### Column Usage by Question Type

| Column | Checkbox Grid | Radio | NPS | Text | Location |
|---|---|---|---|---|---|
| `optionId` | ✅ (column selected) | ✅ | — | — | — |
| `gridRowId` | ✅ (row selected) | — | — | — | — |
| `textValue` | ✅ (if "Other") | ✅ (if "Other") | — | ✅ | — |
| `jsonValue` | — | — | — | — | ✅ |
| `numberValue`* | — | — | ✅ | — | — |

*Add `numberValue Float?` for Rating/NPS questions.

### Why `optionId` and `gridRowId` Are Both Nullable

Not every question type uses both. A plain `CHECKBOX` question uses `optionId` but never `gridRowId`. A `CHECKBOX_GRID` question uses both. Making both nullable keeps the table universal across all question types without separate tables per type.

---

## 10. Stored Answer Example

### Answer Table After One Respondent Submits

| id | sessionId | questionId | gridRowId | optionId | textValue | createdAt |
|---|---|---|---|---|---|---|
| 501 | 101 | 10 | 1 | 1 | null | 2024-12-01 10:00 |
| 502 | 101 | 10 | 1 | 3 | null | 2024-12-01 10:00 |
| 503 | 101 | 10 | 1 | 4 | Internet | 2024-12-01 10:00 |
| 504 | 101 | 10 | 2 | 2 | null | 2024-12-01 10:00 |

### What Each Row Represents

| Row id | Meaning |
|---|---|
| 501 | Session 101 · Village A · Water selected |
| 502 | Session 101 · Village A · Electricity selected |
| 503 | Session 101 · Village A · Other selected + text "Internet" captured |
| 504 | Session 101 · Village B · Road selected |

### Why Multiple Rows?

Storing multiple rows per respondent (rather than a JSON array in a single column) is the single most important design decision. It enables:

```sql
-- This query is instant with an index:
SELECT COUNT(*) FROM answers WHERE questionId=10 AND optionId=1;

-- This query would require JSON parsing (catastrophic at scale):
SELECT COUNT(*) FROM answers WHERE JSON_CONTAINS(selectedOptions, '1');
```

One row per selection = O(1) lookup with a B-tree index. JSON array storage = full table scan with parsing overhead.

---

## 11. Full Data Flow

```
╔══════════════════════════════════════════════════╗
║  STEP 1 — SURVEY SETUP (Admin / Designer)        ║
║                                                  ║
║  a. Create Survey record                         ║
║  b. Create Question:                             ║
║     type = CHECKBOX_GRID                         ║
║     config = { variant, allowOther,              ║
║                otherPlaceholder }                ║
║  c. Insert QuestionOption rows:                  ║
║     Water(isOther=false), Road(isOther=false),   ║
║     Electricity(isOther=false), Other(true)      ║
║  d. Insert QuestionGridRow rows:                 ║
║     Village A, Village B                         ║
╚══════════════════════════════════════════════════╝
                    │
                    ▼ Survey is now live
╔══════════════════════════════════════════════════╗
║  STEP 2 — RESPONDENT OPENS SURVEY                ║
║                                                  ║
║  a. Frontend calls GET /api/survey/:id           ║
║  b. API returns:                                 ║
║     question + config + gridRows + options       ║
║  c. Frontend reads config.variant                ║
║     → renders CHECKBOX_GRID component            ║
║  d. config.allowOther=true + option.isOther=true ║
║     → shows text input next to "Other" column    ║
╚══════════════════════════════════════════════════╝
                    │
                    ▼ User ticks checkboxes
╔══════════════════════════════════════════════════╗
║  STEP 3 — ANSWER SUBMISSION                      ║
║                                                  ║
║  a. Frontend builds answer payload:              ║
║     one object per checkbox ticked               ║
║  b. POST /api/answers                            ║
║  c. API validates:                               ║
║     - gridRowId ∈ this question's rows           ║
║     - optionId  ∈ this question's options        ║
║     - textValue present if isOther=true          ║
║  d. Bulk INSERT into answers table               ║
║  e. Mark session as COMPLETED                    ║
╚══════════════════════════════════════════════════╝
                    │
                    ▼
╔══════════════════════════════════════════════════╗
║  STEP 4 — ANALYTICS                              ║
║                                                  ║
║  a. GROUP BY gridRowId + optionId → heatmap      ║
║  b. GROUP BY optionId → most common issue        ║
║  c. WHERE isOther=true → word cloud              ║
║  d. Materialized view caches results             ║
║  e. Dashboard renders charts                     ║
╚══════════════════════════════════════════════════╝
```

---

## 12. Analytics Queries

### Query 1 — Count Per Option (Overall)

```sql
SELECT
  o.text          AS issue,
  COUNT(a.id)     AS total_reported
FROM answers a
JOIN question_options o ON a.optionId = o.id
WHERE a.questionId = 10
  AND o.isOther = false
GROUP BY o.id, o.text
ORDER BY total_reported DESC;
```

---

### Query 2 — Count Per Grid Row (Per Village)

```sql
SELECT
  gr.text         AS village,
  COUNT(a.id)     AS total_issues
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
WHERE a.questionId = 10
GROUP BY gr.id, gr.text
ORDER BY total_issues DESC;
```

---

### Query 3 — Full Heatmap (Village × Issue)

```sql
SELECT
  gr.text         AS village,
  o.text          AS issue,
  COUNT(a.id)     AS count
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId  = gr.id
JOIN question_options   o  ON a.optionId   = o.id
WHERE a.questionId = 10
  AND o.isOther = false
GROUP BY gr.id, gr.text, gr.orderIndex,
         o.id,  o.text,  o.orderIndex
ORDER BY gr.orderIndex, o.orderIndex;
```

---

### Query 4 — "Other" Free-Text Responses

```sql
SELECT
  gr.text         AS village,
  a.textValue     AS custom_issue,
  COUNT(*)        AS times_written
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
JOIN question_options   o  ON a.optionId  = o.id
WHERE a.questionId = 10
  AND o.isOther = true
  AND a.textValue IS NOT NULL
  AND a.textValue <> ''
GROUP BY gr.text, a.textValue
ORDER BY gr.text, times_written DESC;
```

---

### Query 5 — Percentage Per Row

```sql
WITH row_totals AS (
  SELECT gridRowId, COUNT(*) AS row_total
  FROM answers
  WHERE questionId = 10
  GROUP BY gridRowId
)
SELECT
  gr.text                          AS village,
  o.text                           AS issue,
  COUNT(a.id)                      AS count,
  ROUND(COUNT(a.id) * 100.0
    / rt.row_total, 1)             AS pct
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
JOIN question_options   o  ON a.optionId  = o.id
JOIN row_totals rt          ON rt.gridRowId = a.gridRowId
WHERE a.questionId = 10
GROUP BY gr.id, gr.text, o.id, o.text, rt.row_total
ORDER BY gr.orderIndex, o.orderIndex;
```

---

### Query 6 — Trend Over Time

```sql
SELECT
  DATE_TRUNC('week', a.createdAt) AS week,
  gr.text                          AS village,
  o.text                           AS issue,
  COUNT(*)                         AS weekly_count
FROM answers a
JOIN question_grid_rows gr ON a.gridRowId = gr.id
JOIN question_options   o  ON a.optionId  = o.id
WHERE a.questionId = 10
  AND o.isOther = false
  AND a.createdAt >= NOW() - INTERVAL '90 days'
GROUP BY week, gr.text, o.text
ORDER BY week, gr.text, o.text;
```

---

## 13. Analytics Output

### Heatmap Table (Query 3 Output)

| Village | Water | Road | Electricity | Other |
|---|---|---|---|---|
| Village A | 823 | 341 | 654 | 112 |
| Village B | 210 | 789 | 445 | 67 |

### Percentage Breakdown (Query 5 Output)

| Village | Issue | Count | Percentage |
|---|---|---|---|
| Village A | Water | 823 | 44.3% |
| Village A | Electricity | 654 | 35.2% |
| Village A | Road | 341 | 18.3% |
| Village A | Other | 112 | 6.0% |
| Village B | Road | 789 | 52.1% |
| Village B | Electricity | 445 | 29.4% |
| Village B | Water | 210 | 13.9% |
| Village B | Other | 67 | 4.4% |

### "Other" Word Cloud Data (Query 4 Output)

| Village | Custom Issue | Times Written |
|---|---|---|
| Village A | Internet | 45 |
| Village A | Drainage | 38 |
| Village A | Garbage collection | 22 |
| Village B | Street lighting | 41 |
| Village B | Internet | 26 |
| Village B | Open drainage | 18 |

---

## 14. Chart Generation

### Grouped Bar Chart

One group per village. Each bar in the group = one issue.

```javascript
// Prepare data from Query 3 output
function prepareGroupedBar(heatmapRows) {
  const villages = [...new Set(heatmapRows.map(r => r.village))];
  const issues   = [...new Set(heatmapRows.map(r => r.issue))];

  return {
    labels: villages,
    datasets: issues.map(issue => ({
      label: issue,
      data: villages.map(village => {
        const match = heatmapRows.find(
          r => r.village === village && r.issue === issue
        );
        return match ? match.count : 0;
      })
    }))
  };
}

// Chart.js config
const groupedBarConfig = {
  type: 'bar',
  data: prepareGroupedBar(heatmapRows),
  options: {
    scales: { x: { stacked: false }, y: { stacked: false } }
  }
};
```

### Stacked Bar Chart

Same dataset — change `stacked: true` to show total issue burden per village.

```javascript
const stackedBarConfig = {
  type: 'bar',
  data: prepareGroupedBar(heatmapRows),
  options: {
    scales: { x: { stacked: true }, y: { stacked: true } }
  }
};
```

### Heatmap (D3 or custom)

```javascript
// Pivot heatmap rows into a matrix
function prepareHeatmap(heatmapRows) {
  const villages = [...new Set(heatmapRows.map(r => r.village))];
  const issues   = [...new Set(heatmapRows.map(r => r.issue))];
  const maxCount = Math.max(...heatmapRows.map(r => r.count));

  return {
    villages,
    issues,
    cells: heatmapRows.map(r => ({
      x: issues.indexOf(r.issue),
      y: villages.indexOf(r.village),
      value: r.count,
      // Normalize 0–1 for color intensity
      intensity: r.count / maxCount
    }))
  };
}
```

Color scale: `intensity → 0.0 = white → 1.0 = deep blue/red` depending on design system.

### Summary Table

No transformation needed — render Query 5 output directly as an HTML table. Apply conditional background colors: green < 25%, amber 25–50%, red > 50%.

### "Other" Word Cloud

```javascript
// Feed into WordCloud2.js or D3-cloud
const wordCloudInput = otherRows.map(r => [r.custom_issue, r.times_written]);
WordCloud(document.getElementById('cloud'), { list: wordCloudInput });
```

---

## 15. Indexing

### Recommended Indexes

```sql
-- Core lookup indexes
CREATE INDEX idx_answers_questionId  ON answers (questionId);
CREATE INDEX idx_answers_sessionId   ON answers (sessionId);
CREATE INDEX idx_answers_optionId    ON answers (optionId);
CREATE INDEX idx_answers_gridRowId   ON answers (gridRowId);
CREATE INDEX idx_answers_createdAt   ON answers (createdAt);

-- Composite: covers the full heatmap query (most important)
CREATE INDEX idx_answers_heatmap
  ON answers (questionId, gridRowId, optionId);

-- Partial: covers "Other" text queries (sparse = cheap)
CREATE INDEX idx_answers_other_text
  ON answers (questionId, gridRowId, textValue)
  WHERE textValue IS NOT NULL;

-- Supports trend queries
CREATE INDEX idx_answers_question_date
  ON answers (questionId, createdAt);

-- Sessions: for completion-rate analytics
CREATE INDEX idx_sessions_survey_status
  ON survey_sessions (surveyId, status, completedAt);
```

### Index Cost vs Query Coverage

| Index | Covers | Write Cost |
|---|---|---|
| `(questionId)` | All analytics base filter | Low |
| `(questionId, gridRowId, optionId)` | Full heatmap GROUP BY | Medium |
| `(textValue) WHERE NOT NULL` | "Other" word cloud | Very low (partial, sparse) |
| `(questionId, createdAt)` | Trend queries | Low |

### Materialized View for Dashboard Caching

```sql
CREATE MATERIALIZED VIEW mv_grid_counts AS
SELECT
  questionId,
  gridRowId,
  optionId,
  COUNT(*) AS cnt
FROM answers
WHERE optionId IS NOT NULL
  AND gridRowId IS NOT NULL
GROUP BY questionId, gridRowId, optionId;

-- Refresh every 5 minutes (non-blocking)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_grid_counts;
```

Dashboards query `mv_grid_counts` instead of `answers` directly. This reduces query time from seconds to milliseconds at scale.

---

## 16. Why Config JSON Is Powerful

### Dynamic UI Without Code Changes

When a new config key is added (e.g., `"allowNA": true` to add a "Not Applicable" column), the frontend can start reading it immediately. No database migration, no new column, no deployment of a new API version.

```json
// Before: standard grid
{ "variant": "checkbox-grid", "allowOther": true }

// After: same DB row, new behavior
{ "variant": "checkbox-grid", "allowOther": true, "allowNA": true, "shuffleRows": true }
```

### No Schema Change for New Variants

Every new grid variant reuses the same `Question` model:

| Variant | Config Change | Schema Change |
|---|---|---|
| Standard Checkbox Grid | `"variant": "checkbox-grid"` | None |
| Compact Grid | `"variant": "checkbox-grid-compact"` | None |
| Grid with NA column | `"allowNA": true` | None |
| Grid with max-select | `"maxSelectionsPerRow": 2` | None |
| Grid with shuffle | `"shuffleOptions": true` | None |
| Grid with image options | `"optionDisplayMode": "image"` | None |

### Config as Feature Flags

Config doubles as a runtime feature flag system. You can roll out new features to a subset of surveys by setting flags in config without changing any survey engine code:

```json
{
  "variant": "checkbox-grid",
  "experimentalInlineValidation": true,
  "betaProgressiveReveal": false
}
```

### Future Question Types Require Zero Table Changes

The same `answers` table and the same `Question` model will handle question types that don't exist yet. The config JSON absorbs all new settings. This is why Google Forms, Typeform, and SurveyMonkey can ship new question types without downtime or migrations.

---

## 17. Final Architecture

```
Survey
 └── Question
       │  id=10
       │  type=CHECKBOX_GRID
       │  config = {
       │    "variant": "checkbox-grid",
       │    "allowOther": true,
       │    "otherPlaceholder": "Describe the issue..."
       │  }
       │
       ├── QuestionOption []              ← COLUMNS of the grid
       │    ├── id=1  text="Water"        isOther=false
       │    ├── id=2  text="Road"         isOther=false
       │    ├── id=3  text="Electricity"  isOther=false
       │    └── id=4  text="Other"        isOther=true ← config.allowOther gates text input
       │
       ├── QuestionGridRow []             ← ROWS of the grid
       │    ├── id=1  text="Village A"
       │    └── id=2  text="Village B"
       │
       └── Answer []                      ← ONE ROW PER CHECKBOX TICKED
            ├── { sessionId=101, questionId=10, gridRowId=1, optionId=1, textValue=null }
            │   → Village A → Water
            ├── { sessionId=101, questionId=10, gridRowId=1, optionId=3, textValue=null }
            │   → Village A → Electricity
            ├── { sessionId=101, questionId=10, gridRowId=1, optionId=4, textValue="Internet" }
            │   → Village A → Other + "Internet"
            └── { sessionId=101, questionId=10, gridRowId=2, optionId=2, textValue=null }
                → Village B → Road

ANALYTICS:
  answers (indexed) ──→ GROUP BY gridRowId + optionId ──→ Heatmap
                   ──→ GROUP BY optionId              ──→ Bar chart
                   ──→ WHERE isOther=true             ──→ Word cloud
                   ──→ Materialized view              ──→ Dashboard cache
```

### The One-Sentence Mental Model

> The `config` JSON tells the **frontend how to render** the question; the `type` enum tells the **backend how to store and validate** answers; and the `answers` table — one row per checkbox ticked — makes every analytics query a simple `GROUP BY`.

---

*Enterprise Survey Engine · Checkbox Grid With Config · Engineering Reference v1.0*
