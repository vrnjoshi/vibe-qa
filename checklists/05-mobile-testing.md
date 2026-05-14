# 📱 Mobile Testing Checklist for AI-Generated Code

> **Why this checklist exists:** AI tools generate mobile interfaces by default targeting desktop browsers. Mobile is an afterthought — and it shows. Touch interactions, platform-specific behaviors, device permissions, and offline scenarios are almost never handled correctly in vibe-coded apps without explicit testing. This checklist covers both mobile web and mobile app (React Native, Expo, Flutter) scenarios.
>
> Items marked ⚠️ are the patterns AI tools produce most frequently on mobile. Items marked 🔴 are failures that make the app unusable for mobile users, who typically represent 50–70% of real-world traffic.

---

## 1. Touch and Gesture Interactions

Mobile users interact with fingers, not a mouse. AI generates mouse-optimized interactions by default.

- [ ] 🔴 ⚠️ All interactive elements (buttons, links, form fields) are large enough to tap accurately — minimum **44×44px** touch target size
- [ ] ⚠️ Elements that are close together have sufficient spacing between them — tapping one does not accidentally trigger the neighbor
- [ ] Hover-dependent interactions do not exist as the only way to access functionality — mobile devices have no hover state
- [ ] ⚠️ Tooltips triggered only on hover are inaccessible on mobile — verify they have a tap-accessible alternative
- [ ] Dropdown menus open and close correctly with a tap — they do not require hover to open
- [ ] ⚠️ Swipe gestures (if implemented) work correctly and do not conflict with the browser's native scroll behavior
- [ ] Long-press interactions (if implemented) have a visible alternative — not every user discovers long-press intuitively
- [ ] Drag-and-drop functionality (if implemented) has a tap-based alternative on mobile
- [ ] Double-tap to zoom is disabled only where intentional — use `touch-action: manipulation` rather than disabling all zoom globally
- [ ] ⚠️ Custom scroll containers scroll smoothly with momentum on iOS — add `-webkit-overflow-scrolling: touch` or `overflow: auto` correctly

---

## 2. Viewport and Layout on Real Devices

DevTools emulation does not catch everything. Test on real devices or accurate simulators.

- [ ] 🔴 ⚠️ The page does not require horizontal scrolling on any standard mobile screen size (320px, 375px, 390px, 414px widths)
- [ ] ⚠️ `100vh` is not used for full-screen layouts on iOS Safari — the browser chrome (address bar) is not included in `100vh` on iOS, causing content to be cut off. Use `100dvh` or a JavaScript-based fix instead
- [ ] Fixed/sticky headers and footers do not overlap main content when the keyboard is open
- [ ] ⚠️ The virtual keyboard opening does not push critical UI elements off screen or cause the layout to break
- [ ] Content is not hidden behind a fixed bottom navigation bar on any screen size
- [ ] ⚠️ Safe area insets are respected on notched devices (iPhone X and later) — use `env(safe-area-inset-*)` CSS variables for content near the edges
- [ ] Landscape orientation renders correctly — test by rotating the device
- [ ] ⚠️ Modals and bottom sheets do not exceed the viewport height — they are scrollable internally if their content is long
- [ ] Text does not auto-resize unexpectedly — disable `text-size-adjust: 100%` to prevent iOS from adjusting font sizes

---

## 3. Mobile Browser Compatibility

- [ ] ⚠️ **Safari on iOS** — test all core flows here. Safari on iOS is the most common source of mobile-specific failures:
  - [ ] `position: sticky` works correctly inside scrollable containers
  - [ ] `overflow: hidden` on `<body>` correctly prevents scroll when a modal is open
  - [ ] `gap` property in Flexbox works (requires iOS 14.5+)
  - [ ] `aspect-ratio` CSS property works (requires iOS 15+)
  - [ ] Date inputs show the native iOS date picker, not a broken text field
  - [ ] Form inputs do not auto-zoom on focus — ensure `font-size` is at least 16px on input elements to prevent iOS auto-zoom
- [ ] ⚠️ **Chrome on Android** — test all core flows. Pay attention to:
  - [ ] The bottom navigation bar (gesture navigation) does not overlap content
  - [ ] Custom scroll behavior works correctly
  - [ ] Web share API (`navigator.share`) works if implemented
- [ ] **Samsung Internet** — if a significant portion of users are on Android, test in Samsung Internet browser which has some rendering differences
- [ ] ⚠️ CSS features used by the AI have sufficient mobile browser support — check [caniuse.com](https://caniuse.com) for any unfamiliar properties

---

## 4. Forms and Keyboard Behavior

Forms on mobile have unique failure modes that only appear with a physical or virtual keyboard.

- [ ] 🔴 ⚠️ The correct keyboard type appears for each input field:
  - [ ] `type="email"` shows email keyboard (@ symbol prominent)
  - [ ] `type="tel"` shows number pad
  - [ ] `type="number"` shows numeric keyboard
  - [ ] `type="url"` shows URL keyboard (/ and .com keys)
  - [ ] `type="search"` shows search keyboard (search/go button)
- [ ] ⚠️ `inputmode` attribute is set correctly where `type` alone is not sufficient (e.g., `inputmode="decimal"` for decimal numbers)
- [ ] The `autocomplete` attribute is set correctly on form fields to enable browser autofill — AI often omits this
- [ ] ⚠️ `autocapitalize="off"` is set on username, email, and password fields — iOS capitalizes the first letter by default
- [ ] `autocorrect="off"` and `spellcheck="false"` are set on fields where autocorrect would be harmful (usernames, codes, passwords)
- [ ] The form is still usable when the virtual keyboard is open — the active input field is visible and not hidden behind the keyboard
- [ ] ⚠️ Tapping "Next" on the keyboard moves focus to the next form field in the correct order
- [ ] Tapping "Done" or "Go" on the keyboard submits the form or closes the keyboard as appropriate
- [ ] ⚠️ After form submission, the keyboard closes automatically — an open keyboard with a blank form after submission is a common AI oversight

---

## 5. Performance on Mobile Hardware

Mobile devices have significantly less CPU and memory than desktop development machines. AI-generated code is often tested only on the developer's desktop.

- [ ] 🔴 ⚠️ Test on a mid-range or older device, not just the latest flagship — the AI's code may run smoothly on a MacBook but lag on a 3-year-old Android
- [ ] Animations run at 60fps on mobile — test by enabling "Show paint flashing" in Chrome DevTools mobile mode
- [ ] ⚠️ Excessive re-renders do not cause jank during scrolling — especially in lists and feeds with many items
- [ ] Images are appropriately sized for mobile — a 4000px image served to a 390px screen wastes bandwidth and memory
- [ ] ⚠️ The app loads on a 3G/4G connection in a reasonable time — use browser DevTools to throttle the network and measure load time
- [ ] JavaScript bundle size is reasonable — large bundles cause slow initial loads on mobile. AI-generated apps often include entire libraries when only small utilities are needed
- [ ] ⚠️ Memory usage does not grow over time during normal use — navigate through the app for 10+ minutes and check if it becomes sluggish (memory leak indicator)
- [ ] Long lists are paginated or virtualized — rendering 500+ items in a list at once causes visible lag on mobile

---

## 6. Device Permissions and Native Features

- [ ] ⚠️ Camera permission requests show a clear explanation of why the permission is needed before the browser prompt appears
- [ ] Microphone permission requests explain the use case before prompting
- [ ] Location permission requests explain the use case and gracefully handle the case where permission is denied
- [ ] ⚠️ Push notification permission is requested at an appropriate time — not immediately on first app load, which users almost always decline
- [ ] The app works correctly if the user denies any permission — a denied camera permission should show a helpful message, not crash
- [ ] ⚠️ AI-generated code that uses `navigator.geolocation`, `navigator.mediaDevices`, or `Notification` APIs handles the case where the browser does not support these APIs
- [ ] File upload via camera capture works on mobile — `<input type="file" accept="image/*" capture="environment">` opens the camera correctly
- [ ] ⚠️ Clipboard access (`navigator.clipboard`) works on mobile and handles the case where it is not supported or permission is denied

---

## 7. Offline and Poor Connectivity

- [ ] ⚠️ The app shows a meaningful message when the device has no internet connection — not a blank white screen or a raw browser error page
- [ ] A user mid-flow (e.g., filling out a form) who loses connectivity does not lose their entered data
- [ ] ⚠️ The app recovers gracefully when connectivity is restored — it does not require a full page reload to resume working
- [ ] If the app has a service worker or offline mode, test that it actually works by enabling airplane mode after initial load
- [ ] Images that fail to load (due to connectivity) show a graceful fallback — alt text or a placeholder image, not a broken image icon

---

## 8. Mobile-Specific UI Patterns

- [ ] ⚠️ The app does not use UI patterns that work on desktop but are broken on mobile:
  - [ ] Right-click context menus have a mobile-accessible alternative
  - [ ] Multi-select with Ctrl+Click has a touch-based alternative
  - [ ] Drag handles for resizable panels have a touch-based alternative
- [ ] Bottom navigation bars (if used) follow platform conventions — iOS uses tab bars at the bottom, Android uses either bottom nav or top app bars
- [ ] ⚠️ Pull-to-refresh (if implemented) does not conflict with the native browser pull-to-refresh behavior
- [ ] Back button behavior on Android is correct — tapping the system back button navigates to the previous screen, not exits the app unexpectedly
- [ ] ⚠️ Modals and sheets can be dismissed by tapping outside them or swiping down — users expect this on mobile
- [ ] The status bar color/style is set appropriately for the app's header color on mobile browsers that support it

---

## 9. Mobile App Specific (React Native / Expo / Flutter)

*Skip this section if you are testing a mobile web app, not a native or hybrid app.*

- [ ] 🔴 ⚠️ The app builds and runs without errors on both iOS and Android — AI-generated React Native code often works on one platform and fails on the other
- [ ] Platform-specific code (`Platform.OS === 'ios'`) is tested on both platforms — not just the developer's platform
- [ ] ⚠️ Native module dependencies are correctly linked — AI sometimes imports native modules without including the setup steps
- [ ] The app handles being backgrounded and foregrounded correctly — data is not lost when the user switches apps and returns
- [ ] Deep links open the correct screen in the app
- [ ] ⚠️ Keyboard avoiding behavior is implemented for forms — `KeyboardAvoidingView` or equivalent prevents the keyboard from covering input fields
- [ ] The app requests only the permissions it actually uses — AI often generates permission requests for features that were scaffolded but not fully implemented
- [ ] Push notifications work end-to-end: permission granted → notification received → tapping notification opens the correct screen
- [ ] ⚠️ The app does not crash when the device runs low on memory — test by running multiple other apps simultaneously
- [ ] Over-the-air update behavior (if using Expo Updates or similar) is tested — updates do not break the running app

---

## 10. Quick Mobile Smoke Test (Run After Every Deployment)

- [ ] Open the app on a real iOS device (or iOS Simulator) and navigate the main flows
- [ ] Open the app on a real Android device (or Android Emulator) and navigate the main flows
- [ ] Rotate both devices to landscape — check for broken layouts
- [ ] Submit the main form on mobile — confirm the keyboard behaves correctly
- [ ] Enable airplane mode — confirm the app shows a connectivity error message
- [ ] Check that no horizontal scroll exists on the main pages
- [ ] Confirm all tap targets are large enough to tap without accidentally hitting neighbors

---

## Related Checklists

- 🎨 [UI/UX Testing](./04-ui-ux-testing.md) — visual consistency, responsive design, and cross-browser checks
- ⚡ [Performance Testing](./06-performance-testing.md) — load times, memory, and efficiency
- 🚀 [Pre-Launch Final Checklist](./08-pre-launch-final.md) — the 20 must-pass checks before going live

---

*Found a mobile failure pattern in AI-generated code not listed here? [Open a PR](../CONTRIBUTING.md) — real-world additions make this more useful for everyone.*
