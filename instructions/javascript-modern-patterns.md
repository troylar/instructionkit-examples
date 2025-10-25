# Modern JavaScript Patterns

Contemporary JavaScript patterns using ES6+ features including async/await, destructuring, modules, and functional programming. These guidelines help AI assistants generate clean, modern JavaScript code.

## Core Guidelines

### 1. Use const and let, Never var

Declare variables with `const` by default. Use `let` only when reassignment is needed. Never use `var` due to function-scoping issues.

**Example**:
```javascript
// Good
const API_URL = 'https://api.example.com';
let counter = 0;
counter += 1;

// Avoid
var API_URL = 'https://api.example.com';
var counter = 0;
```

### 2. Prefer async/await Over Promises

Use async/await for asynchronous code instead of `.then()` chains. Handle errors with try-catch blocks. Always await promises.

**Example**:
```javascript
// Good
async function fetchUser(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        const user = await response.json();
        return user;
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}

// Avoid
function fetchUser(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(user => user)
        .catch(error => {
            console.error('Failed to fetch user:', error);
            throw error;
        });
}
```

### 3. Use Destructuring for Objects and Arrays

Destructure function parameters, objects, and arrays to extract values clearly. Use default values in destructuring when appropriate.

**Example**:
```javascript
// Good
function createUser({ name, email, role = 'user' }) {
    return { name, email, role, createdAt: new Date() };
}

const [first, second, ...rest] = items;
const { id, name } = user;

// Avoid
function createUser(options) {
    const name = options.name;
    const email = options.email;
    const role = options.role || 'user';
    return { name, email, role, createdAt: new Date() };
}
```

### 4. Use Arrow Functions Appropriately

Prefer arrow functions for callbacks and functional operations. Use regular functions for methods that need their own `this` context.

**Example**:
```javascript
// Good
const doubled = numbers.map(n => n * 2);
const filtered = users.filter(user => user.isActive);

setTimeout(() => console.log('Done'), 1000);

// Good (method needs this)
const obj = {
    name: 'Example',
    greet() {
        return `Hello, ${this.name}`;
    }
};

// Avoid (unnecessary function keyword)
const doubled = numbers.map(function(n) { return n * 2; });
```

### 5. Use Template Literals for String Interpolation

Use template literals with backticks for string interpolation instead of concatenation. Supports multi-line strings naturally.

**Example**:
```javascript
// Good
const message = `User ${user.name} (${user.email}) logged in`;
const html = `
    <div class="user">
        <h2>${user.name}</h2>
        <p>${user.email}</p>
    </div>
`;

// Avoid
const message = 'User ' + user.name + ' (' + user.email + ') logged in';
const html = '<div class="user">' +
    '<h2>' + user.name + '</h2>' +
    '<p>' + user.email + '</p>' +
    '</div>';
```

### 6. Use ES Modules for Imports

Use `import`/`export` syntax for modules. Prefer named exports over default exports for better refactoring support.

**Example**:
```javascript
// Good (named exports)
// utils.js
export function formatDate(date) { /* ... */ }
export function parseJSON(str) { /* ... */ }

// app.js
import { formatDate, parseJSON } from './utils.js';

// Avoid (CommonJS)
const { formatDate, parseJSON } = require('./utils.js');
module.exports = { formatDate, parseJSON };
```

### 7. Use Functional Array Methods

Use `map`, `filter`, `reduce`, `find`, `some`, and `every` instead of loops when transforming or querying arrays. Chain methods for clarity.

**Example**:
```javascript
// Good
const activeUserNames = users
    .filter(user => user.isActive)
    .map(user => user.name)
    .sort();

const total = items.reduce((sum, item) => sum + item.price, 0);

// Avoid
const activeUserNames = [];
for (let i = 0; i < users.length; i++) {
    if (users[i].isActive) {
        activeUserNames.push(users[i].name);
    }
}
activeUserNames.sort();
```

## Quick Reference

- [ ] Variables declared with `const` or `let`, never `var`
- [ ] Async operations use `async`/`await` with try-catch
- [ ] Objects and arrays destructured to extract values
- [ ] Arrow functions used for callbacks, regular functions for methods
- [ ] Template literals used for string interpolation
- [ ] ES modules (`import`/`export`) used instead of CommonJS
- [ ] Array transformations use functional methods (map, filter, reduce)
