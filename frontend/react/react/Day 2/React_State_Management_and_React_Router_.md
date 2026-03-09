# **📊 React State Management - From Basic to Advanced**

## **🎯 Understanding State Management**

### **Concept (70%)**
State management is **how you handle data that changes over time** in your React application. Think of it as the **"memory"** of your app - what users see, what data they've entered, what's loaded from APIs.

**Why State Management Matters:**
1. **Data Flow**: How data moves through your app
2. **Predictability**: Same state → same UI
3. **Performance**: Avoid unnecessary re-renders
4. **Maintainability**: Organized, testable code
5. **Scalability**: Handle complex app states

**The State Management Spectrum:**
```
Local State → Component State → Global State → Server State
Simple     →     Complex     →  Shared      → External
```

---

## **1️⃣ Context API (Built-in React)**

### **Concept (70%)**
Context is React's **built-in solution for sharing data** deeply through the component tree without prop drilling. It's like creating a **"global variable"** that any component can access.

**When to Use Context:**
- ✅ **Theme settings** (dark/light mode)
- ✅ **User authentication** (current user, permissions)
- ✅ **Language preferences** (i18n)
- ✅ **Global UI state** (modals, notifications)
- ❌ **Frequently updated data** (forms, real-time data)
- ❌ **Large datasets** (performance issues)

**Context Architecture:**
```
Provider (holds state)
  ├── Consumer Component 1
  ├── Consumer Component 2
  └── Consumer Component 3
```

### **Code (30%)**
```jsx
// 1. Basic Context Setup
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = {
    theme,
    setTheme,
    isDark: theme === 'dark',
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light')
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook for better DX
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 2. Multiple Contexts for Separation of Concerns
const UserContext = createContext();
const CartContext = createContext();
const SettingsContext = createContext();

function AppProviders({ children }) {
  return (
    <UserContext.Provider value={useUserState()}>
      <CartContext.Provider value={useCartState()}>
        <SettingsContext.Provider value={useSettingsState()}>
          {children}
        </SettingsContext.Provider>
      </CartContext.Provider>
    </UserContext.Provider>
  );
}

// 3. Optimizing Context with useMemo
function OptimizedProvider({ children }) {
  const [state, setState] = useState(initialState);
  
  const contextValue = useMemo(() => ({
    state,
    update: (newState) => setState(prev => ({ ...prev, ...newState }))
  }), [state]); // Only recreates when state changes
  
  return (
    <MyContext.Provider value={contextValue}>
      {children}
    </MyContext.Provider>
  );
}

// Usage
function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={`header-${theme}`}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </header>
  );
}
```

---

## **2️⃣ Zustand (The Minimalist)**

### **Concept (70%)**
Zustand (German for "state") is a **minimal, unopinionated state management library** that combines the simplicity of Context with the power of Redux patterns.

**Zustand Philosophy:**
- 🎯 **Zero boilerplate** - Just create a store
- ⚡ **No providers needed** - Components subscribe directly
- 🔧 **Flexible** - Use any React pattern you like
- 📦 **Tiny bundle** (~1KB)
- 🎭 **Immutable updates** with Immer integration

**Why Zustand?**
- Simpler than Redux
- More performant than Context for frequent updates
- Great TypeScript support
- Middleware support (persist, devtools)

### **Code (30%)**
```jsx
import create from 'zustand';
import { persist } from 'zustand/middleware';

// 1. Basic Store
const useStore = create((set, get) => ({
  // State
  bears: 0,
  fish: 10,
  
  // Actions
  increaseBears: () => set(state => ({ bears: state.bears + 1 })),
  increaseFish: () => set(state => ({ fish: state.fish + 1 })),
  
  // Async action
  fetchBears: async () => {
    const response = await fetch('/api/bears');
    const bears = await response.json();
    set({ bears });
  },
  
  // Computed values (derived state)
  totalAnimals: () => get().bears + get().fish,
  
  // Reset action
  reset: () => set({ bears: 0, fish: 0 })
}));

// 2. With Middleware (persist + devtools)
const useAuthStore = create(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: () => !!get().token,
      
      login: (userData, authToken) => set({ 
        user: userData, 
        token: authToken 
      }),
      
      logout: () => set({ user: null, token: null })
    }),
    {
      name: 'auth-storage', // localStorage key
      getStorage: () => localStorage,
    }
  )
);

// 3. Sliced Store Pattern (for larger apps)
const createAuthSlice = (set, get) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null })
});

const createCartSlice = (set, get) => ({
  items: [],
  addItem: (item) => set(state => ({
    items: [...state.items, item]
  })),
  total: () => get().items.reduce((sum, item) => sum + item.price, 0)
});

const useBoundStore = create((...a) => ({
  ...createAuthSlice(...a),
  ...createCartSlice(...a)
}));

// Usage in Components
function BearCounter() {
  const bears = useStore(state => state.bears);
  const increaseBears = useStore(state => state.increaseBears);
  
  return (
    <div>
      <h1>{bears} bears around here</h1>
      <button onClick={increaseBears}>Add bear</button>
    </div>
  );
}

// Select multiple values
function ZooDashboard() {
  const { bears, fish, totalAnimals } = useStore(state => ({
    bears: state.bears,
    fish: state.fish,
    totalAnimals: state.totalAnimals()
  }));
  
  return (
    <div>
      <p>Bears: {bears}</p>
      <p>Fish: {fish}</p>
      <p>Total: {totalAnimals}</p>
    </div>
  );
}
```

---

## **3️⃣ Jotai (Atomic State)**

### **Concept (70%)**
Jotai (Japanese for "state") takes a **truly atomic approach** to state management. Each piece of state is an "atom" that can be read and written independently.

**Jotai Core Concepts:**
- ⚛️ **Atoms**: Minimal units of state (like useState but shareable)
- 🔗 **Derived Atoms**: Computed from other atoms (like selectors)
- 🎯 **No Keys**: Unlike Recoil, no string keys needed
- 📦 **Tiny**: ~3KB bundle
- 🎭 **React-ish**: Feels like using useState/useContext

**Jotai vs Zustand:**
- Jotai: **Bottom-up**, compose atoms
- Zustand: **Top-down**, single store

### **Code (30%)**
```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// 1. Basic Atoms
const countAtom = atom(0);
const nameAtom = atom('John');

// 2. Derived Atoms (computed)
const doubledCountAtom = atom((get) => get(countAtom) * 2);
const userInfoAtom = atom((get) => ({
  name: get(nameAtom),
  count: get(countAtom),
  doubled: get(doubledCountAtom)
}));

// 3. Write-only Atoms (actions)
const incrementAtom = atom(
  null, // read function (not used)
  (get, set) => set(countAtom, get(countAtom) + 1)
);

// 4. Async Atoms
const fetchUserAtom = atom(async (get) => {
  const userId = get(userIdAtom);
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// 5. Persisted Atom (localStorage)
const themeAtom = atomWithStorage('theme', 'light');

// 6. Atom Families (dynamic atoms)
const itemAtoms = atomFamily((id) => 
  atom({ id, name: '', quantity: 0 })
);

// Usage in Components
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubled = useAtomValue(doubledCountAtom);
  const increment = useSetAtom(incrementAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={increment}>Increment via Action</button>
    </div>
  );
}

function UserProfile() {
  const userInfo = useAtomValue(userInfoAtom);
  const [theme, setTheme] = useAtom(themeAtom);
  
  return (
    <div className={`theme-${theme}`}>
      <h1>{userInfo.name}</h1>
      <p>Count: {userInfo.count}</p>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}

// 7. Complex Example: Shopping Cart
const cartAtom = atom([]);
const cartTotalAtom = atom((get) => 
  get(cartAtom).reduce((sum, item) => sum + item.price * item.quantity, 0)
);

const addToCartAtom = atom(
  null,
  (get, set, newItem) => {
    const cart = get(cartAtom);
    const existingItem = cart.find(item => item.id === newItem.id);
    
    if (existingItem) {
      set(cartAtom, cart.map(item => 
        item.id === newItem.id 
          ? { ...item, quantity: item.quantity + 1 }
          : item
      ));
    } else {
      set(cartAtom, [...cart, { ...newItem, quantity: 1 }]);
    }
  }
);
```

---

## **4️⃣ MobX (Reactive State)**

### **Concept (70%)**
MobX takes a **reactive, observable approach** to state management. It automatically tracks what state is used and updates components when that state changes.

**MobX Philosophy:**
- 🎯 **Automatic reactivity** - No manual subscriptions
- ⚡ **Makes state mutable** (but tracked)
- 🔍 **Fine-grained updates** - Only re-renders what changed
- 🏗️ **OOP-friendly** - Classes and decorators
- 🔄 **Two-way data binding** if you want it

**Core Concepts:**
- **Observable**: State that can be observed
- **Action**: Functions that modify state
- **Computed**: Derived values (auto-cached)
- **Reaction**: Side effects (autorun, reaction)

### **Code (30%)**
```jsx
import { makeAutoObservable, autorun, reaction } from 'mobx';
import { observer } from 'mobx-react-lite';

// 1. Store Class
class TodoStore {
  todos = [];
  filter = 'all';
  
  constructor() {
    makeAutoObservable(this);
  }
  
  // Action
  addTodo(text) {
    this.todos.push({
      id: Date.now(),
      text,
      completed: false
    });
  }
  
  // Action
  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }
  
  // Computed property
  get completedTodos() {
    return this.todos.filter(t => t.completed);
  }
  
  get filteredTodos() {
    switch (this.filter) {
      case 'completed':
        return this.completedTodos;
      case 'active':
        return this.todos.filter(t => !t.completed);
      default:
        return this.todos;
    }
  }
}

// 2. Root Store Pattern
class RootStore {
  constructor() {
    this.todoStore = new TodoStore();
    this.userStore = new UserStore();
    this.uiStore = new UIStore();
  }
}

// 3. React Integration
const todoStore = new TodoStore();

// Observer HOC
const TodoList = observer(() => {
  return (
    <div>
      {todoStore.filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  );
});

// Observer component
const TodoItem = observer(({ todo }) => {
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => todoStore.toggleTodo(todo.id)}
      />
      <span>{todo.text}</span>
    </div>
  );
});

// 4. Async Actions
class UserStore {
  users = [];
  loading = false;
  
  constructor() {
    makeAutoObservable(this);
  }
  
  // Async action
  async fetchUsers() {
    this.loading = true;
    try {
      const response = await fetch('/api/users');
      this.users = await response.json();
    } catch (error) {
      console.error(error);
    } finally {
      this.loading = false;
    }
  }
}

// 5. Reactions (side effects)
autorun(() => {
  console.log('Todo count:', todoStore.todos.length);
  console.log('Completed:', todoStore.completedTodos.length);
});

// Conditional reaction
reaction(
  () => todoStore.todos.length,
  (todoCount) => {
    if (todoCount > 10) {
      console.warn('Too many todos!');
    }
  }
);

// 6. Local Observable in Function Component
const useLocalStore = (initializer) => {
  const store = useMemo(() => {
    const store = {};
    makeAutoObservable(store);
    initializer(store);
    return store;
  }, []);
  
  return store;
};

function CounterComponent() {
  const counter = useLocalStore(() => ({
    count: 0,
    increment() {
      this.count++;
    },
    get doubled() {
      return this.count * 2;
    }
  }));
  
  return (
    <div>
      <p>Count: {counter.count}</p>
      <p>Doubled: {counter.doubled}</p>
      <button onClick={() => counter.increment()}>Increment</button>
    </div>
  );
}
```

---

## **🔄 State Management Comparison**

| **Library** | **Size** | **Learning Curve** | **Performance** | **Best For** |
|------------|----------|-------------------|----------------|-------------|
| **Context** | 0KB | Low | Poor (frequent updates) | Theme, auth, simple global state |
| **Zustand** | ~1KB | Low | Excellent | Most apps, simple to complex |
| **Jotai** | ~3KB | Medium | Excellent | Component-driven, derived state |
| **MobX** | ~15KB | High | Excellent | Complex reactive systems, OOP lovers |

**Rule of Thumb:**
- Start with **Context** for simple global state
- Use **Zustand** for most applications
- Try **Jotai** if you like atomic composition
- Choose **MobX** for complex reactive systems

---

## **🚪 React Routers**

## **1️⃣ React Router (Most Popular)**

### **Concept (70%)**
React Router is the **standard routing library** for React that enables navigation between components while keeping the UI in sync with the URL.

**Core Concepts:**
- **Declarative routing**: Define routes as components
- **Nested routes**: Parent-child route relationships
- **Dynamic segments**: `/users/:id` style URLs
- **Navigation**: `<Link>` for SPA navigation
- **Code splitting**: Lazy loading with routes

**React Router v6 Features:**
- Simplified API (`<Routes>` instead of `<Switch>`)
- Relative links and routes
- `useNavigate` hook replaces `useHistory`
- `Outlet` for nested routes

### **Code (30%)**
```jsx
// 1. Basic Setup
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/users">Users</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />} />
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// 2. Nested Routes with Layouts
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <nav>
        <Link to="stats">Stats</Link>
        <Link to="settings">Settings</Link>
      </nav>
      
      {/* Child routes render here */}
      <Outlet />
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="dashboard" element={<Dashboard />}>
          <Route index element={<DashboardHome />} />
          <Route path="stats" element={<Stats />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Route>
    </Routes>
  );
}

// 3. Programmatic Navigation
function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();
  
  const handleLogin = async () => {
    await login();
    
    // Redirect to where they came from, or home
    const from = location.state?.from?.pathname || '/';
    navigate(from, { replace: true });
  };
  
  return (
    <div>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}

// 4. Protected Routes
function RequireAuth({ children }) {
  const auth = useAuth();
  const location = useLocation();
  
  if (!auth.user) {
    // Redirect to login, but save where they tried to go
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Usage
<Route 
  path="/protected" 
  element={
    <RequireAuth>
      <ProtectedPage />
    </RequireAuth>
  } 
/>

// 5. Data Loading with Loaders (v6.4+)
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    loader: () => fetch('/api/data'), // Runs before rendering
    children: [
      {
        path: 'user/:id',
        element: <User />,
        loader: async ({ params }) => {
          return fetch(`/api/users/${params.id}`);
        }
      }
    ]
  }
]);

function App() {
  return <RouterProvider router={router} />;
}

// Access loader data in component
function User() {
  const user = useLoaderData(); // Get data from loader
  return <div>{user.name}</div>;
}
```

---

## **2️⃣ TanStack Router (Formerly React Location)**

### **Concept (70%)**
TanStack Router is a **type-safe, performant router** built by the creators of React Query. It focuses on **developer experience** and **TypeScript integration**.

**Key Features:**
- 🔒 **Full TypeScript support** - Type-safe routes
- 📊 **Built-in data loading** - Integrated with TanStack Query
- ⚡ **Suspense-ready** - Built for concurrent features
- 🔧 **Flexible** - File-based or code-based routing
- 🎯 **Search params** - First-class support

**Why TanStack Router?**
- If you use **TanStack Query** (React Query)
- Need **excellent TypeScript support**
- Want **integrated data loading**
- Like **file-based routing** (like Next.js)

### **Code (30%)**
```jsx
// 1. Basic Setup with File-based Routing
// File structure:
// routes/
//   __root.tsx
//   index.tsx
//   about.tsx
//   users/
//     index.tsx
//     $userId.tsx

// __root.tsx
import { createRootRoute, Outlet } from '@tanstack/react-router';

export const Route = createRootRoute({
  component: RootComponent,
});

function RootComponent() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Outlet />
    </div>
  );
}

// routes/users/$userId.tsx (Dynamic route)
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/users/$userId')({
  component: UserComponent,
  loader: async ({ params }) => {
    // Data loading - integrates with TanStack Query
    const user = await fetchUser(params.userId);
    return { user };
  },
});

function UserComponent() {
  const { user } = Route.useLoaderData();
  return <div>{user.name}</div>;
}

// 2. Code-based Routing
import { createRouter, Route } from '@tanstack/react-router';

const rootRoute = new Route({
  getParentRoute: () => null,
  path: '/',
  component: Root,
});

const indexRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/',
  component: Home,
});

const aboutRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/about',
  component: About,
});

const routeTree = rootRoute.addChildren([indexRoute, aboutRoute]);

const router = createRouter({ routeTree });

// 3. Type-Safe Navigation
function NavigationExample() {
  const navigate = useNavigate();
  
  // Type-safe! Knows available routes
  const goToUser = (userId: string) => {
    navigate({
      to: '/users/$userId',
      params: { userId }, // Type-checked!
      search: { tab: 'profile' }, // Type-checked search params
    });
  };
  
  return <button onClick={() => goToUser('123')}>Go to User 123</button>;
}

// 4. Search Params with Type Safety
// routes/dashboard.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/dashboard')({
  validateSearch: (search) => {
    // Type-safe search params validation
    return z.object({
      tab: z.enum(['overview', 'analytics', 'settings']).catch('overview'),
      page: z.number().catch(1),
    }).parse(search);
  },
  component: Dashboard,
});

function Dashboard() {
  // search is fully typed!
  const { tab, page } = Route.useSearch();
  
  return (
    <div>
      <h1>Dashboard - {tab}</h1>
      <p>Page: {page}</p>
    </div>
  );
}

// 5. Integrated Data Loading with TanStack Query
// routes/posts.tsx
import { createFileRoute } from '@tanstack/react-router';
import { useQuery } from '@tanstack/react-query';

export const Route = createFileRoute('/posts')({
  component: PostsComponent,
  loader: async ({ context }) => {
    // Prefetch data
    await context.queryClient.prefetchQuery({
      queryKey: ['posts'],
      queryFn: fetchPosts,
    });
  },
});

function PostsComponent() {
  // Use the same query in component
  const { data: posts } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });
  
  return (
    <div>
      {posts?.map(post => <div key={post.id}>{post.title}</div>)}
    </div>
  );
}

// 6. Route Guards (Authentication)
// routes/protected.tsx
export const Route = createFileRoute('/protected')({
  beforeLoad: ({ context, location }) => {
    // Check authentication
    if (!context.auth.isAuthenticated) {
      throw redirect({
        to: '/login',
        search: {
          redirect: location.href,
        },
      });
    }
  },
  component: ProtectedComponent,
});
```

---

## **🔄 Router Comparison**

| **Feature** | **React Router** | **TanStack Router** |
|------------|-----------------|-------------------|
| **Popularity** | Industry standard | Growing, newer |
| **TypeScript** | Good | Excellent (full type safety) |
| **Data Loading** | Basic (v6.4+) | Advanced (TanStack Query) |
| **File-based** | No (community libs) | Yes (built-in) |
| **Bundle Size** | ~15KB | ~30KB (with Query) |
| **Learning Curve** | Low | Medium |
| **Best For** | Most projects | TypeScript apps, TanStack ecosystem |

**Choosing a Router:**
- **React Router**: Default choice, large ecosystem
- **TanStack Router**: If you use TanStack Query, want type safety

---

## **🎯 State + Router Integration**

### **Global State with Routing**
```jsx
// Zustand store with router integration
const useAppStore = create((set, get) => ({
  // State
  currentPage: '/',
  
  // Actions
  navigateTo: (path) => {
    // Update store
    set({ currentPage: path });
    // Update URL
    window.history.pushState({}, '', path);
  },
  
  // Sync with browser back/forward
  initRouter: () => {
    window.addEventListener('popstate', () => {
      set({ currentPage: window.location.pathname });
    });
  }
}));

// TanStack Router + Zustand integration
function App() {
  const store = useAppStore();
  
  return (
    <Router
      context={{ store }} // Pass store to router context
      routeTree={routeTree}
    />
  );
}

// In route components
function ProtectedRoute() {
  const { store } = useRouterContext();
  const isAuthenticated = store(state => state.isAuthenticated);
  
  if (!isAuthenticated) {
    throw redirect({ to: '/login' });
  }
  
  return <Outlet />;
}
```

---

## **📊 Final Recommendations**

### **State Management Choice:**
1. **Start with Context** for simple apps
2. **Move to Zustand** when Context becomes limiting
3. **Try Jotai** if you prefer atomic composition
4. **Use MobX** for complex reactive systems

### **Router Choice:**
1. **React Router** for most projects (stable, familiar)
2. **TanStack Router** for TanStack ecosystem or type-heavy projects

### **Golden Rule:**
> **Keep state close to where it's used.** Don't put everything in global state. Start local, lift up when needed, go global only when necessary.

**Remember**: The best state management is the one that fits your team's mental model and your application's needs! 🚀