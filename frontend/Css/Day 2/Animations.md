# **COMPLETE GUIDE: CSS TRANSFORMS, TRANSITIONS & ANIMATIONS**



## **PART 1: CSS TRANSFORMS**

### **What are Transforms?**
Transforms **modify the shape, position, and orientation** of elements without affecting the document flow.

```css
.element {
  transform: function(value);
}
```

### **1. Basic Transform Functions**

#### **Translate (Move)**
```css
/* Move along X, Y axes */
.translate {
  transform: translateX(50px);    /* Move right 50px */
  transform: translateY(-30px);   /* Move up 30px */
  transform: translate(20px, 40px); /* X Y */
  transform: translate(50%);      /* 50% of element's own width/height */
}

/* 3D translation */
.translate-3d {
  transform: translate3d(20px, 30px, 40px); /* X Y Z */
  transform: translateZ(100px); /* Move toward/away from viewer */
}
```

#### **Scale (Resize)**
```css
.scale {
  transform: scale(1.5);          /* 150% size */
  transform: scale(0.5);          /* 50% size */
  transform: scaleX(2);           /* Double width */
  transform: scaleY(0.8);         /* 80% height */
  transform: scale(1.2, 0.8);     /* Width 120%, height 80% */
  transform: scale3d(1, 1, 2);    /* 3D scaling */
}
```

#### **Rotate (Spin)**
```css
.rotate {
  transform: rotate(45deg);       /* Clockwise 45° */
  transform: rotate(-90deg);      /* Counter-clockwise 90° */
  transform: rotate(1rad);        /* Using radians */
  transform: rotate(0.5turn);     /* Half turn (180°) */
  
  /* 3D rotation */
  transform: rotateX(45deg);      /* Flip forward/backward */
  transform: rotateY(180deg);     /* Flip horizontally */
  transform: rotateZ(90deg);      /* Same as rotate() */
  transform: rotate3d(1, 1, 0, 45deg); /* Custom 3D axis */
}
```

#### **Skew (Distort)**
```css
.skew {
  transform: skewX(20deg);        /* Tilt horizontally */
  transform: skewY(-15deg);       /* Tilt vertically */
  transform: skew(20deg, 15deg);  /* X Y */
}
```

### **2. Advanced Transform Features**

#### **Transform Origin**
Controls the point around which transforms happen.

```css
.element {
  transform-origin: center center; /* Default: 50% 50% */
  transform-origin: left top;      /* Top-left corner */
  transform-origin: 100px 200px;   /* Specific coordinates */
  transform-origin: 0% 100%;       /* Bottom-left corner */
  transform-origin: 30% 80%;
}

/* Different origins for different effects */
.rotate-center { transform-origin: center; }
.rotate-corner { transform-origin: top left; }
```

#### **Multiple Transforms**
```css
.combined {
  transform: translate(50px, 30px) rotate(45deg) scale(1.2);
  /* ORDER MATTERS! Different order = different result */
}

/* Compare: */
.first-translate {
  transform: translate(50px, 30px) rotate(45deg);
  /* Move THEN rotate */
}

.first-rotate {
  transform: rotate(45deg) translate(50px, 30px);
  /* Rotate THEN move (different direction!) */
}
```

#### **3D Transforms & Perspective**
```css
.container {
  perspective: 1000px; /* Depth of 3D space */
  perspective-origin: 50% 50%; /* Vanishing point */
}

.card {
  transform-style: preserve-3d; /* Children exist in 3D space */
  transform: rotateY(45deg);
  backface-visibility: hidden; /* Hide back side when flipped */
}

/* Create a 3D cube */
.cube-face {
  position: absolute;
  transform: rotateY(0deg) translateZ(100px);
}
```

### **3. Practical Transform Examples**

```html
<style>
  .transform-demo {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
    margin: 30px 0;
  }
  
  .demo-box {
    width: 150px;
    height: 150px;
    background: linear-gradient(135deg, #3498db, #2ecc71);
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    transition: transform 0.3s ease;
  }
  
  .demo-box:hover {
    transform: scale(1.1);
  }
  
  .translate-demo { transform: translate(20px, 20px); }
  .scale-demo { transform: scale(0.8); }
  .rotate-demo { transform: rotate(15deg); }
  .skew-demo { transform: skew(10deg, 5deg); }
  .combined-demo { transform: translate(10px, -10px) rotate(-10deg); }
  .origin-demo { 
    transform-origin: top left;
    transform: rotate(20deg);
  }
</style>

<div class="transform-demo">
  <div class="demo-box translate-demo">Translate</div>
  <div class="demo-box scale-demo">Scale</div>
  <div class="demo-box rotate-demo">Rotate</div>
  <div class="demo-box skew-demo">Skew</div>
  <div class="demo-box combined-demo">Combined</div>
  <div class="demo-box origin-demo">Custom Origin</div>
</div>
```

---

## **PART 2: CSS TRANSITIONS**

### **What are Transitions?**
Transitions **animate changes** between CSS property values.

### **1. Basic Transition Syntax**
```css
.element {
  /* Shorthand: property duration timing-function delay */
  transition: all 0.3s ease 0s;
  
  /* Individual properties */
  transition-property: all; /* Which properties to animate */
  transition-duration: 0.3s; /* How long animation takes */
  transition-timing-function: ease; /* Speed curve */
  transition-delay: 0s; /* Wait before starting */
}
```

### **2. Transition Properties**

#### **transition-property**
```css
/* Specific properties */
.box {
  transition-property: transform, background-color;
  transition-property: width, height, opacity;
  transition-property: all; /* Animate ALL animatable properties */
  transition-property: none; /* No transitions */
}

/* Multiple with different settings */
.box {
  transition-property: transform, opacity, background-color;
  transition-duration: 0.3s, 0.5s, 0.8s;
}
```

#### **transition-duration**
```css
.box {
  transition-duration: 0.3s;
  transition-duration: 500ms; /* Milliseconds */
  transition-duration: 2s, 1s; /* Multiple durations */
}
```

#### **transition-timing-function (Easing)**
```css
.box {
  /* Keyword values */
  transition-timing-function: ease; /* Default */
  transition-timing-function: linear; /* Constant speed */
  transition-timing-function: ease-in; /* Slow start */
  transition-timing-function: ease-out; /* Slow end */
  transition-timing-function: ease-in-out; /* Slow start and end */
  transition-timing-function: step-start; /* Jump to end */
  transition-timing-function: step-end; /* Jump at end */
  
  /* Cubic Bézier curves */
  transition-timing-function: cubic-bezier(0.1, 0.7, 1.0, 0.1);
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55); /* Bounce */
  
  /* Steps */
  transition-timing-function: steps(4, jump-start);
  transition-timing-function: steps(10, jump-end);
}

/* Common easing presets */
.ease-bounce { transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55); }
.ease-elastic { transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55); }
```

#### **transition-delay**
```css
.box {
  transition-delay: 0.2s; /* Wait 200ms before starting */
  transition-delay: 1s, 0.5s, 0s; /* Different delays for different properties */
}
```

### **3. Transition Examples**

```html
<style>
  .transition-demo {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    margin: 30px 0;
  }
  
  .demo-card {
    width: 150px;
    height: 150px;
    background: #3498db;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    cursor: pointer;
  }
  
  /* Hover transitions */
  .color-transition {
    transition: background-color 0.5s ease;
  }
  
  .color-transition:hover {
    background-color: #e74c3c;
  }
  
  .size-transition {
    transition: all 0.3s ease;
  }
  
  .size-transition:hover {
    width: 180px;
    height: 180px;
    font-size: 1.2em;
  }
  
  .transform-transition {
    transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55);
  }
  
  .transform-transition:hover {
    transform: rotate(15deg) scale(1.1);
  }
  
  .delay-transition {
    transition: all 0.3s ease 0.5s; /* 0.5s delay */
  }
  
  .delay-transition:hover {
    background: #2ecc71;
    transform: translateY(-10px);
  }
  
  .multiple-transition {
    transition: 
      background-color 0.4s ease,
      transform 0.6s cubic-bezier(0.68, -0.55, 0.27, 1.55),
      border-radius 0.3s ease;
  }
  
  .multiple-transition:hover {
    background-color: #9b59b6;
    transform: scale(1.2) rotate(5deg);
    border-radius: 50%;
  }
  
  .easing-demo {
    transition: all 1s;
  }
  
  .easing-demo:hover {
    width: 300px;
  }
  
  .linear { transition-timing-function: linear; }
  .ease-in { transition-timing-function: ease-in; }
  .ease-out { transition-timing-function: ease-out; }
  .ease-in-out { transition-timing-function: ease-in-out; }
  .custom-ease { transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55); }
</style>

<div class="transition-demo">
  <div class="demo-card color-transition">Color</div>
  <div class="demo-card size-transition">Size</div>
  <div class="demo-card transform-transition">Transform</div>
  <div class="demo-card delay-transition">Delay</div>
  <div class="demo-card multiple-transition">Multiple</div>
</div>

<h3>Easing Comparison</h3>
<div class="transition-demo">
  <div class="demo-card easing-demo linear">Linear</div>
  <div class="demo-card easing-demo ease-in">Ease-in</div>
  <div class="demo-card easing-demo ease-out">Ease-out</div>
  <div class="demo-card easing-demo ease-in-out">Ease-in-out</div>
  <div class="demo-card easing-demo custom-ease">Custom</div>
</div>
```

### **4. Transition Events (JavaScript)**
```javascript
const element = document.querySelector('.transition-box');

element.addEventListener('transitionstart', () => {
  console.log('Transition started');
});

element.addEventListener('transitionend', (e) => {
  console.log(`Transition ended: ${e.propertyName}`);
  console.log(`Duration: ${e.elapsedTime}s`);
});

element.addEventListener('transitioncancel', () => {
  console.log('Transition cancelled');
});

element.addEventListener('transitionrun', () => {
  console.log('Transition created (may not have started yet)');
});
```

---

## **PART 3: CSS ANIMATIONS & KEYFRAMES**

### **What are Animations?**
Animations are **complex sequences** controlled by **@keyframes** rules.

### **1. @keyframes Rule**
Define animation sequence.

```css
@keyframes animation-name {
  /* Starting state */
  from {
    transform: translateX(0);
    opacity: 1;
  }
  
  /* Ending state */
  to {
    transform: translateX(100px);
    opacity: 0.5;
  }
}

/* Multiple keyframes */
@keyframes bounce {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-30px);
  }
  100% {
    transform: translateY(0);
  }
}

/* Complex sequence */
@keyframes complex-animation {
  0% { transform: scale(1); }
  25% { transform: scale(1.2) rotate(10deg); }
  50% { transform: scale(1) rotate(-10deg); }
  75% { transform: scale(0.8) rotate(5deg); }
  100% { transform: scale(1) rotate(0deg); }
}
```

### **2. Animation Properties**

```css
.element {
  /* Shorthand: name duration timing-function delay iteration-count direction fill-mode play-state */
  animation: bounce 2s ease-in-out 1s infinite alternate both running;
  
  /* Individual properties */
  animation-name: bounce; /* Which @keyframes to use */
  animation-duration: 2s; /* How long one cycle takes */
  animation-timing-function: ease-in-out; /* Speed curve */
  animation-delay: 1s; /* Wait before starting */
  animation-iteration-count: infinite; /* Number of times to play */
  animation-direction: normal; /* Play direction */
  animation-fill-mode: both; /* Styles before/after animation */
  animation-play-state: running; /* paused or running */
}

/* Multiple animations */
.element {
  animation: 
    slide-in 0.5s ease-out,
    fade-in 1s ease-in 0.5s,
    color-change 2s linear infinite;
}
```

#### **animation-iteration-count**
```css
.box {
  animation-iteration-count: 1; /* Play once */
  animation-iteration-count: 3; /* Play three times */
  animation-iteration-count: infinite; /* Loop forever */
  animation-iteration-count: 0.5; /* Play half cycle */
}
```

#### **animation-direction**
```css
.box {
  animation-direction: normal; /* 0% → 100% */
  animation-direction: reverse; /* 100% → 0% */
  animation-direction: alternate; /* normal then reverse */
  animation-direction: alternate-reverse; /* reverse then normal */
}
```

#### **animation-fill-mode**
```css
.box {
  animation-fill-mode: none; /* Default: no styles applied before/after */
  animation-fill-mode: forwards; /* Keep last keyframe styles */
  animation-fill-mode: backwards; /* Apply first keyframe styles during delay */
  animation-fill-mode: both; /* Both forwards and backwards */
}
```

#### **animation-play-state**
```css
.box {
  animation-play-state: running; /* Default: playing */
  animation-play-state: paused; /* Pause animation */
}

/* Control with JavaScript or hover */
.box:hover {
  animation-play-state: paused;
}
```

### **3. Animation Examples**

```html
<style>
  .animation-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
    margin: 30px 0;
  }
  
  .animation-box {
    width: 150px;
    height: 150px;
    background: linear-gradient(135deg, #3498db, #2ecc71);
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
  }
  
  /* Keyframe definitions */
  @keyframes slide-in {
    from {
      transform: translateX(-100%);
      opacity: 0;
    }
    to {
      transform: translateX(0);
      opacity: 1;
    }
  }
  
  @keyframes bounce {
    0%, 100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(-30px);
    }
  }
  
  @keyframes pulse {
    0% {
      transform: scale(1);
      box-shadow: 0 0 0 0 rgba(52, 152, 219, 0.7);
    }
    70% {
      transform: scale(1.1);
      box-shadow: 0 0 0 10px rgba(52, 152, 219, 0);
    }
    100% {
      transform: scale(1);
      box-shadow: 0 0 0 0 rgba(52, 152, 219, 0);
    }
  }
  
  @keyframes rotate {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
  
  @keyframes color-change {
    0% { background-color: #3498db; }
    25% { background-color: #2ecc71; }
    50% { background-color: #e74c3c; }
    75% { background-color: #9b59b6; }
    100% { background-color: #3498db; }
  }
  
  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
    20%, 40%, 60%, 80% { transform: translateX(5px); }
  }
  
  /* Applying animations */
  .slide-in {
    animation: slide-in 1s ease-out;
  }
  
  .bounce {
    animation: bounce 1s ease-in-out infinite;
  }
  
  .pulse {
    animation: pulse 2s infinite;
  }
  
  .rotate {
    animation: rotate 3s linear infinite;
  }
  
  .color-change {
    animation: color-change 5s linear infinite;
  }
  
  .shake {
    animation: shake 0.5s ease-in-out;
  }
  
  /* Animation controls */
  .animation-controls {
    display: flex;
    gap: 10px;
    margin: 20px 0;
  }
  
  .control-btn {
    padding: 10px 20px;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  .pausable {
    animation: rotate 3s linear infinite;
  }
</style>

<div class="animation-container">
  <div class="animation-box slide-in">Slide In</div>
  <div class="animation-box bounce">Bounce</div>
  <div class="animation-box pulse">Pulse</div>
  <div class="animation-box rotate">Rotate</div>
  <div class="animation-box color-change">Color Change</div>
  <div class="animation-box shake">Shake</div>
</div>

<h3>Animation Controls</h3>
<div class="animation-controls">
  <button onclick="startAnimation()" class="control-btn">Start</button>
  <button onclick="pauseAnimation()" class="control-btn">Pause</button>
  <button onclick="reverseAnimation()" class="control-btn">Reverse</button>
  <button onclick="resetAnimation()" class="control-btn">Reset</button>
</div>

<div class="animation-box pausable" id="controlled-animation">Controlled Animation</div>

<script>
  const animElement = document.getElementById('controlled-animation');
  
  function startAnimation() {
    animElement.style.animationPlayState = 'running';
  }
  
  function pauseAnimation() {
    animElement.style.animationPlayState = 'paused';
  }
  
  function reverseAnimation() {
    animElement.style.animationDirection = 'reverse';
    animElement.style.animationPlayState = 'running';
  }
  
  function resetAnimation() {
    animElement.style.animation = 'none';
    setTimeout(() => {
      animElement.style.animation = 'rotate 3s linear infinite';
    }, 10);
  }
</script>
```

### **4. Advanced Animation Techniques**

#### **Chaining Animations**
```css
@keyframes move-right {
  to { transform: translateX(200px); }
}

@keyframes move-down {
  to { transform: translateY(200px); }
}

.chain-animation {
  animation: 
    move-right 2s ease forwards,
    move-down 2s ease 2s forwards; /* Start after first ends */
}
```

#### **Staggered Animations**
```css
.stagger-item {
  animation: fade-in 0.5s ease forwards;
  opacity: 0;
}

.stagger-item:nth-child(1) { animation-delay: 0.1s; }
.stagger-item:nth-child(2) { animation-delay: 0.2s; }
.stagger-item:nth-child(3) { animation-delay: 0.3s; }
.stagger-item:nth-child(4) { animation-delay: 0.4s; }
```

#### **3D Flip Animation**
```html
<style>
  .flip-container {
    perspective: 1000px;
    width: 200px;
    height: 200px;
  }
  
  .flipper {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.8s;
    transform-style: preserve-3d;
  }
  
  .flip-container:hover .flipper {
    transform: rotateY(180deg);
  }
  
  .front, .back {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 24px;
    font-weight: bold;
  }
  
  .front {
    background: linear-gradient(135deg, #3498db, #2ecc71);
    color: white;
  }
  
  .back {
    background: linear-gradient(135deg, #e74c3c, #9b59b6);
    color: white;
    transform: rotateY(180deg);
  }
</style>

<div class="flip-container">
  <div class="flipper">
    <div class="front">FRONT</div>
    <div class="back">BACK</div>
  </div>
</div>
```

#### **Scroll-triggered Animations**
```css
@keyframes scroll-reveal {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.reveal-on-scroll {
  opacity: 0;
  animation: scroll-reveal 0.8s ease forwards;
}

/* JavaScript adds class when element is in viewport */
```

---

## **PART 4: COMBINING ALL TECHNIQUES**

### **1. Interactive Gallery with All Effects**
```html
<style>
  .gallery {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
    margin: 30px 0;
  }
  
  .gallery-item {
    position: relative;
    overflow: hidden;
    border-radius: 10px;
    aspect-ratio: 4/3;
    cursor: pointer;
    
    /* Initial state */
    transform: scale(0.95);
    opacity: 0.9;
    transition: all 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55);
  }
  
  .gallery-item:hover {
    /* Transition effects */
    transform: scale(1.05);
    opacity: 1;
    box-shadow: 0 20px 40px rgba(0,0,0,0.3);
    
    /* Animation on hover */
    animation: gentle-bounce 0.5s ease;
  }
  
  @keyframes gentle-bounce {
    0%, 100% { transform: scale(1.05); }
    50% { transform: scale(1.08); }
  }
  
  .gallery-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.8s ease;
  }
  
  .gallery-item:hover .gallery-img {
    transform: scale(1.1);
  }
  
  .gallery-caption {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    background: linear-gradient(to top, rgba(0,0,0,0.8), transparent);
    color: white;
    padding: 20px;
    transform: translateY(100%);
    transition: transform 0.4s ease 0.1s; /* With delay */
  }
  
  .gallery-item:hover .gallery-caption {
    transform: translateY(0);
  }
  
  .badge {
    position: absolute;
    top: 10px;
    right: -30px;
    background: #e74c3c;
    color: white;
    padding: 5px 30px;
    transform: rotate(45deg);
    font-size: 0.8em;
    animation: badge-pulse 2s infinite;
  }
  
  @keyframes badge-pulse {
    0%, 100% { opacity: 0.8; }
    50% { opacity: 1; }
  }
</style>

<div class="gallery">
  <div class="gallery-item">
    <div class="badge">NEW</div>
    <img src="https://via.placeholder.com/400x300" alt="Gallery" class="gallery-img">
    <div class="gallery-caption">
      <h3>Beautiful Sunset</h3>
      <p>Hover to see effects</p>
    </div>
  </div>
  <!-- More items -->
</div>
```

### **2. Loading Animation System**
```html
<style>
  .loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 300px;
    background: #f8f9fa;
    border-radius: 10px;
    padding: 40px;
  }
  
  .spinner {
    width: 60px;
    height: 60px;
    border: 5px solid rgba(52, 152, 219, 0.2);
    border-top-color: #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
  
  @keyframes spin {
    to { transform: rotate(360deg); }
  }
  
  .dots {
    display: flex;
    gap: 8px;
    margin: 30px 0;
  }
  
  .dot {
    width: 20px;
    height: 20px;
    background: #3498db;
    border-radius: 50%;
    animation: bounce 1.4s infinite;
  }
  
  .dot:nth-child(2) { animation-delay: 0.2s; }
  .dot:nth-child(3) { animation-delay: 0.4s; }
  
  @keyframes bounce {
    0%, 80%, 100% { 
      transform: translateY(0);
      opacity: 0.5;
    }
    40% { 
      transform: translateY(-20px);
      opacity: 1;
    }
  }
  
  .progress-bar {
    width: 300px;
    height: 10px;
    background: #e0e0e0;
    border-radius: 5px;
    overflow: hidden;
    margin: 20px 0;
  }
  
  .progress-fill {
    height: 100%;
    width: 0%;
    background: linear-gradient(to right, #3498db, #2ecc71);
    animation: progress 3s ease-in-out infinite;
  }
  
  @keyframes progress {
    0% { width: 0%; }
    50% { width: 100%; }
    100% { width: 0%; }
  }
  
  .pulse-text {
    font-size: 1.2em;
    font-weight: bold;
    color: #2c3e50;
    animation: text-pulse 2s infinite;
  }
  
  @keyframes text-pulse {
    0%, 100% { opacity: 0.5; }
    50% { opacity: 1; }
  }
</style>

<div class="loading-container">
  <div class="spinner"></div>
  
  <div class="dots">
    <div class="dot"></div>
    <div class="dot"></div>
    <div class="dot"></div>
  </div>
  
  <div class="progress-bar">
    <div class="progress-fill"></div>
  </div>
  
  <div class="pulse-text">Loading...</div>
</div>
```

### **3. Interactive Form with Validation Animations**
```html
<style>
  .form-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 30px;
    background: white;
    border-radius: 10px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  }
  
  .form-group {
    margin-bottom: 25px;
  }
  
  .form-label {
    display: block;
    margin-bottom: 8px;
    font-weight: 600;
    color: #2c3e50;
  }
  
  .form-input {
    width: 100%;
    padding: 12px 15px;
    border: 2px solid #ddd;
    border-radius: 6px;
    font-size: 16px;
    transition: all 0.3s ease;
  }
  
  .form-input:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
    transform: translateY(-2px);
  }
  
  /* Success animation */
  .form-input.success {
    border-color: #2ecc71;
    animation: success-pulse 0.5s ease;
  }
  
  @keyframes success-pulse {
    0% { box-shadow: 0 0 0 0 rgba(46, 204, 113, 0.4); }
    70% { box-shadow: 0 0 0 10px rgba(46, 204, 113, 0); }
    100% { box-shadow: 0 0 0 0 rgba(46, 204, 113, 0); }
  }
  
  /* Error animation */
  .form-input.error {
    border-color: #e74c3c;
    animation: shake 0.5s ease;
  }
  
  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    20%, 60% { transform: translateX(-5px); }
    40%, 80% { transform: translateX(5px); }
  }
  
  .error-message {
    color: #e74c3c;
    font-size: 0.875em;
    margin-top: 5px;
    opacity: 0;
    transform: translateY(-10px);
    transition: all 0.3s ease;
  }
  
  .error-message.show {
    opacity: 1;
    transform: translateY(0);
  }
  
  .submit-btn {
    width: 100%;
    padding: 15px;
    background: linear-gradient(to right, #3498db, #2980b9);
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
  }
  
  .submit-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 20px rgba(52, 152, 219, 0.3);
  }
  
  .submit-btn:active {
    transform: translateY(-1px);
  }
  
  /* Ripple effect */
  .submit-btn::after {
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
  
  .submit-btn:focus:not(:active)::after {
    animation: ripple 1s ease-out;
  }
  
  @keyframes ripple {
    0% {
      transform: scale(0, 0);
      opacity: 0.5;
    }
    100% {
      transform: scale(40, 40);
      opacity: 0;
    }
  }
  
  /* Loading state */
  .submit-btn.loading {
    background: #bdc3c7;
    cursor: not-allowed;
  }
  
  .submit-btn.loading::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    width: 20px;
    height: 20px;
    margin: -10px 0 0 -10px;
    border: 2px solid rgba(255,255,255,0.3);
    border-top-color: white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
</style>

<div class="form-container">
  <form id="animated-form">
    <div class="form-group">
      <label class="form-label">Email Address</label>
      <input type="email" class="form-input" id="email" required>
      <div class="error-message" id="email-error">Please enter a valid email</div>
    </div>
    
    <div class="form-group">
      <label class="form-label">Password</label>
      <input type="password" class="form-input" id="password" required>
      <div class="error-message" id="password-error">Password must be at least 8 characters</div>
    </div>
    
    <button type="submit" class="submit-btn" id="submit-btn">
      <span id="btn-text">Submit</span>
    </button>
  </form>
</div>

<script>
  const form = document.getElementById('animated-form');
  const emailInput = document.getElementById('email');
  const passwordInput = document.getElementById('password');
  const submitBtn = document.getElementById('submit-btn');
  const btnText = document.getElementById('btn-text');
  
  emailInput.addEventListener('input', () => {
    if (emailInput.value.includes('@')) {
      emailInput.classList.add('success');
      emailInput.classList.remove('error');
      document.getElementById('email-error').classList.remove('show');
    }
  });
  
  passwordInput.addEventListener('input', () => {
    if (passwordInput.value.length >= 8) {
      passwordInput.classList.add('success');
      passwordInput.classList.remove('error');
      document.getElementById('password-error').classList.remove('show');
    }
  });
  
  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    
    let hasError = false;
    
    // Email validation
    if (!emailInput.value.includes('@')) {
      emailInput.classList.add('error');
      emailInput.classList.remove('success');
      document.getElementById('email-error').classList.add('show');
      hasError = true;
    }
    
    // Password validation
    if (passwordInput.value.length < 8) {
      passwordInput.classList.add('error');
      passwordInput.classList.remove('success');
      document.getElementById('password-error').classList.add('show');
      hasError = true;
    }
    
    if (!hasError) {
      // Show loading state
      submitBtn.classList.add('loading');
      btnText.style.visibility = 'hidden';
      
      // Simulate API call
      setTimeout(() => {
        submitBtn.classList.remove('loading');
        btnText.style.visibility = 'visible';
        btnText.textContent = 'Success!';
        submitBtn.style.background = 'linear-gradient(to right, #2ecc71, #27ae60)';
        
        // Reset after 2 seconds
        setTimeout(() => {
          btnText.textContent = 'Submit';
          submitBtn.style.background = 'linear-gradient(to right, #3498db, #2980b9)';
        }, 2000);
      }, 2000);
    }
  });
</script>
```

### **4. Advanced 3D Animation**
```html
<style>
  .scene-3d {
    width: 300px;
    height: 300px;
    margin: 50px auto;
    perspective: 1000px;
  }
  
  .cube {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    animation: rotate-cube 20s infinite linear;
  }
  
  @keyframes rotate-cube {
    0% { transform: rotateX(0) rotateY(0); }
    100% { transform: rotateX(360deg) rotateY(360deg); }
  }
  
  .face {
    position: absolute;
    width: 300px;
    height: 300px;
    background: rgba(52, 152, 219, 0.8);
    border: 2px solid white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2em;
    font-weight: bold;
    color: white;
    backface-visibility: visible;
  }
  
  .front { transform: translateZ(150px); background: rgba(231, 76, 60, 0.8); }
  .back { transform: rotateY(180deg) translateZ(150px); background: rgba(46, 204, 113, 0.8); }
  .right { transform: rotateY(90deg) translateZ(150px); background: rgba(155, 89, 182, 0.8); }
  .left { transform: rotateY(-90deg) translateZ(150px); background: rgba(52, 73, 94, 0.8); }
  .top { transform: rotateX(90deg) translateZ(150px); background: rgba(241, 196, 15, 0.8); }
  .bottom { transform: rotateX(-90deg) translateZ(150px); background: rgba(230, 126, 34, 0.8); }
  
  .controls {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-top: 20px;
  }
  
  .control-btn {
    padding: 10px 20px;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
</style>

<div class="scene-3d">
  <div class="cube">
    <div class="face front">1</div>
    <div class="face back">2</div>
    <div class="face right">3</div>
    <div class="face left">4</div>
    <div class="face top">5</div>
    <div class="face bottom">6</div>
  </div>
</div>

<div class="controls">
  <button onclick="pauseAnimation()" class="control-btn">Pause</button>
  <button onclick="resumeAnimation()" class="control-btn">Resume</button>
  <button onclick="speedUp()" class="control-btn">Speed Up</button>
  <button onclick="slowDown()" class="control-btn">Slow Down</button>
</div>

<script>
  const cube = document.querySelector('.cube');
  let animationSpeed = 20;
  
  function pauseAnimation() {
    cube.style.animationPlayState = 'paused';
  }
  
  function resumeAnimation() {
    cube.style.animationPlayState = 'running';
  }
  
  function speedUp() {
    animationSpeed = Math.max(2, animationSpeed / 2);
    cube.style.animationDuration = `${animationSpeed}s`;
  }
  
  function slowDown() {
    animationSpeed *= 2;
    cube.style.animationDuration = `${animationSpeed}s`;
  }
</script>
```

---

## **PART 5: PERFORMANCE & BEST PRACTICES**

### **Performance Tips**
```css
/* 1. Animate transform and opacity (cheap) */
.good {
  transition: transform 0.3s, opacity 0.3s;
}

/* 2. Avoid animating layout properties (expensive) */
.bad {
  transition: width 0.3s, height 0.3s, margin 0.3s; /* Triggers reflow */
}

/* 3. Use will-change for complex animations */
.complex-animation {
  will-change: transform; /* Hint to browser */
}

/* 4. Use 3D transforms for GPU acceleration */
.gpu-accelerated {
  transform: translate3d(0, 0, 0);
}

/* 5. Limit simultaneous animations */
.too-many {
  animation: 
    move 1s,
    fade 1s,
    color-change 1s,
    rotate 1s; /* Too many! */
}
```

### **Accessibility Considerations**
```css
/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Provide pause controls for long animations */
.animation-controls {
  position: absolute;
  top: 10px;
  right: 10px;
}
```

### **Debugging Animations**
```css
/* Add debug outlines */
.debug-animation * {
  outline: 1px solid rgba(255,0,0,0.1);
}

/* Pause all animations for inspection */
body.paused * {
  animation-play-state: paused !important;
  transition: none !important;
}
```

## **SUMMARY: WHEN TO USE EACH**

### **Transforms**
- **When:** Need to change element's position/size/rotation
- **Use:** `transform: translate() rotate() scale()`
- **Example:** Hover effects, micro-interactions

### **Transitions**
- **When:** Simple state changes (hover, focus, class changes)
- **Use:** `transition: property duration easing`
- **Example:** Button hover, form focus, toggle switches

### **Animations**
- **When:** Complex sequences, automatic playback, looping
- **Use:** `@keyframes` with `animation` properties
- **Example:** Loading spinners, entrance animations, notifications

### **Combined**
- **When:** Creating polished, interactive experiences
- **Use:** Transitions for interaction, animations for automatic effects
- **Example:** Interactive galleries, animated interfaces, games

### **Golden Rules:**
1. **Use transforms instead of position/width/height** for animations
2. **Prefer opacity over visibility** for fade effects
3. **Use `will-change` sparingly** - only for complex animations
4. **Test with `prefers-reduced-motion`**
5. **Keep animations under 300ms** for interactions
6. **Use easing to make animations feel natural**
7. **Avoid animating too many elements simultaneously**

This comprehensive guide covers everything from basic transforms to complex 3D animations, giving you the tools to create engaging, performant, and accessible motion experiences on the web!