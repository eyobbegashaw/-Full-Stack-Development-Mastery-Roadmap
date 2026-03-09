# **🔄 React Rendering - From Basic to Advanced**

## **🎯 Foundation: The React Rendering Philosophy**

### **Concept (70%)**
React's rendering is **declarative, not imperative**. You describe **WHAT** the UI should look like based on props and state, and React figures out **HOW** to update the DOM efficiently. Think of it like giving a recipe (JSX) rather than step-by-step cooking instructions.

**Key rendering concepts:**
1. **Virtual DOM** - JavaScript representation of real DOM
2. **Reconciliation** - Diffing algorithm comparing virtual DOMs
3. **Batched Updates** - Multiple state updates in one render cycle
4. **Render Phases** - Render (computes changes) → Commit (applies to DOM)

---

## **1️⃣ Component Lifecycle**

### **Concept (70%)**
Components have a **lifecycle** - they're born (mount), live (update), and die (unmount). Understanding this helps you run code at the right time.

**Class Component Lifecycle (Traditional):**
```
Mounting: constructor → render → componentDidMount
Updating: shouldComponentUpdate → render → componentDidUpdate
Unmounting: componentWillUnmount
```

**Function Component Lifecycle (Modern with Hooks):**
```
Mounting: useState/useReducer → render → useEffect (with [])
Updating: useState/useReducer setters → render → useEffect (with deps)
Unmounting: useEffect cleanup function
```

**Key Moments:**
- **Initial Render**: Fetch data, setup subscriptions
- **Updates**: Respond to prop/state changes
- **Cleanup**: Remove event listeners, cancel API calls

### **Code (30%)**
```jsx
// Functional Component Lifecycle with Hooks
function UserProfile({ userId }) {
  // 1. MOUNT: useState initializes
  const [user, setUser] = useState(null);
  
  // 2. MOUNT: useEffect runs after render
  useEffect(() => {
    console.log('Component mounted');
    fetchUser(userId).then(setUser);
    
    // 5. UNMOUNT: Cleanup function
    return () => {
      console.log('Component unmounting');
      // Cancel subscriptions, timers, etc.
    };
  }, []); // Empty array = mount only
  
  // 3. UPDATE: useEffect with dependencies
  useEffect(() => {
    if (userId) {
      console.log('UserId changed:', userId);
      fetchUser(userId).then(setUser);
    }
  }, [userId]); // Runs when userId changes
  
  // 4. RENDER: JSX returned
  console.log('Rendering UserProfile');
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}

// Lifecycle visualization:
// Mount: [useState] → [render] → [useEffect]
// Update: [setState] → [render] → [useEffect with deps]
// Unmount: [cleanup function]
```

---

## **2️⃣ Lists and Keys**

### **Concept (70%)**
When rendering lists, React needs a way to identify which items have changed, been added, or removed. **Keys** are React's ID system for list items.

**Why Keys Matter:**
- **Performance**: Helps React identify minimal DOM operations
- **State Preservation**: Maintains component state across re-renders
- **Identity**: Tracks items when list order changes

**Golden Rules for Keys:**
1. ✅ **Unique** among siblings
2. ✅ **Stable** across re-renders (don't use index!)
3. ✅ **Predictable** (same item = same key)

**Common Anti-patterns:**
- ❌ Using **array index** as key (causes bugs when list changes)
- ❌ Using **random values** (loses state on every render)
- ❌ Missing keys entirely (React warns, uses index)

### **Code (30%)**
```jsx
// BAD: Using index as key (causes issues when list changes)
function BadTodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <TodoItem key={index} todo={todo} /> // ❌ ANTI-PATTERN
      ))}
    </ul>
  );
}

// GOOD: Using stable, unique ID
function GoodTodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} /> // ✅
      ))}
    </ul>
  );
}

// When no ID exists, generate stable key
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        // Using combination of properties
        <li key={`${user.name}-${user.email}`}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}

// Complex list with reordering
function SortableList({ items, onReorder }) {
  return (
    <div>
      {items.map(item => (
        <DraggableItem 
          key={item.id} // Stable key maintains state during drag
          item={item}
          onDragEnd={onReorder}
        />
      ))}
    </div>
  );
}
```

---

## **3️⃣ Render Props Pattern**

### **Concept (70%)**
Render Props is a **pattern for sharing code** between components using a prop whose value is a function. It's like giving a component a "template" to render.

**When to use Render Props:**
- Share reusable logic (data fetching, mouse tracking)
- Create flexible, composable components
- Avoid prop drilling
- Alternative to Higher-Order Components

**How it works:**
1. Component accepts a function as a prop (usually `render` or `children`)
2. Component calls that function with some data
3. Function returns JSX using that data

### **Code (30%)**
```jsx
// 1. Basic Render Prop Component
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const handleMouseMove = (event) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };
  
  return (
    <div onMouseMove={handleMouseMove} style={{ height: '100vh' }}>
      {/* Call render prop with position data */}
      {render(position)}
    </div>
  );
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <h1>Move the mouse!</h1>
          <p>Position: {x}, {y}</p>
        </div>
      )}
    />
  );
}

// 2. Children as Function Pattern (More Common)
function DataFetcher({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .finally(() => setLoading(false));
  }, [url]);
  
  return children({ data, loading });
}

// Usage
function UserProfile() {
  return (
    <DataFetcher url="/api/user/123">
      {({ data, loading }) => (
        loading 
          ? <div>Loading...</div>
          : <div>Hello, {data.name}!</div>
      )}
    </DataFetcher>
  );
}

// 3. Render Props for Shared Logic
function Toggle({ children }) {
  const [on, setOn] = useState(false);
  
  const toggle = () => setOn(!on);
  
  return children({
    on,
    toggle,
    togglerProps: {
      'aria-pressed': on,
      onClick: toggle
    }
  });
}

// Usage
function App() {
  return (
    <Toggle>
      {({ on, toggle, togglerProps }) => (
        <div>
          <button {...togglerProps}>
            {on ? 'ON' : 'OFF'}
          </button>
          {on && <div>Content is visible!</div>}
        </div>
      )}
    </Toggle>
  );
}
```

---

## **4️⃣ Refs**

### **Concept (70%)**
Refs provide a way to **access DOM nodes or React elements** directly, or to **store mutable values** that don't trigger re-renders.

**Three Main Uses:**
1. **DOM Access**: Focus inputs, measure elements, integrate with third-party libraries
2. **Mutable Values**: Store data that persists across renders without causing re-renders
3. **Previous Values**: Track previous props/state

**Important Notes:**
- Changing ref.current **does NOT trigger re-render**
- Refs are **mutable** (can modify .current)
- Use sparingly (breaks React's declarative model)

### **Code (30%)**
```jsx
// 1. DOM Element Access
function TextInputWithFocus() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();
    inputRef.current.select(); // Select all text
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// 2. Storing Mutable Values (like instance variables)
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };
  
  useEffect(() => {
    return () => clearInterval(intervalRef.current); // Cleanup
  }, []);
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}

// 3. Forwarding Refs (access child's DOM from parent)
const FancyButton = React.forwardRef((props, ref) => {
  return (
    <button ref={ref} className="fancy-button">
      {props.children}
    </button>
  );
});

function Parent() {
  const buttonRef = useRef(null);
  
  useEffect(() => {
    console.log('Button width:', buttonRef.current.offsetWidth);
  }, []);
  
  return <FancyButton ref={buttonRef}>Click me!</FancyButton>;
}

// 4. Storing Previous Values
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  useEffect(() => {
    prevCountRef.current = count;
  });
  
  const prevCount = prevCountRef.current;
  
  return (
    <div>
      <p>Now: {count}, Before: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

---

## **5️⃣ Events**

### **Concept (70%)**
React events are **synthetic events** - wrappers around native browser events with cross-browser compatibility.

**Key Characteristics:**
- **SyntheticEvent**: Cross-browser wrapper
- **Event Pooling**: Events are pooled for performance (can't be accessed asynchronously)
- **Automatic Binding**: No need for `.bind(this)` in class components
- **Propagation**: Same as DOM (capturing → target → bubbling)

**Common Patterns:**
1. **Inline handlers**: Simple actions
2. **Extracted handlers**: Complex logic
3. **Parameter passing**: Pass data to handlers
4. **Event delegation**: Handle multiple events

### **Code (30%)**
```jsx
// 1. Basic Event Handling
function ButtonClickExample() {
  const handleClick = (event) => {
    event.preventDefault(); // Prevent default behavior
    event.stopPropagation(); // Stop bubbling
    console.log('Button clicked!', event.target);
  };
  
  return <button onClick={handleClick}>Click me</button>;
}

// 2. Passing Parameters to Handlers
function ItemList({ items }) {
  const handleItemClick = (itemId, event) => {
    console.log('Item clicked:', itemId);
    console.log('Event:', event.type);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id}
          onClick={(e) => handleItemClick(item.id, e)} // Arrow function
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// 3. Using data-* attributes
function DataAttributeExample() {
  const handleClick = (event) => {
    const itemId = event.target.dataset.id;
    const itemType = event.target.dataset.type;
    console.log(`Clicked ${itemType} with ID: ${itemId}`);
  };
  
  return (
    <button 
      data-id="123" 
      data-type="user"
      onClick={handleClick}
    >
      Click me
    </button>
  );
}

// 4. Form Events
function FormExample() {
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  const handleChange = (event) => {
    const { name, value } = event.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// 5. Custom Events with Event Emitter pattern
function EventEmitterExample() {
  const [logs, setLogs] = useState([]);
  
  useEffect(() => {
    const handleCustomEvent = (event) => {
      setLogs(prev => [...prev, `Event: ${event.detail.message}`]);
    };
    
    window.addEventListener('customEvent', handleCustomEvent);
    
    return () => {
      window.removeEventListener('customEvent', handleCustomEvent);
    };
  }, []);
  
  const triggerEvent = () => {
    window.dispatchEvent(
      new CustomEvent('customEvent', {
        detail: { message: 'Hello from custom event!' }
      })
    );
  };
  
  return (
    <div>
      <button onClick={triggerEvent}>Trigger Custom Event</button>
      <ul>
        {logs.map((log, i) => <li key={i}>{log}</li>)}
      </ul>
    </div>
  );
}
```

---

## **6️⃣ Higher-Order Components (HOCs)**

### **Concept (70%)**
HOCs are **functions that take a component and return a new component** with enhanced capabilities. They're a pattern for **reusing component logic**.

**HOC Characteristics:**
- **Function that returns a function**: Component → EnhancedComponent
- **Props manipulation**: Add, remove, or transform props
- **Logic abstraction**: Extract cross-cutting concerns

**Common Use Cases:**
- Authentication/Authorization
- Data fetching
- Logging/Analytics
- Styling/Theming
- Error boundaries

**HOC vs Render Props:**
- HOCs: Wrap components, harder to compose
- Render Props: More flexible, easier to understand

### **Code (30%)**
```jsx
// 1. Basic HOC Pattern
function withLogger(WrappedComponent) {
  return function LoggedComponent(props) {
    console.log(`Rendering ${WrappedComponent.name} with props:`, props);
    
    useEffect(() => {
      console.log(`${WrappedComponent.name} mounted`);
      return () => {
        console.log(`${WrappedComponent.name} unmounted`);
      };
    }, []);
    
    return <WrappedComponent {...props} />;
  };
}

// Usage
const EnhancedButton = withLogger(Button);

// 2. HOC with Additional Props
function withUserData(WrappedComponent) {
  return function UserDataComponent(props) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
      fetch('/api/current-user')
        .then(res => res.json())
        .then(setUser)
        .finally(() => setLoading(false));
    }, []);
    
    return (
      <WrappedComponent
        {...props}
        user={user}
        loading={loading}
      />
    );
  };
}

// 3. HOC for Authentication
function withAuth(WrappedComponent) {
  return function AuthComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [checking, setChecking] = useState(true);
    
    useEffect(() => {
      checkAuthToken().then(authStatus => {
        setIsAuthenticated(authStatus);
        setChecking(false);
      });
    }, []);
    
    if (checking) return <div>Checking authentication...</div>;
    if (!isAuthenticated) return <div>Please log in</div>;
    
    return <WrappedComponent {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);

// 4. Composing Multiple HOCs (Can get messy)
const SuperEnhancedComponent = withLogger(
  withAuth(
    withUserData(MyComponent)
  )
);

// 5. HOC with Config Options
function withLoadingIndicator(WrappedComponent, loadingMessage = 'Loading...') {
  return function LoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>{loadingMessage}</div>;
    }
    
    return <WrappedComponent {...props} />;
  };
}

// Usage with different messages
const UserProfileWithLoading = withLoadingIndicator(
  UserProfile, 
  'Fetching user data...'
);
const PostListWithLoading = withLoadingIndicator(
  PostList,
  'Loading posts...'
);

// 6. HOC that Injects Props
function withTheme(WrappedComponent) {
  return function ThemedComponent(props) {
    const [theme, setTheme] = useState('light');
    
    const toggleTheme = () => {
      setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };
    
    return (
      <WrappedComponent
        {...props}
        theme={theme}
        toggleTheme={toggleTheme}
        themeStyles={theme === 'light' ? lightStyles : darkStyles}
      />
    );
  };
}

// Usage
const ThemedButton = withTheme(Button);
// Button now receives theme, toggleTheme, themeStyles as props
```

---

## **🎯 Advanced Patterns & Best Practices**

### **Render Optimization Strategies**
```jsx
// 1. React.memo for component memoization
const ExpensiveComponent = React.memo(function({ data }) {
  // Only re-renders when props change
  return <div>{JSON.stringify(data)}</div>;
});

// 2. useMemo for expensive calculations
function OptimizedComponent({ items }) {
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.value - b.value);
  }, [items]); // Only recompute when items change
}

// 3. Code splitting with React.lazy
const LazyComponent = React.lazy(() => import('./HeavyComponent'));
```

### **Choosing the Right Pattern**
| **Use Case** | **Recommended Pattern** |
|-------------|----------------------|
| **Share UI logic** | Custom Hooks |
| **Share rendering logic** | Render Props |
| **Cross-cutting concerns** | HOCs |
| **DOM access** | Refs |
| **List rendering** | Keys |
| **Event handling** | Synthetic Events |

---

## **📊 Key Takeaways:**

1. **Lifecycle**: Understand mount/update/unmount for side effects
2. **Keys**: Essential for list performance and state preservation
3. **Render Props**: Flexible pattern for logic sharing
4. **Refs**: Escape hatch for imperative DOM operations
5. **Events**: Synthetic, cross-browser compatible
6. **HOCs**: Pattern for logic reuse (less common with hooks)

**Remember**: React's rendering is about **declarative UI**. Use these patterns to write clean, efficient, and maintainable code while letting React handle the DOM updates!