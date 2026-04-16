# Pitfall: Browser APIs Don't Work in Tauri Webview

**Source:** `openman/docs/environment-creation-bug-fix.md`

## Symptom

`window.prompt()` or `window.alert()` silently fails or does nothing in the Tauri app, but works fine in browser dev mode (`npm run dev`).

## Root Cause

Tauri's embedded webview doesn't support `window.prompt()`, `window.alert()`, or `window.confirm()`. These are browser chrome APIs that require the browser's own UI surface (the native dialog boxes browsers show). Tauri's webview (WebkitGTK on Linux, WKWebView on macOS, WebView2 on Windows) doesn't provide these surfaces.

## Fix

Use custom React modal components instead:

```typescript
// WRONG
const name = window.prompt("Enter environment name:")
if (name) {
  await createEnvironment(name)
}

// CORRECT — use a modal component
const [showModal, setShowModal] = useState(false)
// <CreateEnvironmentModal 
//   open={showModal} 
//   onClose={() => setShowModal(false)}
//   onSubmit={createEnvironment} 
// />
```

The modal component should:
- Provide a text input for user entry
- Show validation errors
- Handle Tauri error responses (which may be strings, not Error instances)

## Affected APIs

The following browser APIs may not work or behave differently in Tauri:

| API | Works in Tauri? | Alternative |
|-----|-----------------|-------------|
| `window.prompt()` | ❌ | Custom modal |
| `window.alert()` | ❌ | Custom toast/dialog |
| `window.confirm()` | ⚠️ Unreliable | Custom confirmation dialog |
| `window.open()` | ⚠️ Opens in external browser | `shell.open(url)` from Tauri |
| `navigator.clipboard` | ⚠️ May need permissions | `@tauri-apps/plugin-clipboard-manager` |

## How to Avoid

Any UI that relies on browser chrome (prompt, alert, confirm) needs a custom implementation for Tauri. Test in the actual Tauri app (`pnpm tauri dev`), not just the browser dev server.
