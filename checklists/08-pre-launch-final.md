# 🚀 Pre-Launch Final Checklist for AI-Generated Code

> **How to use this checklist:** Run through every item below before making your vibe-coded app publicly available. This is not a comprehensive test suite — it is the minimum bar. Every item here represents a category of failure that has caused a real app to be embarrassed, breached, or shut down after launch. If any item fails, fix it before going live.
>
> Time required: **30–60 minutes** for a typical app.
> Prerequisites: Your app is deployed to its production environment, not localhost.

---

## 🔴 STOP. Do these 5 things first.

If you only have 10 minutes, do these. They are the highest-severity failures found in vibe-coded apps.

- [ ] **No secrets in the codebase.** Run: `grep -r "api_key\|password\|secret\|token\|private_key" --include="*.js" --include="*.ts" --include="*.py" --include="*.env" .` — any matches in source files are a critical failure
- [ ] **`.env` file is NOT committed to git.** Run: `git log --all -- "**/.env" "*.env"` — any results mean credentials may be compromised even if the file was later deleted
- [ ] **A logged-out user cannot access protected pages.** Open an incognito window. Try navigating directly to `/dashboard`, `/admin`, `/settings`, or any authenticated route. You should be redirected to login, not shown the page
- [ ] **User A cannot see User B's data.** Log in as one user. Copy the URL of a record. Log in as a different user. Paste the URL. You should see an error or be redirected, not the other user's data
- [ ] **The app loads without a white screen.** Open the production URL in an incognito window with no cache. The app loads and shows real content within 5 seconds

---

## Security (8 checks)

- [ ] All pages are served over **HTTPS** — the browser shows a padlock, not a "Not Secure" warning
- [ ] **CORS is not set to wildcard** (`Access-Control-Allow-Origin: *`) on any authenticated API endpoint — check your server configuration or response headers in browser DevTools
- [ ] **Rate limiting** is active on login, registration, and password reset endpoints — verify by checking your server middleware or API gateway configuration
- [ ] **Debug mode is OFF** in production — no `DEBUG=true`, `NODE_ENV=development`, or equivalent in your production environment variables
- [ ] **Default credentials are changed** on any admin panels, databases, or third-party services configured during development
- [ ] **Cloud storage buckets** (S3, GCS, Azure Blob) storing private user data are not publicly accessible — test by copying a private file URL and opening it while logged out
- [ ] Run your production URL through **[Mozilla Observatory](https://observatory.mozilla.org)** — fix any F or D grade findings before launch
- [ ] **No `console.log` statements** expose sensitive data — search the codebase and browser console during normal use for accidentally logged tokens, user data, or internal details

---

## Functionality (5 checks)

- [ ] **Core user flow works end-to-end** in production — sign up → complete the main action → sign out, all without errors
- [ ] **Every navigation link resolves** — click through every item in the main navigation. No 404s, no blank pages
- [ ] **Forms submit and save correctly** — data entered in the main form appears correctly after submission and after a page refresh
- [ ] **Error states show user-friendly messages** — deliberately submit an empty form and disconnect the network briefly. Users should see helpful messages, not stack traces or blank screens
- [ ] **Email delivery works** — trigger a transactional email (welcome email, password reset). Confirm it arrives, is not in spam, and all links in the email work

---

## Data and Infrastructure (4 checks)

- [ ] **Database backups are configured** — before real users create real data, confirm automated backups are enabled and you know how to restore from one
- [ ] **No test or seed data is visible** in production — no "test@test.com" users, no "Lorem ipsum" content, no dummy records from development
- [ ] **Environment variables are set correctly** in production — not pointing to development databases, staging APIs, or test payment keys
- [ ] **Payment processing uses live keys** (not test/sandbox keys) if the app charges money — and conversely, confirm test keys were NOT accidentally left in production

---

## Performance (3 checks)

- [ ] **Mobile load time is acceptable** — open the production URL on a real mobile device using cellular data (not WiFi). It should load and be usable within 5 seconds
- [ ] Run the production URL through **[PageSpeed Insights](https://pagespeed.web.dev)** — aim for a mobile score of **70+** before launch. Below 50 will cause significant user abandonment
- [ ] **No API endpoint takes over 3 seconds** to respond under normal conditions — check the Network tab in browser DevTools while using the app normally

---

## Legal and Compliance (3 checks)

- [ ] **Privacy Policy exists and is linked** from the footer or signup page — required in virtually every jurisdiction if you collect any user data. Required by Apple and Google for app store listings
- [ ] **Terms of Service exists** if users are creating accounts or making purchases
- [ ] **Cookie consent** is shown if the app sets non-essential cookies (analytics, advertising, personalization) — required under GDPR for EU users and CCPA for California users

---

## Monitoring (2 checks)

- [ ] **Error monitoring is set up** — at minimum, [Sentry](https://sentry.io) free tier. You need to know when users hit errors in production, not find out through a complaint days later
- [ ] **Uptime monitoring is set up** — [UptimeRobot](https://uptimerobot.com) free tier monitors your URL every 5 minutes and emails you if it goes down. Takes 2 minutes to set up

---

## ✅ Launch Readiness Summary

Before you flip the switch, confirm:

| Category | Status |
|---|---|
| 🔴 Critical 5 | All passing |
| 🔐 Security (8) | All passing |
| ✅ Functionality (5) | All passing |
| 🗄️ Data & Infrastructure (4) | All passing |
| ⚡ Performance (3) | All passing |
| ⚖️ Legal (3) | All passing |
| 📡 Monitoring (2) | All passing |

**Total: 30 checks. If any are failing — fix before launch.**

---

## After Launch: First 24 Hours

- [ ] Monitor your error tracker (Sentry) for any new errors appearing from real users
- [ ] Check your uptime monitor is active and reporting correctly
- [ ] Manually test the core user flow once more on the live production URL
- [ ] Watch your server resource usage (CPU, memory) for the first hour of real traffic

---

## Going Deeper

This checklist covers the minimum. For comprehensive coverage before a major launch, work through the full checklists:

| Full Checklist | What it covers |
|---|---|
| [Functional Testing](./01-functional-testing.md) | All user flows, edge cases, and AI-specific logic failures |
| [Security Testing](./02-security-testing.md) | 50+ security checks across auth, injection, secrets, and config |
| [API Testing](./03-api-testing.md) | Every endpoint, validation, and error scenario |
| [UI/UX Testing](./04-ui-ux-testing.md) | Responsive design, loading states, cross-browser |
| [Mobile Testing](./05-mobile-testing.md) | Touch targets, keyboard behavior, iOS Safari quirks |
| [Performance Testing](./06-performance-testing.md) | Database queries, memory leaks, load testing |
| [Accessibility Testing](./07-accessibility-testing.md) | WCAG AA compliance, keyboard nav, screen readers |
| [Known AI Code Failures](../bug-patterns/known-ai-code-failures.md) | Documented repeating bugs in AI-generated code |

---

*Shipped something using this checklist? Found an item that should be here? [Open a PR](../CONTRIBUTING.md) — this list grows through real-world experience.*
