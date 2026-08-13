# HTML Cheatsheet

A comprehensive, practical reference covering HTML5 setup, document structure, semantic elements, forms, tables, accessibility, SEO, and advanced features.

## 📘 1. HTML Fundamentals & Setup

HTML is native to web browsers — no installation is required. Simply create `.html` files in any text editor.

**Recommended Setup**

- Text Editor: VS Code (recommended), Sublime Text, Atom, Notepad++
- Browser Testing: Chrome DevTools (F12), Firefox Developer Tools, Safari Developer Tools, Edge Tools

**System Requirements**

| Requirement | Specification | Notes |
|---|---|---|
| Operating System | Windows, macOS, Linux | Any OS with a web browser |
| Text Editor | Any plain text editor | VS Code recommended |
| Web Browser | Chrome, Firefox, Safari, Edge | HTML5 support required |
| RAM | 4GB minimum | 8GB+ recommended |
| Storage | Minimal (MB) | Only for source files |

**VS Code HTML Setup**

1. Install VS Code from https://code.visualstudio.com
2. Install extensions:
   - Live Server (Ritwick Dey)
   - HTML CSS Support (ecmel)
   - Prettier (Code Formatter)
3. Create a workspace folder
4. Create a new file: `index.html`
5. Type `!` and press Tab for the auto HTML5 template
6. Right-click the file → Open with Live Server

## 2. HTML Document Structure

**Complete HTML5 Document Template**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Page description for SEO">
    <meta name="keywords" content="html, css, web development">
    <meta name="author" content="Your Name">
    <meta property="og:title" content="Page Title">
    <meta property="og:description" content="Description">
    <meta property="og:image" content="image.jpg">
    <meta name="theme-color" content="#4285F4">
    <link rel="icon" type="image/svg+xml" href="favicon.svg">
    <link rel="stylesheet" href="style.css">
    <title>Your Page Title | Website Name</title>
</head>
<body>
    <header></header>
    <nav></nav>
    <main></main>
    <aside></aside>
    <footer></footer>
    <script src="script.js" defer></script>
</body>
</html>
```

**Essential Head Elements**

| Element | Purpose | Notes |
|---|---|---|
| `<!DOCTYPE html>` | Document type declaration | Must be the first line |
| `<meta charset="UTF-8">` | Character encoding | Always include UTF-8 |
| `<meta name="viewport" ...>` | Responsive design | Required for mobile |
| `<title>` | Browser tab title | 50–60 chars recommended |
| `<link rel="icon">` | Favicon | PNG or SVG format |
| `<link rel="stylesheet">` | External CSS | Place in the head section |

## 3. Semantic HTML Elements

**Layout Structure**

```html
<header>
  Logo, Navigation, Banner
</header>
<nav>
  Primary navigation links
</nav>
<main>
  Main content
</main>
<aside>
  Sidebar, Related content
</aside>
<footer>
  Copyright, Links, Info
</footer>
```

**Content Sections**

```html
<article>
  Self-contained content
</article>
<section>
  Thematic grouping
</section>
<div>
  Generic container
</div>
<figure>
  Illustration, diagram
</figure>
<figcaption>
  Caption for figure
</figcaption>
```

**Semantic Best Practices**

- ✓ DO: Use semantic HTML (`header`, `nav`, `main`, `article`, `section`, `footer`)
- ✗ DON'T: Use `<div>` for everything — use semantic tags for clarity and accessibility
- Benefits: Better SEO, improved accessibility, cleaner code, easier maintenance

## 4. Headings & Text Content

**Heading Hierarchy**

```html
<h1>Main Page Title (one per page)</h1>
<h2>Section Heading</h2>
<h3>Subsection Heading</h3>
<h4>Sub-subsection Heading</h4>
<h5>Minor heading</h5>
<h6>Least important heading</h6>
```

IMPORTANT: Don't skip levels (h1 → h3 is wrong). Use the h1 → h2 → h3 → h4 progression.

**Text Formatting Elements**

| Element | Semantic Meaning | When to Use | Example |
|---|---|---|---|
| `<strong>` | Strong importance | Important information | `<strong>Critical</strong>` |
| `<em>` | Emphasized text | Emphasized words | `<em>Important</em>` |
| `<mark>` | Highlighted/marked | Search results, alerts | `<mark>Highlighted</mark>` |
| `<del>` | Deleted text | Removed content | `<del>Old price</del>` |
| `<ins>` | Inserted text | New/added content | `<ins>New price</ins>` |
| `<small>` | Small print | Legal text, disclaimers | `<small>Fine print</small>` |

**Code & Preformatted Text**

```html
<!-- Inline code -->
<code>const x = 5;</code>

<!-- Block of code with formatting preserved -->
<pre><code>
Block of code
with formatting
preserved
</code></pre>

<!-- Keyboard input -->
<kbd>Ctrl</kbd> + <kbd>C</kbd>

<!-- Variables -->
<var>variable_name</var>

<!-- Sample output -->
<samp>Output text</samp>
```

## 5. Links & Navigation

**Anchor Links — Complete Guide**

```html
<!-- Absolute URL -->
<a href="https://example.com">External Link</a>

<!-- Relative URL -->
<a href="about.html">About Page</a>
<a href="pages/contact.html">Contact</a>
<a href="../index.html">Home</a>

<!-- Anchor links -->
<a href="#section1">Jump to Section</a>
<section id="section1">Section 1</section>

<!-- Special links -->
<a href="mailto:email@example.com">Email Us</a>
<a href="tel:+1234567890">Call Us</a>
<a href="sms:+1234567890">Text Us</a>

<!-- Download link -->
<a href="file.pdf" download>Download PDF</a>

<!-- New tab/window -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Link in New Tab
</a>
```

**Link Attributes & Best Practices**

| Attribute | Purpose | Example | Best Practice |
|---|---|---|---|
| `href` | Link destination | `href="page.html"` | Always required |
| `target` | Where to open | `target="_blank"` | External links only |
| `rel` | Link relationship | `rel="noopener noreferrer"` | Required with `target="_blank"` |
| `title` | Tooltip text | `title="Visit our site"` | Descriptive and helpful |
| `download` | Download file | `download="filename.pdf"` | For file downloads |

**Navigation Structure**

```html
<nav aria-label="Main Navigation">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/services">Services</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>

<!-- Breadcrumb Navigation -->
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/products">Products</a></li>
    <li aria-current="page">Product Name</li>
  </ol>
</nav>
```

## 6. Images & Media — Complete Guide

**Responsive Images**

```html
<!-- Basic image -->
<img src="image.jpg" alt="Descriptive text">

<!-- Lazy loading -->
<img src="image.jpg" alt="Description" loading="lazy">

<!-- Responsive with sizes/srcset -->
<img src="image.jpg"
     alt="Description"
     sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 33vw"
     srcset="image-sm.jpg 480w, image-md.jpg 768w, image-lg.jpg 1200w">

<!-- Picture element for art direction -->
<picture>
  <source srcset="image-wide.jpg" media="(min-width: 1200px)">
  <source srcset="image-med.jpg" media="(min-width: 768px)">
  <img src="image-small.jpg" alt="Fallback">
</picture>

<!-- WebP format with fallback -->
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description">
</picture>
```

**Figure with Caption**

```html
<figure>
  <img src="chart.jpg" alt="Sales data 2024">
  <figcaption>
    <strong>Figure 1:</strong>
    Annual sales growth showing 35% increase
  </figcaption>
</figure>
```

**Audio & Video Elements**

```html
<!-- Audio -->
<audio controls>
  <source src="audio.mp3" type="audio/mpeg">
  <source src="audio.ogg" type="audio/ogg">
  Your browser doesn't support audio.
</audio>

<!-- Video -->
<video width="320" height="240" controls poster="thumb.jpg">
  <source src="video.mp4" type="video/mp4">
  <source src="video.webm" type="video/webm">
  Your browser doesn't support video.
</video>
```

**Image Best Practices**

- ✓ Always include descriptive alt text
- ✓ Use responsive images with `srcset`
- ✓ Lazy load images outside the viewport
- ✓ Use WebP format with fallbacks
- ✓ Optimize image file sizes
- ✓ Use `<picture>` for art direction
- ✓ Include `width`/`height` attributes
- ✓ Use semantic `figure`/`figcaption`
- ✗ Don't skip alt text for accessibility
- ✗ Don't use images for text

## 7. Lists — All Types

**Unordered List**

```html
<ul>
  <li>First item</li>
  <li>Second item</li>
  <li>Third item
    <ul>
      <li>Nested item 1</li>
      <li>Nested item 2</li>
    </ul>
  </li>
</ul>
```

**Ordered List with Options**

```html
<ol>
  <li>First step</li>
  <li>Second step</li>
  <li>Third step</li>
</ol>

<!-- Reversed -->
<ol reversed>
  <li>Step 3</li>
  <li>Step 2</li>
  <li>Step 1</li>
</ol>

<!-- Custom start -->
<ol start="5">
  <li>Item 5</li>
  <li>Item 6</li>
</ol>
```

**Definition List**

```html
<dl>
  <dt>HTML</dt>
  <dd>HyperText Markup Language</dd>

  <dt>CSS</dt>
  <dd>Cascading Style Sheets</dd>

  <dt>JavaScript</dt>
  <dd>Programming language for web</dd>
</dl>
```

## 8. Forms & Input Elements — Complete Reference

**Complete Form Example**

```html
<form action="/submit" method="POST" enctype="multipart/form-data">
  <fieldset>
    <legend>Contact Information</legend>

    <label for="name">Full Name:</label>
    <input type="text" id="name" name="name"
           required minlength="3" maxlength="50">

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <label for="phone">Phone:</label>
    <input type="tel" id="phone" name="phone"
           pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}">
  </fieldset>

  <fieldset>
    <legend>Preferences</legend>

    <label>
      <input type="checkbox" name="subscribe">
      Subscribe to newsletter
    </label>

    <label>
      <input type="radio" name="priority" value="high">
      High Priority
    </label>
  </fieldset>

  <button type="submit">Submit</button>
  <button type="reset">Clear</button>
</form>
```

**Input Types Reference**

| Input Type | Purpose | HTML Example | When to Use |
|---|---|---|---|
| `text` | Single line text | `<input type="text">` | Names, addresses |
| `password` | Hidden text input | `<input type="password">` | Passwords, PINs |
| `email` | Email validation | `<input type="email">` | Email addresses |
| `number` | Numeric only | `<input type="number" min="0" max="100">` | Quantities, ages |
| `range` | Slider control | `<input type="range" min="0" max="100" value="50">` | Volume, brightness |
| `date` | Date picker | `<input type="date">` | Birthdate, events |
| `time` | Time picker | `<input type="time">` | Clock times |
| `datetime-local` | Date & time | `<input type="datetime-local">` | Scheduling |
| `color` | Color picker | `<input type="color" value="#667eea">` | Color selection |
| `file` | File upload | `<input type="file" accept=".pdf,.jpg">` | File uploads |
| `search` | Search input | `<input type="search" placeholder="Search...">` | Search functionality |
| `url` | URL validation | `<input type="url">` | Website URLs |
| `tel` | Telephone number | `<input type="tel">` | Phone numbers |
| `checkbox` | Multiple selection | `<input type="checkbox" name="item">` | Multi-select options |
| `radio` | Single selection | `<input type="radio" name="item">` | Single choice only |

**Form Input Attributes**

| Attribute | Purpose | Example |
|---|---|---|
| `required` | Field must be filled | `<input required>` |
| `disabled` | Disable input | `<input disabled>` |
| `readonly` | Read-only field | `<input readonly>` |
| `placeholder` | Hint text | `<input placeholder="Enter name">` |
| `value` | Default value | `<input value="default">` |
| `minlength` | Minimum characters | `<input minlength="5">` |
| `maxlength` | Maximum characters | `<input maxlength="20">` |
| `min`/`max` | Min/max value | `<input type="number" min="0" max="100">` |
| `pattern` | Regex validation | `<input pattern="[A-Z]{3}">` |
| `autocomplete` | Browser autocomplete | `<input autocomplete="email">` |
| `autofocus` | Focus on page load | `<input autofocus>` |
| `accept` | File types allowed | `<input type="file" accept=".jpg,.png">` |
| `multiple` | Multiple selections | `<input type="file" multiple>` |

**Select Dropdown**

```html
<select name="category" required>
  <option value="">Select an option</option>
  <optgroup label="Fruits">
    <option value="apple">Apple</option>
    <option value="banana">Banana</option>
  </optgroup>
  <optgroup label="Vegetables">
    <option value="carrot">Carrot</option>
  </optgroup>
</select>
```

**Textarea**

```html
<textarea name="message"
          rows="4" cols="50"
          placeholder="Your message here..."
          required></textarea>
```

**Datalist (Autocomplete Suggestions)**

```html
<label for="datalist">Suggestions:</label>
<input list="suggestions" id="datalist">
<datalist id="suggestions">
  <option value="Option 1">
  <option value="Option 2">
</datalist>
```

## 9. Data Tables — Complete Guide

**Complete Table Example**

```html
<table>
  <caption>Monthly Sales Data 2024</caption>

  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Sales</th>
      <th scope="col">Growth</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>January</td>
      <td>$45,000</td>
      <td>+12%</td>
    </tr>
    <tr>
      <td>February</td>
      <td>$52,000</td>
      <td>+15%</td>
    </tr>
  </tbody>

  <tfoot>
    <tr>
      <th>Total</th>
      <td>$97,000</td>
      <td></td>
    </tr>
  </tfoot>
</table>
```

**Complex Table with Merging**

```html
<table border="1">
  <thead>
    <tr>
      <th rowspan="2">Product</th>
      <th colspan="3">Sales by Quarter</th>
    </tr>
    <tr>
      <th>Q1</th>
      <th>Q2</th>
      <th>Q3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Product A</td>
      <td>1000</td>
      <td>1500</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
```

**Table Best Practices**

- ✓ Use `<thead>`, `<tbody>`, `<tfoot>` sections
- ✓ Use `<th>` for headers with a `scope` attribute
- ✓ Include a table `<caption>` for clarity
- ✓ Use data attributes for sorting
- ✓ Make tables responsive with overflow
- ✓ Use appropriate borders for readability
- ✓ Ensure proper heading hierarchy
- ✗ Don't use tables for layout
- ✗ Don't skip headers or scope

## 10. Accessibility (WCAG 2.1) & ARIA

**Essential ARIA Attributes**

| ARIA Attribute | Purpose | Example | Priority |
|---|---|---|---|
| `aria-label` | Accessible name | `<button aria-label="Close menu">✕</button>` | High |
| `aria-labelledby` | Label from element | `<input aria-labelledby="label-id">` | High |
| `aria-describedby` | Description text | `<input aria-describedby="hint">` | Medium |
| `aria-live` | Dynamic updates | `<div aria-live="polite">` | High |
| `aria-hidden` | Hide from screen readers | `<span aria-hidden="true">` | Medium |
| `role` | Element semantic role | `<div role="button">` | High |
| `aria-required` | Required field | `<input aria-required="true">` | Medium |
| `aria-disabled` | Disabled state | `<button aria-disabled="true">` | Medium |
| `aria-current` | Current page | `<a aria-current="page">` | Medium |
| `aria-expanded` | Expandable state | `<button aria-expanded="false">` | Medium |

**Accessible Form Example**

```html
<form>
  <label for="email">
    Email Address <span aria-label="required">*</span>
  </label>
  <input type="email"
         id="email"
         name="email"
         aria-required="true"
         aria-describedby="email-hint">
  <small id="email-hint">
    We'll never share your email
  </small>

  <button type="submit" aria-label="Submit form">
    Submit
  </button>
</form>

<!-- Navigation with ARIA -->
<nav aria-label="Main Navigation">
  <ul role="menubar">
    <li><a href="/" aria-current="page">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

<!-- Live region for updates -->
<div aria-live="polite" aria-atomic="true" id="notification">
  Messages appear here
</div>
```

**WCAG 2.1 Checklist**

- ✓ All images have descriptive alt text
- ✓ Heading hierarchy is proper (h1 → h2 → h3)
- ✓ Color contrast ratio 4.5:1 (normal text)
- ✓ Color contrast ratio 3:1 (large text)
- ✓ All buttons and links keyboard accessible
- ✓ Form inputs have associated labels
- ✓ Focus indicators visible
- ✓ Video captions and transcripts provided
- ✓ ARIA labels for screen readers
- ✓ Semantic HTML elements used
- ✗ Don't rely on color alone
- ✗ Don't skip heading levels
- ✗ Don't use images for text

## 11. Meta Tags & SEO Optimization

**Complete Meta Tags Setup**

```html
<!-- Character Encoding -->
<meta charset="UTF-8">

<!-- Viewport for Responsive -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- SEO Meta Tags -->
<meta name="description" content="Page description 150-160 chars">
<meta name="keywords" content="keyword1, keyword2, keyword3">
<meta name="author" content="Your Name">

<!-- Social Media (Open Graph) -->
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Description">
<meta name="twitter:image" content="image.jpg">

<!-- Theme & Browser -->
<meta name="theme-color" content="#667eea">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

<!-- Robots & Crawling -->
<meta name="robots" content="index, follow">
<meta name="googlebot" content="index, follow">

<!-- Language -->
<meta http-equiv="Content-Language" content="en-US">
```

**SEO Best Practices**

- ✓ Use 50–60 character title tags
- ✓ Write 150–160 character descriptions
- ✓ Include the primary keyword in the title
- ✓ Use the H1 tag once per page
- ✓ Create descriptive URLs
- ✓ Use semantic HTML elements
- ✓ Optimize images with alt text
- ✓ Internal links with anchor text
- ✓ Mobile-friendly responsive design
- ✓ Fast page load speed
- ✗ Don't keyword stuff
- ✗ Don't use duplicate titles
- ✗ Don't hide text or links

## 12. Advanced HTML Features

**Using Custom Data Attributes**

```html
<div class="product"
     data-product-id="12345"
     data-price="99.99"
     data-category="electronics"
     data-in-stock="true">
  Product Name
</div>

<!-- Accessed via JavaScript -->
<script>
const product = document.querySelector('.product');
console.log(product.dataset.productId);      // "12345"
console.log(product.dataset.price);          // "99.99"
console.log(product.dataset.category);       // "electronics"
console.log(product.dataset.inStock);        // "true"
</script>
```

**Web Components: Template & Custom Elements**

```html
<template id="card-template">
  <style>
    .card {
      border: 1px solid #ddd;
      padding: 20px;
      border-radius: 8px;
    }
  </style>
  <div class="card">
    <h2><slot name="title"></slot></h2>
    <p><slot></slot></p>
  </div>
</template>

<!-- Custom Element Usage -->
<custom-card>
  <span slot="title">Card Title</span>
  Card content goes here
</custom-card>
```

**Structured Data (Schema.org JSON-LD)**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "image": "https://example.com/image.jpg",
  "datePublished": "2024-01-15",
  "dateModified": "2024-01-20",
  "author": {
    "@type": "Person",
    "name": "Author Name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Website Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  },
  "description": "Article description"
}
</script>

<!-- Product Schema -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": "image.jpg",
  "description": "Product description",
  "offers": {
    "@type": "Offer",
    "price": "99.99",
    "priceCurrency": "USD",
    "availability": "InStock"
  }
}
</script>
```

**Resource Hints & Performance Optimization**

```html
<!-- DNS Prefetch -->
<link rel="dns-prefetch" href="//cdn.example.com">

<!-- Preconnect -->
<link rel="preconnect" href="//api.example.com">
<link rel="preconnect" href="//fonts.googleapis.com">

<!-- Preload Critical Resources -->
<link rel="preload" as="font" href="font.woff2" type="font/woff2">
<link rel="preload" as="style" href="critical.css">

<!-- Prefetch for Next Page -->
<link rel="prefetch" href="next-page.html">

<!-- Lazy Load Images -->
<img src="image.jpg" loading="lazy" alt="Description">
```

## 13. Global Attributes Reference

| Attribute | Purpose | Values | Example |
|---|---|---|---|
| `id` | Unique element identifier | Any unique string | `<div id="main">` |
| `class` | CSS class(es) | Space-separated names | `<div class="container active">` |
| `style` | Inline CSS styles | CSS properties | `<div style="color: red;">` |
| `title` | Tooltip text | Any text | `<button title="Save file">` |
| `dir` | Text direction | `ltr`, `rtl`, `auto` | `<p dir="rtl">` |
| `lang` | Language code | Language codes | `<p lang="es">` |
| `tabindex` | Tab order | Integer (-1 to 32767) | `<button tabindex="0">` |
| `hidden` | Hide element | Boolean | `<div hidden>` |
| `contenteditable` | Editable content | `true`, `false`, `inherit` | `<div contenteditable="true">` |
| `spellcheck` | Spell checking | `true`, `false`, `default` | `<input spellcheck="true">` |
| `draggable` | Draggable element | `true`, `false`, `auto` | `<div draggable="true">` |

## 14. Common Mistakes & Best Practices

**Critical Mistakes to Avoid**

❌ MISTAKE: Missing Alt Text

```html
<!-- Wrong -->
<img src="photo.jpg">

<!-- Correct -->
<img src="photo.jpg" alt="Descriptive text">
```

❌ MISTAKE: Using Wrong Semantic Elements

```html
<!-- Wrong -->
<div class="heading">Title</div>

<!-- Correct -->
<h1>Title</h1>
```

❌ MISTAKE: Skipping Heading Levels

```html
<!-- Wrong -->
<h1>Title</h1><h3>Subtitle</h3>

<!-- Correct -->
<h1>Title</h1><h2>Subtitle</h2>
```

❌ MISTAKE: Missing Viewport Meta Tag

```html
<!-- Missing responsive design capability -->
<!-- Correct -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

❌ MISTAKE: Not Closing Tags Properly

```html
<!-- Wrong: unclosed tags cause layout issues -->
<p>Paragraph<div>

<!-- Correct -->
<p>Paragraph</p><div>...</div>
```

**HTML Best Practices**

DO's:
- ✓ Use semantic HTML
- ✓ Validate HTML regularly
- ✓ Use descriptive alt text
- ✓ Keep HTML organized
- ✓ Use proper indentation
- ✓ Include meta tags
- ✓ Test accessibility
- ✓ Use external CSS

DON'Ts:
- ✗ Don't use `<div>` for everything
- ✗ Don't skip alt text
- ✗ Don't use deprecated tags
- ✗ Don't nest incompatible elements
- ✗ Don't skip DOCTYPE
- ✗ Don't use inline CSS
- ✗ Don't ignore accessibility
- ✗ Don't use tables for layout

**Professional Tips**

- 💡 Tip 1 — Use Descriptive Names: Use meaningful `id` and `class` names (`header`, `main-nav`) not (`div1`, `div2`)
- 💡 Tip 2 — Mobile First: Design for mobile first, then enhance for larger screens
- 💡 Tip 3 — Validate Regularly: Use the W3C Validator: https://validator.w3.org
- 💡 Tip 4 — Performance Matters: Use lazy loading, optimize images, defer non-critical scripts
- 💡 Tip 5 — Test Accessibility: Use screen readers, test keyboard navigation, check contrast ratios

## 15. Quick Reference — Essential Commands

**HTML5 Essential Elements Quick Reference**

| Category | Element | Purpose | Use Case |
|---|---|---|---|
| Structure | `<header>` | Top section | Logo, navigation, banner |
| Structure | `<nav>` | Navigation | Menu links |
| Structure | `<main>` | Main content | Page content |
| Structure | `<footer>` | Bottom section | Copyright, links |
| Content | `<article>` | Self-contained | Blog posts, news |
| Content | `<section>` | Thematic group | Content sections |
| Content | `<h1>`–`<h6>` | Headings | Page structure |
| Content | `<p>` | Paragraph | Text content |
| Lists | `<ul><li>` | Unordered list | Bullet points |
| Lists | `<ol><li>` | Ordered list | Numbered items |
| Forms | `<form>` | Form container | User input |
| Forms | `<input>` | Input field | Various input types |
| Forms | `<label>` | Field label | Form accessibility |
| Forms | `<button>` | Clickable button | Form submission |
| Media | `<img>` | Images | Pictures, graphics |
| Media | `<video>` | Video content | Video playback |
| Media | `<audio>` | Audio content | Sound/music |
| Embedded | `<iframe>` | Embedded page | Maps, videos, widgets |
| Tables | `<table>` | Data table | Tabular data |

**Meta Tags & Head Quick Reference**

Essential meta tags:
- `charset="UTF-8"`
- `viewport` for responsive design
- `description` for SEO
- Open Graph tags
- Twitter Card tags
- `theme-color`

Common attributes:
- `required`, `disabled`
- `autocomplete`, `autofocus`
- `placeholder`, `value`
- `min`, `max`, `pattern`
- `accept`, `multiple`

---
*Source: adapted from the HTML cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
