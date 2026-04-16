---
name: tauri-desktop-pitfalls
description: |
  Prevent common bugs and wasted debugging time when building Tauri desktop applications (especially Tauri v2).
  Covers: release pipeline failures (bundler opt-in), WKWebView drag-and-drop quirks, Tauri parameter naming (snake_case→camelCase auto-conversion),
  browser API limitations in webview (prompt/alert), and E2E test configuration anti-patterns.
  Use this skill whenever working on Tauri apps — especially during v1→v2 migration, setting up CI/CD release workflows,
  implementing drag-and-drop, writing Tauri invoke() calls, configuring Playwright E2E tests, or when encountering any
  "no artifacts", "missing required key", or "drop never fires" errors. If the project uses Tauri, load this skill.
---

# Tauri Desktop App Pitfalls

A field-guide of real, production-encountered bugs in Tauri applications (Tauri v1 and v2).

## Reference Index

| # | Pitfall | Reference |
|---|---------|-----------|
| 1 | Release pipeline: "No artifacts were found" (bundler opt-in) | [release-pipeline.md](references/release-pipeline.md) |
| 2 | `invoke()` params: snake_case → camelCase auto-conversion | [invoke-parameter-naming.md](references/invoke-parameter-naming.md) |
| 3 | Drag-and-drop: `drop` event never fires in WKWebView | [drag-drop-fire.md](references/drag-drop-fire.md) |
| 4 | Stale `sourceCollectionId` in drag payload | [stale-drag-payload.md](references/stale-drag-payload.md) |
| 5 | `window.prompt()` / `alert()` silently fail | [browser-apis.md](references/browser-apis.md) |
| 6 | E2E test over-engineering | [e2e-over-engineering.md](references/e2e-over-engineering.md) |

## Quick Reference: Tauri v1 vs v2 Differences

| Area | Tauri v1 | Tauri v2 |
|------|----------|----------|
| Bundler | Auto-runs when `bundle.active: true` | Opt-in via `--bundles` CLI arg |
| Parameter naming | Manual or convention-dependent | Auto snake_case → camelCase conversion |
| Config structure | `tauri.conf.json` with nested `tauri.bundle` | Restructured top-level keys |
| Webview default | Varies | `dragDropEnabled: true` (can interfere) |

## Diagnostic Checklist

When debugging a Tauri app issue, check in this order:

1. **Is this a Tauri v1→v2 migration?** Check for behavioral changes in bundler, config, parameter naming
2. **Is the event pipeline working?** Log `dragstart` → `dragover` → `drop` before assuming store logic is wrong
3. **Are we using browser-specific APIs?** `prompt`, `alert`, `confirm` don't work — need custom components
4. **Is state stale at gesture time?** Resolve authoritative ownership at execution, not at capture
5. **Is the test config minimal?** Start with one browser, no speculative conditionals
