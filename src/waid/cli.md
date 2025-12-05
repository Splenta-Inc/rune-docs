# CLI Commands

The `waid` command-line tool provides commands for building, developing, and managing Rune projects.

## Commands Overview

| Command | Description |
|---------|-------------|
| `new` | Create a new project |
| `add` | Add pages/components |
| `build` | Build Rune package |
| `release` | Create `.rune` archive |
| `dev` | Development mode with hot reload |
| `compile` | Compile TypeScript to WASM |
| `ir` | Output IR as JSON |
| `routes` | List defined routes |
| `clean` | Clean build artifacts |
| `doctor` | Health check |

---

## waid new

Create a new WAID project from template.

```bash
waid new <project-name> [--force]
```

**Options:**
- `--force`, `-f`: Overwrite existing directory

**Example:**
```bash
waid new my-app
cd my-app
waid dev
```

**Creates:**
```
my-app/
├── waid.config.json
├── rune.toml
└── app/
    └── pages/
        └── home/
            ├── index.html
            ├── controller.ts
            └── service.ts
```

---

## waid add

Add pages or components to an existing project.

```bash
waid add page <page-name>
```

**Example:**
```bash
waid add page settings
# Creates app/pages/settings/{index.html, controller.ts, service.ts}
```

---

## waid build

Build the project into a Rune package.

```bash
waid build [--in <dir>] [--out <dir>] [--no-wasm]
```

**Options:**
- `--in <dir>`: Input directory (default: auto-detect)
- `--out <dir>`: Output directory (default: `build/<project-name>`)
- `--no-wasm`: Skip TypeScript → WASM compilation

**Example:**
```bash
waid build
# Output: build/my-app/

waid build --in ./src --out ./dist
```

**Output:**
```
build/my-app/
├── RUNE.MANIFEST.json
├── views/
│   └── home.view.json
├── logic/
│   └── home.wasm
├── service/
│   └── home.wasm
├── data/
│   └── home.data.json
└── assets/
```

---

## waid release

Create a distributable `.rune` archive.

```bash
waid release [--in <dir>] [--out <file>]
```

**Options:**
- `--in <dir>`: Input directory
- `--out <file>`: Output file (default: `<project-name>.rune`)

**Example:**
```bash
waid release
# Creates: my-app.rune

waid release --out dist/app-v1.0.rune
```

The `.rune` file is a ZIP archive containing:
- All package files
- `META-INF/BUILD.json` with build metadata
- Computed SHA256 hash

---

## waid dev

Development mode with file watching and hot reload.

```bash
waid dev [--in <dir>] [--port <port>] [--no-server] [--no-browser]
```

**Options:**
- `--in <dir>`: Input directory
- `--apps-dir <dir>`: Directory for rune-server to find apps
- `--port <port>`: Server port (default: 3000 or from config)
- `--no-server`: Don't start rune-server
- `--no-browser`: Don't start rune-browser

**Example:**
```bash
waid dev
# Starts watching, rebuilding, and hot reloading

waid dev --port 8080 --no-browser
# Custom port, use external browser
```

**What it does:**
1. Initial build
2. Start `rune-server` (unless `--no-server`)
3. Start `rune-browser` (unless `--no-browser`)
4. Watch for file changes
5. Rebuild on change
6. Send HMR reload to browser

---

## waid compile

Compile a single TypeScript file to WASM.

```bash
waid compile <file.ts> [--target client|service] [--out <file.wasm>] [--emit-wat]
```

**Options:**
- `--target <target>`: `client` (default) or `service`
- `--out <file>`: Output file (default: same name with `.wasm`)
- `--emit-wat`: Also emit `.wat` text format for debugging

**Example:**
```bash
waid compile controller.ts
# Output: controller.wasm

waid compile service.ts --target service --emit-wat
# Output: service.wasm, service.wat
```

---

## waid ir

Output the parsed IR (Intermediate Representation) as JSON.

```bash
waid ir [--in <dir>]
```

**Example:**
```bash
waid ir > ir.json
waid ir --in ./examples/counter
```

Useful for debugging and understanding how HTML is parsed.

---

## waid routes

List all defined routes in the project.

```bash
waid routes [--in <dir>]
```

**Example:**
```bash
waid routes
# Routes:
#   /
#   /settings
#   /users/:id
```

---

## waid clean

Clean build artifacts.

```bash
waid clean [--force] [--dir <dir>]
```

**Options:**
- `--force`, `-f`: Actually delete (without this, just shows what would be deleted)
- `--dir <dir>`: Directory to clean (default: `build`)

**Example:**
```bash
waid clean --force
# Removes build directory
```

---

## waid doctor

Check project health and dependencies.

```bash
waid doctor [--print-schema]
```

**Options:**
- `--print-schema`: Output the IR JSON schema

**Example:**
```bash
waid doctor
# Doctor: OK (schema available with --print-schema)

waid doctor --print-schema > schema.json
```

---

## Configuration

Projects can have a `waid.config.json`:

```json
{
  "project_id": "proj_abc123",
  "name": "my-app",
  "targets": ["web"],
  "pages_dir": "app/pages",
  "output": {
    "client_wasm_dir": "logic",
    "service_wasm_dir": "service"
  },
  "dev": {
    "rune_server": "../rune-server/target/release/rune-server",
    "rune_browser": "../rune/target/release/rune-scene",
    "rune_config": "./rune.toml",
    "port": 3000
  }
}
```

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `RUNE_SERVER` | Path to rune-server binary |
| `RUNE_BROWSER` | Path to rune-browser binary |
| `RUNE_CONFIG_FILE` | Path to rune.toml |
| `RUNE_SERVER_CONFIG_FILE` | Path to rune-server.toml |
