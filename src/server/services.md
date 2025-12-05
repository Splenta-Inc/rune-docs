# Services

Services are server-side WASM modules that handle client events and provide backend functionality.

## Service Structure

Each page/route can have a `service.ts` file that compiles to a WASM module:

```
app/pages/home/
├── index.html      # View template
├── controller.ts   # Client-side logic → logic/home.wasm
└── service.ts      # Server-side logic → service/home.wasm
```

## Service Exports

Services should export these functions:

| Export | Signature | Description |
|--------|-----------|-------------|
| `start` | `() -> void` | Called on instantiation |
| `on_client_event` | `(ptr, len) -> void` | Handle client events |
| `on_timer` | `(ptr, len) -> void` | Handle timer callbacks |
| `tick` | `() -> void` | Called periodically (optional) |

## Basic Service

```typescript
// service.ts

export function start(): void {
    rune.log_info("Service started");
}

export function on_client_event(event: object): void {
    switch (event.type) {
        case "load":
            handleLoad();
            break;
        case "save":
            handleSave(event.data);
            break;
        default:
            rune.log_warn("Unknown event: " + event.type);
    }
}

function handleLoad(): void {
    const rows = rune.db_query({
        query_type: "select",
        sql: "SELECT * FROM items ORDER BY created_at DESC"
    });

    const result = rune.get_db_result(rows);
    if (result.success) {
        rune.set_data("items", result.rows);
    }
}

function handleSave(data: object): void {
    rune.db_query({
        query_type: "insert",
        sql: "INSERT INTO items (name, value) VALUES (?, ?)",
        params: [data.name, data.value]
    });

    // Append to client's list
    rune.append_data("items", data);
}
```

## Client-Server Communication

### From Client (controller.ts)

```typescript
// Send event to server
export function loadItems(): void {
    rune.send_to_server({ type: "load" });
}

export function saveItem(name: string, value: number): void {
    rune.send_to_server({
        type: "save",
        data: { name, value }
    });
}

// RPC-style call with response
export function getUser(userId: string): void {
    const requestId = rune.server_rpc("getUser", { id: userId });
    // Response arrives via on_server_response
}

export function on_server_response(response: object): void {
    if (response.method === "getUser") {
        rune.update_data("user-profile", response.result);
    }
}
```

### From Server (service.ts)

```typescript
export function on_client_event(event: object): void {
    // Process event and emit data updates
    rune.set_data("items", items);

    // Or send specific message
    rune.send_to_client({
        type: "notification",
        message: "Item saved successfully"
    });
}
```

## Async Operations

Server operations (HTTP, DB, FS) are async. Use the queue pattern:

```typescript
export function on_client_event(event: object): void {
    if (event.type === "fetch_weather") {
        // Queue HTTP request
        const reqId = rune.http_fetch({
            method: "GET",
            url: "https://api.weather.com/current?city=" + event.city
        });

        // Store request ID for later
        rune.set_state("pending_weather", reqId);
    }
}

// Called periodically to check for responses
export function tick(): void {
    const reqId = rune.get_state("pending_weather");
    if (reqId) {
        const response = rune.get_http_response(reqId);
        if (response) {
            rune.delete_state("pending_weather");

            if (response.success) {
                const weather = JSON.parse(response.body);
                rune.set_data("weather", weather);
            } else {
                rune.set_data("weather_error", response.error);
            }
        }
    }
}
```

## Service Isolation

Each service instance has:
- Its own state storage (via `get/set_state`)
- Shared cache (via `cache_*` functions)
- Shared database connection
- Sandboxed filesystem access

## Best Practices

1. **Validate input**: Always validate client events before processing
2. **Handle errors**: Check `success` on async results, handle failures gracefully
3. **Use transactions**: For multi-step DB operations, use transactions
4. **Cache wisely**: Cache expensive computations with appropriate TTL
5. **Log appropriately**: Use log levels correctly (debug for verbose, error for failures)
6. **Return minimal data**: Only send data the client needs
