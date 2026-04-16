# Pitfall: Stale State in Drag Payloads (Store Consistency)

**Source:** `openman/docs/sidebar-drag-drop-bug-fix.md`

## Symptom

Cross-collection moves work, but reordering within the same collection sometimes fails with "Request not found in source collection."

## Root Cause

Drag code captures `sourceCollectionId` from the rendered row at `dragstart` time. By `drop` time, this can be stale if another operation moved the item or async updates changed reality.

`moveRequest` looked up the request **only** in the collection id passed from the sidebar ref (`sourceCollectionId` from dragstart). If the item had already been moved to another collection while the drag payload still carried the old id, the lookup failed.

### What Went Wrong

Drag code stored `sourceCollectionId` from the row that **rendered** the item. That was correct at render time but could be **stale** by drop time:
- If another drag operation moved the same request
- If async updates reconciled state between drag-start and drop
- The first argument from the UI remained for API compatibility but was no longer the sole source of truth

### How We Diagnosed

Logs showed `drop_request_zone` and `moveRequest` **did** run for same-collection reorder, but sometimes threw "Request not found in source collection." Other lines in the same session showed the same request id had already been moved to another collection while the drag payload still carried the **old** `sourceCollectionId`.

## Fix

Resolve the authoritative owner at drop time, not drag time:

```typescript
// Don't trust the declared sourceId from dragstart
function moveRequest(requestId: string, declaredSourceId: string, targetId: string) {
  // Find where the request actually lives NOW
  const resolvedSourceId = findOwningCollectionForRequest(requestId)
  // Use resolvedSourceId for the actual operation
  if (resolvedSourceId === targetId) {
    // Same-collection reorder
    updateCollection(resolvedSourceId, { /* reorder items */ })
  } else {
    // Cross-collection move
    removeRequest(resolvedSourceId, requestId)
    addRequest(targetId, requestId)
  }
}
```

The key helper — find the request across **all** collections:
```typescript
function findOwningCollectionForRequest(requestId: string): string | null {
  for (const collection of collections) {
    if (collection.items.some((item: any) => item.id === requestId)) {
      return collection.id
    }
  }
  return null
}
```

## Rule

Gestures should converge on authoritative state at execution time (where the item lives **now** in the store), not on props captured at `dragstart`.
