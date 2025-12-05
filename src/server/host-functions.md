# Server Host Functions

Server host functions provide backend capabilities to WASM service modules. Unlike client functions, server functions handle database access, HTTP requests, caching, and filesystem operations.

## Implementation Status

| Category | Function | Status | Notes |
|----------|----------|--------|-------|
| **Logging** | `log_debug` | ✅ Implemented | |
| | `log_info` | ✅ Implemented | |
| | `log_warn` | ✅ Implemented | |
| | `log_error` | ✅ Implemented | |
| **State** | `get_state` | ✅ Implemented | Per-instance |
| | `set_state` | ✅ Implemented | |
| | `delete_state` | ✅ Implemented | |
| | `clear_state` | ✅ Implemented | |
| **Data Updates** | `set_data` | ✅ Implemented | Returns to client |
| | `delete_data` | ✅ Implemented | |
| | `append_data` | ✅ Implemented | |
| | `remove_data` | ✅ Implemented | |
| | `merge_data` | ✅ Implemented | |
| **HTTP** | `http_fetch` | ✅ Types defined | Queue-based |
| | `get_http_response` | ✅ Types defined | |
| **Database** | `db_query` | ✅ Types defined | Queue-based |
| | `get_db_result` | ✅ Types defined | |
| **Cache** | `cache_get` | ✅ Implemented | With TTL |
| | `cache_set` | ✅ Implemented | |
| | `cache_delete` | ✅ Implemented | |
| | `cache_has` | ✅ Implemented | |
| | `cache_clear` | ✅ Implemented | |
| **Filesystem** | `fs_read` | ✅ Types defined | Sandboxed |
| | `fs_write` | ✅ Types defined | |
| | `fs_delete` | ✅ Types defined | |
| | `fs_exists` | ✅ Types defined | |
| | `fs_list` | ✅ Types defined | |
| | `fs_mkdir` | ✅ Types defined | |
| **Timers** | `set_timer` | ✅ Implemented | |
| | `clear_timer` | ✅ Implemented | |

**Legend:** ✅ Implemented/Types defined | 🔨 Partial | ❌ Not implemented

---

## Logging

```typescript
rune.log_debug(message: string): void
rune.log_info(message: string): void
rune.log_warn(message: string): void
rune.log_error(message: string): void
```

Logs are collected and flushed to the server's tracing output.

---

## State Management

Per-instance key-value storage for service state.

```typescript
rune.get_state(key: string): Uint8Array | null
rune.set_state(key: string, value: Uint8Array): void
rune.delete_state(key: string): boolean
rune.clear_state(): void
```

---

## Data Updates

**Important:** Server services return data updates, NOT UI mutations. The client receives these and updates its local state/UI.

```typescript
// Set/replace a value
rune.set_data(key: string, value: any): void

// Delete a value
rune.delete_data(key: string): void

// Append to an array
rune.append_data(key: string, item: any): void

// Remove from an array
rune.remove_data(key: string, index?: number, whereId?: any): void

// Merge/patch an object
rune.merge_data(key: string, patch: object): void
```

**Example:**
```typescript
export function on_client_event(event: object): void {
    if (event.type === "load_todos") {
        const rows = rune.db_query("SELECT * FROM todos");
        rune.set_data("todos", rows);
    }

    if (event.type === "add_todo") {
        rune.db_execute("INSERT INTO todos (text) VALUES (?)", [event.text]);
        rune.append_data("todos", { id: lastId, text: event.text });
    }
}
```

### Data Update Types

| Type | Payload | Description |
|------|---------|-------------|
| `set` | `{ key, value }` | Set/replace value |
| `delete` | `{ key }` | Delete value |
| `append` | `{ key, value }` | Append to array |
| `remove` | `{ key, index?, where_id? }` | Remove from array |
| `merge` | `{ key, patch }` | Merge into object |

---

## HTTP Requests

Make outbound HTTP requests from service code.

```typescript
// Queue a request
const requestId = rune.http_fetch({
    method: "GET",
    url: "https://api.example.com/data",
    headers: { "Authorization": "Bearer token" },
    timeout_ms: 5000
});

// Get response (async - check periodically or use callback)
const response = rune.get_http_response(requestId);
if (response) {
    if (response.error) {
        rune.log_error("HTTP failed: " + response.error);
    } else {
        const data = JSON.parse(response.body);
        rune.set_data("api_data", data);
    }
}
```

### HttpRequest

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Request ID (auto-generated) |
| `method` | string | GET, POST, PUT, DELETE, etc. |
| `url` | string | Full URL |
| `headers` | object | Request headers |
| `body` | Uint8Array? | Request body |
| `timeout_ms` | number? | Timeout in milliseconds |

### HttpResponse

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Request ID |
| `status` | number | HTTP status code |
| `headers` | object | Response headers |
| `body` | Uint8Array | Response body |
| `error` | string? | Error message if failed |

---

## Database

Execute SQL queries against the configured database.

```typescript
// SELECT query
const queryId = rune.db_query({
    query_type: "select",
    sql: "SELECT * FROM users WHERE id = ?",
    params: [userId]
});

// Get result
const result = rune.get_db_result(queryId);
if (result.success) {
    rune.set_data("user", result.rows[0]);
} else {
    rune.log_error("Query failed: " + result.error);
}

// INSERT/UPDATE/DELETE
const insertId = rune.db_query({
    query_type: "insert",
    sql: "INSERT INTO users (name, email) VALUES (?, ?)",
    params: [name, email]
});
const insertResult = rune.get_db_result(insertId);
// insertResult.last_insert_id, insertResult.affected_rows
```

### DbQuery

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Query ID (auto-generated) |
| `query_type` | string | select, insert, update, delete, execute |
| `sql` | string | SQL with ? placeholders |
| `params` | array | Query parameters |
| `timeout_ms` | number? | Timeout |

### DbResult

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Query ID |
| `success` | boolean | Whether query succeeded |
| `rows` | array | Result rows (SELECT) |
| `affected_rows` | number? | Rows affected (INSERT/UPDATE/DELETE) |
| `last_insert_id` | number? | Last insert ID |
| `error` | string? | Error message |

---

## Cache

In-memory cache with optional TTL. Shared across service instances.

```typescript
// Set with 1 hour TTL
rune.cache_set("user:123", JSON.stringify(userData), 3600);

// Get (returns null if expired or missing)
const cached = rune.cache_get("user:123");
if (cached) {
    return JSON.parse(cached);
}

// Check existence
if (rune.cache_has("user:123")) { ... }

// Delete
rune.cache_delete("user:123");

// Clear all
rune.cache_clear();

// List keys
const keys = rune.cache_keys();
```

---

## Filesystem

Sandboxed filesystem access. Only paths in `allowed_paths` config are accessible.

```typescript
// Read file
const readId = rune.fs_read("/uploads/doc.txt");
const result = rune.get_fs_result(readId);
if (result.success) {
    const content = new TextDecoder().decode(result.data);
}

// Write file
rune.fs_write("/uploads/new.txt", new TextEncoder().encode("content"));

// Check existence
rune.fs_exists("/uploads/doc.txt");

// List directory
rune.fs_list("/uploads");
// result.entries: [{ name, is_dir, size, modified_ms }]

// Create directory
rune.fs_mkdir("/uploads/subdir");

// Delete
rune.fs_delete("/uploads/old.txt");
```

### Security

Filesystem operations are sandboxed:
- Only paths under `allowed_paths` in config are accessible
- Path traversal (`../`) is blocked
- Operations outside allowed paths return errors

---

## Timers

Schedule delayed or repeated execution.

```typescript
// One-shot timer (5 seconds)
rune.set_timer("reminder", 5000, false);

// Repeating timer (every second)
rune.set_timer("heartbeat", 1000, true);

// Cancel timer
rune.clear_timer("heartbeat");
```

Timer callbacks invoke `on_timer(timer_id)` export in the service module.
