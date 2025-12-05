# Architecture

## Overview

Rune uses a **compile-time + runtime** architecture:

1. **Compile time (WAID)**: Transforms HTML + TypeScript into platform-neutral artifacts
2. **Runtime (rune-wasm/rune-server)**: Executes WASM and renders native UI

## Package Structure

A compiled `.rune` package contains:

```
my-app.rune/
├── RUNE.MANIFEST.json    # App metadata, routes, integrity
├── views/
│   ├── main.view.json    # UI structure (from HTML)
│   └── settings.view.json
├── logic/
│   ├── main.wasm         # Client controller (from controller.ts)
│   └── settings.wasm
├── service/
│   └── main.wasm         # Server service (from service.ts)
├── data/
│   └── main.data.json    # Initial state
└── assets/
    ├── images/
    └── styles/
```

## Runtime Architecture

### Client (rune-wasm)

```
┌─────────────────────────────────────────────────────────┐
│                    rune-browser                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              rune-scene (Renderer)               │    │
│  │  • Skia-based rendering                         │    │
│  │  • Native UI components                         │    │
│  │  • Event handling                               │    │
│  └─────────────────────────────────────────────────┘    │
│                         │                                │
│                         ▼                                │
│  ┌─────────────────────────────────────────────────┐    │
│  │              rune-wasm (Runtime)                 │    │
│  │  • Wasmtime WASM executor                       │    │
│  │  • Host function implementation                 │    │
│  │  • State management                             │    │
│  │  • Mutation dispatch                            │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### Server (rune-server)

```
┌─────────────────────────────────────────────────────────┐
│                    rune-server                           │
│  ┌─────────────────────────────────────────────────┐    │
│  │              HTTP Server (Axum)                  │    │
│  │  • App hosting (/apps/{name}/)                  │    │
│  │  • WebSocket connections                        │    │
│  │  • Asset serving                                │    │
│  └─────────────────────────────────────────────────┘    │
│                         │                                │
│                         ▼                                │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Service Runtime                     │    │
│  │  • WASM execution for services                  │    │
│  │  • Database access                              │    │
│  │  • HTTP client                                  │    │
│  │  • Cache, filesystem                            │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

## WASM Host Functions

WASM modules communicate with the runtime through **host functions** - imports provided by the runtime that the WASM code can call.

Two types of host functions:

### Direct Calls
- Fast, no serialization overhead
- Simple parameters (i32, i64, f64, pointers)
- Used for: logging, time, random, simple state access

### Routed Calls (Mutations)
- JSON-serialized payloads
- Complex operations
- Used for: UI updates, navigation, server communication

See [Client Host Functions](./client/host-functions.md) and [Server Host Functions](./server/host-functions.md) for details.

## String Passing Convention

Strings are passed between WASM and host using pointer + length:

```
WASM Memory:
┌──────────────────────────────────────────┐
│ "Hello, World!" at offset 0x0400, len 13 │
└──────────────────────────────────────────┘

Function call: rune.log_info(0x0400, 13)
                              ↑      ↑
                             ptr    len
```

For return values, host writes to a buffer and guest reads via `read_return_buffer`.
