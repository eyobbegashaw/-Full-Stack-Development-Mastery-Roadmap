# **COMPLETE GUIDE: Z-Index, Specificity, Tables, Lists, Images & Filters**



## **PART 1: Z-INDEX & STACKING CONTEXT**

### **What is Z-Index?**
The `z-index` property controls the **vertical stacking order** of positioned elements.

```css
.element {
  position: relative; /* or absolute, fixed, sticky */
  z-index: 1; /* Higher value = closer to viewer */
}
```

### **Understanding Stacking Context**
A **stacking context** is a 3D conceptualization of HTML elements along the Z-axis.

```css
/* Elements that create a stacking context: */
.creates-context {
  position: relative | absolute | fixed | sticky;
  z-index: auto; /* But NOT auto with position: static */
  opacity: < 1;
  transform: scale(0.5) | rotate(45deg) | translateX(10px);
  filter: blur(5px) | drop-shadow(2px 2px 2px black);
  isolation: isolate; /* Explicitly creates context */
}
```

### **Basic Z-Index Rules**
```css
/* Higher z-index appears above lower */
.background {
  z-index: 1;
}

.foreground {
  z-index: 2; /* Appears above background */
}

/* Negative z-index */
.behind {
  z-index: -1; /* Goes behind parent */
}
```

### **Nested Stacking Contexts**
```html
<style>
  .parent1 {
    position: relative;
    z-index: 1;
  }
  .child1 {
    position: absolute;
    z-index: 100; /* Only matters within parent1 */
  }
  
  .parent2 {
    position: relative;
    z-index: 2; /* Will appear above parent1 AND all its children */
  }
  .child2 {
    position: absolute;
    z-index: 1; /* But still above parent1's z-index:100 */
  }
</style>

<div class="parent1">
  <div class="child1">z-index 100 (inside parent z-index 1)</div>
</div>
<div class="parent2">
  <div class="child2">z-index 1 (inside parent z-index 2)</div>
</div>
<!-- parent2 (z-index:2) appears above child1 (z-index:100) -->
```

### **Stacking Order (without z-index)**
1. Background and borders of the root element
2. **Negative z-index positioned elements**
3. **Block-level elements** in normal flow
4. **Floated elements**
5. **Inline elements** in normal flow
6. **Positioned elements** with `z-index: auto` or 0
7. **Positive z-index positioned elements**

### **Practical Examples**
```css
/* Modal system */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  z-index: 1000;
}

.modal {
  position: fixed;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1001; /* Above overlay */
}

/* Dropdown menu */
.dropdown {
  position: relative;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  z-index: 10;
}

/* Tooltip */
.tooltip {
  position: absolute;
  z-index: 9999; /* Very high - above everything */
}

/* Fixed header with dropdown */
.header {
  position: fixed;
  z-index: 100;
}

.dropdown-in-header {
  position: absolute;
  z-index: 101; /* Must be higher than header */
}
```

### **Common Z-Index Scale (Recommended)**
```css
:root {
  --z-negative: -1;
  --z-base: 0;
  --z-content: 1;
  --z-dropdown: 10;
  --z-sticky: 100;
  --z-fixed: 200;
  --z-modal-overlay: 1000;
  --z-modal: 1001;
  --z-popover: 2000;
  --z-tooltip: 3000;
  --z-toast: 4000;
}
```

### **Debugging Z-Index Issues**
```css
/* Add debugging outlines */
* {
  outline: 1px solid rgba(255,0,0,0.1);
}

/* Specific debugging */
.debug-z-index {
  box-shadow: 0 0 0 2px red;
}

/* Use browser DevTools to visualize stacking */
```

---

## **PART 2: CSS SPECIFICITY**

### **What is Specificity?**
A weight/score that determines which CSS rule applies when multiple rules target the same element.

### **Specificity Hierarchy**
```
Inline styles (1000) > IDs (100) > Classes/Attributes/Pseudo-classes (10) > Elements/Pseudo-elements (1)
```

### **Calculating Specificity**
```css
/* Specificity: 0,0,1 */
p { color: black; }

/* Specificity: 0,1,0 */
.text { color: blue; }

/* Specificity: 1,0,0 */
#main { color: red; }

/* Specificity: 0,1,1 */
p.text { color: green; }

/* Specificity: 1,1,1 */
#main p.text { color: purple; }
```

### **Specificity Examples**
```css
/* EXAMPLE 1 */
.button { color: blue; }          /* Specificity: 0,1,0 */
#submit { color: red; }          /* Specificity: 1,0,0 - WINS! */

/* EXAMPLE 2 */
div p { color: black; }          /* Specificity: 0,0,2 */
.container p { color: blue; }    /* Specificity: 0,1,1 - WINS! */

/* EXAMPLE 3 */
a:hover { color: red; }          /* Specificity: 0,1,1 (0 IDs, 1 class, 1 element) */
.nav a { color: blue; }          /* Specificity: 0,1,1 - TIE! */
/* SOURCE ORDER decides - last rule wins */
```

### **Important! Declaration**
```css
.button {
  color: blue !important; /* Overrides everything except other !important */
}

#special.button {
  color: red !important; /* Wins over blue !important (same importance, higher specificity) */
}
```

### **Specificity Tools & Tips**
```css
/* Use :where() for zero specificity */
:where(.button) { color: blue; } /* Specificity: 0,0,0 */

/* Use :is() to match specificity */
:is(.container, .wrapper) p { color: red; }

/* CSS Custom Properties don't affect specificity */
:root { --main-color: blue; }
.button { color: var(--main-color); } /* Specificity: 0,1,0 */
```

### **Best Practices**
1. **Avoid !important** - use only as last resort
2. **Use classes over IDs** for styling
3. **Keep specificity low** for maintainability
4. **Use BEM or similar methodology**
5. **Leverage CSS Custom Properties**

---

## **PART 3: TABLES**

### **Basic Table Styling**
```css
/* Basic table */
table {
  border-collapse: collapse; /* OR separate */
  border-spacing: 10px; /* Only works with separate */
  width: 100%;
  caption-side: top | bottom;
  empty-cells: show | hide;
  table-layout: auto | fixed;
}

/* Table cells */
td, th {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
  vertical-align: top | middle | bottom;
}

/* Table headers */
th {
  background-color: #f2f2f2;
  font-weight: bold;
}

/* Zebra striping */
tr:nth-child(even) {
  background-color: #f9f9f9;
}

tr:hover {
  background-color: #f5f5f5;
}
```

### **Advanced Table Features**
```css
/* Fixed header table */
.table-container {
  max-height: 400px;
  overflow: auto;
}

.table-container thead th {
  position: sticky;
  top: 0;
  background: white;
  z-index: 10;
}

/* Responsive tables */
@media (max-width: 768px) {
  table {
    display: block;
    overflow-x: auto;
    white-space: nowrap;
  }
}

/* Alternative: stack on mobile */
@media (max-width: 768px) {
  table, thead, tbody, th, td, tr {
    display: block;
  }
  
  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }
  
  tr {
    border: 1px solid #ccc;
    margin-bottom: 10px;
  }
  
  td {
    border: none;
    position: relative;
    padding-left: 50%;
  }
  
  td:before {
    position: absolute;
    left: 10px;
    content: attr(data-label);
    font-weight: bold;
  }
}
```

### **Complete Table Example**
```html
<style>
  .data-table {
    border-collapse: collapse;
    width: 100%;
    margin: 20px 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }
  
  .data-table thead {
    background: linear-gradient(to right, #3498db, #2980b9);
    color: white;
  }
  
  .data-table th {
    padding: 15px;
    text-align: left;
    font-weight: 600;
    border-bottom: 3px solid #2980b9;
  }
  
  .data-table td {
    padding: 12px 15px;
    border-bottom: 1px solid #eee;
  }
  
  .data-table tbody tr:hover {
    background-color: #f8f9fa;
    transform: translateX(5px);
    transition: all 0.2s ease;
  }
  
  .data-table tbody tr:last-child td {
    border-bottom: none;
  }
  
  .data-table caption {
    caption-side: bottom;
    padding: 10px;
    font-style: italic;
    color: #666;
  }
</style>

<table class="data-table">
  <caption>Employee Directory</caption>
  <thead>
    <tr>
      <th>Name</th>
      <th>Department</th>
      <th>Position</th>
      <th>Salary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>Engineering</td>
      <td>Senior Developer</td>
      <td>$95,000</td>
    </tr>
    <tr>
      <td>Jane Smith</td>
      <td>Marketing</td>
      <td>Marketing Manager</td>
      <td>$85,000</td>
    </tr>
  </tbody>
</table>
```

---

## **PART 4: LISTS**

### **Unordered Lists (ul)**
```css
/* Custom bullet points */
.custom-list {
  list-style-type: none; /* Remove default */
  padding-left: 0;
}

.custom-list li {
  padding-left: 30px;
  position: relative;
  margin-bottom: 10px;
}

.custom-list li:before {
  content: '✓';
  position: absolute;
  left: 0;
  color: #2ecc71;
  font-weight: bold;
}

/* Alternative bullets */
.star-list li:before {
  content: '★';
  color: gold;
}

.number-list li:before {
  content: counter(item) ". ";
  counter-increment: item;
  font-weight: bold;
}
```

### **Ordered Lists (ol)**
```css
/* Custom numbering */
.custom-ol {
  list-style-type: none;
  counter-reset: item;
}

.custom-ol li {
  counter-increment: item;
  margin-bottom: 15px;
}

.custom-ol li:before {
  content: counter(item);
  background: #3498db;
  color: white;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-right: 10px;
}

/* Different numbering styles */
.roman { list-style-type: upper-roman; }
.alpha { list-style-type: lower-alpha; }
.decimal-leading-zero { list-style-type: decimal-leading-zero; }
```

### **List Styling Properties**
```css
ul, ol {
  list-style-type: disc | circle | square | none;
  list-style-position: inside | outside;
  list-style-image: url('bullet.png');
  
  /* Shorthand: type position image */
  list-style: square inside url('bullet.png');
}

/* Custom property examples */
.fancy-list {
  list-style: none;
  padding: 0;
}

.fancy-list li {
  background: linear-gradient(to right, #f8f9fa, #e9ecef);
  margin: 5px 0;
  padding: 15px;
  border-left: 5px solid #3498db;
  border-radius: 0 5px 5px 0;
}
```

### **Navigation Menu from List**
```css
.nav-menu {
  list-style: none;
  display: flex;
  gap: 20px;
  padding: 0;
  margin: 0;
}

.nav-menu li {
  position: relative;
}

.nav-menu a {
  text-decoration: none;
  color: #333;
  padding: 10px 15px;
  display: block;
  transition: all 0.3s ease;
}

.nav-menu a:hover {
  background: #3498db;
  color: white;
  border-radius: 4px;
}

/* Dropdown submenu */
.nav-menu ul {
  position: absolute;
  top: 100%;
  left: 0;
  background: white;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  display: none;
  min-width: 200px;
}

.nav-menu li:hover > ul {
  display: block;
}
```

---

## **PART 5: IMAGES AND FILTERS**

### **Image Basics**
```css
img {
  max-width: 100%; /* Responsive images */
  height: auto; /* Maintain aspect ratio */
  display: block; /* Remove inline spacing */
}

/* Image alignment */
.img-left {
  float: left;
  margin-right: 20px;
  margin-bottom: 20px;
}

.img-right {
  float: right;
  margin-left: 20px;
  margin-bottom: 20px;
}

/* Circular images */
.avatar {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  object-fit: cover; /* Prevent distortion */
}

/* Responsive images with srcset */
/* HTML: <img src="small.jpg" srcset="medium.jpg 1000w, large.jpg 2000w" alt="..."> */
```

### **CSS Filters**
```css
/* Basic filters */
.image {
  filter: blur(5px);
  filter: brightness(150%); /* 150% brighter */
  filter: contrast(200%); /* Double contrast */
  filter: grayscale(100%); /* Black & white */
  filter: hue-rotate(90deg); /* Shift colors */
  filter: invert(100%); /* Invert colors */
  filter: opacity(50%); /* 50% transparent */
  filter: saturate(200%); /* Double saturation */
  filter: sepia(100%); /* Vintage effect */
  
  /* Multiple filters */
  filter: grayscale(100%) blur(2px) brightness(80%);
}

/* Drop shadow (different from box-shadow) */
.image {
  filter: drop-shadow(5px 5px 5px rgba(0,0,0,0.5));
}

/* Backdrop filter (for blurring background) */
.modal {
  backdrop-filter: blur(10px);
}
```

### **Image Hover Effects**
```css
/* Zoom on hover */
.zoom-effect {
  transition: transform 0.3s ease;
  overflow: hidden;
}

.zoom-effect img {
  transition: transform 0.5s ease;
}

.zoom-effect:hover img {
  transform: scale(1.1);
}

/* Grayscale to color */
.grayscale-hover {
  filter: grayscale(100%);
  transition: filter 0.3s ease;
}

.grayscale-hover:hover {
  filter: grayscale(0%);
}

/* Overlay effect */
.image-container {
  position: relative;
  overflow: hidden;
}

.image-overlay {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(52, 152, 219, 0.8);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-container:hover .image-overlay {
  opacity: 1;
}
```

### **Object-Fit Property**
```css
/* Control how image fits container */
.cover-image {
  width: 300px;
  height: 200px;
  object-fit: cover; /* Crop to fill */
  object-fit: contain; /* Fit entire image */
  object-fit: fill; /* Stretch to fill */
  object-fit: none; /* Keep original size */
  object-fit: scale-down; /* Like contain but smaller */
  
  /* Position the image */
  object-position: center top; /* x y */
}
```

### **Advanced Image Gallery**
```html
<style>
  .gallery {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
    padding: 20px;
  }
  
  .gallery-item {
    position: relative;
    overflow: hidden;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    transition: all 0.3s ease;
  }
  
  .gallery-item img {
    width: 100%;
    height: 200px;
    object-fit: cover;
    transition: transform 0.5s ease;
  }
  
  .gallery-item:hover {
    transform: translateY(-10px);
    box-shadow: 0 15px 30px rgba(0,0,0,0.2);
  }
  
  .gallery-item:hover img {
    transform: scale(1.1);
    filter: brightness(110%) saturate(120%);
  }
  
  .gallery-caption {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(0,0,0,0.7);
    color: white;
    padding: 15px;
    transform: translateY(100%);
    transition: transform 0.3s ease;
  }
  
  .gallery-item:hover .gallery-caption {
    transform: translateY(0);
  }
</style>

<div class="gallery">
  <div class="gallery-item">
    <img src="image1.jpg" alt="Landscape">
    <div class="gallery-caption">Beautiful Landscape</div>
  </div>
  <!-- More items -->
</div>
```

### **CSS Blend Modes**
```css
/* Image blending */
.blend-image {
  background-image: url('image.jpg');
  background-color: #3498db;
  background-blend-mode: multiply | screen | overlay | darken | lighten | color-dodge;
  
  /* Or with two images */
  background-image: url('image1.jpg'), url('image2.jpg');
  background-blend-mode: multiply;
}

/* Element blending */
.element {
  mix-blend-mode: normal | multiply | screen | overlay | difference;
}
```

### **Performance Tips for Images**
```css
/* Lazy loading */
img {
  loading: lazy; /* Native browser lazy loading */
}

/* Optimize loading */
img {
  content-visibility: auto; /* Modern optimization */
}

/* Responsive with CSS */
.hero-image {
  background-image: url('small.jpg');
}

@media (min-width: 768px) {
  .hero-image {
    background-image: url('medium.jpg');
  }
}

@media (min-width: 1200px) {
  .hero-image {
    background-image: url('large.jpg');
  }
}
```

---

## **COMPREHENSIVE EXAMPLE COMBINING ALL CONCEPTS**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    /* Z-INDEX SYSTEM */
    :root {
      --z-background: -1;
      --z-base: 0;
      --z-dropdown: 10;
      --z-modal: 1000;
      --z-tooltip: 2000;
    }
    
    /* BASE STYLES */
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      line-height: 1.6;
      color: #333;
      background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
      min-height: 100vh;
      padding: 20px;
    }
    
    /* TABLE WITH Z-INDEX & SPECIFICITY */
    .data-table-container {
      position: relative;
      background: white;
      border-radius: 10px;
      padding: 20px;
      margin: 30px 0;
      box-shadow: 0 5px 15px rgba(0,0,0,0.1);
      z-index: var(--z-base);
    }
    
    /* Specificity: 0,1,0 */
    .data-table {
      width: 100%;
      border-collapse: collapse;
    }
    
    /* Specificity: 0,0,2 */
    .data-table th,
    .data-table td {
      padding: 15px;
      border-bottom: 1px solid #eee;
      text-align: left;
    }
    
    /* Specificity: 0,1,2 - WINS over .data-table th */
    .data-table thead th {
      background: linear-gradient(to right, #3498db, #2980b9);
      color: white;
      position: sticky;
      top: 0;
      z-index: var(--z-dropdown);
    }
    
    /* Zebra striping - higher specificity than tr:hover */
    .data-table tbody tr:nth-child(even) {
      background-color: #f9f9f9;
    }
    
    /* Hover effect */
    .data-table tbody tr:hover {
      background-color: #e3f2fd;
      transform: translateX(5px);
      transition: all 0.3s ease;
    }
    
    /* STYLED LISTS */
    .features-list {
      background: white;
      border-radius: 10px;
      padding: 30px;
      margin: 30px 0;
      box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    }
    
    .features-list ul {
      list-style: none;
      counter-reset: feature-counter;
      padding-left: 0;
    }
    
    .features-list li {
      counter-increment: feature-counter;
      padding: 15px 15px 15px 60px;
      margin-bottom: 15px;
      background: #f8f9fa;
      border-left: 5px solid #3498db;
      border-radius: 5px;
      position: relative;
      transition: all 0.3s ease;
    }
    
    .features-list li:hover {
      background: #e3f2fd;
      transform: translateX(10px);
    }
    
    .features-list li:before {
      content: counter(feature-counter);
      position: absolute;
      left: 15px;
      top: 50%;
      transform: translateY(-50%);
      background: #3498db;
      color: white;
      width: 30px;
      height: 30px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
    }
    
    /* IMAGE GALLERY WITH FILTERS */
    .gallery-section {
      margin: 30px 0;
    }
    
    .gallery-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
      gap: 20px;
      margin-top: 20px;
    }
    
    .gallery-item {
      position: relative;
      overflow: hidden;
      border-radius: 10px;
      box-shadow: 0 5px 15px rgba(0,0,0,0.1);
      transition: all 0.3s ease;
      z-index: var(--z-base);
    }
    
    .gallery-item:hover {
      transform: translateY(-10px);
      box-shadow: 0 20px 40px rgba(0,0,0,0.2);
      z-index: var(--z-dropdown); /* Bring forward on hover */
    }
    
    .gallery-img {
      width: 100%;
      height: 200px;
      object-fit: cover;
      filter: grayscale(20%) brightness(95%);
      transition: all 0.5s ease;
    }
    
    .gallery-item:hover .gallery-img {
      filter: grayscale(0%) brightness(110%) saturate(120%);
      transform: scale(1.1);
    }
    
    .gallery-caption {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background: rgba(0,0,0,0.8);
      color: white;
      padding: 15px;
      transform: translateY(100%);
      transition: transform 0.3s ease;
      backdrop-filter: blur(5px);
    }
    
    .gallery-item:hover .gallery-caption {
      transform: translateY(0);
    }
    
    /* FILTER DEMOS */
    .filter-demo {
      display: flex;
      gap: 20px;
      flex-wrap: wrap;
      margin: 30px 0;
    }
    
    .filter-box {
      width: 150px;
      height: 150px;
      border-radius: 10px;
      overflow: hidden;
      position: relative;
    }
    
    .filter-box img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    
    /* Different filters */
    .blur-filter { filter: blur(3px); }
    .brightness-filter { filter: brightness(150%); }
    .contrast-filter { filter: contrast(200%); }
    .grayscale-filter { filter: grayscale(100%); }
    .hue-filter { filter: hue-rotate(90deg); }
    .invert-filter { filter: invert(100%); }
    .sepia-filter { filter: sepia(100%); }
    .saturate-filter { filter: saturate(200%); }
    
    /* MODAL WITH HIGH Z-INDEX */
    .modal-trigger {
      background: #3498db;
      color: white;
      border: none;
      padding: 12px 24px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background 0.3s ease;
    }
    
    .modal-trigger:hover {
      background: #2980b9;
    }
    
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(0,0,0,0.5);
      display: none;
      justify-content: center;
      align-items: center;
      z-index: var(--z-modal);
    }
    
    .modal-content {
      background: white;
      padding: 30px;
      border-radius: 10px;
      max-width: 500px;
      width: 90%;
      position: relative;
      z-index: calc(var(--z-modal) + 1);
      box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    }
    
    .modal-close {
      position: absolute;
      top: 15px;
      right: 15px;
      background: none;
      border: none;
      font-size: 24px;
      cursor: pointer;
      color: #666;
    }
    
    /* SPECIFICITY DEMO */
    /* Specificity battle */
    .special-item {
      color: blue; /* 0,1,0 */
    }
    
    #unique-item {
      color: red; /* 1,0,0 - WINS! */
    }
    
    .container .special-item {
      color: green; /* 0,2,0 - WINS over #unique-item? NO! IDs beat classes */
    }
    
    /* Using !important */
    .override {
      color: purple !important; /* Wins everything */
    }
    
    /* RESPONSIVE DESIGN */
    @media (max-width: 768px) {
      .gallery-grid {
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
      }
      
      .filter-demo {
        justify-content: center;
      }
      
      .data-table-container {
        overflow-x: auto;
      }
      
      .data-table {
        min-width: 600px;
      }
    }
  </style>
</head>
<body>
  
  <h1>CSS: Z-Index, Specificity, Tables, Lists & Filters</h1>
  
  <!-- TABLE SECTION -->
  <div class="data-table-container">
    <h2>Data Table with Sticky Header</h2>
    <table class="data-table">
      <thead>
        <tr>
          <th>Product</th>
          <th>Category</th>
          <th>Price</th>
          <th>Stock</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Laptop Pro</td>
          <td>Electronics</td>
          <td>$1,299</td>
          <td>In Stock</td>
        </tr>
        <tr>
          <td>Wireless Mouse</td>
          <td>Accessories</td>
          <td>$49</td>
          <td>In Stock</td>
        </tr>
        <tr>
          <td>Mechanical Keyboard</td>
          <td>Accessories</td>
          <td>$129</td>
          <td>Out of Stock</td>
        </tr>
      </tbody>
    </table>
  </div>
  
  <!-- LIST SECTION -->
  <div class="features-list">
    <h2>Features List</h2>
    <ul>
      <li>Responsive Design</li>
      <li>High Performance</li>
      <li>Modern UI Components</li>
      <li>Accessibility Compliance</li>
      <li>Cross-browser Compatibility</li>
    </ul>
  </div>
  
  <!-- IMAGE GALLERY -->
  <div class="gallery-section">
    <h2>Image Gallery with Hover Effects</h2>
    <div class="gallery-grid">
      <div class="gallery-item">
        <img src="https://via.placeholder.com/300x200/3498db/ffffff" alt="Placeholder" class="gallery-img">
        <div class="gallery-caption">Blue Landscape</div>
      </div>
      <div class="gallery-item">
        <img src="https://via.placeholder.com/300x200/2ecc71/ffffff" alt="Placeholder" class="gallery-img">
        <div class="gallery-caption">Green Valley</div>
      </div>
      <div class="gallery-item">
        <img src="https://via.placeholder.com/300x200/e74c3c/ffffff" alt="Placeholder" class="gallery-img">
        <div class="gallery-caption">Red Mountains</div>
      </div>
    </div>
  </div>
  
  <!-- FILTER DEMO -->
  <h2>CSS Filter Effects</h2>
  <div class="filter-demo">
    <div class="filter-box">
      <img src="https://via.placeholder.com/150/3498db/ffffff" alt="Original">
      <div style="position: absolute; bottom: 5px; left: 5px; background: rgba(0,0,0,0.7); color: white; padding: 2px 5px; font-size: 12px;">Original</div>
    </div>
    <div class="filter-box blur-filter">
      <img src="https://via.placeholder.com/150/3498db/ffffff" alt="Blur">
      <div style="position: absolute; bottom: 5px; left: 5px; background: rgba(0,0,0,0.7); color: white; padding: 2px 5px; font-size: 12px;">Blur</div>
    </div>
    <div class="filter-box grayscale-filter">
      <img src="https://via.placeholder.com/150/3498db/ffffff" alt="Grayscale">
      <div style="position: absolute; bottom: 5px; left: 5px; background: rgba(0,0,0,0.7); color: white; padding: 2px 5px; font-size: 12px;">Grayscale</div>
    </div>
    <div class="filter-box sepia-filter">
      <img src="https://via.placeholder.com/150/3498db/ffffff" alt="Sepia">
      <div style="position: absolute; bottom: 5px; left: 5px; background: rgba(0,0,0,0.7); color: white; padding: 2px 5px; font-size: 12px;">Sepia</div>
    </div>
  </div>
  
  <!-- MODAL -->
  <button class="modal-trigger" onclick="openModal()">Open Modal</button>
  
  <div class="modal-overlay" id="modal">
    <div class="modal-content">
      <button class="modal-close" onclick="closeModal()">×</button>
      <h2>Modal Window</h2>
      <p>This modal has high z-index to appear above everything else.</p>
      <p>Notice how it overlays all other content on the page.</p>
    </div>
  </div>
  
  <!-- SPECIFICITY DEMO -->
  <div style="margin: 30px 0; padding: 20px; background: white; border-radius: 10px;">
    <h2>CSS Specificity Demo</h2>
    <p class="special-item" id="unique-item">This text is RED (ID selector wins)</p>
    <p class="special-item override">This text is PURPLE (!important wins)</p>
  </div>
  
  <script>
    function openModal() {
      document.getElementById('modal').style.display = 'flex';
    }
    
    function closeModal() {
      document.getElementById('modal').style.display = 'none';
    }
    
    // Close modal on outside click
    document.getElementById('modal').addEventListener('click', function(e) {
      if (e.target === this) {
        closeModal();
      }
    });
    
    // Close on Escape key
    document.addEventListener('keydown', function(e) {
      if (e.key === 'Escape') {
        closeModal();
      }
    });
  </script>
  
</body>
</html>
```

## **KEY TAKEAWAYS**

### **Z-Index:**
1. Only works on positioned elements (not static)
2. Creates stacking contexts that isolate children
3. Use a consistent scale (e.g., 10s, 100s, 1000s)
4. Higher values appear closer to the viewer

### **Specificity:**
1. Inline > ID > Class > Element
2. Avoid !important when possible
3. Use methodologies (BEM) to keep specificity low
4. Understand that CSS rules are evaluated from top to bottom

### **Tables:**
1. Use `border-collapse` for clean borders
2. Make tables responsive with overflow or stacked layout
3. Use `position: sticky` for fixed headers
4. Add hover states for better UX

### **Lists:**
1. Customize bullets with `list-style` or pseudo-elements
2. Use counters for custom numbering
3. Create navigation menus from lists
4. Remove default styling with `list-style: none`

### **Images & Filters:**
1. Always use `max-width: 100%` for responsive images
2. Use `object-fit` to control image cropping
3. Filters can create visual effects without image editing
4. Combine filters for complex effects
5. Use `backdrop-filter` for glass/morphism effects

These concepts are essential for creating sophisticated, maintainable, and visually appealing web interfaces!