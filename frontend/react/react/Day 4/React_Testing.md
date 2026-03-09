# **🧪 React Testing - Complete Guide**

## **🎯 Understanding Testing in React**

### **Concept (70%)**
Testing ensures your React application **works correctly, reliably, and predictably**. It's about preventing bugs, enabling refactoring, and documenting how components should work.

**The Testing Pyramid:**
```
     Unit Tests (Most)
        /      \
 Integration  E2E Tests (Least)
```
- **Unit Tests**: Test individual components/functions in isolation
- **Integration Tests**: Test component interactions
- **E2E Tests**: Test complete user flows

**Why Test React Apps?**
1. **Catch bugs early** - Before they reach users
2. **Enable refactoring** - Change code with confidence
3. **Document behavior** - Tests describe how code should work
4. **Improve design** - Testable code is usually better designed

---

## **1️⃣ Jest - The Test Runner**

### **Concept (70%)**
Jest is a **JavaScript testing framework** that provides test runners, assertion libraries, and mocking capabilities. It's the **default choice** for React testing.

**Jest Features:**
- 🏃 **Test runner** - Runs tests, reports results
- ✅ **Assertions** - `expect()`, `toBe()`, `toEqual()`
- 🎭 **Mocking** - Functions, modules, timers
- 📊 **Coverage** - Code coverage reports
- ⚡ **Fast** - Parallel test execution
- 🔄 **Watch mode** - Run tests on file changes

**Jest vs Vitest:**
- **Jest**: Mature, widely adopted, Facebook-backed
- **Vitest**: Faster, native ES modules, Vite ecosystem

### **Code (30%)**
```javascript
// math.js - Function to test
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// math.test.js - Jest test file
import { add, multiply } from './math';

// Basic test
describe('Math functions', () => {
  test('adds two numbers', () => {
    expect(add(1, 2)).toBe(3);
    expect(add(-1, 1)).toBe(0);
  });

  test('multiplies two numbers', () => {
    expect(multiply(2, 3)).toBe(6);
    expect(multiply(0, 5)).toBe(0);
  });
});

// Mocking examples
describe('API calls', () => {
  // Mock a function
  const fetchData = jest.fn();
  
  beforeEach(() => {
    fetchData.mockClear();
  });

  test('calls API once', async () => {
    fetchData.mockResolvedValue({ data: 'test' });
    
    await fetchData();
    
    expect(fetchData).toHaveBeenCalledTimes(1);
  });

  test('returns mocked data', async () => {
    fetchData.mockResolvedValue({ data: 'mocked' });
    
    const result = await fetchData();
    
    expect(result).toEqual({ data: 'mocked' });
  });
});

// Async testing
describe('Async operations', () => {
  test('fetches user data', async () => {
    const user = await fetchUser(1);
    expect(user.id).toBe(1);
    expect(user.name).toBeDefined();
  });

  test('handles fetch errors', async () => {
    await expect(fetchInvalidUser()).rejects.toThrow('User not found');
  });
});

// Snapshot testing
describe('Component snapshot', () => {
  test('matches snapshot', () => {
    const component = renderer.create(<Button>Click me</Button>);
    const tree = component.toJSON();
    expect(tree).toMatchSnapshot();
  });
});
```

---

## **2️⃣ Vitest - The Modern Alternative**

### **Concept (70%)**
Vitest is a **blazing fast unit test framework** built on top of Vite. It's designed for **modern JavaScript** with native ESM and TypeScript support.

**Why Vitest?**
- ⚡ **Extremely fast** - Uses Vite's dev server
- 🔧 **Vite ecosystem** - Same config, same plugins
- 📦 **ESM first** - Native ES modules support
- 🎯 **Jest compatible** - Same API, easier migration
- 🔍 **Watch mode** - Instant feedback

**Vitest vs Jest:**
- **Speed**: Vitest is faster, especially in watch mode
- **Config**: Vitest uses Vite config, Jest has own config
- **ESM**: Vitest has better ESM support
- **Ecosystem**: Jest has more maturing plugins

### **Code (30%)**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.js'],
    coverage: {
      provider: 'istanbul',
      reporter: ['text', 'json', 'html'],
    },
  },
});

// calculator.test.js
import { describe, test, expect, vi } from 'vitest';
import { calculateTotal } from './calculator';

describe('Calculator', () => {
  test('calculates total with tax', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 1 },
    ];
    
    expect(calculateTotal(items, 0.1)).toBe(27.5); // (20 + 5) * 1.1
  });

  test('handles empty cart', () => {
    expect(calculateTotal([], 0.1)).toBe(0);
  });
});

// Mocking with vi (Vitest's jest.fn equivalent)
describe('API service', () => {
  const fetchUsers = vi.fn();
  
  beforeEach(() => {
    vi.clearAllMocks();
  });

  test('fetches users', async () => {
    fetchUsers.mockResolvedValue([{ id: 1, name: 'John' }]);
    
    const users = await fetchUsers();
    
    expect(fetchUsers).toHaveBeenCalled();
    expect(users).toHaveLength(1);
  });

  // Mock timers
  test('debounces search', () => {
    vi.useFakeTimers();
    
    const search = vi.fn();
    const debouncedSearch = debounce(search, 100);
    
    debouncedSearch('test');
    debouncedSearch('test');
    
    vi.advanceTimersByTime(100);
    
    expect(search).toHaveBeenCalledTimes(1);
    
    vi.useRealTimers();
  });
});

// Component testing with happy-dom or jsdom
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Button component', () => {
  test('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  test('handles click', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    await userEvent.click(screen.getByText('Click'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

---

## **3️⃣ React Testing Library (RTL) - Component Testing**

### **Concept (70%)**
React Testing Library is a **testing utility** that encourages testing components the way users interact with them. It's about testing **behavior, not implementation**.

**RTL Philosophy:**
> "The more your tests resemble the way your software is used, the more confidence they can give you."

**Core Principles:**
1. **Test behavior, not implementation**
2. **Find elements by accessibility roles**
3. **Interact like a user would** (clicks, typing)
4. **Avoid testing implementation details**

**Query Priority (What to use first):**
1. `getByRole` - Most accessible
2. `getByLabelText` - Forms
3. `getByPlaceholderText` - Inputs
4. `getByText` - Content
5. `getByDisplayValue` - Form values
6. `getByAltText` - Images
7. `getByTitle` - Titles
8. `getByTestId` - Last resort

### **Code (30%)**
```jsx
// Button.jsx
function Button({ onClick, children, disabled = false }) {
  return (
    <button 
      onClick={onClick}
      disabled={disabled}
      aria-label={typeof children === 'string' ? children : undefined}
    >
      {children}
    </button>
  );
}

// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

describe('Button Component', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    
    // Best: Find by role
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });

  test('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    const button = screen.getByRole('button');
    
    // fireEvent (basic) vs userEvent (more realistic)
    fireEvent.click(button);
    expect(handleClick).toHaveBeenCalledTimes(1);
    
    // userEvent is preferred - simulates actual user interaction
    await userEvent.click(button);
    expect(handleClick).toHaveBeenCalledTimes(2);
  });

  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  test('does not call onClick when disabled', async () => {
    const handleClick = jest.fn();
    render(
      <Button onClick={handleClick} disabled>
        Click
      </Button>
    );
    
    const button = screen.getByRole('button');
    await userEvent.click(button);
    
    expect(handleClick).not.toHaveBeenCalled();
  });
});

// Form testing
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      onSubmit({ email, password });
    }}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter email"
      />
      
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Enter password"
      />
      
      <button type="submit">Login</button>
    </form>
  );
}

// LoginForm.test.jsx
describe('LoginForm', () => {
  test('submits form with email and password', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    // Find inputs by their labels (most accessible)
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /login/i });
    
    // Type like a user
    await userEvent.type(emailInput, 'test@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);
    
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });

  test('shows validation for empty fields', async () => {
    // Test with empty submission
  });
});

// Async component testing
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// UserProfile.test.jsx
describe('UserProfile', () => {
  test('shows loading then user data', async () => {
    // Mock the fetch
    const mockUser = { id: 1, name: 'John', email: 'john@test.com' };
    jest.spyOn(global, 'fetch').mockResolvedValue({
      json: () => Promise.resolve(mockUser),
    });
    
    render(<UserProfile userId={1} />);
    
    // Should show loading initially
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
    
    // Wait for user data
    const userName = await screen.findByText('John');
    expect(userName).toBeInTheDocument();
    
    // Email should also be visible
    expect(screen.getByText('john@test.com')).toBeInTheDocument();
    
    // Loading should be gone
    expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
  });
});

// Testing custom hooks
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  test('increments counter', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

---

## **4️⃣ Cypress - E2E Testing**

### **Concept (70%)**
Cypress is an **end-to-end testing framework** that runs tests in a real browser. It tests complete user workflows from start to finish.

**Cypress Features:**
- 🌐 **Real browser** - Runs in Chrome, Firefox, Edge
- 🎥 **Time travel** - Debug with snapshots
- 🔍 **Automatic waiting** - No more async hell
- 📹 **Video recording** - Record test runs
- 📸 **Screenshot capture** - On failures
- 🔧 **Dev tools** - Integrated debugging

**When to use Cypress:**
- Critical user journeys
- Testing across multiple pages
- Testing with real backend
- Visual regression testing

### **Code (30%)**
```javascript
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.js',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
  },
});

// cypress/e2e/auth.cy.js
describe('Authentication', () => {
  beforeEach(() => {
    // Reset state before each test
    cy.clearCookies();
    cy.clearLocalStorage();
    cy.visit('/');
  });

  it('logs in successfully', () => {
    cy.visit('/login');
    
    // Fill form
    cy.get('[data-testid="email-input"]')
      .type('user@example.com')
      .should('have.value', 'user@example.com');
    
    cy.get('[data-testid="password-input"]')
      .type('password123')
      .should('have.value', 'password123');
    
    // Submit form
    cy.get('[data-testid="login-button"]').click();
    
    // Assertions
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="welcome-message"]')
      .should('contain', 'Welcome back');
    
    // Check localStorage
    cy.window().its('localStorage.token').should('exist');
  });

  it('shows error on invalid login', () => {
    cy.visit('/login');
    
    cy.get('[data-testid="email-input"]').type('wrong@email.com');
    cy.get('[data-testid="password-input"]').type('wrong');
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
  });

  it('can log out', () => {
    // Login first
    cy.login('user@example.com', 'password123');
    
    // Click logout
    cy.get('[data-testid="user-menu"]').click();
    cy.get('[data-testid="logout-button"]').click();
    
    // Should be redirected to login
    cy.url().should('include', '/login');
    cy.window().its('localStorage.token').should('not.exist');
  });
});

// cypress/e2e/shopping.cy.js
describe('Shopping Cart', () => {
  it('adds items to cart', () => {
    cy.visit('/products');
    
    // Add first product to cart
    cy.get('[data-testid="product-card"]')
      .first()
      .within(() => {
        cy.get('[data-testid="add-to-cart"]').click();
      });
    
    // Cart badge should update
    cy.get('[data-testid="cart-count"]').should('contain', '1');
    
    // Go to cart
    cy.get('[data-testid="cart-icon"]').click();
    
    // Verify item in cart
    cy.get('[data-testid="cart-item"]').should('have.length', 1);
    cy.contains('Proceed to Checkout').click();
  });

  it('completes checkout flow', () => {
    // Add items to cart
    cy.addToCart(1); // Custom command
    cy.addToCart(2);
    
    cy.visit('/checkout');
    
    // Fill shipping info
    cy.get('[name="fullName"]').type('John Doe');
    cy.get('[name="address"]').type('123 Main St');
    cy.get('[name="city"]').type('New York');
    
    // Select payment
    cy.get('[data-testid="credit-card"]').click();
    cy.get('[name="cardNumber"]').type('4111111111111111');
    
    // Place order
    cy.get('[data-testid="place-order"]').click();
    
    // Success page
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-testid="success-message"]').should('be.visible');
  });
});

// Custom commands (cypress/support/commands.js)
Cypress.Commands.add('login', (email, password) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(email);
    cy.get('[data-testid="password-input"]').type(password);
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('addToCart', (productId) => {
  cy.request('POST', '/api/cart', { productId });
});

// Component testing in Cypress (for isolated component tests)
describe('Button.cy.jsx', () => {
  it('plays button', () => {
    cy.mount(<Button>Click me</Button>);
    cy.get('button').should('contain', 'Click me');
  });
});
```

---

## **5️⃣ Playwright - The Modern E2E Alternative**

### **Concept (70%)**
Playwright is a **cross-browser testing framework** from Microsoft that supports Chromium, Firefox, and WebKit. It's **faster than Cypress** and supports multiple tabs/windows.

**Playwright Advantages:**
- 🚀 **Faster execution** - Parallel by default
- 🌐 **Multi-browser** - Chrome, Firefox, Safari
- 🎭 **Multi-context** - Test multiple tabs/sessions
- 📱 **Mobile emulation** - Test responsive designs
- 🌍 **Cross-platform** - Windows, macOS, Linux
- 📦 **Auto-wait** - Smarter than Cypress's waiting

**Playwright vs Cypress:**
- **Speed**: Playwright is faster
- **Browsers**: Playwright supports all major browsers
- **API**: Playwright has more low-level control
- **Community**: Cypress has larger community

### **Code (30%)**
```javascript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true, // Run tests in parallel
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    // Mobile testing
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
});

// tests/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('successful login', async ({ page }) => {
    await page.goto('/login');
    
    // Fill form using locators
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    
    // Click and wait for navigation
    await page.getByRole('button', { name: 'Login' }).click();
    await page.waitForURL('**/dashboard');
    
    // Assertions
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome back')).toBeVisible();
    
    // Check localStorage
    const token = await page.evaluate(() => localStorage.getItem('token'));
    expect(token).toBeTruthy();
  });

  test('login validation errors', async ({ page }) => {
    await page.goto('/login');
    
    // Try to submit empty form
    await page.getByRole('button', { name: 'Login' }).click();
    
    // Should show validation errors
    await expect(page.getByText('Email is required')).toBeVisible();
    await expect(page.getByText('Password is required')).toBeVisible();
  });

  test('login with invalid credentials', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('wrong@email.com');
    await page.getByLabel('Password').fill('wrong');
    await page.getByRole('button', { name: 'Login' }).click();
    
    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });
});

// tests/cart.spec.ts
test.describe('Shopping Cart', () => {
  test('add to cart and checkout', async ({ page }) => {
    // Add items using API for faster setup
    await page.request.post('/api/cart', {
      data: { productId: 1, quantity: 2 }
    });
    
    await page.goto('/cart');
    
    // Verify cart contents
    await expect(page.getByText('Cart (2 items)')).toBeVisible();
    
    // Update quantity
    const quantityInput = page.getByLabel('Quantity');
    await quantityInput.fill('3');
    await expect(quantityInput).toHaveValue('3');
    
    // Proceed to checkout
    await page.getByRole('button', { name: 'Checkout' }).click();
    await page.waitForURL('**/checkout');
    
    // Fill shipping info
    await page.getByLabel('Full Name').fill('John Doe');
    await page.getByLabel('Address').fill('123 Main St');
    
    // Select payment method
    await page.getByText('Credit Card').click();
    await page.frameLocator('iframe[name="card-frame"]')
      .getByPlaceholder('Card number')
      .fill('4242424242424242');
    
    // Place order
    await page.getByRole('button', { name: 'Place Order' }).click();
    
    // Verify success
    await expect(page.getByText('Order confirmed!')).toBeVisible();
    await expect(page).toHaveURL(/order-confirmation/);
  });
});

// tests/mobile.spec.ts - Responsive testing
test.describe('Mobile responsiveness', () => {
  test('mobile navigation menu', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    
    await page.goto('/');
    
    // Hamburger menu should be visible on mobile
    await expect(page.getByRole('button', { name: 'Menu' })).toBeVisible();
    
    // Click hamburger menu
    await page.getByRole('button', { name: 'Menu' }).click();
    
    // Menu should expand
    await expect(page.getByRole('navigation')).toBeVisible();
    
    // Desktop navigation should be hidden
    await expect(page.getByRole('link', { name: 'Home' })).toBeHidden();
  });
});

// tests/api.spec.ts - API testing
test.describe('API endpoints', () => {
  test('GET /api/products', async ({ request }) => {
    const response = await request.get('/api/products');
    
    expect(response.ok()).toBeTruthy();
    
    const products = await response.json();
    expect(Array.isArray(products)).toBeTruthy();
    expect(products.length).toBeGreaterThan(0);
    
    // Validate product structure
    products.forEach(product => {
      expect(product).toHaveProperty('id');
      expect(product).toHaveProperty('name');
      expect(product).toHaveProperty('price');
    });
  });

  test('POST /api/cart requires authentication', async ({ request }) => {
    const response = await request.post('/api/cart', {
      data: { productId: 1 }
    });
    
    expect(response.status()).toBe(401);
  });
});

// tests/visual.spec.ts - Visual testing
test.describe('Visual regression', () => {
  test('homepage matches screenshot', async ({ page }) => {
    await page.goto('/');
    
    // Take screenshot and compare
    expect(await page.screenshot()).toMatchSnapshot('homepage.png');
  });

  test('product page matches screenshot', async ({ page }) => {
    await page.goto('/products/1');
    
    // Full page screenshot
    await expect(page).toHaveScreenshot('product-page.png', {
      fullPage: true,
      animations: 'disabled', // Disable animations for consistent screenshots
    });
  });
});
```

---

 

**Remember**: Good tests give you **confidence to refactor** and **prevent regressions**. Write tests that **fail when behavior changes**, not when implementation changes! 🚀