# 🚀 Mastering Puppeteer for PDF Generation
### A Complete Beginner-to-Production Guide

> **Goal:** After reading this guide, you will be able to generate any type of PDF — invoices, reports, dashboards, analytics — from dynamic backend data and deploy confidently on a VPS.

---

## Table of Contents

1. [Core Concepts of Puppeteer](#1-core-concepts-of-puppeteer)
2. [Setup & Hello World PDF](#2-setup--hello-world-pdf)
3. [HTML to PDF Workflow](#3-html-to-pdf-workflow)
4. [Styling & Layout](#4-styling--layout)
5. [Dynamic Data Injection](#5-dynamic-data-injection)
6. [Advanced PDF Features](#6-advanced-pdf-features)
7. [Generating Complex Documents](#7-generating-complex-documents)
8. [Charts in PDF](#8-charts-in-pdf)
9. [puppeteer vs puppeteer-core](#9-puppeteer-vs-puppeteer-core)
10. [VPS / Production Deployment](#10-vps--production-deployment)
11. [Performance Optimization](#11-performance-optimization)
12. [File Handling](#12-file-handling)
13. [Common Errors & Fixes](#13-common-errors--fixes)
14. [Best Practices](#14-best-practices)
15. [Practice Exercises](#15-practice-exercises)

---

## 1. Core Concepts of Puppeteer

### What is Puppeteer?

Puppeteer is a Node.js library maintained by Google that provides a high-level API to control **Chromium** (or Chrome) over the **DevTools Protocol**. It allows you to:

- Automate browser interactions (click, type, scroll)
- Take screenshots
- **Generate PDFs from HTML pages** ← our focus
- Run end-to-end tests
- Scrape data from websites

The key insight: **Puppeteer is essentially remote-controlling a real browser programmatically.**

### How Headless Chromium Works

Chromium (the open-source base of Google Chrome) can run in two modes:

| Mode | Description |
|------|-------------|
| **Headed** | Opens a visible browser window (for debugging) |
| **Headless** | Runs entirely in memory, no GUI, no display needed |

When you run headless Chromium:

```
Your Node.js Script
      ↓
  Puppeteer API
      ↓
DevTools Protocol (WebSocket)
      ↓
Headless Chromium Process
      ↓
  Renders HTML/CSS/JS
      ↓
  Outputs PDF bytes
```

The browser is a **real rendering engine** — the exact same layout and rendering as Chrome. This is what makes Puppeteer so powerful for PDFs: you get pixel-perfect rendering.

### Puppeteer vs Puppeteer-core vs Playwright

| Feature | `puppeteer` | `puppeteer-core` | `playwright` |
|---------|-------------|------------------|--------------|
| **Bundled Chromium** | ✅ Yes (downloads ~170MB) | ❌ No | ❌ No (downloads separately) |
| **Install size** | ~200MB | ~10MB | ~200MB+ |
| **Browser support** | Chromium only | Any browser via path | Chrome, Firefox, Safari, Edge |
| **API style** | Puppeteer API | Puppeteer API | Playwright API (different) |
| **Use case** | Simple setups | VPS/Docker/Custom | Multi-browser testing |
| **VPS recommended** | ❌ (size issues) | ✅ Use system Chromium | ✅ Alternative |

**Rule of thumb:**
- Learning/local dev → `puppeteer`
- Production VPS → `puppeteer-core` + system Chromium
- Multi-browser needs → `playwright`

### How PDF Generation Actually Works

The complete flow:

```
1. You write HTML + CSS (your template)
         ↓
2. Puppeteer loads it into Chromium
         ↓
3. Chromium renders it exactly like a webpage
         ↓
4. You call page.pdf() — Chromium's print engine kicks in
         ↓
5. Chromium applies print CSS (@media print)
         ↓
6. Output: PDF binary buffer
         ↓
7. Save to disk OR stream to HTTP response
```

**Critical understanding:** PDF generation is essentially "printing a webpage". The same CSS `@media print` rules that control browser printing apply here. This means CSS is your main layout tool.

---

## 2. Setup & Hello World PDF

### Install Puppeteer

```bash
# Create a new project
mkdir pdf-generator && cd pdf-generator
npm init -y

# Install puppeteer (downloads Chromium automatically)
npm install puppeteer

# For production VPS (no bundled Chromium):
# npm install puppeteer-core
```

### Project Structure

```
pdf-generator/
├── src/
│   ├── templates/        # HTML templates
│   ├── generators/       # PDF generation logic
│   └── utils/            # Helpers
├── output/               # Generated PDFs
├── package.json
└── index.js
```

### Hello World PDF — Complete Working Example

```javascript
// generate-hello.js
const puppeteer = require('puppeteer');
const path = require('path');

async function generateHelloPDF() {
  // 1. Launch browser
  const browser = await puppeteer.launch({
    headless: 'new',  // Use new headless mode (recommended)
    args: ['--no-sandbox', '--disable-setuid-sandbox']  // Required on most Linux servers
  });

  // 2. Open a new page (tab)
  const page = await browser.newPage();

  // 3. Set HTML content directly
  await page.setContent(`
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <style>
          body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: #f0f4ff;
          }
          .card {
            background: white;
            padding: 60px 80px;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            text-align: center;
          }
          h1 { color: #2563eb; font-size: 48px; margin: 0 0 16px; }
          p  { color: #64748b; font-size: 20px; margin: 0; }
        </style>
      </head>
      <body>
        <div class="card">
          <h1>Hello, Puppeteer! 👋</h1>
          <p>Your first PDF has been generated successfully.</p>
          <p>Generated at: ${new Date().toLocaleString()}</p>
        </div>
      </body>
    </html>
  `, { waitUntil: 'networkidle0' }); // Wait until all network requests settle

  // 4. Generate the PDF
  const outputPath = path.join(__dirname, 'output', 'hello-world.pdf');
  
  await page.pdf({
    path: outputPath,          // Save to disk (omit this to get a Buffer)
    format: 'A4',              // Page size
    printBackground: true,     // IMPORTANT: Print background colors/images
    margin: {
      top: '20mm',
      right: '20mm',
      bottom: '20mm',
      left: '20mm'
    }
  });

  // 5. Always close the browser
  await browser.close();

  console.log(`✅ PDF saved to: ${outputPath}`);
}

// Create output directory and run
const fs = require('fs');
fs.mkdirSync('./output', { recursive: true });

generateHelloPDF().catch(console.error);
```

```bash
# Run it
node generate-hello.js
# ✅ PDF saved to: /your/path/output/hello-world.pdf
```

### Key `page.pdf()` Options Reference

```javascript
await page.pdf({
  path: 'output.pdf',          // File path. Omit to get Buffer
  format: 'A4',                // 'A4' | 'Letter' | 'Legal' | 'A3' | 'Tabloid'
  width: '210mm',              // Custom width (overrides format)
  height: '297mm',             // Custom height (overrides format)
  landscape: false,            // Landscape orientation
  printBackground: true,       // Print background colors & images (ALWAYS set true)
  margin: {                    // Page margins
    top: '20mm',
    right: '20mm',
    bottom: '20mm',
    left: '20mm'
  },
  scale: 1,                    // Scale factor 0.1–2
  pageRanges: '1-5',           // Print specific pages only
  displayHeaderFooter: true,   // Enable header/footer
  headerTemplate: '<div>...</div>',
  footerTemplate: '<div>...</div>',
  omitBackground: false,       // Make PDF background transparent
  tagged: true,                // Generate tagged (accessible) PDF
  timeout: 30000               // Timeout in ms
});
```

---

## 3. HTML to PDF Workflow

### Why HTML is the Base for All PDFs

HTML + CSS is the most powerful document layout system ever built. Using it for PDFs gives you:

- **Tables** with complex spanning
- **Flexbox and Grid** layouts
- **Custom fonts** via Google Fonts or local files
- **Colors, gradients, shadows**
- **Dynamic content** via JavaScript template literals
- **Print-specific CSS** via `@media print`
- **Full Unicode support** — emojis, Arabic, Hindi, etc.

### Structuring HTML Templates

A production-quality HTML template structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    /* ===== RESET & BASE ===== */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    
    /* ===== PAGE CONFIG ===== */
    /* Controls the physical page in print/PDF */
    @page {
      size: A4;
      margin: 15mm 20mm;
    }
    
    body {
      font-family: 'Arial', sans-serif;
      font-size: 14px;
      color: #1e293b;
      line-height: 1.5;
      background: white;
    }

    /* ===== PRINT RULES ===== */
    @media print {
      .no-print { display: none !important; }
      .page-break { page-break-after: always; }
      .avoid-break { page-break-inside: avoid; }
    }

    /* ===== LAYOUT ===== */
    .page { 
      width: 210mm;         /* Match A4 width */
      min-height: 297mm;    /* Match A4 height */
      margin: 0 auto;
      padding: 20mm;
    }

    /* ===== COMPONENTS ===== */
    /* Your document-specific styles */
  </style>
</head>
<body>
  <!-- Your dynamic content here -->
</body>
</html>
```

### Inline Styles vs External CSS

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Inline `<style>`** | Self-contained, no file loading | Verbose | Always preferred for Puppeteer |
| **External CSS file** | Clean, reusable | Must use `file://` path or wait for load | Large shared stylesheets |
| **CDN CSS** (Tailwind) | Powerful utilities | Requires network, slow | OK with `waitUntil: 'networkidle0'` |

**Best practice:** Use inline `<style>` tags. It makes your template self-contained and eliminates file-loading timing issues.

```javascript
// ✅ CORRECT — CSS inline, always loads
await page.setContent(`
  <html><head><style>
    body { font-family: Arial; }
  </style></head>
  <body>Content</body></html>
`, { waitUntil: 'networkidle0' });

// ⚠️ External CSS — must set base URL
await page.setContent(html, { waitUntil: 'networkidle0' });
// OR use page.goto() with a file:// URL:
await page.goto(`file://${path.resolve('./template.html')}`);
```

### Using Tailwind CSS in Puppeteer

```javascript
// Use Tailwind CDN — works fine with networkidle0
const html = `
  <!DOCTYPE html>
  <html>
  <head>
    <script src="https://cdn.tailwindcss.com"></script>
  </head>
  <body class="bg-gray-50 p-8">
    <div class="bg-white rounded-xl shadow-lg p-8">
      <h1 class="text-3xl font-bold text-blue-600">Tailwind PDF</h1>
      <p class="text-gray-500 mt-2">This uses Tailwind utility classes.</p>
    </div>
  </body>
  </html>
`;

await page.setContent(html, { waitUntil: 'networkidle0' }); // Wait for Tailwind to load
```

### Core Example: JSON Data → HTML → PDF

This is the fundamental pattern you'll use for everything.

```javascript
// src/generators/dataReport.js
const puppeteer = require('puppeteer');
const fs = require('fs');

// --- SAMPLE DATA (from your backend/database) ---
const reportData = {
  title: 'Monthly Sales Report',
  period: 'January 2025',
  generatedAt: new Date().toLocaleDateString(),
  summary: {
    totalRevenue: 84320,
    totalOrders: 342,
    avgOrderValue: 246.55,
    growth: '+12.4%'
  },
  topProducts: [
    { name: 'Pro Subscription', units: 128, revenue: 38400 },
    { name: 'Starter Plan', units: 214, revenue: 21400 },
    { name: 'Enterprise License', units: 12, revenue: 24000 },
    { name: 'API Add-on', units: 89, revenue: 8900 }
  ]
};

// --- HTML TEMPLATE FUNCTION ---
function buildReportHTML(data) {
  const productRows = data.topProducts.map(p => `
    <tr>
      <td>${p.name}</td>
      <td style="text-align:center">${p.units}</td>
      <td style="text-align:right">$${p.revenue.toLocaleString()}</td>
    </tr>
  `).join('');

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        body { font-family: Arial, sans-serif; color: #1e293b; margin: 0; padding: 0; }
        .header { background: #2563eb; color: white; padding: 30px 40px; }
        .header h1 { font-size: 28px; margin: 0; }
        .header p { margin: 8px 0 0; opacity: 0.8; }
        .content { padding: 30px 40px; }
        .summary-grid {
          display: grid;
          grid-template-columns: repeat(4, 1fr);
          gap: 16px;
          margin-bottom: 30px;
        }
        .summary-card {
          background: #f8fafc;
          border: 1px solid #e2e8f0;
          border-radius: 8px;
          padding: 16px;
        }
        .summary-card .label { font-size: 12px; color: #64748b; text-transform: uppercase; }
        .summary-card .value { font-size: 24px; font-weight: bold; color: #2563eb; margin-top: 4px; }
        table { width: 100%; border-collapse: collapse; margin-top: 8px; }
        th { background: #1e40af; color: white; padding: 12px 16px; text-align: left; font-size: 13px; }
        td { padding: 12px 16px; border-bottom: 1px solid #e2e8f0; font-size: 14px; }
        tr:last-child td { border-bottom: none; }
        tr:nth-child(even) td { background: #f8fafc; }
        .section-title { font-size: 18px; font-weight: bold; margin-bottom: 12px; color: #1e293b; }
        .footer { margin-top: 40px; padding-top: 16px; border-top: 1px solid #e2e8f0;
                  font-size: 12px; color: #94a3b8; text-align: center; }
      </style>
    </head>
    <body>
      <div class="header">
        <h1>${data.title}</h1>
        <p>Period: ${data.period} &nbsp;|&nbsp; Generated: ${data.generatedAt}</p>
      </div>
      <div class="content">
        <div class="summary-grid">
          <div class="summary-card">
            <div class="label">Total Revenue</div>
            <div class="value">$${data.summary.totalRevenue.toLocaleString()}</div>
          </div>
          <div class="summary-card">
            <div class="label">Total Orders</div>
            <div class="value">${data.summary.totalOrders}</div>
          </div>
          <div class="summary-card">
            <div class="label">Avg Order Value</div>
            <div class="value">$${data.summary.avgOrderValue}</div>
          </div>
          <div class="summary-card">
            <div class="label">Growth</div>
            <div class="value" style="color:#16a34a">${data.summary.growth}</div>
          </div>
        </div>

        <div class="section-title">Top Products</div>
        <table>
          <thead>
            <tr>
              <th>Product</th>
              <th style="text-align:center">Units Sold</th>
              <th style="text-align:right">Revenue</th>
            </tr>
          </thead>
          <tbody>
            ${productRows}
          </tbody>
        </table>
        
        <div class="footer">
          Confidential — ${data.title} — ${data.period}
        </div>
      </div>
    </body>
    </html>
  `;
}

// --- GENERATE PDF ---
async function generateDataReport(data) {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });

  const page = await browser.newPage();
  const html = buildReportHTML(data);

  await page.setContent(html, { waitUntil: 'networkidle0' });

  const pdfBuffer = await page.pdf({
    format: 'A4',
    printBackground: true,
    margin: { top: '0', right: '0', bottom: '20mm', left: '0' }
  });

  await browser.close();
  fs.writeFileSync('./output/sales-report.pdf', pdfBuffer);
  console.log('✅ Report generated!');
  
  return pdfBuffer; // Return buffer for API streaming
}

generateDataReport(reportData);
```

---

## 4. Styling & Layout

### Page Size Configuration

```javascript
// Using named formats
await page.pdf({ format: 'A4' });       // 210mm × 297mm
await page.pdf({ format: 'Letter' });   // 8.5in × 11in
await page.pdf({ format: 'Legal' });    // 8.5in × 14in
await page.pdf({ format: 'A3' });       // 297mm × 420mm
await page.pdf({ format: 'Tabloid' });  // 11in × 17in

// Custom size
await page.pdf({ width: '200mm', height: '150mm' }); // Custom receipt size

// Landscape A4
await page.pdf({ format: 'A4', landscape: true });
```

### Margins

```javascript
// Uniform margins
margin: { top: '20mm', right: '20mm', bottom: '20mm', left: '20mm' }

// Different top/bottom for header/footer space
margin: { top: '30mm', right: '15mm', bottom: '25mm', left: '15mm' }

// Units supported: mm, cm, in, px (mm recommended for print)
```

### Fonts

```javascript
// ---- Option 1: System fonts (fastest, always available) ----
font-family: Arial, Helvetica, sans-serif;
font-family: 'Times New Roman', Times, serif;
font-family: 'Courier New', Courier, monospace;

// ---- Option 2: Google Fonts via CDN ----
// In HTML head (requires waitUntil: 'networkidle0')
`<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">`
font-family: 'Inter', sans-serif;

// ---- Option 3: Base64 embedded fonts (self-contained, no network) ----
// Convert font to base64: base64 -i font.ttf
const fontBase64 = fs.readFileSync('./fonts/Inter.ttf').toString('base64');
const html = `
<style>
  @font-face {
    font-family: 'Inter';
    src: url('data:font/truetype;base64,${fontBase64}') format('truetype');
    font-weight: normal;
  }
  body { font-family: 'Inter', sans-serif; }
</style>
`;
```

### Colors, Tables, Flexbox/Grid

```css
/* ===== COLORS ===== */
.primary    { color: #2563eb; }
.success    { color: #16a34a; }
.danger     { color: #dc2626; }
.muted      { color: #64748b; }
.bg-accent  { background: #eff6ff; }

/* ===== STYLED TABLE ===== */
table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}
thead th {
  background: #1e40af;
  color: white;
  padding: 10px 14px;
  text-align: left;
  font-weight: 600;
}
tbody td {
  padding: 10px 14px;
  border-bottom: 1px solid #e2e8f0;
}
tbody tr:nth-child(even) td {
  background: #f8fafc;
}
tbody tr:hover td {
  background: #eff6ff; /* Works in PDF rendering! */
}
tfoot td {
  font-weight: bold;
  border-top: 2px solid #2563eb;
  padding: 12px 14px;
}

/* ===== FLEXBOX IN PDF ===== */
.flex-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
}

/* ===== GRID IN PDF ===== */
.grid-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

/* Puppeteer fully supports Flexbox and Grid! */
```

### Example: Styled Table Report (Excel-Style Export)

```javascript
// src/generators/tableReport.js
async function generateTableReport(data) {
  const rows = data.rows.map((row, i) => `
    <tr>
      <td>${i + 1}</td>
      <td>${row.name}</td>
      <td>${row.department}</td>
      <td style="text-align:right">${row.sales.toLocaleString()}</td>
      <td style="text-align:center">
        <span class="badge badge-${row.status === 'Active' ? 'green' : 'gray'}">
          ${row.status}
        </span>
      </td>
      <td style="text-align:right">
        <span style="color:${row.growth > 0 ? '#16a34a' : '#dc2626'}">
          ${row.growth > 0 ? '▲' : '▼'} ${Math.abs(row.growth)}%
        </span>
      </td>
    </tr>
  `).join('');

  const total = data.rows.reduce((sum, r) => sum + r.sales, 0);

  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        body { font-family: Arial, sans-serif; margin: 0; color: #1e293b; font-size: 13px; }
        .header { padding: 24px 32px; border-bottom: 3px solid #2563eb; display: flex; 
                  justify-content: space-between; align-items: center; }
        .logo { font-size: 22px; font-weight: bold; color: #2563eb; }
        .meta { text-align: right; color: #64748b; font-size: 12px; }
        .content { padding: 24px 32px; }
        h2 { font-size: 18px; margin: 0 0 16px; }
        table { width: 100%; border-collapse: collapse; }
        th { background: #1e40af; color: white; padding: 10px 12px; text-align: left; }
        td { padding: 9px 12px; border-bottom: 1px solid #e2e8f0; }
        tr:nth-child(even) td { background: #f8fafc; }
        tfoot td { font-weight: bold; background: #eff6ff !important; 
                   border-top: 2px solid #2563eb; }
        .badge { padding: 2px 8px; border-radius: 999px; font-size: 11px; font-weight: 600; }
        .badge-green { background: #dcfce7; color: #166534; }
        .badge-gray  { background: #f1f5f9; color: #64748b; }
      </style>
    </head>
    <body>
      <div class="header">
        <div class="logo">📊 SalesForce Corp</div>
        <div class="meta">
          <div>${data.title}</div>
          <div>Export Date: ${new Date().toLocaleDateString()}</div>
        </div>
      </div>
      <div class="content">
        <h2>${data.title}</h2>
        <table>
          <thead>
            <tr>
              <th>#</th>
              <th>Name</th>
              <th>Department</th>
              <th style="text-align:right">Sales ($)</th>
              <th style="text-align:center">Status</th>
              <th style="text-align:right">Growth</th>
            </tr>
          </thead>
          <tbody>${rows}</tbody>
          <tfoot>
            <tr>
              <td colspan="3">Total</td>
              <td style="text-align:right">$${total.toLocaleString()}</td>
              <td colspan="2"></td>
            </tr>
          </tfoot>
        </table>
      </div>
    </body>
    </html>
  `;

  const browser = await puppeteer.launch({
    headless: 'new', args: ['--no-sandbox']
  });
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: 'networkidle0' });
  const buffer = await page.pdf({
    format: 'A4', landscape: true,
    printBackground: true,
    margin: { top: '0', right: '0', bottom: '0', left: '0' }
  });
  await browser.close();
  return buffer;
}
```

---

## 5. Dynamic Data Injection

### Template Literals (Simplest Approach)

```javascript
// Direct injection via template literals
const user = { name: 'Ravi Shankar', email: 'ravi@example.com', plan: 'Pro' };

const html = `
  <html>
    <body>
      <h1>Welcome, ${user.name}!</h1>
      <p>Email: ${user.email}</p>
      <p>Plan: ${user.plan}</p>
    </body>
  </html>
`;
```

### Handling Arrays and Complex Data

```javascript
// Map over arrays
const items = [
  { name: 'Item A', qty: 2, price: 50 },
  { name: 'Item B', qty: 1, price: 120 },
];

const tableRows = items.map(item => `
  <tr>
    <td>${item.name}</td>
    <td>${item.qty}</td>
    <td>$${(item.qty * item.price).toFixed(2)}</td>
  </tr>
`).join('');

// Conditional rendering
const statusBadge = (status) => status === 'paid'
  ? '<span style="color:green;font-weight:bold">✓ PAID</span>'
  : '<span style="color:red;font-weight:bold">✗ UNPAID</span>';
```

### Using Handlebars (Clean Separation)

```bash
npm install handlebars
```

```javascript
// src/templates/invoice.hbs
/*
<!DOCTYPE html>
<html>
<body>
  <h1>Invoice #{{invoice.number}}</h1>
  <p>Client: {{client.name}}</p>
  <table>
    {{#each items}}
    <tr>
      <td>{{name}}</td>
      <td>{{qty}}</td>
      <td>{{price}}</td>
    </tr>
    {{/each}}
  </table>
  {{#if isPaid}}
    <span class="paid-stamp">PAID</span>
  {{else}}
    <p>Due: {{invoice.dueDate}}</p>
  {{/if}}
</body>
</html>
*/

// src/generators/invoiceGenerator.js
const handlebars = require('handlebars');
const fs = require('fs');
const puppeteer = require('puppeteer');

async function generateInvoicePDF(data) {
  // Load and compile template
  const templateSource = fs.readFileSync('./src/templates/invoice.hbs', 'utf8');
  const template = handlebars.compile(templateSource);
  
  // Register helpers
  handlebars.registerHelper('currency', val => `$${val.toFixed(2)}`);
  handlebars.registerHelper('multiply', (a, b) => (a * b).toFixed(2));
  handlebars.registerHelper('formatDate', date => new Date(date).toLocaleDateString());
  
  // Inject data into template
  const html = template(data);
  
  // Generate PDF
  const browser = await puppeteer.launch({ headless: 'new', args: ['--no-sandbox'] });
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: 'networkidle0' });
  const buffer = await page.pdf({ format: 'A4', printBackground: true });
  await browser.close();
  
  return buffer;
}
```

### Using EJS

```bash
npm install ejs
```

```javascript
const ejs = require('ejs');

// Can use inline EJS or file
const template = `
  <html>
    <body>
      <h1><%= title %></h1>
      <ul>
        <% items.forEach(function(item) { %>
          <li><%= item.name %> — $<%= item.price %></li>
        <% }); %>
      </ul>
      <% if (total > 1000) { %>
        <p class="discount">💰 You qualify for a bulk discount!</p>
      <% } %>
    </body>
  </html>
`;

const html = ejs.render(template, { title: 'Order Summary', items, total });

// OR from file:
const html = await ejs.renderFile('./templates/invoice.ejs', data);
```

### Example: Survey Report PDF with Dynamic Q&A

```javascript
// Survey report with dynamic questions and answers
const surveyData = {
  title: 'Employee Satisfaction Survey 2025',
  respondent: 'Priya Mehta',
  submittedAt: '2025-01-15',
  score: 78,
  questions: [
    {
      question: 'How satisfied are you with your current role?',
      type: 'rating',
      answer: 4,
      maxRating: 5
    },
    {
      question: 'What aspects of work do you enjoy most?',
      type: 'text',
      answer: 'Collaboration with team members and solving complex problems.'
    },
    {
      question: 'Which benefits do you value most?',
      type: 'multiselect',
      answer: ['Health Insurance', 'Remote Work', 'Learning Budget']
    },
    {
      question: 'Would you recommend this company to a friend?',
      type: 'yesno',
      answer: 'Yes'
    }
  ]
};

function renderAnswer(q) {
  switch (q.type) {
    case 'rating':
      const stars = '★'.repeat(q.answer) + '☆'.repeat(q.maxRating - q.answer);
      return `<span style="color:#f59e0b;font-size:20px">${stars}</span>
              <span style="margin-left:8px;color:#64748b">${q.answer}/${q.maxRating}</span>`;
    
    case 'text':
      return `<p style="margin:0;padding:12px;background:#f8fafc;
              border-left:3px solid #2563eb;border-radius:4px;
              font-style:italic;color:#374151">${q.answer}</p>`;
    
    case 'multiselect':
      return q.answer.map(a =>
        `<span style="display:inline-block;margin:3px;padding:4px 12px;
         background:#eff6ff;color:#1d4ed8;border-radius:999px;
         font-size:12px;font-weight:600">${a}</span>`
      ).join('');
    
    case 'yesno':
      const color = q.answer === 'Yes' ? '#16a34a' : '#dc2626';
      return `<span style="color:${color};font-weight:bold;font-size:16px">
              ${q.answer === 'Yes' ? '✓' : '✗'} ${q.answer}</span>`;
    
    default:
      return `<span>${q.answer}</span>`;
  }
}

function buildSurveyHTML(data) {
  const qBlocks = data.questions.map((q, i) => `
    <div style="margin-bottom:24px;padding:20px;background:white;
         border:1px solid #e2e8f0;border-radius:8px;
         page-break-inside:avoid">
      <div style="font-size:12px;color:#94a3b8;margin-bottom:6px">
        Question ${i + 1}
      </div>
      <div style="font-weight:600;margin-bottom:12px;color:#1e293b">
        ${q.question}
      </div>
      <div>${renderAnswer(q)}</div>
    </div>
  `).join('');

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        * { box-sizing: border-box; }
        body { font-family: Arial, sans-serif; margin: 0; background: #f1f5f9; }
        .header { background: linear-gradient(135deg, #1e40af, #2563eb); color: white; padding: 40px; }
        .header h1 { font-size: 26px; margin: 0 0 8px; }
        .meta { display: flex; gap: 24px; margin-top: 16px; font-size: 13px; opacity: 0.85; }
        .score-badge { background: white; color: #1e40af; padding: 4px 16px;
                       border-radius: 999px; font-weight: bold; font-size: 18px; }
        .content { padding: 32px 40px; }
      </style>
    </head>
    <body>
      <div class="header">
        <h1>📋 ${data.title}</h1>
        <div class="meta">
          <span>👤 ${data.respondent}</span>
          <span>📅 ${data.submittedAt}</span>
          <span>Score: <span class="score-badge">${data.score}/100</span></span>
        </div>
      </div>
      <div class="content">
        <h2 style="font-size:18px;margin:0 0 20px;color:#1e293b">Responses</h2>
        ${qBlocks}
      </div>
    </body>
    </html>
  `;
}
```

---

## 6. Advanced PDF Features

### Headers & Footers

Headers and footers have **critical restrictions** you must know:

1. They run in a **separate rendering context** — they cannot access styles from your main HTML
2. They **must use inline styles only**
3. Special template variables are available via class names
4. Margins must be set to accommodate them

```javascript
await page.pdf({
  format: 'A4',
  displayHeaderFooter: true,
  
  // Header template (inline styles mandatory)
  headerTemplate: `
    <div style="
      width: 100%;
      padding: 8px 40px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-family: Arial, sans-serif;
      font-size: 10px;
      color: #64748b;
      border-bottom: 1px solid #e2e8f0;
    ">
      <span style="font-weight: bold; color: #1e40af;">📊 Monthly Report</span>
      <span>Generated: ${new Date().toLocaleDateString()}</span>
    </div>
  `,

  // Footer template with page numbers
  footerTemplate: `
    <div style="
      width: 100%;
      padding: 8px 40px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-family: Arial, sans-serif;
      font-size: 10px;
      color: #94a3b8;
      border-top: 1px solid #e2e8f0;
    ">
      <span>Confidential — Internal Use Only</span>
      <span>Page <span class="pageNumber"></span> of <span class="totalPages"></span></span>
    </div>
  `,

  // Must increase margins to accommodate header/footer
  margin: {
    top: '25mm',     // Space for header
    right: '15mm',
    bottom: '20mm',  // Space for footer
    left: '15mm'
  }
});
```

**Available template classes:**

| Class | Content |
|-------|---------|
| `<span class="pageNumber">` | Current page number |
| `<span class="totalPages">` | Total page count |
| `<span class="url">` | Page URL |
| `<span class="title">` | Document title |
| `<span class="date">` | Formatted print date |

### Page Breaks

```css
/* CSS for controlling page breaks */

/* Force a page break after element */
.page-break-after  { page-break-after: always; }

/* Force a page break before element */
.page-break-before { page-break-before: always; }

/* Prevent breaking inside element (keep table rows together) */
.no-break { page-break-inside: avoid; }

/* Modern equivalents (use both for compatibility) */
.page-break-after {
  page-break-after: always;
  break-after: page;
}
.no-break {
  page-break-inside: avoid;
  break-inside: avoid;
}
```

```html
<!-- In HTML structure -->
<div class="section">
  <!-- Section 1 content -->
</div>

<div class="page-break-after"></div>  <!-- Force new page -->

<div class="section">
  <!-- Section 2 content on new page -->
</div>

<!-- Tables that shouldn't split mid-row -->
<table>
  <tr style="page-break-inside: avoid">
    <td>This row</td>
    <td>stays together</td>
  </tr>
</table>
```

### Multi-Page Documents with Headers/Footers

```javascript
// src/generators/multiPageReport.js
const puppeteer = require('puppeteer');

async function generateMultiPageReport(sections) {
  const sectionHTML = sections.map((section, i) => `
    <div class="${i > 0 ? 'page-break' : ''}">
      <h2 class="section-title">${section.title}</h2>
      ${section.content}
    </div>
  `).join('');

  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: Arial, sans-serif; font-size: 14px; color: #1e293b; }
        
        @page { size: A4; }
        
        .page-break { page-break-before: always; break-before: page; padding-top: 8px; }
        .avoid-break { page-break-inside: avoid; break-inside: avoid; }
        
        .section-title {
          font-size: 22px; font-weight: bold;
          color: #1e40af; margin-bottom: 16px;
          padding-bottom: 8px;
          border-bottom: 2px solid #2563eb;
        }
        
        .card {
          background: #f8fafc;
          border: 1px solid #e2e8f0;
          border-radius: 8px;
          padding: 20px;
          margin-bottom: 16px;
        }
        
        /* Ensure tables don't break mid-row */
        tr { page-break-inside: avoid; break-inside: avoid; }
        
        /* Cover page */
        .cover {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          min-height: 245mm; /* A4 height minus margins */
          text-align: center;
        }
        .cover h1 { font-size: 36px; color: #1e40af; margin-bottom: 16px; }
        .cover .subtitle { font-size: 18px; color: #64748b; }
        .cover .date { font-size: 14px; color: #94a3b8; margin-top: 32px; }
      </style>
    </head>
    <body>
      <!-- Cover Page -->
      <div class="cover">
        <div style="font-size:48px;margin-bottom:24px">📑</div>
        <h1>Annual Performance Report</h1>
        <p class="subtitle">Comprehensive Analysis · FY 2024–2025</p>
        <p class="date">Prepared on ${new Date().toLocaleDateString()}</p>
      </div>

      <!-- Content Sections -->
      ${sectionHTML}
    </body>
    </html>
  `;

  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: 'networkidle0' });

  const buffer = await page.pdf({
    format: 'A4',
    printBackground: true,
    displayHeaderFooter: true,
    headerTemplate: `
      <div style="width:100%;padding:6px 40px;font-family:Arial,sans-serif;
           font-size:9px;color:#94a3b8;border-bottom:1px solid #e2e8f0;
           display:flex;justify-content:space-between">
        <span style="font-weight:bold;color:#2563eb">Annual Performance Report</span>
        <span>FY 2024–2025 · Confidential</span>
      </div>
    `,
    footerTemplate: `
      <div style="width:100%;padding:6px 40px;font-family:Arial,sans-serif;
           font-size:9px;color:#94a3b8;border-top:1px solid #e2e8f0;
           display:flex;justify-content:space-between">
        <span>© 2025 Your Company Name</span>
        <span>Page <span class="pageNumber"></span> / <span class="totalPages"></span></span>
      </div>
    `,
    margin: { top: '20mm', right: '15mm', bottom: '20mm', left: '15mm' }
  });

  await browser.close();
  return buffer;
}
```

---

## 7. Generating Complex Documents

### Invoice PDF — Complete Production Example

```javascript
// src/generators/invoice.js
const puppeteer = require('puppeteer');

const invoiceData = {
  invoice: {
    number: 'INV-2025-0042',
    date: '2025-01-15',
    dueDate: '2025-02-15',
    status: 'Unpaid'
  },
  company: {
    name: 'TechSolutions Pvt. Ltd.',
    address: '42 MG Road, Indore, MP 452001',
    phone: '+91 98765 43210',
    email: 'billing@techsolutions.in',
    gstin: '23AABCT1234A1Z5'
  },
  client: {
    name: 'Startup Hub Inc.',
    address: '15 Innovation Avenue, Mumbai, MH 400001',
    email: 'accounts@startuphub.com',
    gstin: '27AABCS5555B1ZA'
  },
  items: [
    { description: 'Website Development (React + Node.js)', qty: 1, rate: 85000 },
    { description: 'UI/UX Design (10 screens)', qty: 1, rate: 25000 },
    { description: 'API Integration & Testing', qty: 20, rate: 1500 },
    { description: 'Deployment & DevOps Setup', qty: 1, rate: 12000 }
  ],
  tax: { gst: 18 },
  notes: 'Payment due within 30 days. Late payment attracts 2% monthly interest.'
};

function buildInvoiceHTML(data) {
  const subtotal = data.items.reduce((sum, item) => sum + (item.qty * item.rate), 0);
  const taxAmount = subtotal * (data.tax.gst / 100);
  const total = subtotal + taxAmount;

  const itemRows = data.items.map((item, i) => `
    <tr>
      <td style="padding:10px 14px;border-bottom:1px solid #e2e8f0">${i + 1}</td>
      <td style="padding:10px 14px;border-bottom:1px solid #e2e8f0">${item.description}</td>
      <td style="padding:10px 14px;border-bottom:1px solid #e2e8f0;text-align:center">${item.qty}</td>
      <td style="padding:10px 14px;border-bottom:1px solid #e2e8f0;text-align:right">
        ₹${item.rate.toLocaleString('en-IN')}
      </td>
      <td style="padding:10px 14px;border-bottom:1px solid #e2e8f0;text-align:right;font-weight:600">
        ₹${(item.qty * item.rate).toLocaleString('en-IN')}
      </td>
    </tr>
  `).join('');

  const isPaid = data.invoice.status === 'Paid';

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: Arial, sans-serif; color: #1e293b; font-size: 13px; background: white; }
        
        .invoice-wrapper { max-width: 800px; margin: 0 auto; padding: 40px; }
        
        /* Header */
        .invoice-header {
          display: flex;
          justify-content: space-between;
          align-items: flex-start;
          margin-bottom: 40px;
          padding-bottom: 24px;
          border-bottom: 2px solid #2563eb;
        }
        .company-name { font-size: 24px; font-weight: bold; color: #1e40af; }
        .company-details { font-size: 12px; color: #64748b; margin-top: 6px; line-height: 1.6; }
        .invoice-meta { text-align: right; }
        .invoice-title { font-size: 32px; font-weight: bold; color: #94a3b8; letter-spacing: 2px; }
        .invoice-number { font-size: 14px; color: #2563eb; font-weight: 600; margin-top: 4px; }
        .status-badge {
          display: inline-block;
          margin-top: 8px;
          padding: 4px 16px;
          border-radius: 999px;
          font-size: 12px;
          font-weight: bold;
          background: ${isPaid ? '#dcfce7' : '#fef3c7'};
          color: ${isPaid ? '#166534' : '#92400e'};
        }
        
        /* Billing info */
        .billing-section {
          display: flex;
          gap: 40px;
          margin-bottom: 32px;
        }
        .billing-block { flex: 1; }
        .billing-label { font-size: 11px; font-weight: bold; color: #94a3b8; 
                         text-transform: uppercase; letter-spacing: 1px; margin-bottom: 8px; }
        .billing-name { font-size: 15px; font-weight: 600; margin-bottom: 4px; }
        .billing-details { font-size: 12px; color: #64748b; line-height: 1.6; }

        /* Dates row */
        .dates-row {
          display: flex;
          gap: 24px;
          margin-bottom: 32px;
          padding: 16px;
          background: #f8fafc;
          border-radius: 8px;
        }
        .date-item { flex: 1; }
        .date-label { font-size: 11px; color: #94a3b8; text-transform: uppercase; 
                      letter-spacing: 1px; margin-bottom: 4px; }
        .date-value { font-size: 14px; font-weight: 600; }
        
        /* Items table */
        table { width: 100%; border-collapse: collapse; margin-bottom: 24px; }
        thead th {
          background: #1e40af; color: white;
          padding: 10px 14px; text-align: left;
          font-size: 12px; font-weight: 600;
        }
        
        /* Totals */
        .totals-section {
          display: flex;
          justify-content: flex-end;
        }
        .totals-table { min-width: 300px; }
        .totals-row {
          display: flex;
          justify-content: space-between;
          padding: 8px 0;
          border-bottom: 1px solid #e2e8f0;
          font-size: 13px;
        }
        .totals-row.total {
          font-size: 16px; font-weight: bold;
          color: #1e40af; border-top: 2px solid #2563eb;
          border-bottom: none; padding-top: 12px; margin-top: 4px;
        }
        
        /* Watermark for paid invoices */
        ${isPaid ? `
        .watermark {
          position: fixed;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%) rotate(-35deg);
          font-size: 120px;
          font-weight: bold;
          color: rgba(22, 163, 74, 0.08);
          z-index: 0;
          pointer-events: none;
          white-space: nowrap;
        }
        ` : ''}
        
        .notes { margin-top: 32px; padding: 16px; background: #fffbeb;
                 border: 1px solid #fbbf24; border-radius: 8px; }
        .notes-title { font-weight: bold; color: #92400e; margin-bottom: 6px; }
        .notes-text { font-size: 12px; color: #78350f; }
        
        .footer { margin-top: 40px; padding-top: 16px; border-top: 1px solid #e2e8f0;
                  text-align: center; font-size: 11px; color: #94a3b8; }
      </style>
    </head>
    <body>
      <div class="invoice-wrapper">
        ${isPaid ? '<div class="watermark">PAID</div>' : ''}
        
        <div class="invoice-header">
          <div>
            <div class="company-name">${data.company.name}</div>
            <div class="company-details">
              ${data.company.address}<br>
              📞 ${data.company.phone} · ✉️ ${data.company.email}<br>
              GSTIN: ${data.company.gstin}
            </div>
          </div>
          <div class="invoice-meta">
            <div class="invoice-title">INVOICE</div>
            <div class="invoice-number">${data.invoice.number}</div>
            <div class="status-badge">${data.invoice.status.toUpperCase()}</div>
          </div>
        </div>
        
        <div class="billing-section">
          <div class="billing-block">
            <div class="billing-label">Bill From</div>
            <div class="billing-name">${data.company.name}</div>
            <div class="billing-details">${data.company.address}<br>GSTIN: ${data.company.gstin}</div>
          </div>
          <div class="billing-block">
            <div class="billing-label">Bill To</div>
            <div class="billing-name">${data.client.name}</div>
            <div class="billing-details">${data.client.address}<br>GSTIN: ${data.client.gstin}</div>
          </div>
        </div>
        
        <div class="dates-row">
          <div class="date-item">
            <div class="date-label">Invoice Date</div>
            <div class="date-value">${data.invoice.date}</div>
          </div>
          <div class="date-item">
            <div class="date-label">Due Date</div>
            <div class="date-value" style="color:#dc2626">${data.invoice.dueDate}</div>
          </div>
          <div class="date-item">
            <div class="date-label">Payment Terms</div>
            <div class="date-value">Net 30</div>
          </div>
        </div>
        
        <table>
          <thead>
            <tr>
              <th style="width:40px">#</th>
              <th>Description</th>
              <th style="width:60px;text-align:center">Qty</th>
              <th style="width:110px;text-align:right">Rate</th>
              <th style="width:110px;text-align:right">Amount</th>
            </tr>
          </thead>
          <tbody>${itemRows}</tbody>
        </table>
        
        <div class="totals-section">
          <div class="totals-table">
            <div class="totals-row">
              <span>Subtotal</span>
              <span>₹${subtotal.toLocaleString('en-IN')}</span>
            </div>
            <div class="totals-row">
              <span>GST (${data.tax.gst}%)</span>
              <span>₹${taxAmount.toLocaleString('en-IN', { maximumFractionDigits: 2 })}</span>
            </div>
            <div class="totals-row total">
              <span>Total Due</span>
              <span>₹${total.toLocaleString('en-IN', { maximumFractionDigits: 2 })}</span>
            </div>
          </div>
        </div>
        
        <div class="notes">
          <div class="notes-title">📝 Notes</div>
          <div class="notes-text">${data.notes}</div>
        </div>
        
        <div class="footer">
          Thank you for your business! · ${data.company.name} · ${data.company.email}
        </div>
      </div>
    </body>
    </html>
  `;
}

async function generateInvoice(data) {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });
  const page = await browser.newPage();
  await page.setContent(buildInvoiceHTML(data), { waitUntil: 'networkidle0' });
  const buffer = await page.pdf({
    format: 'A4',
    printBackground: true,
    margin: { top: '0', right: '0', bottom: '0', left: '0' }
  });
  await browser.close();
  return buffer;
}

module.exports = { generateInvoice, buildInvoiceHTML };
```

---

## 8. Charts in PDF

### Using Chart.js Inside HTML → PDF

This is a two-step process:
1. Render Chart.js inside the HTML with `<canvas>`
2. Wait for chart to render before taking PDF

```javascript
// src/generators/dashboardReport.js
const puppeteer = require('puppeteer');

const analyticsData = {
  monthly: [42000, 58000, 51000, 73000, 67000, 84320],
  labels: ['Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Jan'],
  userGrowth: [120, 180, 210, 290, 340, 420],
  categories: { 'Pro Plan': 45, 'Starter': 35, 'Enterprise': 20 }
};

function buildDashboardHTML(data) {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: Arial, sans-serif; background: #f1f5f9; color: #1e293b; }
        
        .header {
          background: linear-gradient(135deg, #1e40af 0%, #2563eb 100%);
          color: white;
          padding: 32px 40px;
        }
        .header h1 { font-size: 26px; margin-bottom: 4px; }
        .header p { opacity: 0.8; font-size: 14px; }
        
        .dashboard { padding: 24px 32px; }
        
        .kpi-grid {
          display: grid;
          grid-template-columns: repeat(4, 1fr);
          gap: 16px;
          margin-bottom: 24px;
        }
        .kpi-card {
          background: white;
          border-radius: 10px;
          padding: 20px;
          border-left: 4px solid;
        }
        .kpi-card.blue   { border-color: #2563eb; }
        .kpi-card.green  { border-color: #16a34a; }
        .kpi-card.orange { border-color: #ea580c; }
        .kpi-card.purple { border-color: #7c3aed; }
        .kpi-label { font-size: 11px; color: #94a3b8; text-transform: uppercase; 
                     letter-spacing: 1px; margin-bottom: 8px; }
        .kpi-value { font-size: 26px; font-weight: bold; }
        .kpi-change { font-size: 12px; margin-top: 4px; }
        .up   { color: #16a34a; }
        .down { color: #dc2626; }
        
        .charts-grid {
          display: grid;
          grid-template-columns: 2fr 1fr;
          gap: 16px;
          margin-bottom: 24px;
        }
        .chart-card {
          background: white;
          border-radius: 10px;
          padding: 20px;
        }
        .chart-title { font-size: 14px; font-weight: 600; margin-bottom: 16px; color: #374151; }
        canvas { max-width: 100%; }
      </style>
    </head>
    <body>
      <div class="header">
        <h1>📊 Analytics Dashboard — January 2025</h1>
        <p>Performance overview · Generated ${new Date().toLocaleString()}</p>
      </div>
      
      <div class="dashboard">
        <div class="kpi-grid">
          <div class="kpi-card blue">
            <div class="kpi-label">Monthly Revenue</div>
            <div class="kpi-value">$84,320</div>
            <div class="kpi-change up">▲ 12.4% vs last month</div>
          </div>
          <div class="kpi-card green">
            <div class="kpi-label">Active Users</div>
            <div class="kpi-value">420</div>
            <div class="kpi-change up">▲ 23.5% growth</div>
          </div>
          <div class="kpi-card orange">
            <div class="kpi-label">Conversion Rate</div>
            <div class="kpi-value">3.8%</div>
            <div class="kpi-change down">▼ 0.2% decrease</div>
          </div>
          <div class="kpi-card purple">
            <div class="kpi-label">Avg Session</div>
            <div class="kpi-value">4m 32s</div>
            <div class="kpi-change up">▲ 8% increase</div>
          </div>
        </div>
        
        <div class="charts-grid">
          <div class="chart-card">
            <div class="chart-title">Revenue Trend (6 Months)</div>
            <canvas id="revenueChart" height="200"></canvas>
          </div>
          <div class="chart-card">
            <div class="chart-title">Plan Distribution</div>
            <canvas id="pieChart" height="200"></canvas>
          </div>
        </div>
        
        <div class="chart-card">
          <div class="chart-title">User Growth</div>
          <canvas id="userChart" height="120"></canvas>
        </div>
      </div>

      <script>
        // Revenue line chart
        new Chart(document.getElementById('revenueChart'), {
          type: 'bar',
          data: {
            labels: ${JSON.stringify(data.labels)},
            datasets: [{
              label: 'Revenue ($)',
              data: ${JSON.stringify(data.monthly)},
              backgroundColor: 'rgba(37, 99, 235, 0.15)',
              borderColor: '#2563eb',
              borderWidth: 2,
              borderRadius: 4,
              fill: true
            }]
          },
          options: {
            responsive: true,
            animation: false,  // IMPORTANT: Disable animation for PDF
            plugins: { legend: { display: false } },
            scales: {
              y: { beginAtZero: true, grid: { color: 'rgba(0,0,0,0.05)' } },
              x: { grid: { display: false } }
            }
          }
        });
        
        // Pie chart
        new Chart(document.getElementById('pieChart'), {
          type: 'doughnut',
          data: {
            labels: ${JSON.stringify(Object.keys(data.categories))},
            datasets: [{
              data: ${JSON.stringify(Object.values(data.categories))},
              backgroundColor: ['#2563eb', '#7c3aed', '#ea580c'],
              borderWidth: 0
            }]
          },
          options: {
            animation: false,
            plugins: {
              legend: { position: 'bottom', labels: { padding: 16, font: { size: 11 } } }
            }
          }
        });
        
        // User growth line chart
        new Chart(document.getElementById('userChart'), {
          type: 'line',
          data: {
            labels: ${JSON.stringify(data.labels)},
            datasets: [{
              label: 'Active Users',
              data: ${JSON.stringify(data.userGrowth)},
              borderColor: '#16a34a',
              backgroundColor: 'rgba(22, 163, 74, 0.1)',
              tension: 0.4,
              fill: true,
              pointRadius: 4
            }]
          },
          options: {
            animation: false,
            responsive: true,
            plugins: { legend: { display: false } },
            scales: {
              y: { beginAtZero: true },
              x: { grid: { display: false } }
            }
          }
        });
        
        // Signal that charts are rendered
        window.chartsReady = true;
      </script>
    </body>
    </html>
  `;
}

async function generateDashboardPDF(data) {
  const browser = await puppeteer.launch({
    headless: 'new',
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });
  const page = await browser.newPage();
  
  await page.setContent(buildDashboardHTML(data), { waitUntil: 'networkidle0' });
  
  // CRITICAL: Wait for Chart.js to finish rendering
  await page.waitForFunction('window.chartsReady === true');
  await page.waitForTimeout(500); // Extra buffer for rendering
  
  const buffer = await page.pdf({
    format: 'A4',
    landscape: true,           // Dashboard fits better in landscape
    printBackground: true,
    margin: { top: '0', right: '0', bottom: '0', left: '0' }
  });
  
  await browser.close();
  return buffer;
}

module.exports = { generateDashboardPDF };
```

> **Key insight:** Set `animation: false` on all Chart.js charts. Without this, charts may be mid-animation when the PDF snapshot is taken, resulting in partial or blank charts.

---

## 9. puppeteer vs puppeteer-core

### When to Use Each

```
puppeteer         → Development, simple projects
puppeteer-core    → Production VPS, Docker, custom Chromium path
```

### Using puppeteer-core with System Chromium

```bash
# Install puppeteer-core (no bundled Chromium)
npm install puppeteer-core

# Install system Chromium on Ubuntu
sudo apt-get install -y chromium-browser
# OR
sudo apt-get install -y chromium

# Find its path
which chromium-browser   # → /usr/bin/chromium-browser
which chromium           # → /usr/bin/chromium
```

```javascript
// src/browser.js
const puppeteer = require('puppeteer-core');

// Common Chromium paths by OS
const CHROMIUM_PATHS = {
  linux: [
    '/usr/bin/chromium-browser',
    '/usr/bin/chromium',
    '/usr/bin/google-chrome',
    '/usr/bin/google-chrome-stable',
    '/snap/bin/chromium'
  ],
  darwin: [
    '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
    '/Applications/Chromium.app/Contents/MacOS/Chromium'
  ]
};

const fs = require('fs');

function findChromium() {
  const platform = process.platform;
  const paths = CHROMIUM_PATHS[platform === 'darwin' ? 'darwin' : 'linux'];
  
  for (const p of paths) {
    if (fs.existsSync(p)) return p;
  }
  
  throw new Error('Chromium not found! Install with: sudo apt-get install chromium-browser');
}

async function getBrowser() {
  return puppeteer.launch({
    headless: 'new',
    executablePath: process.env.CHROMIUM_PATH || findChromium(),
    args: [
      '--no-sandbox',
      '--disable-setuid-sandbox',
      '--disable-dev-shm-usage',   // Prevent /dev/shm issues in Docker
      '--disable-gpu',
      '--no-zygote'
    ]
  });
}

module.exports = { getBrowser };
```

### Bundle Size Comparison

```
puppeteer:      ~200MB (includes Chromium download)
puppeteer-core: ~10MB  (just the API, no browser)
```

For Docker deployments, `puppeteer-core` + system Chromium is significantly cleaner.

---

## 10. VPS / Production Deployment

### Setting Up on Ubuntu VPS

#### Step 1: Install Required System Dependencies

```bash
# Update package list
sudo apt-get update

# Install Chromium and all its shared library dependencies
sudo apt-get install -y \
  chromium-browser \
  ca-certificates \
  fonts-liberation \
  libappindicator3-1 \
  libasound2 \
  libatk-bridge2.0-0 \
  libatk1.0-0 \
  libc6 \
  libcairo2 \
  libcups2 \
  libdbus-1-3 \
  libexpat1 \
  libfontconfig1 \
  libgbm1 \
  libgcc1 \
  libglib2.0-0 \
  libgtk-3-0 \
  libnspr4 \
  libnss3 \
  libpango-1.0-0 \
  libpangocairo-1.0-0 \
  libstdc++6 \
  libx11-6 \
  libx11-xcb1 \
  libxcb1 \
  libxcomposite1 \
  libxcursor1 \
  libxdamage1 \
  libxext6 \
  libxfixes3 \
  libxi6 \
  libxrandr2 \
  libxrender1 \
  libxss1 \
  libxtst6 \
  lsb-release \
  wget \
  xdg-utils

# Install Node.js (v18+)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installations
node --version        # v20.x.x
chromium-browser --version  # Chromium 120.x.x

# Install fonts for Hindi/Arabic/CJK support
sudo apt-get install -y \
  fonts-noto \
  fonts-noto-cjk \
  fonts-noto-color-emoji \
  fonts-freefont-ttf
```

#### Step 2: Project Setup on VPS

```bash
# Clone or upload your project
git clone https://github.com/yourname/pdf-service.git
cd pdf-service

npm install  # Uses puppeteer-core, no Chromium download

# Create output directory
mkdir -p output

# Set environment variable for Chromium path
echo "CHROMIUM_PATH=/usr/bin/chromium-browser" >> .env
```

#### Step 3: Production Server with Express

```javascript
// server.js — Production Express server
require('dotenv').config();
const express = require('express');
const { getBrowser } = require('./src/browser');
const { generateInvoice } = require('./src/generators/invoice');
const { generateDashboardPDF } = require('./src/generators/dashboardReport');

const app = express();
app.use(express.json({ limit: '10mb' }));

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime() });
});

// Generate invoice PDF
app.post('/api/pdf/invoice', async (req, res) => {
  try {
    const buffer = await generateInvoice(req.body);
    
    res.set({
      'Content-Type': 'application/pdf',
      'Content-Disposition': `attachment; filename="invoice-${req.body.invoice?.number || 'doc'}.pdf"`,
      'Content-Length': buffer.length
    });
    
    res.send(buffer);
  } catch (err) {
    console.error('Invoice generation failed:', err);
    res.status(500).json({ error: 'PDF generation failed', message: err.message });
  }
});

// Generate dashboard PDF
app.post('/api/pdf/dashboard', async (req, res) => {
  try {
    const buffer = await generateDashboardPDF(req.body);
    res.set({
      'Content-Type': 'application/pdf',
      'Content-Disposition': 'attachment; filename="dashboard.pdf"'
    });
    res.send(buffer);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`🚀 PDF Service running on port ${PORT}`));
```

#### Step 4: Set Up PM2 for Process Management

```bash
# Install PM2 globally
sudo npm install -g pm2

# Start your app
pm2 start server.js --name "pdf-service" \
  --instances 2 \          # Run 2 instances (adjust per CPU cores)
  --max-memory-restart 500M  # Restart if memory exceeds 500MB

# Save PM2 config
pm2 save

# Auto-start on system reboot
pm2 startup
# Run the command PM2 prints

# Monitor
pm2 monit
pm2 logs pdf-service
pm2 status
```

#### Step 5: PM2 Ecosystem File (Recommended)

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'pdf-service',
    script: 'server.js',
    instances: 2,
    exec_mode: 'cluster',
    watch: false,
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
      CHROMIUM_PATH: '/usr/bin/chromium-browser'
    },
    error_file: './logs/pm2-error.log',
    out_file: './logs/pm2-out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
};

// pm2 start ecosystem.config.js
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:20-slim

# Install Chromium and dependencies
RUN apt-get update && apt-get install -y \
  chromium \
  ca-certificates \
  fonts-liberation \
  libappindicator3-1 \
  libasound2 \
  libatk-bridge2.0-0 \
  libatk1.0-0 \
  libcups2 \
  libdbus-1-3 \
  libgbm1 \
  libgtk-3-0 \
  libnss3 \
  libxcomposite1 \
  libxdamage1 \
  libxrandr2 \
  fonts-noto \
  --no-install-recommends \
  && rm -rf /var/lib/apt/lists/*

# Set Chromium path
ENV CHROMIUM_PATH=/usr/bin/chromium

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

RUN mkdir -p output logs

EXPOSE 3000
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  pdf-service:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - CHROMIUM_PATH=/usr/bin/chromium
    restart: unless-stopped
    mem_limit: 1g
    volumes:
      - ./output:/app/output
      - ./logs:/app/logs
```

```bash
# Build and run
docker-compose up -d

# Test
curl -X POST http://localhost:3000/health
```

### Common VPS Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Error: Failed to launch the browser process` | Missing deps | Run the apt-get install command above |
| `Running as root without --no-sandbox` | Root user, no sandbox | Add `--no-sandbox` to args |
| `Error: spawn /usr/bin/chromium ENOENT` | Wrong path | Run `which chromium-browser` and update path |
| `libgbm.so.1: cannot open` | Missing libgbm1 | `sudo apt-get install libgbm1` |
| `/dev/shm` too small (Docker) | Default /dev/shm is 64MB | Add `--disable-dev-shm-usage` to args |
| `TimeoutError: Navigation Timeout` | Slow server | Increase timeout: `page.setDefaultTimeout(60000)` |
| Blank PDF | Page not loaded | Use `waitUntil: 'networkidle0'` |

---

## 11. Performance Optimization

### The Problem: Don't Launch a Browser Per Request

```javascript
// ❌ BAD — Launches and kills Chromium for every request
// 300ms+ overhead per request, huge memory spikes
app.post('/pdf', async (req, res) => {
  const browser = await puppeteer.launch(...);  // Slow!
  const page = await browser.newPage();
  // ... generate pdf ...
  await browser.close();  // Wasteful
});
```

### Solution: Singleton Browser Instance

```javascript
// src/browserManager.js — Production-grade browser pool
const puppeteer = require('puppeteer-core');

class BrowserManager {
  constructor() {
    this.browser = null;
    this.launching = false;
    this.activePagesCount = 0;
  }

  async getBrowser() {
    if (this.browser && this.browser.isConnected()) {
      return this.browser;
    }
    
    if (this.launching) {
      // Wait for ongoing launch
      await new Promise(resolve => setTimeout(resolve, 100));
      return this.getBrowser();
    }
    
    this.launching = true;
    try {
      this.browser = await puppeteer.launch({
        headless: 'new',
        executablePath: process.env.CHROMIUM_PATH,
        args: [
          '--no-sandbox',
          '--disable-setuid-sandbox',
          '--disable-dev-shm-usage',
          '--disable-gpu',
          '--no-zygote'
        ]
      });

      // Auto-restart if browser crashes
      this.browser.on('disconnected', () => {
        console.log('Browser disconnected — will relaunch on next request');
        this.browser = null;
      });

      console.log('✅ Browser launched');
      return this.browser;
    } finally {
      this.launching = false;
    }
  }

  async generatePDF(htmlContent, options = {}) {
    const browser = await this.getBrowser();
    const page = await browser.newPage();
    this.activePagesCount++;
    
    try {
      // Optimize: Block unnecessary resources for pure HTML templates
      await page.setRequestInterception(true);
      page.on('request', (request) => {
        const resourceType = request.resourceType();
        // Block images, media (unless your PDF uses them)
        if (['image', 'media', 'font'].includes(resourceType) && 
            !request.url().includes('data:')) {
          request.abort();
        } else {
          request.continue();
        }
      });

      await page.setContent(htmlContent, { waitUntil: 'networkidle0' });
      
      const buffer = await page.pdf({
        format: 'A4',
        printBackground: true,
        ...options
      });
      
      return buffer;
    } finally {
      this.activePagesCount--;
      await page.close(); // Always close the page, not the browser
    }
  }
}

// Export singleton
module.exports = new BrowserManager();
```

```javascript
// Usage in routes
const browserManager = require('./src/browserManager');

app.post('/api/pdf/invoice', async (req, res) => {
  const html = buildInvoiceHTML(req.body);
  const buffer = await browserManager.generatePDF(html, { format: 'A4' });
  res.type('pdf').send(buffer);
});
```

### Concurrency Handling with Queue

```javascript
// For high traffic: use a queue to limit concurrent Chromium pages
const PQueue = require('p-queue'); // npm install p-queue

const pdfQueue = new PQueue({ concurrency: 3 }); // Max 3 PDFs at once

app.post('/api/pdf', (req, res) => {
  pdfQueue.add(async () => {
    const buffer = await browserManager.generatePDF(html);
    res.type('pdf').send(buffer);
  });
});
```

### Memory Management

```javascript
// Avoid memory leaks
// ✅ Always close pages after use
const page = await browser.newPage();
try {
  // ... work ...
} finally {
  await page.close(); // In finally block — always runs
}

// ✅ Set reasonable timeouts
page.setDefaultTimeout(30000); // 30 seconds max

// ✅ Monitor memory in PM2
pm2 monit

// ✅ Schedule browser restart every N hours
setInterval(async () => {
  if (browserManager.browser && browserManager.activePagesCount === 0) {
    await browserManager.browser.close();
    browserManager.browser = null;
    console.log('🔄 Scheduled browser restart complete');
  }
}, 3 * 60 * 60 * 1000); // Every 3 hours
```

---

## 12. File Handling

### Save PDF to Disk

```javascript
const fs = require('fs');
const path = require('path');

async function savePDF(data, filename) {
  const buffer = await generatePDF(data);
  const outputPath = path.join('./output', filename);
  fs.writeFileSync(outputPath, buffer);
  console.log(`Saved: ${outputPath}`);
  return outputPath;
}
```

### Stream PDF in API Response (Download)

```javascript
// Express route — triggers browser download
app.post('/api/pdf/invoice', async (req, res) => {
  try {
    const buffer = await generateInvoice(req.body);
    
    res.set({
      'Content-Type': 'application/pdf',
      'Content-Disposition': 'attachment; filename="invoice.pdf"',  // Forces download
      'Content-Length': buffer.length,
      'Cache-Control': 'no-cache'
    });
    
    res.end(buffer); // Send buffer directly
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

### Open PDF in Browser (Inline)

```javascript
// Change 'attachment' to 'inline' — opens in browser PDF viewer
res.set({
  'Content-Type': 'application/pdf',
  'Content-Disposition': 'inline; filename="report.pdf"'  // Inline view
});
```

### Return as Base64 (for Frontend/Mobile)

```javascript
app.post('/api/pdf/base64', async (req, res) => {
  const buffer = await generatePDF(req.body);
  const base64 = buffer.toString('base64');
  
  res.json({
    success: true,
    pdf: base64,
    filename: 'document.pdf',
    mimeType: 'application/pdf'
  });
});

// Frontend usage:
// const link = document.createElement('a');
// link.href = `data:application/pdf;base64,${response.pdf}`;
// link.download = response.filename;
// link.click();
```

### Save to Cloud (S3 Example)

```javascript
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

const s3 = new S3Client({ region: 'ap-south-1' });

async function uploadPDFToS3(buffer, key) {
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET,
    Key: key,
    Body: buffer,
    ContentType: 'application/pdf',
    ContentDisposition: 'inline'
  }));
  
  return `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`;
}
```

---

## 13. Common Errors & Fixes

### Error: Chromium Not Found

```
Error: Failed to launch the browser process!
/usr/bin/chromium: not found
```

```bash
# Fix 1: Install Chromium
sudo apt-get install -y chromium-browser

# Fix 2: Find correct path
which chromium
which chromium-browser
ls /usr/bin/chrom*

# Fix 3: Set path explicitly in code
executablePath: '/usr/bin/chromium-browser'

# Fix 4: Use CHROMIUM_PATH env variable
export CHROMIUM_PATH=$(which chromium-browser)
```

### Error: Font Issues / Missing Characters

```
Symptoms: □□□ squares instead of characters, or fonts not matching
```

```bash
# Fix: Install fonts on server
sudo apt-get install -y \
  fonts-noto fonts-noto-cjk fonts-noto-color-emoji \
  fonts-liberation fonts-freefont-ttf

# Refresh font cache
sudo fc-cache -fv
```

```javascript
// Fix in code: Use web-safe fonts or embed fonts as base64
// ❌ Don't rely on system fonts that may not exist
font-family: 'Poppins';  // May not be installed

// ✅ Use fallback stacks
font-family: 'Poppins', Arial, Helvetica, sans-serif;

// ✅ Or load from Google Fonts (with networkidle0)
<link href="https://fonts.googleapis.com/css2?family=Inter&display=swap" rel="stylesheet">
```

### Error: Blank PDF

```
File generates but is empty or shows blank pages
```

```javascript
// Fix 1: Add printBackground: true
await page.pdf({ printBackground: true }); // ← Critical!

// Fix 2: Wait for content to load
await page.setContent(html, { waitUntil: 'networkidle0' }); // ← Not just 'load'

// Fix 3: Wait for specific element to appear
await page.waitForSelector('.content', { timeout: 10000 });

// Fix 4: Debug — screenshot to see what Chromium sees
await page.screenshot({ path: 'debug.png', fullPage: true });
// Then inspect debug.png to see the rendered state

// Fix 5: For Chart.js or JS-rendered content
await page.waitForFunction('window.chartsReady === true');
await page.waitForTimeout(300);
```

### Error: Layout Breaking in PDF

```
PDF looks fine in browser but layout is broken in PDF
```

```css
/* Fix 1: Don't use vh/vw units in PDF */
/* ❌ Bad */
height: 100vh;
/* ✅ Good */
height: 297mm; /* or min-height for content sections */

/* Fix 2: Avoid position:fixed (only works on first page) */
/* Use header/footer template instead */

/* Fix 3: Use explicit widths on grid/flex children */
.grid { display: grid; grid-template-columns: repeat(3, 1fr); }
.grid > * { min-width: 0; } /* Prevents overflow */

/* Fix 4: Add @page rule */
@page { size: A4; margin: 0; }
```

### Error: Running as Root (Docker/VPS)

```
Running as root without --no-sandbox is not supported
```

```javascript
// Fix: Always include these args
args: [
  '--no-sandbox',
  '--disable-setuid-sandbox',
  '--disable-dev-shm-usage'  // Also add this for Docker
]
```

---

## 14. Best Practices

### 1. Separate Data, Template, and Generator

```
src/
├── data/           # Data fetching (DB queries, API calls)
│   └── invoiceData.js
├── templates/      # HTML template functions (data → HTML string)
│   └── invoiceTemplate.js
├── generators/     # PDF generation (HTML → Buffer)
│   └── invoiceGenerator.js
└── routes/         # HTTP routes (request → response)
    └── pdfRoutes.js
```

```javascript
// ✅ Clean separation example

// templates/invoiceTemplate.js — ONLY builds HTML
function invoiceTemplate(data) {
  return `<html>...</html>`;
}

// generators/invoiceGenerator.js — ONLY handles Puppeteer
async function generateInvoicePDF(html) {
  const browser = await getBrowser();
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: 'networkidle0' });
  const buffer = await page.pdf({ format: 'A4', printBackground: true });
  await page.close();
  return buffer;
}

// routes/pdfRoutes.js — Orchestrates
router.post('/invoice', async (req, res) => {
  const data = await fetchInvoiceData(req.body.id); // Fetch from DB
  const html = invoiceTemplate(data);               // Build HTML
  const pdf = await generateInvoicePDF(html);       // Generate PDF
  res.type('pdf').send(pdf);                        // Send to client
});
```

### 2. Reusable PDF Components

```javascript
// src/templates/components.js

// Reusable page header
const pageHeader = (title, subtitle, date = new Date()) => `
  <div style="display:flex;justify-content:space-between;align-items:center;
       padding:0 0 20px;border-bottom:2px solid #2563eb;margin-bottom:24px">
    <div>
      <h1 style="font-size:24px;color:#1e40af;margin:0">${title}</h1>
      ${subtitle ? `<p style="color:#64748b;margin:4px 0 0;font-size:13px">${subtitle}</p>` : ''}
    </div>
    <div style="text-align:right;font-size:12px;color:#94a3b8">
      <div>Generated</div>
      <div style="font-weight:600;color:#374151">${date.toLocaleDateString()}</div>
    </div>
  </div>
`;

// Reusable stat card
const statCard = (label, value, change, color = '#2563eb') => `
  <div style="background:white;border:1px solid #e2e8f0;border-radius:8px;
       padding:16px;border-left:4px solid ${color}">
    <div style="font-size:11px;color:#94a3b8;text-transform:uppercase;
         letter-spacing:1px;margin-bottom:6px">${label}</div>
    <div style="font-size:24px;font-weight:bold;color:${color}">${value}</div>
    ${change ? `<div style="font-size:12px;margin-top:4px">${change}</div>` : ''}
  </div>
`;

// Reusable table
const dataTable = (headers, rows) => `
  <table style="width:100%;border-collapse:collapse">
    <thead>
      <tr>${headers.map(h => 
        `<th style="background:#1e40af;color:white;padding:10px 14px;
               text-align:left;font-size:12px">${h}</th>`
      ).join('')}</tr>
    </thead>
    <tbody>
      ${rows.map((row, i) => `
        <tr>
          ${row.map(cell => 
            `<td style="padding:10px 14px;border-bottom:1px solid #e2e8f0;
                  ${i % 2 === 1 ? 'background:#f8fafc' : ''}">${cell}</td>`
          ).join('')}
        </tr>
      `).join('')}
    </tbody>
  </table>
`;

module.exports = { pageHeader, statCard, dataTable };
```

### 3. Environment Configuration

```javascript
// config.js
module.exports = {
  chromium: {
    path: process.env.CHROMIUM_PATH || '/usr/bin/chromium-browser',
    args: [
      '--no-sandbox',
      '--disable-setuid-sandbox',
      '--disable-dev-shm-usage',
      '--disable-gpu',
      '--no-zygote',
      '--disable-extensions'
    ]
  },
  pdf: {
    defaultFormat: 'A4',
    defaultMargin: { top: '20mm', right: '15mm', bottom: '20mm', left: '15mm' },
    timeout: 30000
  }
};
```

---

## 15. Practice Exercises

Work through these in order. Each builds on the previous.

---

### Exercise 1: Hello World PDF with Styling

**Task:** Generate a styled "Welcome Letter" PDF.

**Requirements:**
- Company logo (text-based)
- Recipient name and date
- Body text with 3 paragraphs
- Signature section

**Solution Starter:**

```javascript
// exercises/ex1-welcome-letter.js
const puppeteer = require('puppeteer');
const fs = require('fs');

const letterData = {
  recipient: 'Rahul Gupta',
  date: new Date().toLocaleDateString('en-IN'),
  company: 'Acme Technologies',
  role: 'Senior Developer'
};

async function generateWelcomeLetter(data) {
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        body { font-family: 'Times New Roman', serif; max-width: 700px; 
               margin: 0 auto; padding: 50px; color: #1e293b; }
        .logo { font-size: 28px; font-weight: bold; color: #2563eb; margin-bottom: 40px; }
        .date { text-align: right; color: #64748b; margin-bottom: 40px; }
        h2 { color: #1e40af; }
        p { line-height: 1.8; margin-bottom: 16px; }
        .signature { margin-top: 60px; }
        .sig-line { border-top: 1px solid #1e293b; width: 200px; margin-top: 50px; padding-top: 8px; }
      </style>
    </head>
    <body>
      <div class="logo">🚀 ${data.company}</div>
      <div class="date">${data.date}</div>
      <p>Dear <strong>${data.recipient}</strong>,</p>
      <h2>Welcome to ${data.company}!</h2>
      <p>We are thrilled to welcome you as our new <strong>${data.role}</strong>.
         Your skills and experience will be a tremendous asset to our team.</p>
      <p>Please report to our office on your first day at 9:00 AM. 
         The HR team will guide you through the onboarding process.</p>
      <p>We look forward to your contributions and wish you a successful journey with us.</p>
      <div class="signature">
        <p>Warm regards,</p>
        <div class="sig-line">
          <strong>HR Department</strong><br>
          <span style="color:#64748b;font-size:13px">${data.company}</span>
        </div>
      </div>
    </body>
    </html>
  `;
  
  const browser = await puppeteer.launch({ headless: 'new', args: ['--no-sandbox'] });
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: 'networkidle0' });
  const buffer = await page.pdf({ format: 'A4', printBackground: true,
    margin: { top: '20mm', right: '20mm', bottom: '20mm', left: '20mm' }
  });
  await browser.close();
  
  fs.mkdirSync('./output', { recursive: true });
  fs.writeFileSync('./output/welcome-letter.pdf', buffer);
  console.log('✅ Welcome letter generated!');
}

generateWelcomeLetter(letterData);
```

---

### Exercise 2: Invoice PDF

**Task:** Build a complete invoice PDF.

**Requirements:**
- Company and client info
- Line items with qty, rate, amount
- Subtotal, tax, total calculation
- Payment instructions
- PAID/UNPAID watermark

> **Solution:** Full working code provided in [Section 7](#7-generating-complex-documents). Extend it by:
> - Adding bank details section
> - Adding QR code (use `qrcode` npm package, generate as base64 image)

---

### Exercise 3: Survey Results PDF

**Task:** Generate a PDF from survey responses with various question types.

**Requirements:**
- Rating questions (star display)
- Text responses (quote block)
- Multiple choice (chips/tags)
- Overall score bar
- One page per respondent (page breaks)

```javascript
// exercises/ex3-survey-results.js
// Generate multi-page survey results
const surveyResponses = [
  {
    respondent: 'Ananya Sharma',
    score: 82,
    questions: [
      { q: 'Rate your experience', type: 'rating', answer: 4, max: 5 },
      { q: 'What did you like most?', type: 'text', answer: 'The team was very responsive and helpful.' },
      { q: 'Select services used', type: 'multi', answer: ['API', 'Dashboard', 'Reports'] }
    ]
  },
  {
    respondent: 'Vikram Patel',
    score: 65,
    questions: [
      { q: 'Rate your experience', type: 'rating', answer: 3, max: 5 },
      { q: 'What did you like most?', type: 'text', answer: 'Good documentation.' },
      { q: 'Select services used', type: 'multi', answer: ['API', 'Webhooks'] }
    ]
  }
];

function buildSurveyResultsHTML(responses) {
  const pages = responses.map((resp, i) => `
    <div class="${i > 0 ? 'page-break' : ''}">
      <div class="resp-header">
        <div>
          <div class="resp-name">${resp.respondent}</div>
          <div class="score-bar-wrap">
            <div class="score-bar" style="width:${resp.score}%"></div>
          </div>
          <div class="score-label">Score: ${resp.score}/100</div>
        </div>
      </div>
      ${resp.questions.map((q, qi) => `
        <div class="q-block">
          <div class="q-num">Q${qi + 1}</div>
          <div class="q-text">${q.q}</div>
          <div class="q-answer">
            ${q.type === 'rating' ?
              '★'.repeat(q.answer) + '☆'.repeat(q.max - q.answer) :
              q.type === 'multi' ?
              q.answer.map(a => `<span class="chip">${a}</span>`).join('') :
              `<div class="quote">${q.answer}</div>`
            }
          </div>
        </div>
      `).join('')}
    </div>
  `).join('');

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <style>
        * { box-sizing: border-box; }
        body { font-family: Arial, sans-serif; color: #1e293b; padding: 40px; }
        .page-break { page-break-before: always; padding-top: 40px; }
        .resp-header { background: #eff6ff; border-radius: 10px; 
                       padding: 20px 24px; margin-bottom: 24px; }
        .resp-name { font-size: 20px; font-weight: bold; margin-bottom: 10px; }
        .score-bar-wrap { height: 8px; background: #dbeafe; border-radius: 4px; 
                          width: 300px; overflow: hidden; }
        .score-bar { height: 100%; background: #2563eb; border-radius: 4px; }
        .score-label { margin-top: 6px; font-size: 13px; color: #64748b; }
        .q-block { margin-bottom: 20px; padding: 16px; border: 1px solid #e2e8f0; border-radius: 8px; }
        .q-num { font-size: 11px; color: #94a3b8; margin-bottom: 4px; }
        .q-text { font-weight: 600; margin-bottom: 10px; }
        .chip { display:inline-block; margin:2px; padding:3px 12px; 
                background:#eff6ff; color:#1d4ed8; border-radius:999px; font-size:12px; }
        .quote { padding:10px 14px; border-left:3px solid #2563eb; 
                 background:#f8fafc; font-style:italic; border-radius:4px; }
      </style>
    </head>
    <body>
      <h1 style="margin-bottom:32px">📋 Survey Results — ${new Date().toLocaleDateString()}</h1>
      ${pages}
    </body>
    </html>
  `;
}
```

---

### Exercise 4: Analytics Dashboard PDF

**Task:** Create a multi-section analytics report.

**Requirements:**
- Executive summary (KPIs)
- Revenue trend chart (Chart.js)
- Top 10 items table
- Geographic breakdown
- Page numbers in footer

> **Solution:** Combine the dashboard HTML from [Section 8](#8-charts-in-pdf) with the multi-page structure from [Section 6](#6-advanced-pdf-features).

**Extension Challenge:**
- Add a Table of Contents on page 1
- Use `class="pageNumber"` in footer
- Add conditional color coding (red for negative metrics, green for positive)

---

### Exercise 5: Multi-Page Annual Report

**Task:** Generate a formal 5-page annual report.

**Requirements:**
- Page 1: Cover page with company branding
- Page 2: Executive summary + KPI cards
- Page 3: Financial data table (12 months)
- Page 4: Charts (revenue, growth, breakdown)
- Page 5: Outlook and closing

**Structure:**

```javascript
// exercises/ex5-annual-report.js
const sections = [
  {
    title: 'Executive Summary',
    content: buildExecutiveSummary(data.summary)
  },
  {
    title: 'Financial Performance',
    content: buildFinancialTable(data.monthly)
  },
  {
    title: 'Visual Analytics',
    content: buildChartsSection(data.charts)
  },
  {
    title: 'Outlook 2026',
    content: buildOutlookSection(data.outlook)
  }
];

await generateMultiPageReport(sections); // From Section 6
```

---

### Exercise 6: User Activity Report

**Task:** Generate a per-user activity PDF for a SaaS platform.

**Requirements:**
- User profile card (avatar placeholder, name, email, plan)
- 30-day login heatmap (HTML table simulating a grid)
- Feature usage breakdown
- Usage compared to plan limits
- Recommendations section

```javascript
// Heatmap using HTML table
function buildLoginHeatmap(logins) {
  // logins = array of 30 booleans (true if logged in that day)
  const days = ['Mon','Tue','Wed','Thu','Fri','Sat','Sun'];
  const weeks = [];
  
  for (let i = 0; i < 30; i += 7) {
    weeks.push(logins.slice(i, i + 7));
  }
  
  return `
    <table style="border-collapse:separate;border-spacing:4px">
      <tr>
        ${days.map(d => `<th style="font-size:10px;color:#94a3b8;width:28px">${d}</th>`).join('')}
      </tr>
      ${weeks.map(week => `
        <tr>
          ${week.map(active => `
            <td style="width:28px;height:28px;border-radius:4px;
                background:${active ? '#2563eb' : '#e2e8f0'}"></td>
          `).join('')}
        </tr>
      `).join('')}
    </table>
  `;
}
```

---

## Quick Reference Card

```javascript
// ===== PUPPETEER CHEAT SHEET =====

// Launch options
const browser = await puppeteer.launch({
  headless: 'new',
  executablePath: process.env.CHROMIUM_PATH,  // For puppeteer-core
  args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage']
});

// Set content
await page.setContent(html, { waitUntil: 'networkidle0' });

// PDF options
const buffer = await page.pdf({
  path: 'file.pdf',          // Omit for Buffer
  format: 'A4',              // 'A4' | 'Letter' | 'Legal'
  landscape: false,
  printBackground: true,     // ALWAYS true
  margin: { top: '20mm', right: '15mm', bottom: '20mm', left: '15mm' },
  displayHeaderFooter: true,
  headerTemplate: `<div style="..."><span class="pageNumber"></span></div>`,
  footerTemplate: `<div style="...">Page <span class="pageNumber"></span></div>`
});

// Always close
await page.close();   // Close page (reuse browser)
await browser.close(); // Close everything

// HTTP Response
res.set('Content-Type', 'application/pdf');
res.set('Content-Disposition', 'attachment; filename="doc.pdf"');
res.send(buffer);

// CSS for page breaks
.page-break  { page-break-before: always; }
.no-break    { page-break-inside: avoid; }
@page        { size: A4; margin: 15mm; }
```

---

*This guide covers Puppeteer v22+ and Node.js 18+. Always check the [official Puppeteer docs](https://pptr.dev) for the latest API changes.*

*Happy generating! 🚀*
