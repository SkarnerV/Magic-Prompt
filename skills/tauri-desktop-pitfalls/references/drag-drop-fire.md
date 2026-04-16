# Pitfall: HTML5 Drag-and-Drop Not Firing in Tauri/WKWebView

**Source:** `openman/docs/sidebar-drag-drop-bug-fix.md`

## Symptom

Drag starts visually (UI reacts) but dropping does nothing — `drop` event never fires, `moveRequest` never called.

## Root Cause

Openman ships with Tauri using a **WKWebView**-backed webview. Three issues compound:

1. **Tauri intercepts drags by default** — When `dragDropEnabled` is `true` (the default), Tauri's native drag handling can prevent in-page HTML5 `drop` from executing normally.

2. **WebKit requires `text/plain` on dataTransfer** — Without it, internal HTML5 drags may never complete a drop. This is a known WebKit quirk.

3. **Drop targets need `preventDefault()` on `dragenter`** — Not just `dragover` but `dragenter` too, matching what several engines expect for valid drop targets.

### How We Diagnosed

Added lightweight instrumentation (NDJSON logs) around `dragstart` / `dragend` / `drop` handlers and `moveRequest` in the store. Logs showed many `dragstart` / `dragend` pairs but **no** `drop` events — proving the bug was platform-level, not store logic.

## Fix

### 1. Disable Tauri native drag handling

In `tauri.conf.json`:
```json
{
  "app": {
    "windows": [{
      "dragDropEnabled": false
    }]
  }
}
```

### 2. Set both data types on dataTransfer

```typescript
e.dataTransfer.setData("application/json", JSON.stringify(payload))
e.dataTransfer.setData("text/plain", "") // Required for WebKit
```

### 3. Call preventDefault on both dragenter AND dragover

```typescript
onDragEnter={(e) => { e.preventDefault(); /* highlight */ }}
onDragOver={(e) => { e.preventDefault(); /* highlight */ }}
onDrop={(e) => { e.preventDefault(); /* handle drop */ }}
```

## Diagnostic Tip

**Confirm the event pipeline first** (`dragstart` → `dragover`/`dragenter` → `drop`) before optimizing store logic. Add lightweight logging — if `drop` never appears in logs, the bug is platform-level, not application-level.
