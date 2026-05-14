# ♿ Accessibility Testing Checklist for AI-Generated Code

> **Why this checklist exists:** AI tools generate visually functional interfaces that are often completely unusable for people with disabilities. Keyboard navigation is broken, screen readers encounter unlabeled elements, color contrast fails, and focus management is missing entirely. Beyond being the right thing to do, accessibility failures are a legal liability in many countries (ADA in the US, EN 301 549 in the EU, AODA in Canada). This checklist covers the most common WCAG 2.1 AA failures in AI-generated code — the standard required for legal compliance in most jurisdictions.
>
> Items marked ⚠️ are the WCAG failures AI tools produce most frequently. Items marked 🔴 are failures that make the app completely unusable for users with specific disabilities.

---

## 1. Automated Scanning (Start Here)

Run automated tools first — they catch 30–40% of accessibility issues with minimal effort. Automated tools are a starting point, not a finish line. Manual testing is still required.

- [ ] Run [axe DevTools](https://www.deque.com/axe/devtools/) browser extension on every page — fix all "Critical" and "Serious" violations before moving to manual testing
- [ ] Run [WAVE](https://wave.webaim.org) on every page — pay attention to errors (red icons) and contrast errors (yellow icons)
- [ ] Run [Lighthouse Accessibility audit](https://developer.chrome.com/docs/lighthouse) (built into Chrome DevTools) — aim for a score of 90+ before launch
- [ ] ⚠️ Run automated tools in all major states: logged out, logged in, with a modal open, with an error message showing — AI-generated dynamic content often introduces new violations when state changes
- [ ] Address all automated violations before proceeding to manual testing — do not use "we'll fix it later" as a reason to skip this step

---

## 2. Keyboard Navigation

Keyboard accessibility is the foundation of all other assistive technology support. If keyboard navigation is broken, screen readers, switch controls, and voice navigation are also broken.

- [ ] 🔴 ⚠️ Every interactive element on the page is reachable using only the Tab key — no mouse required
- [ ] 🔴 ⚠️ Tab order follows the logical visual reading order (left to right, top to bottom) — AI-generated layouts frequently have tab order that jumps around the page in unexpected sequences
- [ ] ⚠️ AI-generated code that uses `tabindex` values greater than 0 (e.g., `tabindex="2"`) is corrected — positive tabindex values disrupt natural tab order and are almost always a mistake
- [ ] Every interactive element can be activated using the keyboard:
  - [ ] Buttons activate with Enter and Space keys
  - [ ] Links activate with Enter key
  - [ ] Dropdowns open and navigate with arrow keys, close with Escape
  - [ ] Modals close with the Escape key
  - [ ] Checkboxes and radio buttons toggle with Space key
- [ ] ⚠️ `outline: none` or `outline: 0` is not applied to focused elements without providing an equivalent visible focus indicator — this is one of the most common AI-generated accessibility failures. Every focused element must have a clearly visible focus ring
- [ ] The focus indicator is clearly visible — a 2px solid outline in a contrasting color meets WCAG 2.1 AA minimum
- [ ] ⚠️ Keyboard focus is never trapped in a part of the page (except intentionally in modals) — pressing Tab repeatedly should cycle through all interactive elements without getting stuck
- [ ] Custom interactive components (custom dropdowns, sliders, date pickers) support full keyboard interaction — AI-generated custom components almost never implement keyboard support

---

## 3. Focus Management

Focus management is how the app directs attention when content changes. AI almost never implements this.

- [ ] ⚠️ When a modal or dialog opens, focus moves to the first focusable element inside it — not left on the element that triggered it
- [ ] 🔴 ⚠️ Focus is trapped inside open modals — Tab key should cycle only through modal content, not the page behind it
- [ ] When a modal closes, focus returns to the element that triggered it (e.g., the button that opened the modal)
- [ ] ⚠️ When navigating to a new page in a single-page app (SPA), focus moves to the top of the page or the main content heading — AI-generated SPAs leave focus wherever it was, which is confusing for screen reader users
- [ ] After submitting a form, focus moves to the confirmation message or error summary so screen reader users know what happened
- [ ] ⚠️ When dynamic content appears (inline error, success toast, new list items), focus management or a live region announces the change to screen reader users

---

## 4. Images and Non-Text Content

- [ ] 🔴 ⚠️ Every `<img>` element has an `alt` attribute — the absence of `alt` entirely is a WCAG Level A failure. AI sometimes generates images without alt attributes
- [ ] ⚠️ Alt text describes the content and purpose of the image, not just its appearance — "Profile photo of Jane Smith" not "photo" or "image1.jpg"
- [ ] Decorative images that convey no information have empty alt text (`alt=""`) so screen readers skip them — AI often adds redundant alt text like "decorative swirl" to purely decorative elements
- [ ] ⚠️ Icons that convey meaning (warning icon, close button icon, success icon) have accessible labels — either via `aria-label`, visually hidden text, or a visible text label
- [ ] Icon-only buttons (a trash can icon with no text) have an `aria-label` that describes the action — "Delete item" not "trash" or nothing
- [ ] Infographics and complex images have a text alternative that conveys the same information
- [ ] Video content has captions — auto-generated captions must be reviewed and corrected for accuracy
- [ ] Audio-only content has a transcript

---

## 5. Color and Contrast

- [ ] 🔴 ⚠️ Text contrast ratio meets WCAG AA minimums — test using the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/):
  - Normal text (under 18pt): minimum **4.5:1** contrast ratio
  - Large text (18pt+ or 14pt+ bold): minimum **3:1** contrast ratio
  - UI components (input borders, focus indicators): minimum **3:1** contrast ratio
- [ ] ⚠️ AI-generated color palettes frequently fail contrast — especially light gray text on white backgrounds, which is a common "modern" design choice that fails WCAG
- [ ] Information is not conveyed by color alone — a red error message must also have text or an icon indicating it is an error, not just the color red
- [ ] Form validation errors are not indicated only with a red border — add an error message and/or error icon
- [ ] ⚠️ Links are distinguishable from surrounding text by more than color alone — they should be underlined or have another non-color visual differentiator
- [ ] Dark mode meets the same contrast requirements as light mode — test both if dark mode is supported

---

## 6. Forms and Inputs

Forms are one of the highest-impact areas for accessibility. Inaccessible forms prevent users from completing critical tasks.

- [ ] 🔴 ⚠️ Every form input has a visible, programmatically associated label — use `<label for="inputId">` or wrap the input in the `<label>` element. Placeholder text is not a substitute for a label
- [ ] ⚠️ `aria-label` or `aria-labelledby` is used when a visible label is not possible — never leave an input with no label at all
- [ ] Required fields are indicated both visually and programmatically — use `required` attribute on the input and indicate it visually with an asterisk or "Required" text
- [ ] ⚠️ Error messages are programmatically associated with their input using `aria-describedby` — screen readers will not read error messages that are just visually near the input without this connection
- [ ] Error messages are specific — "Please enter a valid email address" not just "Invalid input"
- [ ] ⚠️ `autocomplete` attributes are set on common fields (name, email, password, address) — required for WCAG 1.3.5 (AA) and dramatically improves usability for users with cognitive disabilities
- [ ] Form submission errors are announced to screen readers — use an `aria-live` region or move focus to an error summary
- [ ] Input purpose is clear without relying on placeholder text alone — placeholders disappear when the user types, leaving them without context

---

## 7. Semantic HTML Structure

AI generates visually correct layouts using `div` and `span` for everything. Semantic HTML is critical for screen reader users.

- [ ] ⚠️ The page has exactly one `<h1>` — the main page heading. AI often generates multiple `<h1>` elements or skips them entirely
- [ ] Heading levels are hierarchical — `<h1>` → `<h2>` → `<h3>` with no levels skipped (e.g., no jumping from `<h2>` to `<h4>`)
- [ ] ⚠️ Navigation landmarks are present: `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>` — AI often wraps everything in `<div>` without landmarks. Screen reader users navigate by landmarks
- [ ] There is exactly one `<main>` element per page
- [ ] ⚠️ Buttons that trigger actions use `<button>` elements — not `<div>` or `<span>` styled to look like buttons. Divs are not keyboard accessible or announced as buttons by screen readers
- [ ] Links that navigate to a URL use `<a href>` elements — not `<div onClick>` styled as links
- [ ] ⚠️ Lists use `<ul>` or `<ol>` with `<li>` items — not `<div>` elements with manual spacing. Screen readers announce list count and item position
- [ ] Tables use proper table markup: `<table>`, `<thead>`, `<tbody>`, `<th scope>` — not just visually aligned `<div>` grids

---

## 8. Dynamic Content and ARIA

- [ ] ⚠️ `aria-live="polite"` is used on regions that update dynamically (status messages, search results updating, notifications) — without this, screen readers are not informed of changes
- [ ] `aria-live="assertive"` is only used for time-sensitive alerts — overuse interrupts the screen reader user constantly
- [ ] ⚠️ ARIA roles, states, and properties are used correctly — incorrect ARIA is worse than no ARIA. If unsure, use semantic HTML instead. Common AI mistakes:
  - [ ] `role="button"` on a `<div>` without also adding `tabindex="0"` and keyboard event handlers
  - [ ] `aria-expanded` not toggled when an accordion or dropdown opens/closes
  - [ ] `aria-hidden="true"` on elements that are visible on screen
  - [ ] `aria-label` that duplicates visible text (redundant but not harmful) vs. contradicts visible text (harmful)
- [ ] Loading states announce to screen readers — use `aria-busy="true"` on loading regions or an `aria-live` region that announces when loading completes
- [ ] ⚠️ Tooltips and popovers are accessible — they appear on keyboard focus as well as mouse hover, and can be dismissed with Escape key

---

## 9. Screen Reader Testing

Automated tools and keyboard testing find most issues. Screen reader testing confirms the real experience.

**Recommended free screen readers:**
- **NVDA + Chrome** (Windows) — most common screen reader on Windows, free
- **VoiceOver + Safari** (Mac/iOS) — built into Apple devices, no download needed (Command+F5 to enable on Mac)
- **TalkBack** (Android) — built into Android devices

**What to test with a screen reader:**
- [ ] ⚠️ Navigate the page using only the screen reader (no mouse) — confirm all content is announced in a logical order
- [ ] All images are announced with meaningful descriptions
- [ ] All form labels are read before their input fields
- [ ] Error messages are announced when they appear
- [ ] ⚠️ Modal dialogs are announced as dialogs and focus is contained within them
- [ ] Buttons and links announce their purpose clearly — "Delete" with context, not just "button"
- [ ] ⚠️ Dynamic content changes (loading complete, item added, error appeared) are announced without requiring navigation

---

## 10. Motion and Animation

- [ ] ⚠️ Animations respect `prefers-reduced-motion` media query — users who have indicated they prefer reduced motion (common for users with vestibular disorders) should see a reduced or eliminated animation. AI never implements this
  ```css
  @media (prefers-reduced-motion: reduce) {
    * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
  }
  ```
- [ ] No content flashes more than 3 times per second — rapidly flashing content can trigger seizures
- [ ] Auto-playing video or animation has a pause control
- [ ] Parallax scrolling effects can be disabled or are suppressed for `prefers-reduced-motion` users

---

## 11. Quick Accessibility Smoke Test (Minimum Before Launch)

- [ ] Tab through the entire main user flow using only the keyboard — confirm every step is reachable and operable
- [ ] Run axe DevTools on the homepage, login page, and main feature page — fix all Critical violations
- [ ] Check color contrast on body text, headings, and form labels using WebAIM Contrast Checker
- [ ] Verify every form input has a visible label
- [ ] Enable VoiceOver (Mac) or NVDA (Windows) and navigate the homepage — confirm the page makes sense when heard
- [ ] Check that every image has alt text by right-clicking → Inspect and searching for `<img` tags without `alt`

---

## Recommended Accessibility Tools

| Tool | Purpose | Cost |
|---|---|---|
| [axe DevTools](https://www.deque.com/axe/devtools/) | Automated WCAG scanning in browser | Free tier |
| [WAVE](https://wave.webaim.org) | Visual accessibility feedback overlay | Free |
| [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) | Color contrast verification | Free |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse) | Built-in Chrome accessibility audit | Free |
| [NVDA](https://www.nvaccess.org) | Screen reader for Windows testing | Free |
| [Accessibility Insights](https://accessibilityinsights.io) | Microsoft's guided accessibility testing | Free |
| [Sa11y](https://sa11y.netlify.app) | In-page accessibility checker | Free |

---

## Related Checklists

- 🎨 [UI/UX Testing](./04-ui-ux-testing.md) — visual consistency, responsive design, and browser compatibility
- 📱 [Mobile Testing](./05-mobile-testing.md) — mobile-specific touch and interaction testing
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found an accessibility failure pattern in AI-generated code not listed here? [Open a PR](../CONTRIBUTING.md) — real-world additions make this more useful for everyone.*
