# CSS Cheatsheet

Complete reference from CSS basics to advanced techniques.

## 1. Selectors - The Foundation

### Basic Selectors

| Selector | Type | Example | Purpose |
|---|---|---|---|
| `*` | Universal | `* { margin: 0; }` | Targets all elements |
| `element` | Type | `p { color: blue; }` | Targets specific element |
| `.class` | Class | `.button { padding: 10px; }` | Targets elements with class |
| `#id` | ID | `#header { background: navy; }` | Targets element with ID |
| `[attr]` | Attribute | `[type="text"] { }` | Targets by attribute |

### Combinators

| Combinator | Syntax | Description |
|---|---|---|
| Descendant | `div p` | Any `p` inside `div` (any level) |
| Child | `div > p` | Direct `p` child of `div` |
| Adjacent Sibling | `h1 + p` | `p` immediately after `h1` |
| General Sibling | `h1 ~ p` | Any `p` sibling of `h1` |

### Pseudo-Classes & Pseudo-Elements

```css
/* Pseudo-Classes (:) */
:hover           /* Mouse over */
:focus           /* Has focus */
:active          /* Being clicked */
:visited         /* Visited link */
:nth-child(n)    /* nth element */
:first-child     /* First child */
:last-child      /* Last child */
:not(selector)   /* Negation */
:disabled        /* Disabled element */
:enabled         /* Enabled element */
```

```css
/* Pseudo-Elements (::) */
::before         /* Add before */
::after          /* Add after */
::first-letter   /* First letter */
::first-line     /* First line */
::selection      /* Selected text */
::placeholder    /* Placeholder */
```

## 2. Box Model - Core Concept

Complete box model example:

```css
.element {
  /* Outer spacing */
  margin: 20px;
  margin: 10px 20px 30px 40px; /* top right bottom left */

  /* Inner spacing */
  padding: 15px;
  padding: 10px 20px;

  /* Border */
  border: 2px solid #333;
  border-radius: 8px;

  /* Size */
  width: 100%;
  height: auto;
  min-width: 200px;
  max-width: 1200px;

  /* Box sizing */
  box-sizing: border-box;
}
```

### Essential Box Model Properties

| Property | Values | Default |
|---|---|---|
| `margin` | px, %, em, rem, auto | 0 |
| `padding` | px, %, em, rem | 0 |
| `border-width` | px, thin, medium, thick | medium |
| `border-style` | solid, dashed, dotted, double | none |
| `border-color` | color values | currentColor |
| `box-sizing` | content-box, border-box | content-box |

> **BEST PRACTICE:** Always use `* { box-sizing: border-box; }` in your reset CSS. This makes sizing calculations much more intuitive.

### Display Property Values

| Value | Width | Height | New Line | Use Case |
|---|---|---|---|---|
| `block` | 100% | Content | Yes | Paragraphs, divs, headings |
| `inline` | Content | Content | No | Spans, links, emphasis |
| `inline-block` | Set | Set | No | Images, buttons, input |
| `flex` | Flexible | Flexible | No | Navigation, layouts |
| `grid` | Flexible | Flexible | No | Complex layouts |
| `none` | Hidden | Hidden | No | Hide elements |

## 3. Display & Layout Properties

### Positioning Property

| Value | Reference Point | Affects Layout Flow | Use Case |
|---|---|---|---|
| `static` | Document flow | No | Default positioning |
| `relative` | Own normal position | No (keeps space) | Slight adjustments |
| `absolute` | Nearest positioned parent | Yes (removed from flow) | Overlays, badges |
| `fixed` | Browser viewport | Yes (removed from flow) | Headers, footers |
| `sticky` | Scrolling container | No (until sticky) | Sticky headers |

## 4. Flexbox - Flexible Layouts

Flexbox container setup:

```css
.container {
  display: flex;
  flex-direction: row; /* row, column, row-reverse, column-reverse */
  flex-wrap: wrap; /* nowrap, wrap, wrap-reverse */
  justify-content: center; /* flex-start, flex-end, center, space-between, space-around, space-evenly */
  align-items: center; /* flex-start, flex-end, center, baseline, stretch */
  align-content: center; /* For multiple lines */
  gap: 20px; /* Space between items */
}
```

Flexbox item properties:

```css
.item {
  flex: 1; /* Shorthand for grow, shrink, basis */
  flex-grow: 2; /* Growth factor (0-1000) */
  flex-shrink: 1; /* Shrink factor (default 1) */
  flex-basis: 200px; /* Default size before flex */
  order: 1; /* Visual order */
  align-self: flex-end; /* Override container alignment */
}
```

> **PRO TIP:** Use `flex: 1` for equal-width columns. Use `justify-content: center; align-items: center;` to center both axes.

## 5. CSS Grid - Advanced Layouts

Grid container setup:

```css
.container {
  display: grid;

  /* Define columns */
  grid-template-columns: 200px 1fr 200px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));

  /* Define rows */
  grid-template-rows: 100px auto 100px;

  /* Named areas */
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";

  /* Gaps */
  gap: 20px;
  row-gap: 10px;
  column-gap: 20px;
}
```

Placing grid items:

```css
.item {
  grid-column: 1 / 3; /* Start at 1, end at 3 */
  grid-column: span 2; /* Span 2 columns */
  grid-row: 1 / 4; /* Start at 1, end at 4 */
  grid-row: span 2; /* Span 2 rows */
  grid-area: header; /* Use named area */
}
```

> **RESPONSIVE GRID:** `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))` creates responsive columns that fit content.

## 6. Colors & Gradients

### Color Formats

| Format | Example | Transparency | Use Case |
|---|---|---|---|
| Named | `red, blue, navy` | No | Basic colors |
| Hex | `#FF0000` or `#F00` | Optional | Most common |
| RGB | `rgb(255, 0, 0)` | No | Programmatic |
| RGBA | `rgba(255, 0, 0, 0.5)` | Yes (alpha) | With transparency |
| HSL | `hsl(0, 100%, 50%)` | No | Intuitive colors |
| HSLA | `hsla(0, 100%, 50%, 0.5)` | Yes (alpha) | HSL + transparency |

### Gradients

```css
/* Linear Gradient */
background: linear-gradient(to right, red, blue);
background: linear-gradient(45deg, red, blue, green);
background: linear-gradient(to right, red 0%, blue 50%, green 100%);

/* Radial Gradient */
background: radial-gradient(circle, red, blue);
background: radial-gradient(ellipse at center, red, blue);

/* Conic Gradient */
background: conic-gradient(from 0deg, red, blue, red);

/* Repeating Gradient */
background: repeating-linear-gradient(45deg, red, red 10px, blue 10px, blue 20px);
```

## 7. Typography & Fonts

### Font Properties

| Property | Values | Default |
|---|---|---|
| `font-family` | Arial, Helvetica, sans-serif | depends on browser |
| `font-size` | 16px, 1em, 1rem, 100% | 16px (browser default) |
| `font-weight` | normal, bold, 100-900 | normal (400) |
| `font-style` | normal, italic, oblique | normal |
| `line-height` | 1.6, 24px, 150% | normal (1-1.2) |
| `letter-spacing` | 2px, 0.1em, normal | normal (0) |

### Text Properties

| Property | Values | Purpose |
|---|---|---|
| `text-align` | left, center, right, justify | Horizontal alignment |
| `text-decoration` | none, underline, overline, line-through | Text decoration |
| `text-transform` | none, uppercase, lowercase, capitalize | Transform text case |
| `text-indent` | 20px, 2em | First line indent |
| `text-overflow` | clip, ellipsis | Overflow handling |
| `white-space` | normal, nowrap, pre, pre-wrap | Whitespace handling |

### Web Fonts Import

```html
<!-- In HTML head -->
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap" rel="stylesheet">
```

```css
/* In CSS */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap');

/* Custom @font-face */
@font-face {
  font-family: 'MyFont';
  src: url('font.woff2') format('woff2'),
       url('font.woff') format('woff');
  font-weight: 400;
  font-display: swap;
}

body { font-family: 'Poppins', sans-serif; }
```

## 8. Effects & Transforms

### Transform Functions

```css
/* Translation */
transform: translate(20px, 30px);
transform: translateX(100px);
transform: translateY(50px);

/* Rotation */
transform: rotate(45deg);
transform: rotateX(45deg);
transform: rotateY(45deg);

/* Scaling */
transform: scale(1.5);
transform: scaleX(2);
transform: scaleY(0.5);

/* Skewing */
transform: skew(10deg, 20deg);
transform: skewX(10deg);

/* 3D */
transform: translate3d(10px, 20px, 30px);
transform: rotateZ(45deg);

/* Multiple transforms */
transform: translate(20px) rotate(45deg) scale(1.2);

/* Transform origin */
transform-origin: center;
transform-origin: top left;
```

### Transitions

```css
/* Basic transition */
transition: all 0.3s ease;

/* Detailed */
transition-property: background, color;
transition-duration: 0.3s;
transition-timing-function: ease-in-out;
transition-delay: 0.1s;

/* Timing functions:
   ease (default)
   linear
   ease-in
   ease-out
   ease-in-out
   cubic-bezier(0.4, 0.0, 0.2, 1)
   steps(5)
*/

/* Usage */
.button {
  background: blue;
  transition: background 0.3s ease;
}
.button:hover {
  background: red;
}
```

### Animations

```css
/* Define animation */
@keyframes slide {
  0% {
    transform: translateX(0);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translateX(100%);
    opacity: 1;
  }
}

/* Apply animation */
.element {
  animation: slide 2s ease-in-out infinite;
  animation-direction: normal;
  animation-fill-mode: forwards;
  animation-delay: 0.5s;
}

/* Animation properties */
animation-name: slide;
animation-duration: 2s;
animation-timing-function: ease;
animation-delay: 0s;
animation-iteration-count: 1; /* or infinite */
animation-direction: normal | reverse | alternate;
animation-fill-mode: none | forwards | backwards | both;
animation-play-state: running | paused;
```

> **PERFORMANCE TIP:** Use `transform` and `opacity` for animations - they're GPU-accelerated and perform better than animating position or size.

## 9. Responsive Design & Media Queries

Viewport meta tag (essential):

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

Media query examples:

```css
/* Mobile first */
.container { width: 100%; }

/* Tablet and above */
@media (min-width: 768px) {
  .container { width: 750px; }
}

/* Desktop and above */
@media (min-width: 1024px) {
  .container { width: 960px; }
}

/* Large desktop */
@media (min-width: 1200px) {
  .container { width: 1140px; }
}

/* Combined conditions */
@media (min-width: 768px) and (max-width: 1023px) {
  /* Tablets only */
}

/* Other media features */
@media (orientation: landscape) { }
@media (prefers-color-scheme: dark) { }
@media (prefers-reduced-motion: reduce) { }
@media (hover: none) { /* Touch devices */ }
```

### Common Breakpoints

| Device | Breakpoint | CSS Media Query |
|---|---|---|
| Mobile | < 480px | Default / no media query |
| Small Mobile | 480px - 600px | `@media (min-width: 480px)` |
| Tablet | 600px - 900px | `@media (min-width: 600px)` |
| Tablet Landscape | 900px - 1200px | `@media (min-width: 900px)` |
| Desktop | 1200px+ | `@media (min-width: 1200px)` |

## 10. CSS Variables (Custom Properties)

```css
/* Define variables */
:root {
  --primary-color: #4285F4;
  --secondary-color: #34A853;
  --spacing: 8px;
  --border-radius: 4px;
  --transition: 0.3s ease;
}

/* Use variables */
.button {
  background: var(--primary-color);
  padding: var(--spacing);
  border-radius: var(--border-radius);
  transition: all var(--transition);
}
.button:hover {
  background: var(--secondary-color);
}

/* With fallback */
color: var(--text-color, #333);

/* Variables are inherited */
.dark-theme {
  --primary-color: #1a237e;
  --text-color: #ffffff;
}
```

### Benefits of CSS Variables

| Advantages | When to Use |
|---|---|
| Centralized theming | Easy maintenance |
| DRY principle | Runtime changes |
| Dynamic themes | Reduced duplication |
| Colors & branding | Spacing scales |
| Typography | Theme switching |
| Component variants | — |

## 11. CSS Units & Sizing

### Unit Types

| Unit | Type | Relative To | Best For |
|---|---|---|---|
| `px` | Absolute | Pixels (1/96 inch) | Borders, shadows |
| `rem` | Relative | Root font-size (default 16px) | Font sizes (RECOMMENDED) |
| `em` | Relative | Parent font-size | Component sizing |
| `%` | Relative | Parent value | Widths, heights |
| `vw` | Viewport | 1% viewport width | Full-width elements |
| `vh` | Viewport | 1% viewport height | Full-height sections |
| `vmin` | Viewport | 1% smaller dimension | Responsive sizing |
| `vmax` | Viewport | 1% larger dimension | Responsive sizing |

### Modern Responsive Sizing: `clamp()`

```css
/* Fluid typography */
font-size: clamp(14px, 2vw, 24px);
/* Minimum 14px, preferred 2vw, maximum 24px */

/* Responsive padding */
padding: clamp(10px, 5%, 40px);

/* Responsive spacing */
margin: clamp(1rem, 5vw, 3rem);
```

## 12. CSS Pitfalls & Best Practices

### Common Mistakes to Avoid

**MISTAKE: Using `!important` everywhere** — Breaks cascade, hard to debug, maintenance nightmare.
**CORRECT:** Fix specificity instead of using `!important`.

**MISTAKE: High specificity selectors** — `#header .navbar ul li a { }` is too specific, hard to override.
**CORRECT:** Use simple classes like `.nav-link`.

**MISTAKE: Missing `box-sizing: border-box`** — Width calculations inconsistent and confusing.
**CORRECT:** `* { box-sizing: border-box; }` in reset.

**MISTAKE: Not testing responsive design** — Works on desktop, broken on mobile.
**CORRECT:** Use a mobile-first design approach.

**MISTAKE: Animating non-GPU properties** — Animating `width`, `height`, `left`, `position` is slow.
**CORRECT:** Animate `transform` and `opacity` (fast).

### CSS Best Practices

**DO's**
- Use semantic class names
- Mobile-first approach
- CSS variables for theming
- Keep specificity low
- Use shorthand properties
- Organize code logically
- Test on real devices
- Use flexbox/grid

**DON'Ts**
- Don't use inline styles
- Don't overuse `!important`
- Don't use IDs for styling
- Don't hardcode colors
- Don't use floats for layout
- Don't forget vendor prefixes
- Don't use px for everything
- Don't skip media queries

### Professional Tips

**Tip 1: Specificity Strategy**
Use classes for styling, avoid IDs. Specificity `0-1-0` (class) is better than `1-0-0` (ID).

**Tip 2: Mobile First**
Write CSS for mobile, then add media queries for larger screens. Reduces code and improves performance.

**Tip 3: Use DevTools**
Chrome DevTools is essential. Inspect elements, debug styles, test responsive design, check performance.

**Tip 4: Performance Matters**
Minify CSS in production, use shorthand properties, avoid unnecessary selectors, remove unused CSS.

**Tip 5: Accessibility First**
Contrast ratio 4.5:1, focus states, keyboard navigation, don't remove outline, test with screen readers.

## 13. CSS Performance Optimization

### Optimization Steps

- Minify CSS in production
- Remove unused CSS (PurgeCSS)
- Use CSS variables to reduce duplication
- Inline critical CSS
- Load non-critical CSS async
- Use shorthand properties
- Avoid `!important` to keep file smaller
- Enable GZIP compression on server

### CSS Minification Example

Before minification:

```css
.button {
  background-color: #4285F4;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
}
```

After minification:

```css
.button{background-color:#4285F4;color:#fff;padding:10px 20px;border:0;border-radius:4px;cursor:pointer;transition:all .3s ease}
```

## 14. Quick Reference Table

### Most Used CSS Properties

| Category | Property | Common Values |
|---|---|---|
| Display | `display` | block, inline, inline-block, flex, grid, none |
| Display | `position` | static, relative, absolute, fixed, sticky |
| Display | `z-index` | 10, 100, 1000 (positive integers) |
| Sizing | `width`, `height` | 100px, 50%, 100%, auto |
| Sizing | `max-width`, `min-width` | 500px, 100%, 1200px |
| Sizing | `padding`, `margin` | 10px, 1em, 1rem, auto |
| Colors | `color` | red, #FF0000, rgb(255,0,0), hsl(0,100%,50%) |
| Colors | `background` | #FFF, linear-gradient(...), url(...) |
| Colors | `opacity` | 0.5, 0.8, 1 (0-1) |
| Typography | `font-size`, `font-weight`, `font-style` | 16px, 1rem, 700, italic |
| Typography | `line-height`, `letter-spacing` | 1.6, 24px, 2px |
| Typography | `text-align`, `text-decoration` | center, none, underline |
| Effects | `box-shadow`, `text-shadow` | 5px 5px 10px rgba(0,0,0,0.3) |
| Effects | `transform`, `transition` | rotate(45deg), all 0.3s ease |

---

*Source: adapted from the CSS cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
