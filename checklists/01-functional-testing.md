# ✅ Functional Testing Checklist for AI-Generated Code

> **How to use this checklist:** Work through every section before marking a feature complete. Each item is written as a test to perform, not just a concept to be aware of. Check items off as you verify them. Items marked ⚠️ are the ones AI tools most commonly get wrong.

---

## 1. Core Feature Verification

These checks confirm that what the AI built actually matches what you asked for.

- [ ] Every feature described in the original prompt exists and is reachable by the user
- [ ] Features work end-to-end, not just in isolation (e.g., a form that submits AND saves AND shows confirmation)
- [ ] ⚠️ Features work when accessed in a different order than the one you tested during development
- [ ] ⚠️ Features work when the user is NOT in the "first time user" state (e.g., returning user with existing data)
- [ ] Any feature that was working earlier still works after new code was added (regression check)
- [ ] All buttons, links, and interactive elements do something — none are placeholder/dead UI
- [ ] All visible text is real content, not Lorem Ipsum or placeholder copy left in by the AI

---

## 2. Input Handling and Validation

AI almost always builds for the happy path. This section tests everything that isn't the happy path.

### Empty and Missing Inputs
- [ ] Submit a form with every field left empty — does the app handle it gracefully?
- [ ] Submit a form with only required fields filled — does it accept it?
- [ ] Submit a form with only optional fields filled and required fields empty — does it reject it with a clear message?
- [ ] ⚠️ Leave a single required field empty while filling all others — test each field individually

### Boundary Values
- [ ] Enter a value of exactly 0 in any numeric field
- [ ] Enter a negative number in any numeric field that should only accept positive values
- [ ] Enter the maximum allowed value and one above it
- [ ] Enter the minimum allowed value and one below it
- [ ] ⚠️ Enter extremely large numbers (e.g., 999999999) in any number input
- [ ] Enter decimal values in fields that expect whole numbers

### Text Input Edge Cases
- [ ] Enter a single character in text fields
- [ ] ⚠️ Enter a very long string (500+ characters) in any text field — does the UI break or truncate correctly?
- [ ] Enter only spaces in text fields — should not be treated as valid input
- [ ] Enter special characters: `!@#$%^&*()_+-=[]{}|;':",.<>?/\`
- [ ] ⚠️ Enter HTML tags as input: `<script>alert('test')</script>` — should be displayed as text, never executed
- [ ] Enter emoji characters 🎉 in text fields
- [ ] Enter text in a non-Latin script (e.g., Arabic, Chinese, Japanese) if internationalization matters
- [ ] Copy-paste text that contains hidden characters or line breaks

### Date and Time Inputs
- [ ] Enter a date in the past where a future date is expected
- [ ] Enter a date far in the future (e.g., 2099)
- [ ] Enter an end date that is before the start date
- [ ] ⚠️ Enter February 29 — does it handle leap years correctly?
- [ ] Test time zone edge cases if the app is used across time zones

### File Uploads (if applicable)
- [ ] Upload a file that exceeds the size limit
- [ ] Upload a file type that is not allowed
- [ ] Upload an empty file (0 bytes)
- [ ] ⚠️ Upload a file with a double extension (e.g., `image.png.exe`)
- [ ] Upload a file with a very long filename
- [ ] Upload multiple files simultaneously if that is supported

---

## 3. AI-Specific Logic Failures

These are the failure patterns that appear specifically in AI-generated code. Check each one carefully.

- [ ] ⚠️ **Happy path only:** Manually trace every `if/else` branch in the AI-generated logic. AI frequently writes the `if` branch correctly and writes a placeholder or incorrect `else` branch
- [ ] ⚠️ **Loop termination:** Any AI-generated loop (for, while, forEach) has a valid exit condition. Verify by testing with 0 items, 1 item, and a large number of items
- [ ] ⚠️ **Hardcoded assumptions:** Search the codebase for hardcoded values (user IDs, email addresses, URLs, dates) that the AI used during generation and forgot to replace
- [ ] ⚠️ **Fake function calls:** Verify that every function the AI calls actually exists. AI sometimes generates a call to a function it did not create
- [ ] ⚠️ **Hallucinated API endpoints:** Test every external API call. Verify the endpoint URL is real and the request/response format matches the actual API documentation
- [ ] ⚠️ **Conditional logic inversion:** Test the "negative" case for every condition. AI sometimes inverts conditions (using `===` when it should be `!==`, or `>` when it should be `<`)
- [ ] ⚠️ **Console.log in production:** Search for `console.log`, `print()`, `debugger`, or similar debug statements that AI left in the code. These can expose sensitive data in production
- [ ] ⚠️ **TODO and FIXME comments:** Search the codebase for `TODO`, `FIXME`, `HACK`, and `PLACEHOLDER` — AI often generates these to mark code it did not finish

---

## 4. User Flows (End-to-End Paths)

Test the complete journey, not individual steps. AI builds individual components well but misses the connections between them.

### Authentication Flows
- [ ] New user can register successfully with valid data
- [ ] New user sees a clear error when registering with an already-used email
- [ ] Existing user can log in with correct credentials
- [ ] ⚠️ Existing user sees a clear, non-technical error message with incorrect credentials (not a 500 error or stack trace)
- [ ] Logged-in user can log out and is redirected correctly
- [ ] After logging out, the browser back button does NOT show protected pages
- [ ] ⚠️ Password reset flow works completely: request → email → link → new password → login
- [ ] Session expires after the expected timeout period

### Create, Read, Update, Delete (CRUD) Flows
- [ ] Create: A new record can be created and immediately appears in the list/view
- [ ] Read: The created record shows all the data that was entered, not placeholder or default data
- [ ] Update: Editing a record saves the new values and the old values are gone
- [ ] ⚠️ Delete: Deleting a record removes it from all places it appears, not just the current view
- [ ] ⚠️ Delete: Deleting a record also removes or handles its related/linked data (orphan records)
- [ ] Performing a CRUD operation and immediately navigating away and back shows the correct updated state

### Navigation Flows
- [ ] Every link in the navigation goes to the correct destination
- [ ] ⚠️ Navigating directly to a URL (deep linking) works and does not show a blank page or error
- [ ] The browser back button works and shows the correct previous page
- [ ] Refreshing the page on any route works and does not lose the user's context
- [ ] ⚠️ Navigating to a URL that does not exist shows a proper 404 page, not a blank white screen

---

## 5. Error States and Messaging

AI generates success states well. It generates error states poorly. Deliberately trigger every error below.

- [ ] ⚠️ Disconnect the internet (or disable the API) while the app is running — does it show a user-friendly message or crash?
- [ ] ⚠️ Submit a form while the network is slow (use browser DevTools to throttle to "Slow 3G") — does the button disable to prevent duplicate submissions?
- [ ] Trigger a server error (500) — does the user see a friendly message or a raw error/stack trace?
- [ ] ⚠️ What happens when an API returns no data? Does the app show "no results found" or does it break?
- [ ] All error messages are in plain language, not developer jargon (no "TypeError", "null reference", "500 Internal Server Error" visible to users)
- [ ] ⚠️ Error messages are specific enough to help the user fix the problem (not just "Something went wrong")
- [ ] After an error, the user can retry the action without refreshing the entire page
- [ ] Loading states are shown during any operation that takes more than 1 second

---

## 6. Data Integrity

AI-generated code frequently causes silent data corruption. These checks find it before users do.

- [ ] ⚠️ Data saved in one part of the app appears correctly in all other parts of the app that display it
- [ ] ⚠️ Updating a record in one place updates it everywhere it is shown (no stale data displayed)
- [ ] Numbers are stored and displayed with the correct precision (e.g., currency is not rounded incorrectly)
- [ ] ⚠️ Concurrent actions: open the same record in two browser tabs, edit it in both, and save — does the app handle the conflict or silently overwrite?
- [ ] Dates and times are stored and displayed in the correct time zone
- [ ] ⚠️ Sorting and filtering: sorted lists are actually sorted correctly (AI often generates sort logic that works for simple cases but fails for edge cases like identical values or null values)
- [ ] Pagination: page 2 does not show the same records as page 1
- [ ] Search results are accurate and complete — no records that match are missing

---

## 7. State Management

These checks verify that the app correctly tracks and updates its own state.

- [ ] ⚠️ After completing an action, the UI updates to reflect the new state without requiring a manual page refresh
- [ ] A counter, badge, or indicator that shows a count (e.g., items in cart, unread notifications) updates immediately after the relevant action
- [ ] ⚠️ Undo/undo history (if the app has it) works correctly and does not leave the app in an inconsistent state
- [ ] Navigating away from a form with unsaved changes warns the user (if appropriate)
- [ ] ⚠️ Filters and search terms applied on a list persist correctly if the user navigates away and returns
- [ ] Multi-step forms/wizards: going back to a previous step retains the data entered in later steps

---

## 8. Multi-User and Permission Scenarios

- [ ] ⚠️ A regular user cannot access admin-only features by navigating directly to the URL
- [ ] A user can only see their own data, not other users' data
- [ ] ⚠️ Changing a user's role or permissions takes effect immediately without requiring re-login
- [ ] A deactivated or deleted user cannot log in
- [ ] Shared resources (e.g., a document shared with multiple users) show the correct version to all users

---

## 9. Smoke Test (Run After Every Deployment)

These are the 10 most critical checks to run quickly after any new code is deployed.

- [ ] The app loads without a blank screen or console errors
- [ ] A user can log in
- [ ] The most-used feature works end-to-end
- [ ] A user can log out
- [ ] The homepage/dashboard loads correctly for a new user
- [ ] The most recent change that was deployed works as expected
- [ ] No debug/test data is visible in the production environment
- [ ] The app works on mobile screen size
- [ ] All navigation links resolve (no 404s in main navigation)
- [ ] No obvious UI elements are broken or misaligned

---

## Reporting Issues Found

Use the [Bug Report Template](../test-templates/bug-report-template.md) to document any issues found during this checklist. Include:
- Which checklist item failed
- Steps to reproduce
- Expected vs actual behavior
- Screenshot or screen recording if possible

---

## Related Checklists

- 🔐 [Security Testing](./02-security-testing.md) — for auth, input sanitization, and vulnerability checks
- 🔌 [API Testing](./03-api-testing.md) — for backend endpoint verification
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found a failure pattern not covered here? [Open a PR](../CONTRIBUTING.md) — every real-world addition makes this more useful for everyone.*
