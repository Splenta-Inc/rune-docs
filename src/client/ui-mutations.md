# UI Mutations

UI mutations are JSON payloads sent through `core_dispatch_mutation` to update the view tree.

## Mutation Types

| Type | Description | Status |
|------|-------------|--------|
| `UpdateData` | Update node's data bindings | ✅ |
| `UpdateView` | Update node's visual properties | ✅ |
| `ReplaceText` | Replace text content | ✅ |
| `RemoveNode` | Remove node from tree | ✅ |
| `Navigate` | Change route/view | ✅ |
| `NavigateBack` | Go to previous route | ✅ |
| `ShowOverlay` | Display overlay/modal | ✅ |
| `HideOverlay` | Dismiss overlay | ✅ |

---

## UpdateData

Update a node's data bindings. Used for reactive state changes.

```json
{
  "type": "UpdateData",
  "node_id": "user-name",
  "updates": {
    "text": "Alice",
    "count": 42
  }
}
```

**TypeScript helper:**
```typescript
rune.update_data("user-name", { text: "Alice", count: 42 });
```

**Effect:** The node with `id="user-name"` will have its `data` object updated. Any bindings using `data-bind` will re-render.

---

## UpdateView

Update a node's visual properties (style, class, attributes).

```json
{
  "type": "UpdateView",
  "node_id": "button",
  "props": {
    "class": "btn-primary",
    "disabled": true
  }
}
```

**TypeScript helper:**
```typescript
rune.update_view("button", { class: "btn-primary", disabled: true });
```

---

## ReplaceText

Replace text content of a node directly.

```json
{
  "type": "ReplaceText",
  "target": "message",
  "text": "Hello, World!"
}
```

**TypeScript helper:**
```typescript
rune.replace_text("message", "Hello, World!");
```

---

## RemoveNode

Remove a node from the view tree.

```json
{
  "type": "RemoveNode",
  "node_id": "temp-banner"
}
```

**TypeScript helper:**
```typescript
rune.remove_node("temp-banner");
```

---

## Navigate

Change the current route/view.

```json
{
  "type": "Navigate",
  "view_id": "settings",
  "params": {
    "tab": "profile"
  }
}
```

**TypeScript helper:**
```typescript
rune.navigate("settings", { tab: "profile" });
```

---

## NavigateBack

Return to the previous route.

```json
{
  "type": "NavigateBack"
}
```

**TypeScript helper:**
```typescript
rune.navigate_back();
```

---

## ShowOverlay / HideOverlay

Display or dismiss modal overlays.

```json
{
  "type": "ShowOverlay",
  "overlay_id": "confirm-dialog"
}
```

```json
{
  "type": "HideOverlay",
  "overlay_id": "confirm-dialog"
}
```

**TypeScript helpers:**
```typescript
rune.show_overlay("confirm-dialog");
rune.hide_overlay("confirm-dialog");
```

---

## Batching

Multiple mutations can be batched in a single dispatch for efficiency:

```json
{
  "type": "Batch",
  "mutations": [
    { "type": "UpdateData", "node_id": "a", "updates": { "x": 1 } },
    { "type": "UpdateData", "node_id": "b", "updates": { "y": 2 } }
  ]
}
```

This applies all mutations atomically before re-rendering.
