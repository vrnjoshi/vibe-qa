# ⚡ Performance Testing Checklist for AI-Generated Code

> **Why this checklist exists:** AI tools generate code that works correctly at small scale and in the developer's local environment. They do not optimize for real-world conditions: slow networks, large datasets, many concurrent users, and memory-constrained devices. Performance issues are the most common reason vibe-coded apps fail after launch — not because they break, but because they become too slow to use. This checklist identifies and addresses those failure modes before real users encounter them.
>
> Items marked ⚠️ are patterns AI tools produce most frequently. Items marked 🔴 are failures that will directly cause user abandonment or system outages under real-world load.

---

## 1. Page Load Performance

First impressions are determined by load speed. Studies consistently show users abandon pages that take more than 3 seconds to load.

### Core Web Vitals
Run your app through [PageSpeed Insights](https://pagespeed.web.dev) and [WebPageTest](https://www.webpagetest.org) and verify:

- [ ] 🔴 **Largest Contentful Paint (LCP)** is under 2.5 seconds — the time until the main content is visible
- [ ] 🔴 **Cumulative Layout Shift (CLS)** is under 0.1 — content does not jump around as the page loads
- [ ] **Interaction to Next Paint (INP)** is under 200ms — the app responds quickly to user interactions
- [ ] **First Contentful Paint (FCP)** is under 1.8 seconds — something appears on screen quickly

### Common AI-Generated Performance Problems
- [ ] ⚠️ JavaScript bundle size is audited — run `npm run build` and check the output. AI-generated apps frequently import entire libraries when only one function is needed (e.g., importing all of `lodash` for a single utility)
- [ ] ⚠️ Images are not oversized — a hero image should not be a 4MB uncompressed PNG. Compress images and serve modern formats (WebP, AVIF)
- [ ] Images have explicit `width` and `height` attributes to prevent layout shift during load
- [ ] ⚠️ Fonts are not blocking render — use `font-display: swap` and preload critical fonts. AI often generates synchronous font loading that delays the entire page
- [ ] CSS is not render-blocking — critical CSS is inlined and non-critical CSS is loaded asynchronously
- [ ] ⚠️ The app does not make unnecessary API calls on initial page load — audit the Network tab in browser DevTools and verify every request on load is actually needed
- [ ] Third-party scripts (analytics, chat widgets, ad scripts) are loaded asynchronously and do not block the main thread
- [ ] ⚠️ No unused JavaScript or CSS is shipped — AI-generated scaffolding often includes entire component libraries when only a few components are used

---

## 2. Database and Query Performance

This is the most critical performance category for backend-heavy vibe-coded apps. AI generates queries that work at small scale and become dangerously slow as data grows.

- [ ] 🔴 ⚠️ **N+1 query problem:** AI frequently generates code that makes one database query to fetch a list, then makes one additional query per item in the list to fetch related data. Test by checking the number of database queries when loading a list of 10 items vs 100 items — the query count should not scale with the number of items
- [ ] ⚠️ Database indexes exist on columns used in `WHERE`, `ORDER BY`, and `JOIN` clauses — AI rarely generates indexes. An unindexed query that runs in 10ms with 100 rows will take 10+ seconds with 1 million rows
- [ ] Queries that return large result sets are paginated — no query fetches all records from a table without a `LIMIT`
- [ ] ⚠️ Search queries use indexed full-text search, not `LIKE '%searchterm%'` — a leading wildcard prevents index usage and causes full table scans
- [ ] Slow query logging is enabled in development — identify queries that take more than 100ms
- [ ] ⚠️ ORM-generated queries are reviewed — AI using ORMs (Prisma, SQLAlchemy, ActiveRecord) often generates inefficient queries. Use query logging to see what SQL is actually being generated
- [ ] Queries inside loops are eliminated — any database call inside a `for` loop or `.map()` is a red flag
- [ ] ⚠️ Aggregation queries (COUNT, SUM, GROUP BY) on large tables have appropriate indexes and are cached where possible
- [ ] Database connection pooling is configured — AI-generated apps often open a new database connection per request, which fails under concurrent load

---

## 3. API Response Times

- [ ] All API endpoints respond in under 500ms for typical requests under normal conditions — measure using browser DevTools Network tab or Postman
- [ ] ⚠️ Endpoints that are slow in development will be slower in production — identify any endpoint over 200ms in dev and investigate before launch
- [ ] Heavy computation or data processing is done asynchronously (background jobs/queues) rather than making the user wait in a synchronous request
- [ ] ⚠️ File upload processing (image resizing, document parsing) happens asynchronously — AI often generates synchronous file processing that blocks the response for seconds
- [ ] API responses are appropriately cached for data that does not change frequently
- [ ] ⚠️ Cache invalidation is implemented — cached data is cleared when the underlying data changes. AI often adds caching without adding invalidation, causing stale data to be served indefinitely
- [ ] External API calls (to third-party services) have timeout limits set — AI rarely sets timeouts, so a slow external service can hang requests indefinitely

---

## 4. Memory and Resource Leaks

Memory leaks cause apps to slow down and crash over time. They only appear with sustained usage, not in brief testing.

- [ ] ⚠️ Open the app and use it normally for 15–30 minutes. Monitor memory usage in browser DevTools (Performance tab → Memory). Memory should remain stable, not grow continuously
- [ ] ⚠️ Event listeners are removed when components unmount — AI-generated React/Vue components frequently add event listeners in `useEffect` or `mounted` without removing them in the cleanup function
- [ ] Intervals and timers (`setInterval`, `setTimeout`) are cleared when they are no longer needed
- [ ] ⚠️ WebSocket connections are closed when the user navigates away from the page that opened them
- [ ] Large objects or data arrays are not held in memory longer than necessary — AI often stores entire API response datasets in state when only a subset is needed
- [ ] ⚠️ On the server side, global variables are not used to store per-request data — in Node.js, data stored in global scope persists across requests and grows indefinitely
- [ ] File streams are properly closed after reading or writing — unclosed streams accumulate open file handles

---

## 5. Concurrency and Load

Test how the app behaves when multiple users use it simultaneously. AI generates code that works for one user and often fails for many.

- [ ] ⚠️ Run a basic load test using [k6](https://k6.io), [Artillery](https://www.artillery.io), or [Locust](https://locust.io) with 10–50 simulated concurrent users performing the core user flow — measure response times and error rates
- [ ] The app does not have race conditions — test by performing the same action simultaneously in two browser tabs (e.g., claiming the same limited resource, updating the same record)
- [ ] ⚠️ Database transactions are used where multiple queries must succeed or fail together — AI often generates separate queries where a transaction is required, causing partial updates if one query fails
- [ ] File uploads do not fail under concurrent load — test multiple simultaneous uploads
- [ ] ⚠️ The server does not run out of memory or crash under moderate load (50 concurrent users) — if it does, identify the bottleneck before launch
- [ ] Background jobs and queues handle backlog gracefully — if jobs accumulate faster than they are processed, the queue does not grow unboundedly
- [ ] Rate limiting is in place on expensive endpoints to prevent a single user from degrading performance for others

---

## 6. Frontend Rendering Performance

- [ ] 🔴 ⚠️ Scrolling through a long list is smooth — no janky or stuttering scroll. Test by loading a list of 50+ items and scrolling quickly
- [ ] ⚠️ Large lists are virtualized (only rendering visible items) — AI generates lists that render all items in the DOM simultaneously, which causes severe lag at 200+ items. Libraries like `react-window`, `react-virtual`, or `@tanstack/virtual` solve this
- [ ] ⚠️ Unnecessary re-renders are minimized — in React, use the Profiler tool to check if components re-render when their data has not changed. AI-generated components frequently re-render entire trees on minor state changes
- [ ] Animations use CSS transforms and opacity rather than properties that trigger layout (width, height, top, left) — layout-triggering animations cause dropped frames
- [ ] ⚠️ Images are lazy-loaded below the fold — AI often loads all images on page load including those not yet visible
- [ ] Heavy computations (sorting, filtering, transforming large datasets) are not done on every render — they should be memoized or moved to the server
- [ ] ⚠️ The main thread is not blocked by synchronous JavaScript — operations that take more than 50ms should be chunked or moved to a Web Worker

---

## 7. Network Efficiency

- [ ] ⚠️ API responses are not returning unnecessary data — if the UI only displays 5 fields, the API should not return 50. AI-generated APIs frequently return entire database rows when a subset of columns is needed
- [ ] HTTP compression (gzip or brotli) is enabled on the server — dramatically reduces response sizes for text-based content
- [ ] ⚠️ Static assets (JavaScript, CSS, images) are served with long cache headers (`Cache-Control: max-age=31536000`) and use content hashing for cache busting
- [ ] API calls are not duplicated — verify in the Network tab that the same request is not made twice on page load (AI-generated `useEffect` hooks often fire twice in React strict mode)
- [ ] ⚠️ Real-time features (live updates, notifications) use WebSockets or Server-Sent Events rather than polling every second — AI often generates polling loops that create excessive server load
- [ ] GraphQL queries only request the fields that are needed — avoid `SELECT *` equivalent queries in GraphQL

---

## 8. Performance on Low-End Conditions

Test in conditions that reflect real users, not ideal development environments.

- [ ] ⚠️ Use Chrome DevTools to throttle CPU to 4x slowdown and test the main user flow — simulates mid-range Android devices
- [ ] Use Chrome DevTools to throttle network to "Slow 4G" (or "Fast 3G" for a more challenging test) and measure load times
- [ ] The app is usable on a 4-year-old Android device (or test with CPU throttled in DevTools)
- [ ] ⚠️ The app loads and is usable with browser extensions installed — ad blockers and privacy extensions can block requests that AI-generated apps depend on without fallback handling
- [ ] Test performance with the browser cache cleared (hard reload with Ctrl+Shift+R) — represents first-time visitors

---

## 9. Performance Monitoring (Post-Launch)

Performance degrades over time as data grows and features accumulate. Set up monitoring before launch.

- [ ] ⚠️ Real User Monitoring (RUM) is set up to track actual load times for real users — not just synthetic tests. Options: [Vercel Analytics](https://vercel.com/analytics), [Sentry Performance](https://sentry.io), [Datadog RUM](https://www.datadoghq.com)
- [ ] Core Web Vitals are tracked in [Google Search Console](https://search.google.com/search-console) — performance issues affect SEO ranking
- [ ] Database slow query alerts are configured — get notified when any query exceeds a threshold (e.g., 500ms)
- [ ] ⚠️ Server resource usage (CPU, memory, disk) is monitored with alerts for threshold breaches — AI-generated apps sometimes have resource leaks that only appear after days of running
- [ ] Error rates are monitored — a spike in errors often indicates a performance-related cascade failure

---

## 10. Quick Performance Smoke Test (Run Before Every Launch)

- [ ] Run the app through [PageSpeed Insights](https://pagespeed.web.dev) — aim for a score of 80+ on mobile
- [ ] Open browser DevTools Network tab and reload the page — verify total page size is under 3MB and load time under 3 seconds on a fast connection
- [ ] Check for any API request taking over 1 second in the Network tab on the main pages
- [ ] Scroll through the main list or feed quickly — confirm no stuttering or lag
- [ ] Open the app on a real mobile device on cellular data (not WiFi) and time how long it takes to load
- [ ] Check the browser console for any performance warnings (excessive re-renders, large DOM size, etc.)

---

## Recommended Tools

| Tool | Purpose | Cost |
|---|---|---|
| [PageSpeed Insights](https://pagespeed.web.dev) | Core Web Vitals and load performance | Free |
| [WebPageTest](https://www.webpagetest.org) | Detailed waterfall and film strip analysis | Free |
| [k6](https://k6.io) | Load and stress testing | Free |
| [Artillery](https://www.artillery.io) | API and load testing | Free tier |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse) | Built into Chrome DevTools | Free |
| [Clinic.js](https://clinicjs.org) | Node.js performance profiling | Free |
| [py-spy](https://github.com/benfred/py-spy) | Python performance profiling | Free |
| [Sentry Performance](https://sentry.io) | Real user monitoring + tracing | Free tier |

---

## Related Checklists

- 📱 [Mobile Testing](./05-mobile-testing.md) — mobile-specific performance considerations
- 🔌 [API Testing](./03-api-testing.md) — API response time and rate limiting
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found a performance failure pattern in AI-generated code not listed here? [Open a PR](../CONTRIBUTING.md) — real-world additions make this more useful for everyone.*
