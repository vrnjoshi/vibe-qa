# Bug Report Template for AI-Generated Apps

> **How to use this template:** Copy and fill in one report per bug. Be as specific as possible — a vague bug report gets fixed slowly or not at all. The "AI Context" section is unique to this template and helps identify whether the bug is a known AI code failure pattern.

---

## Bug Summary

**Title:** _(One clear sentence: what is wrong, where, and what happens)_
> Example: "Admin dashboard accessible without authentication on /admin/users route"

---

## Classification

| Field | Value |
|---|---|
| **Severity** | 🔴 Critical / 🟡 High / 🟢 Medium / ⚪ Low |
| **Priority** | Must fix before launch / Fix in next release / Nice to have |
| **Type** | Security / Functional / UI / Performance / Accessibility / Data |
| **Reported by** | |
| **Date found** | |
| **Status** | Open / In Progress / Fixed / Verified / Won't Fix |

**Severity Guide:**
- 🔴 **Critical** — Security vulnerability, data loss, app crashes, core flow completely broken
- 🟡 **High** — Major feature broken, significant user impact, workaround is difficult
- 🟢 **Medium** — Feature partially broken, workaround exists, affects some users
- ⚪ **Low** — Cosmetic issue, minor inconvenience, affects edge case only

---

## Environment

| Field | Value |
|---|---|
| **App URL** | |
| **Environment** | Production / Staging / Development |
| **Browser** | _(e.g., Chrome 124, Safari 17, Firefox 125)_ |
| **Device / OS** | _(e.g., iPhone 15 iOS 17, Windows 11, MacBook M2)_ |
| **Screen size** | _(e.g., 375px mobile, 1440px desktop)_ |
| **User account type** | _(e.g., regular user, admin, new user)_ |

---

## Steps to Reproduce

_List exact steps. Be specific enough that someone who has never seen the app can reproduce the bug._

1. _(e.g., Open the app in an incognito browser window — do not log in)_
2. _(e.g., Navigate directly to: `https://app.example.com/admin/users`)_
3. _(e.g., Observe the result)_

**Reproducibility:**
- [ ] Always (100% of attempts)
- [ ] Often (>50% of attempts)
- [ ] Sometimes (<50% of attempts)
- [ ] Rarely (hard to reproduce consistently)

---

## Expected Result

_What should happen when these steps are followed?_

> Example: "The user should be redirected to the login page at /login since they are not authenticated."

---

## Actual Result

_What actually happens?_

> Example: "The admin user management panel is fully displayed, including a list of all user accounts with their email addresses and roles. No authentication is required."

---

## Evidence

_Attach or link screenshots, screen recordings, or console logs. For security bugs, describe the evidence without including sensitive data._

- [ ] Screenshot attached
- [ ] Screen recording attached
- [ ] Console error log included (below)
- [ ] Network request/response included (below)
- [ ] No visual evidence (describe in Actual Result above)

**Console errors (if any):**
```
(paste browser console errors here)
```

**Relevant network request/response (if applicable):**
```
Request:  GET /admin/users
Headers:  (no Authorization header)
Response: 200 OK
Body:     { "users": [...] }
```

---

## AI Context

_This section is unique to vibe-qa. It helps identify whether this bug matches a known AI code failure pattern._

**Was this feature AI-generated?**
- [ ] Yes — entirely AI-generated
- [ ] Yes — AI-assisted (human reviewed)
- [ ] No — human written
- [ ] Unknown

**AI tool used (if known):** _(e.g., Claude Code, Cursor, GitHub Copilot, Lovable)_

**Does this match a known AI failure pattern?**
_(Check the [Known AI Code Failures](../bug-patterns/known-ai-code-failures.md) document)_

- [ ] Pattern #1 — Client-Side Authentication
- [ ] Pattern #2 — N+1 Database Query
- [ ] Pattern #3 — Orphaned Records on Delete
- [ ] Pattern #4 — Hardcoded Development Values
- [ ] Pattern #5 — Happy Path Only
- [ ] Pattern #6 — Missing Rate Limiting
- [ ] Pattern #8 — Hallucinated API Endpoints
- [ ] Pattern #10 — Wildcard CORS
- [ ] Pattern #12 — Missing Server-Side Validation
- [ ] Pattern #13 — Double Submit
- [ ] Other pattern: _____
- [ ] Does not match a known pattern — consider [contributing it](../CONTRIBUTING.md)

---

## Suggested Fix

_(Optional — fill in if you have a clear idea of the root cause and solution)_

> Example: "The /admin/users API endpoint needs server-side authentication middleware. The frontend hides the link for non-admin users but the endpoint itself has no auth check. Add `requireAuth` and `requireAdmin` middleware to the route definition."

**Reference:**
- [ ] Fix described in [Known AI Code Failures](../bug-patterns/known-ai-code-failures.md) — Pattern #___
- [ ] Fix described in [Security Testing Checklist](../checklists/02-security-testing.md)
- [ ] External reference: _____

---

## Verification Steps

_When the fix is deployed, how do you verify it is resolved? Fill this in when reporting, so the verifier knows exactly what to check._

1. _(e.g., Open an incognito window without logging in)_
2. _(e.g., Navigate directly to /admin/users)_
3. _(e.g., Confirm you are redirected to /login with a 401 response in the Network tab)_
4. _(e.g., Log in as a regular (non-admin) user and repeat — confirm you receive a 403 response)_

---

*Template maintained by [vibe-qa](https://github.com/vrnjoshi/vibe-qa) — the QA handbook for AI-generated code.*
