# **React Hooks - From Basic to Advanced**

## **🏗️ Foundation: Why Hooks Exist**

### **Concept (70%)**
Before Hooks (2019), React had a **split personality**: functional components were "dumb" (only UI) and class components were "smart" (stateful logic). This created:

1. **Wrapper hell** - nested HOCs (Higher-Order Components)
2. **Giant components** - unrelated logic mixed together
3. **Complex lifecycle** - confusing `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`
4. **Learning curve** - `this` binding, method binding patterns

**Hooks solve this by:**
- Letting you **use state and other React features in functional components**
- **Extracting and reusing stateful logic** without changing component hierarchy
- **Splitting one component into smaller functions** based on related pieces
- **Sharing non-visual logic** between components

---

## **📦 Basic Hooks (The Essential Three)**

### **1. useState - Component Memory**
#### **Concept (70%)**
`useState` gives functional components **persistent local state**. Think of it as a **"memory cell"** for your component that persists across re-renders.

**Key characteristics:**
- Returns a `[state, setState]` pair
- **State updates trigger re-renders**
- State is **isolated to each component instance**
- Updates are **asynchronous** (batched for performance)
- Can hold any value: primitives, objects, arrays

#### **Code (30%)**
```jsx
function Counter() {
  // Basic state
  const [count, setCount] = useState(0);
  
  // Functional updates for complex state
  const handleIncrement = () => {
    setCount(prevCount => prevCount + 1);
  };
  
  // Object state
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  // Updating object state (preserve other properties)
  const updateName = (name) => {
    setUser(prev => ({ ...prev, name }));
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

---

### **2. useEffect - Side Effects Manager**
#### **Concept (70%)**
`useEffect` handles **side effects** - operations that interact with the outside world. It unifies three class lifecycle methods into one API.

**The three roles of useEffect:**
1. **Mounting** (`componentDidMount`) - Run once when component mounts
2. **Updating** (`componentDidUpdate`) - Run when dependencies change
3. **Cleanup** (`componentWillUnmount`) - Clean up before unmounting

**Dependency array patterns:**
- `[]` = **Mount only** (run once)
- `[dep1, dep2]` = **Run when dependencies change**
- **No array** = **Run after every render** (use cautiously!)

#### **Code (30%)**
```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Pattern 1: Data fetching with cleanup
  useEffect(() => {
    let isMounted = true; // Cleanup flag
    
    const fetchUser = async () => {
      try {
        const data = await fetch(`/api/users/${userId}`);
        if (isMounted) {
          setUser(await data.json());
          setLoading(false);
        }
      } catch (error) {
        if (isMounted) {
          console.error('Failed to fetch user:', error);
        }
      }
    };
    
    fetchUser();
    
    // Cleanup function
    return () => {
      isMounted = false;
      // Cancel any pending requests here
    };
  }, [userId]); // Re-run when userId changes
  
  // Pattern 2: Event listeners
  useEffect(() => {
    const handleResize = () => {
      console.log('Window resized:', window.innerWidth);
    };
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty array = mount/unmount only
  
  // Pattern 3: Document title update
  useEffect(() => {
    document.title = user ? `${user.name}'s Profile` : 'Loading...';
  }, [user]); // Update when user changes
  
  if (loading) return <div>Loading...</div>;
  return <div>{user.name}'s Profile</div>;
}
```

---

### **3. useContext - Global State without Prop Drilling**
#### **Concept (70%)**
`useContext` provides a way to **share data deep in the component tree** without passing props manually at every level (prop drilling).

**When to use useContext:**
- **Theme settings** (dark/light mode)
- **User authentication** state
- **Language/localization** preferences
- **Global UI state** (modals, notifications)
- **Feature flags**

**Architecture pattern:**
```
ThemeContext.Provider (value={theme})
  ├── App
  │   ├── Navbar (consumes theme)
  │   └── Dashboard
  │       └── Card (consumes theme) ✅ No prop drilling!
```

#### **Code (30%)**
```jsx
// 1. Create context
const ThemeContext = createContext('light'); // Default value

// 2. Provider component
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <MainContent />
      <Footer />
    </ThemeContext.Provider>
  );
}

// 3. Consumer components at any depth
function Header() {
  // Access context value
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <header className={`header-${theme}`}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </header>
  );
}

// 4. Custom hook pattern for better abstraction
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage with custom hook
function Card() {
  const { theme } = useTheme();
  return <div className={`card-${theme}`}>Card content</div>;
}
```

---

## **⚡ Advanced Hooks (Performance & Complex State)**

### **4. useReducer - Complex State Logic**
#### **Concept (70%)**
`useReducer` is like `useState` but for **complex state logic**. It's inspired by Redux pattern and follows the **flux architecture**.

**When to use useReducer over useState:**
- State has **multiple sub-values**
- Next state depends on **previous state** 
- **Complex state transitions** (like forms, multi-step wizards)
- **Shared state logic** between components
- State changes follow a **predictable pattern**

**Redux pattern analogy:**
```
Action → Reducer → New State
```

#### **Code (30%)**
```jsx
// Reducer function (pure function)
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { 
        id: Date.now(), 
        text: action.text, 
        completed: false 
      }];
      
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id 
          ? { ...todo, completed: !todo.completed } 
          : todo
      );
      
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.id);
      
    default:
      return state;
  }
}

function TodoApp() {
  // useReducer returns [state, dispatch]
  const [todos, dispatch] = useReducer(todoReducer, []);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', text: input });
      setInput('');
    }
  };
  
  return (
    <div>
      <input 
        value={input}
        onChange={e => setInput(e.target.value)}
        placeholder="Add todo..."
      />
      <button onClick={addTodo}>Add</button>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => 
              dispatch({ type: 'TOGGLE_TODO', id: todo.id })
            }>
              Toggle
            </button>
            <button onClick={() => 
              dispatch({ type: 'DELETE_TODO', id: todo.id })
            }>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### **5. useMemo - Expensive Calculations Cache**
#### **Concept (70%)**
`useMemo` **memoizes expensive calculations** to avoid re-computing on every render. It's like a **"cache"** for computed values.

**When to use useMemo:**
- **Expensive computations** (sorting large arrays, complex math)
- **Referential equality** for objects/arrays passed as props
- **Derived state** that depends on other state/props
- **Avoiding unnecessary re-renders** of child components

**Important:** Don't overuse! Only for measurable performance bottlenecks.

#### **Code (30%)**
```jsx
function ExpensiveComponent({ items, filter }) {
  // BAD: Recomputes on every render
  // const filteredItems = items.filter(item => item.includes(filter));
  
  // GOOD: Memoizes the result
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => item.includes(filter));
  }, [items, filter]); // Only recompute when items or filter changes
  
  // Another example: Expensive calculation
  const stats = useMemo(() => {
    return {
      average: filteredItems.reduce((a, b) => a + b.value, 0) / filteredItems.length,
      sum: filteredItems.reduce((a, b) => a + b.value, 0),
      count: filteredItems.length
    };
  }, [filteredItems]);
  
  return (
    <div>
      <p>Average: {stats.average}</p>
      <ChildComponent data={filteredItems} />
    </div>
  );
}

// Without useMemo, ChildComponent re-renders every time
// With useMemo, it only re-renders when filteredItems actually changes
const ChildComponent = React.memo(function({ data }) {
  return <div>Items: {data.length}</div>;
});
```

---

### **6. useCallback - Function Identity Cache**
#### **Concept (70%)**
`useCallback` **memoizes functions** to maintain referential equality across renders. It prevents unnecessary re-renders of child components that depend on function props.

**The function identity problem:**
```jsx
// Every render creates NEW function
const handleClick = () => console.log('Clicked');
// handleClick !== handleClick between renders
```

**When to use useCallback:**
- Functions passed as props to **memoized child components**
- Functions used in **dependency arrays** of other hooks
- **Event handlers** in lists (to avoid re-rendering all items)

#### **Code (30%)**
```jsx
function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  // BAD: New function on every render
  // const addItem = () => {
  //   setItems([...items, `Item ${items.length}`]);
  // };
  
  // GOOD: Memoized function
  const addItem = useCallback(() => {
    setItems(prevItems => [...prevItems, `Item ${prevItems.length}`]);
  }, []); // No dependencies, same function always
  
  // With dependency
  const logCount = useCallback(() => {
    console.log('Current count:', count);
  }, [count]); // New function when count changes
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      
      {/* Child won't re-render unnecessarily */}
      <ItemList items={items} onAddItem={addItem} />
      
      {/* Re-renders when count changes (dependency) */}
      <Logger onLog={logCount} />
    </div>
  );
}

// React.memo: Only re-renders when props change
const ItemList = React.memo(function({ items, onAddItem }) {
  console.log('ItemList rendering');
  
  return (
    <div>
      <button onClick={onAddItem}>Add Item</button>
      <ul>
        {items.map((item, i) => <li key={i}>{item}</li>)}
      </ul>
    </div>
  );
});
```

---

### **7. useRef - Mutable Reference Container**
#### **Concept (70%)**
`useRef` creates a **mutable reference** that persists across renders but doesn't trigger re-renders when changed. Think of it as a **"box"** that can hold any value.

**Three main use cases:**
1. **Accessing DOM elements** directly (like `document.getElementById`)
2. **Storing mutable values** that shouldn't trigger re-renders
3. **Keeping previous values** across renders

**Key difference from state:** Changing ref.current **does not cause re-render**.

#### **Code (30%)**
```jsx
function RefExamples() {
  // 1. DOM Reference
  const inputRef = useRef(null);
  
  // 2. Mutable value (counter that doesn't trigger re-render)
  const renderCount = useRef(0);
  const intervalRef = useRef(null);
  
  // 3. Previous value storage
  const [name, setName] = useState('');
  const previousName = useRef('');
  
  // Focus input on mount
  useEffect(() => {
    inputRef.current?.focus();
    renderCount.current += 1;
    
    // Store interval ID for cleanup
    intervalRef.current = setInterval(() => {
      console.log('Interval running...');
    }, 1000);
    
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);
  
  // Track previous name
  useEffect(() => {
    previousName.current = name;
  }, [name]);
  
  const handleSubmit = () => {
    console.log('Input value:', inputRef.current.value);
    inputRef.current.focus();
  };
  
  return (
    <div>
      <p>Renders: {renderCount.current}</p>
      <p>Current name: {name}</p>
      <p>Previous name: {previousName.current}</p>
      
      <input 
        ref={inputRef}
        value={name}
        onChange={e => setName(e.target.value)}
        placeholder="Type something..."
      />
      
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

---

## **🎯 Advanced Patterns & Composition**

### **Custom Hooks: Composing Multiple Hooks**
```jsx
// Custom hook combining useState, useEffect, useRef
function useLocalStorage(key, initialValue) {
  // State to store our value
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Return a wrapped version of useState's setter
  const setValue = useCallback((value) => {
    try {
      // Allow value to be a function (like useState)
      const valueToStore = 
        value instanceof Function ? value(storedValue) : value;
      
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue];
}

// Using the custom hook
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [volume, setVolume] = useLocalStorage('volume', 50);
  
  return (
    <div>
      <select value={theme} onChange={e => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      
      <input 
        type="range" 
        min="0" 
        max="100" 
        value={volume}
        onChange={e => setVolume(Number(e.target.value))}
      />
    </div>
  );
}
```

---

## **📊 Hook Rules & Best Practices**

### **Rules of Hooks:**
1. **Only call hooks at the top level** (not in loops, conditions, or nested functions)
2. **Only call hooks from React functions** (components or custom hooks)
3. **Custom hooks must start with `use`** (so React can identify them)

### **Performance Patterns:**

| Hook | When to Use | Performance Impact |
|------|------------|-------------------|
| `useMemo` | Expensive calculations, derived data | ✅ Reduces computation |
| `useCallback` | Function props, dependency arrays | ✅ Prevents re-renders |
| `useRef` | DOM access, mutable values | ⚡ No re-renders |
| `useReducer` | Complex state logic | ⚖️ Predictable updates |

### **Advanced Hook Flow:**
```jsx
function AdvancedComponent() {
  // 1. State
  const [data, setData] = useState([]);
  const [filter, setFilter] = useState('');
  
  // 2. Refs
  const timeoutRef = useRef();
  const previousFilter = useRef('');
  
  // 3. Memoized values
  const filteredData = useMemo(() => {
    return data.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [data, filter]);
  
  // 4. Memoized callbacks
  const fetchData = useCallback(async () => {
    const response = await fetch('/api/data');
    setData(await response.json());
  }, []);
  
  // 5. Effects with cleanup
  useEffect(() => {
    // Debounced filter
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      if (filter !== previousFilter.current) {
        console.log('Filter changed to:', filter);
        previousFilter.current = filter;
      }
    }, 300);
    
    return () => {
      clearTimeout(timeoutRef.current);
    };
  }, [filter]);
  
  // 6. Initial data fetch
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  // 7. Context
  const { theme } = useContext(ThemeContext);
  
  return (
    <div className={`component-${theme}`}>
      {/* Component JSX */}
    </div>
  );
}
```

---

## **🎓 Key Takeaways:**

1. **Start with basics**: `useState`, `useEffect`, `useContext`
2. **Upgrade when needed**: `useReducer` for complex state, `useMemo/useCallback` for performance
3. **Use `useRef` for**: DOM access, mutable values, previous values
4. **Create custom hooks** to extract and reuse logic
5. **Follow hook rules** and understand dependency arrays
6. **Profile before optimizing** - don't prematurely use `useMemo/useCallback`

**Remember:** Hooks are tools in your toolbox. Choose the right one for the job, and compose them to build powerful, maintainable React applications!