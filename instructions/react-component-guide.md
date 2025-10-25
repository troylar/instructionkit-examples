# React Component Guide

Modern React development using functional components and hooks. These guidelines help AI assistants generate clean, performant React components following current best practices with useState, useEffect, custom hooks, and proper composition patterns.

## Core Guidelines

### 1. Use Functional Components with Hooks

Always use functional components instead of class components. Leverage hooks (useState, useEffect, etc.) for state and side effects. Hooks provide cleaner, more reusable code.

**Example**:
```javascript
// Good
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetchUser(userId).then(data => {
            setUser(data);
            setLoading(false);
        });
    }, [userId]);

    if (loading) return <div>Loading...</div>;
    return <div>{user.name}</div>;
}

// Avoid (class component - legacy)
class UserProfile extends React.Component {
    constructor(props) {
        super(props);
        this.state = { user: null, loading: true };
    }
    // ... class methods
}
```

### 2. Destructure Props and State

Destructure props in function parameters and state values when using. This improves readability and makes dependencies explicit.

**Example**:
```javascript
// Good
function UserCard({ name, email, avatar, onEdit }) {
    const [isEditing, setIsEditing] = useState(false);

    return (
        <div className="user-card">
            <img src={avatar} alt={name} />
            <h2>{name}</h2>
            <p>{email}</p>
            <button onClick={() => setIsEditing(true)}>Edit</button>
        </div>
    );
}

// Avoid
function UserCard(props) {
    return (
        <div className="user-card">
            <h2>{props.name}</h2>
            <p>{props.email}</p>
        </div>
    );
}
```

### 3. Use useEffect with Proper Dependencies

Include all external values used in useEffect in the dependency array. This prevents stale closures and ensures effects run when dependencies change.

**Example**:
```javascript
// Good
function SearchResults({ query, filters }) {
    const [results, setResults] = useState([]);

    useEffect(() => {
        const fetchResults = async () => {
            const data = await searchAPI(query, filters);
            setResults(data);
        };
        fetchResults();
    }, [query, filters]); // All dependencies listed

    return <ResultsList results={results} />;
}

// Avoid (missing dependencies)
function SearchResults({ query, filters }) {
    const [results, setResults] = useState([]);

    useEffect(() => {
        searchAPI(query, filters).then(setResults);
    }, [query]); // Missing 'filters' - will use stale value!

    return <ResultsList results={results} />;
}
```

### 4. Extract Custom Hooks for Reusable Logic

Move reusable stateful logic into custom hooks. Name them with `use` prefix. Custom hooks keep components clean and logic testable.

**Example**:
```javascript
// Good (custom hook)
function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        fetch(url)
            .then(res => res.json())
            .then(setData)
            .catch(setError)
            .finally(() => setLoading(false));
    }, [url]);

    return { data, loading, error };
}

// Usage in component
function UserList() {
    const { data: users, loading, error } = useFetch('/api/users');

    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <div>{users.map(u => <UserCard key={u.id} {...u} />)}</div>;
}

// Avoid (logic mixed in component)
function UserList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);
    // ... fetch logic repeated in every component
}
```

### 5. Memoize Expensive Calculations with useMemo

Use useMemo for expensive computations that don't need to run on every render. Only recompute when dependencies change.

**Example**:
```javascript
// Good
function DataTable({ items, filters }) {
    const filteredItems = useMemo(() => {
        return items.filter(item =>
            Object.entries(filters).every(([key, value]) =>
                item[key] === value
            )
        );
    }, [items, filters]); // Only recompute when these change

    return <Table data={filteredItems} />;
}

// Avoid (expensive filter runs on every render)
function DataTable({ items, filters }) {
    const filteredItems = items.filter(item =>
        Object.entries(filters).every(([key, value]) =>
            item[key] === value
        )
    ); // Runs even when unrelated state changes!

    return <Table data={filteredItems} />;
}
```

### 6. Use useCallback for Stable Function References

Wrap callback functions in useCallback when passing to child components or using in dependencies. Prevents unnecessary re-renders of child components.

**Example**:
```javascript
// Good
function ParentComponent() {
    const [count, setCount] = useState(0);

    const handleIncrement = useCallback(() => {
        setCount(c => c + 1);
    }, []); // Stable reference

    return (
        <div>
            <p>Count: {count}</p>
            <ExpensiveChildComponent onIncrement={handleIncrement} />
        </div>
    );
}

// Avoid (new function on every render)
function ParentComponent() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <ExpensiveChildComponent onIncrement={() => setCount(count + 1)} />
            {/* Child re-renders even if count didn't change! */}
        </div>
    );
}
```

### 7. Compose Components Instead of Prop Drilling

Use component composition and children prop to avoid passing props through multiple levels. Keeps components flexible and reduces coupling.

**Example**:
```javascript
// Good (composition)
function Layout({ children, sidebar }) {
    return (
        <div className="layout">
            <aside>{sidebar}</aside>
            <main>{children}</main>
        </div>
    );
}

function App() {
    return (
        <Layout sidebar={<UserMenu user={currentUser} />}>
            <Dashboard user={currentUser} />
        </Layout>
    );
}

// Avoid (prop drilling)
function Layout({ user }) {
    return (
        <div className="layout">
            <Sidebar user={user} />
            <Main user={user} />
        </div>
    );
}

function Sidebar({ user }) {
    return <UserMenu user={user} />; // Just passing through
}
```

## Quick Reference

- [ ] Components are functional, not classes
- [ ] Props destructured in function parameters
- [ ] useState for component state management
- [ ] useEffect includes all dependencies in array
- [ ] Reusable logic extracted to custom hooks (use* prefix)
- [ ] Expensive calculations wrapped in useMemo
- [ ] Callbacks wrapped in useCallback when passed to children
- [ ] Component composition preferred over prop drilling
