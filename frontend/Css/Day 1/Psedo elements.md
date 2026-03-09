# **COMPLETE GUIDE: PSEUDO CLASSES & PSEUDO ELEMENTS**
 

 

### **Pseudo Elements**
- **Double colon** (`::`) - CSS3 standard (single colon works for backward compatibility)
- **Create virtual elements**
- Style **specific parts** of elements
- Examples: `::before`, `::after`, `::first-line`

---
 

## **PART 2: PSEUDO ELEMENTS**

### **1. `::before` & `::after`**
Create virtual elements before/after content.

```css
.element::before {
  content: ""; /* REQUIRED - can be empty string */
  display: block;
  position: absolute;
  /* Other properties */
}

.element::after {
  content: "After content";
  /* Other properties */
}
```

### **Common Patterns with ::before/::after**

#### **Icons & Decorations**
```css
/* Add icon before link */
.external-link::before {
  content: "↗";
  margin-right: 5px;
  font-weight: bold;
}

/* Decorative quotes */
blockquote::before {
  content: open-quote;
  font-size: 3em;
  color: #3498db;
  line-height: 0.1em;
  vertical-align: -0.4em;
}

blockquote::after {
  content: close-quote;
  font-size: 3em;
  color: #3498db;
  line-height: 0.1em;
  vertical-align: -0.4em;
}
```

#### **Custom Bullets**
```css
.custom-list li::before {
  content: "•";
  color: #3498db;
  font-weight: bold;
  display: inline-block;
  width: 1em;
  margin-left: -1em;
}

.checklist li::before {
  content: "✓";
  color: #2ecc71;
  font-weight: bold;
  margin-right: 10px;
}
```

#### **Tooltips**
```css
.tooltip {
  position: relative;
}

.tooltip:hover::after {
  content: attr(data-tooltip);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  white-space: nowrap;
  font-size: 0.875em;
}
```

#### **Overlay Effects**
```css
.image-container {
  position: relative;
}

.image-container::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(to bottom, transparent, rgba(0,0,0,0.7));
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-container:hover::after {
  opacity: 1;
}
```

#### **Ribbons & Badges**
```css
.product-card {
  position: relative;
  overflow: hidden;
}

.product-card::before {
  content: "SALE";
  position: absolute;
  top: 10px;
  right: -30px;
  background: #e74c3c;
  color: white;
  padding: 5px 30px;
  transform: rotate(45deg);
  font-weight: bold;
  z-index: 1;
}
```

### **2. `::first-letter`**
Style the first letter of a block.

```css
p::first-letter {
  font-size: 3em;
  float: left;
  line-height: 1;
  margin-right: 10px;
  color: #3498db;
  font-weight: bold;
}

/* Drop cap effect */
.drop-cap::first-letter {
  font-size: 5em;
  float: left;
  line-height: 0.8;
  margin: 0.1em 0.1em 0 0;
  padding: 0.1em;
  background: #3498db;
  color: white;
  border-radius: 5px;
}
```

### **3. `::first-line`**
Style the first line of a block.

```css
p::first-line {
  font-weight: bold;
  font-size: 1.1em;
  letter-spacing: 0.5px;
}

article::first-line {
  color: #2c3e50;
  text-transform: uppercase;
  font-variant: small-caps;
}
```

### **4. `::selection`**
Style user-selected text.

```css
::selection {
  background-color: #3498db;
  color: white;
}

/* Specific elements */
p::selection {
  background-color: #2ecc71;
  color: white;
}

/* Remove selection styling */
.no-select::selection {
  background-color: transparent;
  color: inherit;
}
```

### **5. `::placeholder`**
Style placeholder text in inputs.

```css
input::placeholder {
  color: #95a5a6;
  font-style: italic;
}

/* Focus state */
input:focus::placeholder {
  color: #bdc3c7;
}

/* Different browsers */
input::-webkit-input-placeholder { /* Chrome/Safari */
  color: #95a5a6;
}

input::-moz-placeholder { /* Firefox */
  color: #95a5a6;
  opacity: 1;
}

input:-ms-input-placeholder { /* IE/Edge */
  color: #95a5a6;
}
```

### **6. `::marker`**
Style list item markers (bullets/numbers).

```css
/* Custom bullet color */
li::marker {
  color: #3498db;
  font-size: 1.2em;
}

/* Ordered list markers */
ol li::marker {
  font-weight: bold;
  content: counter(list-item) ".";
}

/* Custom bullet shape */
.custom-list li::marker {
  content: "▸";
  color: #e74c3c;
}
```

### **7. `::backdrop`**
Style backdrop of dialog/modal elements.

```css
dialog::backdrop {
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(5px);
}

/* Multiple backdrops with different opacity */
.dialog-light::backdrop {
  background: rgba(255, 255, 255, 0.9);
}
```

### **8. `::file-selector-button`**
Style file input button.

```css
input[type="file"]::file-selector-button {
  background: #3498db;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  margin-right: 10px;
}

input[type="file"]::file-selector-button:hover {
  background: #2980b9;
}
```

---

## **COMPREHENSIVE EXAMPLES**

### **Example 1: Interactive Button System**
```html
<style>
  .btn {
    position: relative;
    display: inline-block;
    padding: 12px 24px;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 16px;
    cursor: pointer;
    overflow: hidden;
    transition: all 0.3s ease;
  }
  
  /* Ripple effect with ::after */
  .btn::after {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    width: 5px;
    height: 5px;
    background: rgba(255, 255, 255, 0.5);
    opacity: 0;
    border-radius: 100%;
    transform: scale(1, 1) translate(-50%);
    transform-origin: 50% 50%;
  }
  
  .btn:focus:not(:active)::after {
    animation: ripple 1s ease-out;
  }
  
  @keyframes ripple {
    0% {
      transform: scale(0, 0);
      opacity: 0.5;
    }
    100% {
      transform: scale(20, 20);
      opacity: 0;
    }
  }
  
  /* Hover effect */
  .btn:hover {
    background: #2980b9;
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(52, 152, 219, 0.3);
  }
  
  /* Active/click effect */
  .btn:active {
    transform: translateY(1px);
    box-shadow: 0 2px 5px rgba(52, 152, 219, 0.3);
  }
  
  /* Focus state */
  .btn:focus {
    outline: none;
    box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.5);
  }
  
  /* Disabled state */
  .btn:disabled {
    background: #bdc3c7;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
  }
  
  .btn:disabled:hover {
    background: #bdc3c7;
    transform: none;
  }
</style>

<button class="btn">Click Me</button>
<button class="btn" disabled>Disabled</button>
```

### **Example 2: Custom Checkbox & Radio**
```html
<style>
  .custom-checkbox {
    position: relative;
    padding-left: 35px;
    cursor: pointer;
    display: inline-block;
  }
  
  /* Hide default checkbox */
  .custom-checkbox input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    height: 0;
    width: 0;
  }
  
  /* Custom checkbox */
  .custom-checkbox .checkmark {
    position: absolute;
    top: 0;
    left: 0;
    height: 25px;
    width: 25px;
    background-color: #eee;
    border-radius: 4px;
    transition: background-color 0.3s;
  }
  
  /* Hover state */
  .custom-checkbox:hover input ~ .checkmark {
    background-color: #ddd;
  }
  
  /* Checked state */
  .custom-checkbox input:checked ~ .checkmark {
    background-color: #2ecc71;
  }
  
  /* Checkmark using ::after */
  .custom-checkbox .checkmark::after {
    content: "";
    position: absolute;
    display: none;
    left: 9px;
    top: 5px;
    width: 5px;
    height: 10px;
    border: solid white;
    border-width: 0 3px 3px 0;
    transform: rotate(45deg);
  }
  
  /* Show checkmark when checked */
  .custom-checkbox input:checked ~ .checkmark::after {
    display: block;
  }
</style>

<label class="custom-checkbox">
  <input type="checkbox">
  <span class="checkmark"></span>
  Accept Terms
</label>
```

### **Example 3: Responsive Navigation with Pseudo Elements**
```html
<style>
  .nav-menu {
    display: flex;
    list-style: none;
    padding: 0;
    margin: 0;
    background: #2c3e50;
    border-radius: 8px;
    overflow: hidden;
  }
  
  .nav-item {
    position: relative;
    flex: 1;
  }
  
  .nav-link {
    display: block;
    padding: 15px 20px;
    color: white;
    text-decoration: none;
    text-align: center;
    transition: all 0.3s ease;
  }
  
  /* Underline effect with ::before */
  .nav-link::before {
    content: "";
    position: absolute;
    bottom: 0;
    left: 50%;
    width: 0;
    height: 3px;
    background: #3498db;
    transition: all 0.3s ease;
    transform: translateX(-50%);
  }
  
  .nav-link:hover::before {
    width: 80%;
  }
  
  .nav-link:hover {
    background: rgba(255, 255, 255, 0.1);
  }
  
  /* Active state indicator with ::after */
  .nav-item.active::after {
    content: "";
    position: absolute;
    top: 50%;
    right: 10px;
    width: 8px;
    height: 8px;
    background: #2ecc71;
    border-radius: 50%;
    transform: translateY(-50%);
    animation: pulse 2s infinite;
  }
  
  @keyframes pulse {
    0% { box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.7); }
    70% { box-shadow: 0 0 0 10px rgba(46, 204, 113, 0); }
    100% { box-shadow: 0 0 0 0 rgba(46, 204, 113, 0); }
  }
  
  /* Dropdown indicator */
  .nav-item.has-dropdown .nav-link::after {
    content: "▼";
    font-size: 0.7em;
    margin-left: 5px;
    opacity: 0.7;
  }
</style>

<ul class="nav-menu">
  <li class="nav-item active">
    <a href="#" class="nav-link">Home</a>
  </li>
  <li class="nav-item">
    <a href="#" class="nav-link">About</a>
  </li>
  <li class="nav-item has-dropdown">
    <a href="#" class="nav-link">Services</a>
  </li>
  <li class="nav-item">
    <a href="#" class="nav-link">Contact</a>
  </li>
</ul>
```

### **Example 4: Pricing Card with Pseudo Elements**
```html
<style>
  .pricing-card {
    position: relative;
    background: white;
    border-radius: 10px;
    padding: 30px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
    overflow: hidden;
  }
  
  /* Popular badge using ::before */
  .pricing-card.popular::before {
    content: "POPULAR";
    position: absolute;
    top: 20px;
    right: -30px;
    background: #e74c3c;
    color: white;
    padding: 5px 30px;
    transform: rotate(45deg);
    font-weight: bold;
    font-size: 0.8em;
    letter-spacing: 1px;
    z-index: 1;
  }
  
  /* Decorative corner using ::after */
  .pricing-card::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 0 60px 60px 0;
    border-color: transparent #f8f9fa transparent transparent;
    z-index: 0;
  }
  
  .pricing-card:hover {
    transform: translateY(-10px);
    box-shadow: 0 15px 30px rgba(0,0,0,0.15);
  }
  
  .pricing-header {
    position: relative;
    padding-bottom: 20px;
    margin-bottom: 20px;
    border-bottom: 2px solid #f0f0f0;
  }
  
  /* Price styling with ::first-letter */
  .price::first-letter {
    font-size: 0.5em;
    vertical-align: super;
    margin-right: 2px;
  }
  
  /* Feature list with checkmarks */
  .features {
    list-style: none;
    padding: 0;
    margin: 0 0 30px 0;
  }
  
  .features li {
    padding: 8px 0;
    position: relative;
    padding-left: 30px;
  }
  
  /* Checkmark using ::before */
  .features li::before {
    content: "✓";
    position: absolute;
    left: 0;
    color: #2ecc71;
    font-weight: bold;
  }
  
  /* Disabled features */
  .features li.disabled {
    color: #bdc3c7;
  }
  
  .features li.disabled::before {
    content: "✗";
    color: #e74c3c;
  }
  
  /* Hover effect on features */
  .features li:not(.disabled):hover {
    background: #f8f9fa;
    padding-left: 35px;
    transition: all 0.3s ease;
  }
  
  /* Button with icon using ::before */
  .pricing-button {
    display: inline-block;
    width: 100%;
    padding: 15px;
    background: #3498db;
    color: white;
    text-decoration: none;
    border-radius: 6px;
    text-align: center;
    font-weight: bold;
    position: relative;
    overflow: hidden;
    transition: background 0.3s ease;
  }
  
  .pricing-button::before {
    content: "→";
    position: absolute;
    right: 15px;
    opacity: 0;
    transition: all 0.3s ease;
  }
  
  .pricing-button:hover {
    background: #2980b9;
    padding-right: 40px;
  }
  
  .pricing-button:hover::before {
    opacity: 1;
    right: 20px;
  }
  
  /* Selected state */
  .pricing-card.selected {
    border: 2px solid #3498db;
  }
  
  .pricing-card.selected .pricing-button {
    background: #2ecc71;
  }
  
  .pricing-card.selected .pricing-button:hover {
    background: #27ae60;
  }
</style>

<div class="pricing-card popular">
  <div class="pricing-header">
    <h3>Professional Plan</h3>
    <div class="price">$29<span>/month</span></div>
  </div>
  <ul class="features">
    <li>10 Projects</li>
    <li>5GB Storage</li>
    <li>Priority Support</li>
    <li class="disabled">Advanced Analytics</li>
  </ul>
  <a href="#" class="pricing-button">Get Started</a>
</div>
```

### **Example 5: Form Validation with Pseudo Classes**
```html
<style>
  .form-group {
    margin-bottom: 20px;
    position: relative;
  }
  
  .form-label {
    display: block;
    margin-bottom: 5px;
    font-weight: 600;
  }
  
  .form-control {
    width: 100%;
    padding: 10px;
    border: 2px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
    transition: all 0.3s ease;
  }
  
  /* Focus state */
  .form-control:focus {
    border-color: #3498db;
    box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
    outline: none;
  }
  
  /* Valid state */
  .form-control:valid {
    border-color: #2ecc71;
    background-color: rgba(46, 204, 113, 0.05);
  }
  
  /* Invalid state */
  .form-control:invalid:not(:placeholder-shown) {
    border-color: #e74c3c;
    background-color: rgba(231, 76, 60, 0.05);
  }
  
  /* Required field indicator */
  .form-control:required + .form-label::after {
    content: " *";
    color: #e74c3c;
  }
  
  /* Validation message using ::after */
  .form-control:invalid:not(:placeholder-shown) + .error-message::after {
    content: attr(data-error);
    display: block;
    color: #e74c3c;
    font-size: 0.875em;
    margin-top: 5px;
  }
  
  /* Checkbox validation */
  .checkbox-group input:invalid ~ .checkbox-label::before {
    border-color: #e74c3c;
  }
  
  /* Password strength indicator */
  .password-strength::after {
    content: "";
    position: absolute;
    bottom: -5px;
    left: 0;
    width: 0;
    height: 3px;
    background: #e74c3c;
    transition: width 0.3s ease;
  }
  
  .password-strength.weak::after {
    width: 33%;
    background: #e74c3c;
  }
  
  .password-strength.medium::after {
    width: 66%;
    background: #f39c12;
  }
  
  .password-strength.strong::after {
    width: 100%;
    background: #2ecc71;
  }
</style>

<div class="form-group">
  <input type="email" class="form-control" placeholder="Email" required>
  <span class="error-message" data-error="Please enter a valid email"></span>
</div>

<div class="form-group">
  <input type="password" class="form-control password-strength weak" 
         placeholder="Password" pattern=".{8,}" required>
</div>
```

### **Example 6: Typography Effects**
```html
<style>
  /* Drop cap with ::first-letter */
  .drop-cap::first-letter {
    float: left;
    font-size: 4em;
    line-height: 0.8;
    margin: 0.1em 0.1em 0 0;
    padding: 0.1em;
    background: #2c3e50;
    color: white;
    border-radius: 5px;
  }
  
  /* First line styling */
  .article-intro::first-line {
    font-weight: bold;
    font-size: 1.2em;
    color: #2c3e50;
    letter-spacing: 0.5px;
  }
  
  /* Highlighted text selection */
  .highlightable::selection {
    background: #3498db;
    color: white;
  }
  
  /* Pull quote using ::before and ::after */
  .pull-quote {
    position: relative;
    padding: 20px;
    margin: 30px 0;
    background: #f8f9fa;
    border-left: 4px solid #3498db;
    font-style: italic;
  }
  
  .pull-quote::before {
    content: open-quote;
    font-size: 3em;
    color: #3498db;
    position: absolute;
    top: -10px;
    left: 10px;
    line-height: 1;
  }
  
  .pull-quote::after {
    content: close-quote;
    font-size: 3em;
    color: #3498db;
    position: absolute;
    bottom: -20px;
    right: 10px;
    line-height: 1;
  }
  
  /* Link with icon */
  .external-link::after {
    content: " ↗";
    font-size: 0.8em;
    opacity: 0.7;
    transition: opacity 0.3s ease;
  }
  
  .external-link:hover::after {
    opacity: 1;
  }
  
  /* List with custom markers */
  .fancy-list {
    list-style: none;
    padding-left: 0;
  }
  
  .fancy-list li {
    padding: 10px 0;
    padding-left: 30px;
    position: relative;
    border-bottom: 1px solid #eee;
  }
  
  .fancy-list li::before {
    content: "▸";
    position: absolute;
    left: 0;
    color: #3498db;
    font-weight: bold;
    transition: transform 0.3s ease;
  }
  
  .fancy-list li:hover::before {
    transform: translateX(5px);
    color: #e74c3c;
  }
</style>

<p class="drop-cap">This is a paragraph with a drop cap effect. The first letter is styled differently to create visual interest.</p>

<p class="article-intro">This is the first line of an article. It will appear bold and slightly larger than the rest of the text. The rest of the paragraph continues normally with regular styling.</p>

<blockquote class="pull-quote">
  The only way to do great work is to love what you do.
</blockquote>

<p class="highlightable">Try selecting this text to see custom selection colors.</p>

<a href="#" class="external-link">Visit our website</a>

<ul class="fancy-list">
  <li>First item with custom marker</li>
  <li>Second item with custom marker</li>
  <li>Third item with custom marker</li>
</ul>
```

## **KEY DIFFERENCES & BEST PRACTICES**

### **Pseudo Classes vs Pseudo Elements:**
- **Pseudo Classes** select existing elements in certain states
- **Pseudo Elements** create new virtual elements
- Use `:` for pseudo classes, `::` for pseudo elements (CSS3+)

### **When to Use Which:**

| Use Case | Use |
|----------|-----|
| **Hover effects** | `:hover` |
| **Focus states** | `:focus`, `:focus-within` |
| **Form validation** | `:valid`, `:invalid`, `:required` |
| **Structural selection** | `:nth-child()`, `:first-child`, `:last-child` |
| **Creating icons** | `::before`, `::after` |
| **Styling text parts** | `::first-letter`, `::first-line` |
| **Custom selections** | `::selection` |
| **Placeholder text** | `::placeholder` |

### **Best Practices:**
1. **Use `::` for pseudo elements** (except for IE8 compatibility)
2. **Always include `content` property** for `::before`/`::after`
3. **Consider accessibility** when hiding content with pseudo elements
4. **Use `:not()`** to create more specific selectors
5. **Combine pseudo classes** for complex interactions
6. **Test with keyboard navigation** (focus states)
7. **Use `:where()`** to reduce specificity

### **Accessibility Considerations:**
```css
/* Don't rely only on color for state */
.button:hover,
.button:focus {
  background-color: #3498db;
  /* Also add non-color indicators */
  outline: 2px solid #3498db;
  transform: scale(1.05);
}

/* Screen reader friendly pseudo content */
.sr-only::before {
  content: "Important: ";
  /* Position off-screen for screen readers */
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### **Performance Tips:**
1. **Limit complex `:nth-child()` calculations** on large lists
2. **Avoid animating pseudo elements** excessively
3. **Use `will-change`** for animated pseudo elements
4. **Test on mobile** - hover states work differently

Pseudo classes and elements are incredibly powerful tools that can help you create sophisticated, interactive, and visually appealing interfaces without adding extra HTML markup!