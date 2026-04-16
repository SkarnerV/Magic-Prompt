# Pitfall: Tauri invoke() Parameter Naming (snake_case ↔ camelCase)

**Source:** `openman/docs/environment-creation-bug-fix.md`

## Symptom

```
invalid args 'workspaceId' for command 'create_environment': command create_environment missing required key workspaceId
```

## Root Cause

Tauri 2 **auto-converts** Rust `snake_case` parameter names to frontend `camelCase` names. If your Rust command defines `workspace_id: String`, the frontend must call it with `workspaceId`.

| Rust (snake_case) | Frontend (camelCase) |
|-------------------|----------------------|
| `workspace_id`    | `workspaceId`        |
| `collection_id`   | `collectionId`       |
| `environment_id`  | `environmentId`      |

### What Went Wrong

The frontend was using camelCase (`workspaceId`), but we incorrectly assumed Tauri required snake_case to match the Rust side. We changed all invoke calls to snake_case, which caused Tauri to reject the parameters because it expected the auto-converted camelCase names.

```typescript
// WRONG — this was our mistaken "fix"
invoke("create_environment", { workspace_id: "ws-1", name: "Production" })
```

## Fix

Always use camelCase on the frontend:
```typescript
// CORRECT — matches Tauri 2 convention
invoke("create_environment", { workspaceId: "ws-1", name: "Production" })
invoke("get_environments", { workspaceId: "ws-1" })
invoke("set_active_environment", { workspaceId: "ws-1", environmentId: "env-1" })
```

## Related: Modal for Creation

`window.prompt()` doesn't work in Tauri's webview environment. Replace with a proper modal component:
```typescript
// WRONG
const name = window.prompt("Enter environment name:")

// CORRECT
const [showModal, setShowModal] = useState(false)
// <CreateEnvironmentModal open={showModal} onSubmit={handleCreate} />
```

## How to Avoid

- The error message itself reveals what Tauri expects — `missing required key workspaceId` literally tells you the expected parameter name
- Rule of thumb: Rust side = `snake_case`, TypeScript side = `camelCase`. This is automatic, not manual.
