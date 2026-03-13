# 📦 Expo SQLite — Complete Setup & Usage Guide

A real-world, production-ready reference for using `expo-sqlite` in React Native Expo apps.

---

## 📋 Table of Contents

1. [Installation](#installation)
2. [Project Structure](#project-structure)
3. [Database Setup](#database-setup)
4. [All SQL Queries with Examples](#all-sql-queries-with-examples)
   - [CREATE TABLE](#create-table)
   - [INSERT](#insert)
   - [SELECT](#select)
   - [UPDATE](#update)
   - [DELETE](#delete)
   - [Transactions](#transactions)
   - [DROP TABLE](#drop-table)
5. [Real-World Example — Notes App](#real-world-example--notes-app)
6. [Database Helper Class](#database-helper-class)
7. [Common Patterns](#common-patterns)
8. [Tips & Best Practices](#tips--best-practices)

---

## Installation

```bash
npx expo install expo-sqlite
```

> ✅ Works with **Expo SDK 50+**. Uses the new **synchronous API** (`expo-sqlite` v14+).

---

## Project Structure

```
my-app/
├── app/
│   ├── index.jsx          # Main screen
│   └── _layout.jsx
├── db/
│   ├── database.js        # DB connection (open once)
│   ├── schema.js          # Table definitions
│   └── queries/
│       ├── users.js       # User queries
│       ├── notes.js       # Notes queries
│       └── categories.js  # Category queries
└── package.json
```

---

## Database Setup

### `db/database.js` — Open once, reuse everywhere

```javascript
import * as SQLite from 'expo-sqlite';

// Open the database ONCE outside any component
const db = SQLite.openDatabaseSync('myapp.db');

export default db;
```

### `db/schema.js` — Create all tables on app start

```javascript
import db from './database';

export function initDatabase() {
  db.execSync(`
    PRAGMA journal_mode = WAL;
    PRAGMA foreign_keys = ON;

    CREATE TABLE IF NOT EXISTS users (
      id        INTEGER PRIMARY KEY AUTOINCREMENT,
      name      TEXT NOT NULL,
      email     TEXT UNIQUE NOT NULL,
      avatar    TEXT,
      created_at TEXT DEFAULT (datetime('now'))
    );

    CREATE TABLE IF NOT EXISTS categories (
      id    INTEGER PRIMARY KEY AUTOINCREMENT,
      name  TEXT NOT NULL UNIQUE,
      color TEXT DEFAULT '#3B82F6'
    );

    CREATE TABLE IF NOT EXISTS notes (
      id          INTEGER PRIMARY KEY AUTOINCREMENT,
      user_id     INTEGER NOT NULL,
      category_id INTEGER,
      title       TEXT NOT NULL,
      body        TEXT,
      is_pinned   INTEGER DEFAULT 0,
      created_at  TEXT DEFAULT (datetime('now')),
      updated_at  TEXT DEFAULT (datetime('now')),
      FOREIGN KEY (user_id)     REFERENCES users(id)      ON DELETE CASCADE,
      FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL
    );
  `);
}
```

### `app/_layout.jsx` — Call `initDatabase()` once at startup

```javascript
import { useEffect } from 'react';
import { Stack } from 'expo-router';
import { initDatabase } from '../db/schema';

export default function RootLayout() {
  useEffect(() => {
    initDatabase();
  }, []);

  return <Stack />;
}
```

---

## All SQL Queries with Examples

### CREATE TABLE

```javascript
// Basic table
db.execSync(`
  CREATE TABLE IF NOT EXISTS products (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        TEXT NOT NULL,
    price       REAL NOT NULL DEFAULT 0.0,
    stock       INTEGER DEFAULT 0,
    is_active   INTEGER DEFAULT 1,        -- 0 = false, 1 = true
    created_at  TEXT DEFAULT (datetime('now'))
  );
`);

// Table with composite unique constraint
db.execSync(`
  CREATE TABLE IF NOT EXISTS user_tags (
    user_id INTEGER NOT NULL,
    tag     TEXT NOT NULL,
    UNIQUE(user_id, tag),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
  );
`);

// Add an index for faster lookups
db.execSync(`
  CREATE INDEX IF NOT EXISTS idx_notes_user_id ON notes(user_id);
  CREATE INDEX IF NOT EXISTS idx_notes_created_at ON notes(created_at DESC);
`);
```

---

### INSERT

```javascript
// Insert a single row
function createUser(name, email) {
  const result = db.runSync(
    'INSERT INTO users (name, email) VALUES (?, ?);',
    [name, email]
  );
  return result.lastInsertRowId; // returns new row ID
}

// Insert with all fields
function createNote(userId, title, body, categoryId = null) {
  const result = db.runSync(
    `INSERT INTO notes (user_id, title, body, category_id)
     VALUES (?, ?, ?, ?);`,
    [userId, title, body, categoryId]
  );
  return result.lastInsertRowId;
}

// Insert or ignore duplicates (UPSERT pattern)
function addTag(userId, tag) {
  db.runSync(
    'INSERT OR IGNORE INTO user_tags (user_id, tag) VALUES (?, ?);',
    [userId, tag]
  );
}

// Insert or replace (overwrite on conflict)
function upsertCategory(name, color) {
  db.runSync(
    'INSERT OR REPLACE INTO categories (name, color) VALUES (?, ?);',
    [name, color]
  );
}
```

---

### SELECT

```javascript
// Get ALL rows
function getAllUsers() {
  return db.getAllSync('SELECT * FROM users ORDER BY created_at DESC;');
}

// Get ONE row by ID
function getUserById(id) {
  return db.getFirstSync('SELECT * FROM users WHERE id = ?;', [id]);
}

// Search / filter
function searchNotes(keyword) {
  return db.getAllSync(
    `SELECT * FROM notes
     WHERE title LIKE ? OR body LIKE ?
     ORDER BY updated_at DESC;`,
    [`%${keyword}%`, `%${keyword}%`]
  );
}

// Filter by column value
function getPinnedNotes(userId) {
  return db.getAllSync(
    `SELECT * FROM notes
     WHERE user_id = ? AND is_pinned = 1
     ORDER BY updated_at DESC;`,
    [userId]
  );
}

// SELECT with LIMIT and OFFSET (pagination)
function getNotesPaginated(userId, page = 1, pageSize = 10) {
  const offset = (page - 1) * pageSize;
  return db.getAllSync(
    `SELECT * FROM notes
     WHERE user_id = ?
     ORDER BY created_at DESC
     LIMIT ? OFFSET ?;`,
    [userId, pageSize, offset]
  );
}

// JOIN — notes with category name and user name
function getNotesWithDetails(userId) {
  return db.getAllSync(
    `SELECT
       n.id,
       n.title,
       n.body,
       n.is_pinned,
       n.created_at,
       u.name  AS author_name,
       c.name  AS category_name,
       c.color AS category_color
     FROM notes n
     JOIN users      u ON n.user_id     = u.id
     LEFT JOIN categories c ON n.category_id = c.id
     WHERE n.user_id = ?
     ORDER BY n.is_pinned DESC, n.updated_at DESC;`,
    [userId]
  );
}

// COUNT rows
function countNotes(userId) {
  const row = db.getFirstSync(
    'SELECT COUNT(*) AS total FROM notes WHERE user_id = ?;',
    [userId]
  );
  return row.total;
}

// GROUP BY — notes count per category
function getNoteCountByCategory(userId) {
  return db.getAllSync(
    `SELECT
       c.name,
       c.color,
       COUNT(n.id) AS note_count
     FROM categories c
     LEFT JOIN notes n ON n.category_id = c.id AND n.user_id = ?
     GROUP BY c.id
     ORDER BY note_count DESC;`,
    [userId]
  );
}

// Check if row exists
function userExists(email) {
  const row = db.getFirstSync(
    'SELECT 1 FROM users WHERE email = ? LIMIT 1;',
    [email]
  );
  return !!row;
}
```

---

### UPDATE

```javascript
// Update a single field
function pinNote(id) {
  db.runSync('UPDATE notes SET is_pinned = 1 WHERE id = ?;', [id]);
}

// Update multiple fields
function updateNote(id, title, body) {
  db.runSync(
    `UPDATE notes
     SET title = ?, body = ?, updated_at = datetime('now')
     WHERE id = ?;`,
    [title, body, id]
  );
}

// Toggle boolean (0 → 1 → 0)
function togglePin(id) {
  db.runSync(
    'UPDATE notes SET is_pinned = CASE WHEN is_pinned = 1 THEN 0 ELSE 1 END WHERE id = ?;',
    [id]
  );
}

// Bulk update
function markAllRead(userId) {
  const result = db.runSync(
    'UPDATE notes SET is_pinned = 0 WHERE user_id = ?;',
    [userId]
  );
  return result.changes; // number of rows updated
}

// Update with condition
function updateUserAvatar(userId, avatarUrl) {
  db.runSync(
    'UPDATE users SET avatar = ? WHERE id = ?;',
    [avatarUrl, userId]
  );
}
```

---

### DELETE

```javascript
// Delete by ID
function deleteNote(id) {
  db.runSync('DELETE FROM notes WHERE id = ?;', [id]);
}

// Delete all rows for a user
function deleteAllNotes(userId) {
  const result = db.runSync(
    'DELETE FROM notes WHERE user_id = ?;',
    [userId]
  );
  return result.changes; // number of deleted rows
}

// Delete with multiple conditions
function deleteOldNotes(userId, olderThan) {
  // olderThan = ISO date string e.g. '2024-01-01'
  db.runSync(
    `DELETE FROM notes
     WHERE user_id = ? AND created_at < ? AND is_pinned = 0;`,
    [userId, olderThan]
  );
}

// Delete all data from a table
function clearTable(tableName) {
  db.execSync(`DELETE FROM ${tableName};`);
}
```

---

### Transactions

Use transactions when running multiple writes together. If one fails, all are rolled back.

```javascript
// Transfer: atomic multi-step write
function createNoteWithTags(userId, title, body, tags = []) {
  db.withTransactionSync(() => {
    // Step 1: insert note
    const result = db.runSync(
      'INSERT INTO notes (user_id, title, body) VALUES (?, ?, ?);',
      [userId, title, body]
    );
    const noteId = result.lastInsertRowId;

    // Step 2: insert each tag
    for (const tag of tags) {
      db.runSync(
        'INSERT OR IGNORE INTO user_tags (user_id, tag) VALUES (?, ?);',
        [userId, tag]
      );
    }

    return noteId;
  });
}

// Bulk insert (much faster than individual inserts)
function bulkInsertNotes(userId, notes) {
  db.withTransactionSync(() => {
    for (const note of notes) {
      db.runSync(
        'INSERT INTO notes (user_id, title, body) VALUES (?, ?, ?);',
        [userId, note.title, note.body]
      );
    }
  });
}
```

---

### DROP TABLE

```javascript
// Drop a table (permanent — use carefully)
function dropTable(tableName) {
  db.execSync(`DROP TABLE IF EXISTS ${tableName};`);
}

// Reset entire database (useful for dev/logout)
function resetDatabase() {
  db.execSync(`
    DROP TABLE IF EXISTS notes;
    DROP TABLE IF EXISTS user_tags;
    DROP TABLE IF EXISTS categories;
    DROP TABLE IF EXISTS users;
  `);
  initDatabase(); // re-create tables
}
```

---

## Real-World Example — Notes App

### `db/queries/notes.js`

```javascript
import db from '../database';

export const notesDB = {
  getAll: (userId) =>
    db.getAllSync(
      `SELECT n.*, c.name AS category_name, c.color AS category_color
       FROM notes n
       LEFT JOIN categories c ON n.category_id = c.id
       WHERE n.user_id = ?
       ORDER BY n.is_pinned DESC, n.updated_at DESC;`,
      [userId]
    ),

  getById: (id) =>
    db.getFirstSync('SELECT * FROM notes WHERE id = ?;', [id]),

  create: (userId, title, body, categoryId = null) => {
    const result = db.runSync(
      'INSERT INTO notes (user_id, title, body, category_id) VALUES (?, ?, ?, ?);',
      [userId, title, body, categoryId]
    );
    return result.lastInsertRowId;
  },

  update: (id, title, body) =>
    db.runSync(
      `UPDATE notes SET title = ?, body = ?, updated_at = datetime('now') WHERE id = ?;`,
      [title, body, id]
    ),

  togglePin: (id) =>
    db.runSync(
      'UPDATE notes SET is_pinned = CASE WHEN is_pinned = 1 THEN 0 ELSE 1 END WHERE id = ?;',
      [id]
    ),

  delete: (id) =>
    db.runSync('DELETE FROM notes WHERE id = ?;', [id]),

  search: (userId, keyword) =>
    db.getAllSync(
      `SELECT * FROM notes
       WHERE user_id = ? AND (title LIKE ? OR body LIKE ?)
       ORDER BY updated_at DESC;`,
      [userId, `%${keyword}%`, `%${keyword}%`]
    ),

  count: (userId) =>
    db.getFirstSync('SELECT COUNT(*) AS total FROM notes WHERE user_id = ?;', [userId])?.total ?? 0,
};
```

### `app/index.jsx` — Notes screen using the DB

```javascript
import { useEffect, useState, useCallback } from 'react';
import {
  View, Text, TextInput, FlatList,
  TouchableOpacity, StyleSheet, Alert
} from 'react-native';
import { notesDB } from '../db/queries/notes';

const USER_ID = 1; // Replace with real auth user ID

export default function NotesScreen() {
  const [notes, setNotes]     = useState([]);
  const [title, setTitle]     = useState('');
  const [body, setBody]       = useState('');
  const [search, setSearch]   = useState('');
  const [editing, setEditing] = useState(null);

  const load = useCallback(() => {
    const data = search.trim()
      ? notesDB.search(USER_ID, search)
      : notesDB.getAll(USER_ID);
    setNotes(data);
  }, [search]);

  useEffect(() => {
    load();
  }, [load]);

  function handleSave() {
    if (!title.trim()) return Alert.alert('Title is required');

    if (editing) {
      notesDB.update(editing.id, title, body);
      setEditing(null);
    } else {
      notesDB.create(USER_ID, title, body);
    }

    setTitle('');
    setBody('');
    load();
  }

  function handleEdit(note) {
    setEditing(note);
    setTitle(note.title);
    setBody(note.body ?? '');
  }

  function handleDelete(id) {
    Alert.alert('Delete Note', 'Are you sure?', [
      { text: 'Cancel', style: 'cancel' },
      {
        text: 'Delete', style: 'destructive',
        onPress: () => { notesDB.delete(id); load(); }
      }
    ]);
  }

  function handlePin(id) {
    notesDB.togglePin(id);
    load();
  }

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>📝 My Notes ({notesDB.count(USER_ID)})</Text>

      {/* Search */}
      <TextInput
        style={styles.input}
        placeholder="Search notes..."
        value={search}
        onChangeText={setSearch}
      />

      {/* Form */}
      <TextInput
        style={styles.input}
        placeholder="Title *"
        value={title}
        onChangeText={setTitle}
      />
      <TextInput
        style={[styles.input, { height: 80 }]}
        placeholder="Body (optional)"
        value={body}
        onChangeText={setBody}
        multiline
      />
      <TouchableOpacity style={styles.btn} onPress={handleSave}>
        <Text style={styles.btnText}>{editing ? '✅ Update Note' : '➕ Add Note'}</Text>
      </TouchableOpacity>

      {/* List */}
      <FlatList
        data={notes}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={[styles.card, item.is_pinned && styles.pinned]}>
            <Text style={styles.noteTitle}>{item.is_pinned ? '📌 ' : ''}{item.title}</Text>
            {item.body ? <Text style={styles.noteBody}>{item.body}</Text> : null}
            {item.category_name ? (
              <Text style={[styles.tag, { backgroundColor: item.category_color }]}>
                {item.category_name}
              </Text>
            ) : null}
            <Text style={styles.date}>{item.updated_at}</Text>
            <View style={styles.actions}>
              <TouchableOpacity onPress={() => handlePin(item.id)}>
                <Text style={styles.action}>{item.is_pinned ? 'Unpin' : 'Pin'}</Text>
              </TouchableOpacity>
              <TouchableOpacity onPress={() => handleEdit(item)}>
                <Text style={styles.action}>Edit</Text>
              </TouchableOpacity>
              <TouchableOpacity onPress={() => handleDelete(item.id)}>
                <Text style={[styles.action, { color: '#EF4444' }]}>Delete</Text>
              </TouchableOpacity>
            </View>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.empty}>No notes yet. Add one above!</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container:  { flex: 1, padding: 16, backgroundColor: '#F9FAFB' },
  heading:    { fontSize: 22, fontWeight: 'bold', marginBottom: 12 },
  input:      { borderWidth: 1, borderColor: '#D1D5DB', borderRadius: 8, padding: 10, marginBottom: 8, backgroundColor: '#fff' },
  btn:        { backgroundColor: '#3B82F6', padding: 12, borderRadius: 8, alignItems: 'center', marginBottom: 16 },
  btnText:    { color: '#fff', fontWeight: '600' },
  card:       { backgroundColor: '#fff', padding: 14, borderRadius: 10, marginBottom: 10, shadowColor: '#000', shadowOpacity: 0.05, shadowRadius: 4, elevation: 2 },
  pinned:     { borderLeftWidth: 4, borderLeftColor: '#3B82F6' },
  noteTitle:  { fontSize: 16, fontWeight: '600', marginBottom: 4 },
  noteBody:   { fontSize: 14, color: '#6B7280', marginBottom: 6 },
  tag:        { alignSelf: 'flex-start', paddingHorizontal: 8, paddingVertical: 2, borderRadius: 12, color: '#fff', fontSize: 12, marginBottom: 4 },
  date:       { fontSize: 11, color: '#9CA3AF', marginBottom: 8 },
  actions:    { flexDirection: 'row', gap: 16 },
  action:     { color: '#3B82F6', fontWeight: '500' },
  empty:      { textAlign: 'center', color: '#9CA3AF', marginTop: 40 },
});
```

---

## Database Helper Class

For larger apps, wrap the DB in a reusable class:

```javascript
// db/DbHelper.js
import * as SQLite from 'expo-sqlite';

class DbHelper {
  constructor(name = 'app.db') {
    this.db = SQLite.openDatabaseSync(name);
  }

  // Run any query that returns rows
  query(sql, params = []) {
    return this.db.getAllSync(sql, params);
  }

  // Run a query that returns one row
  queryOne(sql, params = []) {
    return this.db.getFirstSync(sql, params);
  }

  // Run INSERT / UPDATE / DELETE
  execute(sql, params = []) {
    return this.db.runSync(sql, params);
  }

  // Run multiple statements (schema etc.)
  batch(sql) {
    this.db.execSync(sql);
  }

  // Run multiple writes as a transaction
  transaction(fn) {
    return this.db.withTransactionSync(fn);
  }
}

export default new DbHelper('myapp.db');
```

Usage:

```javascript
import db from './db/DbHelper';

const users = db.query('SELECT * FROM users;');
const note  = db.queryOne('SELECT * FROM notes WHERE id = ?;', [5]);
db.execute('UPDATE notes SET title = ? WHERE id = ?;', ['New Title', 5]);
```

---

## Common Patterns

### Seed data on first launch

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';
import db from './database';

async function seedIfNeeded() {
  const seeded = await AsyncStorage.getItem('db_seeded');
  if (seeded) return;

  db.withTransactionSync(() => {
    db.runSync(`INSERT INTO categories (name, color) VALUES ('Personal', '#3B82F6');`);
    db.runSync(`INSERT INTO categories (name, color) VALUES ('Work', '#10B981');`);
    db.runSync(`INSERT INTO categories (name, color) VALUES ('Ideas', '#F59E0B');`);
  });

  await AsyncStorage.setItem('db_seeded', 'true');
}
```

### Migrations (schema versioning)

```javascript
function migrateDatabase() {
  const { user_version } = db.getFirstSync('PRAGMA user_version;');

  if (user_version < 1) {
    db.execSync(`
      ALTER TABLE notes ADD COLUMN priority INTEGER DEFAULT 0;
      PRAGMA user_version = 1;
    `);
  }

  if (user_version < 2) {
    db.execSync(`
      CREATE TABLE IF NOT EXISTS reminders (
        id      INTEGER PRIMARY KEY AUTOINCREMENT,
        note_id INTEGER,
        remind_at TEXT,
        FOREIGN KEY (note_id) REFERENCES notes(id) ON DELETE CASCADE
      );
      PRAGMA user_version = 2;
    `);
  }
}
```

---

## Tips & Best Practices

| Tip | Why |
|-----|-----|
| Always use `?` placeholders | Prevents SQL injection |
| Open DB **outside** components | Prevents re-opening on re-render |
| Use `PRAGMA foreign_keys = ON` | Enforces relationships |
| Use `PRAGMA journal_mode = WAL` | Faster concurrent reads |
| Wrap bulk writes in transactions | 10–100× faster than individual inserts |
| Add indexes on columns you filter by | Speeds up `WHERE`, `ORDER BY` |
| Use `INTEGER DEFAULT 0` for booleans | SQLite has no native boolean |
| Use `TEXT DEFAULT (datetime('now'))` | ISO timestamps for dates |
| Use `result.lastInsertRowId` after INSERT | Get the new row's ID |
| Use `result.changes` after UPDATE/DELETE | Know how many rows were affected |

---

## Quick SQL Reference Card

```sql
-- Create
CREATE TABLE IF NOT EXISTS name (...);
INSERT INTO name (col1, col2) VALUES (?, ?);

-- Read
SELECT * FROM name;
SELECT * FROM name WHERE id = ?;
SELECT * FROM name WHERE col LIKE '%?%';
SELECT * FROM name ORDER BY col DESC LIMIT 10 OFFSET 20;
SELECT COUNT(*) AS total FROM name;
SELECT a.*, b.name FROM a JOIN b ON a.b_id = b.id;

-- Update
UPDATE name SET col = ? WHERE id = ?;

-- Delete
DELETE FROM name WHERE id = ?;

-- Utils
PRAGMA user_version;
PRAGMA foreign_keys = ON;
PRAGMA journal_mode = WAL;
DROP TABLE IF EXISTS name;
```
