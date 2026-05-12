# 🔐 Security Testing Checklist for AI-Generated Code

> **Why this checklist exists:** Research shows AI co-authored code has 2.74x more security vulnerabilities than human-written code. AI tools are optimized to make code that *works*, not code that is *secure*. The patterns in this checklist are not theoretical — they are documented, repeating failures found in production vibe-coded applications. Work through every section before shipping anything to real users.
>
> Items marked ⚠️ are the vulnerabilities AI tools produce most frequently. Items marked 🔴 are critical — a single failure here can result in a full account takeover, data breach, or system compromise.

---

## 1. Authentication and Session Management

AI generates login flows that look correct but are frequently broken at the security layer.

### Credential Handling
- [ ] 🔴 Passwords are never stored in plain text — verify that the database stores a hashed value (bcrypt, argon2, or scrypt), not the original password
- [ ] 🔴 ⚠️ Password hashing happens on the **server side**, not the client side. AI sometimes generates client-side hashing which provides no real protection
- [ ] Password reset tokens are randomly generated, not predictable (not based on user ID, timestamp, or email)
- [ ] Password reset tokens expire — test that a token from 30+ minutes ago is rejected
- [ ] ⚠️ Password reset tokens are single-use — test that using a reset link a second time is rejected
- [ ] The forgot password flow does not reveal whether an email address is registered (both "email found" and "email not found" states should return the same generic message)
- [ ] There is no way to enumerate valid usernames or emails through login error messages

### Session Security
- [ ] 🔴 ⚠️ Session tokens or JWTs are validated on the **server side** on every protected request, not just at login
- [ ] ⚠️ Logging out invalidates the session token on the server — test by capturing the token before logout, logging out, then making a request with the old token
- [ ] Session tokens are not stored in `localStorage` or `sessionStorage` — these are accessible to any JavaScript on the page. Prefer `HttpOnly` cookies
- [ ] ⚠️ JWTs: verify the `exp` (expiration) claim is checked server-side and expired tokens are rejected
- [ ] ⚠️ JWTs: the algorithm is explicitly set server-side. AI-generated JWT validation sometimes accepts `"alg": "none"` which disables signature verification entirely
- [ ] Sessions have a reasonable timeout for inactivity (typically 15–60 minutes depending on sensitivity)
- [ ] After password change, all existing sessions for that user are invalidated

### Brute Force Protection
- [ ] ⚠️ The login endpoint has rate limiting — test by sending 20+ login attempts in rapid succession. The endpoint should throttle or lock
- [ ] Account lockout triggers after a defined number of failed attempts (typically 5–10)
- [ ] The rate limiter is applied per user/email, not just per IP address (IP-based limiting alone is easily bypassed)
- [ ] CAPTCHA or similar challenge exists for high-frequency failure scenarios

---

## 2. Authorization and Access Control

This is the most critical security category for AI-generated code. AI frequently builds authentication (who you are) but skips or misplaces authorization (what you are allowed to do).

### Vertical Privilege Escalation (User → Admin)
- [ ] 🔴 ⚠️ Log in as a regular user. Try to access admin routes by navigating directly to their URL (e.g., `/admin`, `/dashboard/admin`, `/api/admin/users`). You should receive a 401 or 403, never the actual admin page
- [ ] 🔴 ⚠️ Authorization checks exist on the **server/API layer**, not just the frontend. Hiding an admin button in the UI is not authorization — verify the API endpoint itself rejects unauthorized requests
- [ ] Admin-only API endpoints return 403 when called with a regular user's token — test using a tool like Postman or browser DevTools with the regular user's auth token

### Horizontal Privilege Escalation (User A → User B's Data)
- [ ] 🔴 ⚠️ Log in as User A. Copy the ID of a record belonging to User B (e.g., `/api/orders/12345`). Make a direct request to that URL while authenticated as User A. You should receive a 403 or 404, never User B's data
- [ ] ⚠️ This check must be performed for every resource type: profiles, orders, files, messages, settings — AI often protects one resource type but misses others
- [ ] Update and delete operations also enforce ownership — test `PUT /api/records/OTHER_USER_ID` and `DELETE /api/records/OTHER_USER_ID` as an unauthorized user
- [ ] Search and list endpoints filter results to only the current user's data — a user cannot retrieve records belonging to others through search

### API Authorization
- [ ] ⚠️ Every API endpoint that returns, modifies, or deletes data has an authorization check — not just the ones with obvious names like `/admin`
- [ ] Removing or tampering with the `Authorization` header on any protected request returns 401, not the data
- [ ] ⚠️ GraphQL APIs: introspection is disabled in production. Authorization checks exist at the resolver level, not just at the route level
- [ ] Webhooks validate the signature of incoming requests — unauthenticated webhook calls are rejected

---

## 3. Secrets and Sensitive Data Exposure

AI tools leak secrets more frequently than any other vulnerability class. This section is the most important one to check before a first deployment.

### Hardcoded Secrets
- [ ] 🔴 ⚠️ Search the entire codebase for these strings and verify none appear as hardcoded values: `api_key`, `apikey`, `secret`, `password`, `token`, `private_key`, `access_key`, `auth_token`, `bearer`
- [ ] 🔴 ⚠️ Check `.env` files — they must **not** be committed to the repository. Check `.gitignore` includes `.env`, `.env.local`, `.env.production`
- [ ] Run `git log --all --full-history -- "*.env"` to check whether a `.env` file was ever committed, even if it was later deleted. Secrets in git history are compromised regardless of deletion
- [ ] ⚠️ AI-generated database connection strings do not contain real credentials in the source code
- [ ] No API keys, passwords, or tokens appear in frontend JavaScript files that are served to the browser
- [ ] ⚠️ Third-party service keys (Stripe, SendGrid, Twilio, AWS) are stored as environment variables and accessed server-side only

### Data Exposure in Responses
- [ ] ⚠️ API responses do not include fields that were not explicitly requested — AI-generated serializers often return entire database objects including sensitive fields like `password_hash`, `internal_id`, `admin_flag`
- [ ] The user profile endpoint does not return password hashes, security questions, or internal system fields
- [ ] Error responses do not include stack traces, database query details, or internal file paths in production
- [ ] ⚠️ Search and list endpoints do not return other users' private data even as partial matches
- [ ] Log files do not contain passwords, tokens, or sensitive personal data (check what gets logged on form submission)

### Transport Security
- [ ] All traffic is served over HTTPS in production — no mixed content (HTTP resources loaded on HTTPS pages)
- [ ] Sensitive cookies have the `Secure` flag set (only sent over HTTPS)
- [ ] `HttpOnly` flag is set on session cookies (prevents JavaScript access)
- [ ] The `SameSite` attribute is set on cookies (`Strict` or `Lax`) to mitigate CSRF

---

## 4. Injection Vulnerabilities

### SQL Injection
- [ ] 🔴 ⚠️ Test search fields, filters, and any input that queries a database by entering: `' OR '1'='1` and `'; DROP TABLE users; --`
- [ ] If the app does not crash or return unexpected data, verify that the database driver uses parameterized queries or prepared statements — not string concatenation to build queries
- [ ] ⚠️ AI-generated raw SQL queries must be audited manually — search the codebase for `query(` or `execute(` and check that user input is never directly interpolated into the query string
- [ ] ORM usage does not bypass parameterization through raw query escape hatches

### Cross-Site Scripting (XSS)
- [ ] 🔴 ⚠️ Enter `<script>alert('xss')</script>` in every text input that displays output back to the user (comments, profile names, search fields, messages). If an alert box appears, XSS is present
- [ ] ⚠️ AI-generated code that uses `innerHTML`, `dangerouslySetInnerHTML` (React), or `v-html` (Vue) is a high risk — audit every occurrence and verify the content is sanitized before rendering
- [ ] Stored XSS: content entered by one user and displayed to other users is properly escaped
- [ ] URL parameters displayed on the page are escaped — test by adding `?name=<script>alert(1)</script>` to URLs

### Command Injection
- [ ] ⚠️ If the application executes shell commands, system calls, or runs scripts based on user input, verify user input is never passed directly to the command. AI sometimes generates `exec(userInput)` patterns
- [ ] File path inputs are validated and cannot be used to traverse directories (e.g., `../../etc/passwd`)

---

## 5. Cross-Site Request Forgery (CSRF)

- [ ] ⚠️ State-changing requests (POST, PUT, DELETE) include CSRF token validation, or use the `SameSite=Strict` cookie attribute as a mitigation
- [ ] Test by making a state-changing API call from a different origin (or using a CSRF testing tool) — it should be rejected
- [ ] ⚠️ AI-generated APIs that use JSON bodies are still vulnerable to CSRF in some configurations — verify the Content-Type is validated server-side

---

## 6. Security Misconfigurations

These are the "left unlocked" vulnerabilities — things that are not bugs in logic but failures in configuration that AI tools routinely get wrong.

### CORS (Cross-Origin Resource Sharing)
- [ ] 🔴 ⚠️ The CORS policy is not set to `Access-Control-Allow-Origin: *` on any endpoint that requires authentication. A wildcard origin on an authenticated endpoint allows any website to make authenticated requests on behalf of a logged-in user
- [ ] ⚠️ Allowed origins are an explicit list of trusted domains, not a wildcard or dynamically reflected value from the request's `Origin` header
- [ ] Preflight OPTIONS requests are handled correctly

### HTTP Security Headers
Run your deployed URL through [securityheaders.com](https://securityheaders.com) and verify:
- [ ] `Content-Security-Policy` is set — prevents XSS and data injection attacks
- [ ] `X-Frame-Options` is set to `DENY` or `SAMEORIGIN` — prevents clickjacking
- [ ] `X-Content-Type-Options` is set to `nosniff`
- [ ] `Referrer-Policy` is set appropriately
- [ ] `Permissions-Policy` restricts access to sensitive browser features

### Server and Infrastructure
- [ ] ⚠️ Debug mode / development mode is disabled in production. AI-generated apps frequently have a `DEBUG=true` or `NODE_ENV=development` setting that exposes error details and internal routes
- [ ] Directory listing is disabled on the web server — navigating to `/uploads/` or `/static/` should not show a file listing
- [ ] ⚠️ Default credentials have been changed on any third-party services, databases, or admin panels configured by AI (e.g., default admin/admin, root/root)
- [ ] Unused API endpoints, routes, and admin panels from scaffolding or boilerplate have been removed
- [ ] Database is not publicly accessible — it should only accept connections from the application server
- [ ] ⚠️ Cloud storage buckets (S3, GCS, Azure Blob) are not publicly readable unless specifically intended — AI-generated infrastructure code frequently creates public buckets by default

---

## 7. Input Validation and Business Logic

- [ ] ⚠️ Server-side validation exists for all inputs — client-side validation is a user experience feature, not a security control. Bypass it by sending requests directly to the API
- [ ] Price or quantity values cannot be manipulated — test by modifying the value in an API request (e.g., setting price to 0 or -1 in a purchase request)
- [ ] ⚠️ File uploads are validated server-side for both file type (MIME type, not just extension) and file size
- [ ] Uploaded files are not stored in a publicly executable directory — a user cannot upload a `.php` or `.js` file and then execute it by navigating to its URL
- [ ] ⚠️ Mass assignment is prevented — the API does not allow users to set fields they should not have access to (e.g., setting `is_admin: true` in a profile update request)
- [ ] Numeric inputs used in financial calculations are validated for type, range, and precision before processing

---

## 8. Dependency and Supply Chain Security

- [ ] ⚠️ Run `npm audit` or `pip-audit` or the equivalent for your language — AI-generated dependency lists sometimes include outdated packages with known vulnerabilities
- [ ] Dependencies are pinned to specific versions, not `latest` or open ranges, to prevent unexpected updates introducing vulnerabilities
- [ ] No abandoned or untrusted packages are in the dependency list — AI sometimes generates imports of packages that no longer exist or have been taken over
- [ ] ⚠️ Review any package added by AI that you did not explicitly request — AI occasionally adds unnecessary dependencies

---

## 9. Quick Security Scan (Minimum Before Every Deployment)

If you do nothing else, run these five checks before going live:

- [ ] 🔴 Search codebase for hardcoded secrets: `grep -r "api_key\|password\|secret\|token" --include="*.js" --include="*.py" --include="*.ts" .`
- [ ] 🔴 Confirm `.env` is in `.gitignore` and was never committed: `git log --all -- "**/.env"`
- [ ] 🔴 Test that a logged-out user cannot access any protected page or API endpoint by navigating directly to it
- [ ] 🔴 Test that User A cannot access User B's data by changing an ID in the URL or API request
- [ ] 🔴 Run your deployed URL through [Mozilla Observatory](https://observatory.mozilla.org) — aim for a B grade or higher before launch

---

## Tools Referenced in This Checklist

| Tool | Purpose | Cost |
|---|---|---|
| [securityheaders.com](https://securityheaders.com) | Scan HTTP security headers | Free |
| [Mozilla Observatory](https://observatory.mozilla.org) | Overall security scan | Free |
| [OWASP ZAP](https://www.zaproxy.org) | Active security scanning | Free |
| [truffleHog](https://github.com/trufflesecurity/trufflehog) | Scan git history for leaked secrets | Free |
| [Snyk](https://snyk.io) | Dependency vulnerability scanning | Free tier available |
| [Burp Suite Community](https://portswigger.net/burp/communitydownload) | Manual security testing proxy | Free |

---

## Related Checklists

- ✅ [Functional Testing](./01-functional-testing.md) — core feature and input validation checks
- 🔌 [API Testing](./03-api-testing.md) — endpoint-level verification including auth headers
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found a security pattern in AI-generated code that isn't here? [Open a PR](../CONTRIBUTING.md) — this checklist grows through real-world findings.*
