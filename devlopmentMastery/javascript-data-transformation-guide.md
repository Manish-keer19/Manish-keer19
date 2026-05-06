# JavaScript Data Transformation: The Complete Production Guide

> **Goal:** After reading this guide, you can convert ANY input data structure into ANY required output format — Excel rows, API payloads, PDFs, analytics structures, UI tables, and more.

---

## Table of Contents

1. [Core Mental Model](#1-core-mental-model)
2. [Deep Dive: Core Transformation Functions](#2-deep-dive-core-transformation-functions)
3. [Advanced Patterns](#3-advanced-patterns)
4. [Real-World Scenarios](#4-real-world-scenarios)
5. [Excel & Reporting Use Cases](#5-excel--reporting-use-cases)
6. [Common Mistakes](#6-common-mistakes)
7. [Performance & Clean Code](#7-performance--clean-code)
8. [Practice Problems](#8-practice-problems)

---

## 1. Core Mental Model

### 1.1 Think Like a Data Transformer, Not Just a Coder

Before writing a single line of code, answer these three questions:

1. **What is the shape of my input?**
2. **What is the shape of my output?**
3. **What is the transformation rule that maps one to the other?**

Most developers jump straight to code. Transformers sketch the data first.

```
INPUT SHAPE         TRANSFORMATION RULE        OUTPUT SHAPE
-----------         -------------------        ------------
Array of objects -> pick/rename fields      -> Flat rows for Excel
Nested API res   -> flatten + join          -> UI table format
Survey answers   -> group by user           -> One row per respondent
Raw events       -> count + sum             -> Analytics dashboard
```

---

### 1.2 Analyze Input vs Output Structure

Always audit both ends before coding:

```js
// STEP 1: Log your input and inspect it
console.log(JSON.stringify(inputData, null, 2));

// STEP 2: Sketch the target output manually
const targetExample = {
  userId: "u_001",
  fullName: "Aditya Sharma",
  totalOrders: 5,
  revenue: 4200,
};

// STEP 3: Now write the transformation
```

**Classification checklist for your input:**

| Question                          | Implication                          |
| --------------------------------- | ------------------------------------ |
| Is it an array or object?         | Determines if you iterate or destruct |
| Is it flat or nested?             | May need flattening first            |
| Are keys dynamic or static?       | May need computed/dynamic keys       |
| Are values optional/null?         | Need null-safety guards              |
| Does it need joining with other data? | Need merge/lookup strategy        |

---

### 1.3 The Step-by-Step Framework: Input → Target → Transformation

```
┌─────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   ANALYZE   │ -> │    DECOMPOSE     │ -> │    TRANSFORM    │
│   Input     │    │  Break into      │    │  Apply rules    │
│   Output    │    │  atomic steps    │    │  step by step   │
└─────────────┘    └──────────────────┘    └─────────────────┘
```

**Practical example — Survey data to Excel:**

```js
// INPUT: Nested survey sessions with dynamic questions
const input = [
  {
    sessionId: "s1",
    userId: "u1",
    completedAt: "2024-01-15",
    answers: [
      { questionId: "q1", question: "Age group?", answer: "25-34" },
      { questionId: "q2", question: "City?", answer: "Mumbai" },
    ],
  },
];

// TARGET: Flat object per user for Excel rows
// { sessionId, userId, completedAt, "Age group?": "25-34", "City?": "Mumbai" }

// DECOMPOSE the transformation:
// Step 1 → Extract top-level fields (sessionId, userId, completedAt)
// Step 2 → Extract answers array
// Step 3 → Convert answers array into { [question]: answer } object
// Step 4 → Merge top-level fields + answers object

// TRANSFORM:
const rows = input.map((session) => {
  const base = {
    sessionId: session.sessionId,
    userId: session.userId,
    completedAt: session.completedAt,
  };

  const answers = session.answers.reduce((acc, a) => {
    acc[a.question] = a.answer;
    return acc;
  }, {});

  return { ...base, ...answers };
});

// OUTPUT:
// [{ sessionId: "s1", userId: "u1", completedAt: "2024-01-15", "Age group?": "25-34", "City?": "Mumbai" }]
```

---

## 2. Deep Dive: Core Transformation Functions

### 2.1 `map()` — Shape Transformation

**Syntax:**
```js
const result = array.map((item, index, originalArray) => newItem);
```

**When to use:**
- When you want a **new array of the same length** with each element transformed
- Renaming fields, computing derived values, formatting output

**When NOT to use:**
- When you need to filter out items (use `filter`)
- When you only need side effects with no return value (use `forEach`)
- When you're aggregating into a single value (use `reduce`)

#### Real Backend Example — API response normalization:

```js
// Raw DB response
const dbUsers = [
  { user_id: 1, first_name: "Riya", last_name: "Patel", is_active: 1, created_at: "2024-01-01T10:00:00Z" },
  { user_id: 2, first_name: "Arjun", last_name: "Mehta", is_active: 0, created_at: "2024-02-15T08:30:00Z" },
];

// Transform to camelCase API response
const apiUsers = dbUsers.map((user) => ({
  id: user.user_id,
  fullName: `${user.first_name} ${user.last_name}`,
  isActive: Boolean(user.is_active),
  joinedAt: new Date(user.created_at).toLocaleDateString("en-IN"),
}));

// Result:
// [
//   { id: 1, fullName: "Riya Patel", isActive: true, joinedAt: "1/1/2024" },
//   { id: 2, fullName: "Arjun Mehta", isActive: false, joinedAt: "2/15/2024" }
// ]
```

#### Real Frontend Example — Preparing dropdown options:

```js
const products = [
  { productId: "p1", name: "Laptop", stock: 10 },
  { productId: "p2", name: "Monitor", stock: 0 },
  { productId: "p3", name: "Keyboard", stock: 5 },
];

// Transform for a <select> element
const options = products.map((p) => ({
  value: p.productId,
  label: `${p.name}${p.stock === 0 ? " (Out of Stock)" : ""}`,
  disabled: p.stock === 0,
}));
```

---

### 2.2 `filter()` — Data Pruning

**Syntax:**
```js
const result = array.filter((item, index) => booleanCondition);
```

**When to use:**
- When you want a **subset of the original array**
- Removing inactive records, applying search/sort criteria

**When NOT to use:**
- When you need to transform shape (use `map`)
- When filtering AND transforming at once — chain them: `filter().map()`

#### Real Backend Example — Analytics query:

```js
const orders = [
  { id: "o1", userId: "u1", status: "completed", amount: 1500, createdAt: "2024-03-10" },
  { id: "o2", userId: "u2", status: "refunded",  amount: 800,  createdAt: "2024-03-12" },
  { id: "o3", userId: "u1", status: "completed", amount: 2200, createdAt: "2024-03-15" },
  { id: "o4", userId: "u3", status: "pending",   amount: 500,  createdAt: "2024-03-18" },
];

// Only completed orders from March 2024 for revenue report
const validOrders = orders.filter(
  (o) => o.status === "completed" && o.createdAt.startsWith("2024-03")
);

// Chain with map to get just revenue data
const revenueData = orders
  .filter((o) => o.status === "completed")
  .map((o) => ({ userId: o.userId, amount: o.amount }));
```

#### Real Frontend Example — Live search filter:

```js
function filterUsers(users, searchTerm, activeOnly) {
  return users.filter((user) => {
    const matchesSearch = user.name.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesActive = activeOnly ? user.isActive : true;
    return matchesSearch && matchesActive;
  });
}
```

---

### 2.3 `reduce()` — The Swiss Army Knife

**Syntax:**
```js
const result = array.reduce((accumulator, item, index) => {
  // transform accumulator
  return accumulator;
}, initialValue);
```

**When to use:**
- Summing/counting values
- Transforming an array into an object or different structure
- Grouping, indexing, or building lookup maps
- Any "fold many into one" operation

**When NOT to use:**
- When `map()` or `filter()` is clearer — `reduce` is powerful but can obscure intent
- When you just need a simple sum (`array.reduce((a, b) => a + b, 0)` is fine but `lodash/sumBy` is more readable for complex cases)

#### Real Backend Example — Revenue by category:

```js
const sales = [
  { category: "Electronics", amount: 12000 },
  { category: "Books",       amount: 450 },
  { category: "Electronics", amount: 8500 },
  { category: "Clothing",    amount: 3200 },
  { category: "Books",       amount: 780 },
];

const revenueByCategory = sales.reduce((acc, sale) => {
  acc[sale.category] = (acc[sale.category] || 0) + sale.amount;
  return acc;
}, {});

// Result: { Electronics: 20500, Books: 1230, Clothing: 3200 }
```

#### Real Backend Example — Build a lookup map (O(1) access instead of O(n)):

```js
const users = [
  { id: "u1", name: "Priya" },
  { id: "u2", name: "Rohan" },
  { id: "u3", name: "Neha" },
];

// Bad: orders.map(o => users.find(u => u.id === o.userId))  ← O(n²)

// Good: Build a lookup first
const userMap = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

// Now lookup is O(1)
const orders = [{ orderId: "o1", userId: "u2", amount: 500 }];
const enrichedOrders = orders.map((o) => ({
  ...o,
  userName: userMap[o.userId]?.name ?? "Unknown",
}));
```

#### Real Frontend Example — Cart total with discount:

```js
const cartItems = [
  { name: "Laptop",   price: 55000, qty: 1, discount: 0.1 },
  { name: "Mouse",    price: 1200,  qty: 2, discount: 0 },
  { name: "Keyboard", price: 2800,  qty: 1, discount: 0.05 },
];

const cartSummary = cartItems.reduce(
  (acc, item) => {
    const itemTotal = item.price * item.qty * (1 - item.discount);
    acc.total += itemTotal;
    acc.itemCount += item.qty;
    return acc;
  },
  { total: 0, itemCount: 0 }
);

// Result: { total: 58010, itemCount: 4 }
```

---

### 2.4 `forEach()` — Side Effects Only

**Syntax:**
```js
array.forEach((item, index) => {
  // side effect only — no return value used
});
```

**When to use:**
- Logging, sending emails, writing to DB, mutating an **external** variable
- When the return value is intentionally ignored

**When NOT to use:**
- When you need a transformed array → use `map()`
- When building logic that produces a result → use `reduce()`
- **Never use it as a replacement for `map()`** (common mistake)

```js
// ✅ Correct use — side effect (logging audit trail)
const failedPayments = [];
payments.forEach((p) => {
  if (p.status === "failed") {
    failedPayments.push(p.id);
    logger.error(`Payment failed: ${p.id}`);
    alertService.notify(p.userId);
  }
});

// ❌ Wrong use — should be map()
const names = [];
users.forEach((u) => names.push(u.name)); // Use map() instead
```

---

### 2.5 `find()` — First Match Lookup

**Syntax:**
```js
const item = array.find((item) => booleanCondition);
// Returns the item itself (or undefined), stops at first match
```

**When to use:**
- When you need **one specific item** from an array
- Config lookups, user authentication, settings fetch

**When NOT to use:**
- When you need ALL matching items → use `filter()`
- When you need the index → use `findIndex()`

```js
// Backend: Find active config for a feature flag
const featureFlags = [
  { name: "dark_mode",   enabled: true,  rollout: 100 },
  { name: "new_checkout", enabled: false, rollout: 0 },
  { name: "ai_suggest",  enabled: true,  rollout: 50 },
];

const darkModeFlag = featureFlags.find((f) => f.name === "dark_mode");
// { name: "dark_mode", enabled: true, rollout: 100 }

// Frontend: Find selected product in cart
const selectedProduct = products.find((p) => p.id === selectedId);
```

---

### 2.6 `some()` — Existence Check

**Syntax:**
```js
const exists = array.some((item) => booleanCondition);
// Returns true if AT LEAST ONE item matches
```

**When to use:**
- Permission checks, validation, presence detection
- When you only need to know "does at least one match?"

```js
// Backend: Check if user has admin role
const hasAdminAccess = user.roles.some((role) => role === "admin");

// Frontend: Disable checkout if any item is out of stock
const hasUnavailableItem = cartItems.some((item) => item.stock === 0);
if (hasUnavailableItem) disableCheckout();

// Validation: Check for duplicate email
const isDuplicate = existingUsers.some((u) => u.email === newEmail);
```

---

### 2.7 `every()` — Universal Condition Check

**Syntax:**
```js
const allMatch = array.every((item) => booleanCondition);
// Returns true only if ALL items match
```

**When to use:**
- Form validation (all fields filled?), batch operations (all items valid?)
- Ensuring data integrity before processing

```js
// Backend: Validate all required fields before DB insert
const requiredFields = ["name", "email", "phone"];
const isComplete = requiredFields.every((field) => Boolean(formData[field]?.trim()));

// Frontend: Enable submit button only when all inputs are valid
const allValid = formFields.every((field) => field.isValid);
submitBtn.disabled = !allValid;

// Data pipeline: Ensure all records have IDs before bulk insert
const safeToInsert = records.every((r) => r.id && r.createdAt);
```

---

## 3. Advanced Patterns

### 3.1 Flattening Nested Data

```js
// Input: Nested categories with products
const catalog = [
  { category: "Electronics", products: ["Laptop", "Phone"] },
  { category: "Books",       products: ["Novel", "Textbook", "Comic"] },
];

// Flat list of all products
const allProducts = catalog.flatMap((cat) => cat.products);
// ["Laptop", "Phone", "Novel", "Textbook", "Comic"]

// With context preserved
const productsWithCategory = catalog.flatMap((cat) =>
  cat.products.map((product) => ({ product, category: cat.category }))
);
// [{ product: "Laptop", category: "Electronics" }, ...]

// Deep flatten (arbitrary nesting)
const deepNested = [1, [2, [3, [4]]]];
const flat = deepNested.flat(Infinity); // [1, 2, 3, 4]
```

---

### 3.2 Converting Array → Object (Indexing)

```js
// Use case: Fast lookup map from array
const users = [
  { id: "u1", name: "Anika", tier: "gold" },
  { id: "u2", name: "Dev",   tier: "silver" },
];

// Pattern 1: Object.fromEntries (clean and modern)
const userMap = Object.fromEntries(users.map((u) => [u.id, u]));
// { u1: { id: "u1", name: "Anika", ... }, u2: { ... } }

// Pattern 2: reduce (more control)
const userIndex = users.reduce((acc, u) => ({ ...acc, [u.id]: u }), {});

// Pattern 3: Index by multiple keys
const byTier = users.reduce((acc, u) => {
  if (!acc[u.tier]) acc[u.tier] = [];
  acc[u.tier].push(u);
  return acc;
}, {});
// { gold: [{ id: "u1", ... }], silver: [{ id: "u2", ... }] }
```

---

### 3.3 Dynamic Key Creation

```js
// Use case: Survey answers where question names are keys
const answers = [
  { question: "Preferred language?", answer: "Hindi" },
  { question: "Age range?",          answer: "25-34" },
  { question: "City?",               answer: "Pune" },
];

// Create object with dynamic keys
const answerMap = answers.reduce((acc, { question, answer }) => {
  acc[question] = answer;
  return acc;
}, {});
// { "Preferred language?": "Hindi", "Age range?": "25-34", "City?": "Pune" }

// Use case: Build analytics dimensions dynamically
function buildDimensions(events, dimensions) {
  return events.map((event) =>
    dimensions.reduce((acc, dim) => {
      acc[dim] = event[dim] ?? null;
      return acc;
    }, {})
  );
}
```

---

### 3.4 Grouping Data (SQL GROUP BY equivalent)

```js
const orders = [
  { id: "o1", region: "North", status: "completed", amount: 1200 },
  { id: "o2", region: "South", status: "pending",   amount: 800 },
  { id: "o3", region: "North", status: "completed", amount: 3400 },
  { id: "o4", region: "South", status: "completed", amount: 500 },
  { id: "o5", region: "West",  status: "refunded",  amount: 900 },
];

// GROUP BY single key
function groupBy(array, key) {
  return array.reduce((acc, item) => {
    const group = item[key];
    if (!acc[group]) acc[group] = [];
    acc[group].push(item);
    return acc;
  }, {});
}

const byRegion = groupBy(orders, "region");
// { North: [...], South: [...], West: [...] }

// GROUP BY with aggregation (like SQL GROUP BY + SUM)
function groupAndAggregate(array, groupKey, sumKey) {
  return array.reduce((acc, item) => {
    const group = item[groupKey];
    if (!acc[group]) acc[group] = { count: 0, total: 0, items: [] };
    acc[group].count++;
    acc[group].total += item[sumKey];
    acc[group].items.push(item);
    return acc;
  }, {});
}

const regionSummary = groupAndAggregate(orders, "region", "amount");
// { North: { count: 2, total: 4600, items: [...] }, ... }
```

---

### 3.5 Merging Datasets (JOIN equivalent)

```js
const users = [
  { userId: "u1", name: "Kavya", email: "kavya@example.com" },
  { userId: "u2", name: "Rahul", email: "rahul@example.com" },
];

const orders = [
  { orderId: "o1", userId: "u1", amount: 2500 },
  { orderId: "o2", userId: "u1", amount: 750 },
  { orderId: "o3", userId: "u2", amount: 1800 },
];

// INNER JOIN — Only users with orders
const userMap = Object.fromEntries(users.map((u) => [u.userId, u]));

const enrichedOrders = orders.map((order) => ({
  ...order,
  user: userMap[order.userId] ?? null,
}));

// LEFT JOIN — All users, with their orders (even if none)
const usersWithOrders = users.map((user) => ({
  ...user,
  orders: orders.filter((o) => o.userId === user.userId),
  totalSpent: orders
    .filter((o) => o.userId === user.userId)
    .reduce((sum, o) => sum + o.amount, 0),
}));
```

---

### 3.6 Handling Optional/Null Values Safely

```js
// Optional chaining + nullish coalescing
const city = user?.address?.city ?? "Unknown";
const score = analytics?.engagement?.clickRate ?? 0;

// Safe array access
const firstTag = item?.tags?.[0] ?? "untagged";

// Safe transformation with defaults
const safeTransform = (users) =>
  (users ?? []).map((user) => ({
    id: user?.id ?? "N/A",
    name: user?.name?.trim() ?? "Anonymous",
    age: typeof user?.age === "number" ? user.age : null,
    tags: Array.isArray(user?.tags) ? user.tags : [],
  }));

// Filter nulls before processing
const validUsers = rawData.filter(Boolean); // removes null, undefined, 0, ""
const withIds = records.filter((r) => r?.id != null);

// Null-safe number parsing
const parseAmount = (value) => {
  const parsed = parseFloat(value);
  return isNaN(parsed) ? 0 : parsed;
};
```

---

## 4. Real-World Scenarios

### 4.1 Survey Session Data → Excel Rows

This is one of the most common real-world transformations: nested, dynamic data into flat rows.

```js
// INPUT: Complex nested survey sessions
const sessions = [
  {
    sessionId: "sess_001",
    userId: "u_101",
    surveyName: "Customer Feedback Q1",
    startedAt: "2024-03-01T10:00:00Z",
    completedAt: "2024-03-01T10:08:00Z",
    answers: [
      { questionId: "q1", questionText: "How satisfied are you?", answer: "Very Satisfied", type: "scale" },
      { questionId: "q2", questionText: "Which city?", answer: "Bangalore", type: "text" },
      { questionId: "q3", questionText: "Age group?", answer: "25-34", type: "choice" },
    ],
  },
  {
    sessionId: "sess_002",
    userId: "u_102",
    surveyName: "Customer Feedback Q1",
    startedAt: "2024-03-01T11:00:00Z",
    completedAt: "2024-03-01T11:05:00Z",
    answers: [
      { questionId: "q1", questionText: "How satisfied are you?", answer: "Satisfied", type: "scale" },
      { questionId: "q2", questionText: "Which city?", answer: "Mumbai", type: "text" },
      // q3 not answered
    ],
  },
];

// STEP 1: Collect all unique question texts (for column headers)
function getAllQuestions(sessions) {
  const questionSet = new Map();
  sessions.forEach((session) => {
    session.answers.forEach((a) => {
      if (!questionSet.has(a.questionId)) {
        questionSet.set(a.questionId, a.questionText);
      }
    });
  });
  return questionSet; // Map { q1 -> "How satisfied...", q2 -> "Which city?", ... }
}

// STEP 2: Transform each session into a flat row
function transformSessionToRow(session, allQuestions) {
  // Base fields
  const base = {
    "Session ID":   session.sessionId,
    "User ID":      session.userId,
    "Survey Name":  session.surveyName,
    "Started At":   new Date(session.startedAt).toLocaleString("en-IN"),
    "Completed At": new Date(session.completedAt).toLocaleString("en-IN"),
    "Duration (min)": Math.round(
      (new Date(session.completedAt) - new Date(session.startedAt)) / 60000
    ),
  };

  // Build answer lookup for this session
  const answerLookup = session.answers.reduce((acc, a) => {
    acc[a.questionId] = a.answer;
    return acc;
  }, {});

  // Add one column per question (empty string if not answered)
  const answerColumns = {};
  allQuestions.forEach((questionText, questionId) => {
    answerColumns[questionText] = answerLookup[questionId] ?? "";
  });

  return { ...base, ...answerColumns };
}

// STEP 3: Run the full transformation
function transformSurveysToExcelRows(sessions) {
  const allQuestions = getAllQuestions(sessions);
  return sessions.map((session) => transformSessionToRow(session, allQuestions));
}

const excelRows = transformSurveysToExcelRows(sessions);
console.log(excelRows);

/*
OUTPUT:
[
  {
    "Session ID": "sess_001",
    "User ID": "u_101",
    "Survey Name": "Customer Feedback Q1",
    "Started At": "3/1/2024, 3:30:00 PM",
    "Completed At": "3/1/2024, 3:38:00 PM",
    "Duration (min)": 8,
    "How satisfied are you?": "Very Satisfied",
    "Which city?": "Bangalore",
    "Age group?": "25-34"
  },
  {
    "Session ID": "sess_002",
    "User ID": "u_102",
    ...
    "How satisfied are you?": "Satisfied",
    "Which city?": "Mumbai",
    "Age group?": ""          <-- graceful empty for missing answers
  }
]
*/
```

---

### 4.2 API Response → UI Table Format

```js
// Raw API response from a backend
const apiResponse = {
  status: "success",
  meta: { total: 3, page: 1, perPage: 10 },
  data: {
    users: [
      {
        user_id: "usr_001",
        profile: { first_name: "Sneha", last_name: "Joshi", avatar_url: null },
        account: { plan: "pro", is_active: true, created_at: "2023-11-05T00:00:00Z" },
        stats: { orders: 12, total_spent: 24500, last_login: "2024-03-18T09:22:00Z" },
      },
      {
        user_id: "usr_002",
        profile: { first_name: "Vikram", last_name: "Nair", avatar_url: "https://cdn.example.com/v.jpg" },
        account: { plan: "free", is_active: false, created_at: "2024-01-12T00:00:00Z" },
        stats: { orders: 2, total_spent: 1800, last_login: "2024-02-28T14:00:00Z" },
      },
    ],
  },
};

// Transform to flat UI table rows
function transformToTableRows(apiResponse) {
  return apiResponse.data.users.map((user) => ({
    id:         user.user_id,
    name:       `${user.profile.first_name} ${user.profile.last_name}`,
    avatar:     user.profile.avatar_url ?? "/default-avatar.png",
    plan:       user.account.plan.toUpperCase(),
    status:     user.account.is_active ? "Active" : "Inactive",
    statusBadge: user.account.is_active ? "green" : "red",
    joinDate:   new Date(user.account.created_at).toLocaleDateString("en-IN"),
    orders:     user.stats.orders,
    spent:      `₹${user.stats.total_spent.toLocaleString("en-IN")}`,
    lastSeen:   formatRelativeTime(user.stats.last_login),
  }));
}

function formatRelativeTime(isoString) {
  const diff = Date.now() - new Date(isoString).getTime();
  const days = Math.floor(diff / 86400000);
  if (days === 0) return "Today";
  if (days === 1) return "Yesterday";
  return `${days} days ago`;
}

const tableRows = transformToTableRows(apiResponse);
```

---

### 4.3 Building Dynamic Columns from Unknown Questions

```js
// Use case: You don't know the questions in advance — they come from the DB
// This is common in survey tools, form builders, CMS platforms

async function buildDynamicTable(surveyId) {
  // Fetch from API
  const sessions = await fetchSurveySessions(surveyId);

  // Discover all unique questions dynamically
  const questionRegistry = new Map(); // questionId -> questionText

  sessions.forEach((session) => {
    session.answers.forEach((answer) => {
      if (!questionRegistry.has(answer.questionId)) {
        questionRegistry.set(answer.questionId, answer.questionText);
      }
    });
  });

  // Build column definitions for your table component
  const staticColumns = [
    { key: "userId",      header: "User ID",      width: 120 },
    { key: "completedAt", header: "Completed At",  width: 150 },
  ];

  const dynamicColumns = Array.from(questionRegistry.entries()).map(
    ([qId, qText]) => ({
      key:    qId,
      header: qText,
      width:  200,
      render: (value) => value || "—",
    })
  );

  const columns = [...staticColumns, ...dynamicColumns];

  // Build rows
  const rows = sessions.map((session) => {
    const row = {
      userId:      session.userId,
      completedAt: new Date(session.completedAt).toLocaleDateString(),
    };

    session.answers.forEach((a) => {
      row[a.questionId] = a.answer;
    });

    return row;
  });

  return { columns, rows };
}
```

---

### 4.4 Preparing Data for Charts/Graphs

```js
const salesData = [
  { date: "2024-01-05", product: "Laptop",   region: "North", revenue: 55000 },
  { date: "2024-01-05", product: "Phone",    region: "South", revenue: 32000 },
  { date: "2024-01-12", product: "Laptop",   region: "South", revenue: 48000 },
  { date: "2024-01-12", product: "Tablet",   region: "North", revenue: 21000 },
  { date: "2024-01-19", product: "Laptop",   region: "North", revenue: 61000 },
  { date: "2024-01-19", product: "Phone",    region: "North", revenue: 29000 },
];

// Transform for a LINE CHART (revenue over time)
function toLineChartData(sales) {
  const grouped = groupBy(sales, "date");

  return Object.entries(grouped).map(([date, items]) => ({
    x: date,
    y: items.reduce((sum, item) => sum + item.revenue, 0),
    label: `₹${items.reduce((sum, i) => sum + i.revenue, 0).toLocaleString()}`,
  }));
}

// Transform for a BAR CHART (revenue by product)
function toBarChartData(sales) {
  const grouped = groupBy(sales, "product");

  return Object.entries(grouped).map(([product, items]) => ({
    category: product,
    value:    items.reduce((sum, i) => sum + i.revenue, 0),
  }));
}

// Transform for a PIE CHART (share by region)
function toPieChartData(sales) {
  const total  = sales.reduce((sum, s) => sum + s.revenue, 0);
  const byRegion = groupBy(sales, "region");

  return Object.entries(byRegion).map(([region, items]) => {
    const regionTotal = items.reduce((sum, i) => sum + i.revenue, 0);
    return {
      name:       region,
      value:      regionTotal,
      percentage: ((regionTotal / total) * 100).toFixed(1),
    };
  });
}

// Helper (reused from section 3.4)
function groupBy(array, key) {
  return array.reduce((acc, item) => {
    const g = item[key];
    if (!acc[g]) acc[g] = [];
    acc[g].push(item);
    return acc;
  }, {});
}
```

---

## 5. Excel & Reporting Use Cases

### 5.1 Preparing Data for ExcelJS

```js
const ExcelJS = require("exceljs");

async function generateExcelReport(sessions) {
  const workbook  = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet("Survey Results");

  // STEP 1: Transform your data into flat rows
  const rows       = transformSurveysToExcelRows(sessions); // from section 4.1
  const columnKeys = Object.keys(rows[0] ?? {});

  // STEP 2: Set column definitions
  worksheet.columns = columnKeys.map((key) => ({
    header: key,
    key,
    width: key.length > 20 ? 35 : 20,
  }));

  // STEP 3: Style the header row
  worksheet.getRow(1).eachCell((cell) => {
    cell.font      = { bold: true, color: { argb: "FFFFFFFF" } };
    cell.fill      = { type: "pattern", pattern: "solid", fgColor: { argb: "FF2E75B6" } };
    cell.alignment = { horizontal: "center" };
  });

  // STEP 4: Add data rows
  rows.forEach((row) => worksheet.addRow(row));

  // STEP 5: Auto-filter for all columns
  worksheet.autoFilter = {
    from: { row: 1, column: 1 },
    to:   { row: 1, column: columnKeys.length },
  };

  await workbook.xlsx.writeFile("survey_report.xlsx");
}
```

---

### 5.2 Row-Based vs Column-Based Strategies

#### Row-Based (One row per session — most common)

```js
// When: You want to filter, sort, or analyze by session
// Output: Each row = one survey completion

const rowBased = sessions.map((session) => ({
  "User ID":     session.userId,
  "Q1: Satisfaction": session.answers.find((a) => a.questionId === "q1")?.answer ?? "",
  "Q2: City":         session.answers.find((a) => a.questionId === "q2")?.answer ?? "",
}));

/*
| User ID | Q1: Satisfaction | Q2: City  |
|---------|-----------------|-----------|
| u1      | Very Satisfied  | Bangalore |
| u2      | Satisfied       | Mumbai    |
*/
```

#### Column-Based (One column per session — for comparison)

```js
// When: You want to compare answers side-by-side across sessions
// Output: Each row = one question, each column = one user

function toColumnBased(sessions, questions) {
  return questions.map((q) => {
    const row = { Question: q.questionText };
    sessions.forEach((session) => {
      const answer = session.answers.find((a) => a.questionId === q.questionId);
      row[session.userId] = answer?.answer ?? "";
    });
    return row;
  });
}

/*
| Question          | u1              | u2        |
|-------------------|-----------------|-----------|
| Satisfaction?     | Very Satisfied  | Satisfied |
| City?             | Bangalore       | Mumbai    |
*/
```

---

### 5.3 One Row Per Answer (Unpivoted format)

```js
// When: You need raw data for further analysis (pivot tables, BI tools)
function toOneRowPerAnswer(sessions) {
  return sessions.flatMap((session) =>
    session.answers.map((answer) => ({
      sessionId:    session.sessionId,
      userId:       session.userId,
      completedAt:  session.completedAt,
      questionId:   answer.questionId,
      questionText: answer.questionText,
      answer:       answer.answer,
      answerType:   answer.type,
    }))
  );
}

/*
| sessionId | userId | questionId | questionText       | answer         |
|-----------|--------|------------|--------------------|----------------|
| sess_001  | u_101  | q1         | How satisfied...   | Very Satisfied |
| sess_001  | u_101  | q2         | Which city?        | Bangalore      |
| sess_002  | u_102  | q1         | How satisfied...   | Satisfied      |
*/
```

---

## 6. Common Mistakes

### 6.1 Overusing `forEach` Instead of `map`

```js
// ❌ Wrong — using forEach to build array
const names = [];
users.forEach((u) => names.push(u.name));

// ✅ Correct — use map
const names = users.map((u) => u.name);

// ❌ Wrong — forEach can't be chained
users.forEach((u) => u.name).filter(Boolean); // TypeError!

// ✅ Correct — map chains properly
users.map((u) => u.name).filter(Boolean);
```

---

### 6.2 Misusing `reduce`

```js
// ❌ Wrong — reduce with no initial value on empty array crashes
[].reduce((acc, x) => acc + x); // TypeError: Reduce of empty array with no initial value

// ✅ Always provide an initial value
[].reduce((acc, x) => acc + x, 0); // 0

// ❌ Wrong — reduce mutating the accumulator object directly
const result = users.reduce((acc, u) => {
  acc[u.id] = u; // Mutating the accumulator — works but dangerous in complex cases
  return acc;
}, {});

// ✅ Better — spread (immutable pattern)
const result = users.reduce((acc, u) => ({ ...acc, [u.id]: u }), {});
// Note: spread has O(n²) cost for large arrays — direct mutation is fine for performance-sensitive paths
```

---

### 6.3 Mutating Objects Incorrectly

```js
// ❌ Wrong — mutating original data
const processed = orders.map((order) => {
  order.total = order.price * order.qty; // mutates original!
  return order;
});

// ✅ Correct — create new object
const processed = orders.map((order) => ({
  ...order,
  total: order.price * order.qty,
}));

// ❌ Wrong — mutating nested objects with spread
const user   = { id: 1, address: { city: "Delhi" } };
const updated = { ...user, address: { ...user.address, city: "Mumbai" } }; // ✅ correct deep copy
const wrong  = { ...user };
wrong.address.city = "Mumbai"; // ❌ mutates original user.address!
```

---

### 6.4 Not Handling `undefined`/`null`

```js
// ❌ Crashes if user or address is null
const city = user.address.city;

// ✅ Safe with optional chaining
const city = user?.address?.city ?? "Unknown";

// ❌ Crashes if answers is undefined
session.answers.map((a) => a.answer);

// ✅ Defensive
(session.answers ?? []).map((a) => a.answer);

// ❌ Wrong type check
if (value == null) { /* misses undefined in some contexts */ }

// ✅ Catches both null and undefined
if (value == null) { /* this actually catches both! == null catches null and undefined */ }
if (value === null || value === undefined) { /* explicit version */ }
```

---

## 7. Performance & Clean Code

### 7.1 Avoid Nested Loops When Possible

```js
// ❌ O(n²) — nested find inside map
const enriched = orders.map((order) => ({
  ...order,
  user: users.find((u) => u.id === order.userId), // O(n) for each order
}));

// ✅ O(n) — build lookup map first
const userMap   = Object.fromEntries(users.map((u) => [u.id, u]));
const enriched  = orders.map((order) => ({
  ...order,
  user: userMap[order.userId] ?? null,
}));
```

---

### 7.2 Writing Readable Transformations

```js
// ❌ One giant reduce that does everything
const result = sessions.reduce((acc, s) => {
  const base = { id: s.sessionId, userId: s.userId };
  const answers = s.answers.reduce((a, ans) => { a[ans.questionText] = ans.answer; return a; }, {});
  acc.push({ ...base, ...answers });
  return acc;
}, []);

// ✅ Decompose into named helper functions
function extractBase(session) {
  return { id: session.sessionId, userId: session.userId };
}

function extractAnswers(answers) {
  return answers.reduce((acc, a) => ({ ...acc, [a.questionText]: a.answer }), {});
}

function transformSession(session) {
  return { ...extractBase(session), ...extractAnswers(session.answers) };
}

const result = sessions.map(transformSession);
```

---

### 7.3 Functional vs Imperative Approach

```js
// IMPERATIVE — step-by-step with mutations
function getTopSpenders(orders, limit) {
  const spendMap = {};
  for (const order of orders) {
    if (!spendMap[order.userId]) spendMap[order.userId] = 0;
    spendMap[order.userId] += order.amount;
  }
  const sorted = Object.entries(spendMap).sort((a, b) => b[1] - a[1]);
  return sorted.slice(0, limit).map(([userId, total]) => ({ userId, total }));
}

// FUNCTIONAL — pipeline of transformations (more readable, composable)
function getTopSpenders(orders, limit) {
  return Object.entries(
    orders.reduce((acc, { userId, amount }) => ({
      ...acc,
      [userId]: (acc[userId] ?? 0) + amount,
    }), {})
  )
    .map(([userId, total]) => ({ userId, total }))
    .sort((a, b) => b.total - a.total)
    .slice(0, limit);
}
```

---

### 7.4 Batch Large Datasets

```js
// For very large arrays, process in chunks to avoid blocking the event loop
async function processInBatches(items, batchSize, processFn) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch  = items.slice(i, i + batchSize);
    const result = await processFn(batch);
    results.push(...result);

    // Yield to event loop between batches
    await new Promise((resolve) => setImmediate(resolve));
  }

  return results;
}

// Usage
const excelRows = await processInBatches(largeSessions, 500, transformSurveysToExcelRows);
```

---

## 8. Practice Problems

### Problem 1 — Normalize a Raw API Response (Beginner)

**Input:**
```js
const rawUsers = [
  { user_id: 1, user_name: "priya_k", email_address: "priya@test.com", account_status: "active" },
  { user_id: 2, user_name: "rahul_m", email_address: "rahul@test.com", account_status: "suspended" },
];
```

**Task:** Transform to `{ id, username, email, isActive }` format.

**Solution:**
```js
const users = rawUsers.map(({ user_id, user_name, email_address, account_status }) => ({
  id:       user_id,
  username: user_name,
  email:    email_address,
  isActive: account_status === "active",
}));
```

---

### Problem 2 — Calculate Cart Total (Beginner)

**Input:**
```js
const cart = [
  { name: "Book",   price: 350,  qty: 2 },
  { name: "Pen",    price: 25,   qty: 5 },
  { name: "Bag",    price: 1200, qty: 1 },
];
```

**Task:** Return `{ subtotal, tax (18%), total, itemCount }`.

**Solution:**
```js
const summary = cart.reduce(
  (acc, item) => {
    const lineTotal = item.price * item.qty;
    return {
      subtotal:  acc.subtotal + lineTotal,
      itemCount: acc.itemCount + item.qty,
    };
  },
  { subtotal: 0, itemCount: 0 }
);

const tax   = summary.subtotal * 0.18;
const total = summary.subtotal + tax;
// { subtotal: 2825, itemCount: 8, tax: 508.5, total: 3333.5 }
```

---

### Problem 3 — Group Orders by Status (Intermediate)

**Input:**
```js
const orders = [
  { id: "o1", status: "completed", amount: 1500 },
  { id: "o2", status: "pending",   amount: 800 },
  { id: "o3", status: "completed", amount: 2200 },
  { id: "o4", status: "refunded",  amount: 500 },
  { id: "o5", status: "pending",   amount: 1100 },
];
```

**Task:** Return `{ completed: { count, total }, pending: { count, total }, refunded: { count, total } }`.

**Solution:**
```js
const grouped = orders.reduce((acc, order) => {
  const g = acc[order.status] ?? { count: 0, total: 0 };
  acc[order.status] = { count: g.count + 1, total: g.total + order.amount };
  return acc;
}, {});
// { completed: { count: 2, total: 3700 }, pending: { count: 2, total: 1900 }, refunded: { count: 1, total: 500 } }
```

---

### Problem 4 — Flatten Nested Comments (Intermediate)

**Input:**
```js
const posts = [
  {
    postId: "p1",
    title: "Intro to JS",
    comments: [
      { id: "c1", text: "Great post!", author: "Anika" },
      { id: "c2", text: "Very helpful", author: "Dev" },
    ],
  },
  {
    postId: "p2",
    title: "Advanced Promises",
    comments: [
      { id: "c3", text: "Mind blown!", author: "Riya" },
    ],
  },
];
```

**Task:** Flat array of comments with their post title included.

**Solution:**
```js
const flatComments = posts.flatMap((post) =>
  post.comments.map((comment) => ({
    ...comment,
    postId:    post.postId,
    postTitle: post.title,
  }))
);
```

---

### Problem 5 — Dynamic Survey to Excel Rows (Advanced)

**Task:** Already covered in full in Section 4.1. Extended challenge: add a `"Completion Rate"` column showing what percentage of questions each user answered.

**Solution:**
```js
function addCompletionRate(sessions, totalQuestions) {
  return sessions.map((session) => ({
    ...session,
    completionRate: `${Math.round((session.answers.length / totalQuestions) * 100)}%`,
  }));
}
```

---

### Problem 6 — Merge Two Datasets and Compute Metrics (Advanced)

**Input:**
```js
const customers = [
  { customerId: "c1", name: "Meera", tier: "gold" },
  { customerId: "c2", name: "Sanjay", tier: "silver" },
  { customerId: "c3", name: "Tanvi", tier: "bronze" },
];

const transactions = [
  { txId: "t1", customerId: "c1", amount: 5000, date: "2024-03" },
  { txId: "t2", customerId: "c1", amount: 3200, date: "2024-03" },
  { txId: "t3", customerId: "c2", amount: 1800, date: "2024-03" },
  { txId: "t4", customerId: "c1", amount: 7000, date: "2024-04" },
];
```

**Task:** Produce `{ customerId, name, tier, txCount, totalSpent, avgOrderValue }`.

**Solution:**
```js
// Build transaction summary per customer
const txSummary = transactions.reduce((acc, tx) => {
  const s = acc[tx.customerId] ?? { txCount: 0, totalSpent: 0 };
  acc[tx.customerId] = {
    txCount:    s.txCount + 1,
    totalSpent: s.totalSpent + tx.amount,
  };
  return acc;
}, {});

// Merge with customer info
const report = customers.map((customer) => {
  const stats = txSummary[customer.customerId] ?? { txCount: 0, totalSpent: 0 };
  return {
    customerId:    customer.customerId,
    name:          customer.name,
    tier:          customer.tier,
    txCount:       stats.txCount,
    totalSpent:    stats.totalSpent,
    avgOrderValue: stats.txCount > 0 ? Math.round(stats.totalSpent / stats.txCount) : 0,
  };
});

/*
[
  { customerId: "c1", name: "Meera",  tier: "gold",   txCount: 3, totalSpent: 15200, avgOrderValue: 5067 },
  { customerId: "c2", name: "Sanjay", tier: "silver",  txCount: 1, totalSpent: 1800,  avgOrderValue: 1800 },
  { customerId: "c3", name: "Tanvi",  tier: "bronze",  txCount: 0, totalSpent: 0,     avgOrderValue: 0    }
]
*/
```

---

### Problem 7 — Prepare Chart-Ready Time Series (Advanced)

**Task:** From the `transactions` above, produce monthly revenue data for a line chart.

```js
const monthlyRevenue = Object.entries(
  transactions.reduce((acc, tx) => ({
    ...acc,
    [tx.date]: (acc[tx.date] ?? 0) + tx.amount,
  }), {})
)
  .sort(([a], [b]) => a.localeCompare(b))
  .map(([month, revenue]) => ({ month, revenue }));

// [{ month: "2024-03", revenue: 10000 }, { month: "2024-04", revenue: 7000 }]
```

---

## Quick Reference Cheat Sheet

```
┌──────────────┬──────────────────────────────────────────┬────────────────────────────┐
│ Function     │ Use When                                 │ Returns                    │
├──────────────┼──────────────────────────────────────────┼────────────────────────────┤
│ map()        │ Transform each element                   │ New array (same length)    │
│ filter()     │ Keep matching elements                   │ New array (≤ length)       │
│ reduce()     │ Fold array into any single value         │ Anything you return        │
│ forEach()    │ Side effects only                        │ undefined (not chainable)  │
│ find()       │ Get first matching element               │ Element or undefined       │
│ findIndex()  │ Get index of first match                 │ Number (-1 if not found)   │
│ some()       │ Check if any element matches             │ Boolean                    │
│ every()      │ Check if all elements match              │ Boolean                    │
│ flat()       │ Flatten nested arrays                    │ New flat array             │
│ flatMap()    │ map() then flat(1)                       │ New flat array             │
└──────────────┴──────────────────────────────────────────┴────────────────────────────┘
```

---

## Transformation Decision Tree

```
You have an array. What do you need?
│
├── Same length, different shape?        → map()
├── Fewer elements?                      → filter()
├── Both filter AND transform?           → filter().map()
├── Single value / object / count?       → reduce()
├── Just side effects (log, write, etc)? → forEach()
├── One specific item?                   → find()
├── Does any item match?                 → some()
├── Do ALL items match?                  → every()
├── Nested arrays to flatten?            → flat() or flatMap()
└── Array → lookup object?              → reduce() or Object.fromEntries(map())
```

---

*This guide covers the core patterns you'll encounter in 95% of real-world data transformation tasks. The remaining 5% are variations of these same fundamentals — once you master the mental model, you can tackle any shape of data.*
