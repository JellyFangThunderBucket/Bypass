# Refactor Analysis

## Performance analysis

- Added a shared `timers` module with `sleep()` and `debounce()` so repeated DOM cleanup can be coalesced instead of firing on every mutation.
- Replaced the one-off XHR-only listener with a shared `NetworkMonitor` that installs interception lazily only when a rule registers a listener.
- Added `FeatureManager` so optional heavy features are only enabled when their configuration flags are active.
- Added a selector cache option for `bp()` callers that need repeated stable queries without re-parsing selectors.

## Bugs found

- `Listener()` previously overwrote `XMLHttpRequest.prototype.open` every time it was called, which could stack wrappers and duplicate callbacks.
- Popup blocking replaced `window.open` with a Promise-returning function, which can be risky for sites expecting a `WindowProxy` or `null`.
- Several rules still use intervals for captcha or delayed button flows; these should be converted gradually to `waitForElement()` or shared observers.
- `NoPrompts()` recursively schedules itself with `setTimeout`, which can run forever on long-lived pages.
- Some anti-debug overrides make console properties non-configurable, which prevents restoration and can break developer tooling.

## Suggested improvements

- Move rule groups into generated domain-rule tables during a future build step while keeping this userscript self-contained for userscript managers.
- Convert high-frequency `CheckVisibility()` polling to a shared observer plus timeout fallback.
- Add a small test harness with mocked `document`, `unsafeWindow`, `GM_xmlhttpRequest`, and `MonkeyConfig` to validate core utilities.
- Use `AbortController` in every wait helper so feature disable operations can cancel pending work.

## Risky features

- Anti-debug protection modifies console methods, `Function.prototype.constructor`, `document.createElement`, `performance.now`, and unload handling.
- Timer acceleration changes site timing assumptions and may trigger anti-bot checks.
- Same-tab and popup features override navigation behavior globally.
- Prompt/notification blocking can remove legitimate cookie, privacy, or account dialogs on normal websites.

## Dead code candidates

- Duplicate `BypassedByBloggerPemula(/.*/)` feature bootstrapping should eventually be a single startup module.
- Domain rules that only log missing elements and have not matched in recent versions can be moved to an archived rules file.
- Repeated button-click sequences can be replaced with a single universal continue-button detector.

## Size reduction opportunities

- Compress domain regex groups into arrays and generate the final regex at startup.
- Share common actions such as `click after captcha`, `submit form`, `extract onclick URL`, and `wait then redirect`.
- Move CSS for notifications and dialogs into reusable template helpers.

## Compatibility notes

- Violentmonkey: expected to work best because of broad GM API support and `window.onurlchange` compatibility.
- Tampermonkey: should work, but `unsafeWindow` and `GM_xmlhttpRequest` behavior can differ by browser and sandbox mode.
- iOS Safari userscript managers: global API overrides, popups, downloads, clipboard, and GM APIs may be limited or unavailable; features should fail isolated through `FeatureManager` rather than crashing the script.
