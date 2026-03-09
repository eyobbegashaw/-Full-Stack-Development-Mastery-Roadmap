 This image breaks down the fundamental **syntax structure of CSS**. Here's a detailed explanation of each component:

## **Complete CSS Syntax Structure:**
```css
/* This is a Comment */
selector {
  property: value;  /* This is a Declaration */
}
```

---

## **1. Selector**
The **Selector** targets the HTML element(s) you want to style.

**Types of Selectors:**
- **Element selector:** `p`, `h1`, `div`
- **Class selector:** `.my-class` (targets `class="my-class"`)
- **ID selector:** `#my-id` (targets `id="my-id"`)
- **Attribute selector:** `[type="text"]`
- **Pseudo-class:** `:hover`, `:first-child`
- **Combination:** `div.my-class`, `ul > li`

**Example:**
```css
p                 /* Targets all <p> elements */
.button           /* Targets elements with class="button" */
#header           /* Targets element with id="header" */
```

## **2. Declaration Block**
Everything inside the curly braces `{ }` is a **declaration block**. It contains one or more declarations.

## **3. Declaration**
A **Declaration** is a single style instruction consisting of a **property-value pair**.

**Structure:**
```
property: value;
```

**Parts:**
- **Property:** The style attribute you want to change
- **Colon:** Separates property from value
- **Value:** The setting for the property
- **Semicolon:** Ends the declaration (required for each except the last)

**Example:**
```css
color: blue;      /* Declaration: property=color, value=blue */
font-size: 16px;  /* Declaration: property=font-size, value=16px */
```

## **4. Rule Set (or "Rules")**
A **Rule Set** (often called just "Rule") is the complete construct:
```
selector + declaration block = rule set
```

**Example of a complete rule set:**
```css
h1 {
  color: navy;
  font-size: 2em;
  margin-bottom: 20px;
}
```
- **Selector:** `h1`
- **Declaration block:** Everything inside `{ }`
- **Declarations:** 3 declarations separated by semicolons
- **This entire thing is one Rule Set**

## **5. Comments**
**Comments** are notes in your CSS that are ignored by browsers. They help document your code.

**Syntax:**
- Single-line: `/* This is a comment */`
- Multi-line: 
```css
/* 
  This is a 
  multi-line comment 
*/
```

**Example with all components:**
```css
/* This styles all paragraphs - COMMENT */
p {                         /* SELECTOR */
  color: darkgray;          /* DECLARATION: property=color, value=darkgray */
  font-size: 18px;          /* DECLARATION */
  line-height: 1.6;         /* DECLARATION */
}                           /* This entire block is a RULE SET */
```

---

## **Real-World Example Breakdown:**
```css
/* Navigation Styling */
.nav-bar {                     /* SELECTOR (class selector) */
  background-color: #333;      /* DECLARATION */
  padding: 15px 0;            /* DECLARATION */
  font-family: 'Arial', sans-serif; /* DECLARATION */
}

.nav-bar a {                  /* SELECTOR (descendant selector) */
  color: white;               /* DECLARATION */
  text-decoration: none;      /* DECLARATION */
  margin: 0 20px;             /* DECLARATION */
}                             /* Complete RULE SET */
```

---

## **Visual Relationship:**
```
CSS Document
├── Comment: "Page header styles"
├── Rule Set 1:
│   ├── Selector: h1
│   └── Declaration Block:
│       ├── Declaration 1: color: blue;
│       ├── Declaration 2: font-size: 2em;
│       └── Declaration 3: margin: 10px;
├── Rule Set 2:
│   ├── Selector: .container
│   └── Declaration Block: {...}
└── More Rules...
```

## **Key Points to Remember:**
1. **One selector** can have **multiple declarations**
2. **Each declaration** must end with a semicolon (except the last one, but it's good practice to always use it)
3. **Comments** don't affect styling but are crucial for documentation
4. **Rule Sets** are independent - they don't affect each other's order (except through the cascade)

This structure is the foundation of all CSS - every stylesheet is essentially a collection of these rule sets with selectors targeting elements and declarations defining their appearance.