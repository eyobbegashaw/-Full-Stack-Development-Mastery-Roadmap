 This image combines two important CSS topics: **Attribute Selectors** and **Font Properties**. Let me explain both sections in detail:

# **PART 1: ATTRIBUTE SELECTORS**

Attribute selectors target HTML elements based on their attributes or attribute values.

## **Basic Syntax:**
```css
element[attribute] {
  /* styles */
}
```

## **Types of Attribute Selectors:**

### **1. `[attribute]` - Existence**
Selects elements that have the attribute, regardless of value.
```css
/* Selects all elements with a title attribute */
[title] {
  border: 1px dotted gray;
}

/* Selects all <a> with href attribute */
a[href] {
  color: blue;
}
```

### **2. `[attribute="value"]` - Exact Match**
Selects elements where attribute equals exactly "value".
```css
/* Selects input with type="text" */
input[type="text"] {
  width: 200px;
}

/* Selects links to exact URL */
a[href="https://example.com"] {
  color: green;
}
```

### **3. `[attribute~="value"]` - Contains Word**
Selects elements where attribute contains the word "value".
```css
/* Selects elements with class containing "btn" */
[class~="btn"] {
  padding: 10px;
}

<!-- HTML -->
<button class="primary btn">Click</button>  <!-- Selected -->
<button class="btn-secondary">Click</button> <!-- NOT selected -->
```

### **4. `[attribute|="value"]` - Starts with Word (or word followed by hyphen)**
Selects elements where attribute equals "value" or starts with "value-".
```css
/* Selects lang attributes starting with "en" */
[lang|="en"] {
  font-family: Arial;
}

<!-- HTML -->
<p lang="en">Selected</p>
<p lang="en-US">Selected</p>
<p lang="english">NOT selected</p>
```

### **5. `[attribute^="value"]` - Starts With**
Selects elements where attribute value BEGINS with "value".
```css
/* Selects links starting with "https" */
a[href^="https"] {
  color: green;
}

/* Selects IDs starting with "user_" */
[id^="user_"] {
  border: 1px solid blue;
}
```

### **6. `[attribute$="value"]` - Ends With**
Selects elements where attribute value ENDS with "value".
```css
/* Selects links to PDF files */
a[href$=".pdf"] {
  background: url(pdf-icon.png);
}

/* Selects images ending with .jpg */
img[src$=".jpg"] {
  border: 2px solid black;
}
```

### **7. `[attribute*="value"]` - Contains Substring**
Selects elements where attribute value CONTAINS "value" anywhere.
```css
/* Selects any element containing "error" in class */
[class*="error"] {
  color: red;
}

/* Selects links containing "example" */
a[href*="example"] {
  background: yellow;
}
```

## **Case-Insensitive Matching (CSS4):**
```css
/* Matches regardless of case */
a[href*="login" i] {
  font-weight: bold;
}
/* Would match: LOGIN, Login, login, LoGiN */
```

---

# **PART 2: FONT PROPERTIES**

## **1. Font Families (`font-family`)**
Specifies the typeface for text.

```css
p {
  /* List fonts in order of preference */
  font-family: "Helvetica Neue", Arial, sans-serif;
}

/* Common stacks */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}
```

**Generic families (fallbacks):**
- `serif` (Times New Roman, Georgia)
- `sans-serif` (Arial, Helvetica)
- `monospace` (Courier, Consolas)
- `cursive` (Comic Sans)
- `fantasy` (Impact)

## **2. Font Style (`font-style`)**
Controls italic/oblique text.

```css
.normal {
  font-style: normal;   /* Default */
}

.italic {
  font-style: italic;   /* True italic font */
}

.oblique {
  font-style: oblique;  /* Slanted version */
}
```

## **3. Font Size (`font-size`)**
Sets text size using various units.

```css
/* Absolute units */
p {
  font-size: 16px;      /* Pixels - most common */
  font-size: 12pt;      /* Points (1pt = 1/72 inch) */
}

/* Relative units - better for accessibility */
body {
  font-size: 100%;      /* Usually 16px */
}

h1 {
  font-size: 2em;       /* 2 × parent size */
  font-size: 2rem;      /* 2 × root (html) size */
}

.small {
  font-size: 0.875em;   /* 14px if parent is 16px */
}

/* Keywords */
.large-text {
  font-size: large;     /* small | medium | large | x-large */
}
```

## **4. Font Variant (`font-variant`)**
Controls typographic variants.

```css
.normal {
  font-variant: normal;     /* Default */
}

.small-caps {
  font-variant: small-caps; /* Uppercase letters, smaller capitals */
}

/* CSS3 expanded properties */
.advanced {
  font-variant-numeric: oldstyle-nums;  /* 12345 */
  font-variant-ligatures: common-ligatures; /* fi, fl combinations */
  font-variant-caps: all-small-caps;    /* All small capitals */
}
```

## **5. Font Shorthand (`font`)**
Combines multiple font properties in one declaration.

**Syntax:** `font: style variant weight size/line-height family;`

```css
/* Individual properties */
p {
  font-style: italic;
  font-weight: bold;
  font-size: 16px;
  line-height: 1.5;
  font-family: Arial, sans-serif;
}

/* Shorthand equivalent */
p {
  font: italic bold 16px/1.5 Arial, sans-serif;
}

/* Minimum required: size and family */
p {
  font: 16px Arial;  /* style, weight revert to normal */
}

/* With all properties */
.heading {
  font: italic small-caps 700 24px/1.2 "Georgia", serif;
}
```

**Order matters!** Must follow: `style variant weight size/line-height family`

## **6. Google Fonts**
Web fonts from Google's free library.

### **Method 1: Link in HTML**
```html
<head>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
</head>
```
```css
body {
  font-family: 'Roboto', sans-serif;
}
```

### **Method 2: @import in CSS**
```css
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@300;400;600&display=swap');

body {
  font-family: 'Open Sans', sans-serif;
}
```

### **Method 3: Download & Self-Host**
```css
@font-face {
  font-family: 'MyFont';
  src: url('fonts/myfont.woff2') format('woff2'),
       url('fonts/myfont.woff') format('woff');
  font-weight: normal;
  font-style: normal;
  font-display: swap; /* Controls loading behavior */
}
```

---

## **Complete Font Example:**

```html
<!DOCTYPE html>
<html>
<head>
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&family=Roboto+Slab&display=swap" rel="stylesheet">
  <style>
    /* Attribute Selector Example */
    input[type="email"] {
      border: 2px solid #3498db;
      padding: 10px;
    }
    
    a[href^="https://"]::after {
      content: "🔒";
      margin-left: 5px;
    }
    
    /* Font Properties Example */
    body {
      font-family: 'Montserrat', sans-serif;
      font-size: 16px;
      line-height: 1.6;
    }
    
    h1 {
      font: 700 2.5rem/1.2 'Roboto Slab', serif;
      font-variant: small-caps;
      letter-spacing: 1px;
    }
    
    .intro {
      font-style: italic;
      font-size: 1.1em;
      font-weight: 300;
    }
    
    .code {
      font-family: 'Courier New', monospace;
      font-size: 0.9em;
      font-variant-numeric: tabular-nums;
    }
  </style>
</head>
<body>
  <h1>Welcome to My Site</h1>
  <p class="intro">This is styled with Google Fonts</p>
  <form>
    <input type="email" placeholder="Your email">
  </form>
  <p>Visit our <a href="https://secure.example.com">secure site</a></p>
  <p class="code">console.log("Hello World");</p>
</body>
</html>
```

## **Best Practices:**

1. **Font Loading:** Use `font-display: swap` for web fonts to prevent FOIT
2. **Font Stacks:** Always provide fallback fonts
3. **Relative Units:** Use `rem` for font sizes for better accessibility
4. **Line Height:** Unitless values (1.5) are best for line-height
5. **Web Fonts:** Limit to 2-3 font families per page for performance
6. **Attribute Selectors:** Great for styling forms, links, and dynamic content without adding classes

These tools give you precise control over text styling and element selection in your web projects!