# TypeScript Support

WAID compiles a subset of TypeScript to WebAssembly. This page documents what's supported and what's not.

## Implementation Status

| Feature | Status | Notes |
|---------|--------|-------|
| **Types** | | |
| `number` | ✅ | Maps to i32 (integers) or f64 (floats) |
| `boolean` | ✅ | Maps to i32 (0/1) |
| `string` | ✅ | Pointer + length |
| `void` | ✅ | No return value |
| `any` | ❌ | Not supported |
| `object` | 🔨 | Limited (JSON serialization) |
| `array` | 🔨 | Limited |
| **Variables** | | |
| `let` | ✅ | Mutable local |
| `const` | ✅ | Immutable local |
| `var` | ✅ | Treated as `let` |
| **Operators** | | |
| Arithmetic (`+`, `-`, `*`, `/`, `%`) | ✅ | |
| Comparison (`==`, `!=`, `<`, `>`, `<=`, `>=`) | ✅ | |
| Logical (`&&`, `\|\|`, `!`) | ✅ | |
| Bitwise (`&`, `\|`, `^`, `~`, `<<`, `>>`) | ✅ | |
| Assignment (`=`, `+=`, `-=`, `*=`, `/=`) | ✅ | |
| **Control Flow** | | |
| `if`/`else` | ✅ | |
| `while` | ✅ | |
| `for` | ✅ | C-style for loops |
| `break` | ✅ | |
| `continue` | ✅ | |
| `return` | ✅ | |
| `switch` | ❌ | Use if/else |
| `for...of` | ❌ | Use index-based for |
| `for...in` | ❌ | Not supported |
| **Functions** | | |
| Function declarations | ✅ | |
| Parameters | ✅ | Typed parameters |
| Return values | ✅ | |
| `export` | ✅ | Creates WASM export |
| Arrow functions | ❌ | Use function declarations |
| Default parameters | ❌ | Not yet |
| Rest parameters | ❌ | Not supported |
| **Classes** | | |
| Class declarations | ❌ | Not yet |
| Methods | ❌ | Not yet |
| Constructors | ❌ | Not yet |
| **Other** | | |
| Imports | 🔨 | Only `rune.*` host functions |
| Async/await | ❌ | Use callbacks |
| Generators | ❌ | Not supported |
| Decorators | ❌ | Not supported |

**Legend:** ✅ Supported | 🔨 Partial | ❌ Not supported

---

## Supported Patterns

### Basic Function

```typescript
export function add(a: number, b: number): number {
    return a + b;
}
```

### Variables and Arithmetic

```typescript
export function calculate(): number {
    let x = 10;
    let y = 20;
    const sum = x + y;
    x += 5;
    return sum * 2;
}
```

### Control Flow

```typescript
export function max(a: number, b: number): number {
    if (a > b) {
        return a;
    } else {
        return b;
    }
}

export function sum(n: number): number {
    let total = 0;
    for (let i = 0; i < n; i = i + 1) {
        total = total + i;
    }
    return total;
}

export function factorial(n: number): number {
    let result = 1;
    while (n > 1) {
        result = result * n;
        n = n - 1;
    }
    return result;
}
```

### Host Function Calls

```typescript
export function greet(name: string): void {
    rune.log_info("Hello, " + name);
}

export function onClick(): void {
    rune.update_data("counter", { value: 42 });
}

export function getTime(): number {
    return rune.get_time_ms();
}
```

### Comparison and Logical Operators

```typescript
export function validate(x: number, y: number): boolean {
    return x > 0 && y > 0 && x != y;
}

export function classify(value: number): number {
    if (value < 0) {
        return -1;
    } else if (value == 0) {
        return 0;
    } else {
        return 1;
    }
}
```

---

## Type Mapping

| TypeScript | WASM Type | Notes |
|------------|-----------|-------|
| `number` | `i32` or `f64` | Integers use i32, floats use f64 |
| `boolean` | `i32` | 0 = false, 1 = true |
| `string` | `(i32, i32)` | Pointer and length |
| `void` | none | No return |

---

## Strings

Strings are stored in WASM linear memory and passed as (pointer, length) pairs:

```typescript
// This string literal:
rune.log_info("Hello");

// Becomes:
// 1. String "Hello" stored at memory offset (e.g., 0x400)
// 2. Call: log_info(0x400, 5)
```

String concatenation creates new strings in memory:

```typescript
const greeting = "Hello, " + name + "!";
```

---

## Limitations

### No Classes (Yet)

```typescript
// NOT SUPPORTED
class Counter {
    count: number = 0;
    inc() { this.count++; }
}

// USE INSTEAD: Functions with state via host
export function inc(): void {
    const count = rune.get_local_state("count") ?? 0;
    rune.set_local_state("count", count + 1);
    rune.update_data("counter", { value: count + 1 });
}
```

### No Arrow Functions

```typescript
// NOT SUPPORTED
const add = (a, b) => a + b;

// USE INSTEAD
function add(a: number, b: number): number {
    return a + b;
}
```

### No Async/Await

```typescript
// NOT SUPPORTED
async function fetchData() {
    const response = await fetch(url);
    return response.json();
}

// USE INSTEAD: Callback pattern via host
export function fetchData(): void {
    rune.send_to_server({ type: "fetch", url: url });
}

export function on_server_response(response: object): void {
    rune.update_data("data", response);
}
```

### No Complex Objects

```typescript
// NOT SUPPORTED (complex nested objects)
const user = {
    name: "Alice",
    address: {
        city: "NYC",
        zip: 10001
    }
};

// USE INSTEAD: JSON through host functions
rune.update_data("user", {
    name: "Alice",
    city: "NYC"
});
```

---

## Best Practices

1. **Keep it simple**: WASM is best for compute, not complex data structures
2. **Use host functions for I/O**: Don't try to do HTTP/DB in pure TS
3. **Prefer numbers and booleans**: Strings and objects have overhead
4. **Avoid deep nesting**: Flat code compiles better
5. **Export what's needed**: Only export functions that need to be called externally
