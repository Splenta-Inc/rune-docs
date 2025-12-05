# WAID Compiler Overview

WAID (WebAssembly Interface Definition) is a compiler that transforms HTML + TypeScript applications into Rune packages. It parses HTML with `data-*` attributes, TypeScript controllers/services, and emits platform-neutral artifacts.

## What WAID Does

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Your App      │     │      WAID        │     │   .rune pkg     │
│  index.html     │ ──► │    Compiler      │ ──► │  views/*.json   │
│  controller.ts  │     │  (Rust)          │     │  logic/*.wasm   │
│  service.ts     │     │                  │     │  service/*.wasm │
│  assets/*       │     │                  │     │  assets/*       │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

## Compilation Pipeline

1. **Parse HTML** → IR (Intermediate Representation)
   - Extract `data-*` attributes
   - Build view tree
   - Identify bindings and event handlers

2. **Parse TypeScript** → AST (via SWC)
   - Extract state definitions
   - Identify exported functions
   - Resolve host function calls

3. **Compile to WASM** → Binary modules
   - TypeScript → WASM via wasm-encoder
   - Controllers → `logic/*.wasm`
   - Services → `service/*.wasm`

4. **Generate Package** → `.rune` bundle
   - `RUNE.MANIFEST.json`
   - View JSON files
   - WASM modules
   - Static assets

## Architecture

```
waid_frontend/
├── src/
│   ├── parser_html/    # HTML parser with data-* extraction
│   ├── ir/             # Intermediate representation types
│   ├── wasm/           # TypeScript → WASM compiler
│   │   ├── compiler.rs # Main compiler
│   │   ├── codegen.rs  # WASM instruction generation
│   │   ├── types.rs    # Type mapping (TS ↔ WASM)
│   │   └── host.rs     # Host function definitions
│   └── lib.rs
│
waid_emitters/
├── src/
│   └── rune/           # Rune package generator
│
waidc/
├── src/
│   └── main.rs         # CLI entry point
└── scaffold/           # Project templates
```

## Key Features

| Feature | Status | Notes |
|---------|--------|-------|
| HTML parsing | ✅ | `data-*` attributes |
| TypeScript parsing | ✅ | Via SWC |
| WASM compilation | ✅ | Primitives, functions, control flow |
| Host functions | ✅ | 35+ client/server functions |
| Package generation | ✅ | `.rune` bundles |
| Hot reload | ✅ | `waid dev` with HMR |
| Project scaffolding | ✅ | `waid new` |

## Supported TypeScript Features

See [TypeScript Support](./typescript.md) for details on what TypeScript features compile to WASM.

## HTML Attributes

See [HTML Attributes](./html-attributes.md) for the `data-*` attribute reference.
