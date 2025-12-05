# Data Flow

## Event → Mutation → Render

The core data flow in Rune follows a unidirectional pattern:

```
┌─────────┐     ┌─────────┐     ┌──────────┐     ┌────────┐
│  Event  │ ──► │  WASM   │ ──► │ Mutation │ ──► │ Render │
│ (click) │     │ Handler │     │ Dispatch │     │   UI   │
└─────────┘     └─────────┘     └──────────┘     └────────┘
```

### Example Flow

1. User clicks a button with `data-onclick="inc"`
2. Runtime calls exported `inc` function in WASM
3. WASM calls `rune.update_data("counter", { value: 5 })`
4. Runtime applies mutation to view state
5. UI re-renders affected components

## Client ↔ Server Communication

```
┌────────────────────┐                    ┌────────────────────┐
│      Client        │                    │      Server        │
│                    │                    │                    │
│  controller.ts     │   WebSocket/RPC    │   service.ts       │
│  ───────────────   │◄──────────────────►│   ────────────     │
│  rune.send_to_     │                    │   on_client_       │
│    server(event)   │                    │     event(e)       │
│                    │                    │                    │
│  on_server_        │                    │   rune.send_to_    │
│    response(r)     │                    │     client(data)   │
└────────────────────┘                    └────────────────────┘
```

### Message Types

| Direction | Method | Description |
|-----------|--------|-------------|
| Client → Server | `rune.send_to_server(event)` | Fire-and-forget event |
| Client → Server | `rune.server_rpc(method, params)` | Request with response |
| Server → Client | `rune.send_to_client(data)` | Push update to client |

## IR Mutations

All UI changes go through the IR Mutation system:

```typescript
// TypeScript (in controller)
rune.update_data("user-name", { text: "Alice" });

// Becomes mutation
{
  "type": "UpdateData",
  "node_id": "user-name",
  "updates": { "text": "Alice" }
}

// Applied to view
{
  "id": "user-name",
  "tag": "span",
  "data": { "text": "Alice" }  // ← updated
}
```

### Mutation Types

| Mutation | Description |
|----------|-------------|
| `UpdateData` | Update node's data bindings |
| `UpdateView` | Update node's visual properties |
| `ReplaceText` | Replace text content |
| `RemoveNode` | Remove node from tree |
| `Navigate` | Change current route/view |
| `ShowOverlay` | Display modal/overlay |
| `HideOverlay` | Dismiss overlay |

## State Scopes

| Scope | Location | Persistence | Use Case |
|-------|----------|-------------|----------|
| View State | In view JSON | Per render | UI bindings |
| Local State | Client runtime | Session | User preferences |
| Server State | Server runtime | Session/DB | Shared data |

```typescript
// View state (via mutations)
rune.update_data("node", { count: 5 });

// Local state (client only)
rune.set_local_state("theme", "dark");
const theme = rune.get_local_state("theme");

// Server state (via service)
// In service.ts:
rune.db_query("SELECT * FROM users");
```
