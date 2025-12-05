# Status & Roadmap

Current implementation status across all Rune platform components.

## Overall Status

| Component | Status | Notes |
|-----------|--------|-------|
| **WAID Compiler** | 🟢 Functional | HTML, TS→WASM, package gen |
| **rune-wasm (Client)** | 🟡 Partial | Core host functions done |
| **rune-browser** | 🟢 Functional | Skia rendering, events |
| **rune-server** | 🟡 Partial | HTTP server, types defined |

**Legend:** 🟢 Functional | 🟡 Partial | 🔴 Not started

---

## WAID Compiler

### Core Features

| Feature | Status |
|---------|--------|
| HTML parsing | ✅ Done |
| `data-*` attribute extraction | ✅ Done |
| IR generation | ✅ Done |
| TypeScript parsing (SWC) | ✅ Done |
| WASM compilation | ✅ Done |
| Package generation | ✅ Done |
| `.rune` bundling | ✅ Done |
| Hot reload (`waid dev`) | ✅ Done |
| Project scaffolding | ✅ Done |

### TypeScript → WASM

| Feature | Status |
|---------|--------|
| Primitives (number, boolean, string) | ✅ Done |
| Variables (let, const) | ✅ Done |
| Arithmetic operators | ✅ Done |
| Comparison operators | ✅ Done |
| Logical operators | ✅ Done |
| Control flow (if/else, while, for) | ✅ Done |
| Functions (export, params, return) | ✅ Done |
| Host function calls | ✅ Done |
| Classes | ❌ Not yet |
| Arrow functions | ❌ Not yet |
| Async/await | ❌ Not planned |

---

## Client Runtime (rune-wasm)

### Host Functions

| Category | Function | Status |
|----------|----------|--------|
| **Logging** | log_debug/info/warn/error | ✅ Done |
| **Local State** | get/set/delete/clear | ✅ Done |
| **Utilities** | get_time_ms | ✅ Done |
| | get_random | ✅ Done |
| | uuid_v4 | ✅ Done |
| **Mutations** | core_dispatch_mutation | ✅ Done |
| | update_data | 🔨 Via dispatch |
| | update_view | 🔨 Via dispatch |
| | navigate | 🔨 Via dispatch |
| **Server Comm** | send_to_server | ❌ Not yet |
| | server_rpc | ❌ Not yet |
| **Timers** | set_timeout/interval | ❌ Not yet |
| | clear_timer | ❌ Not yet |
| **Clipboard** | read/write | ❌ Not yet |

### Rendering (rune-scene)

| Feature | Status |
|---------|--------|
| Skia canvas | ✅ Done |
| Text rendering | ✅ Done |
| Event handling | ✅ Done |
| View tree rendering | ✅ Done |
| HMR support | ✅ Done |
| Routing/navigation | 🔨 Partial |

---

## Server Runtime (rune-server)

### Core

| Feature | Status |
|---------|--------|
| HTTP server (Axum) | ✅ Done |
| App hosting | ✅ Done |
| WebSocket connections | ✅ Done |
| Asset serving | ✅ Done |
| WASM service execution | 🔨 Types defined |

### Host Functions

| Category | Function | Status |
|----------|----------|--------|
| **Logging** | log_* | ✅ Done |
| **State** | get/set/delete/clear | ✅ Done |
| **Data Updates** | set/delete/append/merge | ✅ Done |
| **HTTP** | http_fetch | 🔨 Types defined |
| **Database** | db_query | 🔨 Types defined |
| **Cache** | get/set/delete/clear | ✅ Done |
| **Filesystem** | read/write/list/mkdir | 🔨 Types defined |
| **Timers** | set_timer/clear_timer | ✅ Done |

---

## Integration Status

| Flow | Status | Notes |
|------|--------|-------|
| WAID → Package | ✅ Working | Full pipeline |
| Package → rune-browser | ✅ Working | View rendering |
| Package → rune-server | ✅ Working | App hosting |
| WASM → Client mutations | ✅ Working | update_data, navigate |
| Client ↔ Server events | 🔨 Partial | WebSocket connected |
| Server → DB queries | ❌ Not yet | Types ready, impl pending |
| Server → HTTP fetch | ❌ Not yet | Types ready, impl pending |

---

## Roadmap

### Near Term
- [ ] Complete client ↔ server event flow
- [ ] Implement server DB execution
- [ ] Implement server HTTP fetch
- [ ] Add timer support to client
- [ ] Improve error messages

### Medium Term
- [ ] Class support in TS→WASM
- [ ] Component composition
- [ ] Shared state management
- [ ] Animation support
- [ ] Form handling helpers

### Future
- [ ] Android emitter (Jetpack Compose)
- [ ] iOS emitter (SwiftUI)
- [ ] Desktop emitter (Tauri)
- [ ] Plugin system
- [ ] Package registry

---

## Version History

| Version | Date | Highlights |
|---------|------|------------|
| 0.1.0 | Current | Initial WASM compiler, basic runtime |

---

## Contributing

The project is in active development. Key areas needing work:

1. **Server host function execution** - Connect types to actual DB/HTTP/FS
2. **Client-server integration** - Complete the event flow
3. **TypeScript features** - Classes, more operators
4. **Documentation** - Examples, tutorials

See the individual repos for contribution guidelines.
