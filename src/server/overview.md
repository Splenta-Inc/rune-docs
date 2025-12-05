# Server Overview

The Rune server (`rune-server`) hosts Rune applications, executes server-side WASM services, and provides backend capabilities like database access, HTTP requests, caching, and filesystem operations.

## Components

| Component | Crate | Description |
|-----------|-------|-------------|
| **rune-server** | `rune-server/crates/rune-server` | HTTP server, WebSocket, routes |
| **rune-server-wasm** | `rune-server/crates/rune-server-wasm` | WASM executor, host functions |
| **rune-server-runtime** | `rune-server/crates/rune-server-runtime` | App store, service management |
| **rune-server-core** | `rune-server/crates/rune-server-core` | IR types, package loading |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        rune-server                               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ HTTP Server (Axum)                                        │  │
│  │  • GET /apps/{name}/ → Serve app package                  │  │
│  │  • WS  /apps/{name}/ws → WebSocket for RPC                │  │
│  │  • GET /apps/{name}/assets/* → Static assets              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                             │                                    │
│                             ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ App Store                                                 │  │
│  │  • Load .rune packages from --apps-dir                    │  │
│  │  • Parse manifests                                        │  │
│  │  • Hot reload on changes                                  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                             │                                    │
│                             ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Service Runtime                                           │  │
│  │  • Load service/*.wasm modules                            │  │
│  │  • Execute on client events                               │  │
│  │  • Provide host functions (DB, HTTP, cache, FS)           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Data Flow

**Important:** The server only returns **data updates**. It does not emit UI mutations. The client receives data updates and is responsible for updating the UI.

```
Client                          Server
  │                                │
  │  send_to_server({ type: "save", data: ... })
  │ ─────────────────────────────► │
  │                                │ service.ts handles event
  │                                │ db_execute(...)
  │                                │ emit data update
  │                                │
  │  { type: "set", key: "doc", value: {...} }
  │ ◄───────────────────────────── │
  │                                │
  │ Client updates UI based on data
```

## CLI Usage

```bash
# Start server
rune-server --port 3000 --apps-dir ./apps

# Options
--port <PORT>       Server port (default: 3000)
--apps-dir <PATH>   Directory containing .rune packages
--config <FILE>     Path to rune-server.toml config
```

## Configuration

`rune-server.toml`:
```toml
[server]
port = 3000
apps_dir = "./apps"

[database]
url = "sqlite:./data.db"

[cache]
max_size_mb = 100
default_ttl_secs = 3600

[filesystem]
allowed_paths = ["./uploads", "./temp"]
```

## Service Lifecycle

1. **Load**: Server reads `service/*.wasm` from package
2. **Instantiate**: Wasmtime compiles and instantiates module
3. **Initialize**: Call `start()` export if present
4. **Handle Events**: Call `on_client_event(ptr, len)` for each client message
5. **Return Data**: Collect data updates and send to client
