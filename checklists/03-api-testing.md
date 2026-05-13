# 🔌 API Testing Checklist for AI-Generated Code

> **Why this checklist exists:** AI tools generate API endpoints quickly but consistently miss error handling, input validation, and edge cases. The endpoint works perfectly when called correctly — but breaks, leaks data, or causes silent failures when called in any other way. This checklist tests the full surface area of every API endpoint in your vibe-coded application.
>
> Items marked ⚠️ are the patterns AI tools get wrong most frequently. Items marked 🔴 are critical failures that can cause data loss, security breaches, or system outages.

---

## 1. Endpoint Existence and Routing

AI sometimes generates references to endpoints it did not actually create, and scaffolds routes it did not fully implement.

- [ ] ⚠️ Every endpoint referenced in the frontend code actually exists on the backend — test each one with a real request
- [ ] ⚠️ Every endpoint referenced in the README, docs, or comments actually exists — AI frequently documents endpoints it planned but did not build
- [ ] Endpoints return the correct HTTP method — a `GET` request to a `POST`-only endpoint returns `405 Method Not Allowed`, not a 500 error or silent failure
- [ ] ⚠️ No endpoint returns `200 OK` for a request that actually failed — check that success status codes are only returned on actual success
- [ ] Routes that do not exist return `404`, not a blank response or a 500 error
- [ ] API versioning is consistent — if the API uses `/v1/`, all endpoints follow the same convention with no orphaned unversioned routes

---

## 2. Request Validation

Test what happens when the API receives unexpected, malformed, or malicious input.

### Missing and Invalid Fields
- [ ] Send a request with the entire request body missing — should return `400 Bad Request` with a clear message
- [ ] ⚠️ Send a request with each required field removed one at a time — each should return a specific `400` error identifying the missing field, not a `500` crash
- [ ] Send a request with extra/unexpected fields in the body — the API should ignore them, not error or store them
- [ ] Send a request with the correct fields but wrong data types (e.g., a string where a number is expected) — should return `400`, not a `500`

### Field-Level Validation
- [ ] ⚠️ String fields: test with empty string `""`, whitespace only `"   "`, extremely long strings (10,000+ characters), and special characters
- [ ] Numeric fields: test with `0`, negative numbers, decimals where integers are expected, and extremely large numbers
- [ ] Email fields: test with missing `@`, missing domain, and clearly invalid formats
- [ ] Date fields: test with invalid dates, past dates where future is required, and incorrect formats
- [ ] ⚠️ Enum/dropdown fields: test with a value that is not in the allowed list — should return `400`, not silently store the invalid value
- [ ] Boolean fields: test with strings `"true"` and `"false"` instead of actual booleans

### Content Type
- [ ] Sending `Content-Type: text/plain` instead of `application/json` to a JSON endpoint returns `415 Unsupported Media Type` or a clear error
- [ ] ⚠️ An empty `Content-Type` header does not cause a `500` crash

---

## 3. Response Validation

Test what the API returns, not just whether it responds.

### Status Codes
- [ ] Successful creation returns `201 Created`, not `200 OK`
- [ ] Successful deletion returns `204 No Content` (or `200` with confirmation), not `200` with an empty body that looks like a failure
- [ ] ⚠️ Unauthorized requests return `401 Unauthorized`, not `403 Forbidden` — these are different and should be used correctly
- [ ] Forbidden requests (authenticated but not permitted) return `403 Forbidden`
- [ ] ⚠️ AI-generated APIs often return `200 OK` with `{ success: false }` in the body for errors — this breaks standard API clients and should use proper status codes instead

### Response Body
- [ ] ⚠️ Response bodies do not include sensitive fields that were not requested: `password_hash`, `internal_id`, `admin_notes`, `raw_token`, etc.
- [ ] Response field names are consistent across all endpoints — AI sometimes uses `userId` in one endpoint and `user_id` in another
- [ ] Null/empty responses are explicit — a `null` result returns `404` or an empty array `[]`, not an ambiguous empty `200` body
- [ ] ⚠️ List endpoints always return an array, even when there are zero results — never return `null` where an empty array is expected
- [ ] Date and time values are returned in a consistent format (preferably ISO 8601: `2026-05-13T14:30:00Z`) across all endpoints
- [ ] Numeric values are returned as numbers, not strings

### Error Responses
- [ ] All error responses follow a consistent structure (e.g., `{ error: "message", code: "ERROR_CODE" }`) — AI often returns different formats for different types of errors
- [ ] Error messages are descriptive enough to help a developer debug but do not expose internal system details (no stack traces, no database query text)
- [ ] ⚠️ Validation errors identify which field failed and why — not just "validation failed"

---

## 4. Authentication and Authorization on Endpoints

> See also: [Security Testing Checklist — Section 2](./02-security-testing.md#2-authorization-and-access-control) for full authorization testing.

- [ ] 🔴 Every protected endpoint returns `401` when called with no auth token
- [ ] 🔴 Every protected endpoint returns `401` when called with an expired token
- [ ] 🔴 Every protected endpoint returns `401` when called with a malformed or fake token
- [ ] ⚠️ Test every endpoint — not just the ones with "admin" or "private" in the name. AI frequently forgets to add auth middleware to non-obvious endpoints
- [ ] `OPTIONS` preflight requests do not require authentication — they must succeed for CORS to work
- [ ] ⚠️ The auth check happens before any database query — a `401` response should not trigger database activity for unauthorized requests

---

## 5. Hallucinated and Broken External API Calls

This is one of the most unique failure modes in AI-generated code — the AI generates calls to external APIs using endpoint structures, parameters, or response formats it invented.

- [ ] ⚠️ For every external API call in the codebase, verify the endpoint URL against the official documentation of that service
- [ ] ⚠️ Verify the request parameters match the actual API spec — AI often adds parameters that do not exist or omits required ones
- [ ] ⚠️ Verify the response parsing matches the actual API response structure — AI sometimes assumes a response field exists that the API does not return
- [ ] Test each external API call in isolation (using a tool like Postman) to confirm it works before testing the full application flow
- [ ] ⚠️ Check that API keys for external services are valid and have the correct permissions — AI sometimes generates placeholder key formats that look real but are not
- [ ] Verify rate limits for external APIs — AI-generated code rarely implements retry logic or rate limit handling
- [ ] ⚠️ Webhook URLs registered with third-party services actually point to a real, working endpoint in the application

---

## 6. Rate Limiting and Abuse Prevention

- [ ] ⚠️ Endpoints that perform expensive operations (search, file processing, email sending) have rate limiting applied
- [ ] The rate limiter returns `429 Too Many Requests` with a `Retry-After` header, not a generic `500` error
- [ ] ⚠️ Authentication endpoints (login, password reset, register) have stricter rate limiting than general endpoints
- [ ] Rate limiting is applied per user/token for authenticated endpoints, not just per IP
- [ ] Bulk operation endpoints (e.g., import, export, batch delete) have appropriate limits on the number of records per request

---

## 7. Pagination, Filtering, and Sorting

AI generates list endpoints that work for small datasets but break or become dangerous at scale.

- [ ] ⚠️ List endpoints are paginated — an endpoint that returns all records without a limit will fail or cause timeouts when the dataset grows
- [ ] Pagination parameters (`page`, `limit`, `offset`, `cursor`) are validated — test with `limit=0`, `limit=-1`, and `limit=99999`
- [ ] ⚠️ The maximum page size is enforced server-side — a user cannot bypass pagination by sending `limit=999999` in the request
- [ ] Filter parameters only filter on allowed fields — test filtering on fields like `password` or `internal_id` that should not be filterable
- [ ] Sort parameters only sort on allowed fields — test sorting on sensitive or non-indexed fields
- [ ] ⚠️ Sorting and filtering work correctly together — apply both at the same time and verify the results
- [ ] Total count is returned alongside paginated results so clients know how many pages exist

---

## 8. Idempotency and Duplicate Requests

- [ ] ⚠️ Submitting the same `POST` request twice (e.g., double-clicking submit) does not create two records — AI rarely implements idempotency
- [ ] A duplicate payment, order, or transaction cannot be created by re-submitting the same request
- [ ] `PUT` requests are idempotent — sending the same update twice results in the same final state, not two different states
- [ ] ⚠️ Retrying a failed request after a network error does not cause duplicate side effects (duplicate emails sent, duplicate charges, etc.)

---

## 9. Data Relationships and Integrity

- [ ] ⚠️ Deleting a parent record handles related child records — they are either deleted (cascade), blocked (foreign key constraint), or reassigned, not left as orphaned records
- [ ] Creating a record with a reference to a non-existent related record returns a clear error, not a silent failure or `500`
- [ ] ⚠️ Updating a record's relationship (e.g., reassigning an item to a different user) is validated — you cannot reassign records to users who do not exist
- [ ] Concurrent updates to the same record are handled — test by sending two conflicting `PUT` requests at the same time

---

## 10. File Upload Endpoints (if applicable)

- [ ] ⚠️ File size limits are enforced server-side — test by uploading a file larger than the stated limit
- [ ] File type validation happens server-side by checking the actual file content (MIME type), not just the file extension
- [ ] Upload endpoint returns the URL or identifier of the uploaded file in the response
- [ ] ⚠️ Uploaded files are stored in a non-executable location — cannot be executed by navigating to their URL
- [ ] Files are stored with unpredictable names (not sequential IDs) so they cannot be enumerated
- [ ] Uploading a malformed file (e.g., a text file renamed as `.jpg`) is handled gracefully

---

## 11. API Documentation Accuracy (if applicable)

- [ ] ⚠️ Every endpoint in the API documentation actually exists and works — AI often generates docs and code independently, causing them to diverge
- [ ] Request and response examples in the documentation match what the API actually returns
- [ ] Error codes and messages in the documentation match what the API actually returns
- [ ] Authentication requirements are documented correctly for every endpoint

---

## 12. Quick API Smoke Test (Run After Every Deployment)

Run these against every new deployment before considering it stable:

- [ ] `GET /health` or equivalent health check endpoint returns `200` — confirms the server is running
- [ ] A simple authenticated request succeeds with a valid token
- [ ] A simple authenticated request fails with `401` when no token is provided
- [ ] The most-used `GET` endpoint returns data in the expected format
- [ ] The most-used `POST` endpoint creates a record successfully
- [ ] No endpoint returns a `500` error for a well-formed request

---

## Recommended Tools for API Testing

| Tool | Best For | Cost |
|---|---|---|
| [Postman](https://www.postman.com) | Manual endpoint testing and collections | Free tier |
| [Hoppscotch](https://hoppscotch.io) | Lightweight browser-based API testing | Free |
| [REST Client (VS Code)](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) | Testing from within your editor | Free |
| [k6](https://k6.io) | Load and performance testing of APIs | Free |
| [Dredd](https://dredd.org) | Validating API against OpenAPI/Swagger spec | Free |
| [Schemathesis](https://github.com/schemathesis/schemathesis) | Automated property-based API testing | Free |

---

## Related Checklists

- ✅ [Functional Testing](./01-functional-testing.md) — end-to-end user flow verification
- 🔐 [Security Testing](./02-security-testing.md) — auth, secrets, and injection vulnerabilities
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Encountered an API failure pattern in AI-generated code not listed here? [Open a PR](../CONTRIBUTING.md) — real-world additions make this more useful for everyone.*
