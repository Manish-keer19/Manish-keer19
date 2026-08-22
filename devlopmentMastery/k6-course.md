# k6 Load Testing — Beginner to Advanced

A practical course for developers who already know JavaScript/Node.js/Express, and want to become capable of real-world API and website load testing with **k6**.

Every section follows the same pattern where useful:
**What is it? → Why do I need it? → Simple example → Node.js example → Command → Expected result → Common mistake**

Throughout the course we use **one small Node.js + Express API** as our test target. It's defined in Section 0 — build it once, use it everywhere.

---

## Table of Contents

0. [The practice API](#0-the-practice-api)
1. [What is performance testing and why k6](#1-what-is-performance-testing-and-why-k6)
2. [Install k6 and run your first test](#2-install-k6-and-run-your-first-test)
3. [k6 basic structure](#3-k6-basic-structure)
4. [Basic HTTP testing](#4-basic-http-testing)
5. [Responses and checks](#5-responses-and-checks)
6. [Important k6 metrics](#6-important-k6-metrics)
7. [Thresholds and PASS/FAIL testing](#7-thresholds-and-passfail-testing)
8. [sleep() and realistic user behavior](#8-sleep-and-realistic-user-behavior)
9. [Authentication and JWT testing](#9-authentication-and-jwt-testing)
10. [Test data and complete user workflows](#10-test-data-and-complete-user-workflows)
11. [Types of performance testing](#11-types-of-performance-testing)
12. [Load control](#12-load-control)
13. [Scenarios and executors](#13-scenarios-and-executors)
14. [VU-based vs arrival-rate testing](#14-vu-based-vs-arrival-rate-testing)
15. [Custom metrics](#15-custom-metrics)
16. [Groups and tags](#16-groups-and-tags)
17. [Analyzing results and finding bottlenecks](#17-analyzing-results-and-finding-bottlenecks)
18. [Testing the real Express API — full suite](#18-testing-the-real-express-api--full-suite)
19. [Prometheus + Grafana visualization](#19-prometheus--grafana-visualization)
20. [Professional performance reports](#20-professional-performance-reports)
21. [Running k6 in CI/CD](#21-running-k6-in-cicd)
22. [Final real-world project](#22-final-real-world-project)
23. [Cheat sheet](#23-cheat-sheet)

---

## 0. The practice API

A tiny in-memory Express API. Save as `server.js`.

```js
// server.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();
app.use(express.json());

const SECRET = 'dev-secret';
let users = [{ id: 1, name: 'Alice', email: 'alice@test.com', password: '1234' }];
let products = [{ id: 1, name: 'Keyboard', price: 49 }, { id: 2, name: 'Mouse', price: 19 }];
let orders = [];
let nextUserId = 2, nextOrderId = 1;

// auth
app.post('/auth/login', (req, res) => {
  const { email, password } = req.body;
  const user = users.find(u => u.email === email && u.password === password);
  if (!user) return res.status(401).json({ error: 'invalid credentials' });
  const token = jwt.sign({ id: user.id, email: user.email }, SECRET, { expiresIn: '1h' });
  res.json({ token });
});

function auth(req, res, next) {
  const header = req.headers.authorization || '';
  const token = header.replace('Bearer ', '');
  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'unauthorized' });
  }
}

// users
app.get('/users', (req, res) => res.json(users));
app.post('/users', (req, res) => {
  const user = { id: nextUserId++, ...req.body };
  users.push(user);
  res.status(201).json(user);
});

// products
app.get('/products', (req, res) => {
  const { minPrice } = req.query;
  const result = minPrice ? products.filter(p => p.price >= Number(minPrice)) : products;
  res.json(result);
});

// orders (protected)
app.post('/orders', auth, (req, res) => {
  const order = { id: nextOrderId++, userId: req.user.id, ...req.body, status: 'created' };
  orders.push(order);
  res.status(201).json(order);
});
app.get('/orders', auth, (req, res) => res.json(orders.filter(o => o.userId === req.user.id)));
app.put('/orders/:id', auth, (req, res) => {
  const order = orders.find(o => o.id === Number(req.params.id));
  if (!order) return res.status(404).json({ error: 'not found' });
  Object.assign(order, req.body);
  res.json(order);
});
app.delete('/orders/:id', auth, (req, res) => {
  orders = orders.filter(o => o.id !== Number(req.params.id));
  res.status(204).end();
});

app.listen(3000, () => console.log('API running on http://localhost:3000'));
```

```bash
npm init -y
npm install express jsonwebtoken
node server.js
```

Keep this running in one terminal for the whole course; run k6 in another.

---

## 1. What is performance testing and why k6

**What is it?**
Performance testing answers one question: *"How does my system behave under real traffic?"* It's not about whether the code works (that's functional testing) — it's about whether it works **fast enough, for enough people, for long enough**.

**Why is it needed?**
Code that works fine for 1 user can fall over at 500 users — slow queries, connection pool exhaustion, memory leaks, CPU saturation. You want to find that *before* production does.

**Why k6 specifically?**
- Test scripts are plain **JavaScript** (you already know this).
- Lightweight — a single k6 process can simulate thousands of virtual users, unlike browser-based tools.
- Built for **CI/CD**: scriptable, exits with pass/fail codes.
- First-class support for **thresholds** (automatic pass/fail), metrics, and integrations (Grafana, Prometheus, InfluxDB).

**Mental model — keep this picture in your head for the whole course:**

```
k6
 ↓
options            (how much load, how long, what rules)
 ↓
scenario/executor   (the traffic pattern engine)
 ↓
VU                  (a virtual user — a worker running your script)
 ↓
iteration           (one full run of your script by one VU)
 ↓
HTTP requests        (the calls your script makes)
 ↓
response            (status, body, timing)
 ↓
metrics             (numbers k6 collects: duration, count, rate...)
 ↓
checks              (did the response look correct?)
 ↓
thresholds          (is the performance good enough?)
 ↓
PASS/FAIL
```

Keep this diagram in mind — every section below fills in one layer of it.

**Common mistake:** Treating a load test as "does it return 200?" — that's just a functional check. Real load testing cares about *behavior under concurrency and time*.

---

## 2. Install k6 and run your first test

**Install**

```bash
# macOS
brew install k6

# Windows
choco install k6

# Linux (Debian/Ubuntu)
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update && sudo apt-get install k6

# Or just use Docker — works everywhere
docker pull grafana/k6
```

**Your first script** — `hello.js`:

```js
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
  http.get('https://test.k6.io');
  sleep(1);
}
```

**Run it:**

```bash
k6 run hello.js
```

**Expected result:** k6 prints a summary — request counts, response time percentiles, data transferred, and (since we didn't set VUs/duration) it ran with **1 VU for 1 iteration**.

**Common mistake:** Expecting k6 to open a real browser. It doesn't render pages or JS in the browser — it makes raw HTTP calls. (There's a separate `k6 browser` module for real browser testing, out of scope for this course since it's rarely needed for API load testing.)

---

## 3. k6 basic structure

Every k6 script has the same skeleton:

```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '30s',
};

export default function () {
  http.get('http://localhost:3000/products');
  sleep(1);
}
```

### `k6 run`
The command that executes a script file. `k6 run script.js`.

### `options`
A config object exported from the script. Tells k6 **how much load to generate and for how long**, plus thresholds, tags, etc. Think of it as the test's settings panel.

### `default()`
The function every **VU** executes, over and over, for each **iteration**. This is your actual test logic — the "user journey."

### VU (Virtual User)
A VU is a lightweight worker (like a thread) that runs your `default()` function repeatedly. 10 VUs = 10 independent simulated users hitting your app at the same time.

### Iteration
**One full execution of `default()`** by one VU, start to finish. If a VU runs `default()` 5 times, that's 5 iterations.

### HTTP request
A single network call made with `http.get()`, `http.post()`, etc. One iteration can contain many requests (e.g., login, then fetch products, then place order = 3 requests, 1 iteration).

**The critical distinction — VU vs iteration vs request:**

| Concept | What it means | Example |
|---|---|---|
| **VU** | A simulated user (worker) | 50 VUs = 50 concurrent users |
| **Iteration** | One pass through `default()` by a VU | Each VU may run `default()` many times during the test |
| **Request** | One HTTP call | One iteration might fire 3 requests (login → browse → order) |

So: **1 VU can produce many iterations, and 1 iteration can produce many requests.** Don't confuse "VUs" with "requests per second" — a VU with a 3-second `sleep()` produces far fewer requests/sec than one with no sleep.

**Command to run:**
```bash
k6 run script.js
```

**Expected result:** Summary showing `vus`, `iterations`, `http_reqs`, and timing stats.

**Common mistake:** Assuming more VUs always means proportionally more load. If each iteration is slow (e.g., waiting on a slow endpoint), throughput doesn't scale linearly with VUs — see [Section 14](#14-vu-based-vs-arrival-rate-testing).

---

## 4. Basic HTTP testing

All examples below hit our practice API at `http://localhost:3000`.

### GET
```js
import http from 'k6/http';
import { check } from 'k6';

export default function () {
  const res = http.get('http://localhost:3000/products');
  check(res, { 'status is 200': (r) => r.status === 200 });
}
```

### GET with query parameters
```js
http.get('http://localhost:3000/products?minPrice=20');
```

### POST with JSON body and headers
```js
const payload = JSON.stringify({ name: 'Bob', email: 'bob@test.com', password: 'pass' });
const params = { headers: { 'Content-Type': 'application/json' } };

const res = http.post('http://localhost:3000/users', payload, params);
```

### PUT
```js
const res = http.put(
  'http://localhost:3000/orders/1',
  JSON.stringify({ status: 'shipped' }),
  { headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` } }
);
```

### DELETE
```js
http.del('http://localhost:3000/orders/1', null, {
  headers: { Authorization: `Bearer ${token}` },
});
```

**Command:** `k6 run script.js`

**Expected result:** Each `check()` line reports pass % in the summary; `http_req_duration` shows timing.

**Common mistake:** Forgetting `Content-Type: application/json` on POST/PUT — Express's `express.json()` middleware will silently leave `req.body` empty, and your test will "pass" against an API that isn't actually doing what you think.

---

## 5. Responses and checks

**What is it?**
`check()` validates that a response is *correct* — right status code, right field in the body, right header. It does **not** fail the test or stop execution — it just records a pass/fail count.

**Why is it needed?**
A fast response that returns the wrong data (or a 500 error) is not a successful request. Checks catch functional problems that appear only under load (e.g., race conditions).

```js
import http from 'k6/http';
import { check } from 'k6';

export default function () {
  const res = http.get('http://localhost:3000/products');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'has products': (r) => JSON.parse(r.body).length > 0,
    'response time < 300ms': (r) => r.timings.duration < 300,
  });
}
```

**Node example (against our API):**
```js
const res = http.post(
  'http://localhost:3000/auth/login',
  JSON.stringify({ email: 'alice@test.com', password: '1234' }),
  { headers: { 'Content-Type': 'application/json' } }
);
check(res, {
  'login succeeded': (r) => r.status === 200,
  'token returned': (r) => JSON.parse(r.body).token !== undefined,
});
```

**Command:** `k6 run script.js`

**Expected result:** Summary shows `checks.........: 100.00% ✓ 30 ✗ 0` (or the failing %).

**Common mistake:** Wrapping `JSON.parse()` unsafely — if a request fails and returns HTML/empty body, `JSON.parse` throws and crashes the check function silently as a failure. Guard with `r.status === 200 &&` first, using short-circuit evaluation.

> **Rule to remember:** `check()` = correctness ("did we get the right answer?"). `threshold` (next section) = performance requirement ("was it fast/reliable enough overall?").

---

## 6. Important k6 metrics

k6 automatically collects these — you don't need to compute them yourself.

| Metric | Meaning |
|---|---|
| **http_reqs** | Total request count |
| **requests/sec** | Throughput — `http_reqs` ÷ test duration |
| **http_req_duration** | Response time per request (this is where avg/p90/p95/p99 come from) |
| **avg** | Mean response time — easily skewed by outliers, not very reliable alone |
| **p90 / p95 / p99** | 90/95/99th percentile response time — "X% of requests were faster than this" |
| **http_req_failed** | Error rate — % of requests that failed |
| **vus / vus_max** | Current / max concurrent virtual users |
| **iterations** | Total completed iterations |

**Why percentiles matter more than average:**
Average hides pain. If 95 requests take 100ms and 5 take 5000ms, the average looks fine (~340ms) but 5% of your real users are suffering. **p95/p99 represent user experience under slower conditions** — the "tail" of unlucky or heavy requests. Professional load testing is judged on p95/p99, not average.

```bash
k6 run script.js
```

**Expected output snippet:**
```
http_req_duration..............: avg=45ms min=12ms med=38ms max=890ms p(90)=72ms p(95)=105ms
http_req_failed.................: 0.42%  ✓ 3  ✗ 717
http_reqs........................: 720   24.1/s
vus...............................: 20    min=20 max=20
iterations.........................: 720   24.1/s
```

**Common mistake:** Reporting only the average response time to stakeholders. Always report **p95 and p99** — they tell the real story.

---

## 7. Thresholds and PASS/FAIL testing

**What is it?**
Thresholds are pass/fail rules attached to metrics, defined in `options`. If a threshold is breached, `k6 run` exits with a non-zero code — perfect for CI/CD gates.

**Why is it needed?**
Without thresholds you have to eyeball a summary and decide "was this good?" manually. Thresholds encode your **performance requirements as code**.

```js
export const options = {
  vus: 20,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<300', 'p(99)<600'], // 95% under 300ms, 99% under 600ms
    http_req_failed: ['rate<0.01'],                 // error rate under 1%
    checks: ['rate>0.99'],                           // 99%+ checks pass
  },
};
```

**Command:** `k6 run script.js`

**Expected result:** Thresholds that pass show `✓`; failing ones show `✗` and the process exits with code `99` (non-zero), letting CI pipelines fail the build automatically.

```
✗ http_req_duration..............: p(95)=410ms p(99)=720ms
    ↳ p(95)<300 ... FAILED
```

**Common mistake:** Setting thresholds so loose they never fail (defeats the purpose) — or so tight they fail on your dev laptop where results are noisy. Base thresholds on real SLAs/SLOs, tested against a stable staging environment.

---

## 8. sleep() and realistic user behavior

**What is it?**
`sleep(seconds)` pauses the VU inside an iteration — simulating the time a real user spends reading a page, thinking, or filling a form.

**Why is it needed?**
Without `sleep()`, VUs hammer your API as fast as physically possible — not realistic, and it produces a much higher requests/sec than real users ever would, giving misleading results.

```js
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
  http.get('http://localhost:3000/products');
  sleep(Math.random() * 2 + 1); // pause 1–3s, like a real user browsing
}
```

**Command:** `k6 run script.js`

**Expected result:** Lower, more realistic `requests/sec` compared to a no-sleep version with the same VU count.

**Common mistake:** Forgetting `sleep()` entirely and then panicking when "10 VUs produced 5000 req/s" — that's not 10 real users, that's a tight loop.

---

## 9. Authentication and JWT testing

**What is it?**
Most real APIs require a token per user. In k6, you log in **once per VU/iteration**, capture the token, and reuse it in subsequent requests — exactly like a browser or mobile client would.

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function () {
  // 1. Login
  const loginRes = http.post(
    'http://localhost:3000/auth/login',
    JSON.stringify({ email: 'alice@test.com', password: '1234' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(loginRes, { 'logged in': (r) => r.status === 200 });
  const token = JSON.parse(loginRes.body).token;

  // 2. Use token for a protected request
  const ordersRes = http.get('http://localhost:3000/orders', {
    headers: { Authorization: `Bearer ${token}` },
  });
  check(ordersRes, { 'orders fetched': (r) => r.status === 200 });

  sleep(1);
}
```

**Command:** `k6 run script.js`

**Expected result:** Both checks pass; `/orders` returns `200` instead of `401`.

**Common mistake:** Logging in **inside a loop of requests** instead of once per iteration — this wastes capacity hammering the login endpoint instead of the endpoints you actually want to test. For high VU counts, also consider pre-generating tokens (Section 10) to avoid making your login endpoint the bottleneck of the whole test.

---

## 10. Test data and complete user workflows

**What is it?**
Real tests need varied data per VU (different users/emails), not one hardcoded value — otherwise you're testing one code path and possibly hitting artificial unique-constraint errors.

**Using `SharedArray` for test data (loaded once, shared across VUs, memory-efficient):**

```js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { SharedArray } from 'k6/data';

const users = new SharedArray('users', function () {
  return JSON.parse(open('./users.json')); // [{ email, password }, ...]
});

export default function () {
  const user = users[Math.floor(Math.random() * users.length)];

  const loginRes = http.post(
    'http://localhost:3000/auth/login',
    JSON.stringify(user),
    { headers: { 'Content-Type': 'application/json' } }
  );
  const token = JSON.parse(loginRes.body).token;
  check(loginRes, { 'login ok': (r) => r.status === 200 });

  sleep(1);
}
```

**A full workflow — simulating a real customer journey (login → browse → order):**

```js
export default function () {
  // Login
  const token = JSON.parse(
    http.post('http://localhost:3000/auth/login', JSON.stringify({ email: 'alice@test.com', password: '1234' }),
      { headers: { 'Content-Type': 'application/json' } }).body
  ).token;
  const authHeaders = { headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` } };

  sleep(1); // user reads the homepage

  // Browse products
  const products = JSON.parse(http.get('http://localhost:3000/products').body);
  sleep(2); // user thinks about what to buy

  // Place order
  const order = http.post('http://localhost:3000/orders',
    JSON.stringify({ productId: products[0].id, qty: 1 }), authHeaders);
  check(order, { 'order created': (r) => r.status === 201 });

  sleep(1);
}
```

**Command:** `k6 run --iterations 50 workflow.js`

**Expected result:** Each iteration performs a realistic multi-step journey; check the summary for per-check pass rates.

**Common mistake:** Using the **same single test user** for all VUs when the workflow creates data (e.g., orders) — this can create unrealistic contention or unique-key collisions that don't happen with real, distinct users.

---

## 11. Types of performance testing

| Type | Goal | Typical shape |
|---|---|---|
| **Smoke** | Sanity check — does the system work at minimal load? | 1–2 VUs, 1 min |
| **Baseline/Load** | Normal expected traffic — measure typical performance | e.g., 50 VUs, 10 min |
| **Stress** | Find the breaking point — push beyond normal | Ramp VUs up until errors/latency spike |
| **Spike** | Sudden burst of traffic (flash sale, viral post) | Jump from low → very high VUs quickly |
| **Soak** | Long-duration steady load — find memory leaks, slow degradation | Moderate VUs, 1–12+ hours |

### Smoke test
```js
export const options = { vus: 1, duration: '1m',
  thresholds: { http_req_failed: ['rate<0.01'] } };
```
**Why:** Runs before anything else — catches broken scripts/endpoints cheaply before spending time on a full load test.

### Baseline / Load test
```js
export const options = { vus: 50, duration: '10m',
  thresholds: { http_req_duration: ['p(95)<300'], http_req_failed: ['rate<0.01'] } };
```
**Why:** Confirms the system meets its SLA under expected, everyday traffic.

### Stress test
```js
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 300 },
    { duration: '5m', target: 600 },
    { duration: '2m', target: 0 },
  ],
};
```
**Why:** Reveals the point where response time or error rate becomes unacceptable — your real capacity ceiling.

### Spike test
```js
export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '10s', target: 500 }, // sudden spike
    { duration: '1m', target: 500 },
    { duration: '30s', target: 20 },
  ],
};
```
**Why:** Tests recovery behavior — does the system recover gracefully after the spike, or does it stay degraded/crash?

### Soak test
```js
export const options = { vus: 50, duration: '2h',
  thresholds: { http_req_duration: ['p(95)<300'] } };
```
**Why:** Catches issues that only appear over time — memory leaks, connection pool exhaustion, disk filling with logs.

**Command for any of these:** `k6 run <test-type>.js`

**Common mistake:** Only ever running load tests and skipping stress/soak — those two catch the most damaging production incidents (capacity failures, slow memory leaks).

---

## 12. Load control

### `vus`
Fixed number of concurrent virtual users for the whole test.
```js
export const options = { vus: 20, duration: '1m' };
```

### `duration`
How long the test runs (used with a fixed `vus`).

### `stages`
Define a **ramping** VU profile — a list of `{ duration, target }` steps. k6 gradually moves VU count toward each `target` over each stage's `duration`.
```js
export const options = {
  stages: [
    { duration: '1m', target: 50 },   // ramp up
    { duration: '3m', target: 50 },   // stay steady
    { duration: '1m', target: 0 },    // ramp down
  ],
};
```

### Ramping VUs (visual)
```
VUs
 50 |        ______________
    |       /              \
    |      /                \
  0 |_____/                  \____
    0    1m       4m          5m   time
```

**Why ramp instead of jumping straight to max VUs?** Real traffic builds up gradually (morning login rush, etc.), and ramping helps you see **at what VU count** problems start — a straight jump to 50 tells you pass/fail but not *where* things degrade.

**Command:** `k6 run script.js`

**Expected result:** Summary shows `vus_max` matching your highest stage target; the test duration equals the sum of all stage durations.

**Common mistake:** Using both `vus`/`duration` AND `stages` in the same `options` — k6 will use `stages` and ignore the flat `vus`/`duration`, silently confusing people who forgot to remove the old config.

---

## 13. Scenarios and executors

**What is it?**
`scenarios` let you define **multiple independent traffic patterns** in a single test (e.g., "50 users browsing" + "10 users checking out"), each with its own **executor** — the algorithm that controls how VUs/iterations are scheduled.

**Why is it needed?**
Flat `vus`/`stages` only gives you one pattern. Real systems face multiple simultaneous, differently-shaped traffic types. Scenarios also let you target *different endpoints* with independent load profiles in one script/run.

The executors that matter day-to-day:

| Executor | What it does | When to use |
|---|---|---|
| **constant-vus** | Fixed VUs for a fixed duration | Simple steady load test |
| **ramping-vus** | VUs change over stages | Simulating gradual traffic build-up |
| **shared-iterations** | Fixed total iteration count, shared across VUs | "Run exactly 1000 requests total, as fast as N VUs allow" |
| **per-vu-iterations** | Each VU runs a fixed number of iterations | "Every VU must do exactly 10 iterations" |
| **constant-arrival-rate** | Fixed number of **iterations started per second**, regardless of VUs needed | Real-world "X requests/sec" targets |
| **ramping-arrival-rate** | Iterations/sec ramps over stages | Simulating traffic growing to a target throughput |

```js
export const options = {
  scenarios: {
    browsing: {
      executor: 'constant-vus',
      vus: 30,
      duration: '5m',
      exec: 'browse',
    },
    checkout: {
      executor: 'ramping-arrival-rate',
      startRate: 5,
      timeUnit: '1s',
      preAllocatedVUs: 50,
      maxVUs: 200,
      stages: [
        { duration: '2m', target: 20 },
        { duration: '3m', target: 20 },
      ],
      exec: 'checkout',
    },
  },
};

export function browse() { /* ... */ }
export function checkout() { /* ... */ }
```

**Command:** `k6 run script.js`

**Expected result:** Summary breaks down metrics per scenario tag, letting you see `browsing` vs `checkout` performance separately.

**Common mistake:** Forgetting `exec` — without it, every scenario runs the same `default()` function, defeating the purpose of having separate scenarios.

---

## 14. VU-based vs arrival-rate testing

This is one of the most important — and most misunderstood — concepts in k6.

**VU-based load** (`constant-vus`, `ramping-vus`, `shared-iterations`, `per-vu-iterations`):
You control the **number of virtual users**. Throughput (requests/sec) is a *side effect* — it depends on how fast your API responds. If your API gets slower, each VU completes fewer iterations, so **req/s actually drops** even though VU count stays the same. This can hide performance problems.

**Arrival-rate load** (`constant-arrival-rate`, `ramping-arrival-rate`):
You directly control **how much work starts per second**, independent of response time. k6 automatically adds more VUs (up to `maxVUs`) to sustain that rate. This is how real-world traffic actually behaves — users don't wait for your server to be ready; new requests arrive on their own schedule.

```
VU-based:            "I have 50 users clicking as fast as they can."
                      → throughput depends on response speed.

Arrival-rate:         "Exactly 100 requests must start every second."
                      → k6 adds VUs as needed to hit that number.
```

**Example — arrival rate:**
```js
export const options = {
  scenarios: {
    steady_rps: {
      executor: 'constant-arrival-rate',
      rate: 100,            // 100 iterations
      timeUnit: '1s',        // per second
      duration: '5m',
      preAllocatedVUs: 100,  // VUs reserved upfront
      maxVUs: 300,           // hard ceiling if API slows down
    },
  },
};
```

**When to use which:**
- Use **VU-based** when modeling "X concurrent users browsing" (typical for internal apps, dashboards).
- Use **arrival-rate** when you have a real, known target like "we need to handle 200 checkout requests/sec" (typical for public APIs, e-commerce).

**Command:** `k6 run script.js`

**Expected result:** With arrival-rate, `http_reqs`/duration stays close to your configured `rate` even as response times vary — until `maxVUs` is exhausted, at which point k6 reports **dropped iterations**.

**Common mistake:** Using VU-based load to test a specific "requests per second" SLA — VU count doesn't reliably map to req/s, so you end up guessing and re-tuning VUs instead of testing the real target directly.

---

## 15. Custom metrics

**What is it?**
Beyond the built-in metrics, you can track your own — e.g., time spent only in the "checkout" step, or a business counter like "orders placed."

```js
import http from 'k6/http';
import { Trend, Counter, Rate } from 'k6/metrics';

const checkoutTime = new Trend('checkout_duration');
const ordersPlaced = new Counter('orders_placed');
const loginFailureRate = new Rate('login_failures');

export default function () {
  const loginRes = http.post('http://localhost:3000/auth/login',
    JSON.stringify({ email: 'alice@test.com', password: '1234' }),
    { headers: { 'Content-Type': 'application/json' } });
  loginFailureRate.add(loginRes.status !== 200);

  const start = Date.now();
  const orderRes = http.post('http://localhost:3000/orders', JSON.stringify({ productId: 1 }),
    { headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${JSON.parse(loginRes.body).token}` } });
  checkoutTime.add(Date.now() - start);

  if (orderRes.status === 201) ordersPlaced.add(1);
}
```

Add thresholds on custom metrics too:
```js
export const options = {
  thresholds: {
    checkout_duration: ['p(95)<500'],
    login_failures: ['rate<0.02'],
  },
};
```

**Command:** `k6 run script.js`

**Expected result:** Custom metrics appear in the summary alongside built-ins, e.g. `checkout_duration...: avg=210ms p(95)=430ms`.

**Common mistake:** Creating a new `Trend`/`Counter` instance *inside* `default()` — metric objects must be created **once, at the module level**, or k6 will not aggregate them correctly across iterations.

---

## 16. Groups and tags

**Groups** — organize requests inside a script for clearer reporting:
```js
import { group } from 'k6';

export default function () {
  group('login', function () {
    http.post('http://localhost:3000/auth/login', /* ... */);
  });
  group('browse products', function () {
    http.get('http://localhost:3000/products');
  });
}
```

**Tags** — attach metadata to requests for filtering/grouping in results or dashboards:
```js
http.get('http://localhost:3000/products', { tags: { endpoint: 'products' } });
```

**Why is it needed?** In a script with 10+ requests, a flat summary is hard to read. Groups/tags let you see "how did the *login* step perform?" separately from "how did *checkout* perform?" — critical when exporting to Grafana too.

**Command:** `k6 run script.js`

**Expected result:** Summary output is broken into indented sections per group.

**Common mistake:** Over-nesting groups (groups inside groups inside groups) — it clutters the summary more than it helps. One level is usually enough.

---

## 17. Analyzing results and finding bottlenecks

**Reading the summary — top-down approach:**

1. **Check thresholds first** — did anything fail? That's your headline.
2. **Check `http_req_failed`** — is the error rate elevated? If yes, look at error types before even looking at speed.
3. **Check `http_req_duration` p95/p99** — is latency high, or just a few outliers?
4. **Check `iterations` and `dropped_iterations`** — did k6 actually generate the load you asked for?
5. **Break down by endpoint/group** — which specific request is slow, not just "the system."

**Common problems and how to investigate:**

| Symptom | Likely cause | How to investigate |
|---|---|---|
| **High p95/p99, low avg** | A subset of requests hit a slow path (cold cache, GC pause, lock contention) | Compare p50 vs p95/p99 gap; check app logs/APM for slow traces during that time window |
| **High error rate** | App crashing under load, timeouts, validation errors from bad test data | Look at response bodies/status codes in k6 logs (`--http-debug`); check server logs simultaneously |
| **Low throughput despite many VUs** | Server-side bottleneck limiting concurrency, or VUs blocked waiting on slow responses | Check server CPU/memory; check if arrival-rate test (not VU-based) shows the same ceiling |
| **Response time increases over test duration** | Memory leak, growing queue/log file, DB connections not released | Run a soak test; monitor memory/CPU trend over time, not just final numbers |
| **CPU bottleneck** | App/DB maxing out CPU | Watch `top`/`htop` or cloud metrics during the test; scale up or optimize hot code paths |
| **Memory problems** | Leak or unbounded cache growth | Watch RSS memory over a soak test; look for it climbing and never releasing |
| **Database bottleneck** | Slow queries, missing indexes, lock contention | Enable slow query logs; compare API response time to raw DB query time |
| **Connection pool exhaustion** | Too few DB/HTTP connections for concurrent load | Check pool size vs concurrent VUs; look for "pool timeout" errors in server logs |
| **Dropped iterations** (arrival-rate) | `maxVUs` too low — k6 ran out of VUs to sustain the requested rate | Increase `preAllocatedVUs`/`maxVUs`, or accept that the target rate isn't achievable with current server capacity |

**Command for deeper request-level detail:**
```bash
k6 run --http-debug="full" script.js   # print every request/response (use on small tests only)
k6 run --out json=results.json script.js  # export every data point for offline analysis
```

**Common mistake:** Diagnosing purely from k6's numbers without looking at the **server side** at the same time (CPU, memory, DB, logs). k6 tells you *that* something is slow — the server tells you *why*.

---

## 18. Testing the real Express API — full suite

Full smoke/baseline/stress/spike/soak tests against our Section 0 API. Save as separate files.

**`smoke.js`**
```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 2, duration: '30s',
  thresholds: { http_req_failed: ['rate<0.01'], http_req_duration: ['p(95)<500'] } };

export default function () {
  const res = http.get('http://localhost:3000/products');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

**`baseline.js`**
```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 30, duration: '5m',
  thresholds: { http_req_duration: ['p(95)<300'], http_req_failed: ['rate<0.01'] } };

export default function () {
  const res = http.get('http://localhost:3000/products');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

**`stress.js`**
```js
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 300 },
    { duration: '5m', target: 600 },
    { duration: '2m', target: 0 },
  ],
  thresholds: { http_req_failed: ['rate<0.05'] },
};
// same default() as above
```

**`spike.js`**
```js
export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '10s', target: 500 },
    { duration: '1m', target: 500 },
    { duration: '30s', target: 20 },
  ],
};
// same default() as above
```

**`soak.js`**
```js
export const options = { vus: 50, duration: '2h',
  thresholds: { http_req_duration: ['p(95)<300'] } };
// same default() as above
```

**Full workflow test — `full-workflow.js`** (combines auth + CRUD, use with any of the load shapes above):
```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function () {
  const login = http.post('http://localhost:3000/auth/login',
    JSON.stringify({ email: 'alice@test.com', password: '1234' }),
    { headers: { 'Content-Type': 'application/json' } });
  check(login, { 'login ok': (r) => r.status === 200 });
  const token = JSON.parse(login.body).token;
  const auth = { headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` } };

  http.get('http://localhost:3000/products');
  sleep(1);

  const order = http.post('http://localhost:3000/orders', JSON.stringify({ productId: 1, qty: 1 }), auth);
  check(order, { 'order created': (r) => r.status === 201 });

  http.get('http://localhost:3000/orders', auth);
  sleep(1);
}
```

**Command to run any of these:**
```bash
k6 run smoke.js
k6 run baseline.js
k6 run stress.js
k6 run spike.js
k6 run soak.js
```

**Expected result:** Smoke passes trivially; baseline should meet thresholds; stress should show *where* thresholds start failing; spike shows recovery behavior; soak reveals slow degradation if any.

**Common mistake:** Running stress/spike/soak tests against a shared staging environment other teams depend on — always confirm the target environment can safely absorb heavy load, or use an isolated environment.

---

## 19. Prometheus + Grafana visualization

**Why:** k6's terminal summary is fine for one-off runs, but for tracking trends over time and sharing with a team, you want persistent, queryable dashboards.

**Flow:**
```
k6  →  Prometheus (remote-write)  →  Grafana (dashboards)
```

**1. Run Prometheus + Grafana (quick local setup with Docker Compose):**

```yaml
# docker-compose.yml
version: '3'
services:
  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]
    volumes: ["./prometheus.yml:/etc/prometheus/prometheus.yml"]
    command: ["--config.file=/etc/prometheus/prometheus.yml", "--web.enable-remote-write-receiver"]
  grafana:
    image: grafana/grafana
    ports: ["3001:3000"]
```

```yaml
# prometheus.yml
global:
  scrape_interval: 5s
scrape_configs:
  - job_name: 'prometheus'
    static_configs: [{ targets: ['localhost:9090'] }]
```

```bash
docker compose up -d
```

**2. Send k6 metrics to Prometheus using the built-in output:**
```bash
k6 run --out experimental-prometheus-rw baseline.js
```
(Requires `K6_PROMETHEUS_RW_SERVER_URL=http://localhost:9090/api/v1/write` as an env var, or pass it as a flag.)

```bash
K6_PROMETHEUS_RW_SERVER_URL=http://localhost:9090/api/v1/write \
  k6 run --out experimental-prometheus-rw baseline.js
```

**3. In Grafana:** add Prometheus (`http://localhost:9090`) as a data source, then import the official **k6 Prometheus dashboard** (search dashboard ID `19665` in Grafana's dashboard library), or build your own panels:

| Panel | Query idea |
|---|---|
| **VUs over time** | `k6_vus` |
| **Requests/sec** | `rate(k6_http_reqs_total[1m])` |
| **p95 latency** | `k6_http_req_duration{quantile="0.95"}` |
| **p99 latency** | `k6_http_req_duration{quantile="0.99"}` |
| **Error rate** | `rate(k6_http_req_failed_total[1m])` |
| **Response time trend** | `k6_http_req_duration` (avg/min/max over time) |

**Expected result:** A live dashboard updating while `k6 run` executes, and a historical record afterward for comparing test runs over weeks/months.

**Common mistake:** Only looking at the dashboard *after* the test finishes. Watch it **live** — you can spot a problem (e.g., error rate spiking) and stop the test early instead of waiting for a 2-hour soak test to finish.

---

## 20. Professional performance reports

A good report is short, scannable, and answers: **"Did we meet our target, and if not, why?"**

**Recommended structure:**

```markdown
# Load Test Report — Orders API — 2026-08-22

## Objective
Validate the Orders API meets SLA of p95 < 300ms and error rate < 1%
under 300 concurrent users.

## Test Configuration
- Tool: k6 v0.5x
- Scenario: ramping-vus, 0→300 over 5 min, sustained 10 min
- Target: staging environment (2 vCPU, 4GB RAM, RDS db.t3.medium)

## Results Summary
| Metric | Target | Actual | Result |
|---|---|---|---|
| p95 latency | < 300ms | 245ms | ✅ Pass |
| p99 latency | < 600ms | 890ms | ❌ Fail |
| Error rate | < 1% | 0.3% | ✅ Pass |
| Throughput | ≥ 150 req/s | 162 req/s | ✅ Pass |

## Findings
- p99 breaches during the ramp from 200→300 VUs, correlating with DB CPU
  hitting 95% (see Grafana snapshot).
- No errors observed — the system slows down but does not fail outright.

## Recommendation
Add a read replica or index on `orders.user_id` before scaling past 250
concurrent users in production.

## Attachments
- Grafana dashboard screenshot
- Raw k6 summary (results.json)
```

**How to generate the raw data to back this up:**
```bash
k6 run --summary-export=summary.json baseline.js
```

**Command:** produce a `summary.json` and embed the key figures + a Grafana screenshot into the markdown/PDF report above.

**Common mistake:** Sending raw k6 terminal output to stakeholders. Non-engineers won't parse it — always translate into a pass/fail table with plain-language findings and a recommendation.

---

## 21. Running k6 in CI/CD

**Why:** Catch performance regressions automatically on every deploy, instead of relying on someone remembering to run a test manually.

**GitHub Actions example — `.github/workflows/load-test.yml`:**
```yaml
name: k6 load test

on:
  push:
    branches: [main]

jobs:
  load-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Start API
        run: |
          npm install
          node server.js &
          sleep 3

      - name: Run k6 smoke test
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/smoke.js

      - name: Run k6 baseline test with thresholds
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/baseline.js
```

Because thresholds make `k6 run` exit non-zero on failure, **the CI job fails automatically** if performance regresses — no extra logic needed.

**Typical pipeline strategy:**
- **On every PR:** run only the fast **smoke test** (seconds, catches broken endpoints).
- **On merge to main / nightly:** run **baseline** and maybe **stress**, since they take longer.
- **Before major releases:** run full **soak** test separately (hours), not blocking normal deploys.

**Command locally (mirrors CI):**
```bash
k6 run tests/smoke.js && k6 run tests/baseline.js
```

**Expected result:** Green pipeline when thresholds pass; red pipeline with the specific failing threshold logged when they don't.

**Common mistake:** Running long stress/soak tests on every single PR — it slows down the whole team's pipeline. Reserve heavy tests for scheduled/nightly runs.

---

## 22. Final real-world project

Put everything together. Using the Section 0 Express API:

1. **Write smoke, baseline, stress, spike, and soak tests** for `/products`, `/users`, `/auth/login`, and the full order workflow (Section 18 has starting points).
2. **Add thresholds** for p95/p99 latency and error rate to each test based on a target SLA you define (e.g., p95 < 300ms, errors < 1%).
3. **Add a `constant-arrival-rate` scenario** targeting a specific "orders/sec" business requirement (e.g., "handle 50 orders/sec at peak").
4. **Add custom metrics** for checkout duration and orders placed.
5. **Wire up Prometheus + Grafana** and take a dashboard screenshot during a stress test.
6. **Write a one-page performance report** (Section 20 template) summarizing findings and a recommendation.
7. **Add a CI/CD workflow** that runs the smoke + baseline tests on every push and fails the build if thresholds are breached.

**Definition of done:** You can explain, for this API: current capacity (VUs/req-s it handles within SLA), where it breaks (stress test ceiling), how it behaves under a traffic spike, whether it degrades over long duration (soak), and you have a dashboard + report to prove it.

---

## 23. Cheat sheet

**Common commands**
```bash
k6 run script.js                                  # run a test
k6 run --vus 10 --duration 30s script.js           # override VUs/duration via CLI
k6 run --iterations 100 script.js                  # run a fixed number of iterations
k6 run --summary-export=summary.json script.js     # export summary as JSON
k6 run --out json=results.json script.js           # export every data point
k6 run --http-debug="full" script.js                # print full request/response (debugging)
K6_PROMETHEUS_RW_SERVER_URL=<url> k6 run --out experimental-prometheus-rw script.js  # stream to Prometheus
```

**Basic options**
```js
export const options = {
  vus: 10,
  duration: '30s',
  stages: [{ duration: '1m', target: 50 }],
  thresholds: { http_req_duration: ['p(95)<300'] },
};
```

**GET / POST examples**
```js
http.get('http://api/products?minPrice=10');
http.post('http://api/users', JSON.stringify({ name: 'A' }), { headers: { 'Content-Type': 'application/json' } });
```

**Checks**
```js
check(res, { 'status 200': (r) => r.status === 200 });
```

**Thresholds**
```js
thresholds: {
  http_req_duration: ['p(95)<300', 'p(99)<600'],
  http_req_failed: ['rate<0.01'],
}
```

**Stages**
```js
stages: [
  { duration: '2m', target: 100 },
  { duration: '5m', target: 100 },
  { duration: '2m', target: 0 },
]
```

**Scenarios / important executors**
```js
scenarios: {
  steady: { executor: 'constant-vus', vus: 50, duration: '5m' },
  ramp:   { executor: 'ramping-vus', startVUs: 0, stages: [{ duration: '1m', target: 50 }] },
  fixedN: { executor: 'shared-iterations', vus: 10, iterations: 200 },
  perVU:  { executor: 'per-vu-iterations', vus: 10, iterations: 20 },
  rate:   { executor: 'constant-arrival-rate', rate: 100, timeUnit: '1s', duration: '5m', preAllocatedVUs: 100, maxVUs: 300 },
  rampRate: { executor: 'ramping-arrival-rate', startRate: 5, timeUnit: '1s', preAllocatedVUs: 50, maxVUs: 200, stages: [{ duration: '2m', target: 50 }] },
}
```

**Important metrics**
```
http_reqs             total requests
http_req_duration     response time (avg, min, med, max, p90, p95, p99)
http_req_failed       error rate
vus / vus_max         concurrent virtual users
iterations            completed iterations
dropped_iterations    iterations k6 couldn't start (arrival-rate maxVUs too low)
```

**One-line mental model, always:**
```
options → scenario/executor → VU → iteration → HTTP request → response → metrics → checks → thresholds → PASS/FAIL
```
