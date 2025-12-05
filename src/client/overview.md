# Client Overview

The Rune client runtime (`rune-wasm`) executes WebAssembly modules compiled from TypeScript controllers. It provides host functions for UI updates, state management, and communication with the server.

## Components

| Component | Crate | Description |
|-----------|-------|-------------|
| **rune-wasm** | `rune/crates/rune-wasm` | WASM executor, host functions |
| **rune-scene** | `rune/rune-scene` | Skia renderer, event handling |
| **rune-ir** | `rune/crates/rune-ir` | IR types, view/data parsing |

## Execution Model

```
┌──────────────────────────────────────────────────────────┐
│                      rune-scene                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Event Loop                                         │  │
│  │  • User input (click, key, scroll)                │  │
│  │  • Timer events                                   │  │
│  │  • Server messages                                │  │
│  └────────────────────────────────────────────────────┘  │
│                          │                               │
│                          ▼                               │
│  ┌────────────────────────────────────────────────────┐  │
│  │ rune-wasm Runtime                                  │  │
│  │  • Load .wasm modules from logic/                 │  │
│  │  • Call exported handlers (onclick, oninput)      │  │
│  │  • Process host function calls                    │  │
│  │  • Queue mutations                                │  │
│  └────────────────────────────────────────────────────┘  │
│                          │                               │
│                          ▼                               │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Mutation Dispatch                                  │  │
│  │  • Apply UI mutations to view tree                │  │
│  │  • Trigger re-render                              │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

## Module Lifecycle

1. **Load**: Runtime reads `.wasm` files from `logic/` directory
2. **Instantiate**: Wasmtime compiles and instantiates module
3. **Initialize**: Call `start()` export if present
4. **Execute**: Call event handlers as user interacts
5. **Tick**: Call `tick()` export each frame (if present)

## Exported Functions

Controllers should export these functions:

| Export | Signature | Called When |
|--------|-----------|-------------|
| `start` | `() -> void` | Module instantiated |
| `tick` | `() -> void` | Each frame (optional) |
| `on_*` | `(ptr, len) -> void` | Event handlers |

Event handlers receive JSON event data at `(ptr, len)` in WASM memory.

## Capabilities

The runtime can grant capabilities to WASM modules:

| Capability | Description |
|------------|-------------|
| `network` | Allow `assets_fetch` and HTTP |
| `storage` | Allow persistent local storage |

```rust
runtime.set_capabilities(["network", "storage"]);
```
