# 🧪 vibe-qa

### The QA Handbook for AI-Generated Code

**Checklists, bug patterns, test templates, and testing workflows for vibe-coded apps**

---

![Stars](https://img.shields.io/github/stars/vrnjoshi/vibe-qa?style=social)
![Last Commit](https://img.shields.io/github/last-commit/vrnjoshi/vibe-qa)
![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

---

## The Problem Nobody Is Talking About

**46% of all new code committed to GitHub today is AI-generated.**

Tools like Cursor, Claude Code, GitHub Copilot, Lovable, Replit, and Bolt have made software development faster than ever. But the testing gap is the most underrated problem in software right now:

- 🔴 **36%** of vibe coders admit to skipping QA entirely — relying on "reprompting instead of debugging"
- 🔴 AI co-authored code has **2.74x more security vulnerabilities** than human-written code
- 🔴 AI-generated code contains **75% more misconfigurations**
- 🔴 **48%** of AI-generated code has at least one security flaw
- 🔴 Over **1 in 3** Stack Overflow visits now come from developers debugging AI-generated code

The tools for *generating* code with AI have matured. The tools for *testing* it have not caught up. This repo exists to close that gap.

---

## Who This Is For

- ✅ **Developers** using Cursor, Claude Code, GitHub Copilot, or any AI coding tool
- ✅ **Solo founders and indie hackers** shipping products built with Lovable, Bolt, or Replit
- ✅ **QA engineers** onboarding onto teams that have adopted vibe coding workflows
- ✅ **Engineering leads** establishing quality standards for AI-assisted development
- ✅ **Non-technical founders** who vibe-coded an MVP and want to know what to check before launch

> **Built by a QA engineer.** This repo is maintained by someone who has spent 10+ years testing security panels, web applications, and mobile apps — including AI-assisted products. Everything here comes from real-world testing experience, not theory.

---

## ⚡ Quick Start

**Just shipped something with an AI tool and want to know if it's safe to go live?**

👉 Start with the **[Pre-Launch Final Checklist](./checklists/08-pre-launch-final.md)** — 20 critical checks, 30 minutes, before you ship.

**Building something new and want a full QA strategy from the start?**

👉 Read the **[Functional Testing Checklist](./checklists/01-functional-testing.md)** first, then follow the table of contents below in order.

**Debugging a strange bug in AI-generated code?**

👉 Check **[Known AI Code Failure Patterns](./bug-patterns/known-ai-code-failures.md)** — a growing catalog of the specific, repeating bugs that AI tools produce.

---

## 📋 Table of Contents

### Checklists
| # | Checklist | Description |
|---|---|---|
| 01 | [Functional Testing](./checklists/01-functional-testing.md) | Core features, happy paths, edge cases, and AI-specific logic failures |
| 02 | [Security Testing](./checklists/02-security-testing.md) | Auth, input validation, API keys, CORS, SQL injection, and more |
| 03 | [API Testing](./checklists/03-api-testing.md) | Endpoints, error handling, rate limiting, and hallucinated routes |
| 04 | [UI/UX Testing](./checklists/04-ui-ux-testing.md) | Visual consistency, responsive design, loading and error states |
| 05 | [Mobile Testing](./checklists/05-mobile-testing.md) | iOS and Android specific issues in AI-generated mobile apps |
| 06 | [Performance Testing](./checklists/06-performance-testing.md) | Load times, memory leaks, and inefficient AI-generated queries |
| 07 | [Accessibility Testing](./checklists/07-accessibility-testing.md) | WCAG compliance gaps commonly missed by AI code generators |
| 08 | [Pre-Launch Final Checklist](./checklists/08-pre-launch-final.md) | ⭐ The 20 must-pass checks before you ship anything |

### Knowledge Base
| Resource | Description |
|---|---|
| [Known AI Code Failure Patterns](./bug-patterns/known-ai-code-failures.md) | A catalog of documented, repeating bugs in AI-generated code with how to find and fix each one |
| [Test Plan Template](./test-templates/test-plan-template.md) | A lightweight test plan template designed for fast-moving vibe coding projects |
| [Bug Report Template](./test-templates/bug-report-template.md) | Structured bug report format optimized for AI-generated code issues |
| [GitHub Actions QA Workflow](./workflows/github-actions-qa.yml) | A ready-to-use CI/CD workflow that runs automated quality checks on every push |
| [Recommended Tools](./resources/tools.md) | Curated list of free and paid tools for testing AI-generated apps |

---

## Why AI-Generated Code Needs Its Own QA Approach

Traditional QA assumes a human wrote the code and understands what it does. Vibe coding breaks that assumption in four specific ways:

**1. AI optimizes for the happy path.**
When you prompt an AI to build a feature, it generates code that works for your exact prompt. It does not automatically consider what happens when a user enters unexpected data, loses their connection mid-flow, or hits the feature in a sequence the prompt didn't describe.

**2. AI places security logic in the wrong layer.**
One of the most documented failure patterns is AI generating authentication and authorization checks on the frontend (client-side) instead of the backend. This looks correct in a demo but is trivially bypassed in production.

**3. AI does not know your system.**
When AI generates a delete function, it deletes the record you asked about. It does not know about the linked records in three other tables, the uploaded files in S3, or the entry in your analytics system. Data integrity issues are silent until a user notices their data is corrupted.

**4. AI is confidently wrong.**
Unlike a human developer who might add a comment saying "I'm not sure about this," AI-generated code looks clean, complete, and correct even when it has a critical flaw. This makes bugs harder to spot in code review.

This repo addresses all four failure modes with actionable, checklist-driven QA that does not require you to be a testing expert.

---

## How to Use This Repo

### For individual projects
Copy the relevant checklists into your project's `/docs` folder or a Notion/Linear page. Work through them before every major release.

### For teams
Add the [GitHub Actions QA Workflow](./workflows/github-actions-qa.yml) to your repo for automated checks on every PR. Use the [Bug Report Template](./test-templates/bug-report-template.md) to standardize how your team reports issues.

### For QA engineers
Use the [Known AI Code Failure Patterns](./bug-patterns/known-ai-code-failures.md) as a reference when onboarding onto a codebase you did not write. These are the first places to look.

---

## Contributing

This repo grows through real-world experience. If you have:

- Found a bug pattern that repeats in AI-generated code
- Tested a vibe-coded app and discovered something not covered here
- A checklist item from your own QA practice that belongs here

**Please open a PR or Issue.** See [CONTRIBUTING.md](./CONTRIBUTING.md) for the simple format.

Every contribution is credited. This is a community knowledge base, not a solo project.

---

## Stats and Background

This repo was created in May 2026 by a Senior Quality Engineer with 10+ years of experience testing security hardware, web applications, and mobile apps — including products built with AI assistance using Claude (via Azure AI Foundry) and GitHub Copilot.

The checklists, bug patterns, and workflows here reflect real testing work, not generated content.

---

## License

MIT — free to use, adapt, and share. Attribution appreciated but not required.

---

*If this helped you ship something safer, star the repo. It helps others find it.*
