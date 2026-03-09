 This image outlines the main types of **Simple Selectors** in CSS. Let me explain each one with detailed examples:

## **1. Element Selector (Type Selector)**
Selects all elements of a specific HTML tag name.

**Syntax:** `elementname`
```css
/* Targets ALL <p> elements */
p {
  color: blue;
}

/* Targets ALL <h1> elements */
h1 {
  font-size: 2em;
}

/* Targets ALL <div> elements */
div {
  border: 1px solid black;
}
```

## **2. Class Selector**
Selects all elements with a specific `class` attribute. **Most commonly used selector.**

**Syntax:** `.classname`
```css
/* Targets elements with class="highlight" */
.highlight {
  background-color: yellow;
}

/* Targets elements with class="button" */
.button {
  padding: 10px 20px;
  border-radius: 5px;
}

/* An element can have multiple classes */
<div class="box red shadow">Content</div>
.box { width: 100px; }
.red { background: red; }
.shadow { box-shadow: 2px 2px 5px gray; }
```

## **3. ID Selector**
Selects a **single element** with a specific `id` attribute. IDs should be unique on a page.

**Syntax:** `#idname`
```css
/* Targets the element with id="header" */
#header {
  background: navy;
  color: white;
}

/* Targets element with id="main-content" */
#main-content {
  width: 80%;
  margin: 0 auto;
}

<!-- HTML -->
<nav id="main-nav">...</nav>  <!-- Only one element with this ID -->
```

## **4. Universal Selector**
Selects **ALL elements** on the page. Use sparingly as it affects every single element.

**Syntax:** `*`
```css
/* Resets margin and padding on ALL elements */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box; /* Popular reset technique */
}

/* Make ALL text red (not recommended) */
* {
  color: red;
}
```

## **5. Grouping Selector**
Allows you to apply the same styles to **multiple different selectors** at once.

**Syntax:** `selector1, selector2, selector3`
```css
/* Apply same styles to h1, h2, and h3 */
h1, h2, h3 {
  font-family: 'Arial', sans-serif;
  color: #333;
}

/* Group different selector types */
.button, #submit-btn, input[type="submit"] {
  background: blue;
  color: white;
  padding: 10px;
}

/* More complex grouping */
.main-title, 
.subtitle, 
.section-header {
  text-align: center;
  margin-bottom: 20px;
}
```

---

## **Comparison & Specificity**

| Selector | Example | Specificity | Use Case |
|----------|---------|-------------|----------|
| **Element** | `p` | Lowest (0,0,1) | Styling all instances of an element |
| **Class** | `.menu` | Medium (0,1,0) | Reusable styles for multiple elements |
| **ID** | `#header` | High (1,0,0) | Unique element styling |
| **Universal** | `*` | Lowest (0,0,0) | Resets/global styles |
| **Grouping** | `h1, h2, h3` | Varies | Applying same styles to multiple selectors |

---

## **Practical Examples:**

### **Example 1: Complete Stylesheet**
```css
/* Universal reset */
* {
  margin: 0;
  padding: 0;
}

/* Element selectors */
body {
  font-family: Arial;
  background: #f5f5f5;
}

/* Class selectors */
.container {
  max-width: 1200px;
  margin: 0 auto;
}

.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
}

/* ID selector */
#main-header {
  background: #333;
  color: white;
  padding: 20px;
}

/* Grouping selectors */
h1, h2, h3, h4 {
  color: #2c3e50;
  line-height: 1.3;
}

.btn, 
.button, 
input[type="button"] {
  display: inline-block;
  padding: 10px 20px;
  cursor: pointer;
}
```

### **Example 2: Specificity in Action**
```css
/* Specificity: 0,0,1 */
p {
  color: black;
}

/* Specificity: 0,1,0 - WINS! */
.article-text {
  color: blue;
}

/* Specificity: 1,0,0 - Would win over class */
#special-paragraph {
  color: green;
}

<!-- HTML -->
<p>Black text (element selector)</p>
<p class="article-text">Blue text (class selector wins)</p>
<p class="article-text" id="special-paragraph">Green text (ID selector wins)</p>
```

### **Example 3: Efficient Grouping**
```css
/* Instead of this repetitive code: */
h1 { text-align: center; }
h2 { text-align: center; }
h3 { text-align: center; }

/* Do this (more efficient): */
h1, h2, h3 {
  text-align: center;
}

/* Group related elements */
nav li, 
footer li, 
.sidebar li {
  list-style: none;
  display: inline-block;
}
```

---

## **Best Practices:**
1. **Use classes** for reusable styling (most common)
2. **Use IDs sparingly** - only for unique elements
3. **Avoid universal selector** for general styling
4. **Use grouping** to keep your CSS DRY (Don't Repeat Yourself)
5. **Remember specificity hierarchy:** ID > Class > Element > Universal

These simple selectors form the foundation of CSS targeting - you'll use them in virtually every stylesheet you write!