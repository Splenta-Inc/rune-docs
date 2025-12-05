# Client Host Functions

Host functions are imports provided by the runtime that WASM modules can call. The client runtime provides functions for logging, state management, UI mutations, and utilities.

## Implementation Status

| Function | Type | Status | Notes |
|----------|------|--------|-------|
| `log_debug` | Direct | ✅ Implemented | |
| `log_info` | Direct | ✅ Implemented | |
| `log_warn` | Direct | ✅ Implemented | |
| `log_error` | Direct | ✅ Implemented | |
| `get_local_state` | Direct | ✅ Implemented | |
| `set_local_state` | Direct | ✅ Implemented | |
| `delete_local_state` | Direct | ✅ Implemented | |
| `clear_local_state` | Direct | ✅ Implemented | |
| `get_time_ms` | Direct | ✅ Implemented | |
| `get_random` | Direct | ✅ Implemented | Simple LCG, not crypto |
| `uuid_v4` | Direct | ✅ Implemented | |
| `read_return_buffer` | Direct | ✅ Implemented | For string returns |
| `core_dispatch_mutation` | Routed | ✅ Implemented | |
| `update_data` | Routed | 🔨 Via dispatch | |
| `update_view` | Routed | 🔨 Via dispatch | |
| `replace_text` | Routed | 🔨 Via dispatch | |
| `remove_node` | Routed | 🔨 Via dispatch | |
| `navigate` | Routed | 🔨 Via dispatch | |
| `navigate_back` | Routed | 🔨 Via dispatch | |
| `show_overlay` | Routed | 🔨 Via dispatch | |
| `hide_overlay` | Routed | 🔨 Via dispatch | |
| `send_to_server` | Routed | ❌ Not yet | |
| `server_rpc` | Routed | ❌ Not yet | |
| `set_timeout` | Direct | ❌ Not yet | |
| `set_interval` | Direct | ❌ Not yet | |
| `clear_timer` | Direct | ❌ Not yet | |
| `clipboard_read` | Direct | ❌ Not yet | |
| `clipboard_write` | Direct | ❌ Not yet | |
| `assets_fetch` | Direct | 🔨 Stub only | |

**Legend:** ✅ Implemented | 🔨 Partial/Stub | ❌ Not implemented

---

## Logging

Fast, direct calls for debugging output.

```typescript
rune.log_debug(message: string): void
rune.log_info(message: string): void
rune.log_warn(message: string): void
rune.log_error(message: string): void
```

**WASM signature:** `(ptr: i32, len: i32) -> void`

**Example:**
```typescript
export function start(): void {
    rune.log_info("Controller initialized");
}
```

---

## Local State

Client-side key-value storage. Data persists for the session.

```typescript
rune.get_local_state(key: string): Uint8Array | null
rune.set_local_state(key: string, value: Uint8Array): void
rune.delete_local_state(key: string): boolean
rune.clear_local_state(): void
```

**WASM signatures:**
- `get_local_state(key_ptr, key_len) -> (has_value: i32, len: i32)`
- `set_local_state(key_ptr, key_len, val_ptr, val_len) -> void`
- `delete_local_state(key_ptr, key_len) -> i32`
- `clear_local_state() -> void`

**Example:**
```typescript
export function saveTheme(theme: string): void {
    rune.set_local_state("theme", theme);
}

export function loadTheme(): string {
    return rune.get_local_state("theme") ?? "light";
}
```

---

## Utilities

```typescript
rune.get_time_ms(): i64      // Unix timestamp in milliseconds
rune.get_random(): f64       // Random number 0.0-1.0
rune.uuid_v4(): string       // Random UUID v4
```

**WASM signatures:**
- `get_time_ms() -> i64`
- `get_random() -> f64`
- `uuid_v4() -> (has_value: i32, len: i32)` (use `read_return_buffer` to get string)

---

## Return Buffer

For functions that return strings, the host writes to an internal buffer. The guest must call `read_return_buffer` to copy the data into WASM memory.

```typescript
rune.read_return_buffer(dest_ptr: i32, max_len: i32): i32
```

Returns number of bytes copied.

**Pattern:**
```
1. Call uuid_v4() -> (1, 36)     // Returns (has_value=1, length=36)
2. Allocate 36 bytes in WASM memory
3. Call read_return_buffer(ptr, 36) -> 36
4. Read UUID string from ptr
```

---

## Mutation Dispatch

Complex operations go through `core_dispatch_mutation` with a JSON payload:

```typescript
rune.core_dispatch_mutation(json_ptr: i32, json_len: i32): void
```

The JSON must have a `type` field. See [UI Mutations](./ui-mutations.md) for mutation types.

**Example payload:**
```json
{
  "type": "UpdateData",
  "node_id": "counter",
  "updates": { "value": 42 }
}
```
