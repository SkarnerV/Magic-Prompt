# Pitfall: E2E Test Over-Engineering

**Source:** `openman/docs/playwright-e2e-setup.md`

## Symptom

Playwright config has 6 browser projects, Tauri mode conditionals, PDF generation on failure, video recording, snapshot dirs — tests take forever and WebKit crashes on macOS with "bus error."

## Root Cause

Premature complexity. Cross-browser matrix multiplies test runs: 72 tests × 6 browsers = 432 executions. WebKit crashes are a known macOS issue.

The over-engineered config included:
- 7 browser project entries (desktop, chromium, firefox, webkit, mobile-chrome, mobile-safari)
- `TAURI_MODE` conditional logic for web server and base URL
- PDF generation on failure (Tauri-only)
- JSON reporter alongside HTML and list
- Snapshot directory configuration
- Video recording with specific dimensions
- Reuse context settings

All for a task that was just "fix failing tests."

## Fix

Start minimal:

```typescript
// playwright.config.ts — start here
export default defineConfig({
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  use: {
    baseURL: 'http://localhost:1420',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
})
```

### Practical Fixes for Test Stability

| Problem | Fix |
|---------|-----|
| 30-second waits on page load | `networkidle` → `domcontentloaded` |
| Errors when browser crashed | Add `page.isClosed()` checks in helpers and `afterEach` |
| `isVisible()` timeouts | Add explicit timeout: `await locator.isVisible({ timeout: 5000 })` |
| Cascading timeouts from nested `if` | Simplify flat assertions |

```typescript
// WRONG — nested ifs cause cascading timeouts
if (await page.locator('#btn').isVisible()) {
  if (await page.locator('#btn').isEnabled()) {
    await page.locator('#btn').click()
  }
}

// CORRECT — flat, explicit timeouts
await page.locator('#btn').click({ timeout: 5000 })
```

## Why Start Minimal

1. **YAGNI** — Cross-browser testing adds CI time and complexity. Add Firefox/WebKit only when a specific cross-browser bug is reported.
2. **Reduced maintenance** — Every browser project multiplies test runs and potential flakes.
3. **No premature abstraction** — `TAURI_MODE` conditionals, PDF generation, and video configs are speculative. Add them when you actually need them.
4. **Fewer points of failure** — WebKit's bus error on macOS is a known issue. By not including it, you avoid that entire class of problems.
5. **Faster feedback** — Single browser means faster CI runs and quicker iteration.

## Good Test Practices

- Install browsers first: `npx playwright install`
- Start with one browser, expand when needed
- Platform-specific issues should inform config (e.g., skip WebKit on macOS if it crashes)
- Target fixes to the actual problem — "fix failing tests" doesn't mean "build comprehensive E2E infrastructure"
- Test configuration should match the actual goal, not every conceivable eventual requirement
