# **React Component Fundamentals - From Basic to Advanced**

## **1. Component Basics**
### **Concept (70%)**
Components are the **building blocks** of React applications. Think of them like **LEGO bricks** - each component is a reusable piece of UI that can be combined to build complex interfaces. Components can be **class components** (older style) or **function components** (modern approach). They encapsulate **structure**, **logic**, and **appearance** in one package.

### **Code (30%)**
```jsx
// Simplest component - returns JSX
function Welcome() {
  return <h1>Hello, User!</h1>;
}

// Using the component
function App() {
  return <Welcome />;  // Renders: <h1>Hello, User!</h1>
}
```

---

## **2. Functional Components**
### **Concept (70%)**
Functional components are **JavaScript functions** that return JSX. They're **simpler** and **more concise** than class components. With the introduction of **Hooks** in React 16.8, functional components can now handle state and lifecycle, making them the **preferred approach** today. They're easier to read, test, and debug.

**Key characteristics:**
- Plain JavaScript functions
- Accept `props` as first parameter
- Must return JSX (or `null`)
- No `this` keyword needed
- Can use Hooks for state and effects

### **Code (30%)**
```jsx
// Functional Component with props
function Greeting(props) {
  return <h2>Welcome, {props.name}!</h2>;
}

// Using the component
function App() {
  return <Greeting name="Alice" />;
}

// Arrow function syntax (common)
const Button = ({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
);
```

---

## **3. JSX (JavaScript XML)**
### **Concept (70%)**
JSX is a **syntax extension** for JavaScript that looks similar to HTML but gets **compiled to React elements**. It's not a template language; it's **JavaScript with superpowers**. Under the hood, JSX becomes calls to `React.createElement()`.

**Important rules:**
1. **Single root element** - JSX must return one parent element
2. **className instead of class** - `class` is reserved in JS
3. **JavaScript expressions** inside `{}`
4. **Self-closing tags** for elements without children
5. **CamelCase properties** - `onClick` not `onclick`

### **Code (30%)**
```jsx
// JSX compilation
const element = <h1 className="title">Hello</h1>;
// Compiles to:
const element = React.createElement('h1', {className: 'title'}, 'Hello');

// JavaScript in JSX
function UserCard({ user, isAdmin }) {
  return (
    <div className={`card ${isAdmin ? 'admin' : ''}`}>
      <h2>{user.name.toUpperCase()}</h2>
      <p>Age: {user.age}</p>
      {user.bio && <p>{user.bio}</p>}  // Conditional rendering
    </div>
  );
}
```

---

## **4. Props vs State**
### **Concept (70%)**
Understanding the difference between props and state is **crucial** for React development.

| **Aspect** | **Props (Properties)** | **State** |
|------------|---------------------|----------|
| **Ownership** | Passed **from parent** to child | Managed **within component** |
| **Mutability** | **Immutable** (read-only) | **Mutable** (can change) |
| **Purpose** | Configuration data | Component's internal memory |
| **Updates** | Parent re-renders with new props | `setState()` or `useState()` setter |

**Golden Rule:** Props flow **down**, state stays **local**.

### **Code (30%)**
```jsx
// PROPS - Configuration from parent
function ChildComponent({ message, count }) {
  return <div>{message} - {count}</div>;
}

// STATE - Internal memory
function Counter() {
  const [count, setCount] = useState(0); // State hook
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// Parent passing props to child
function ParentComponent() {
  const [total, setTotal] = useState(0);
  
  return (
    <div>
      <ChildComponent 
        message="Total clicks" 
        count={total} 
      />
      <Counter />
    </div>
  );
}
```

---

## **5. Conditional Rendering**
### **Concept (70%)**
Conditional rendering allows components to **show/hide elements** based on conditions. React gives you multiple patterns:

1. **If/else statements** - Traditional logic flow
2. **Ternary operators** - Inline conditional (condition ? true : false)
3. **Logical && operator** - Render or nothing
4. **Early returns** - Exit function early
5. **Component state** - Show different UIs based on state

**Best Practice:** Keep conditional logic **simple** and **readable**. Extract complex conditions into variables or functions.

### **Code (30%)**
```jsx
// Multiple patterns in action
function UserProfile({ user, isLoading, error }) {
  // 1. Early return for error/loading states
  if (error) return <ErrorDisplay message={error} />;
  if (isLoading) return <Spinner />;
  
  // 2. Conditional rendering with &&
  return (
    <div className="profile">
      <h1>{user.name}</h1>
      
      {/* 3. Ternary operator */}
      {user.isAdmin ? (
        <AdminPanel />
      ) : (
        <UserDashboard />
      )}
      
      {/* 4. Logical && for optional content */}
      {user.bio && <p className="bio">{user.bio}</p>}
      
      {/* 5. Conditional classes */}
      <div className={`status ${user.isActive ? 'active' : 'inactive'}`}>
        Status: {user.isActive ? 'Online' : 'Offline'}
      </div>
    </div>
  );
}
```

---

## **6. Composition**
### **Concept (70%)**
Composition is the **most powerful** React pattern. Instead of inheritance (like in OOP), React favors **composition** - building complex UIs by combining simple components. Think: "Has-a" rather than "Is-a".

**Composition patterns:**
1. **Containment** - Components that don't know their children ahead of time
2. **Specialization** - Specific components that render more general ones
3. **Render Props** - Components that receive functions as children
4. **Higher-Order Components** - Functions that take components and return enhanced ones
5. **Custom Hooks** - Reusable logic composition

**Philosophy:** "Build small, focused components and compose them together."

### **Code (30%)**
```jsx
// 1. CONTAINMENT - Using children prop
function Card({ children, title }) {
  return (
    <div className="card">
      {title && <h3>{title}</h3>}
      <div className="content">{children}</div>
    </div>
  );
}

// Usage
function App() {
  return (
    <Card title="User Profile">
      <Avatar />
      <UserInfo />
      <ActionButtons />
    </Card>
  );
}

// 2. SPECIALIZATION
function WelcomeDialog() {
  return (
    <Dialog title="Welcome" message="Thank you for visiting!">
      <SignupForm />
    </Dialog>
  );
}

// 3. RENDER PROPS PATTERN
function Toggle({ children }) {
  const [on, setOn] = useState(false);
  
  return children({
    on,
    toggle: () => setOn(!on)
  });
}

// Usage
<Toggle>
  {({ on, toggle }) => (
    <button onClick={toggle}>
      {on ? 'ON' : 'OFF'}
    </button>
  )}
</Toggle>

// 4. COMPOSING CUSTOM HOOKS
function useUserData(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading };
}

// Use multiple hooks together
function UserProfile({ userId }) {
  const { user, loading } = useUserData(userId);
  const { permissions } = usePermissions(userId);
  const { notifications } = useNotifications(userId);
  
  if (loading) return <Spinner />;
  
  return (
    <div>
      <UserHeader user={user} />
      <UserContent user={user} permissions={permissions} />
      <NotificationList items={notifications} />
    </div>
  );
}
```

---

## **Advanced Composition Pattern: Compound Components**
```jsx
// Building a flexible Tab system
function Tabs({ children }) {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return React.Children.map(children, (child, index) => {
    return React.cloneElement(child, {
      isActive: index === activeIndex,
      onSelect: () => setActiveIndex(index)
    });
  });
}

function Tab({ children, isActive, onSelect }) {
  return (
    <button 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={onSelect}
    >
      {children}
    </button>
  );
}

function TabPanel({ children, isActive }) {
  return isActive ? <div className="panel">{children}</div> : null;
}

// Usage - Beautifully declarative
function App() {
  return (
    <Tabs>
      <Tab>Profile</Tab>
      <TabPanel>Profile content...</TabPanel>
      
      <Tab>Settings</Tab>
      <TabPanel>Settings content...</TabPanel>
      
      <Tab>Messages</Tab>
      <TabPanel>Messages content...</TabPanel>
    </Tabs>
  );
}
```

---

## **Key Takeaways:**
1. **Start simple** with functional components and JSX
2. **Master props/state distinction** - this is fundamental
3. **Use conditional rendering** to create dynamic UIs
4. **Embrace composition** - it's React's superpower
5. **Build small, reusable components** that do one thing well
6. **Combine patterns** as you advance (composition + conditional rendering + state)

The progression from **Component Basics → Functional Components → JSX → Props/State → Conditional Rendering → Composition** represents the journey from a React beginner to an advanced practitioner. Each concept builds on the previous one, creating a solid foundation for building scalable, maintainable applications.