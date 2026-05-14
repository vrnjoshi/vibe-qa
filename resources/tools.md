# 🛠️ Recommended Tools for Testing AI-Generated Apps

> A curated list of tools referenced throughout the vibe-qa checklists. Every tool listed here is either free or has a free tier sufficient for individual developers and small teams. Organized by testing category.

---

## Automated Scanning

Tools that find issues without manual testing effort. Run these first — they catch the easy wins.

| Tool | What It Does | Cost |
|---|---|---|
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | Scans git history and codebase for leaked secrets, API keys, and credentials | Free |
| [Mozilla Observatory](https://observatory.mozilla.org) | Scans a live URL for HTTP security headers, TLS, and common misconfigurations | Free |
| [securityheaders.com](https://securityheaders.com) | Grades HTTP security headers with specific fix recommendations | Free |
| [Snyk](https://snyk.io) | Scans dependencies for known vulnerabilities (npm, pip, Maven, etc.) | Free tier |
| [axe DevTools](https://www.deque.com/axe/devtools/) | Browser extension that scans pages for WCAG accessibility violations | Free tier |
| [WAVE](https://wave.webaim.org) | Visual accessibility feedback overlay — good for understanding what screen readers encounter | Free |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse) | Built into Chrome DevTools — audits performance, accessibility, SEO, and best practices | Free |

---

## Security Testing

| Tool | What It Does | Cost |
|---|---|---|
| [OWASP ZAP](https://www.zaproxy.org) | Active security scanner — crawls your app and tests for common vulnerabilities | Free |
| [Burp Suite Community](https://portswigger.net/burp/communitydownload) | Industry-standard manual security testing proxy — intercept and modify requests | Free |
| [sqlmap](https://sqlmap.org) | Automated SQL injection detection and exploitation (use only on apps you own) | Free |
| [Nikto](https://github.com/sullo/nikto) | Web server scanner for outdated software, dangerous files, and misconfigurations | Free |
| [jwt.io](https://jwt.io) | Decode and inspect JWT tokens — useful for finding JWT algorithm issues | Free |

---

## API Testing

| Tool | What It Does | Cost |
|---|---|---|
| [Postman](https://www.postman.com) | The standard tool for manual API testing — build and save request collections | Free tier |
| [Hoppscotch](https://hoppscotch.io) | Lightweight browser-based alternative to Postman — no install required | Free |
| [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) | VS Code extension — test APIs directly from `.http` files in your editor | Free |
| [Insomnia](https://insomnia.rest) | API testing client with good GraphQL support | Free tier |
| [Dredd](https://dredd.org) | Validates your API implementation against OpenAPI/Swagger specs automatically | Free |
| [Schemathesis](https://github.com/schemathesis/schemathesis) | Automatically generates and runs test cases from your OpenAPI schema | Free |

---

## Performance Testing

| Tool | What It Does | Cost |
|---|---|---|
| [PageSpeed Insights](https://pagespeed.web.dev) | Google's Core Web Vitals analysis for any public URL | Free |
| [WebPageTest](https://www.webpagetest.org) | Detailed waterfall analysis, film strip view, and multi-location testing | Free |
| [k6](https://k6.io) | Developer-friendly load testing — write tests in JavaScript | Free |
| [Artillery](https://www.artillery.io) | Load testing for APIs and web apps — YAML-based test scenarios | Free tier |
| [Locust](https://locust.io) | Python-based load testing — good for complex user behavior simulation | Free |
| [Clinic.js](https://clinicjs.org) | Node.js performance profiling — identifies memory leaks, event loop delays | Free |
| [py-spy](https://github.com/benfred/py-spy) | Sampling profiler for Python applications | Free |

---

## Browser and UI Testing

| Tool | What It Does | Cost |
|---|---|---|
| [Playwright](https://playwright.dev) | Microsoft's end-to-end browser testing framework — Chrome, Firefox, Safari | Free |
| [Cypress](https://www.cypress.io) | End-to-end testing with excellent developer experience and visual debugging | Free tier |
| [Selenium](https://www.selenium.dev) | The original browser automation framework — wide language support | Free |
| [BrowserStack](https://www.browserstack.com) | Test on real devices and browsers in the cloud | Paid (free trial) |
| [Percy](https://percy.io) | Visual regression testing — catches unintended UI changes automatically | Free tier |

---

## Mobile Testing

| Tool | What It Does | Cost |
|---|---|---|
| [Android Studio Emulator](https://developer.android.com/studio) | Official Android emulator for testing Android apps | Free |
| [Xcode Simulator](https://developer.apple.com/xcode/) | iOS/iPadOS simulator for testing iOS apps (Mac only) | Free |
| [Appium](https://appium.io) | Mobile app automation for native, hybrid, and mobile web apps | Free |
| [Detox](https://wix.github.io/Detox/) | Gray-box end-to-end testing for React Native apps | Free |

---

## Accessibility Testing

| Tool | What It Does | Cost |
|---|---|---|
| [NVDA](https://www.nvaccess.org) | Free screen reader for Windows — the most commonly used screen reader globally | Free |
| [Accessibility Insights](https://accessibilityinsights.io) | Microsoft's guided accessibility testing tool — walks you through WCAG checks step by step | Free |
| [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) | Verify color contrast ratios against WCAG AA and AAA standards | Free |
| [Sa11y](https://sa11y.netlify.app) | In-page accessibility checker — add a script to any page for real-time feedback | Free |
| [Color Oracle](https://colororacle.org) | Simulates color blindness on your entire screen | Free |

---

## Monitoring (Post-Launch)

| Tool | What It Does | Cost |
|---|---|---|
| [Sentry](https://sentry.io) | Error monitoring and performance tracing — get notified when users hit errors | Free tier |
| [UptimeRobot](https://uptimerobot.com) | Monitors your URL every 5 minutes and alerts you when it goes down | Free tier |
| [Vercel Analytics](https://vercel.com/analytics) | Real user monitoring with Core Web Vitals — built into Vercel deployments | Free tier |
| [LogRocket](https://logrocket.com) | Session replay + error tracking — see exactly what users experienced when a bug occurred | Free tier |
| [Datadog](https://www.datadoghq.com) | Full-stack monitoring, APM, and alerting | Free trial |

---

## CI/CD Integration

| Tool | What It Does | Cost |
|---|---|---|
| [GitHub Actions](https://github.com/features/actions) | Automate QA checks on every push — see the [included workflow](../workflows/github-actions-qa.yml) | Free for public repos |
| [Codecov](https://codecov.io) | Track test coverage over time and enforce coverage minimums on PRs | Free for public repos |

---

## Quick-Start Recommendation

If you are testing your first vibe-coded app and don't know where to start, use these five free tools in order:

1. **[TruffleHog](https://github.com/trufflesecurity/trufflehog)** — scan for leaked secrets (5 minutes)
2. **[Mozilla Observatory](https://observatory.mozilla.org)** — scan your live URL for security issues (2 minutes)
3. **[Lighthouse](https://developer.chrome.com/docs/lighthouse)** — run in Chrome DevTools on your main pages (10 minutes)
4. **[axe DevTools](https://www.deque.com/axe/devtools/)** — install the browser extension and scan for accessibility issues (10 minutes)
5. **[Postman](https://www.postman.com)** — test your main API endpoints manually (30 minutes)

Total time: about 1 hour. These five checks will surface the most critical issues in any vibe-coded app.

---

*Missing a tool that should be here? [Open a PR](../CONTRIBUTING.md) and suggest it.*
