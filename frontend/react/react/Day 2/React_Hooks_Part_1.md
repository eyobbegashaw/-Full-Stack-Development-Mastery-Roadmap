# **📌 Creating Custom Hooks - The Short Guide**

## **What are Custom Hooks?**
Custom Hooks are **JavaScript functions starting with "use"** that can call other hooks. They're a way to **extract and reuse stateful logic** between components.

### **Simple Definition:**
> "A custom hook is a JavaScript function that uses React hooks inside it, allowing you to extract component logic into reusable functions."

---

## **🌟 Why Create Custom Hooks?**
1. **DRY Principle** - Don't Repeat Yourself
2. **Clean Components** - Move logic out of components
3. **Share Logic** - Use same logic across multiple components
4. **Testable** - Test logic separately from UI

---

## **🔧 How to Create Custom Hooks**

### **Pattern 1: Extract Existing Logic**
```jsx
// BEFORE: Logic inside component
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json())
      .then(data => setUser(data))
      .finally(() => setLoading(false));
  }, []);
  
  // ... rest of component
}

// AFTER: Custom Hook
function useUserData(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(`/api/user/${userId}`)
      .then(res => res.json())
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading };
}

// Usage
function UserProfile({ userId }) {
  const { user, loading } = useUserData(userId);
  // Clean component!
}
```

### **Pattern 2: Combine Multiple Hooks**
```jsx
// Custom hook combining useState and localStorage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  
  return [value, setValue];
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  // theme persists in localStorage!
}
```

---

## **📋 Custom Hook Rules**

### **Must Follow:**
1. ✅ **Name must start with `use`** (React convention)
2. ✅ **Can call other hooks** (useState, useEffect, etc.)
3. ✅ **Return values** (state, functions, whatever needed)

### **Must NOT:**
1. ❌ **Call inside loops/conditions** (same as all hooks)
2. ❌ **Use in regular JavaScript functions** (only in components/hooks)
3. ❌ **Return JSX** (that's what components do)

---

# **🎯 Hooks Best Practices - Short Checklist**

## **1. Hook Dependency Arrays**
```jsx
// ✅ GOOD: Complete dependencies
useEffect(() => {
  doSomething(a, b);
}, [a, b]); // All variables used

// ❌ BAD: Missing dependencies (causes bugs!)
useEffect(() => {
  doSomething(a, b);
}, [a]); // Missing b

// ✅ Use exhaustive-deps ESLint rule
```

## **2. Performance Optimizations**
```jsx
// Only when needed:
const memoizedValue = useMemo(() => 
  expensiveCalculation(a, b), 
  [a, b] // Recompute when a or b changes
);

const memoizedFunction = useCallback(() => {
  doSomething(a, b);
}, [a, b]); // New function only when a/b changes
```

## **3. useState vs useReducer**
```jsx
// ✅ useState for simple state
const [count, setCount] = useState(0);

// ✅ useReducer for complex state logic
const [state, dispatch] = useReducer(reducer, initialState);
// When: multiple sub-values, complex transitions
```

## **4. useEffect Cleanup**
```jsx
useEffect(() => {
  const subscription = api.subscribe();
  
  // ✅ Always cleanup side effects
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

## **5. Custom Hook Design**
```jsx
// ✅ Good: Returns needed values
function useTimer(initialTime = 0) {
  const [time, setTime] = useState(initialTime);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    let interval;
    if (isRunning) {
      interval = setInterval(() => {
        setTime(t => t + 1);
      }, 1000);
    }
    return () => clearInterval(interval);
  }, [isRunning]);
  
  return { 
    time, 
    isRunning, 
    start: () => setIsRunning(true),
    pause: () => setIsRunning(false),
    reset: () => setTime(initialTime)
  };
}
```

---

## **🚀 Quick Best Practices List**

### **DO:**
1. **Keep hooks at top level** (no conditionals/loops)
2. **Use exhaustive dependency arrays**
3. **Extract logic to custom hooks early**
4. **Clean up side effects in useEffect**
5. **Use useMemo/useCallback only when needed**
6. **Name custom hooks starting with "use"**
7. **One effect per concern** (separate concerns)

### **DON'T:**
1. **Call hooks conditionally**
2. **Overuse useMemo/useCallback** (profile first!)
3. **Forget cleanup functions**
4. **Create huge custom hooks** (split them up)
5. **Return JSX from custom hooks**

---

## **💡 Pro Tips**

### **Tip 1: Hook Composition**
```jsx
// Build complex hooks from simple ones
function useUserDashboard(userId) {
  const user = useUser(userId);
  const posts = useUserPosts(userId);
  const notifications = useNotifications(userId);
  
  return { user, posts, notifications };
}
```

### **Tip 2: Testing Custom Hooks**
```jsx
// Test hooks with @testing-library/react-hooks
import { renderHook } from '@testing-library/react-hooks';

test('should use counter', () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0);
});
```

### **Tip 3: Share via npm**
```json
{
  "name": "use-local-storage",
  "main": "index.js"
}
// Package reusable hooks for team/community
```

---

## **🎯 One-Line Summary:**

**Custom Hooks**: Extract component logic into reusable functions starting with `"use"`

**Best Practices**: Follow hook rules, optimize wisely, clean up effects, and compose hooks for clean, maintainable code!

**Remember**: Custom hooks make your code **DRY**, **testable**, and **beautiful**! 🚀