# Contributing to vibe-qa

Thank you for wanting to contribute. This repo grows through real-world experience — every addition comes from someone who actually found a failure in AI-generated code and took the time to document it. That is the only kind of contribution that makes this useful.

---

## What Makes a Good Contribution

The best contributions to this repo share one characteristic: **they come from something that actually happened.**

- You tested a vibe-coded app and found a bug that is not in the checklists → add it
- You found a repeating AI code failure pattern that is not documented → document it
- A checklist item was unclear or wrong → fix it
- A tool should be added or removed from the resources list → update it
- A checklist item is missing for a specific framework, platform, or scenario → add it

We do not accept:
- Items that are not specific to AI-generated code (generic QA advice belongs elsewhere)
- Theoretical failure patterns that have not been observed in practice
- Self-promotional links to paid tools or services

---

## How to Contribute

### Option 1: Open an Issue (easiest)
If you found a bug pattern or missing checklist item but do not want to write the full entry yourself, [open an issue](https://github.com/vrnjoshi/vibe-qa/issues/new) and describe it. Use the title format:

- `[bug pattern] <short description>` for new failure patterns
- `[checklist] <filename> — <short description>` for checklist additions
- `[fix] <short description>` for corrections to existing content

### Option 2: Open a Pull Request

1. **Fork** the repository
2. **Create a branch** with a descriptive name:
   ```bash
   git checkout -b add-pattern-missing-csrf-protection
   ```
3. **Make your changes** following the format below
4. **Submit a pull request** with a clear title and description of what you added and why

PRs are reviewed within a few days. All contributors are credited in the file they contributed to.

---

## Format for New Bug Patterns

When adding to `bug-patterns/known-ai-code-failures.md`, follow this structure exactly. Consistency makes the document scannable.

```markdown
## N. Pattern Name

**Severity:** 🔴 Critical / 🟡 High / 🟢 Medium

### What it looks like
Describe what the developer sees — the symptom, not the cause.
Include a code example showing the AI-generated pattern if possible.

```javascript
// ⚠️ AI-generated pattern — describe the problem
const badExample = 'show what the AI generates';
```

### Why AI generates it
Explain the root cause — why does AI produce this pattern specifically?
This is the most important section. It should make the reader think "yes, that makes sense."

### How to find it
Give specific, actionable steps to locate this pattern:
- Search terms to use in the codebase
- Manual test steps to trigger the failure
- Tools that detect it automatically

### How to fix it
Show the corrected code or configuration:

```javascript
// ✅ Correct pattern
const goodExample = 'show the fix';
```
```

---

## Format for New Checklist Items

When adding items to an existing checklist file, follow these rules:

- Write each item as an action, not a concept: **"Submit a form with all fields empty"** not **"Test empty form submission"**
- Add `⚠️` before items that AI tools specifically get wrong most often
- Add `🔴` before critical items (security vulnerabilities, data loss, core functionality broken)
- Keep items specific enough to be unambiguous — the reader should know exactly what to do
- Group related items together — do not add a security item in the middle of a UI section
- If you are adding more than 3 items to one section, consider whether a new section is warranted

**Example of a good checklist item:**
```markdown
- [ ] ⚠️ Submit the password reset form with an already-used token — confirm it is rejected with a clear error, not silently accepted
```

**Example of a checklist item that is too vague:**
```markdown
- [ ] Test password reset security
```

---

## Format for New Tools

When adding to `resources/tools.md`, use this table row format:

```markdown
| [Tool Name](https://tool-url.com) | One sentence describing what it does and why it is relevant to AI-generated code testing | Free / Free tier / Paid |
```

Only suggest tools that:
- You have personally used
- Are relevant to testing AI-generated applications specifically
- Have a free tier or are completely free

---

## Style Guide

- **Write for the reader who is stressed and in a hurry.** They have a bug, or they are about to ship. Be specific and direct.
- **Use plain language.** No jargon that requires a QA background to understand — this repo is used by developers, founders, and non-technical product owners.
- **Be specific about AI.** Generic QA advice belongs in other resources. Every item here should explain why it specifically matters for AI-generated code.
- **No fluff.** Do not add preamble, caveats, or marketing language. Get to the point.

---

## Recognition

Every contributor is recognized:

- Your GitHub username is credited in the commit history for every file you contribute to
- Significant contributors (3+ merged PRs) are listed in the README acknowledgements section
- If you contribute a new bug pattern that gets widely referenced, we will call that out specifically

---

## Questions

Open an issue with the label `question` and we will respond within a few days.

---

*This repo exists because real QA experience is not generated — it is accumulated. Thank you for sharing yours.*
