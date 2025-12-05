# State Management

Rune has three levels of state, each with different scope and persistence.

## State Levels

| Level | Scope | Persistence | Access |
|-------|-------|-------------|--------|
| **View State** | Per component | Render cycle | `data-bind`, mutations |
| **Local State** | Client session | Session | `get/set_local_state` |
| **Server State** | All clients | Database | Via service.ts |

---

## View State

View state lives in the IR view JSON and is updated via mutations.

**Initial state** (from `data/main.data.json`):
```json
{
  "counter": {
    "value": 0
  }
}
```

**HTML binding:**
```html
<span id="counter" data-bind="value">0</span>
```

**Update from controller:**
```typescript
export function inc(): void {
    // This updates view state
    rune.update_data("counter", { value: currentValue + 1 });
}
```

**Characteristics:**
- Reactive: Changes trigger re-render
- Scoped to view/component
- Lost on navigation (unless persisted)

---

## Local State

Client-side key-value storage via host functions.

```typescript
// Save
rune.set_local_state("user_preferences", JSON.stringify({
    theme: "dark",
    fontSize: 16
}));

// Load
const prefs = rune.get_local_state("user_preferences");
if (prefs) {
    const parsed = JSON.parse(prefs);
    applyTheme(parsed.theme);
}

// Delete
rune.delete_local_state("user_preferences");

// Clear all
rune.clear_local_state();
```

**Characteristics:**
- Persists across navigation
- Lost on app close (session only)
- Client-specific (not synced)
- No size limit (but be reasonable)

**Use cases:**
- User preferences
- Form drafts
- UI state (collapsed panels, scroll position)
- Cached data

---

## Server State

State that needs to persist or be shared across clients goes through server services.

**In controller.ts (client):**
```typescript
export function saveDocument(doc: object): void {
    rune.send_to_server({
        type: "save_document",
        data: doc
    });
}
```

**In service.ts (server):**
```typescript
export function on_client_event(event: object): void {
    if (event.type === "save_document") {
        rune.db_execute(
            "INSERT INTO documents (data) VALUES (?)",
            [JSON.stringify(event.data)]
        );
    }
}
```

**Characteristics:**
- Persists across sessions
- Shared across clients
- Requires server round-trip
- Subject to permissions/auth

---

## State Flow Example

```
┌─────────────────────────────────────────────────────────────────┐
│                        User clicks "Save"                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ controller.ts                                                    │
│                                                                  │
│   // 1. Update view state (immediate feedback)                   │
│   rune.update_data("save-btn", { text: "Saving..." });          │
│                                                                  │
│   // 2. Cache locally (offline support)                          │
│   rune.set_local_state("draft", JSON.stringify(doc));           │
│                                                                  │
│   // 3. Send to server (persistence)                             │
│   rune.send_to_server({ type: "save", data: doc });             │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ service.ts (server)                                              │
│                                                                  │
│   // 4. Persist to database                                      │
│   rune.db_execute("INSERT INTO docs ...", [doc]);               │
│                                                                  │
│   // 5. Notify client                                            │
│   rune.send_to_client({ type: "saved", id: newId });            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ controller.ts (on response)                                      │
│                                                                  │
│   // 6. Update UI with confirmation                              │
│   rune.update_data("save-btn", { text: "Saved ✓" });            │
│                                                                  │
│   // 7. Clear local draft                                        │
│   rune.delete_local_state("draft");                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Best Practices

1. **Use view state for UI**: Bindings, counters, form values
2. **Use local state for preferences**: Theme, layout, cached data
3. **Use server state for persistence**: User data, documents, shared state
4. **Optimistic updates**: Update UI immediately, sync with server async
5. **Handle offline**: Cache in local state, sync when reconnected
