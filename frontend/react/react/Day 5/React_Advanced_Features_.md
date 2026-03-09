# React Advanced Features Explained: From Basic to Advanced

## 1. **Error Boundaries** (Basic to Advanced)

### **Concept (70%)**
Error Boundaries are React components that **catch JavaScript errors anywhere in their child component tree**, log those errors, and display a fallback UI instead of crashing the entire application. They work like JavaScript `catch {}` blocks but for React components.

**Key Characteristics:**
- **Class components only** (though hooks exist for functional components)
- Catch errors during **rendering**, **lifecycle methods**, and **constructors**
- Do NOT catch errors in:
  - Event handlers (use regular try-catch for these)
  - Asynchronous code
  - Server-side rendering
  - The error boundary itself

**When to Use:**
- Prevent entire app crashes from isolated component failures
- Provide user-friendly error messages
- Log errors to monitoring services

### **Code (30%)**

```jsx
// BASIC: Simple Error Boundary
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <p>{this.state.error.toString()}</p>
          </details>
        </div>
      );
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyUnstableComponent />
</ErrorBoundary>

// ADVANCED: Error Boundary with Recovery
class SmartErrorBoundary extends React.Component {
  state = { 
    hasError: false, 
    error: null,
    retryCount: 0 
  };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  handleRetry = () => {
    this.setState({ 
      hasError: false, 
      error: null, 
      retryCount: this.state.retryCount + 1 
    });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-recovery">
          <h3>Component Failed</h3>
          <button onClick={this.handleRetry}>
            Retry (Attempt {this.state.retryCount + 1})
          </button>
          {this.state.retryCount > 2 && (
            <button onClick={() => window.location.reload()}>
              Reload Page
            </button>
          )}
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

## 2. **Server APIs** (Basic to Advanced)

### **Concept (70%)**
Server APIs in React refer to **data fetching and server-side interactions**, not to be confused with React's server components. This encompasses patterns for loading, caching, and synchronizing data with backend services.

**Evolution of Data Fetching in React:**
1. **Basic**: `useEffect` + `fetch` in components
2. **Intermediate**: Custom hooks for data fetching
3. **Advanced**: Data fetching libraries (React Query, SWR) with caching, synchronization, and optimistic updates

**Key Concepts:**
- **Loading states**: Show spinners or skeletons while data loads
- **Error handling**: Manage failed API requests
- **Caching**: Store responses to avoid redundant requests
- **Refetching**: Update data based on events or intervals
- **Optimistic updates**: Update UI before server confirmation
- **Pagination/Infinite scroll**: Handle large datasets

### **Code (30%)**

```jsx
// BASIC: Simple useEffect Fetching
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <ProfileCard user={user} />;
}

// INTERMEDIATE: Custom Hook
function useApi(endpoint, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      const response = await fetch(endpoint, options);
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err);
    } finally {
      setLoading(false);
    }
  }, [endpoint, options]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// ADVANCED: React Query Example (with caching, retries, etc.)
import { useQuery, useMutation, useQueryClient } from 'react-query';

function AdvancedUserProfile({ userId }) {
  const queryClient = useQueryClient();
  
  // Automatic caching, retries, and refetching
  const { data: user, isLoading, error } = useQuery(
    ['user', userId],
    () => fetch(`/api/users/${userId}`).then(res => res.json()),
    {
      staleTime: 5 * 60 * 1000, // Cache for 5 minutes
      retry: 3,
      retryDelay: attempt => attempt * 1000
    }
  );

  // Optimistic update mutation
  const updateUser = useMutation(
    (newData) => fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify(newData)
    }),
    {
      onMutate: async (newData) => {
        // Cancel outgoing refetches
        await queryClient.cancelQueries(['user', userId]);
        
        // Snapshot previous value
        const previousUser = queryClient.getQueryData(['user', userId]);
        
        // Optimistically update
        queryClient.setQueryData(['user', userId], newData);
        
        return { previousUser };
      },
      onError: (err, newData, context) => {
        // Rollback on error
        queryClient.setQueryData(['user', userId], context.previousUser);
      },
      onSettled: () => {
        // Refetch after error or success
        queryClient.invalidateQueries(['user', userId]);
      }
    }
  );
}
```

---

## 3. **Suspense** (Basic to Advanced)

### **Concept (70%)**
Suspense is a React feature that **lets components "wait" for something before rendering**. Initially designed for code splitting, it now supports data fetching, enabling declarative loading states.

**Core Concepts:**
- **Fallback UI**: Show loading indicators while waiting
- **Suspense boundaries**: Wrap components that might suspend
- **Concurrent features**: Works with React 18+ concurrent features
- **Two main use cases**:
  1. **Code splitting**: Lazy loading components
  2. **Data fetching**: Waiting for async data

**How It Works:**
1. When a component "suspends" (needs to wait for something)
2. React looks up the nearest `<Suspense>` boundary
3. Shows the fallback until the resource loads
4. Then renders the actual component

### **Code (30%)**

```jsx
// BASIC: Code Splitting with Suspense
import React, { Suspense, lazy } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

function App() {
  const [currentTab, setCurrentTab] = useState('dashboard');
  
  return (
    <div>
      <nav>
        <button onClick={() => setCurrentTab('dashboard')}>Dashboard</button>
        <button onClick={() => setCurrentTab('settings')}>Settings</button>
      </nav>
      
      <Suspense fallback={<Spinner />}>
        {currentTab === 'dashboard' ? <Dashboard /> : <Settings />}
      </Suspense>
    </div>
  );
}

// INTERMEDIATE: Multiple Suspense Boundaries
function UserPage() {
  return (
    <div>
      {/* Header loads immediately */}
      <Header />
      
      {/* User info loads with its own spinner */}
      <Suspense fallback={<UserCardSkeleton />}>
        <UserProfile userId={123} />
      </Suspense>
      
      {/* Posts load separately */}
      <Suspense fallback={<PostListSkeleton />}>
        <UserPosts userId={123} />
      </Suspense>
      
      {/* Nested Suspense for comments */}
      <Suspense fallback={<CommentsSkeleton />}>
        <UserComments userId={123}>
          {/* Comments can have their own loading */}
          <Suspense fallback={<CommentSkeleton />}>
            <RecentComments />
          </Suspense>
        </UserComments>
      </Suspense>
    </div>
  );
}

// ADVANCED: Data Fetching with Suspense (React 18+)
// First, create a Suspense-compatible data source
function wrapPromise(promise) {
  let status = 'pending';
  let result;
  
  const suspender = promise.then(
    (r) => {
      status = 'success';
      result = r;
    },
    (e) => {
      status = 'error';
      result = e;
    }
  );
  
  return {
    read() {
      if (status === 'pending') throw suspender;
      if (status === 'error') throw result;
      return result;
    }
  };
}

// Custom hook for Suspense data fetching
function useSuspenseFetch(url) {
  const [resource] = useState(() => wrapPromise(fetch(url).then(r => r.json())));
  return resource.read(); // This will suspend if data isn't ready
}

// Component using Suspense for data
function UserData({ userId }) {
  // This line SUSPENDS until data is ready
  const userData = useSuspenseFetch(`/api/users/${userId}`);
  
  // No loading state needed in component!
  return (
    <div>
      <h1>{userData.name}</h1>
      <p>{userData.email}</p>
    </div>
  );
}

// Usage in App
function App() {
  return (
    <Suspense fallback={<GlobalLoader />}>
      <UserData userId={123} />
    </Suspense>
  );
}
```

---

## 4. **Portals** (Basic to Advanced)

### **Concept (70%)**
Portals provide a way to **render children into a DOM node that exists outside the parent component's DOM hierarchy**, while keeping them in the React component tree.

**Why Use Portals:**
- **Modal dialogs**: Escape parent overflow/transform styles
- **Tooltips/popovers**: Position relative to viewport
- **Global elements**: Notifications, loaders, sidebars
- **Break out of CSS containers**: Override `z-index`, `overflow: hidden`

**Key Characteristics:**
- Portal children remain in React tree (context, events bubble up)
- Rendered to different DOM location
- Useful for visual "escape hatches"
- Event bubbling works through portals (events fire from portal target, bubble to React parents)

### **Code (30%)**

```jsx
// BASIC: Modal with Portal
import ReactDOM from 'react-dom';

function Modal({ children, isOpen, onClose }) {
  if (!isOpen) return null;
  
  return ReactDOM.createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.getElementById('modal-root') // Portal target
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <div className="app-container">
      <button onClick={() => setShowModal(true)}>Open Modal</button>
      
      <Modal isOpen={showModal} onClose={() => setShowModal(false)}>
        <h2>Modal Title</h2>
        <p>This renders in portal, outside app container!</p>
      </Modal>
    </div>
  );
}

// HTML needed
{/* 
<body>
  <div id="root"></div>
  <div id="modal-root"></div> <!-- Portal target -->
</body>
*/}

// INTERMEDIATE: Dynamic Portal Target
function SmartPortal({ children, targetId = 'portal-root' }) {
  const [targetElement, setTargetElement] = useState(null);
  
  useEffect(() => {
    // Find or create portal target
    let element = document.getElementById(targetId);
    
    if (!element) {
      element = document.createElement('div');
      element.id = targetId;
      document.body.appendChild(element);
    }
    
    setTargetElement(element);
    
    // Cleanup
    return () => {
      if (element && element.parentNode) {
        element.parentNode.removeChild(element);
      }
    };
  }, [targetId]);
  
  if (!targetElement) return null;
  
  return ReactDOM.createPortal(children, targetElement);
}

// ADVANCED: Context-Aware Portal with Event Handling
const PortalContext = React.createContext();

function PortalProvider({ children }) {
  const [portals, setPortals] = useState(new Map());
  
  const addPortal = useCallback((id, element) => {
    setPortals(prev => new Map(prev).set(id, element));
  }, []);
  
  const removePortal = useCallback((id) => {
    setPortals(prev => {
      const next = new Map(prev);
      next.delete(id);
      return next;
    });
  }, []);
  
  return (
    <PortalContext.Provider value={{ portals, addPortal, removePortal }}>
      {children}
      {/* Render all portals */}
      {Array.from(portals.entries()).map(([id, children]) => (
        <React.Fragment key={id}>
          {ReactDOM.createPortal(children, document.getElementById(id))}
        </React.Fragment>
      ))}
    </PortalContext.Provider>
  );
}

function ContextAwarePortal({ id, children }) {
  const { addPortal, removePortal } = useContext(PortalContext);
  
  useEffect(() => {
    addPortal(id, children);
    return () => removePortal(id);
  }, [id, children, addPortal, removePortal]);
  
  return null; // This component doesn't render anything itself
}

// Tooltip using portal with click-outside detection
function Tooltip({ content, children }) {
  const [isVisible, setIsVisible] = useState(false);
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const triggerRef = useRef(null);
  
  const updatePosition = useCallback(() => {
    if (triggerRef.current) {
      const rect = triggerRef.current.getBoundingClientRect();
      setPosition({
        x: rect.left + rect.width / 2,
        y: rect.top - 10 // Position above
      });
    }
  }, []);
  
  useEffect(() => {
    if (isVisible) {
      updatePosition();
      window.addEventListener('resize', updatePosition);
      window.addEventListener('scroll', updatePosition);
      
      // Click outside to close
      const handleClickOutside = (e) => {
        if (!triggerRef.current?.contains(e.target)) {
          setIsVisible(false);
        }
      };
      document.addEventListener('mousedown', handleClickOutside);
      
      return () => {
        window.removeEventListener('resize', updatePosition);
        window.removeEventListener('scroll', updatePosition);
        document.removeEventListener('mousedown', handleClickOutside);
      };
    }
  }, [isVisible, updatePosition]);
  
  return (
    <>
      <span 
        ref={triggerRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
        onClick={() => setIsVisible(!isVisible)}
      >
        {children}
      </span>
      
      {isVisible && (
        <Portal targetId="tooltip-root">
          <div 
            className="tooltip"
            style={{
              position: 'fixed',
              left: `${position.x}px`,
              top: `${position.y}px`,
              transform: 'translateX(-50%)',
              zIndex: 9999
            }}
          >
            {content}
          </div>
        </Portal>
      )}
    </>
  );
}
```

---

## **Integration Example: Using All Features Together**

```jsx
function AdvancedApp() {
  return (
    <ErrorBoundary>
      <PortalProvider>
        <Suspense fallback={<AppLoader />}>
          <Dashboard>
            {/* Server data with Suspense */}
            <Suspense fallback={<UserCardSkeleton />}>
              <UserProfile userId={123} />
            </Suspense>
            
            {/* Modal using Portal */}
            <UserSettingsModal />
            
            {/* Tooltip with Portal */}
            <Tooltip content="This is a helpful tip">
              <button>Hover me</button>
            </Tooltip>
            
            {/* Component that might error */}
            <ErrorBoundary fallback={<ErrorMessage />}>
              <UnstableFeature />
            </ErrorBoundary>
          </Dashboard>
        </Suspense>
      </PortalProvider>
    </ErrorBoundary>
  );
}
```

## **Progression Summary**

1. **Start with basics**: Implement simple Error Boundaries and basic data fetching
2. **Add Portals**: For modals and global UI elements
3. **Introduce Suspense**: For code splitting and better loading states
4. **Advanced patterns**: Combine all features with sophisticated error recovery, caching, and concurrent rendering

These features represent React's evolution toward more declarative, resilient, and performant applications. Mastering them allows you to build production-ready applications with excellent user experiences.