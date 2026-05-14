# 🐛 Known AI Code Failure Patterns

> **What this document is:** A catalog of specific, documented, repeating bugs that AI coding tools produce. These are not random mistakes — they are systematic patterns that appear across different AI tools, different projects, and different developers because they stem from how AI models generate code. Each pattern includes what it looks like, why AI generates it, how to find it, and how to fix it.
>
> This document grows through real-world experience. If you have found a repeating failure pattern in AI-generated code, [open a PR](../CONTRIBUTING.md) and add it.

---

## How to Use This Document

**Debugging a strange bug?** Search this page for keywords related to what you are seeing.

**Reviewing AI-generated code?** Read through the patterns before your review — knowing what to look for is half the battle.

**Starting a new vibe-coded project?** Bookmark this page. The patterns you will encounter are likely already documented here.

---

## Pattern Index

1. [Client-Side Authentication](#1-client-side-authentication)
2. [The N+1 Database Query](#2-the-n1-database-query)
3. [Orphaned Records on Delete](#3-orphaned-records-on-delete)
4. [Hardcoded Development Values](#4-hardcoded-development-values)
5. [The Happy Path Only Problem](#5-the-happy-path-only-problem)
6. [Missing Rate Limiting](#6-missing-rate-limiting)
7. [Synchronous File Processing](#7-synchronous-file-processing)
8. [Hallucinated API Endpoints](#8-hallucinated-api-endpoints)
9. [Event Listener Memory Leaks](#9-event-listener-memory-leaks)
10. [Wildcard CORS on Authenticated Endpoints](#10-wildcard-cors-on-authenticated-endpoints)
11. [JWT Algorithm Confusion](#11-jwt-algorithm-confusion)
12. [Missing Input Validation on the Server](#12-missing-input-validation-on-the-server)
13. [The Double Submit Problem](#13-the-double-submit-problem)
14. [Console.log in Production](#14-consolelog-in-production)
15. [Broken Tab Order](#15-broken-tab-order)
16. [Race Conditions in Async Operations](#16-race-conditions-in-async-operations)
17. [Entire Library Imported for One Function](#17-entire-library-imported-for-one-function)
18. [Missing Error Boundaries](#18-missing-error-boundaries)
19. [Unparameterized Database Queries](#19-unparameterized-database-queries)
20. [Infinite Re-render Loops](#20-infinite-re-render-loops)

---

## 1. Client-Side Authentication

**Severity:** 🔴 Critical

### What it looks like
The app checks whether a user is logged in or is an admin by reading a value from `localStorage`, a cookie, or a React/Vue state variable — and then shows or hides UI elements based on that value. The protected API endpoints themselves do not validate the token or role on every request.

```javascript
// ⚠️ AI-generated pattern — dangerous
if (localStorage.getItem('isAdmin') === 'true') {
  showAdminPanel();
}
```

### Why AI generates it
AI tools optimize for making the demo work. Hiding the admin button based on a localStorage value makes the demo look correct. The AI does not consider that a user can open DevTools, set `localStorage.setItem('isAdmin', 'true')`, and instantly gain admin access.

### How to find it
- Search the codebase for `localStorage.getItem`, `sessionStorage.getItem`, and cookie reads that are used in conditional logic for showing protected UI
- Test by manually setting the value in browser DevTools and reloading the page
- Test API endpoints directly (without the UI) using Postman or curl with a regular user token on admin-only endpoints

### How to fix it
Move all authorization checks to the server. Every API endpoint that returns, modifies, or deletes privileged data must validate the user's token and role server-side on every request — not just at login.

```javascript
// ✅ Correct pattern — server validates on every request
app.get('/api/admin/users', requireAuth, requireAdmin, async (req, res) => {
  // Only reaches here if server confirmed the user is an admin
});
```

---

## 2. The N+1 Database Query

**Severity:** 🔴 Critical (at scale)

### What it looks like
The app makes 1 database query to fetch a list of items, then makes 1 additional query for each item in the list to fetch related data. For a list of 100 items, this is 101 database queries. The page is fast in development with 5 test records and becomes unusably slow in production with 1,000 records.

```javascript
// ⚠️ AI-generated pattern — N+1 problem
const orders = await db.query('SELECT * FROM orders');
for (const order of orders) {
  // One query per order — disaster at scale
  order.user = await db.query('SELECT * FROM users WHERE id = ?', [order.userId]);
}
```

### Why AI generates it
AI generates the most readable, step-by-step code that solves the problem as described. Fetching each related record in a loop is conceptually simple and works correctly at small scale. The AI does not anticipate scale unless specifically prompted to.

### How to find it
- Enable database query logging and count the number of queries when loading a list page
- The number of queries should not increase linearly with the number of items in the list
- Look for database calls inside `for` loops, `.map()`, `.forEach()`, or similar iteration

### How to fix it
Use a JOIN query or fetch all related records in a single second query using `WHERE id IN (...)`.

```javascript
// ✅ Correct pattern — 2 queries regardless of list size
const orders = await db.query('SELECT * FROM orders');
const userIds = orders.map(o => o.userId);
const users = await db.query('SELECT * FROM users WHERE id IN (?)', [userIds]);
const userMap = Object.fromEntries(users.map(u => [u.id, u]));
orders.forEach(order => { order.user = userMap[order.userId]; });
```

---

## 3. Orphaned Records on Delete

**Severity:** 🔴 Critical

### What it looks like
Deleting a record removes it from its primary table but leaves all related records in other tables intact. A deleted user still has their orders, comments, uploaded files, and activity logs in the database — orphaned and unlinked. A deleted post still has its comments, reactions, and tags.

### Why AI generates it
When prompted to "add a delete function for users," the AI generates a query that deletes from the users table. It does not know about the schema of related tables unless that context is explicitly provided in the prompt.

### How to find it
- Delete a record and then query related tables for records referencing the deleted ID
- Check for orphaned records: `SELECT * FROM orders WHERE userId NOT IN (SELECT id FROM users)`
- Check whether uploaded files, S3 objects, or external records are cleaned up alongside database records

### How to fix it
Use database-level cascade deletes, or implement an explicit cleanup sequence before the primary delete:

```sql
-- Option 1: Database cascade (set up in schema)
FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE

-- Option 2: Explicit cleanup in application code
await db.query('DELETE FROM user_files WHERE userId = ?', [userId]);
await db.query('DELETE FROM user_sessions WHERE userId = ?', [userId]);
await db.query('DELETE FROM orders WHERE userId = ?', [userId]);
await db.query('DELETE FROM users WHERE id = ?', [userId]);
```

---

## 4. Hardcoded Development Values

**Severity:** 🟡 High

### What it looks like
The AI uses specific values during code generation — a user ID, an email address, a URL, a date, an API key — and those values end up hardcoded in the source code instead of being configurable or dynamic.

```javascript
// ⚠️ AI-generated patterns — hardcoded values
const ADMIN_EMAIL = 'admin@example.com';
const API_URL = 'http://localhost:3000/api';
const TEST_USER_ID = '12345';
const DEFAULT_ORG_ID = 1;
```

### Why AI generates it
AI uses placeholder values to make the generated code runnable immediately. These values serve as examples but are never replaced because the developer does not notice them.

### How to find it
Search the codebase for:
- `localhost` appearing in non-comment, non-test code
- `example.com` in production code
- Specific numeric IDs (1, 123, 9999) used as defaults
- Email addresses in non-test source files
- Commented-out code referencing specific internal systems

### How to fix it
Move all environment-specific values to environment variables and access them through a configuration object:

```javascript
// ✅ Correct pattern
const API_URL = process.env.API_URL;
const ADMIN_EMAIL = process.env.ADMIN_EMAIL;
if (!API_URL) throw new Error('API_URL environment variable is not set');
```

---

## 5. The Happy Path Only Problem

**Severity:** 🟡 High

### What it looks like
The feature works perfectly when the user does exactly what the developer expected. It breaks, crashes, or behaves incorrectly for any deviation: empty states, unexpected input, network failures, concurrent edits, missing permissions.

### Why AI generates it
AI generates code based on the prompt, which typically describes the success scenario. The prompt "build a checkout flow" does not mention what happens when the payment fails, the item goes out of stock mid-checkout, or the user's session expires during the process.

### How to find it
For every feature, explicitly test:
- What happens with no data / empty state
- What happens when the network fails mid-operation
- What happens if the user refreshes mid-flow
- What happens with the minimum valid input
- What happens with the maximum valid input
- What happens if two users perform the action simultaneously

### How to fix it
Add explicit handling for every branch. When reviewing AI-generated code, look for `if` statements without corresponding `else` clauses, `try` blocks without `catch` clauses, and async operations without error handling.

---

## 6. Missing Rate Limiting

**Severity:** 🟡 High

### What it looks like
Authentication endpoints (login, register, password reset, OTP verification) accept unlimited requests per second from any IP or user. An attacker can attempt millions of password combinations, flood the registration endpoint to create fake accounts, or abuse the password reset endpoint to spam users.

### Why AI generates it
Rate limiting requires middleware configuration that is separate from the feature being built. When asked to "build a login endpoint," the AI builds the login logic but does not add rate limiting unless explicitly asked.

### How to find it
Send 20+ rapid requests to the login endpoint and observe whether the server throttles, blocks, or allows all of them equally.

### How to fix it
Add rate limiting middleware to all authentication-adjacent endpoints:

```javascript
// Express + express-rate-limit example
import rateLimit from 'express-rate-limit';

const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 attempts per window
  message: 'Too many attempts, please try again later',
  standardHeaders: true,
});

app.post('/api/auth/login', authLimiter, loginHandler);
app.post('/api/auth/reset-password', authLimiter, resetPasswordHandler);
```

---

## 7. Synchronous File Processing

**Severity:** 🟡 High

### What it looks like
When a user uploads a file, the server processes it (resizes an image, parses a CSV, converts a document) synchronously before responding. The HTTP request hangs for seconds or minutes while processing occurs. Under concurrent uploads, the server becomes unresponsive.

### Why AI generates it
Processing a file immediately after upload is the simplest implementation. AI generates the path of least resistance unless background processing is explicitly requested.

### How to find it
Upload a large file and measure how long the HTTP response takes. Time it with 5 simultaneous uploads. Monitor server CPU during upload processing.

### How to fix it
Move file processing to a background job queue. Return a `202 Accepted` response immediately, and notify the user when processing is complete:

```javascript
// ✅ Correct pattern
app.post('/api/upload', async (req, res) => {
  const fileRef = await saveFileToStorage(req.file);
  await jobQueue.add('process-file', { fileRef }); // Non-blocking
  res.status(202).json({ message: 'File received, processing in background', fileRef });
});
```

---

## 8. Hallucinated API Endpoints

**Severity:** 🟡 High

### What it looks like
The AI generates code that calls an external API (Stripe, SendGrid, Twilio, OpenAI, etc.) using an endpoint URL, parameter name, or response field that does not exist in that API's actual documentation. The code looks correct and passes a visual review but fails at runtime with a 404 or unexpected response.

### Why AI generates it
AI models are trained on API documentation but that documentation changes over time. The model may have learned an older version of an API, inferred a parameter name from patterns, or simply confabulated a plausible-sounding endpoint.

### How to find it
For every external API call in the codebase:
1. Find the exact endpoint URL being called
2. Look it up in the current official API documentation
3. Verify every request parameter against the docs
4. Test the call in isolation using Postman before testing the full flow

### How to fix it
Never trust AI-generated external API calls without verification. Treat them as a starting point that requires documentation review, not finished code.

---

## 9. Event Listener Memory Leaks

**Severity:** 🟡 High

### What it looks like
The app becomes progressively slower over time during a single session. Memory usage grows steadily. After 10–20 minutes of use, interactions become laggy. A browser tab refresh fixes it temporarily.

### Why AI generates it
AI adds event listeners in component lifecycle hooks (`useEffect`, `mounted`, `componentDidMount`) but forgets to return a cleanup function that removes them when the component unmounts. Each time the component mounts, a new listener is added on top of the old one.

### How to find it
```javascript
// ⚠️ AI-generated pattern — leak
useEffect(() => {
  window.addEventListener('resize', handleResize);
  // No cleanup — listener accumulates on every render
}, []);
```

Open Chrome DevTools → Memory tab → Take heap snapshot → Navigate around the app for 5 minutes → Take another snapshot → Compare. Look for growing listener counts.

### How to fix it
```javascript
// ✅ Correct pattern — cleanup included
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => {
    window.removeEventListener('resize', handleResize); // Cleanup on unmount
  };
}, []);
```

---

## 10. Wildcard CORS on Authenticated Endpoints

**Severity:** 🔴 Critical

### What it looks like
The server responds with `Access-Control-Allow-Origin: *` on API endpoints that require authentication. This means any website on the internet can make authenticated requests to the API on behalf of a logged-in user.

### Why AI generates it
Setting CORS to `*` makes everything work immediately during development without needing to configure specific allowed origins. The security implication is non-obvious and AI tools prioritize making the code functional.

### How to find it
Open browser DevTools → Network tab → Find an authenticated API request → Check the response headers for `Access-Control-Allow-Origin: *`.

### How to fix it
```javascript
// ✅ Correct pattern — explicit allowed origins
app.use(cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
}));
```

---

## 11. JWT Algorithm Confusion

**Severity:** 🔴 Critical

### What it looks like
The JWT validation code does not explicitly specify which algorithm it accepts. An attacker sends a JWT with `"alg": "none"` in the header, bypassing signature verification entirely and gaining access to any account.

### Why AI generates it
AI generates JWT verification code that works for the expected case. The `alg: none` attack is an edge case that requires explicitly rejecting it, which is easily overlooked.

### How to find it
Generate a JWT with `"alg": "none"` and an arbitrary payload using [jwt.io](https://jwt.io). Send it to a protected endpoint. If you receive a successful response, the endpoint is vulnerable.

### How to fix it
```javascript
// ✅ Correct pattern — algorithm explicitly specified
jwt.verify(token, process.env.JWT_SECRET, { algorithms: ['HS256'] });
```

---

## 12. Missing Input Validation on the Server

**Severity:** 🔴 Critical

### What it looks like
The app validates user input on the frontend (required fields, email format, length limits) but the API endpoints do not validate the same inputs server-side. An attacker who bypasses the frontend (using Postman, curl, or browser DevTools) can send invalid, oversized, or malicious data directly to the API.

### Why AI generates it
Frontend validation is visible and immediate — it directly improves the demo experience. Server-side validation is invisible during normal development. AI prioritizes what makes the demo work.

### How to find it
Use Postman or browser DevTools to send API requests directly, bypassing the frontend. Send empty strings, missing required fields, strings that exceed the frontend's length limits, and invalid formats.

### How to fix it
Validate all inputs at the API layer using a schema validation library:

```javascript
// ✅ Using Zod (JavaScript) for server-side validation
const createUserSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(128),
  name: z.string().min(1).max(100),
});

app.post('/api/users', (req, res) => {
  const result = createUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.issues });
  }
  // Proceed with validated data
});
```

---

## 13. The Double Submit Problem

**Severity:** 🟡 High

### What it looks like
A user clicks a submit button twice (slow network, habit, double-click). Two identical records are created: two orders, two payments, two user accounts, two API calls. The app has no defense against duplicate submissions.

### Why AI generates it
AI generates a button that calls a function on click. It does not add a loading state, disable the button during processing, or implement idempotency — these are additional steps that go beyond the immediate prompt.

### How to find it
Click the submit button rapidly twice in succession during a slow operation. Check whether the action was performed once or twice.

### How to fix it
Disable the button and show a loading state during submission:

```jsx
// ✅ Correct pattern
const [isSubmitting, setIsSubmitting] = useState(false);

const handleSubmit = async () => {
  if (isSubmitting) return;
  setIsSubmitting(true);
  try {
    await submitForm();
  } finally {
    setIsSubmitting(false);
  }
};

<button onClick={handleSubmit} disabled={isSubmitting}>
  {isSubmitting ? 'Saving...' : 'Save'}
</button>
```

---

## 14. Console.log in Production

**Severity:** 🟡 Medium

### What it looks like
`console.log`, `console.error`, `print()`, or equivalent debug statements appear in production code, logging sensitive data (user tokens, API responses containing personal data, internal state) to the browser console or server logs where they can be read by developers, support staff, or malicious actors with physical device access.

### Why AI generates it
AI adds logging statements as it generates code to help with debugging during development. These statements serve a useful purpose during generation but are not removed afterward.

### How to find it
```bash
# Search for debug statements
grep -r "console\.log\|console\.error\|console\.warn\|debugger" --include="*.js" --include="*.ts" src/
```

Also open the browser console on the production app and look for any logged output during normal use.

### How to fix it
Remove all debug logging before deployment, or replace with a structured logging library that can be configured to suppress output in production:

```javascript
// ✅ Environment-aware logging
if (process.env.NODE_ENV !== 'production') {
  console.log('Debug info:', data);
}
```

---

## 15. Broken Tab Order

**Severity:** 🟡 Medium

### What it looks like
Pressing the Tab key to navigate through a page jumps between elements in an order that does not match the visual layout. Users who navigate by keyboard (including all screen reader users, power users, and users with motor disabilities) find the interface confusing or unusable.

### Why AI generates it
AI generates layouts using absolute positioning, CSS Grid, Flexbox with `order` property, or visually-reordered elements that look correct on screen but have a different order in the HTML. Tab order follows HTML order, not visual order.

### How to find it
Navigate the page using only the Tab key and observe whether focus moves in a logical left-to-right, top-to-bottom sequence.

### How to fix it
Ensure the HTML source order matches the visual order. Avoid using `tabindex` values greater than 0. If visual reordering with CSS is necessary, ensure the HTML order is still logical for keyboard navigation.

---

## 16. Race Conditions in Async Operations

**Severity:** 🟡 High

### What it looks like
Two async operations run simultaneously and the result depends on which one finishes first, causing inconsistent or incorrect behavior. Classic example: a user types in a search box, two API calls are made in quick succession, and the results from the older (slower) call overwrite the results from the newer (faster) call.

### Why AI generates it
AI generates straightforward async code for the single-operation case. Race condition handling requires cancellation tokens, debouncing, or request sequencing — additional complexity not implied by a simple prompt.

### How to find it
Rapidly trigger the same async operation multiple times (fast typing in a search box, clicking a button multiple times, navigating away and back quickly). Check whether the final displayed state is always correct.

### How to fix it
```javascript
// ✅ Correct pattern — cancel stale requests
useEffect(() => {
  const controller = new AbortController();

  const fetchResults = async () => {
    const response = await fetch(`/api/search?q=${query}`, {
      signal: controller.signal
    });
    const data = await response.json();
    setResults(data);
  };

  fetchResults().catch(err => {
    if (err.name !== 'AbortError') throw err; // Ignore cancelled requests
  });

  return () => controller.abort(); // Cancel on re-render
}, [query]);
```

---

## 17. Entire Library Imported for One Function

**Severity:** 🟡 Medium

### What it looks like
The JavaScript bundle shipped to users is significantly larger than it needs to be because the AI imported an entire library when only one or two functions from it are needed.

```javascript
// ⚠️ AI-generated pattern — imports everything
import _ from 'lodash'; // 70KB for one function
const result = _.groupBy(items, 'category');
```

### Why AI generates it
Importing the full library is the most common pattern in training data. Tree-shaking and named imports are optimizations that require intentional effort.

### How to find it
Run the production build and analyze the bundle. Tools: `webpack-bundle-analyzer`, `vite-plugin-visualizer`, `source-map-explorer`. Look for large libraries where only a small portion is used.

### How to fix it
```javascript
// ✅ Named import — only ships the needed function
import groupBy from 'lodash/groupBy';

// ✅ Or use the native equivalent
const result = Object.groupBy(items, item => item.category); // No library needed
```

---

## 18. Missing Error Boundaries

**Severity:** 🟡 High

### What it looks like
A single JavaScript error in one component causes the entire app to show a white blank screen. The user has no way to recover other than refreshing the page.

### Why AI generates it
AI generates individual components without wrapping them in error boundaries. Error boundaries require a structural decision about the app architecture that goes beyond building any individual component.

### How to find it
Introduce a deliberate error in one component (temporarily) and observe whether it crashes only that component or the entire app.

### How to fix it
Wrap sections of the app in React Error Boundaries (or equivalent in other frameworks):

```jsx
// ✅ Correct pattern
<ErrorBoundary fallback={<p>Something went wrong in this section.</p>}>
  <UserDashboard />
</ErrorBoundary>
```

---

## 19. Unparameterized Database Queries

**Severity:** 🔴 Critical

### What it looks like
User input is concatenated directly into a database query string instead of being passed as a separate parameter. This allows SQL injection attacks where an attacker can read, modify, or delete any data in the database.

```javascript
// ⚠️ AI-generated pattern — SQL injection vulnerability
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;
// If userEmail is: ' OR '1'='1
// Query becomes: SELECT * FROM users WHERE email = '' OR '1'='1'
// Returns all users
```

### Why AI generates it
String interpolation into queries is the most readable way to write them. AI generates readable code. Parameterized queries are a security pattern that must be explicitly enforced.

### How to find it
Search for database query calls where user input appears to be directly interpolated:
```bash
grep -r "query\|execute\|raw" --include="*.js" --include="*.ts" --include="*.py" .
```
Look for template literals or string concatenation inside query functions.

### How to fix it
```javascript
// ✅ Correct pattern — parameterized query
const query = 'SELECT * FROM users WHERE email = ?';
const result = await db.execute(query, [userEmail]);
```

---

## 20. Infinite Re-render Loops

**Severity:** 🟡 High

### What it looks like
The app freezes, the browser tab crashes, or CPU usage spikes to 100% immediately after loading a page or performing an action. The browser console shows "Too many re-renders" or similar.

### Why AI generates it
AI generates `useEffect` hooks (React) or watchers (Vue) with missing or incorrect dependency arrays, causing state updates inside the effect to trigger the effect to run again in an infinite loop.

```javascript
// ⚠️ AI-generated pattern — infinite loop
useEffect(() => {
  setData(processData(data)); // Updates 'data', which triggers this effect again
}, [data]); // 'data' in deps causes infinite loop
```

### How to find it
Check the browser console for re-render warnings. Monitor CPU usage on page load. Look for `useEffect` hooks where a state variable is both in the dependency array and updated inside the effect.

### How to fix it
```javascript
// ✅ Correct pattern — process during render, not in effect
const processedData = useMemo(() => processData(rawData), [rawData]);

// Or separate the source data from the derived data
const [rawData, setRawData] = useState(initialData);
const processedData = processData(rawData); // Derived, not state
```

---

## Contributing New Patterns

Found a repeating failure pattern in AI-generated code that is not listed here? Please [open a PR](../CONTRIBUTING.md).

A good pattern entry includes:
- A descriptive title
- Severity level (🔴 Critical / 🟡 High / 🟢 Medium)
- What it looks like (ideally with a code example)
- Why AI generates it (the root cause)
- How to find it (specific search terms or test steps)
- How to fix it (ideally with a corrected code example)

Every real-world addition makes this document more useful for the entire community.

---

*This document is maintained by QA engineers with real-world experience testing AI-generated applications. It grows through community contributions.*
