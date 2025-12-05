# Platform Features & Flow

This page gives a feature-level overview of the Rune platform: the client runtime, the server runtime, and the WAID compiler, plus how they work together end-to-end.

## Rune (Client Runtime)

Rune runs your compiled applications on end-user devices (desktop, browser, mobile shells).

- Executes WebAssembly produced by WAID
- Renders native-feeling UI from the Rune IR
- Loads app/asset data over HTTP and maintains a WebSocket connection to `rune-server` for RPC and data sync
- Provides host functions for I/O, timers, storage, etc.
- Supports hot reload during development when paired with WAID

## rune-server (Server Runtime)

`rune-server` hosts applications and services in a production or staging environment.

- Serves `.rune` bundles and static assets
- Runs server-side services as WASM modules
- Exposes host functions for HTTP, databases, queues, and other backends
- Manages client connections via WebSocket for real-time sync
- Supports multi-app hosting on a single server

## WAID (Compiler & Tooling)

WAID is the compiler and CLI that turns your source code into something Rune can execute.

- Compiles HTML + TypeScript into Rune IR + WASM
- Produces `.rune` bundles ready to deploy or open
- Provides a dev server with live reload (`waid dev`)
- Offers TypeScript-aware authoring with strong typing
- Includes a CLI for scaffolding, building, and releasing apps

## How It All Operates

At a high level, the flow looks like this:

1. **Authoring** – You write `index.html`, `controller.ts`, and `service.ts` using WAID conventions.
2. **Build** – WAID compiles your project to Rune IR + WASM and packages it into a `.rune` bundle.
3. **Deploy** – You serve the `.rune` bundle from `rune-server` or ship it with a client shell.
4. **Run on Clients** – Rune loads the IR, executes the WASM modules, and renders UI on the client.
5. **Client–Server Sync** – The design is for the client to keep a WebSocket connection to `rune-server` for services and state sync; this flow is only partially implemented today (see [Status & Roadmap](./status.md)).

For more detail, see:

- [Architecture](./architecture.md)
- [Data Flow](./data-flow.md)
- [Client Overview](./client/overview.md)
- [Server Overview](./server/overview.md)
- [WAID Overview](./waid/overview.md)
