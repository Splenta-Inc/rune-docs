# Rune Platform

Rune is a cross-platform application runtime that executes WebAssembly modules with native UI rendering. Applications are written in TypeScript/HTML using WAID, compiled to WASM, and run on clients (browser, desktop, mobile) with optional server-side logic.

## Components

| Component | Description | Repository |
|-----------|-------------|------------|
| **WAID** | Compiler: HTML + TypeScript → WASM + IR | `WAID/` |
| **rune-wasm** | Client runtime: executes WASM, renders UI | `rune/crates/rune-wasm` |
| **rune-browser** | Desktop shell (Tauri-based) | `rune/rune-scene` |
| **rune-server** | Server runtime: hosts apps, runs services | `rune-server/` |

## How It Works

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Your App      │     │      WAID        │     │   .rune pkg     │
│  index.html     │ ──► │    Compiler      │ ──► │  views/*.json   │
│  controller.ts  │     │                  │     │  logic/*.wasm   │
│  service.ts     │     │                  │     │  assets/*       │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                         │
                        ┌────────────────────────────────┴────────────┐
                        ▼                                             ▼
               ┌─────────────────┐                         ┌─────────────────┐
               │   rune-browser  │                         │   rune-server   │
               │   (Client)      │◄───── WebSocket ───────►│   (Server)      │
               │                 │                         │                 │
               │  • Load views   │                         │  • Host apps    │
               │  • Run WASM     │                         │  • Run services │
               │  • Render UI    │                         │  • HTTP/DB/etc  │
               └─────────────────┘                         └─────────────────┘
```

## Quick Start

```bash
# Create a new project
waid new my-app
cd my-app

# Development mode (watch + hot reload)
waid dev

# Build for production
waid build
waid release  # Creates my-app.rune bundle
```

## Current Status

See [Status & Roadmap](./status.md) for what's implemented and what's planned.
