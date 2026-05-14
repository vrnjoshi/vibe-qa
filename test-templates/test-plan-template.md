# Test Plan Template for AI-Generated Apps

> **How to use this template:** Copy this file into your project's `/docs` folder and fill in each section before testing a new feature or release. Delete sections that do not apply. A completed test plan does not need to be long — it needs to be honest about what will and will not be tested.

---

## Project Information

| Field | Value |
|---|---|
| **App / Feature Name** | _(e.g., "User Authentication Flow")_ |
| **Version / Release** | _(e.g., "v1.2.0" or "May 2026 release")_ |
| **AI Tool(s) Used** | _(e.g., "Claude Code, GitHub Copilot")_ |
| **Test Author** | |
| **Test Start Date** | |
| **Target Launch Date** | |
| **Environment** | _(e.g., "Staging at staging.myapp.com")_ |

---

## 1. What Is Being Tested

Describe the feature or release in 2–3 sentences. Be specific about scope.

> Example: "The new user registration and email verification flow, including the registration form, welcome email, email verification link, and first-login redirect. This was built entirely with Claude Code."

**In scope:**
- _(List specific features, pages, or endpoints being tested)_

**Out of scope:**
- _(List what is explicitly NOT being tested in this cycle and why)_

---

## 2. AI-Specific Risk Areas

Because this code was AI-generated, identify the specific risk areas most likely to have issues. Check all that apply and add notes.

- [ ] **Authentication / Authorization** — AI may have placed auth checks on the frontend only
- [ ] **Input validation** — AI may only have validated on the client side
- [ ] **Error handling** — AI may only have built the happy path
- [ ] **Database operations** — Risk of N+1 queries, orphaned records, or missing transactions
- [ ] **External API calls** — Risk of hallucinated endpoints or incorrect request formats
- [ ] **Security** — Hardcoded secrets, CORS misconfiguration, or missing rate limiting
- [ ] **Mobile / responsive** — AI likely only tested at one screen size
- [ ] **Accessibility** — AI likely did not implement keyboard navigation or ARIA
- [ ] **Performance** — AI may not have considered scale or concurrent users
- [ ] Other: _____

---

## 3. Test Scenarios

List the specific scenarios to be tested. For each scenario, write it as: **Given** [starting state] **When** [action] **Then** [expected result].

### Happy Path Scenarios
_(What should work when everything goes correctly)_

| # | Scenario | Priority |
|---|---|---|
| 1 | Given a new visitor, when they complete the registration form with valid data, then they receive a welcome email and are redirected to the dashboard | High |
| 2 | _(Add your scenarios)_ | |

### Edge Case Scenarios
_(Boundary conditions and unusual but valid inputs)_

| # | Scenario | Priority |
|---|---|---|
| 1 | Given a registration form, when the email field contains a valid email with a subdomain (user@mail.example.com), then registration succeeds | Medium |
| 2 | _(Add your scenarios)_ | |

### Negative / Failure Scenarios
_(What should fail gracefully)_

| # | Scenario | Priority |
|---|---|---|
| 1 | Given a registration form, when the email is already registered, then a clear error message is shown and no duplicate account is created | High |
| 2 | Given a registration form, when all fields are empty and submit is clicked, then each required field shows a specific validation error | High |
| 3 | _(Add your scenarios)_ | |

### AI-Specific Scenarios
_(Scenarios targeting known AI code failure patterns)_

| # | Scenario | Priority |
|---|---|---|
| 1 | Given a logged-out user, when they navigate directly to /dashboard, then they are redirected to login (not shown the dashboard) | Critical |
| 2 | Given a regular user, when they call the admin API endpoint directly with their token, then they receive a 403 error | Critical |
| 3 | _(Add your scenarios)_ | |

---

## 4. Checklists to Run

Select which checklists from this repo apply to this test cycle:

- [ ] [Functional Testing](../checklists/01-functional-testing.md)
- [ ] [Security Testing](../checklists/02-security-testing.md)
- [ ] [API Testing](../checklists/03-api-testing.md)
- [ ] [UI/UX Testing](../checklists/04-ui-ux-testing.md)
- [ ] [Mobile Testing](../checklists/05-mobile-testing.md)
- [ ] [Performance Testing](../checklists/06-performance-testing.md)
- [ ] [Accessibility Testing](../checklists/07-accessibility-testing.md)
- [ ] [Pre-Launch Final Checklist](../checklists/08-pre-launch-final.md)
- [ ] [Known AI Code Failures](../bug-patterns/known-ai-code-failures.md) — review applicable patterns

---

## 5. Test Environment and Data

**Test environment URL:** _______________

**Test accounts needed:**

| Account Type | Email | Purpose |
|---|---|---|
| Regular user | test-user@example.com | Standard flow testing |
| Admin user | test-admin@example.com | Admin feature testing |
| New user | _(create fresh during testing)_ | Registration flow |

**Test data setup:**
_(Note any data that needs to exist before testing begins — e.g., existing orders, specific product catalog, etc.)_

**Browsers / devices to test:**
- [ ] Chrome (desktop)
- [ ] Safari (desktop)
- [ ] Firefox (desktop)
- [ ] Safari (iOS — iPhone)
- [ ] Chrome (Android)
- [ ] Other: _____

---

## 6. Pass / Fail Criteria

**This release passes testing if:**
- All Critical and High priority scenarios pass
- No Critical or High severity bugs are open at launch
- The [Pre-Launch Final Checklist](../checklists/08-pre-launch-final.md) passes completely
- _(Add any project-specific criteria)_

**This release will be delayed if:**
- Any Critical severity bug is found
- The core user flow fails end-to-end in the production environment
- Any security check in the Pre-Launch checklist fails
- _(Add any project-specific blockers)_

---

## 7. Bugs Found

Log bugs found during this test cycle. Use the [Bug Report Template](./bug-report-template.md) for full details.

| # | Summary | Severity | Status | Link |
|---|---|---|---|---|
| 1 | _(e.g., "Admin panel accessible without auth")_ | Critical | Open | |
| 2 | | | | |

---

## 8. Test Summary

_Fill this in when testing is complete._

| Metric | Count |
|---|---|
| Total scenarios tested | |
| Scenarios passed | |
| Scenarios failed | |
| Bugs found | |
| Bugs fixed before launch | |
| Bugs deferred to next release | |

**Overall assessment:**
- [ ] ✅ Ready to launch — all criteria met
- [ ] ⚠️ Launch with known issues — risks accepted and documented
- [ ] 🔴 Not ready — critical issues must be resolved first

**Notes:**
_(Any observations, concerns, or recommendations for the next cycle)_

---

*Template maintained by [vibe-qa](https://github.com/vrnjoshi/vibe-qa) — the QA handbook for AI-generated code.*
