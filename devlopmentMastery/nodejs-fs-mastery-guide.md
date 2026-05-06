# 🗂️ Mastering the Node.js File System (fs Module)
### A Complete Guide — Beginner to Production

> **Goal:** After reading this, you'll be able to create, read, update, delete, stream, and manage any file type (`.txt`, `.json`, `.csv`, `.xlsx`, `.pdf`, logs) in a real backend system — including production deployment.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Basic File Operations](#2-basic-file-operations)
3. [Working with JSON Files](#3-working-with-json-files)
4. [Directory Operations](#4-directory-operations)
5. [Streams — Deep Dive](#5-streams--deep-dive)
6. [File Upload & Download](#6-file-upload--download-backend-use-case)
7. [Generating Different File Types](#7-generating-different-file-types)
8. [Dynamic File Paths](#8-dynamic-file-paths)
9. [Temporary Files & Cleanup](#9-temporary-files--cleanup)
10. [Production Best Practices](#10-production-best-practices)
11. [VPS / Deployment Considerations](#11-vps--deployment-considerations)
12. [Common Errors & Fixes](#12-common-errors--fixes)
13. [Real-World Use Cases](#13-real-world-use-cases)
14. [Practice Exercises + Solutions](#14-practice-exercises--solutions)

---

## 1. Core Concepts

### What Is the Node.js `fs` Module?

The `fs` (File System) module is a **built-in Node.js module** that provides an API to interact with the file system on your machine or server. It allows you to:

- Read and write files
- Create and delete directories
- Watch for file changes
- Work with file permissions and metadata

No installation required — it's part of Node.js core.

```js
const fs = require('fs');           // CommonJS (callback-based)
const fs = require('fs/promises');  // CommonJS (Promise-based)
import fs from 'fs/promises';       // ESM
```

---

### Blocking vs Non-Blocking Operations

This is the most critical concept in Node.js file operations.

| | Blocking (Sync) | Non-Blocking (Async) |
|---|---|---|
| **Execution** | Halts everything until done | Continues executing other code |
| **Use case** | CLI scripts, startup config | Web servers, APIs |
| **Method suffix** | `Sync` (e.g., `readFileSync`) | No suffix / Promise-based |
| **Error handling** | `try/catch` | `.catch()` / `try/catch` with `await` |

```js
// ❌ BLOCKING — freezes your server for all users
const data = fs.readFileSync('file.txt', 'utf8');
console.log(data); // nothing else runs until this finishes

// ✅ NON-BLOCKING — Node.js handles other requests while reading
const data = await fs.promises.readFile('file.txt', 'utf8');
console.log(data); // other code can run while waiting
```

> **Rule:** Never use Sync methods in a running web server. Use them only during app startup or in CLI tools.

---

### `fs` vs `fs/promises`

```js
// Old way — callback hell
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Modern way — clean async/await
const fs = require('fs/promises');

async function readMyFile() {
  const data = await fs.readFile('file.txt', 'utf8');
  console.log(data);
}
```

**Use `fs/promises` for all new code.** It's cleaner, more readable, and works perfectly with `async/await`.

---

### When to Use Sync vs Async

```
Use SYNC when:
  ✅ Loading config files at app startup
  ✅ Writing CLI tools or scripts
  ✅ One-time setup (not inside a request handler)

Use ASYNC when:
  ✅ Inside Express/Fastify route handlers
  ✅ Any operation in a running server
  ✅ Processing multiple files concurrently
  ✅ Everything in production
```

---

## 2. Basic File Operations

### Setup

```js
const fs = require('fs/promises');
const path = require('path');
```

---

### Create / Write a File

**Callback style (old):**
```js
const fs = require('fs');

fs.writeFile('hello.txt', 'Hello, World!', 'utf8', (err) => {
  if (err) {
    console.error('Error writing file:', err);
    return;
  }
  console.log('File written successfully!');
});
```

**Async/await style (modern — use this):**
```js
const fs = require('fs/promises');

async function createFile() {
  try {
    await fs.writeFile('hello.txt', 'Hello, World!', 'utf8');
    console.log('File created!');
  } catch (err) {
    console.error('Failed to create file:', err.message);
  }
}

createFile();
```

> `writeFile` **overwrites** the file if it already exists. Use `appendFile` to add to existing content.

---

### Read a File

**Callback style:**
```js
const fs = require('fs');

fs.readFile('hello.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

**Async/await style:**
```js
const fs = require('fs/promises');

async function readMyFile() {
  try {
    const content = await fs.readFile('hello.txt', 'utf8');
    console.log(content); // "Hello, World!"
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.error('File not found!');
    } else {
      throw err;
    }
  }
}

readMyFile();
```

> Always specify encoding (`'utf8'`). Without it, you get a raw `Buffer` object.

---

### Update / Append to a File

```js
const fs = require('fs/promises');

async function updateFile() {
  try {
    // Overwrite entire file
    await fs.writeFile('log.txt', 'New content\n', 'utf8');

    // Append to existing content
    await fs.appendFile('log.txt', 'Another line\n', 'utf8');
    await fs.appendFile('log.txt', `[${new Date().toISOString()}] Event happened\n`);

    console.log('File updated!');
  } catch (err) {
    console.error('Error:', err.message);
  }
}

updateFile();
```

---

### Delete a File

**Callback style:**
```js
const fs = require('fs');

fs.unlink('hello.txt', (err) => {
  if (err) {
    console.error('Could not delete:', err);
    return;
  }
  console.log('File deleted!');
});
```

**Async/await style:**
```js
const fs = require('fs/promises');

async function deleteFile(filePath) {
  try {
    await fs.unlink(filePath);
    console.log(`Deleted: ${filePath}`);
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.log('File already does not exist — nothing to delete.');
    } else {
      throw err;
    }
  }
}

deleteFile('hello.txt');
```

---

### Check If a File Exists

```js
const fs = require('fs/promises');

async function fileExists(filePath) {
  try {
    await fs.access(filePath);
    return true;
  } catch {
    return false;
  }
}

// Usage
if (await fileExists('config.json')) {
  console.log('Config exists!');
} else {
  console.log('Config missing — creating default...');
}
```

---

### Get File Metadata (Stats)

```js
const fs = require('fs/promises');

async function getFileInfo(filePath) {
  const stats = await fs.stat(filePath);

  console.log('Size (bytes):', stats.size);
  console.log('Is file:', stats.isFile());
  console.log('Is directory:', stats.isDirectory());
  console.log('Created:', stats.birthtime);
  console.log('Modified:', stats.mtime);
}

getFileInfo('hello.txt');
```

---

## 3. Working with JSON Files

JSON files are the backbone of configuration, data storage, and API caching in Node.js backends.

### Save JSON Data to a File

```js
const fs = require('fs/promises');

async function saveJSON(filePath, data) {
  const jsonString = JSON.stringify(data, null, 2); // Pretty-print with 2-space indent
  await fs.writeFile(filePath, jsonString, 'utf8');
  console.log(`Saved JSON to ${filePath}`);
}

// Example usage
const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob',   email: 'bob@example.com'   },
];

await saveJSON('users.json', users);
```

**Output file (`users.json`):**
```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com"
  }
]
```

---

### Read and Parse JSON

```js
const fs = require('fs/promises');

async function readJSON(filePath) {
  try {
    const raw = await fs.readFile(filePath, 'utf8');
    const data = JSON.parse(raw);
    return data;
  } catch (err) {
    if (err.code === 'ENOENT') throw new Error(`File not found: ${filePath}`);
    if (err instanceof SyntaxError) throw new Error(`Invalid JSON in: ${filePath}`);
    throw err;
  }
}

// Usage
const users = await readJSON('users.json');
console.log(users[0].name); // "Alice"
```

---

### Real Example: Save API Response to JSON File

```js
const fs = require('fs/promises');

// Simulates saving a fetch() API response to disk
async function cacheAPIResponse(endpoint, data) {
  const filename = endpoint.replace(/[^a-z0-9]/gi, '_') + '.json';
  const cacheDir = './cache';

  // Create cache directory if it doesn't exist
  await fs.mkdir(cacheDir, { recursive: true });

  const filePath = `${cacheDir}/${filename}`;
  const payload = {
    cachedAt: new Date().toISOString(),
    endpoint,
    data,
  };

  await fs.writeFile(filePath, JSON.stringify(payload, null, 2), 'utf8');
  console.log(`Cached API response to ${filePath}`);
  return filePath;
}

// Usage
const apiData = { weather: 'sunny', temp: 28, city: 'Mumbai' };
await cacheAPIResponse('/api/weather', apiData);
```

---

### Update a Field in a JSON File

```js
const fs = require('fs/promises');

async function updateJSONField(filePath, key, value) {
  const data = JSON.parse(await fs.readFile(filePath, 'utf8'));
  data[key] = value;
  await fs.writeFile(filePath, JSON.stringify(data, null, 2), 'utf8');
}

await updateJSONField('config.json', 'version', '2.0.0');
```

---

## 4. Directory Operations

### Create a Directory

```js
const fs = require('fs/promises');

// Single directory
await fs.mkdir('uploads');

// Nested directories (recursive: true prevents errors if already exists)
await fs.mkdir('uploads/2024/reports', { recursive: true });
```

---

### Read Directory Contents

```js
const fs = require('fs/promises');
const path = require('path');

async function listFiles(dirPath) {
  const entries = await fs.readdir(dirPath, { withFileTypes: true });

  for (const entry of entries) {
    const fullPath = path.join(dirPath, entry.name);
    if (entry.isDirectory()) {
      console.log(`📁 ${entry.name}/`);
    } else {
      const stats = await fs.stat(fullPath);
      console.log(`📄 ${entry.name} (${stats.size} bytes)`);
    }
  }
}

listFiles('./uploads');
```

---

### Delete a Directory

```js
const fs = require('fs/promises');

// Delete empty directory
await fs.rmdir('empty-folder');

// Delete directory with all contents (Node.js 14.14+)
await fs.rm('uploads/old', { recursive: true, force: true });
// `force: true` ignores errors if the path doesn't exist
```

---

### Copy a File

```js
const fs = require('fs/promises');

// Simple copy
await fs.copyFile('original.txt', 'backup.txt');

// Copy only if destination doesn't exist
await fs.copyFile('original.txt', 'backup.txt', fs.constants.COPYFILE_EXCL);
```

---

### Rename / Move a File

```js
const fs = require('fs/promises');

// Rename within same directory
await fs.rename('old-name.txt', 'new-name.txt');

// Move to different directory (same as rename in Node.js)
await fs.rename('uploads/temp/file.txt', 'uploads/processed/file.txt');
```

---

### Recursively List All Files

```js
const fs = require('fs/promises');
const path = require('path');

async function getAllFiles(dirPath, allFiles = []) {
  const entries = await fs.readdir(dirPath, { withFileTypes: true });

  for (const entry of entries) {
    const fullPath = path.join(dirPath, entry.name);
    if (entry.isDirectory()) {
      await getAllFiles(fullPath, allFiles); // recurse
    } else {
      allFiles.push(fullPath);
    }
  }

  return allFiles;
}

const files = await getAllFiles('./src');
console.log(files);
// ['./src/index.js', './src/utils/helper.js', ...]
```

---

## 5. Streams — Deep Dive

### What Are Streams?

Streams are **data flowing over time** — instead of loading an entire file into memory at once, you process it in small **chunks**.

Think of it like this:
- **Without streams:** Watch a movie only after it fully downloads (loads entire file into RAM)
- **With streams:** Watch while it buffers (processes chunk by chunk)

### Why Streams Matter

```
Normal readFile on a 2GB log file:
  ❌ Loads entire 2GB into RAM
  ❌ Server runs out of memory
  ❌ Other requests stall

Stream reading the same 2GB log file:
  ✅ Uses ~64KB of RAM at a time
  ✅ Server stays responsive
  ✅ Can process files larger than available RAM
```

### The 4 Stream Types

| Type | Description | Example |
|---|---|---|
| **Readable** | Source of data | `fs.createReadStream()` |
| **Writable** | Destination for data | `fs.createWriteStream()` |
| **Duplex** | Both readable and writable | Network sockets |
| **Transform** | Modifies data as it flows | `zlib.createGzip()` |

---

### Read a Large File Using Streams

```js
const fs = require('fs');

function readLargeFile(filePath) {
  return new Promise((resolve, reject) => {
    const stream = fs.createReadStream(filePath, {
      encoding: 'utf8',
      highWaterMark: 64 * 1024, // 64KB chunks (default)
    });

    let totalChunks = 0;
    let totalBytes = 0;

    stream.on('data', (chunk) => {
      totalChunks++;
      totalBytes += chunk.length;
      // Process each chunk here (e.g., parse lines, search, transform)
      process.stdout.write('.'); // progress indicator
    });

    stream.on('end', () => {
      console.log(`\nDone! ${totalChunks} chunks, ${totalBytes} bytes total.`);
      resolve(totalBytes);
    });

    stream.on('error', (err) => {
      console.error('Stream error:', err);
      reject(err);
    });
  });
}

readLargeFile('./huge-log-file.log');
```

---

### Write a Large File Using Streams

```js
const fs = require('fs');

async function writeLargeFile(filePath, lineCount) {
  return new Promise((resolve, reject) => {
    const writeStream = fs.createWriteStream(filePath, { encoding: 'utf8' });

    let i = 0;

    function writeNext() {
      let ok = true;

      // Write lines until buffer is full or we're done
      while (i < lineCount && ok) {
        const line = `Line ${i + 1}: ${new Date().toISOString()} — some log data here\n`;
        ok = writeStream.write(line);
        i++;
      }

      if (i < lineCount) {
        // Buffer is full — wait for drain before writing more
        writeStream.once('drain', writeNext);
      } else {
        writeStream.end();
      }
    }

    writeStream.on('finish', () => {
      console.log(`Wrote ${lineCount} lines to ${filePath}`);
      resolve();
    });

    writeStream.on('error', reject);

    writeNext();
  });
}

await writeLargeFile('./output.log', 1_000_000); // 1 million lines
```

---

### Pipe Streams (Most Common Pattern)

Piping connects a readable stream to a writable stream — the most elegant way to process files.

```js
const fs = require('fs');

// Copy a file using streams (memory efficient)
function copyFile(src, dest) {
  return new Promise((resolve, reject) => {
    const readStream  = fs.createReadStream(src);
    const writeStream = fs.createWriteStream(dest);

    readStream
      .pipe(writeStream)
      .on('finish', () => { console.log('Copy complete!'); resolve(); })
      .on('error', reject);
  });
}

copyFile('./big-video.mp4', './backup-video.mp4');
```

---

### Compress a File with Streams (Gzip)

```js
const fs   = require('fs');
const zlib = require('zlib');

async function compressFile(inputPath, outputPath) {
  return new Promise((resolve, reject) => {
    const readStream  = fs.createReadStream(inputPath);
    const gzip        = zlib.createGzip();
    const writeStream = fs.createWriteStream(outputPath);

    readStream
      .pipe(gzip)           // compress
      .pipe(writeStream)    // write compressed output
      .on('finish', () => { console.log(`Compressed → ${outputPath}`); resolve(); })
      .on('error', reject);
  });
}

async function decompressFile(inputPath, outputPath) {
  return new Promise((resolve, reject) => {
    const readStream  = fs.createReadStream(inputPath);
    const gunzip      = zlib.createGunzip();
    const writeStream = fs.createWriteStream(outputPath);

    readStream
      .pipe(gunzip)
      .pipe(writeStream)
      .on('finish', () => { console.log(`Decompressed → ${outputPath}`); resolve(); })
      .on('error', reject);
  });
}

await compressFile('large-report.csv', 'large-report.csv.gz');
await decompressFile('large-report.csv.gz', 'restored-report.csv');
```

---

### Stream Lines from a Large File (Real-World)

Process a log file line-by-line without loading it all into memory:

```js
const fs       = require('fs');
const readline = require('readline');

async function processLogFile(filePath) {
  const fileStream = fs.createReadStream(filePath);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity, // Handle Windows line endings
  });

  let errorCount = 0;
  let lineNumber  = 0;

  for await (const line of rl) {
    lineNumber++;
    if (line.includes('ERROR')) {
      errorCount++;
      console.log(`Line ${lineNumber}: ${line}`);
    }
  }

  console.log(`\nScanned ${lineNumber} lines. Found ${errorCount} errors.`);
}

processLogFile('./app.log');
```

---

## 6. File Upload & Download (Backend Use Case)

### Handle File Uploads with Multer (Express)

```bash
npm install express multer
```

```js
const express = require('express');
const multer  = require('multer');
const path    = require('path');
const fs      = require('fs/promises');

const app = express();

// Configure storage
const storage = multer.diskStorage({
  destination: async (req, file, cb) => {
    const uploadDir = './uploads';
    await fs.mkdir(uploadDir, { recursive: true });
    cb(null, uploadDir);
  },
  filename: (req, file, cb) => {
    // Sanitize filename + add timestamp to avoid collisions
    const ext  = path.extname(file.originalname);
    const base = path.basename(file.originalname, ext).replace(/[^a-z0-9]/gi, '_');
    cb(null, `${base}_${Date.now()}${ext}`);
  },
});

// File filter — allow only images
const fileFilter = (req, file, cb) => {
  const allowed = ['image/jpeg', 'image/png', 'image/webp'];
  if (allowed.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Only JPEG, PNG, and WebP images are allowed'), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB max
});

// Upload endpoint
app.post('/upload', upload.single('image'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  res.json({
    message: 'Upload successful!',
    filename: req.file.filename,
    size: req.file.size,
    path: `/uploads/${req.file.filename}`,
  });
});

// Serve uploaded files statically
app.use('/uploads', express.static('./uploads'));

app.listen(3000, () => console.log('Server running on :3000'));
```

---

### Send a File as a Download Response

```js
const express = require('express');
const path    = require('path');
const fs      = require('fs');

const app = express();

// Download a specific file
app.get('/download/:filename', (req, res) => {
  const filename  = path.basename(req.params.filename); // prevent path traversal
  const filePath  = path.join(__dirname, 'exports', filename);

  // Check file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }

  // Stream the file as a download
  res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
  res.setHeader('Content-Type', 'application/octet-stream');

  const stream = fs.createReadStream(filePath);
  stream.pipe(res);

  stream.on('error', (err) => {
    console.error('Stream error during download:', err);
    res.status(500).end();
  });
});

// Generate on-the-fly and send without saving to disk
app.get('/export/report', (req, res) => {
  const csvData = 'Name,Age,City\nAlice,30,Mumbai\nBob,25,Delhi\n';

  res.setHeader('Content-Disposition', 'attachment; filename="report.csv"');
  res.setHeader('Content-Type', 'text/csv');
  res.send(csvData);
});
```

---

## 7. Generating Different File Types

### TXT Files

```js
const fs = require('fs/promises');

async function generateTxtReport(data, outputPath) {
  const lines = [
    '===========================',
    '       DAILY REPORT        ',
    '===========================',
    `Generated: ${new Date().toLocaleString()}`,
    '',
    ...data.map(row => `- ${row.name}: ${row.value}`),
    '',
    `Total: ${data.length} entries`,
  ];

  await fs.writeFile(outputPath, lines.join('\n'), 'utf8');
  console.log(`TXT report saved to ${outputPath}`);
}

await generateTxtReport(
  [{ name: 'Revenue', value: '₹1,20,000' }, { name: 'Users', value: '3,400' }],
  './report.txt'
);
```

---

### JSON Files

```js
const fs = require('fs/promises');

async function exportToJSON(records, outputPath) {
  const payload = {
    exportedAt: new Date().toISOString(),
    count: records.length,
    records,
  };

  await fs.writeFile(outputPath, JSON.stringify(payload, null, 2), 'utf8');
  console.log(`JSON exported to ${outputPath}`);
}
```

---

### CSV Files

```js
const fs = require('fs/promises');

async function exportToCSV(headers, rows, outputPath) {
  const escape = (val) => {
    const str = String(val ?? '');
    // Wrap in quotes if contains comma, quote, or newline
    return str.includes(',') || str.includes('"') || str.includes('\n')
      ? `"${str.replace(/"/g, '""')}"`
      : str;
  };

  const lines = [
    headers.map(escape).join(','),
    ...rows.map(row => headers.map(h => escape(row[h])).join(',')),
  ];

  await fs.writeFile(outputPath, lines.join('\n'), 'utf8');
  console.log(`CSV exported to ${outputPath}`);
}

// Usage
await exportToCSV(
  ['id', 'name', 'email', 'city'],
  [
    { id: 1, name: 'Alice',   email: 'alice@example.com', city: 'Mumbai' },
    { id: 2, name: 'Bob, Jr', email: 'bob@example.com',   city: 'Delhi'  },
  ],
  './users.csv'
);
```

---

### Excel Files (using ExcelJS)

```bash
npm install exceljs
```

```js
const ExcelJS = require('exceljs');
const path    = require('path');

async function exportToExcel(data, outputPath) {
  const workbook  = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet('Survey Results');

  // Define columns
  worksheet.columns = [
    { header: 'ID',         key: 'id',         width: 10  },
    { header: 'Name',       key: 'name',        width: 25  },
    { header: 'Score',      key: 'score',       width: 12  },
    { header: 'Submitted',  key: 'submittedAt', width: 25  },
  ];

  // Style the header row
  worksheet.getRow(1).font      = { bold: true, color: { argb: 'FFFFFFFF' } };
  worksheet.getRow(1).fill      = { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FF2E75B6' } };
  worksheet.getRow(1).alignment = { vertical: 'middle', horizontal: 'center' };

  // Add data rows
  data.forEach(row => worksheet.addRow(row));

  // Add a totals row
  const totalRow = worksheet.addRow({
    id: '',
    name: 'TOTAL',
    score: { formula: `AVERAGE(C2:C${data.length + 1})` },
  });
  totalRow.font = { bold: true };

  // Auto-filter
  worksheet.autoFilter = 'A1:D1';

  // Save
  await workbook.xlsx.writeFile(outputPath);
  console.log(`Excel file saved to ${outputPath}`);
}

await exportToExcel(
  [
    { id: 1, name: 'Alice', score: 88, submittedAt: '2024-01-15' },
    { id: 2, name: 'Bob',   score: 92, submittedAt: '2024-01-16' },
    { id: 3, name: 'Carol', score: 76, submittedAt: '2024-01-17' },
  ],
  './survey_results.xlsx'
);
```

---

### PDF Files (using Puppeteer)

```bash
npm install puppeteer
```

```js
const puppeteer = require('puppeteer');
const fs        = require('fs/promises');
const path      = require('path');

async function generatePDF(htmlContent, outputPath) {
  const browser = await puppeteer.launch({ args: ['--no-sandbox'] });
  const page    = await browser.newPage();

  await page.setContent(htmlContent, { waitUntil: 'networkidle0' });

  await page.pdf({
    path: outputPath,
    format: 'A4',
    margin: { top: '20mm', right: '15mm', bottom: '20mm', left: '15mm' },
    printBackground: true,
  });

  await browser.close();
  console.log(`PDF generated at ${outputPath}`);
}

// Build HTML template
function buildReportHTML(data) {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8">
      <style>
        body     { font-family: Arial, sans-serif; padding: 20px; color: #333; }
        h1       { color: #2E75B6; border-bottom: 2px solid #2E75B6; }
        table    { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th       { background: #2E75B6; color: white; padding: 10px; text-align: left; }
        td       { padding: 8px 10px; border-bottom: 1px solid #ddd; }
        tr:nth-child(even) { background: #f5f5f5; }
        .footer  { margin-top: 30px; font-size: 12px; color: #888; }
      </style>
    </head>
    <body>
      <h1>📊 Monthly Report</h1>
      <p>Generated: ${new Date().toLocaleString()}</p>
      <table>
        <thead>
          <tr><th>Name</th><th>Department</th><th>Score</th></tr>
        </thead>
        <tbody>
          ${data.map(r => `<tr><td>${r.name}</td><td>${r.dept}</td><td>${r.score}</td></tr>`).join('')}
        </tbody>
      </table>
      <div class="footer">Confidential — Internal Use Only</div>
    </body>
    </html>
  `;
}

const reportData = [
  { name: 'Alice', dept: 'Engineering', score: 94 },
  { name: 'Bob',   dept: 'Marketing',   score: 87 },
];

await generatePDF(buildReportHTML(reportData), './monthly_report.pdf');
```

---

## 8. Dynamic File Paths

### Why Path Management Matters

Hard-coded paths break across operating systems:
```
Windows:  C:\Users\name\project\uploads\file.txt
Linux:    /home/name/project/uploads/file.txt
```

Always use the `path` module.

---

### Using the `path` Module

```js
const path = require('path');

// Join paths safely (handles / vs \ automatically)
const filePath = path.join('uploads', '2024', 'report.pdf');
// → 'uploads/2024/report.pdf' on Linux
// → 'uploads\2024\report.pdf' on Windows

// Resolve to absolute path
const absolute = path.resolve('uploads', 'file.txt');
// → '/var/www/app/uploads/file.txt'

// Path info
path.dirname('/uploads/images/photo.jpg')   // → '/uploads/images'
path.basename('/uploads/images/photo.jpg')  // → 'photo.jpg'
path.extname('/uploads/images/photo.jpg')   // → '.jpg'

// Build safe filename
const safeName = path.basename('/etc/passwd/../../../evil');
// → 'evil' (just the file, no traversal)
```

---

### `__dirname` and `__filename`

```js
// __dirname  = directory of the CURRENT file (not where you ran node from)
// __filename = full path of the current file

const path = require('path');

const uploadsDir  = path.join(__dirname, 'uploads');
const configFile  = path.join(__dirname, '..', 'config', 'app.json');
const logsDir     = path.join(__dirname, '..', 'logs');
```

> ⚠️ In ES Modules (`"type": "module"` in package.json), `__dirname` doesn't exist. Use:
> ```js
> import { fileURLToPath } from 'url';
> import { dirname } from 'path';
> const __filename = fileURLToPath(import.meta.url);
> const __dirname  = dirname(__filename);
> ```

---

### Cross-Platform Safe File Operations

```js
const fs   = require('fs/promises');
const path = require('path');

const BASE_DIR = path.join(__dirname, 'data');

async function saveData(subPath, content) {
  // Normalize and resolve the full path
  const fullPath = path.resolve(BASE_DIR, subPath);

  // 🔒 Security: Ensure the resolved path is inside BASE_DIR
  if (!fullPath.startsWith(BASE_DIR + path.sep)) {
    throw new Error('Path traversal detected! Access denied.');
  }

  // Create parent directory if needed
  await fs.mkdir(path.dirname(fullPath), { recursive: true });
  await fs.writeFile(fullPath, content, 'utf8');

  return fullPath;
}

// Safe:  saves to ./data/reports/2024/jan.txt
await saveData('reports/2024/jan.txt', 'content here');

// Blocked: would try to escape to ../../etc/passwd
await saveData('../../etc/passwd', 'hacked'); // throws error
```

---

## 9. Temporary Files & Cleanup

### Create Temp Files

```js
const fs   = require('fs/promises');
const path = require('path');
const os   = require('os');

async function createTempFile(prefix = 'tmp', extension = '.tmp') {
  const tmpDir  = os.tmpdir(); // OS temp directory (/tmp on Linux)
  const tmpFile = path.join(tmpDir, `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2)}${extension}`);

  await fs.writeFile(tmpFile, '', 'utf8'); // create empty file
  return tmpFile;
}

// Usage pattern
async function processWithTempFile() {
  const tmpPath = await createTempFile('export', '.csv');
  console.log(`Temp file: ${tmpPath}`);

  try {
    // ... do work with tmpPath ...
    await fs.writeFile(tmpPath, 'Name,Score\nAlice,90\n', 'utf8');
    // ... send as response, upload to S3, etc ...

  } finally {
    // ALWAYS clean up, even if an error occurs
    await fs.unlink(tmpPath).catch(() => {}); // ignore if already deleted
    console.log('Temp file cleaned up');
  }
}
```

---

### Auto-Cleanup on Process Exit

```js
const fs    = require('fs');
const tmpFiles = [];

function registerTempFile(filePath) {
  tmpFiles.push(filePath);
  return filePath;
}

// Clean up all temp files when the process exits
function cleanupTempFiles() {
  for (const file of tmpFiles) {
    try {
      fs.unlinkSync(file); // sync is OK here — process is dying anyway
      console.log(`Cleaned up: ${file}`);
    } catch {
      // ignore errors during cleanup
    }
  }
}

process.on('exit', cleanupTempFiles);
process.on('SIGINT',  () => { cleanupTempFiles(); process.exit(0); });
process.on('SIGTERM', () => { cleanupTempFiles(); process.exit(0); });
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  cleanupTempFiles();
  process.exit(1);
});
```

---

### Scheduled Cleanup (Delete Old Files)

```js
const fs   = require('fs/promises');
const path = require('path');

async function cleanOldFiles(dirPath, maxAgeMs = 24 * 60 * 60 * 1000) { // default: 24 hours
  const entries = await fs.readdir(dirPath, { withFileTypes: true });
  const now     = Date.now();
  let deleted   = 0;

  for (const entry of entries) {
    if (!entry.isFile()) continue;

    const fullPath = path.join(dirPath, entry.name);
    const stats    = await fs.stat(fullPath);
    const age      = now - stats.mtimeMs;

    if (age > maxAgeMs) {
      await fs.unlink(fullPath);
      deleted++;
      console.log(`Deleted old file: ${entry.name} (age: ${Math.round(age / 3600000)}h)`);
    }
  }

  console.log(`Cleanup complete. Deleted ${deleted} files from ${dirPath}.`);
}

// Run every hour (or use a cron job in production)
setInterval(() => cleanOldFiles('./exports'), 60 * 60 * 1000);
```

---

## 10. Production Best Practices

### 1. Always Use Async — Never Block the Event Loop

```js
// ❌ NEVER in a route handler
app.get('/data', (req, res) => {
  const data = fs.readFileSync('huge-file.txt'); // BLOCKS all users!
  res.send(data);
});

// ✅ ALWAYS async in route handlers
app.get('/data', async (req, res) => {
  try {
    const data = await fs.promises.readFile('file.txt', 'utf8');
    res.send(data);
  } catch (err) {
    res.status(500).json({ error: 'Failed to read file' });
  }
});
```

---

### 2. Use Streams for Large Files

```js
// ❌ Loads entire file into memory — crashes on large files
app.get('/download', async (req, res) => {
  const data = await fs.promises.readFile('./huge-export.csv'); // 500MB → CRASH
  res.send(data);
});

// ✅ Streams it chunk by chunk
app.get('/download', (req, res) => {
  const filePath = path.join(__dirname, 'exports', 'huge-export.csv');
  res.setHeader('Content-Disposition', 'attachment; filename="export.csv"');
  fs.createReadStream(filePath).pipe(res);
});
```

---

### 3. Comprehensive Error Handling

```js
const fs = require('fs/promises');

async function safeReadFile(filePath) {
  try {
    return await fs.readFile(filePath, 'utf8');
  } catch (err) {
    switch (err.code) {
      case 'ENOENT':
        throw new Error(`File not found: ${filePath}`);
      case 'EACCES':
        throw new Error(`Permission denied: ${filePath}`);
      case 'EISDIR':
        throw new Error(`Expected a file, got a directory: ${filePath}`);
      case 'EMFILE':
        throw new Error('Too many open files — check for file descriptor leaks');
      default:
        throw new Error(`File system error (${err.code}): ${err.message}`);
    }
  }
}
```

---

### 4. Safe File Naming Strategy

```js
const path   = require('path');
const crypto = require('crypto');

// Option 1: UUID-based (best for uploads — no collisions)
function generateUniqueFilename(originalName) {
  const ext  = path.extname(originalName).toLowerCase();
  const uuid = crypto.randomUUID();
  return `${uuid}${ext}`;
  // → 'a3f4c2d1-9e8b-4f2a-b1c3-d4e5f6a7b8c9.pdf'
}

// Option 2: Timestamp + hash
function generateTimestampFilename(originalName) {
  const ext  = path.extname(originalName).toLowerCase();
  const base = path.basename(originalName, ext).replace(/[^a-z0-9]/gi, '_').slice(0, 40);
  const ts   = Date.now();
  return `${base}_${ts}${ext}`;
  // → 'user_report_1705312800000.xlsx'
}

// Option 3: Date-based directory structure (good for logs/exports)
function getDateBasedPath(baseDir, filename) {
  const now  = new Date();
  const year = now.getFullYear();
  const mon  = String(now.getMonth() + 1).padStart(2, '0');
  const day  = String(now.getDate()).padStart(2, '0');
  return path.join(baseDir, String(year), mon, day, filename);
  // → './exports/2024/01/15/report.pdf'
}
```

---

### 5. Security — Prevent Path Traversal

```js
const path = require('path');

// NEVER trust user input for file paths
app.get('/file', async (req, res) => {
  const userInput = req.query.name; // e.g., "../../etc/passwd"

  // ❌ DANGEROUS
  const dangerous = `./uploads/${userInput}`;
  // resolves to: /var/www/uploads/../../etc/passwd → /etc/passwd

  // ✅ SAFE
  const safeBase = path.resolve('./uploads');
  const resolved = path.resolve(safeBase, path.basename(userInput)); // basename strips directories

  if (!resolved.startsWith(safeBase + path.sep) && resolved !== safeBase) {
    return res.status(403).json({ error: 'Access denied' });
  }

  // Now safe to use `resolved`
  const data = await fs.promises.readFile(resolved, 'utf8');
  res.send(data);
});
```

---

### 6. Limit Concurrent File Operations

```js
// Process 100 files but only 5 at a time to avoid EMFILE errors
async function processFilesWithConcurrency(files, concurrency = 5) {
  const results = [];
  for (let i = 0; i < files.length; i += concurrency) {
    const batch  = files.slice(i, i + concurrency);
    const output = await Promise.all(batch.map(processFile));
    results.push(...output);
  }
  return results;
}
```

---

## 11. VPS / Deployment Considerations

### File Permissions on Linux

```bash
# Check permissions
ls -la /var/www/app/uploads

# Fix ownership (give app user ownership)
chown -R www-data:www-data /var/www/app/uploads

# Set directory permissions: rwxr-xr-x (755)
chmod 755 /var/www/app/uploads

# Set file permissions: rw-r--r-- (644)
chmod 644 /var/www/app/uploads/*.pdf

# For generated files (app creates + reads): rwxrwxr-x (775)
chmod 775 /var/www/app/exports
```

---

### Create Required Directories on Startup

```js
// startup.js — run before server starts
const fs   = require('fs/promises');
const path = require('path');

const REQUIRED_DIRS = [
  path.join(__dirname, 'uploads'),
  path.join(__dirname, 'uploads', 'images'),
  path.join(__dirname, 'exports'),
  path.join(__dirname, 'logs'),
  path.join(__dirname, 'tmp'),
];

async function ensureDirectories() {
  for (const dir of REQUIRED_DIRS) {
    await fs.mkdir(dir, { recursive: true });
    console.log(`✅ Directory ready: ${dir}`);
  }
}

module.exports = ensureDirectories;
```

```js
// server.js
const ensureDirectories = require('./startup');
const app = require('./app');

ensureDirectories()
  .then(() => app.listen(3000, () => console.log('Server started')))
  .catch(err => { console.error('Startup failed:', err); process.exit(1); });
```

---

### Disk Usage Monitoring

```js
const { exec } = require('child_process');
const fs       = require('fs/promises');

// Check directory size (Linux/Mac)
function getDirSize(dirPath) {
  return new Promise((resolve, reject) => {
    exec(`du -sh ${dirPath}`, (err, stdout) => {
      if (err) reject(err);
      else resolve(stdout.split('\t')[0]); // e.g., "142M"
    });
  });
}

// Check available disk space
function getFreeDiskSpace() {
  return new Promise((resolve, reject) => {
    exec("df -h / | awk 'NR==2{print $4}'", (err, stdout) => {
      if (err) reject(err);
      else resolve(stdout.trim()); // e.g., "45G"
    });
  });
}

// Monitor and alert
async function checkDiskHealth() {
  const free = await getFreeDiskSpace();
  const uploadsSize = await getDirSize('./uploads');
  console.log(`Disk free: ${free} | Uploads: ${uploadsSize}`);

  // Alert if less than 1GB free (parse and check)
  // In production: send alert to Slack/PagerDuty
}

setInterval(checkDiskHealth, 30 * 60 * 1000); // every 30 minutes
```

---

### When to Use Cloud Storage (S3/GCS)

Use cloud object storage instead of local disk when:

| Scenario | Use Local Disk | Use Cloud Storage |
|---|---|---|
| Single server | ✅ | |
| Multiple servers / horizontal scaling | | ✅ |
| Files need CDN delivery | | ✅ |
| Backups & durability requirements | | ✅ |
| Temp files during processing | ✅ | |
| Files > 5GB | | ✅ |

**Quick S3 upload example (AWS SDK v3):**

```bash
npm install @aws-sdk/client-s3
```

```js
const { S3Client, PutObjectCommand, GetObjectCommand } = require('@aws-sdk/client-s3');
const fs = require('fs');

const s3 = new S3Client({ region: 'ap-south-1' });

async function uploadToS3(localPath, bucketName, s3Key) {
  const fileStream = fs.createReadStream(localPath);

  await s3.send(new PutObjectCommand({
    Bucket: bucketName,
    Key: s3Key,
    Body: fileStream,
  }));

  console.log(`Uploaded to s3://${bucketName}/${s3Key}`);
}
```

---

## 12. Common Errors & Fixes

### ENOENT — File Not Found

```
Error: ENOENT: no such file or directory, open 'data/config.json'
```

**Causes & Fixes:**
```js
// Cause 1: Relative path issue — depends on where you run node
// Fix: Always use absolute paths with __dirname
const configPath = path.join(__dirname, 'data', 'config.json');

// Cause 2: Directory doesn't exist yet
// Fix: Create directory first
await fs.mkdir(path.dirname(configPath), { recursive: true });
await fs.writeFile(configPath, '{}', 'utf8');

// Cause 3: Race condition — check-then-act
// Fix: Don't check existence, just handle the error
try {
  const data = await fs.readFile(filePath, 'utf8');
} catch (err) {
  if (err.code === 'ENOENT') {
    // Handle gracefully
  }
}
```

---

### EACCES — Permission Denied

```
Error: EACCES: permission denied, open '/var/log/app.log'
```

**Causes & Fixes:**
```bash
# Check who owns the file
ls -la /var/log/app.log

# Fix: Give the app user ownership
sudo chown nodeuser:nodeuser /var/log/app.log

# Fix: Add write permission
sudo chmod 644 /var/log/app.log

# Fix: Run app as correct user (not root in production!)
sudo -u nodeuser node server.js
```

---

### EMFILE — Too Many Open Files

```
Error: EMFILE: too many open files, open './uploads/file.txt'
```

**Causes & Fixes:**
```js
// Cause: Opening thousands of files without closing them
// Fix 1: Process files with concurrency limits (see section 10)
// Fix 2: Increase OS file descriptor limit

// Check current limit
$ ulimit -n        # typically 1024

// Increase temporarily
$ ulimit -n 65535

// Increase permanently — add to /etc/security/limits.conf:
// nodeuser soft nofile 65535
// nodeuser hard nofile 65535

// Fix 3: Ensure streams are properly destroyed on error
const stream = fs.createReadStream(filePath);
stream.on('error', (err) => {
  stream.destroy(); // release file descriptor
  console.error(err);
});
```

---

### ENOSPC — No Space Left on Device

```
Error: ENOSPC: no space left on device
```

**Causes & Fixes:**
```bash
# Check disk usage
df -h

# Find large directories
du -sh /* 2>/dev/null | sort -rh | head -20

# Clear temp files
rm -rf /tmp/*.tmp

# Clear old logs
find /var/log -name "*.log" -mtime +30 -delete

# In Node.js: wrap writes in try/catch and alert on ENOSPC
try {
  await fs.writeFile(path, data);
} catch (err) {
  if (err.code === 'ENOSPC') {
    // Alert ops team, reject request gracefully
    throw new Error('Storage full — contact admin');
  }
  throw err;
}
```

---

### Large File Crashes / Memory Issues

```
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

**Cause:** Using `readFile` on a file larger than available RAM.

**Fix:** Always use streams for files > 50MB.

```js
// ❌ Crashes on 2GB file
const data = await fs.readFile('huge.csv', 'utf8');

// ✅ Handles 2GB file with ~64KB RAM
const stream = fs.createReadStream('huge.csv', { encoding: 'utf8' });
for await (const chunk of stream) {
  // process chunk
}
```

---

## 13. Real-World Use Cases

### Use Case 1: Export Survey Data to Excel

```js
const ExcelJS = require('exceljs');
const fs      = require('fs/promises');
const path    = require('path');

async function exportSurveyToExcel(surveyId, responses) {
  const workbook = new ExcelJS.Workbook();
  workbook.creator = 'My App';
  workbook.created = new Date();

  const sheet = workbook.addWorksheet('Responses');

  sheet.columns = [
    { header: 'Respondent ID', key: 'id',          width: 18  },
    { header: 'Name',          key: 'name',         width: 25  },
    { header: 'Q1: Rating',    key: 'q1',           width: 14  },
    { header: 'Q2: Feedback',  key: 'q2',           width: 50  },
    { header: 'Submitted At',  key: 'submittedAt',  width: 22  },
  ];

  // Header styling
  sheet.getRow(1).eachCell(cell => {
    cell.font = { bold: true, color: { argb: 'FFFFFFFF' } };
    cell.fill = { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FF1F4E79' } };
    cell.border = { bottom: { style: 'medium' } };
  });

  // Data rows with alternating color
  responses.forEach((r, idx) => {
    const row = sheet.addRow(r);
    if (idx % 2 === 0) {
      row.eachCell(cell => {
        cell.fill = { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FFDAE3F3' } };
      });
    }
  });

  // Save to exports directory
  const exportsDir = path.join(__dirname, 'exports');
  await fs.mkdir(exportsDir, { recursive: true });
  const filename = `survey_${surveyId}_${Date.now()}.xlsx`;
  const filePath = path.join(exportsDir, filename);

  await workbook.xlsx.writeFile(filePath);
  console.log(`Survey exported: ${filename}`);
  return filePath;
}
```

---

### Use Case 2: Generate PDF Report and Store It

```js
const puppeteer = require('puppeteer');
const fs        = require('fs/promises');
const path      = require('path');

async function generateAndStorePDFReport(reportData) {
  const browser = await puppeteer.launch({ args: ['--no-sandbox', '--disable-setuid-sandbox'] });

  try {
    const page = await browser.newPage();

    const html = `
      <html><head>
        <style>
          body { font-family: Arial, sans-serif; margin: 40px; }
          h1   { color: #1F4E79; }
          .metric { display: inline-block; margin: 10px 20px 10px 0; padding: 15px 20px;
                    background: #f0f4f9; border-left: 4px solid #1F4E79; border-radius: 4px; }
          .metric .value { font-size: 28px; font-weight: bold; color: #1F4E79; }
          .metric .label { font-size: 13px; color: #666; margin-top: 4px; }
        </style>
      </head><body>
        <h1>Monthly Performance Report</h1>
        <p>Period: ${reportData.period} | Generated: ${new Date().toLocaleDateString()}</p>
        ${reportData.metrics.map(m => `
          <div class="metric">
            <div class="value">${m.value}</div>
            <div class="label">${m.label}</div>
          </div>
        `).join('')}
      </body></html>
    `;

    await page.setContent(html, { waitUntil: 'networkidle0' });

    const reportsDir = path.join(__dirname, 'reports');
    await fs.mkdir(reportsDir, { recursive: true });

    const filename = `report_${reportData.period.replace(/\s/g, '_')}_${Date.now()}.pdf`;
    const filePath = path.join(reportsDir, filename);

    await page.pdf({ path: filePath, format: 'A4', printBackground: true,
                     margin: { top: '15mm', right: '15mm', bottom: '15mm', left: '15mm' } });

    return filePath;
  } finally {
    await browser.close();
  }
}
```

---

### Use Case 3: Rotating Application Logger

```js
const fs   = require('fs/promises');
const path = require('path');

class FileLogger {
  constructor(logsDir, maxFileSize = 10 * 1024 * 1024) { // 10MB max
    this.logsDir     = logsDir;
    this.maxFileSize = maxFileSize;
    this.currentFile = null;
    this.ready       = false;
  }

  async init() {
    await fs.mkdir(this.logsDir, { recursive: true });
    this.currentFile = this._getLogPath();
    this.ready = true;
  }

  _getLogPath() {
    const date = new Date().toISOString().split('T')[0]; // '2024-01-15'
    return path.join(this.logsDir, `app_${date}.log`);
  }

  async log(level, message, meta = {}) {
    if (!this.ready) await this.init();

    // Rotate if date changed
    this.currentFile = this._getLogPath();

    // Rotate if file too large
    try {
      const stats = await fs.stat(this.currentFile);
      if (stats.size > this.maxFileSize) {
        const rotated = this.currentFile.replace('.log', `_${Date.now()}.log`);
        await fs.rename(this.currentFile, rotated);
      }
    } catch { /* file doesn't exist yet — that's fine */ }

    const entry = JSON.stringify({
      ts: new Date().toISOString(),
      level,
      message,
      ...meta,
    }) + '\n';

    await fs.appendFile(this.currentFile, entry, 'utf8');
  }

  info(msg, meta)  { return this.log('INFO',  msg, meta); }
  warn(msg, meta)  { return this.log('WARN',  msg, meta); }
  error(msg, meta) { return this.log('ERROR', msg, meta); }
}

// Usage
const logger = new FileLogger('./logs');
await logger.init();
await logger.info('Server started', { port: 3000 });
await logger.error('Database connection failed', { host: 'db.example.com', code: 'ETIMEDOUT' });
```

---

### Use Case 4: Upload and Serve Profile Images

```js
const express = require('express');
const multer  = require('multer');
const sharp   = require('sharp');   // npm install sharp
const path    = require('path');
const fs      = require('fs/promises');

const app = express();

const storage = multer.memoryStorage(); // read into RAM first for processing
const upload  = multer({ storage, limits: { fileSize: 10 * 1024 * 1024 } });

app.post('/profile/upload', upload.single('avatar'), async (req, res) => {
  if (!req.file) return res.status(400).json({ error: 'No image provided' });

  try {
    const userId   = req.body.userId || 'anonymous';
    const filename = `${userId}_${Date.now()}.jpg`;
    const savePath = path.join(__dirname, 'uploads', 'avatars', filename);

    await fs.mkdir(path.dirname(savePath), { recursive: true });

    // Resize and optimize with Sharp before saving
    await sharp(req.file.buffer)
      .resize(200, 200, { fit: 'cover' })
      .jpeg({ quality: 85 })
      .toFile(savePath);

    res.json({
      message:  'Avatar uploaded!',
      imageUrl: `/avatars/${filename}`,
    });
  } catch (err) {
    console.error('Upload error:', err);
    res.status(500).json({ error: 'Upload failed' });
  }
});

app.use('/avatars', express.static(path.join(__dirname, 'uploads', 'avatars')));
```

---

## 14. Practice Exercises + Solutions

### Exercise 1: Create and Read a JSON Config File

**Task:** Write a function that reads a config file. If it doesn't exist, create it with default values and return those.

<details>
<summary>💡 Solution</summary>

```js
const fs   = require('fs/promises');
const path = require('path');

const CONFIG_PATH = path.join(__dirname, 'config.json');

const DEFAULT_CONFIG = {
  appName:  'My App',
  version:  '1.0.0',
  port:     3000,
  debug:    false,
  database: { host: 'localhost', port: 5432 },
};

async function loadConfig() {
  try {
    const raw = await fs.readFile(CONFIG_PATH, 'utf8');
    const config = JSON.parse(raw);
    console.log('Config loaded:', config);
    return config;
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.log('Config not found — creating defaults...');
      await fs.writeFile(CONFIG_PATH, JSON.stringify(DEFAULT_CONFIG, null, 2), 'utf8');
      return DEFAULT_CONFIG;
    }
    throw err;
  }
}

const config = await loadConfig();
console.log(`App: ${config.appName} v${config.version}`);
```
</details>

---

### Exercise 2: Build a File Download API

**Task:** Create an Express route `/export` that generates a CSV in memory and sends it as a file download.

<details>
<summary>💡 Solution</summary>

```js
const express = require('express');
const app     = express();

// Sample data (in real app, this comes from DB)
const SALES_DATA = [
  { date: '2024-01-01', product: 'Widget A', qty: 50,  revenue: 2500 },
  { date: '2024-01-02', product: 'Widget B', qty: 30,  revenue: 1800 },
  { date: '2024-01-03', product: 'Widget A', qty: 70,  revenue: 3500 },
];

function toCSV(headers, rows) {
  const lines = [headers.join(',')];
  rows.forEach(row => lines.push(headers.map(h => row[h] ?? '').join(',')));
  return lines.join('\n');
}

app.get('/export', (req, res) => {
  const csv = toCSV(['date', 'product', 'qty', 'revenue'], SALES_DATA);

  res.setHeader('Content-Disposition', 'attachment; filename="sales_export.csv"');
  res.setHeader('Content-Type', 'text/csv; charset=utf-8');
  res.send(csv);
});

app.listen(3000, () => console.log('Server on :3000 — try: curl -O http://localhost:3000/export'));
```
</details>

---

### Exercise 3: Stream a Large File

**Task:** Create a function that reads a large text file line-by-line and counts occurrences of a keyword, reporting progress every 100,000 lines.

<details>
<summary>💡 Solution</summary>

```js
const fs       = require('fs');
const readline = require('readline');

async function countKeywordInFile(filePath, keyword) {
  const fileStream = fs.createReadStream(filePath, { encoding: 'utf8' });
  const rl = readline.createInterface({ input: fileStream, crlfDelay: Infinity });

  let lineCount = 0;
  let matches   = 0;
  const lowerKw = keyword.toLowerCase();

  console.time('Scan time');

  for await (const line of rl) {
    lineCount++;
    if (line.toLowerCase().includes(lowerKw)) matches++;

    if (lineCount % 100_000 === 0) {
      process.stdout.write(`\rProcessed ${lineCount.toLocaleString()} lines...`);
    }
  }

  console.timeEnd('\nScan time');
  console.log(`\nTotal lines: ${lineCount.toLocaleString()}`);
  console.log(`Matches for "${keyword}": ${matches}`);

  return { lineCount, matches };
}

// To test: create a large file first
const fs2 = require('fs');
const ws  = fs2.createWriteStream('./test-large.txt');
for (let i = 0; i < 500_000; i++) {
  ws.write(`Line ${i}: ${Math.random() > 0.99 ? 'ERROR occurred here' : 'all good'}\n`);
}
ws.end(() => countKeywordInFile('./test-large.txt', 'ERROR'));
```
</details>

---

### Exercise 4: Generate a CSV Export for a Survey

**Task:** Write a function that takes an array of survey responses and exports a well-formatted CSV with proper escaping.

<details>
<summary>💡 Solution</summary>

```js
const fs   = require('fs/promises');
const path = require('path');

async function exportSurveyCSV(surveyTitle, responses, outputDir = './exports') {
  await fs.mkdir(outputDir, { recursive: true });

  const escape = (val) => {
    const s = String(val ?? '');
    return (s.includes(',') || s.includes('"') || s.includes('\n'))
      ? `"${s.replace(/"/g, '""')}"` : s;
  };

  const headers = ['Response ID', 'Name', 'Email', 'Q1: Rating (1-5)', 'Q2: Comments', 'Submitted At'];

  const rows = responses.map(r => [
    r.id, r.name, r.email, r.rating, r.comments, r.submittedAt
  ]);

  const csv = [
    `# Survey: ${surveyTitle}`,
    `# Exported: ${new Date().toISOString()}`,
    `# Total responses: ${responses.length}`,
    '',
    headers.map(escape).join(','),
    ...rows.map(row => row.map(escape).join(',')),
  ].join('\n');

  const filename = `survey_${surveyTitle.replace(/\s/g, '_')}_${Date.now()}.csv`;
  const filePath = path.join(outputDir, filename);
  await fs.writeFile(filePath, csv, 'utf8');

  console.log(`Exported ${responses.length} responses to ${filename}`);
  return filePath;
}

// Test it
await exportSurveyCSV('Customer Satisfaction Q1 2024', [
  { id: 1, name: 'Alice',     email: 'alice@ex.com', rating: 5, comments: 'Great service!',       submittedAt: '2024-01-10' },
  { id: 2, name: 'Bob "Jr"',  email: 'bob@ex.com',   rating: 3, comments: 'Could be better, tbh', submittedAt: '2024-01-11' },
  { id: 3, name: 'Carol',     email: 'c@ex.com',     rating: 4, comments: 'Good,\nbut slow.',      submittedAt: '2024-01-12' },
]);
```
</details>

---

### Exercise 5: Build a Simple File-Based Cache

**Task:** Implement a `FileCache` class with `get`, `set`, and `invalidate` methods that stores JSON data in files with a TTL (time-to-live).

<details>
<summary>💡 Solution</summary>

```js
const fs   = require('fs/promises');
const path = require('path');

class FileCache {
  constructor(cacheDir = './.cache', defaultTTL = 60 * 60 * 1000) { // 1 hour default
    this.cacheDir   = cacheDir;
    this.defaultTTL = defaultTTL;
  }

  async _init() {
    await fs.mkdir(this.cacheDir, { recursive: true });
  }

  _keyToPath(key) {
    const safe = key.replace(/[^a-z0-9_-]/gi, '_');
    return path.join(this.cacheDir, `${safe}.cache.json`);
  }

  async get(key) {
    try {
      const raw   = await fs.readFile(this._keyToPath(key), 'utf8');
      const entry = JSON.parse(raw);

      if (Date.now() > entry.expiresAt) {
        await this.invalidate(key); // expired
        return null;
      }

      return entry.value;
    } catch {
      return null; // miss
    }
  }

  async set(key, value, ttl = this.defaultTTL) {
    await this._init();
    const entry = { key, value, createdAt: Date.now(), expiresAt: Date.now() + ttl };
    await fs.writeFile(this._keyToPath(key), JSON.stringify(entry), 'utf8');
  }

  async invalidate(key) {
    await fs.unlink(this._keyToPath(key)).catch(() => {});
  }

  async clear() {
    const files = await fs.readdir(this.cacheDir);
    await Promise.all(
      files.filter(f => f.endsWith('.cache.json'))
           .map(f => fs.unlink(path.join(this.cacheDir, f)))
    );
  }
}

// Test it
const cache = new FileCache('./.cache', 5000); // 5 second TTL

await cache.set('user:123', { name: 'Alice', role: 'admin' });
console.log(await cache.get('user:123')); // { name: 'Alice', role: 'admin' }

await new Promise(r => setTimeout(r, 6000)); // wait 6 seconds
console.log(await cache.get('user:123')); // null (expired)
```
</details>

---

## Quick Reference Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                    fs Module Quick Reference                        │
├────────────────────────┬────────────────────────────────────────────┤
│ OPERATION              │ METHOD                                     │
├────────────────────────┼────────────────────────────────────────────┤
│ Write file             │ fs.writeFile(path, data)                   │
│ Read file              │ fs.readFile(path, 'utf8')                  │
│ Append to file         │ fs.appendFile(path, data)                  │
│ Delete file            │ fs.unlink(path)                            │
│ Copy file              │ fs.copyFile(src, dest)                     │
│ Rename / Move          │ fs.rename(src, dest)                       │
│ File metadata          │ fs.stat(path)                              │
│ Check access           │ fs.access(path)                            │
├────────────────────────┼────────────────────────────────────────────┤
│ Create directory       │ fs.mkdir(path, { recursive: true })        │
│ List directory         │ fs.readdir(path, { withFileTypes: true })  │
│ Delete directory       │ fs.rm(path, { recursive: true })           │
├────────────────────────┼────────────────────────────────────────────┤
│ Read stream            │ fs.createReadStream(path)                  │
│ Write stream           │ fs.createWriteStream(path)                 │
│ Pipe                   │ readStream.pipe(writeStream)               │
├────────────────────────┼────────────────────────────────────────────┤
│ Error codes            │                                            │
│   File not found       │ ENOENT                                     │
│   Permission denied    │ EACCES                                     │
│   Too many open files  │ EMFILE                                     │
│   No disk space        │ ENOSPC                                     │
│   Is a directory       │ EISDIR                                     │
└────────────────────────┴────────────────────────────────────────────┘
```

---

*This guide covers Node.js LTS (18/20/22). All code uses `fs/promises` (recommended) unless streams require the callback-based `fs` module directly.*
