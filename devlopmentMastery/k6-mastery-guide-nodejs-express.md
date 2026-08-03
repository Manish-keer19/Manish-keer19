# k6 Mastery Guide — Load & Performance Testing for Node.js + Express APIs

*From absolute beginner to production-ready performance engineer.*

---

## Table of Contents

1. [Introduction to Performance Testing](#1-introduction-to-performance-testing)
2. [What is k6?](#2-what-is-k6)
3. [Installing k6](#3-installing-k6)
4. [Your First k6 Script](#4-your-first-k6-script)
5. [JavaScript Essentials for k6](#5-javascript-essentials-for-k6)
6. [The HTTP Module](#6-the-http-module)
7. [Checks and Assertions](#7-checks-and-assertions)
8. [Params: Headers, Cookies, Tags](#8-params-headers-cookies-tags)
9. [Test Lifecycle: init, setup, default, teardown](#9-test-lifecycle-init-setup-default-teardown)
10. [Scenarios and Executors](#10-scenarios-and-executors)
11. [Metrics Deep Dive](#11-metrics-deep-dive)
12. [Thresholds, SLAs, SLOs](#12-thresholds-slas-slos)
13. [Options Reference](#13-options-reference)
14. [Load Testing Real REST APIs](#14-load-testing-real-rest-apis)
15. [Testing a Real Express Backend](#15-testing-a-real-express-backend)
16. [Reading and Interpreting Reports](#16-reading-and-interpreting-reports)
17. [HTML, JSON, CSV, JUnit Reports](#17-html-json-csv-junit-reports)
18. [Grafana, InfluxDB, Prometheus Integration](#18-grafana-influxdb-prometheus-integration)
19. [Real Performance Investigation Playbook](#19-real-performance-investigation-playbook)
20. [Advanced k6](#20-advanced-k6)
21. [CI/CD Integration](#21-cicd-integration)
22. [Running k6 in Docker](#22-running-k6-in-docker)
23. [Testing Production Safely](#23-testing-production-safely)
24. [Performance Engineering Best Practices](#24-performance-engineering-best-practices)
25. [Professional Project Structure](#25-professional-project-structure)
26. [Complete Project: E-commerce API](#26-complete-project-e-commerce-api)
27. [50 Common Mistakes](#27-50-common-mistakes)
28. [Interview Questions](#28-interview-questions)
29. [Cheat Sheet](#29-cheat-sheet)
30. [Glossary](#30-glossary)

---

## 1. Introduction to Performance Testing

### What is Performance Testing?

Performance testing answers one question: **"How does the system behave under a given amount of work?"** It measures speed, stability, scalability, and resource usage, rather than whether a feature produces the correct output (that's functional testing).

### Why It Matters

- A feature that "works" but takes 8 seconds under 50 users is a production incident waiting to happen.
- Performance bugs are usually invisible in local development (1 user, empty database, warm cache) and only appear at scale.
- Outages cost money, trust, and SEO ranking. Amazon famously found every 100ms of latency cost them ~1% in sales.

### Testing Types Compared

```
FUNCTIONAL AXIS                     PERFORMANCE AXIS
────────────────────                ─────────────────────
Unit Testing                        Load Testing
  └─ one function, isolated           └─ expected concurrent users
Integration Testing                 Stress Testing
  └─ modules working together        └─ push past capacity, find breaking point
API Testing                         Spike Testing
  └─ correctness of endpoints         └─ sudden traffic burst
                                     Soak / Endurance Testing
                                       └─ sustained load over hours/days (leaks)
                                     Volume Testing
                                       └─ large data volumes (huge payloads/DB rows)
                                     Capacity Testing
                                       └─ max load system can handle at acceptable SLA
                                     Scalability Testing
                                       └─ how throughput changes as resources are added
```

| Type | Question it answers | Typical k6 pattern |
|---|---|---|
| Load Testing | Can we handle expected peak traffic? | `ramping-vus` to target concurrency |
| Stress Testing | Where does it break, and how does it fail? | `ramping-vus` beyond expected peak |
| Spike Testing | Can we survive a sudden 10x burst? | sharp `stages` spike then drop |
| Soak/Endurance | Do we leak memory/connections over time? | `constant-vus` for hours |
| Volume Testing | Does it hold up with large payloads/datasets? | large body/file uploads, big DB |
| Capacity Testing | What's our safe maximum throughput? | `ramping-arrival-rate` until thresholds fail |
| Scalability Testing | Does throughput grow linearly with resources? | same test, repeated per instance count |

### Where k6 Fits

k6 is a **developer-centric, code-first load testing tool**. Unlike JMeter (XML/GUI-heavy) or LoadRunner (enterprise, expensive), k6 tests are JavaScript files that live in your repo, run in CI, and are reviewed like code. It's built for engineers who want load testing to be a first-class part of the software lifecycle, not a separate QA silo.

### Best Practices
- Decide *which* type of test you need before writing scripts — a load test and a stress test have different goals and different pass/fail criteria.
- Always test against an environment that resembles production (data volume, network, hardware).

### Common Mistakes
- Treating "it loaded once in the browser" as proof of performance.
- Running performance tests only right before a big launch, instead of continuously.

### Interview Questions
**Q: What's the difference between load testing and stress testing?**
A: Load testing validates behavior at expected/peak traffic; stress testing intentionally exceeds capacity to find the breaking point and observe failure behavior.

**Q: Why is soak testing important even if load tests pass?**
A: Load tests run for minutes; memory leaks, connection pool exhaustion, and disk-fill issues often only appear after hours of sustained traffic.

### Exercises
1. List three types of tests suited to an e-commerce checkout flow and explain why.
2. Write down what "acceptable performance" means for your current API (P95 target, error rate target).

---

## 2. What is k6?

### History and Architecture

k6 (by Grafana Labs, originally Load Impact) is an open-source load testing tool written in **Go**, with a **JavaScript scripting API**. Go gives it low overhead per virtual user (goroutines instead of OS threads/processes), which lets a single machine simulate far more concurrent users than tools like JMeter.

```
┌─────────────────────────────────────────────────────────┐
│                        k6 Binary (Go)                    │
│                                                           │
│   ┌───────────────┐        ┌─────────────────────────┐  │
│   │  JS Runtime    │  runs  │   Your Test Script (.js) │  │
│   │ (goja engine)  │◄───────┤  export default function │  │
│   └───────┬───────┘        └─────────────────────────┘  │
│           │ spawns                                       │
│   ┌───────▼─────────────────────────────────────────┐   │
│   │  Virtual Users (VUs) — lightweight goroutines     │   │
│   │  VU1  VU2  VU3  VU4  ...  VU_n                    │   │
│   └───────┬───────────────────────────────────────────┘  │
│           │ HTTP/WS/gRPC requests                        │
└───────────┼────────────────────────────────────────────┘
            ▼
     Your Node.js / Express API
            │
     ┌──────▼──────┐
     │  Metrics     │──► stdout summary / JSON / InfluxDB / Prometheus
     │  Collector   │
     └─────────────┘
```

### Core Concepts

- **Virtual User (VU):** a simulated client executing your script in a loop. Each VU is independent state (its own cookies, unless shared deliberately).
- **Iteration:** one full run through the `default function` by one VU.
- **Scenario:** a named configuration describing *how* VUs and iterations are scheduled over time (an executor + timing).
- **Executor:** the algorithm controlling VU/iteration scheduling (e.g., ramp up VUs, or hold a constant request rate).
- **Metrics:** built-in and custom measurements collected per request/iteration (duration, failure rate, counts).
- **Thresholds:** pass/fail conditions applied to metrics, e.g. "95% of requests must be under 200ms."

### Best Practices
- Think in **scenarios**, not just "vus + duration" — scenarios scale to complex, realistic traffic shapes.
- Understand that k6 is a **load generator**; the bottleneck you're finding is almost always on the *server* or *network*, not k6 itself (though k6 itself can become the bottleneck if under-resourced — see Ch. 19).

### Common Mistakes
- Confusing "iterations" with "requests" — one iteration can make many HTTP requests.
- Assuming 1 VU = 1 real user in every sense; VUs execute a script in a loop, not a full browser session (no rendering, no JS execution on the page, no image loading unless your script does it).

### Interview Questions
**Q: Why is k6 written in Go but scripted in JavaScript?**
A: Go provides efficient concurrency (goroutines) for generating high load with low resource usage; JavaScript (via the embedded goja engine) gives testers a familiar, expressive scripting language without needing a JVM or heavy runtime.

**Q: What is the difference between a VU and an iteration?**
A: A VU is a simulated concurrent user/worker; an iteration is one complete execution of the test function by a VU. A single VU can perform many iterations over a test's duration.

### Exercises
1. Draw (in ASCII or ASCII-style notes) how 10 VUs running for 30s with `constant-vus` differ from 10 VUs with `ramping-vus`.

---

## 3. Installing k6

### macOS (Homebrew)
```bash
brew install k6
```

### Windows (Chocolatey)
```powershell
choco install k6
```

### Windows (Scoop)
```powershell
scoop install k6
```

### Windows (Winget)
```powershell
winget install k6 --source winget
```

### Linux (Debian/Ubuntu)
```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6ACFD8
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

### Linux (Fedora/CentOS)
```bash
sudo dnf install https://dl.k6.io/rpm/repo.rpm
sudo dnf install k6
```

### Docker (all platforms)
```bash
docker pull grafana/k6
docker run --rm -i grafana/k6 run - <script.js
```

### Verify Installation
```bash
k6 version
```

### Version Management
- Pin the k6 version in CI (`grafana/k6:0.52.0` Docker tag, or a fixed package version) so a k6 upgrade doesn't silently change threshold behavior.
- Check the [official changelog](https://github.com/grafana/k6/releases) before upgrading — executor defaults and metric names have changed across major versions.

### Best Practices
- Use Docker in CI for reproducibility across machines.
- Keep your local k6 version in sync with CI to avoid "works on my machine" script errors.

### Common Mistakes
- Installing via `npm install k6` — **there is no such package**; k6 is a Go binary, not distributed via npm. (There *is* an unrelated `k6` npm package with a different purpose — don't confuse it.)
- Forgetting to update PATH after a manual binary install on Linux.

### Interview Questions
**Q: Can you install k6 via npm?**
A: No. k6 is a compiled Go binary; it's installed via OS package managers, Docker, or a downloaded binary — not npm.

### Exercises
1. Install k6 using the method appropriate for your OS and run `k6 version`.
2. Pull the official Docker image and run a version check inside a container.

---

## 4. Your First k6 Script

```javascript
// script.js
import http from 'k6/http';   // k6's built-in HTTP client
import { sleep } from 'k6';   // pause between iterations

export default function () {
  // Sends a single GET request and records metrics for it
  http.get('https://test-api.example.com/health');

  // Pause 1 second before this VU starts its next iteration
  sleep(1);
}
```

Run it:
```bash
k6 run script.js
```

### Line-by-Line Explanation

- `import http from 'k6/http';` — k6 ships built-in modules (`k6/http`, `k6/ws`, `k6/crypto`, etc.) that are only available inside the k6 runtime, not regular Node.js `require`.
- `export default function () { ... }` — this is the **VU code**, executed repeatedly, once per iteration, by every virtual user. Everything outside this function (top-level code) runs once **per VU during initialization**, not per iteration.
- `http.get(url)` — performs a real HTTP GET request and automatically records metrics (`http_req_duration`, `http_req_failed`, etc.) tagged to that request.
- `sleep(1)` — simulates "think time"; without it, VUs hammer the server back-to-back with zero pause, which rarely reflects real user behavior.

### What Happens When You Run It

```
k6 run script.js
        │
        ▼
1. init stage runs once (imports, top-level vars)
2. VUs spun up per your options (default: 1 VU, 1 iteration)
3. default function executes → HTTP GET fired → response recorded
4. summary printed to stdout
```

### Best Practices
- Always add realistic `sleep()` calls — a script with zero sleep is a stress test, not a load test, unless that's your intent.
- Keep the default function focused on one logical "user journey"; split unrelated tests into separate files.

### Common Mistakes
- Putting `require()` (Node.js syntax) instead of `import` — k6 uses ES module syntax, not CommonJS.
- Forgetting `sleep()`, causing artificially extreme (and unrealistic) throughput numbers.
- Running `k6 run script.js` and being confused it only sends **one** request — that's correct: without `options`, k6 defaults to 1 VU and 1 iteration.

### Interview Questions
**Q: Why does code outside the default function only run once per VU?**
A: That's the k6 **init context** — it's meant for imports and setup that should be prepared once when a VU is initialized, not repeated needlessly on every iteration, which would waste CPU and skew timing metrics.

### Exercises
1. Modify the script to hit two different endpoints in one iteration.
2. Add a `sleep()` of a random duration between 1–3 seconds to simulate variable think time.

---

## 5. JavaScript Essentials for k6

k6 supports modern ES2015+ syntax via the goja engine (not all of ES2020+ — check compatibility for newer features like optional chaining in older k6 versions).

```javascript
// Variables
const BASE_URL = 'http://localhost:3000';
let counter = 0;

// Arrow functions
const buildUrl = (path) => `${BASE_URL}${path}`;

// Template literals
const endpoint = `${BASE_URL}/api/users/${42}`;

// Objects & destructuring
const user = { id: 1, name: 'Asha', role: 'admin' };
const { id, name } = user;

// Arrays & iteration
const ids = [1, 2, 3];
ids.forEach((n) => console.log(n));
const doubled = ids.map((n) => n * 2);

// Conditions
if (user.role === 'admin') {
  console.log('admin user');
} else {
  console.log('regular user');
}

// Functions
function buildHeaders(token) {
  return { Authorization: `Bearer ${token}`, 'Content-Type': 'application/json' };
}

// JSON
const payload = JSON.stringify({ email: 'a@b.com', password: 'secret' });
const parsed = JSON.parse('{"ok": true}');

// Custom modules — split reusable logic into files
// utils/auth.js
export function login(baseUrl, creds) { /* ... */ }
// script.js
import { login } from './utils/auth.js';
```

### What's Different From Node.js
| Feature | Node.js | k6 |
|---|---|---|
| Module system | CommonJS (`require`) or ESM | ESM only (`import`/`export`) |
| `fetch` | Available (modern Node) | Not native — use `k6/http` |
| `npm install` | Full ecosystem | Only via `k6 build` (Go extensions) or bundling with tools like webpack |
| File system access | `fs` module | Not available inside VU code (sandboxed) |
| `setTimeout`/async I/O | Event loop based | k6 scripts are synchronous per iteration; no real event loop for your test logic |

### Best Practices
- Keep helper functions in separate files under `helpers/` or `utils/` and import them — mirrors good Express project structure.
- Use `SharedArray` (Ch. 20) instead of loading large JSON arrays directly at the top level, to avoid duplicating memory per VU.

### Common Mistakes
- Trying to `require()` npm packages directly — most won't work unless bundled and k6-compatible.
- Using blocking `while(true)` loops without `sleep()`, freezing a VU indefinitely.

### Interview Questions
**Q: Why can't you just `npm install axios` and use it in a k6 script?**
A: k6 scripts run inside a custom Go-embedded JS runtime (goja), not Node.js — there's no Node `require`, no filesystem/network Node APIs, and most npm packages assume a Node or browser environment. k6 provides its own `k6/http` module instead.

### Exercises
1. Write a helper function `randomEmail()` that generates a unique email string, and import it into a test script.

---

## 6. The HTTP Module

### All Methods
```javascript
import http from 'k6/http';
import { check } from 'k6';

const BASE = 'http://localhost:3000/api';

export default function () {
  http.get(`${BASE}/products`);

  http.post(`${BASE}/products`, JSON.stringify({ name: 'Shoe', price: 49.99 }), {
    headers: { 'Content-Type': 'application/json' },
  });

  http.put(`${BASE}/products/1`, JSON.stringify({ price: 39.99 }), {
    headers: { 'Content-Type': 'application/json' },
  });

  http.patch(`${BASE}/products/1`, JSON.stringify({ price: 35.0 }), {
    headers: { 'Content-Type': 'application/json' },
  });

  http.del(`${BASE}/products/1`);

  http.head(`${BASE}/products`);

  http.options(`${BASE}/products`);
}
```

### Multipart Upload
```javascript
import http from 'k6/http';
import { open } from 'k6/experimental/fs'; // or read once in init for older versions

const fileData = open('./sample.png', 'b'); // binary

export default function () {
  const data = {
    file: http.file(fileData, 'sample.png', 'image/png'),
    description: 'test upload',
  };
  http.post('http://localhost:3000/api/upload', data);
}
```

### Headers, Cookies, Auth
```javascript
// Static headers
const params = { headers: { 'X-Api-Key': 'abc123' } };
http.get('http://localhost:3000/api/secure', params);

// Basic Auth
http.get('http://user:pass@localhost:3000/api/secure');

// Bearer / JWT Auth
function authHeaders(token) {
  return { headers: { Authorization: `Bearer ${token}` } };
}
const res = http.post('http://localhost:3000/api/login', JSON.stringify({
  email: 'user@test.com', password: 'pass123',
}), { headers: { 'Content-Type': 'application/json' } });

const token = res.json('token');
http.get('http://localhost:3000/api/profile', authHeaders(token));

// Cookies — automatically stored per-VU cookie jar
const jar = http.cookieJar();
jar.set('http://localhost:3000', 'session_id', 'abc123');
```

### Timeouts, Redirects, Compression
```javascript
http.get('http://localhost:3000/api/slow', {
  timeout: '10s',            // fail if no response in 10s
  redirects: 0,               // don't auto-follow redirects
});

http.get('http://localhost:3000/api/data', {
  headers: { 'Accept-Encoding': 'gzip' }, // request compressed response
});
```

### Connection Reuse & HTTP/2
k6 reuses TCP connections (keep-alive) by default within a VU, mimicking real browser/client behavior. HTTP/2 is negotiated automatically over TLS if the server supports ALPN — no special config needed, though you can force it via `http2: true` in some contexts.

### Best Practices
- Always set `Content-Type` explicitly for POST/PUT/PATCH with JSON bodies — Express's `body-parser`/`express.json()` middleware relies on it.
- Reuse tokens/sessions across iterations where realistic (see Ch. 20 token reuse) instead of logging in on every single iteration, which overloads your auth endpoint disproportionately.

### Common Mistakes
- Sending a JS object directly as the POST body without `JSON.stringify()` — Express will receive `[object Object]` as a string.
- Forgetting the `Content-Type: application/json` header — `express.json()` silently skips parsing and `req.body` is `undefined`.
- Not handling redirects correctly when testing login flows that redirect (301/302).

### Interview Questions
**Q: How do you send a JSON POST body in k6?**
A: `http.post(url, JSON.stringify(payload), { headers: { 'Content-Type': 'application/json' } })` — the body must be a string, and the header tells Express how to parse it.

**Q: Does k6 automatically manage cookies like a browser?**
A: Yes, each VU has its own cookie jar; k6 stores and resends cookies set by `Set-Cookie` response headers automatically, unless you override this behavior.

### Exercises
1. Write a script that logs in, extracts a JWT, and uses it to call a protected `/profile` endpoint.
2. Add a multipart upload test against an Express route using `multer`.

---

## 7. Checks and Assertions

`check()` records pass/fail assertions **without stopping the test** (unlike traditional `assert`). Failed checks show up as a `checks` metric but don't halt execution.

```javascript
import http from 'k6/http';
import { check, fail } from 'k6';

export default function () {
  const res = http.get('http://localhost:3000/api/products');

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response has products array': (r) => Array.isArray(r.json('products')),
    'response time < 300ms': (r) => r.timings.duration < 300,
    'has correct content-type': (r) => r.headers['Content-Type'].includes('application/json'),
  });

  // fail() stops the current iteration immediately (use sparingly)
  if (res.status >= 500) {
    fail('Server returned 5xx — aborting iteration');
  }
}
```

### Custom Assertion Helper
```javascript
function expectStatus(res, expected) {
  return check(res, { [`status is ${expected}`]: (r) => r.status === expected });
}
```

### Best Practices
- Use `check()` for **anything you want visibility into** — it doesn't stop the test, so it's safe to add liberally.
- Reserve `fail()` for truly catastrophic conditions where continuing the iteration is meaningless (e.g., auth completely broken).
- Combine `check()` results with **thresholds** (Ch. 12) so a low check pass-rate actually fails the CI build.

### Common Mistakes
- Confusing `check()` with `throw`/`assert` — a failed check does **not** stop the script by default.
- Not checking response *body* correctness, only status code — a `200 OK` with an empty or malformed body can still be a bug.
- Writing checks that reference the same object property inconsistently between iterations (e.g., assuming array order).

### Interview Questions
**Q: Does a failed `check()` stop the test?**
A: No — it's recorded as a failed check and contributes to the `checks` metric's failure rate, but the script keeps running. Use thresholds on the `checks` metric to fail the overall run.

**Q: When would you use `fail()` instead of `check()`?**
A: When continuing the iteration is pointless or could cause misleading downstream results — e.g., if login itself fails, there's no token to test the next 10 authenticated requests with.

### Exercises
1. Add checks validating status code, response time, and a specific JSON field for a `/api/orders` endpoint.
2. Add a threshold requiring 99% of checks to pass.

---

## 8. Params: Headers, Cookies, Tags

The third argument to any `http.*` call is the **params object** — it controls everything about how the request is sent and recorded.

```javascript
import http from 'k6/http';

const params = {
  headers: { 'Content-Type': 'application/json', 'X-Request-Source': 'k6' },
  cookies: { theme: 'dark' },
  tags: { endpoint: 'create_order', team: 'checkout' }, // custom metric tags
  timeout: '5s',
  redirects: 4,
  compression: 'gzip',
};

http.post('http://localhost:3000/api/orders', JSON.stringify({ item: 'sku_1' }), params);
```

### Why Tags Matter
Tags let you slice metrics later — e.g., "show me `http_req_duration` only for `endpoint:create_order`" — essential once you're testing many endpoints in one script.

```javascript
import { Trend } from 'k6/metrics';
const orderDuration = new Trend('order_creation_duration', true);

export default function () {
  const res = http.post(/* ... */);
  orderDuration.add(res.timings.duration, { endpoint: 'create_order' });
}
```

### Best Practices
- Tag every request with a meaningful `name` param to group dynamic URLs (e.g., `/products/123` → `name: 'get_product_by_id'`), preventing k6 from treating every unique ID as a separate high-cardinality metric bucket.
- Centralize common headers/params in a shared config object.

### Common Mistakes
- Not using the `name` tag for dynamic URLs, causing metrics explosion (one series per unique URL).
- Overwriting the default headers object accidentally by mutating a shared object across requests.

### Interview Questions
**Q: Why should you tag dynamic URLs with a static `name`?**
A: Without it, k6 (and any exported metrics backend) treats each unique URL as a separate time series, which explodes cardinality and makes aggregate reporting (e.g., "average product lookup time") impossible.

### Exercises
1. Tag five different endpoint calls with `name` params and confirm the summary groups them correctly.

---

## 9. Test Lifecycle: init, setup, default, teardown

```
┌────────────┐     runs once, before VUs start
│    init    │  →  imports, option declarations, global consts
└─────┬──────┘
      │
┌─────▼──────┐     runs once, on ONE VU, before the load test starts
│   setup()  │  →  create test data, log in an admin, seed DB
└─────┬──────┘     return value is passed into default() and teardown()
      │
┌─────▼──────┐     runs REPEATEDLY, once per iteration, per VU
│  default() │  →  the actual load — HTTP calls, checks
└─────┬──────┘
      │  (loop for full test duration)
┌─────▼──────┐     runs once, on ONE VU, after the load test ends
│ teardown() │  →  cleanup test data, log summary info
└────────────┘
```

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = { vus: 10, duration: '30s' };

// setup(): runs once before the load starts
export function setup() {
  const res = http.post('http://localhost:3000/api/auth/login', JSON.stringify({
    email: 'loadtest@example.com', password: 'testpass',
  }), { headers: { 'Content-Type': 'application/json' } });

  return { token: res.json('token') }; // passed to default() and teardown()
}

// default(): the actual load test, run per VU per iteration
export default function (data) {
  const res = http.get('http://localhost:3000/api/orders', {
    headers: { Authorization: `Bearer ${data.token}` },
  });
  check(res, { 'status 200': (r) => r.status === 200 });
}

// teardown(): runs once after all VUs finish
export function teardown(data) {
  http.post('http://localhost:3000/api/test/cleanup', JSON.stringify({}), {
    headers: { Authorization: `Bearer ${data.token}`, 'Content-Type': 'application/json' },
  });
  console.log('Test data cleaned up');
}
```

### Best Practices
- Do expensive one-time work (login, data seeding) in `setup()`, not `default()` — otherwise every VU logs in on every iteration, testing your login endpoint instead of the endpoint you meant to test.
- Use `teardown()` to leave the environment as clean as you found it, especially in shared staging environments.

### Common Mistakes
- Putting login calls inside `default()` — this multiplies auth load unrealistically and pollutes your metrics with login latency, not the actual target endpoint.
- Forgetting `setup()`'s return value must be JSON-serializable (no functions, no complex class instances).
- Assuming `setup()`/`teardown()` run once per VU — they run **once total**, on a single VU, regardless of how many VUs the test uses.

### Interview Questions
**Q: Why shouldn't you put authentication logic inside the default function?**
A: It forces every single iteration of every VU to re-authenticate, drastically overloading the login endpoint relative to real traffic patterns, and mixes login latency into metrics meant to measure a different endpoint.

**Q: How many times does `setup()` run in a test with 50 VUs?**
A: Once — regardless of VU count — before the main load test begins.

### Exercises
1. Refactor a script that logs in inside `default()` to instead log in once in `setup()` and reuse the token.

---

## 10. Scenarios and Executors

Scenarios let you run multiple traffic patterns in a single test — e.g., simulate steady browsing traffic *and* a spike of checkout traffic simultaneously.

| Executor | Use case |
|---|---|
| `constant-vus` | Fixed number of concurrent users for a fixed time |
| `ramping-vus` | Gradually increase/decrease concurrent users (classic ramp-up load test) |
| `shared-iterations` | Fixed total iterations shared across a VU pool (good for "run this batch job N times") |
| `per-vu-iterations` | Each VU runs a fixed number of iterations independently |
| `constant-arrival-rate` | Fixed number of iterations **started** per time unit, regardless of how long each takes — best for accurate requests-per-second targets |
| `ramping-arrival-rate` | Gradually increase/decrease the request rate |
| `externally-controlled` | Control VUs/iterations dynamically from an external process (deprecated in newer k6 versions in favor of the REST API) |

```javascript
export const options = {
  scenarios: {
    steady_browsing: {
      executor: 'constant-vus',
      vus: 20,
      duration: '5m',
      exec: 'browseProducts',
    },
    checkout_spike: {
      executor: 'ramping-arrival-rate',
      startRate: 5,
      timeUnit: '1s',
      preAllocatedVUs: 50,
      maxVUs: 200,
      stages: [
        { target: 5, duration: '30s' },
        { target: 100, duration: '1m' }, // sudden spike
        { target: 5, duration: '30s' },
      ],
      exec: 'checkout',
    },
  },
};

export function browseProducts() { /* GET /products */ }
export function checkout() { /* POST /orders */ }
```

### `constant-vus` vs `constant-arrival-rate`
```
constant-vus (20 VUs, 30s)              constant-arrival-rate (20 req/s, 30s)
──────────────────────────              ─────────────────────────────────────
Fixed # of workers loop as              Fixed request RATE is maintained;
fast as their think-time allows.        k6 auto-scales VUs (up to maxVUs)
Throughput DEPENDS on response          to hit the target rate even if
time — if the API slows down,           responses slow down. Best for
throughput drops.                       realistic "X requests/sec" SLAs.
```

### Best Practices
- Use `constant-arrival-rate` / `ramping-arrival-rate` when your goal is "simulate exactly N requests/sec," since VU-based executors' throughput drifts as response time changes.
- Use `ramping-vus` for classic "ramp up, hold, ramp down" user-concurrency load tests.
- Always set `preAllocatedVUs` generously for arrival-rate executors — if k6 runs out of VUs to maintain the rate, it reports errors, not slower throughput.

### Common Mistakes
- Using `constant-vus` and expecting a stable requests/sec number — it's not guaranteed; it depends entirely on response latency.
- Setting `maxVUs` too low for `ramping-arrival-rate`, causing k6 itself to become the bottleneck.

### Interview Questions
**Q: Why would you choose `constant-arrival-rate` over `constant-vus`?**
A: `constant-vus` throughput varies with response time (slower responses = fewer requests/sec from the same VU count); `constant-arrival-rate` explicitly targets and maintains a fixed request rate, which better matches real-world SLA/capacity goals like "handle 200 req/s."

**Q: What happens if `constant-arrival-rate` can't maintain the target rate?**
A: k6 raises a warning/error indicating it ran out of preallocated VUs (hit `maxVUs`), meaning the test itself became resource-constrained rather than accurately measuring the target system.

### Exercises
1. Build a scenario with two concurrent traffic patterns: steady background browsing and a ramping checkout spike.
2. Convert a `constant-vus` test into an equivalent `constant-arrival-rate` test targeting 50 req/s.

---

## 11. Metrics Deep Dive

### Built-in Metrics

| Metric | Type | Meaning |
|---|---|---|
| `http_req_duration` | Trend | Total time for the request (DNS + connect + TLS + send + wait + receive) |
| `http_req_waiting` | Trend | Time-to-first-byte (server processing time — often the most telling metric) |
| `http_req_blocked` | Trend | Time spent waiting for a free TCP connection |
| `http_req_connecting` | Trend | TCP handshake time |
| `http_req_tls_handshaking` | Trend | TLS negotiation time |
| `http_req_sending` | Trend | Time spent sending request data |
| `http_req_receiving` | Trend | Time spent receiving response data |
| `http_req_failed` | Rate | Fraction of requests that failed (network error or configured "bad" status) |
| `http_reqs` | Counter | Total HTTP requests made |
| `vus` | Gauge | Current active VUs |
| `vus_max` | Gauge | Max VUs configured |
| `iterations` | Counter | Total completed iterations |
| `iteration_duration` | Trend | Time for one full default() run |
| `data_sent` / `data_received` | Counter | Network bytes transferred |
| `checks` | Rate | Fraction of `check()` assertions that passed |

### Interpreting Them
- **`http_req_waiting` vs `http_req_duration`:** if `waiting` is high but `sending`/`receiving` are low, the bottleneck is server-side processing (your Express route/DB), not the network.
- **`http_req_failed`** climbing under load usually means the server is shedding load, timing out, or crashing — correlate with server-side CPU/memory graphs.
- **`iterations` vs `http_reqs`:** if one iteration makes 3 requests, `http_reqs` will be ~3x `iterations`.

### Best Practices
- Watch `http_req_waiting` specifically for backend bottleneck diagnosis — it isolates server think-time from network transfer time.
- Export metrics (Ch. 18) rather than relying only on the terminal summary for long/complex tests.

### Common Mistakes
- Only looking at averages — see Ch. 16 on why percentiles matter far more.
- Ignoring `http_req_failed` because the average duration "looks fine" — a bimodal distribution (fast successes + fast failures) can hide serious problems in an average.

### Interview Questions
**Q: What's the difference between `http_req_duration` and `http_req_waiting`?**
A: `http_req_duration` is the full request lifecycle (connect + send + wait + receive); `http_req_waiting` isolates just the server's processing time (time to first byte), making it the better signal for backend performance specifically.

**Q: What kind of metric is `http_req_failed`, and why does that matter?**
A: It's a `Rate` metric (fraction between 0 and 1), which is why thresholds on it typically look like `http_req_failed: ['rate<0.01']` — under 1% failure.

### Exercises
1. Run a test and identify whether latency is dominated by `waiting`, `sending`, or `receiving`.

---

## 12. Thresholds, SLAs, SLOs

Thresholds turn metrics into **pass/fail criteria** — this is what makes k6 usable in CI/CD.

```javascript
export const options = {
  thresholds: {
    http_req_duration: ['p(95)<300', 'p(99)<800'], // 95% under 300ms, 99% under 800ms
    http_req_failed: ['rate<0.01'],                 // less than 1% failures
    checks: ['rate>0.99'],                           // 99%+ checks pass
    'http_req_duration{endpoint:login}': ['p(95)<500'], // per-tag threshold
  },
};
```

### SLA vs SLO vs Error Budget
- **SLA (Service Level Agreement):** a contractual promise to customers (e.g., "99.9% uptime, or you get a refund").
- **SLO (Service Level Objective):** the internal target a team designs to meet the SLA with margin (e.g., 99.95% success rate target, to keep the 99.9% SLA safe).
- **Error budget:** the allowed amount of "badness" before you're out of compliance (100% - SLO). If your SLO is 99.9% success over 30 days, your error budget is 0.1% of requests, or roughly 43 minutes of full downtime equivalent.

### Best Practices
- Base thresholds on real business requirements/SLOs, not arbitrary numbers.
- Use `abortOnFail` for critical thresholds during smoke tests, to stop expensive test runs early on catastrophic failure:
```javascript
thresholds: {
  http_req_failed: [{ threshold: 'rate<0.01', abortOnFail: true }],
}
```

### Common Mistakes
- Setting thresholds so loose they never fail (defeats the purpose in CI).
- Only setting an average-based threshold, ignoring tail latency (P95/P99), which is what users actually feel.

### Interview Questions
**Q: What does `p(95)<300` mean as a threshold?**
A: 95% of the recorded values for that metric (typically request duration) must be under 300ms — i.e., the 95th percentile latency must stay below 300ms.

**Q: How do thresholds enable k6 in CI/CD?**
A: k6 exits with a non-zero status code if any threshold fails, which CI systems interpret as a failed build/step — turning performance regressions into automatic pipeline failures rather than something a human has to notice.

### Exercises
1. Define thresholds for an API with an SLO of "P95 < 250ms, error rate < 0.5%."
2. Add a per-endpoint threshold using tags.

---

## 13. Options Reference

```javascript
export const options = {
  vus: 10,                      // number of virtual users (simple mode)
  duration: '30s',               // total test duration (simple mode)
  iterations: 100,               // total iterations across all VUs (simple mode)

  stages: [                      // ramping profile (alternative to vus/duration)
    { duration: '1m', target: 20 },
    { duration: '3m', target: 20 },
    { duration: '1m', target: 0 },
  ],

  thresholds: { /* see Ch. 12 */ },
  scenarios: { /* see Ch. 10 */ },

  tags: { environment: 'staging', project: 'checkout-api' },

  summaryTrendStats: ['avg', 'min', 'med', 'max', 'p(90)', 'p(95)', 'p(99)'],
  summaryTimeUnit: 'ms',

  noConnectionReuse: false,      // keep-alive on by default
  insecureSkipTLSVerify: false,  // don't use in real prod tests
  discardResponseBodies: true,   // improves performance if you don't need bodies
};
```

### Notes
- `stages` and `vus`/`duration` are mutually exclusive simplified configs — `scenarios` supersedes both for anything non-trivial.
- `discardResponseBodies: true` significantly reduces k6's own memory/CPU usage during large tests — enable it unless you need to inspect bodies in checks (you can override per-request with `responseType: 'text'`).

### Best Practices
- Prefer `scenarios` for any real project — it's explicit and composable.
- Set `summaryTrendStats` to include the percentiles your thresholds care about.

### Common Mistakes
- Defining both `vus`/`duration` and `scenarios` and being surprised which one "wins" (scenarios take precedence).
- Forgetting `discardResponseBodies`, causing k6 itself to run out of memory on very large-scale tests.

### Interview Questions
**Q: When would you use `stages` instead of `scenarios`?**
A: For a simple, single-pattern ramp-up/ramp-down load test where you don't need multiple concurrent traffic shapes or custom executors — `stages` is a convenient shorthand that maps to a `ramping-vus` scenario under the hood.

### Exercises
1. Convert a `stages`-based config into an explicit `ramping-vus` scenario.

---

## 14. Load Testing Real REST APIs

Example Express routes assumed:
```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/refresh
GET    /api/products?page=1&limit=20&sort=price&category=shoes
GET    /api/products/:id
POST   /api/cart
POST   /api/orders
POST   /api/payments
POST   /api/upload
GET    /api/orders/:id  (JWT protected)
```

### Login
```javascript
import http from 'k6/http';
import { check } from 'k6';

export default function () {
  const res = http.post('http://localhost:3000/api/auth/login', JSON.stringify({
    email: 'user1@example.com', password: 'Password123!',
  }), { headers: { 'Content-Type': 'application/json' } });

  check(res, {
    'login succeeded': (r) => r.status === 200,
    'token present': (r) => !!r.json('token'),
  });
}
```

### Register (with dynamic/unique data)
```javascript
import http from 'k6/http';
import { randomString } from 'https://jslib.k6.io/k6-utils/1.5.0/index.js';

export default function () {
  const email = `user_${randomString(8)}@example.com`;
  http.post('http://localhost:3000/api/auth/register', JSON.stringify({
    email, password: 'Password123!', name: 'Load Test User',
  }), { headers: { 'Content-Type': 'application/json' } });
}
```

### Pagination, Filtering, Sorting
```javascript
export default function () {
  const page = Math.floor(Math.random() * 5) + 1;
  http.get(`http://localhost:3000/api/products?page=${page}&limit=20&sort=price&category=shoes`, {
    tags: { name: 'list_products_paginated' },
  });
}
```

### JWT-Protected Endpoint with Refresh
```javascript
export function setup() {
  const login = http.post('http://localhost:3000/api/auth/login', JSON.stringify({
    email: 'user1@example.com', password: 'Password123!',
  }), { headers: { 'Content-Type': 'application/json' } });
  return { token: login.json('token'), refreshToken: login.json('refreshToken') };
}

export default function (data) {
  let res = http.get('http://localhost:3000/api/orders/123', {
    headers: { Authorization: `Bearer ${data.token}` },
  });

  if (res.status === 401) {
    const refreshed = http.post('http://localhost:3000/api/auth/refresh', JSON.stringify({
      refreshToken: data.refreshToken,
    }), { headers: { 'Content-Type': 'application/json' } });
    data.token = refreshed.json('token');
    res = http.get('http://localhost:3000/api/orders/123', {
      headers: { Authorization: `Bearer ${data.token}` },
    });
  }
  check(res, { 'order fetched': (r) => r.status === 200 });
}
```

### File Upload
```javascript
import http from 'k6/http';
const fileData = open('./avatar.jpg', 'b');

export default function () {
  http.post('http://localhost:3000/api/upload', {
    file: http.file(fileData, 'avatar.jpg', 'image/jpeg'),
  });
}
```

### Best Practices
- Use realistic, varied test data (random users, random product IDs) — hitting the exact same row/cache entry every time hides real-world cache-miss and index behavior.
- Group related requests logically per iteration to mimic a real user session (browse → add to cart → checkout), not isolated single-endpoint hammering, unless that's specifically your goal.

### Common Mistakes
- Reusing the same hardcoded test user across all VUs, causing unique-constraint failures on register endpoints or unrealistic cache hit rates.
- Testing only the "happy path" and never invalid input, expired tokens, or edge cases.

### Interview Questions
**Q: Why is using the same static test data across all VUs a problem?**
A: It can cause artificial database contention (row locking), unrealistic caching, or constraint violations (e.g., duplicate registration) — and it hides bugs that only show up with data variety.

### Exercises
1. Build a full "browse → add to cart → checkout" journey script chaining four endpoints.
2. Add invalid-token and expired-token cases to your JWT test.

---

## 15. Testing a Real Express Backend

### Minimal Express App Under Test
```javascript
// server.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

const SECRET = 'demo-secret';
let products = Array.from({ length: 1000 }, (_, i) => ({ id: i + 1, name: `Product ${i + 1}`, price: (i % 50) + 10 }));

app.post('/api/auth/login', (req, res) => {
  const { email, password } = req.body;
  if (email && password) {
    const token = jwt.sign({ email }, SECRET, { expiresIn: '1h' });
    return res.json({ token });
  }
  res.status(400).json({ error: 'Invalid credentials' });
});

function auth(req, res, next) {
  const header = req.headers.authorization;
  if (!header) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(header.split(' ')[1], SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

app.get('/api/products', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const start = (page - 1) * limit;
  res.json({ products: products.slice(start, start + limit), total: products.length });
});

app.get('/api/orders', auth, (req, res) => {
  res.json({ orders: [{ id: 1, item: 'demo' }] });
});

app.listen(3000, () => console.log('API running on :3000'));
```

### The Middleware/Routing/DB Chain, Under Load
```
Request → [Express router] → [auth middleware] → [route handler]
                                                       │
                                            [DB query / ORM call]
                                                       │
                                              [response serialization]
```

Under load, each layer can become the bottleneck:
- **Router:** rarely a bottleneck unless you have thousands of regex routes.
- **Middleware:** synchronous, CPU-heavy middleware (e.g., unoptimized validation, sync crypto) blocks the Node.js event loop for **every** concurrent request.
- **DB queries:** connection pool exhaustion, missing indexes, N+1 patterns — usually the #1 real-world bottleneck.
- **Serialization:** `JSON.stringify()` on huge objects is synchronous and CPU-bound.

### Progressive Load Test
```javascript
export const options = {
  stages: [
    { duration: '1m', target: 10 },
    { duration: '2m', target: 50 },
    { duration: '2m', target: 100 },
    { duration: '2m', target: 200 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<400'],
    http_req_failed: ['rate<0.02'],
  },
};

export default function () {
  http.get('http://localhost:3000/api/products?page=1&limit=20');
}
```

As VUs climb from 10 → 200, watch for the point where `http_req_waiting` starts climbing disproportionately faster than VU count — that's your saturation point. Correlate it with server-side CPU/event-loop-lag graphs (Ch. 19).

### Best Practices
- Load test against a build that mirrors production config (compression, logging level, clustering) — a debug-mode dev server behaves very differently under load.
- Run Node with `NODE_ENV=production` during load tests — this alone materially changes performance (Express disables verbose error stacks, view caching, etc.).

### Common Mistakes
- Load testing a single-process `node server.js` and drawing conclusions about a production deployment that runs with clustering/PM2/multiple replicas behind a load balancer.
- Not resetting test data between soak-test runs, letting an in-memory array or dev DB grow unbounded and skewing later results.

### Interview Questions
**Q: Why does Node.js's single-threaded event loop matter for load testing?**
A: Any synchronous, CPU-bound code in a request handler blocks the entire event loop, delaying *all* concurrent requests — not just the one being processed. Under load this shows up as latency spikes that get worse non-linearly with concurrency, which is exactly what a load test is designed to surface.

### Exercises
1. Load test the sample Express app above and identify the VU count at which P95 latency exceeds 400ms.
2. Add an intentionally blocking synchronous loop to one route, then re-run the test and observe the impact on *all* endpoints, not just the slow one.

---

## 16. Reading and Interpreting Reports

### Sample k6 Summary Output
```
     ✓ status is 200
     ✓ response time < 300ms

     checks.........................: 100.00% ✓ 4820  ✗ 0
     data_received..................: 3.2 MB  53 kB/s
     data_sent.......................: 410 kB  6.8 kB/s
     http_req_blocked...............: avg=1.2ms   min=0s      med=0s      max=45ms   p(90)=0s     p(95)=2ms
     http_req_duration..............: avg=142ms   min=38ms    med=120ms   max=1.8s   p(90)=210ms  p(95)=310ms
     http_req_failed................: 0.41%   ✓ 10    ✗ 2410
     http_reqs.......................: 2420    40.3/s
     iteration_duration..............: avg=1.14s   min=1.03s   med=1.12s   max=2.8s   p(90)=1.21s  p(95)=1.31s
     iterations......................: 2410    40.1/s
     vus..............................: 50      min=50   max=50
```

### What Each Term Means
| Term | Meaning |
|---|---|
| **Average** | Mean of all values — easily skewed by outliers, least reliable for SLAs |
| **Median (p50)** | Middle value — half of requests were faster, half slower |
| **P95** | 95% of requests were faster than this value — the classic "typical worst case" SLA metric |
| **P99** | 99% of requests were faster — captures rarer, often more painful tail latency |
| **Min / Max** | Fastest/slowest single request observed |
| **Throughput** | Requests handled per second (`http_reqs` rate) |
| **Error rate** | `http_req_failed` — fraction of failed requests |

### Why Percentiles Beat Averages
```
Example: 100 requests, 95 take 100ms, 5 take 5000ms (a stuck DB connection pool)
Average  = (95×100 + 5×5000) / 100 = 345ms      ← looks "okay-ish"
P95      = 100ms                                  ← looks great, misleading!
P99      = ~5000ms                                 ← reveals the real problem
```
This is why professional SLAs are defined on **P95/P99**, never on averages — averages hide the tail experience that a meaningful fraction of your real users actually feel.

### What "Good" Looks Like (general guidance, not universal law)
- **P95 < 200–300ms** for typical CRUD JSON APIs is a common target.
- **Error rate < 0.1–1%** depending on criticality.
- **Throughput** should scale close to linearly with added resources until a clear ceiling — a sudden plateau or drop indicates saturation (DB, connection pool, CPU).

### Best Practices
- Always report P95/P99, never just the average, in any performance summary you share with stakeholders.
- Correlate `http_req_failed` spikes with timestamps in your server-side logs/APM.

### Common Mistakes
- Presenting "average response time: 120ms" as the whole story to leadership.
- Ignoring `min`/`max` outliers that might indicate cold-start or GC pause issues.

### Interview Questions
**Q: Why do SLAs use P95/P99 instead of averages?**
A: Averages are easily dominated by the majority of fast requests and can completely hide a meaningful tail of slow requests, which is exactly the group of users experiencing the worst — and most complaint-generating — performance.

### Exercises
1. Given a summary with avg=100ms and p99=2000ms, explain in your own words what's likely happening.

---

## 17. HTML, JSON, CSV, JUnit Reports

### JSON Output (raw, detailed, streams every data point)
```bash
k6 run --out json=results.json script.js
```

### CSV Output
```bash
k6 run --out csv=results.csv script.js
```

### JUnit Output (for CI test-result dashboards, e.g. Jenkins/GitLab)
```bash
k6 run --summary-export=summary.json script.js
```
For proper JUnit XML, combine k6's JSON output with a converter, or use the `k6-reporter`/`xk6-junit` extensions depending on your k6 version.

### HTML Report with `k6-reporter`
```javascript
import http from 'k6/http';
import { textSummary } from 'https://jslib.k6.io/k6-summary/0.0.2/index.js';
import { htmlReport } from 'https://raw.githubusercontent.com/benc-uk/k6-reporter/main/dist/bundle.js';

export const options = { vus: 20, duration: '30s' };

export default function () {
  http.get('http://localhost:3000/api/products');
}

export function handleSummary(data) {
  return {
    'summary.html': htmlReport(data),
    stdout: textSummary(data, { indent: ' ', enableColors: true }),
  };
}
```
Run normally (`k6 run script.js`) and open `summary.html` in a browser for a shareable, styled report with charts.

### Best Practices
- Use JSON output for feeding a metrics pipeline (Ch. 18); use HTML reports for human-readable sharing with non-technical stakeholders.
- Archive reports per CI run (as build artifacts) to track trends over time.

### Common Mistakes
- Streaming full JSON output (`--out json`) for very long soak tests without considering disk space — it can grow to gigabytes.
- Not version-controlling report thresholds alongside the report format, causing confusion about what "pass" meant historically.

### Interview Questions
**Q: What's the difference between the built-in summary and `--out json`?**
A: The built-in summary is an aggregated end-of-run report (percentiles, totals); `--out json` streams a raw event per data point, useful for custom analysis or feeding external systems, but far larger and not meant for direct human reading.

### Exercises
1. Generate an HTML report for a test and open it locally.
2. Export JSON output and write a small script to compute a custom percentile from raw data.

---

## 18. Grafana, InfluxDB, Prometheus Integration

### Architecture
```
┌────────┐   metrics    ┌────────────┐   query    ┌──────────┐
│  k6    │ ───────────► │ InfluxDB / │ ─────────► │ Grafana  │
│ (load) │              │ Prometheus │            │(dashboard)│
└────────┘              └────────────┘            └──────────┘
```

### Option A: InfluxDB (classic, simple)
```bash
k6 run --out influxdb=http://localhost:8086/k6 script.js
```
Then add InfluxDB as a data source in Grafana and import the official k6 dashboard (search "k6 Load Testing Results" in Grafana's dashboard library).

### Option B: Prometheus Remote Write (modern, recommended)
```bash
K6_PROMETHEUS_RW_SERVER_URL=http://localhost:9090/api/v1/write \
  k6 run -o experimental-prometheus-rw script.js
```
Prometheus scrapes/receives these metrics, and Grafana queries Prometheus as the data source.

### docker-compose for a Local Observability Stack
```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:1.8
    ports: ["8086:8086"]
    environment:
      - INFLUXDB_DB=k6
  grafana:
    image: grafana/grafana:latest
    ports: ["3001:3000"]
    depends_on: [influxdb]
```

### Live Monitoring
Grafana dashboards refresh in near real-time as k6 streams metrics, letting you watch latency/error rate/throughput graphs live during a run — critical for long soak tests where you don't want to wait until the end to notice a problem.

### Alerts
Configure Grafana alert rules (e.g., "alert if P95 > 500ms for 2 minutes") so a long-running test can page you rather than requiring someone to watch a dashboard.

### Best Practices
- Use Prometheus remote-write for new setups — InfluxDB v1 is in maintenance mode in many ecosystems.
- Tag metrics consistently (Ch. 8) so Grafana panels can filter/group meaningfully.

### Common Mistakes
- Forgetting to pre-create the InfluxDB database (`k6` db) before running, causing silent write failures.
- Dashboarding only client-side k6 metrics without also displaying server-side metrics (CPU, event loop lag) on the same time axis — correlation is the whole point.

### Interview Questions
**Q: Why export k6 metrics to Grafana instead of just reading the terminal summary?**
A: The terminal summary is a single aggregate snapshot at the end; a live dashboard shows how metrics evolve *during* the test (critical for long soak/spike tests), supports alerting, and lets you correlate client-side load metrics with server-side infrastructure metrics on the same timeline.

### Exercises
1. Stand up the InfluxDB + Grafana docker-compose stack and stream a test run into it.
2. Build a Grafana panel showing P95 latency over time alongside request rate.

---

## 19. Real Performance Investigation Playbook

**Scenario:** Your `/api/orders` endpoint, fast in dev, is timing out under 200 concurrent users in staging. Here's the systematic investigation.

```
                     ┌─────────────────────────┐
                     │   Symptom: high latency  │
                     │   under load             │
                     └────────────┬─────────────┘
                                  ▼
        ┌─────────────────────────────────────────────┐
        │ Step 1: Isolate — client, network, or server? │
        │  Check http_req_blocked/connecting (k6 side)  │
        └────────────────────┬──────────────────────────┘
                              ▼ (server-side confirmed)
        ┌─────────────────────────────────────────────┐
        │ Step 2: OS/process metrics — CPU, RAM, GC     │
        └────────────────────┬──────────────────────────┘
                              ▼
        ┌─────────────────────────────────────────────┐
        │ Step 3: Application layer — event loop lag,   │
        │  blocking code, connection pool exhaustion    │
        └────────────────────┬──────────────────────────┘
                              ▼
        ┌─────────────────────────────────────────────┐
        │ Step 4: Database layer — slow queries,        │
        │  missing indexes, N+1, lock contention         │
        └─────────────────────────────────────────────┘
```

### CPU
- Watch process CPU% during the test (`top`, `htop`, or an APM). Sustained 100% on a single core in Node.js (which is largely single-threaded for JS execution) indicates CPU-bound work blocking the event loop.
- Common culprits: synchronous JSON parsing of huge payloads, synchronous crypto (`bcrypt.hashSync`), regex catastrophic backtracking, unoptimized loops.

### RAM / Memory Leaks
- Watch RSS memory over a long soak test — a steadily climbing line that never plateaus is a leak.
- Common Node.js leak sources: growing arrays/caches with no eviction, unclosed event listeners, unbounded `setInterval` accumulation, unresolved Promises holding references.
- Use `--inspect` + Chrome DevTools heap snapshots, or `clinic.js`/`0x` for flame graphs, to pinpoint the exact leaking allocation.

### Node Event Loop
- Event loop lag (the delay between scheduling a callback and it actually running) is the single most important Node.js-specific performance signal. Libraries like `@nodejs/clinic` or simple `perf_hooks.monitorEventLoopDelay()` expose this directly.
- If event loop lag climbs in lockstep with k6's VU count, you have blocking synchronous code somewhere in the request path.

### Database Bottlenecks
- **Missing indexes:** a query that's fast with 100 rows can be catastrophic at 1M rows without the right index — `EXPLAIN ANALYZE` (Postgres) or `.explain()` (MongoDB) is essential.
- **Connection pool exhaustion:** if your pool max is 10 and you have 200 concurrent requests, 190 of them queue waiting for a connection — this shows up as latency that scales almost linearly with VU count past the pool size.
- **N+1 queries:** an ORM (Prisma, Sequelize, Mongoose) fetching a list then looping to fetch related data per row turns 1 query into 1+N — use `include`/`populate`/joins instead.
- **Slow queries:** enable slow query logging in dev/staging and check it after every load test run.

### Caching / Redis
- A cache-miss storm (e.g., cold cache after a deploy, or a low TTL causing constant re-computation) can make a normally-fast endpoint look catastrophically slow under load — always warm caches before treating cold-cache numbers as representative, unless cold-start behavior is specifically what you're testing.

### Large Payloads & Compression
- Un-gzipped large JSON responses inflate `data_sent`/`data_received` and network time disproportionately; verify `Content-Encoding: gzip` is present under load, not just in a manual curl.

### Logging Overhead
- Synchronous, verbose logging (`console.log` in a hot path, or a logger without proper async transport/buffering) can itself become a bottleneck at high request rates.

### GC Pauses
- V8's garbage collector pauses the event loop briefly during major collections. High allocation rates (creating lots of short-lived objects per request) increase GC frequency/duration. Correlate GC events (`--trace-gc` flag) with latency spikes.

### Best Practices
- Always graph client-side (k6) and server-side (APM/system) metrics on the same timeline — correlation, not isolated numbers, finds the root cause.
- Reproduce the same load pattern in a controlled staging environment before touching production.

### Common Mistakes
- Jumping to "we need more servers" (horizontal scaling) before confirming whether the bottleneck is even scalable that way (e.g., a single shared DB connection pool won't be fixed by adding app servers).
- Fixing the symptom (bumping a timeout) instead of the cause (a genuinely slow query).

### Interview Questions
**Q: How would you diagnose whether a slow API is CPU-bound or I/O-bound?**
A: Watch event loop lag and process CPU% during load: consistently high CPU with rising event loop lag points to CPU-bound blocking code; low CPU but high response latency with requests "waiting" typically points to I/O (DB, network, external API) as the bottleneck.

**Q: What's an N+1 query problem, and how do you detect it under load?**
A: It's when fetching a list of N items triggers N additional individual queries for related data (e.g., fetching 50 orders, then querying the customer for each one separately) instead of a single join/batched query. It's detectable via query logs/APM showing query *count* scaling linearly with response size, and via load tests showing latency degrading much faster than expected as data volume grows.

### Exercises
1. Given a load test where `http_req_waiting` grows linearly with VUs past 50 concurrent users, hypothesize three possible server-side causes and how you'd confirm each.
2. Design a soak test (4+ hours, constant load) specifically to catch a memory leak, and describe what a "leak" looks like in the resulting memory-over-time graph.

---

## 20. Advanced k6

### Custom Metrics
```javascript
import { Trend, Rate, Counter, Gauge } from 'k6/metrics';

const dbQueryTime = new Trend('db_query_time', true);   // custom timing distribution
const cacheHitRate = new Rate('cache_hit_rate');         // custom pass/fail rate
const ordersCreated = new Counter('orders_created');     // custom running total
const activeSessions = new Gauge('active_sessions');     // custom point-in-time value

export default function () {
  const res = http.post(/* create order */);
  ordersCreated.add(1);
  cacheHitRate.add(res.headers['X-Cache'] === 'HIT');
  dbQueryTime.add(Number(res.headers['X-Db-Time-Ms'] || 0));
  activeSessions.add(1);
}
```

### Groups
```javascript
import { group } from 'k6';

export default function () {
  group('checkout flow', function () {
    group('add to cart', function () { http.post(/* ... */); });
    group('apply coupon', function () { http.post(/* ... */); });
    group('pay', function () { http.post(/* ... */); });
  });
}
```
Groups organize metrics hierarchically in the summary output — useful for multi-step journeys.

### SharedArray (memory-efficient data)
```javascript
import { SharedArray } from 'k6/data';

// Loaded once, shared read-only across all VUs (not duplicated per-VU)
const users = new SharedArray('users', function () {
  return JSON.parse(open('./users.json'));
});

export default function () {
  const user = users[Math.floor(Math.random() * users.length)];
  http.post('http://localhost:3000/api/auth/login', JSON.stringify(user), {
    headers: { 'Content-Type': 'application/json' },
  });
}
```

### CSV Data Parameterization
```javascript
import papaparse from 'https://jslib.k6.io/papaparse/5.1.1/index.js';
import { SharedArray } from 'k6/data';

const csvData = new SharedArray('csv-data', function () {
  return papaparse.parse(open('./users.csv'), { header: true }).data;
});
```

### Random / Dynamic Users
```javascript
import { randomIntBetween, randomItem } from 'https://jslib.k6.io/k6-utils/1.5.0/index.js';

const user = randomItem(users);
const delay = randomIntBetween(1, 5);
```

### Token/Session Reuse Across Iterations (per-VU state)
```javascript
let cachedToken = null;

export default function () {
  if (!cachedToken) {
    const res = http.post(/* login */);
    cachedToken = res.json('token');
  }
  http.get('http://localhost:3000/api/profile', {
    headers: { Authorization: `Bearer ${cachedToken}` },
  });
}
```
This module-scope variable persists across iterations **within the same VU** (it's re-initialized fresh per VU, since each VU runs its own isolated JS instance).

### Correlation (extracting a value from one response to use in the next)
```javascript
const createRes = http.post('http://localhost:3000/api/orders', /* ... */);
const orderId = createRes.json('id');
http.get(`http://localhost:3000/api/orders/${orderId}`, { tags: { name: 'get_order_by_id' } });
```

### Reusable Modules
```javascript
// helpers/auth.js
import http from 'k6/http';
export function login(baseUrl, email, password) {
  const res = http.post(`${baseUrl}/api/auth/login`, JSON.stringify({ email, password }), {
    headers: { 'Content-Type': 'application/json' },
  });
  return res.json('token');
}
```

### Environment Variables & Secrets
```bash
k6 run -e BASE_URL=https://staging.example.com -e API_KEY=$SECRET_KEY script.js
```
```javascript
const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';
```
Never hardcode secrets in scripts committed to source control — pass them via `-e` flags fed from your CI's secret store.

### Best Practices
- Use `SharedArray` for any dataset larger than a few KB — loading big JSON arrays as plain top-level variables duplicates that memory **per VU**, which can exhaust k6's own memory at high VU counts.
- Keep custom metrics focused — too many custom metrics create noisy, hard-to-read summaries.

### Common Mistakes
- Loading large JSON files without `SharedArray`, causing out-of-memory errors on high-VU tests.
- Hardcoding secrets/API keys directly in test scripts.

### Interview Questions
**Q: Why use `SharedArray` instead of a plain array for test data?**
A: A plain top-level array is copied into each VU's isolated JS runtime, multiplying memory use by VU count. `SharedArray` loads the data once and shares a read-only reference across all VUs, dramatically reducing memory overhead at scale.

**Q: How do you avoid hardcoding secrets in k6 scripts?**
A: Read them via `__ENV` variables passed at runtime with `-e KEY=value` (fed from a CI secret store or local `.env`/shell export), never committed directly into the script.

### Exercises
1. Convert a script using a large inline JSON array of test users into one using `SharedArray`.
2. Add a custom `Trend` metric that tracks only the checkout endpoint's duration, separate from the global `http_req_duration`.

---

## 21. CI/CD Integration

### GitHub Actions
```yaml
name: Load Test
on: [pull_request]
jobs:
  k6-load-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Start API
        run: npm ci && npm start &
      - name: Wait for API
        run: npx wait-on http://localhost:3000/health
      - name: Run k6
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/load/checkout.js
        env:
          BASE_URL: http://localhost:3000
      # k6-action fails the step (and thus the PR check) if thresholds fail
```

### GitLab CI
```yaml
load_test:
  image: grafana/k6:latest
  stage: test
  script:
    - k6 run --out json=results.json tests/load/checkout.js
  artifacts:
    paths: [results.json]
    when: always
```

### Jenkins (declarative pipeline)
```groovy
pipeline {
  agent any
  stages {
    stage('Load Test') {
      steps {
        sh 'docker run --rm -v $(pwd):/scripts grafana/k6 run /scripts/checkout.js'
      }
    }
  }
}
```

### Azure DevOps
```yaml
steps:
  - script: |
      docker run --rm -v $(System.DefaultWorkingDirectory):/scripts grafana/k6 run /scripts/checkout.js
    displayName: 'Run k6 Load Test'
```

### Failing the Pipeline on Threshold Breach
k6 exits non-zero automatically when a threshold fails — no extra scripting needed. Every CI system above treats a non-zero exit code from the run step as a failed job by default.

### Best Practices
- Run a **lightweight smoke-level load test** on every PR (fast feedback), and a **full-scale load test** on a schedule or pre-release, not on every commit — full-scale tests are slow and resource-heavy.
- Store historical results (as artifacts or in a time-series DB) so you can compare "is this PR slower than last week?" not just pass/fail in isolation.

### Common Mistakes
- Running expensive, long-duration load tests on every single commit, slowing down the whole team's CI feedback loop.
- Not pinning the k6 Docker image version in CI, causing unpredictable behavior changes on k6 upstream releases.

### Interview Questions
**Q: How does k6 signal a failure to a CI system?**
A: It exits with a non-zero process exit code whenever a configured threshold fails, which CI runners interpret as a failed step/job — no custom glue code required.

### Exercises
1. Add a GitHub Actions workflow that runs a smoke-level k6 test (low VUs, short duration, strict thresholds) on every pull request.

---

## 22. Running k6 in Docker

### Basic Run
```bash
docker run --rm -i grafana/k6 run - <script.js
# or, with the script mounted:
docker run --rm -v $(pwd):/scripts grafana/k6 run /scripts/script.js
```

### Docker Compose: k6 + Express App Under Test + Observability
```yaml
version: '3.8'
services:
  api:
    build: ./app
    ports: ["3000:3000"]
    environment:
      - NODE_ENV=production

  k6:
    image: grafana/k6
    depends_on: [api]
    volumes:
      - ./tests:/scripts
    entrypoint: ["k6", "run", "/scripts/load.js"]
    environment:
      - BASE_URL=http://api:3000   # container-to-container networking by service name

  influxdb:
    image: influxdb:1.8
    environment:
      - INFLUXDB_DB=k6

  grafana:
    image: grafana/grafana
    ports: ["3001:3000"]
    depends_on: [influxdb]
```

### Docker Networking Notes
- Inside Compose, containers reach each other by **service name**, not `localhost` — hence `http://api:3000` rather than `http://localhost:3000` from the `k6` service.
- If running k6 standalone against an app on your host machine, use `host.docker.internal` (Mac/Windows) instead of `localhost`, since `localhost` inside a container refers to the container itself.

### Best Practices
- Containerize both the app under test and k6 for CI runs — guarantees a consistent, reproducible test environment every time.
- Give the Docker host enough CPU/memory for k6 itself at high VU counts, or you'll bottleneck the *load generator*, not the API.

### Common Mistakes
- Using `localhost` from inside a k6 container to reach an app running on the host — this fails silently or connects to the wrong thing.
- Under-provisioning the Docker Desktop VM's resources, causing k6 numbers that reflect Docker's own limits, not your API's real capacity.

### Interview Questions
**Q: Why would `http://localhost:3000` fail when running k6 in a Docker container against an app also running in Docker?**
A: `localhost` inside a container refers to that container's own network namespace, not the host or sibling containers. Containers on the same Docker Compose network must address each other by service name (e.g., `http://api:3000`).

### Exercises
1. Write a `docker-compose.yml` running an Express app and a k6 load test against it, using service-name networking.

---

## 23. Testing Production Safely

Load testing production is sometimes necessary (staging never perfectly mirrors real traffic/data), but it's high-risk. Guardrails:

- **Traffic limits:** cap total test traffic to a known-safe percentage of normal peak capacity; never test without a pre-agreed ceiling.
- **Warm-up:** ramp gradually (`ramping-vus`/`ramping-arrival-rate`) rather than an instant full-load spike — this also matches how real traffic actually grows and avoids triggering false-positive autoscaling/alerting storms.
- **Ramp-up:** always start low and increase in stages, watching dashboards between stages, with a plan to abort.
- **Canary testing:** route test traffic to a single canary instance/percentage of the fleet first, isolating blast radius before wider rollout.
- **Blue/Green:** run the load test against the "green" (not-yet-live) environment before flipping traffic, avoiding any live-user risk at all.
- **Feature flags:** gate any risky code path being tested behind a flag you can instantly disable if the load test reveals a problem.
- **Monitoring:** have real-time dashboards and on-call awareness *before* starting — never run a production load test silently/unannounced.
- **Rollback:** have a tested, fast rollback/kill-switch plan ready before you start; know exactly how to stop the k6 run instantly (`Ctrl+C`, or `k6 run` with a hard `duration` cap as a safety net).

### Best Practices
- Always get sign-off from whoever owns production infrastructure/on-call before testing production directly.
- Prefer testing a realistic **staging replica** (same data volume, same infra topology) over production whenever feasible — it removes most of this risk category entirely.

### Common Mistakes
- Running an unannounced stress test against production and triggering a real incident/alert storm for the on-call team.
- No abort plan — realizing mid-test that thresholds are failing badly with no fast way to stop cleanly.

### Interview Questions
**Q: What precautions would you take before running a load test against a live production system?**
A: Get stakeholder/on-call sign-off, set explicit traffic caps well under known capacity, ramp gradually rather than spiking instantly, monitor real-time dashboards throughout, and have a pre-tested rollback/abort plan — ideally testing a canary or blue/green environment instead of full production traffic.

### Exercises
1. Draft a "production load test runbook" checklist for your team, covering sign-off, monitoring, and abort criteria.

---

## 24. Performance Engineering Best Practices

### Little's Law
```
L = λ × W

L = average number of requests "in the system" (concurrency)
λ = average arrival rate (requests/sec)
W = average time each request spends in the system (latency)
```
Practical use: if you know your target throughput (λ) and your measured average latency (W), Little's Law tells you the concurrency (L) — i.e., how many simultaneous connections/threads/workers you need to provision for. Conversely, if concurrency is capped (e.g., a connection pool of 20) and latency is 100ms, max sustainable throughput is `20 / 0.1s = 200 req/s` — pushing more traffic than that just queues, it doesn't increase real throughput.

### Load Profiles & Think Time
- **Load profile:** the shape of traffic over time (steady, ramping, spiky, diurnal). Real production traffic is rarely flat — model your tests after actual traffic patterns (e.g., from real analytics/APM data) when possible.
- **Think time:** the pause between a user's actions (reading a page before clicking). Omitting it turns a load test into an artificial stress test with unrealistically high concurrency-per-user.

### Arrival Rate vs Concurrency vs Throughput vs Latency
| Term | Meaning |
|---|---|
| Arrival rate | How fast new requests/users show up |
| Concurrency | How many requests are being processed *at once* |
| Throughput | How many requests are completed per unit time |
| Latency | How long each individual request takes |

These four are linked by Little's Law — you cannot design a system by looking at only one of them.

### Percentiles (revisited)
Design SLOs around P95/P99 latency, not averages (see Ch. 16) — this is a core performance engineering habit, not just a k6 reporting detail.

### Queueing Theory (intuition)
As utilization approaches 100% of a resource's capacity, queue length (and therefore latency) doesn't grow linearly — it grows **exponentially**. A system at 70% CPU utilization might have very low queueing delay; the same system at 95% utilization can have dramatically higher latency for a relatively small increase in load. This is why capacity planning targets headroom (e.g., "keep steady-state utilization under ~70%"), not "100% and hope."

### Connection Pools & Caching
- Size DB/HTTP connection pools deliberately based on Little's Law math (target throughput × average query time), not arbitrary defaults.
- Caching reduces effective latency (W) and thus increases sustainable throughput at fixed concurrency — but introduces cache-invalidation and cold-start considerations (Ch. 19).

### Horizontal vs Vertical Scaling vs Autoscaling
- **Vertical scaling:** bigger machine (more CPU/RAM) — simple, but has hard ceilings and no redundancy.
- **Horizontal scaling:** more machines/replicas behind a load balancer — better for redundancy and near-linear scaling, but requires stateless design (session storage, sticky-session avoidance).
- **Autoscaling:** dynamically adjusts replica count based on load signals (CPU, request rate, queue depth) — load testing is exactly how you validate your autoscaling thresholds/cooldowns actually respond fast enough before user-facing latency degrades.

### Best Practices
- Use load test results to inform real capacity planning numbers (max sustainable req/s, required replica count), not just a pass/fail check.
- Validate autoscaling behavior explicitly with a load test shaped like a real traffic spike, not just steady load.

### Common Mistakes
- Scaling reactively based on gut feeling ("let's just add more servers") instead of Little's Law-informed math.
- Autoscaling configured with a cooldown/threshold that reacts slower than real traffic spikes grow, causing a latency/error window every time.

### Interview Questions
**Q: Explain Little's Law and why it matters for capacity planning.**
A: `L = λ × W` relates concurrency, arrival rate, and latency. It lets you compute, from measured or target values of any two, the third — e.g., given a target throughput and measured average latency, you can calculate the required concurrency (connection pool size, worker count) to sustain it without unbounded queueing.

**Q: Why does latency increase non-linearly as utilization approaches 100%?**
A: Queueing theory shows that as a resource's utilization nears full capacity, the probability of requests arriving faster than they can be served rises sharply, causing queue lengths — and therefore wait time — to grow exponentially rather than linearly, which is why systems are typically designed with utilization headroom rather than running near 100%.

### Exercises
1. Given a target throughput of 300 req/s and an average query latency of 50ms, use Little's Law to calculate the minimum required DB connection pool size.
2. Explain, using queueing theory intuition, why a system that's "fine" at 60% CPU can fall over at 85% CPU.

---

## 25. Professional Project Structure

```
k6-tests/
├── tests/
│   ├── smoke/                  # fast, low-VU, run on every PR
│   │   └── health-check.js
│   ├── load/                   # realistic expected-traffic tests
│   │   ├── checkout-flow.js
│   │   └── product-browsing.js
│   ├── stress/                 # push-past-capacity tests
│   │   └── orders-stress.js
│   └── soak/                   # long-duration endurance tests
│       └── orders-soak.js
├── helpers/
│   ├── auth.js                 # login/token helpers
│   └── requests.js             # wrapped http.* calls with defaults
├── utils/
│   ├── data-generators.js      # random email/name/etc generators
│   └── assertions.js           # shared check() bundles
├── config/
│   ├── environments.js         # base URLs per env (dev/staging/prod)
│   └── thresholds.js           # shared threshold definitions
├── data/
│   ├── users.json              # SharedArray test data
│   └── products.csv
├── reports/                    # generated HTML/JSON output (gitignored)
├── scripts/
│   └── run-all.sh              # convenience runner
└── env/
    ├── .env.staging
    └── .env.production
```

### Why Each Folder Exists
- **`tests/`** organized by *test type*, not by feature, because CI pipelines gate on test type (smoke on every PR, soak nightly).
- **`helpers/`** — reusable logic that talks to the API (auth, common request wrappers) — mirrors your Express app's own `services/` layer.
- **`utils/`** — pure, stateless utility functions (no HTTP calls).
- **`config/`** — environment-specific and threshold config, kept out of test logic so the same script runs against dev/staging/prod by swapping config only.
- **`data/`** — static or generated test datasets, loaded via `SharedArray`.
- **`reports/`** — build artifacts, never committed to source control.
- **`env/`** — per-environment secrets/URLs, loaded via `-e` flags or a `.env` loader, never hardcoded in scripts.

### Best Practices
- Keep test scripts declarative and thin; push logic into `helpers/`/`utils/` so scripts read like a user journey, not a wall of boilerplate.
- One config source of truth per environment — never hardcode a staging URL inside a test file.

### Common Mistakes
- Mixing smoke, load, and stress tests in one undifferentiated `tests/` folder with no way to selectively run just the fast ones in CI.
- Committing `reports/` output to git, bloating repo history with generated artifacts.

### Interview Questions
**Q: Why separate smoke, load, stress, and soak tests into different folders/pipelines?**
A: They have very different runtime costs and purposes — smoke tests need to run in seconds on every PR for fast feedback, while stress/soak tests are expensive and slow, appropriate only for scheduled or pre-release runs; mixing them makes it impossible to gate CI appropriately.

### Exercises
1. Reorganize a flat folder of k6 scripts into this structure, extracting shared login logic into `helpers/auth.js`.

---

## 26. Complete Project: E-commerce API

### User Journey Script
```javascript
// tests/load/ecommerce-journey.js
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { SharedArray } from 'k6/data';
import { randomItem, randomIntBetween } from 'https://jslib.k6.io/k6-utils/1.5.0/index.js';

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';

const users = new SharedArray('users', function () {
  return JSON.parse(open('../../data/users.json'));
});

export const options = {
  scenarios: {
    shoppers: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '1m', target: 50 },
        { duration: '3m', target: 50 },
        { duration: '1m', target: 0 },
      ],
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<400'],
    http_req_failed: ['rate<0.01'],
    'http_req_duration{name:login}': ['p(95)<300'],
    'http_req_duration{name:checkout}': ['p(95)<600'],
  },
};

export default function () {
  const user = randomItem(users);
  let token;

  group('login', function () {
    const res = http.post(`${BASE_URL}/api/auth/login`, JSON.stringify({
      email: user.email, password: user.password,
    }), { headers: { 'Content-Type': 'application/json' }, tags: { name: 'login' } });
    check(res, { 'login ok': (r) => r.status === 200 });
    token = res.json('token');
  });

  const authHeaders = { headers: { Authorization: `Bearer ${token}` } };
  sleep(randomIntBetween(1, 2));

  group('browse catalog', function () {
    const res = http.get(`${BASE_URL}/api/products?page=1&limit=20`, { tags: { name: 'browse_catalog' } });
    check(res, { 'catalog ok': (r) => r.status === 200 });
  });
  sleep(randomIntBetween(1, 3));

  group('search', function () {
    const res = http.get(`${BASE_URL}/api/products?search=shoes`, { tags: { name: 'search' } });
    check(res, { 'search ok': (r) => r.status === 200 });
  });
  sleep(1);

  let cartId;
  group('add to cart', function () {
    const res = http.post(`${BASE_URL}/api/cart`, JSON.stringify({ productId: randomIntBetween(1, 1000), qty: 1 }),
      { headers: { ...authHeaders.headers, 'Content-Type': 'application/json' }, tags: { name: 'add_to_cart' } });
    check(res, { 'cart ok': (r) => r.status === 201 });
    cartId = res.json('id');
  });
  sleep(randomIntBetween(1, 2));

  group('checkout', function () {
    const res = http.post(`${BASE_URL}/api/orders`, JSON.stringify({ cartId }),
      { headers: { ...authHeaders.headers, 'Content-Type': 'application/json' }, tags: { name: 'checkout' } });
    check(res, { 'checkout ok': (r) => r.status === 201 });
  });

  group('payment', function () {
    const res = http.post(`${BASE_URL}/api/payments`, JSON.stringify({ cartId, method: 'card' }),
      { headers: { ...authHeaders.headers, 'Content-Type': 'application/json' }, tags: { name: 'payment' } });
    check(res, { 'payment ok': (r) => r.status === 200 });
  });

  sleep(randomIntBetween(2, 4));
}
```

### Admin/Reporting Path (separate scenario, lower traffic share)
```javascript
export function adminJourney() {
  group('admin: view all orders', function () {
    http.get(`${BASE_URL}/api/admin/orders?page=1`, { tags: { name: 'admin_orders' } });
  });
  group('admin: view users', function () {
    http.get(`${BASE_URL}/api/admin/users?page=1`, { tags: { name: 'admin_users' } });
  });
}
```

### Combined Multi-Scenario Config
```javascript
export const options = {
  scenarios: {
    shoppers: { executor: 'ramping-vus', exec: 'default', stages: [/* ... */] },
    admins: { executor: 'constant-vus', exec: 'adminJourney', vus: 2, duration: '5m' },
  },
};
```

This models realistic traffic composition: many shoppers, a handful of admins — rather than testing every endpoint at equal, unrealistic weight.

### Best Practices
- Weight scenario traffic to match real production proportions (e.g., 95% browsing/shopping, 5% admin).
- Tag every step (`name:`) so the HTML/Grafana report breaks down performance per business-critical step, not just globally.

### Common Mistakes
- Testing all endpoints at equal VU weight, producing a report that doesn't reflect real risk (e.g., an admin report page getting the same load as checkout).

### Exercises
1. Extend this script with a "search with no results" and "out of stock" edge case, verifying the API handles them gracefully under load.
2. Add a `stages`-based ramp that models a flash-sale spike (sudden 5x jump in shoppers for 2 minutes).

---

## 27. 50 Common Mistakes

1. Running a test with 0 `sleep()`, unintentionally stress-testing instead of load-testing.
2. Confusing VUs with real concurrent users.
3. Not using `SharedArray` for large test datasets — causing k6 to run out of memory.
4. Hardcoding a single test user, causing DB contention or duplicate-key errors.
5. Skipping `setup()` and logging in inside `default()`, overloading the auth endpoint.
6. Forgetting `Content-Type: application/json`, so Express never parses the body.
7. Sending a raw JS object instead of `JSON.stringify()`-ing the POST body.
8. Ignoring `check()` failures because they don't stop the test.
9. Using `fail()` everywhere, aborting iterations unnecessarily.
10. Only reporting average response time, hiding tail latency.
11. Never setting thresholds, so CI can never actually fail on regressions.
12. Setting thresholds so loose they're meaningless.
13. Using `constant-vus` when the goal was a fixed requests/sec target.
14. Setting `maxVUs` too low for arrival-rate executors, causing k6-side errors.
15. Not tagging dynamic URLs with `name`, exploding metric cardinality.
16. Load testing a dev-mode server (`NODE_ENV` unset) and drawing production conclusions.
17. Testing a single Node process when production runs multiple clustered instances.
18. Not warming caches before treating results as representative.
19. Running production load tests without stakeholder/on-call sign-off.
20. No abort/rollback plan before starting a production test.
21. Instant full-load spikes instead of gradual ramp-up.
22. Ignoring `http_req_failed` because average latency "looked fine."
23. Not correlating client-side k6 metrics with server-side CPU/memory/DB metrics.
24. Treating a load test that ran once as sufficient — performance regresses over time without continuous testing.
25. Running expensive full-scale tests on every single commit, slowing CI for everyone.
26. Not pinning the k6 version, causing inconsistent CI results across time.
27. Trying `npm install`-ing arbitrary Node packages into a k6 script.
28. Assuming k6 scripts support the full Node.js API (fs, native modules) — they don't.
29. Forgetting `localhost` inside a Docker container doesn't reach the host or sibling containers.
30. Not accounting for connection pool size when scaling VUs — hitting an artificial DB ceiling, not the app's real ceiling.
31. Missing DB indexes discovered only under load, not in functional testing.
32. Not testing for N+1 query patterns at realistic data volumes.
33. Testing with tiny seed datasets that don't reflect production data volume.
34. Confusing `http_req_duration` and `http_req_waiting` when diagnosing bottlenecks.
35. Not discarding response bodies (`discardResponseBodies`) on large-scale tests, bloating k6's own memory.
36. Committing secrets/API keys directly into test scripts.
37. Committing generated report artifacts into source control.
38. Mixing smoke, load, stress, and soak tests in one undifferentiated pipeline stage.
39. Not modeling realistic traffic composition (equal-weighting rarely used admin endpoints with checkout).
40. Skipping think time entirely, producing unrealistically high concurrency-per-user.
41. Not testing error/edge cases — only the happy path.
42. Assuming horizontal scaling fixes every bottleneck (it doesn't fix a shared DB connection pool limit).
43. Not validating autoscaling behavior with a load test shaped like real spike traffic.
44. Running soak tests too short to catch slow memory leaks.
45. Not checking for GC pause correlation with latency spikes.
46. Believing a passing load test in staging guarantees production behavior without checking data/infra parity.
47. Not versioning threshold changes, losing historical context for "why did we lower this SLA target?"
48. Overusing custom metrics until the summary output becomes unreadable noise.
49. Not using groups for multi-step journeys, making reports hard to interpret.
50. Treating k6 output as the final word without also checking server-side logs/APM for errors invisible to the client (e.g., 200 OK with a logged internal exception).

---

## 28. Interview Questions

### Beginner (30)

1. **What is k6?** — An open-source, developer-centric load testing tool, scripted in JavaScript, built in Go for high-performance load generation.
2. **What language are k6 scripts written in?** — JavaScript (ES2015+ subset, via the goja engine).
3. **What is a Virtual User (VU)?** — A simulated concurrent client executing the test script in a loop.
4. **What is an iteration?** — One complete execution of the default function by a VU.
5. **How do you install k6 on macOS?** — `brew install k6`.
6. **How do you run a k6 script?** — `k6 run script.js`.
7. **What module do you import for HTTP requests?** — `k6/http`.
8. **How do you send a GET request?** — `http.get(url)`.
9. **How do you add think time between requests?** — `sleep(seconds)` from the `k6` module.
10. **What does `check()` do?** — Records a pass/fail assertion without stopping the test.
11. **Does a failed `check()` stop the test?** — No.
12. **What does `fail()` do?** — Immediately aborts the current iteration.
13. **How do you send a JSON POST body?** — `JSON.stringify()` the payload and set `Content-Type: application/json`.
14. **What are `options` in a k6 script?** — An exported config object controlling VUs, duration, thresholds, scenarios, etc.
15. **What's the difference between `vus`/`duration` and `stages`?** — `vus`/`duration` is a flat, single-level load; `stages` ramps VUs up/down over defined time periods.
16. **What is a threshold?** — A pass/fail condition applied to a metric, e.g. `p(95)<300`.
17. **What happens if a threshold fails?** — k6 exits with a non-zero status code, and the summary marks it failed.
18. **What is `http_req_duration`?** — The total time for an HTTP request, from send to full response received.
19. **What is `http_req_failed`?** — A Rate metric representing the fraction of failed requests.
20. **What's the difference between `init` code and the `default` function?** — Init code runs once per VU at startup; the default function runs every iteration.
21. **What does `setup()` do?** — Runs once before the load test begins, for one-time setup (e.g., login, seeding data).
22. **What does `teardown()` do?** — Runs once after the test ends, for cleanup.
23. **How many times does `setup()` run in a 100-VU test?** — Once, total.
24. **What is a Trend metric used for?** — Tracking a distribution of values (e.g., custom timing data) with statistics like avg/min/max/percentiles.
25. **What is a Counter metric used for?** — Tracking a running cumulative total (e.g., total orders created).
26. **What is P95 latency?** — The value below which 95% of recorded response times fall.
27. **Why prefer P95 over average?** — Averages hide tail latency; P95 reflects the "typical worst case" experienced by a meaningful share of users.
28. **How do you run k6 in Docker?** — `docker run --rm -i grafana/k6 run - <script.js` or mount a volume with the script.
29. **What is a scenario in k6?** — A named configuration describing how VUs/iterations are scheduled via an executor.
30. **What HTTP methods does `k6/http` support?** — GET, POST, PUT, PATCH, DELETE (`del`), HEAD, OPTIONS.

### Intermediate (30)

1. **Difference between `constant-vus` and `constant-arrival-rate`?** — `constant-vus` holds a fixed worker count with throughput dependent on response time; `constant-arrival-rate` maintains a fixed request rate regardless of latency, auto-scaling VUs up to `maxVUs`.
2. **Why tag dynamic URLs with `name`?** — To prevent metric cardinality explosion by grouping requests to the same logical endpoint despite different IDs in the URL.
3. **What is `SharedArray` and why use it?** — A construct that loads data once and shares a read-only reference across all VUs, avoiding per-VU memory duplication.
4. **How do you reuse an auth token across iterations?** — Store it in a module-scope variable (persists per-VU across iterations) or fetch it once in `setup()`.
5. **What's the risk of logging in inside `default()`?** — It multiplies login load unrealistically and pollutes target-endpoint metrics with auth latency.
6. **How do groups affect reporting?** — They organize metrics hierarchically by logical step/journey, making multi-step flows easier to interpret.
7. **What does `discardResponseBodies` do, and why use it?** — Skips storing response bodies, reducing k6's own memory/CPU footprint at scale.
8. **How do you export results to Grafana?** — Stream metrics via `--out influxdb=...` or Prometheus remote-write, then visualize in Grafana connected to that data source.
9. **What is `http_req_waiting` and why does it matter?** — Time-to-first-byte, isolating server processing time from network transfer — key for backend bottleneck diagnosis.
10. **What's the difference between `ramping-vus` and `ramping-arrival-rate`?** — The former ramps concurrent workers; the latter ramps the target request rate directly, auto-managing VUs to hit it.
11. **How do custom metrics differ from built-in ones?** — They're explicitly defined (`Trend`, `Rate`, `Counter`, `Gauge`) to track domain-specific measurements not covered by k6's defaults.
12. **What is `abortOnFail` used for in thresholds?** — Immediately stops the test run when a critical threshold fails, saving time/resources on a clearly broken run.
13. **How do you parameterize test data from a CSV file?** — Parse it (e.g., with the `papaparse` jslib) inside a `SharedArray` loader function.
14. **Why avoid hardcoding secrets in scripts?** — Security risk if committed to source control; use `__ENV` variables passed via `-e` flags instead.
15. **What is Little's Law, briefly?** — `L = λ × W`; relates concurrency, arrival rate, and latency, letting you derive one from the other two for capacity planning.
16. **What is an N+1 query problem?** — Fetching a list then querying related data per-row individually instead of via a single join/batch query, multiplying DB round-trips.
17. **How do connection pool limits affect load test results?** — Latency can climb sharply once concurrent requests exceed the pool size, as requests queue for an available connection — an artificial ceiling unrelated to raw app performance.
18. **What's the difference between vertical and horizontal scaling?** — Vertical scaling adds resources to one machine; horizontal scaling adds more machine instances behind a load balancer.
19. **Why might horizontal scaling fail to fix a bottleneck?** — If the bottleneck is a shared resource (e.g., one DB's connection pool or the DB itself), adding more app servers doesn't relieve pressure on that shared resource.
20. **What's event loop lag, and why does it matter in Node.js load testing?** — The delay between scheduling and executing a callback; rising lag under load indicates blocking synchronous code degrading all concurrent request handling.
21. **How would you detect a memory leak via k6/soak testing?** — Run a long `constant-vus` soak test while monitoring server RSS memory; a steadily climbing, non-plateauing line indicates a leak.
22. **What's the purpose of `preAllocatedVUs` in arrival-rate executors?** — Reserves VU capacity upfront so k6 can scale to the target rate without runtime allocation overhead or hitting `maxVUs` errors.
23. **How do you make a k6 test runnable against multiple environments (dev/staging/prod)?** — Externalize base URLs/config via `__ENV` variables or a config file, avoiding hardcoded URLs in test logic.
24. **What is a canary deployment, in the context of safe production testing?** — Routing a small percentage of traffic (real or test) to a new/isolated instance first, limiting blast radius before wider rollout.
25. **Why is `NODE_ENV=production` important during load testing?** — It changes real Express/Node behavior (error verbosity, caching), so testing without it can give misleading performance numbers.
26. **What does a Rate metric represent?** — A fraction (0 to 1) of occurrences meeting some condition, like `http_req_failed` or `checks`.
27. **How can you make k6 fail a CI pipeline automatically?** — Define thresholds; k6 exits non-zero on threshold failure, which CI systems treat as a failed step by default.
28. **What's the danger of testing with unrealistically small seed datasets?** — Missing indexes, cache behavior, and query performance issues that only appear at real production data volume.
29. **How do you handle a JWT refresh flow in a k6 script?** — Detect a 401 response, call the refresh endpoint with the stored refresh token, update the token variable, and retry the original request.
30. **What's the benefit of using `constant-arrival-rate` for capacity planning tests specifically?** — It directly and reliably targets a defined requests/sec figure, matching how capacity/SLA targets are usually defined in the first place.

### Advanced (30)

1. **Explain how k6's Go-based architecture enables higher VU counts than JVM-based tools like JMeter.** — Go's goroutines are lightweight (KBs of stack, cooperatively scheduled by the Go runtime) compared to JMeter's one-OS-thread-per-user model, letting a single machine simulate far more concurrent virtual users with less memory/CPU overhead per VU.
2. **How would you design a load test to validate autoscaling configuration?** — Shape traffic (via `ramping-arrival-rate`) to mimic a real spike pattern the autoscaler must respond to, then measure whether latency/error thresholds stay within SLO during the scale-up window, tuning scaling thresholds/cooldowns based on observed lag.
3. **Explain queueing theory's relevance to setting utilization-based capacity targets.** — As utilization approaches 100% of a resource, queue length and wait time grow non-linearly (approaching infinity near saturation), so capacity plans intentionally target headroom (e.g., ~70% utilization) rather than 100%, to keep latency predictable.
4. **How do you isolate whether a bottleneck is CPU, I/O, or database-bound during a load test?** — Correlate `http_req_waiting` trends with server-side CPU%, event loop lag, and DB query timing/logs on the same timeline; high CPU with rising event loop lag suggests CPU-bound blocking code, while high wait time with low CPU suggests I/O/DB-bound behavior.
5. **Design a soak test strategy to catch both memory leaks and slow resource exhaustion (e.g., connection leaks).** — Run `constant-vus` at a moderate, sustainable load for several hours, sampling server RSS memory, open DB connections, and file descriptor counts at regular intervals; alert on any monotonically increasing trend that doesn't plateau.
6. **How would you structure a multi-scenario k6 test to reflect realistic production traffic composition?** — Define separate scenarios with different executors and VU/rate weights per traffic type (e.g., 90% browsing via `ramping-vus`, 8% checkout via `ramping-arrival-rate`, 2% admin via `constant-vus`), matching real analytics-derived proportions.
7. **What's the tradeoff of using `discardResponseBodies` and when would you not use it?** — It reduces k6's memory/CPU load significantly at scale, but you lose the ability to assert on response body content in checks; disable it selectively per-request when body validation is essential.
8. **Explain Little's Law's use in sizing a database connection pool.** — Given a target throughput λ (req/s) and measured average query latency W, the required concurrency L = λ × W tells you the minimum connections needed to avoid queueing beyond the pool; sizing below this creates an artificial bottleneck independent of the DB's actual capacity.
9. **How would you detect and diagnose GC-pause-induced latency spikes in a Node.js app under load?** — Run the app with `--trace-gc` (or an APM with GC instrumentation) alongside a k6 test, then correlate timestamps of major GC events with spikes in `http_req_duration`/`http_req_waiting`; frequent major GCs under high allocation rates point to excessive short-lived object creation in hot paths.
10. **What's the architectural difference between exporting metrics via InfluxDB vs Prometheus remote-write, and when would you choose each?** — InfluxDB is push-based and simpler for classic time-series storage but is in maintenance mode in some ecosystems; Prometheus remote-write integrates with the broader, actively developed Prometheus/Grafana ecosystem and is generally the more future-proof modern choice.
11. **How do you prevent metric cardinality explosion in a large-scale k6 test suite?** — Consistently tag dynamic URL segments with static `name` tags, avoid creating per-unique-value custom metrics, and periodically audit which tags actually feed dashboards/thresholds versus which are unused noise.
12. **Explain why an average-latency-only SLA is a bad practice, using a concrete distribution example.** — A bimodal distribution (e.g., 95% of requests at 50ms, 5% stuck at 5000ms due to pool exhaustion) can still yield a deceptively "fine" average (~295ms) while masking a serious tail-latency problem that a meaningful fraction of real users experience — P95/P99 reveal this, averages hide it.
13. **How would you approach load testing a system with external third-party dependencies (e.g., a payment gateway) without hammering the real third party?** — Use a local mock/stub server that mimics realistic latency and failure-rate characteristics of the real dependency (based on production observability data), so the load test measures your system's behavior under dependency latency without violating the third party's rate limits/ToS.
14. **What's the risk of running `constant-arrival-rate` with `maxVUs` set too low, and how does it manifest in results?** — k6 can't allocate enough VUs to sustain the configured rate once response times rise, so it starts logging dropped iterations/errors that reflect k6's own resource exhaustion rather than the target system's real behavior — skewing results to look artificially bad (or masking the true throughput ceiling).
15. **How do you validate that gzip compression is actually active under load, not just in manual testing?** — Inspect the `Content-Encoding` response header and compare `data_received` metrics with/without compression enabled in the test/app config; a load test with compression disabled can show misleadingly high network-time metrics.
16. **Explain the tradeoffs of testing against a staging environment versus production directly.** — Staging avoids risk to real users/revenue and allows more aggressive stress testing, but often lacks true data volume, traffic patterns, and infra topology parity, which can hide real bottlenecks (e.g., cache warm state, sharding behavior) that only appear at production scale — a well-maintained, production-parity staging environment mitigates this gap.
17. **How would you design a canary-based production load test?** — Route a small, capped percentage of test (or shadowed real) traffic specifically to one canary instance/environment behind the load balancer, monitor its metrics in isolation against baseline instances, and have an automatic rollback trigger if canary error rate/latency deviates beyond a defined threshold.
18. **What's the difference between horizontal scalability being "linear" versus hitting a shared-resource ceiling, and how would a load test reveal it?** — Linear scalability means throughput grows proportionally as replicas are added; a load test run at increasing replica counts that shows throughput plateauing despite more app instances usually reveals contention on a shared resource (single DB, single cache, shared connection pool) that isn't scaled alongside the app tier.
19. **How would you use k6 to specifically validate a rate-limiting middleware's correctness under load?** — Configure a scenario exceeding the configured rate limit deliberately, assert via `check()` that requests beyond the limit correctly return the expected status (e.g., 429) with appropriate headers (e.g., `Retry-After`), and verify legitimate traffic within the limit is unaffected.
20. **What is the significance of `http_req_blocked` climbing under high VU counts?** — It indicates VUs are waiting for a free connection slot (often due to `noConnectionReuse` misconfiguration or an OS-level file-descriptor/connection limit on the client or server side), which can itself be a false signal of "server slowness" that's actually a client-side or OS-level constraint.
21. **How would you structure custom metrics to track business-relevant KPIs during a load test (e.g., successful checkouts per second)?** — Define a `Counter` incremented only on a verified-successful checkout response (status + body validation), tagged appropriately, so the summary/dashboard directly answers "how many real business transactions succeeded per second," not just raw HTTP success rate.
22. **Explain the performance implications of synchronous bcrypt hashing in an authentication-heavy load test.** — `bcrypt.hashSync`/`compareSync` block the Node.js event loop for the hash's full computation time; under concurrent load, this serializes all authentication requests (and blocks unrelated requests too), making it a common, easily-overlooked CPU bottleneck — the fix is using the async variants.
23. **How would you approach comparing two API implementations' performance objectively using k6?** — Run identical scenarios (same executor, VU/rate profile, data, and duration) against both implementations in equivalent environments, then compare P95/P99 latency, error rate, and throughput at matched load levels — never compare runs with different load shapes or environments.
24. **What's a realistic strategy for load testing a system with strong eventual-consistency characteristics (e.g., testing that a created order eventually appears in a read replica)?** — Use polling with a bounded retry/backoff inside the check logic (or a dedicated consistency-lag custom Trend metric) rather than asserting immediate consistency, to measure and validate actual replication lag under load rather than producing false failures.
25. **How does k6's per-VU JS isolation affect script design decisions around shared state?** — Each VU runs its own JS engine instance, so module-scope variables aren't shared *across* VUs (only persist within a single VU's iterations) — any genuinely shared, read-only state must go through `SharedArray`, and genuinely cross-VU coordination (e.g., a global counter) requires external state (a metrics backend or an API call), not in-script variables.
26. **What load testing strategy would you use to validate a caching layer's effectiveness under realistic traffic?** — Model a realistic key-access distribution (e.g., Zipfian/power-law, matching real product-popularity patterns) rather than uniform random keys, since uniform random access defeats caching in a way real traffic wouldn't, giving a falsely pessimistic cache-hit rate.
27. **How would you performance-test a paginated endpoint to catch "deep pagination" degradation (e.g., OFFSET-based pagination slowing at high page numbers)?** — Deliberately include high-page-number requests in the test's data variation (not just page 1), and track duration as a function of page number as a custom tag/metric, since OFFSET-based SQL pagination often degrades linearly or worse with page depth, unlike cursor-based pagination.
28. **Explain how you'd validate that a load test's own client-side resource usage isn't the actual bottleneck.** — Monitor the k6 process's own CPU/memory/network usage (or split load across multiple k6 instances/distributed execution) during the run; if k6's own CPU is saturated or `http_req_blocked` is elevated for connection-availability reasons unrelated to the server, results reflect the load generator's limits, not the target system's.
29. **How would you design a test to validate zero-downtime deployment behavior under load?** — Run a sustained `constant-vus` or `constant-arrival-rate` load through a rolling deployment/restart of the app, tracking `http_req_failed` and any latency spikes during the exact deployment window, to confirm connection draining and health-check-gated rollout are working as intended.
30. **What's the argument for testing with production-realistic network conditions (latency, packet loss) rather than a same-datacenter/localhost setup?** — Same-datacenter tests can substantially understate real-world latency (mobile networks, geographic distance, intermediate proxies/CDNs), so results can be overly optimistic; realistic tests either run from geographically distributed load generators or deliberately inject representative network latency to validate true end-user experience.

---

## 29. Cheat Sheet

```
INSTALL           brew install k6 | choco install k6 | docker pull grafana/k6
RUN                k6 run script.js
RUN W/ ENV VAR     k6 run -e BASE_URL=https://api.example.com script.js
RUN W/ JSON OUT    k6 run --out json=results.json script.js
RUN W/ INFLUXDB    k6 run --out influxdb=http://localhost:8086/k6 script.js

IMPORTS
  import http from 'k6/http';
  import { check, sleep, group, fail } from 'k6';
  import { Trend, Rate, Counter, Gauge } from 'k6/metrics';
  import { SharedArray } from 'k6/data';

BASIC REQUEST
  http.get(url, params)
  http.post(url, body, params)   // body must be a string (JSON.stringify)
  http.put/patch/del/head/options(...)

CHECK
  check(res, { 'status 200': (r) => r.status === 200 });

THRESHOLDS
  thresholds: {
    http_req_duration: ['p(95)<300'],
    http_req_failed: ['rate<0.01'],
  }

LIFECYCLE
  export function setup() { return data; }
  export default function (data) { /* per-iteration */ }
  export function teardown(data) { /* once, at end */ }

SCENARIOS (key executors)
  constant-vus | ramping-vus | shared-iterations | per-vu-iterations
  constant-arrival-rate | ramping-arrival-rate

KEY METRICS
  http_req_duration   http_req_waiting   http_req_failed
  http_reqs           iterations         vus
  data_sent/received  checks

CUSTOM METRICS
  const t = new Trend('name'); t.add(value);
  const c = new Counter('name'); c.add(1);
  const r = new Rate('name'); r.add(true/false);

SHARED DATA
  const arr = new SharedArray('name', () => JSON.parse(open('./file.json')));

TAGS (avoid cardinality explosion)
  http.get(url, { tags: { name: 'get_product' } });

DOCKER
  docker run --rm -v $(pwd):/scripts grafana/k6 run /scripts/script.js

CI EXIT CODE
  k6 exits non-zero automatically on threshold failure
```

---

## 30. Glossary

- **Arrival Rate** — The rate at which new requests/iterations begin, independent of how long they take to complete.
- **Canary Testing** — Rolling out or testing against a small subset of infrastructure/traffic before wider release.
- **Capacity Testing** — Determining the maximum load a system can sustain while meeting acceptable performance criteria.
- **Check** — A k6 assertion (`check()`) that records pass/fail without halting execution.
- **Concurrency** — The number of requests/operations being processed simultaneously.
- **Connection Pool** — A managed set of reusable database (or HTTP) connections, sized to bound resource usage.
- **Endurance/Soak Testing** — Sustained load over an extended period, designed to catch leaks and slow degradation.
- **Error Budget** — The amount of allowable failure (100% minus your SLO) before you're out of compliance.
- **Executor** — The k6 algorithm controlling how VUs/iterations are scheduled over time within a scenario.
- **Event Loop Lag** — The delay between when a Node.js callback is scheduled and when it actually executes; a key indicator of blocking code under load.
- **Iteration** — One complete run through the default function by a single VU.
- **Latency** — The time taken for a single request to complete.
- **Little's Law** — `L = λ × W`; relates concurrency, arrival rate, and latency.
- **Load Profile** — The shape of traffic intensity over time during a test.
- **Load Testing** — Verifying system behavior under expected/peak traffic.
- **N+1 Query Problem** — Fetching a list then issuing one additional query per item instead of a single batched/joined query.
- **P95 / P99** — The 95th/99th percentile of a metric's distribution — a key tail-latency indicator.
- **Percentile** — A value below which a given percentage of observations fall.
- **Queueing Theory** — The mathematical study of waiting lines, explaining why latency grows non-linearly near resource saturation.
- **Scenario** — A named, configured traffic pattern combining an executor with timing/VU parameters.
- **SharedArray** — A k6 construct for loading data once and sharing it read-only across all VUs.
- **SLA (Service Level Agreement)** — A contractual performance/availability commitment.
- **SLO (Service Level Objective)** — An internal target designed to safely meet the SLA.
- **Spike Testing** — Testing behavior under a sudden, sharp burst of traffic.
- **Stress Testing** — Deliberately exceeding expected capacity to find the breaking point.
- **Think Time** — Simulated pause between a user's actions, mimicking real behavior.
- **Threshold** — A pass/fail condition applied to a k6 metric.
- **Throughput** — The number of completed operations per unit time.
- **Trend/Rate/Counter/Gauge** — k6's custom metric types for distributions, fractions, running totals, and point-in-time values respectively.
- **Virtual User (VU)** — A simulated concurrent client executing the test script.
- **Volume Testing** — Testing behavior under large data volumes (payloads or dataset size).

---

*End of guide. Treat this as a living document: as your API evolves, keep your k6 scripts, thresholds, and dashboards evolving alongside it.*
