# 🎨 UI/UX Testing Checklist for AI-Generated Code

> **Why this checklist exists:** AI tools generate UIs that look good in the one browser, one screen size, and one happy-path state the developer tested in. They break everywhere else. This checklist covers the full range of visual, interaction, and user experience failures specific to vibe-coded interfaces — from responsive layout to loading states to cross-browser inconsistencies.
>
> Items marked ⚠️ are the patterns AI tools produce most frequently. Items marked 🔴 are failures that directly damage user trust or prevent core functionality.

---

## 1. Visual Consistency

AI generates UI components independently, which means they often look inconsistent when placed next to each other on a real page.

- [ ] ⚠️ Font sizes are consistent across the app — headings, body text, labels, and captions follow a defined scale, not arbitrary values chosen per component
- [ ] ⚠️ Colors are consistent — buttons, links, and alerts use the same color values throughout, not slightly different shades per section
- [ ] Spacing between elements is consistent — padding and margins follow a system, not random values
- [ ] ⚠️ Button styles are uniform — primary, secondary, and destructive buttons look the same everywhere they appear
- [ ] Icon sizes and styles are consistent throughout — AI often mixes icon libraries (e.g., Heroicons in one place, Material Icons in another)
- [ ] Border radius values are consistent — elements are not randomly rounded in some places and sharp-cornered in others
- [ ] ⚠️ Text alignment is consistent — left-aligned text should not randomly become centered in certain components
- [ ] Brand colors, if defined, are applied consistently and not overridden by default component library styles

---

## 2. Responsive Design

This is the most common UI failure in vibe-coded apps. AI tests at one screen size and never checks others.

### Mobile (320px–480px)
- [ ] 🔴 ⚠️ The layout does not overflow horizontally — horizontal scrolling on mobile is almost always a bug
- [ ] ⚠️ Text is readable without zooming — minimum 16px font size for body text on mobile
- [ ] All buttons and interactive elements are large enough to tap — minimum 44×44px touch target
- [ ] Navigation collapses correctly into a hamburger menu or mobile-appropriate pattern
- [ ] ⚠️ Tables do not overflow — they either scroll horizontally within a container or reformat for small screens
- [ ] Forms are usable on mobile — input fields are full-width, labels are visible, and the keyboard does not obscure the submit button
- [ ] Images scale correctly and do not overflow their containers
- [ ] ⚠️ Modals and dialogs fit within the mobile viewport — AI-generated modals are frequently too wide or too tall on small screens

### Tablet (768px–1024px)
- [ ] ⚠️ The layout transitions smoothly between mobile and desktop — check for awkward in-between states where the layout is neither mobile nor desktop
- [ ] Sidebars and navigation panels behave correctly at tablet widths
- [ ] Grid layouts reflow correctly — a 4-column desktop grid should become 2 columns on tablet, not stay at 4 columns with overflowing content

### Desktop (1280px+)
- [ ] Content does not stretch to full width on very wide screens — a max-width container keeps text readable
- [ ] ⚠️ The layout looks correct at 1920px wide — AI often designs for 1280px and content becomes uncomfortably spread at larger sizes

### Testing Method
- Use browser DevTools (F12 → Toggle Device Toolbar) to test all breakpoints
- Test on a real mobile device — DevTools emulation does not catch everything, especially touch interactions and font rendering

---

## 3. Loading and Empty States

AI generates the populated, success state of every UI. It almost never generates what the UI looks like before data loads or when there is no data.

- [ ] 🔴 ⚠️ Every data-fetching component has a loading state — no component should show blank space or broken layout while waiting for an API response
- [ ] ⚠️ Loading spinners or skeletons are shown immediately — users should see feedback within 300ms of triggering any action
- [ ] 🔴 ⚠️ Every list, table, or data view has an empty state — a message like "No items yet" instead of a completely blank section
- [ ] ⚠️ Empty state messages are helpful — they explain why there is no data and what the user can do (e.g., "You haven't created any projects yet. Create your first project →")
- [ ] The empty state for a search with no results is different from the empty state for a section with no data at all
- [ ] ⚠️ Loading states do not flash briefly for fast connections — use a minimum display time or skeleton screens to prevent jarring flashes
- [ ] Loading indicators stop when the request completes, whether it succeeds or fails — a spinner that spins forever after an error is a common AI-generated bug

---

## 4. Error States in the UI

Separate from API error handling — this covers how errors are actually presented to the user.

- [ ] 🔴 ⚠️ Failed API requests display a user-friendly error message in the UI — not a blank screen, not a console error, not a raw JSON response
- [ ] ⚠️ Form validation errors appear next to the relevant field, not just at the top or bottom of the form
- [ ] Validation error messages disappear when the user corrects the input — they do not persist after the problem is fixed
- [ ] ⚠️ The page does not crash (white screen of death) when any single component throws an error — error boundaries or equivalent patterns should contain failures
- [ ] A network failure during a form submission does not silently lose the user's input — the form data is preserved so they can retry
- [ ] ⚠️ Error messages use plain language — "Unable to save your changes. Please try again." not "Error 422: Unprocessable Entity"
- [ ] Error states have a clear recovery action — a retry button, a link to contact support, or instructions for what to do next

---

## 5. Interactive Elements

- [ ] ⚠️ Every button has a visible hover state — no button looks identical hovered and unhovered
- [ ] Every button has a visible focus state (outline or highlight) for keyboard navigation — AI often removes focus outlines with `outline: none` without providing an alternative
- [ ] ⚠️ Buttons that trigger async actions (save, submit, delete) show a loading/disabled state while the action is in progress — prevents double submissions
- [ ] Destructive actions (delete, cancel, remove) have a confirmation step — a single click should not irreversibly delete data
- [ ] ⚠️ Disabled buttons look visually distinct from enabled buttons — reduced opacity or different color, not just a changed cursor
- [ ] Clickable areas match the visual element — the entire button area is clickable, not just the text label
- [ ] ⚠️ Dropdown menus close when the user clicks outside them — AI-generated dropdowns frequently stay open
- [ ] Tooltips appear on hover and disappear when the cursor moves away — they do not get stuck open

---

## 6. Forms

Forms are one of the most failure-prone areas in AI-generated UI. Test every form thoroughly.

- [ ] ⚠️ Required fields are clearly marked — an asterisk (*) or "Required" label, with a legend explaining the convention
- [ ] Field labels are always visible — do not rely on placeholder text as the only label. Placeholder text disappears when the user starts typing
- [ ] ⚠️ Tab order through form fields is logical and follows the visual layout — AI-generated forms frequently have broken tab order
- [ ] The Enter/Return key submits the form when focus is inside a text field (single-line inputs)
- [ ] ⚠️ Multi-line textareas do not submit on Enter — pressing Enter in a textarea should add a new line, not submit the form
- [ ] Password fields have a show/hide toggle
- [ ] ⚠️ Auto-fill works correctly — browser-saved passwords and addresses populate the right fields
- [ ] Date pickers work on mobile — native date inputs on mobile open the platform's date picker, which AI-generated custom date pickers often block
- [ ] Long forms are broken into logical sections or steps — a single page with 20+ fields is poor UX and often an AI default
- [ ] Form state is preserved if the user accidentally navigates away and returns (where appropriate)

---

## 7. Navigation and Routing

- [ ] ⚠️ The active/current page is visually indicated in the navigation — the user always knows where they are
- [ ] Browser back and forward buttons work correctly throughout the app
- [ ] ⚠️ Navigating directly to any URL (deep linking) renders the correct page — no "Page not found" errors for valid routes
- [ ] The page title (`<title>` tag) updates when navigating between pages — every page should have a unique, descriptive title
- [ ] ⚠️ Breadcrumbs or back links, if present, navigate to the correct previous location — not always to the homepage
- [ ] Anchor links (`#section`) scroll to the correct position and account for any fixed header height
- [ ] ⚠️ External links open in a new tab (`target="_blank"`) with `rel="noopener noreferrer"` for security

---

## 8. Cross-Browser Compatibility

AI generates code that works in the developer's browser. Test in at least three browsers before launch.

- [ ] ⚠️ Chrome (or Chromium-based: Edge, Brave) — most AI tools test here by default
- [ ] ⚠️ Safari — the most common source of cross-browser failures. CSS features like `gap` in flexbox, `position: sticky`, and certain input styles behave differently
- [ ] Firefox — test for layout and font rendering differences
- [ ] ⚠️ Safari on iOS — especially important for mobile layouts. Viewport height (`100vh`) behaves differently on iOS Safari due to the browser chrome
- [ ] Check for console errors in each browser — cross-browser JavaScript errors are often silent in one browser and visible in another

**Common AI-generated cross-browser failures to check specifically:**
- [ ] CSS Grid and Flexbox layouts render consistently across browsers
- [ ] Custom fonts load correctly in all browsers
- [ ] Date input types render correctly or fall back gracefully in Safari
- [ ] CSS variables (custom properties) work in all target browsers
- [ ] ⚠️ `backdrop-filter` (blur effects) — not supported in all browsers without a prefix

---

## 9. Dark Mode (if supported)

- [ ] All text is readable in dark mode — sufficient contrast between text and background
- [ ] ⚠️ No hardcoded colors remain — elements that use fixed `color: #000` or `background: #fff` will be unreadable in dark mode
- [ ] Images with white backgrounds look correct in dark mode — consider using transparent PNGs or CSS filters
- [ ] ⚠️ Form inputs, dropdowns, and modals apply dark mode styles correctly — AI often misses these
- [ ] The dark/light mode toggle persists across page refreshes and sessions

---

## 10. Performance Perception (How Fast It Feels)

Actual performance is covered in the Performance Testing checklist. This section covers how the UI *feels* to users regardless of actual speed.

- [ ] ⚠️ There is no layout shift after the page loads — content does not jump around as images, fonts, or async data loads in
- [ ] Images have explicit `width` and `height` attributes or CSS dimensions to prevent layout shift
- [ ] ⚠️ Fonts load without a flash of unstyled text — use `font-display: swap` or preload critical fonts
- [ ] Page transitions feel smooth — no jarring full-page flashes when navigating
- [ ] ⚠️ Long lists and tables are paginated or virtualized — rendering 1,000+ rows causes visible lag

---

## 11. Copy and Content

AI generates UI copy that is functional but often generic, inconsistent, or contextually wrong.

- [ ] ⚠️ No placeholder text remains in the UI — "Lorem ipsum", "Sample text", "TODO: add copy", "Title goes here"
- [ ] Action button labels are specific — "Save Changes" not "Submit", "Delete Account" not "Confirm"
- [ ] ⚠️ Success and confirmation messages are friendly and specific — "Your profile has been updated" not "Success"
- [ ] Error messages are written in the user's language — no technical jargon, no HTTP status codes
- [ ] All UI copy is reviewed for spelling and grammar — AI generates typos less often than humans but not never
- [ ] ⚠️ Capitalization is consistent — sentence case vs. title case should follow one convention throughout the app
- [ ] Empty state copy is encouraging and actionable, not just "No data"

---

## 12. Quick UI Smoke Test (Run After Every Deployment)

- [ ] Open the app on a real mobile device and scroll through the main pages — check for overflow or broken layout
- [ ] Open the app in Safari if you primarily developed in Chrome
- [ ] Click through the main navigation — confirm every link resolves and the active state highlights correctly
- [ ] Trigger one loading state — confirm a spinner or skeleton appears
- [ ] Trigger one error state — confirm an error message appears (try submitting an empty form)
- [ ] Check the browser console for red errors on the main pages
- [ ] Resize the browser window from mobile width to desktop width — watch for layout breakpoints

---

## Related Checklists

- ✅ [Functional Testing](./01-functional-testing.md) — end-to-end feature and flow verification
- ♿ [Accessibility Testing](./07-accessibility-testing.md) — WCAG compliance and keyboard navigation
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found a UI/UX failure pattern in AI-generated code not listed here? [Open a PR](../CONTRIBUTING.md) — real-world additions make this more useful for everyone.*
