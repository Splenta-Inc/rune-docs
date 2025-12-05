# Getting Started

This guide walks you through creating your first Rune application with WAID.

## Prerequisites

- Rust toolchain (for building WAID)
- rune-server and rune-browser (for running apps)

## Installation

### Build WAID

```bash
cd WAID
cargo build --release
# Binary at: target/release/waid
```

### Build Rune Runtime

```bash
cd rune
cargo build --release
# rune-scene at: target/release/rune-scene
```

### Build Rune Server

```bash
cd rune-server
cargo build --release
# rune-server at: target/release/rune-server
```

## Create a New Project

```bash
waid new my-app
cd my-app
```

This creates:

```
my-app/
├── waid.config.json      # Project configuration
├── rune.toml             # Runtime configuration
└── app/
    └── pages/
        └── home/
            ├── index.html      # View template
            ├── controller.ts   # Client-side logic
            └── service.ts      # Server-side logic
```

## Project Structure

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>My App</title>
</head>
<body>
    <div data-component="home" data-controller="./controller.ts">
        <h1>Counter</h1>
        <p>Count: <span id="count" data-bind="count">0</span></p>
        <button data-onclick="inc">+</button>
        <button data-onclick="dec">-</button>
    </div>
</body>
</html>
```

### controller.ts

```typescript
let count = 0;

export function start(): void {
    rune.log_info("Controller started");
}

export function inc(): void {
    count = count + 1;
    rune.update_data("count", { text: count.toString() });
}

export function dec(): void {
    count = count - 1;
    rune.update_data("count", { text: count.toString() });
}
```

### service.ts (optional)

```typescript
export function start(): void {
    rune.log_info("Service started");
}

export function on_client_event(event: object): void {
    rune.log_info("Received event: " + JSON.stringify(event));
}
```

## Development

Start the development server with hot reload:

```bash
waid dev
```

This:
1. Builds your project
2. Starts rune-server on port 3000
3. Opens rune-browser
4. Watches for file changes
5. Rebuilds and hot reloads on changes

Open in browser: `http://127.0.0.1:3000/apps/my-app/`

## Build for Production

```bash
# Build package
waid build

# Create distributable archive
waid release
# Creates: my-app.rune
```

## Configuration

### waid.config.json

```json
{
  "name": "my-app",
  "dev": {
    "port": 3000,
    "rune_server": "/path/to/rune-server",
    "rune_browser": "/path/to/rune-scene"
  }
}
```

### rune.toml

```toml
[app]
name = "my-app"
entry = "/"

[ui]
background_color = "#ffffff"
```

## Adding Pages

```bash
waid add page settings
```

Creates `app/pages/settings/` with index.html, controller.ts, service.ts.

Link to the new page:

```html
<a data-link="/settings">Settings</a>
```

Or navigate programmatically:

```typescript
export function goToSettings(): void {
    rune.navigate("settings");
}
```

## Common Patterns

### Data Binding

```html
<input id="name-input" data-oninput="updateName($event.target.value)">
<p>Hello, <span id="greeting" data-bind="name">World</span>!</p>
```

```typescript
export function updateName(name: string): void {
    rune.update_data("greeting", { text: name });
}
```

### Conditional Rendering

```html
<div data-if="isLoading">Loading...</div>
<div data-if="!isLoading">
    <span data-bind="data">No data</span>
</div>
```

### List Rendering

```html
<ul data-for="item in items">
    <li data-bind="item.name">Item</li>
</ul>
```

### Server Communication

```typescript
// In controller.ts
export function loadData(): void {
    rune.send_to_server({ type: "load" });
}

// In service.ts
export function on_client_event(event: object): void {
    if (event.type === "load") {
        const data = rune.db_query("SELECT * FROM items");
        rune.set_data("items", data.rows);
    }
}
```

## Next Steps

- Read [HTML Attributes](./waid/html-attributes.md) for full attribute reference
- Read [TypeScript Support](./waid/typescript.md) for what compiles to WASM
- Read [Client Host Functions](./client/host-functions.md) for available APIs
- Check [Status & Roadmap](./status.md) for what's implemented
