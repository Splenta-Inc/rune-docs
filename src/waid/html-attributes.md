# HTML Attributes

WAID extends standard HTML with `data-*` attributes for bindings, events, control flow, and routing.

## Attribute Reference

| Attribute | Description | Example |
|-----------|-------------|---------|
| `data-component` | Define component boundary | `<div data-component="counter">` |
| `data-controller` | Link to TypeScript controller | `data-controller="./controller.ts"` |
| `data-bind` | Bind text content to state | `data-bind="count"` |
| `data-bind-*` | Bind attribute to state | `data-bind-class="theme"` |
| `data-on*` | Event handlers | `data-onclick="inc"` |
| `data-if` | Conditional rendering | `data-if="isLoggedIn"` |
| `data-for` | List iteration | `data-for="item in items"` |
| `data-route` | Define route | `data-route="/users/:id"` |
| `data-link` | Navigation link | `data-link="/settings"` |
| `id` | Node identifier (for mutations) | `id="counter-value"` |

---

## Components

Define component boundaries with `data-component`:

```html
<div data-component="counter" data-controller="./controller.ts">
    <span data-bind="count">0</span>
    <button data-onclick="inc">+</button>
</div>
```

Components:
- Have their own controller
- Define a scope for bindings
- Can be nested

---

## Controllers

Link a TypeScript controller to a component:

```html
<div data-component="app" data-controller="./controller.ts">
    ...
</div>
```

The controller exports functions that can be called from events:

```typescript
// controller.ts
export function inc(): void {
    rune.update_data("count", { value: currentValue + 1 });
}

export function dec(): void {
    rune.update_data("count", { value: currentValue - 1 });
}
```

---

## Data Binding

### Text Content

Bind element text to state:

```html
<span data-bind="username">Guest</span>
```

The initial text ("Guest") is the default value. When state updates, the text changes.

### Attribute Binding

Bind any attribute using `data-bind-<attribute>`:

```html
<!-- Bind class -->
<div data-bind-class="theme">light</div>

<!-- Bind src -->
<img data-bind-src="avatarUrl" src="/default.png">

<!-- Bind href -->
<a data-bind-href="profileUrl" href="#">Profile</a>

<!-- Bind disabled -->
<button data-bind-disabled="isLoading">Submit</button>
```

### Nested Bindings

Use dot notation for nested state:

```html
<span data-bind="user.name">Unknown</span>
<span data-bind="user.address.city">N/A</span>
```

---

## Events

Handle events with `data-on<event>`:

```html
<!-- Click -->
<button data-onclick="handleClick">Click me</button>

<!-- Input -->
<input data-oninput="handleInput" type="text">

<!-- Change -->
<select data-onchange="handleSelect">
    <option>A</option>
    <option>B</option>
</select>

<!-- Submit -->
<form data-onsubmit="handleSubmit">
    ...
</form>

<!-- Key events -->
<input data-onkeydown="handleKey" data-onkeyup="handleKeyUp">

<!-- Focus/blur -->
<input data-onfocus="handleFocus" data-onblur="handleBlur">
```

### Event Parameters

Pass parameters to handlers:

```html
<button data-onclick="inc(5)">+5</button>
<button data-onclick="setPage('settings')">Settings</button>
```

In controller:
```typescript
export function inc(step: number): void {
    // step = 5
}

export function setPage(page: string): void {
    // page = "settings"
}
```

### Event Object

Access the event object via `$event`:

```html
<input data-oninput="updateValue($event.target.value)">
```

---

## Conditional Rendering

Show/hide elements based on state:

```html
<div data-if="isLoggedIn">
    Welcome back!
</div>

<div data-if="!isLoggedIn">
    Please log in.
</div>
```

Supports:
- Boolean state: `data-if="isActive"`
- Negation: `data-if="!isActive"`
- Comparison: `data-if="count > 0"`

---

## List Rendering

Iterate over arrays:

```html
<ul data-for="item in items">
    <li data-bind="item.name">Name</li>
</ul>
```

### With Index

```html
<ul data-for="(item, index) in items">
    <li>
        <span data-bind="index">0</span>:
        <span data-bind="item.name">Name</span>
    </li>
</ul>
```

### Nested Loops

```html
<div data-for="category in categories">
    <h2 data-bind="category.name">Category</h2>
    <ul data-for="product in category.products">
        <li data-bind="product.name">Product</li>
    </ul>
</div>
```

---

## Routing

### Define Routes

Use `data-route` in `index.html`:

```html
<!-- app/pages/index.html -->
<html>
<body>
    <nav>
        <a data-link="/">Home</a>
        <a data-link="/settings">Settings</a>
        <a data-link="/users/123">User 123</a>
    </nav>

    <main data-route="/">
        <div data-component="home" data-controller="./home/controller.ts">
            ...
        </div>
    </main>

    <main data-route="/settings">
        <div data-component="settings" data-controller="./settings/controller.ts">
            ...
        </div>
    </main>

    <main data-route="/users/:id">
        <div data-component="user" data-controller="./user/controller.ts">
            ...
        </div>
    </main>
</body>
</html>
```

### Route Parameters

Access route params in controller:

```html
<main data-route="/users/:id">
    <span data-bind="userId">0</span>
</main>
```

```typescript
// controller.ts
export function start(params: object): void {
    const userId = params.id;
    rune.update_data("userId", { value: userId });
}
```

### Navigation Links

```html
<!-- Static link -->
<a data-link="/settings">Settings</a>

<!-- Dynamic link -->
<a data-bind-href="'/users/' + userId">View Profile</a>

<!-- Programmatic navigation -->
<button data-onclick="goToSettings">Settings</button>
```

```typescript
export function goToSettings(): void {
    rune.navigate("settings");
}
```

---

## Node IDs

Use `id` to identify nodes for mutations:

```html
<div id="user-card">
    <span id="user-name" data-bind="name">Guest</span>
    <span id="user-email" data-bind="email">none</span>
</div>
```

```typescript
// Update specific nodes
rune.update_data("user-name", { text: "Alice" });
rune.update_data("user-email", { text: "alice@example.com" });
```

---

## Complete Example

```html
<!DOCTYPE html>
<html>
<head>
    <title>Todo App</title>
</head>
<body>
    <div data-component="todo-app" data-controller="./controller.ts">
        <h1>Todos</h1>

        <!-- Input form -->
        <form data-onsubmit="addTodo">
            <input id="new-todo" type="text" placeholder="What needs to be done?">
            <button type="submit">Add</button>
        </form>

        <!-- Filter tabs -->
        <div>
            <button data-onclick="setFilter('all')" data-bind-class="filter == 'all' ? 'active' : ''">All</button>
            <button data-onclick="setFilter('active')" data-bind-class="filter == 'active' ? 'active' : ''">Active</button>
            <button data-onclick="setFilter('done')" data-bind-class="filter == 'done' ? 'active' : ''">Done</button>
        </div>

        <!-- Todo list -->
        <ul data-for="todo in filteredTodos">
            <li>
                <input type="checkbox"
                       data-bind-checked="todo.done"
                       data-onchange="toggleTodo(todo.id)">
                <span data-bind="todo.text" data-bind-class="todo.done ? 'done' : ''">Task</span>
                <button data-onclick="deleteTodo(todo.id)">×</button>
            </li>
        </ul>

        <!-- Footer -->
        <div data-if="todos.length > 0">
            <span data-bind="activeCount">0</span> items left
            <button data-if="doneCount > 0" data-onclick="clearDone">Clear completed</button>
        </div>
    </div>
</body>
</html>
```
