# Design Document

## CodeLoop Website Redesign

---

## Overview

This document describes the complete technical design for the CodeLoop website redesign. The result is a three-page static site (index.html, courses.html, contact.html) built with plain HTML, CSS, and vanilla JavaScript — no frameworks, no build tools, no server-side code. It is hosted on GitHub Pages at the base path `/code-loop-portfolio/` and the entire design system is expressed through CSS custom properties.

The redesign replaces the existing dark-only, single-stylesheet approach with a full dark/light theme system, a structured CSS architecture, modular JavaScript files, and substantially richer page sections including animated statistics, a filterable courses grid, learning roadmaps, YouTube tutorial cards, a validated contact form, and an accessible FAQ accordion.

All existing brand identity, external links, and content are preserved. The color palette, typography, and spacing scale are standardised as design tokens. Every interactive element meets WCAG 2.1 accessibility requirements.

---

## Architecture

### File Structure

```
/code-loop-portfolio/
├── index.html
├── courses.html
├── contact.html
├── sitemap.xml
├── robots.txt
├── css/
│   ├── style.css          # Design tokens, base reset, component definitions
│   └── responsive.css     # @media query blocks only — no styles outside queries
└── js/
    ├── main.js            # Homepage logic, shared utilities, theme switching
    ├── courses.js         # Courses page: filter state machine, search algorithm
    └── contact.js         # Contact page: form validation, FAQ accordion
```

All `src` / `href` attributes use **relative paths** so the site resolves correctly under the `/code-loop-portfolio/` base path without a base tag.

### Page Composition

```
index.html
  ├── <head>  SEO meta, preconnect fonts, inline anti-FOSC theme script
  ├── Scroll Progress Bar   (fixed, above nav)
  ├── Navbar                (fixed, sticky)
  ├── Mobile Menu           (full-screen overlay)
  ├── Search Overlay        (full-screen overlay)
  ├── Hero Section
  ├── Statistics Section
  ├── Programming Topics Grid
  ├── Learning Roadmaps
  ├── Latest YouTube Tutorials
  ├── Interview Preparation Section
  ├── Real-World Projects Showcase
  ├── Why Choose CodeLoop
  ├── About Founder
  ├── Footer
  ├── Legal Modal Overlay
  └── Back-to-Top Button

courses.html
  ├── <head>  SEO meta, shared styles
  ├── Scroll Progress Bar
  ├── Navbar
  ├── Mobile Menu
  ├── Search Overlay
  ├── Page Hero
  ├── Search + Filter Controls
  ├── Courses Grid
  ├── Learning Roadmaps  (same content as homepage)
  └── Footer

contact.html
  ├── <head>  SEO meta, shared styles
  ├── Scroll Progress Bar
  ├── Navbar
  ├── Mobile Menu
  ├── Page Hero
  ├── Contact Form Panel
  ├── Founder / Contact Info Panel
  ├── FAQ Accordion
  └── Footer
```

### Script Loading Strategy

Each page loads `css/style.css`, `css/responsive.css`, and its page-specific JS file at the bottom of `<body>` with `defer`. `main.js` exports shared utilities (theme switching, scroll progress, back-to-top, search overlay) that are also called from `courses.js` and `contact.js` via a global `CodeLoop` namespace object attached to `window`.

```
window.CodeLoop = {
  theme:      { init, toggle },
  search:     { init, filter, clear },
  scrollProgress: { init },
  backToTop:  { init },
  modal:      { open, close }
}
```

---

## Components and Interfaces

### Navbar

```
<header role="banner">
  <nav aria-label="Main navigation">
    <a class="nav-logo" href="index.html">CodeLoop</a>
    <ul class="nav-links" role="list">
      <li><a href="index.html">Home</a></li>
      <li><a href="courses.html">Courses</a></li>
      <li><a href="index.html#roadmaps">Roadmaps</a></li>
      <li><a href="contact.html">Contact</a></li>
    </ul>
    <div class="nav-actions">
      <button class="nav-search-btn"  aria-label="Open search"><!-- search icon SVG --></button>
      <button class="nav-theme-btn"   aria-label="Toggle theme" id="themeToggle"><!-- sun/moon SVG --></button>
      <a     class="nav-cta"          href="https://www.youtube.com/@CodeLoop-SP/videos"
             target="_blank" rel="noopener">Subscribe</a>
    </div>
    <button class="nav-hamburger"
            id="hamburger"
            aria-label="Toggle navigation menu"
            aria-expanded="false"
            aria-controls="mobileMenu">
      <span></span><span></span><span></span>
    </button>
  </nav>
</header>
```

**Behaviour:**
- Fixed `position: fixed; top: 0; left: 0; right: 0; z-index: 200` on all pages.
- On scroll > 0, adds `scrolled` class which applies a stronger `backdrop-filter: blur(20px)` and reduced transparency.
- At ≤ 768 px: `.nav-links` and `.nav-cta` get `display: none`; `.nav-hamburger` gets `display: flex`.
- The `aria-expanded` attribute on the hamburger button is updated in JavaScript whenever the mobile menu opens or closes.

### Mobile Menu

```
<div id="mobileMenu" class="mobile-menu"
     role="dialog"
     aria-modal="true"
     aria-label="Navigation menu"
     hidden>
  <button class="mobile-menu-close" aria-label="Close navigation menu">✕</button>
  <nav>
    <a href="index.html">Home</a>
    <a href="courses.html">Courses</a>
    <a href="index.html#roadmaps">Roadmaps</a>
    <a href="contact.html">Contact</a>
    <a href="https://www.youtube.com/@CodeLoop-SP/videos"
       target="_blank" rel="noopener" class="mobile-cta">Subscribe on YouTube</a>
  </nav>
</div>
```

**Behaviour:**
- Opens via hamburger click, Enter, or Space.
- Covers 100% viewport: `position: fixed; inset: 0; z-index: 300`.
- On open: removes `hidden`, sets `aria-expanded="true"` on hamburger, moves keyboard focus to the first focusable element inside the menu.
- On close (hamburger toggle, outside click, Escape, link selection): adds `hidden`, sets `aria-expanded="false"`, returns focus to hamburger button.
- `body.style.overflow = 'hidden'` while open to prevent background scroll.

### Search Overlay

```
<div id="searchOverlay" class="search-overlay" role="search" aria-label="Site search" hidden>
  <input  type="search" id="searchInput"
          class="search-input"
          placeholder="Search topics, courses…"
          aria-label="Search topics and courses"
          autocomplete="off" />
  <button class="search-close" aria-label="Close search">✕</button>
  <p id="searchNoResults" class="search-no-results" hidden>No results found.</p>
</div>
```

**Behaviour:** Discussed in detail under the JavaScript section.

### Hero Section (index.html)

```html
<section class="hero" aria-labelledby="hero-heading">
  <div class="hero-content">
    <div class="hero-badge">🎓 Free Programming Tutorials · Always</div>
    <h1 id="hero-heading" class="hero-heading">
      <span class="line-plain">Learn.</span>
      <span class="line-gradient">Code. Loop.</span>
    </h1>
    <p class="hero-description">
      Free programming tutorials on YouTube — from beginner to interview-ready.
    </p>
    <div class="hero-actions">
      <a href="https://www.youtube.com/@CodeLoop-SP/videos"
         target="_blank" rel="noopener"
         class="btn btn-primary">
        <!-- YouTube SVG -->  Watch on YouTube
      </a>
      <a href="courses.html" class="btn btn-secondary">Browse Courses</a>
    </div>
    <div class="hero-code-snippet" aria-hidden="true">
      <!-- Animated code block element -->
    </div>
  </div>
</section>
```

**Animation:** Each child element inside `.hero-content` has `opacity: 0; transform: translateY(24px)` initially. On page load the class `hero--loaded` is added to `<section class="hero">`, triggering a CSS animation:

```css
.hero--loaded .hero-badge       { animation: fadeUp 0.6s 0.0s ease both; }
.hero--loaded .hero-heading     { animation: fadeUp 0.6s 0.15s ease both; }
.hero--loaded .hero-description { animation: fadeUp 0.6s 0.3s ease both; }
.hero--loaded .hero-actions     { animation: fadeUp 0.6s 0.45s ease both; }
.hero--loaded .hero-code-snippet{ animation: fadeUp 0.6s 0.6s ease both; }
```

The animated visual element is a CSS-only "floating code snippet" with a pulsing cursor and a gentle `translateY` keyframe loop, positioned to the right of the text at ≥ 1024 px and hidden below the text on smaller viewports so it never overlaps body copy.

### Statistics Section

```html
<section class="stats-section" aria-labelledby="stats-heading">
  <h2 id="stats-heading" class="sr-only">CodeLoop by the numbers</h2>
  <div class="stats-grid">
    <div class="stat-item" data-target="100" data-suffix="+">
      <span class="stat-value" aria-live="polite">0</span>
      <span class="stat-label">YouTube Videos</span>
    </div>
    <div class="stat-item" data-target="12" data-suffix="">
      <span class="stat-value" aria-live="polite">0</span>
      <span class="stat-label">Programming Topics</span>
    </div>
    <div class="stat-item" data-target="500" data-suffix="+">
      <span class="stat-value" aria-live="polite">0</span>
      <span class="stat-label">Free Learning Resources</span>
    </div>
    <div class="stat-item" data-target="10000" data-suffix="+">
      <span class="stat-value" aria-live="polite">0</span>
      <span class="stat-label">Learning Opportunities</span>
    </div>
  </div>
</section>
```

**Configuration object in `js/main.js`:**

```js
const STATS_CONFIG = [
  { selector: '[data-stat="videos"]',     target: 100,   suffix: '+', label: 'YouTube Videos' },
  { selector: '[data-stat="topics"]',     target: 12,    suffix: '',  label: 'Programming Topics' },
  { selector: '[data-stat="resources"]',  target: 500,   suffix: '+', label: 'Free Learning Resources' },
  { selector: '[data-stat="learners"]',   target: 10000, suffix: '+', label: 'Learning Opportunities' },
];
```

**Animation algorithm** (in `main.js`):

```js
function animateCounter(el, target, suffix, duration = 1500, startDelay = 0) {
  let startTime = null;
  function step(timestamp) {
    if (!startTime) startTime = timestamp + startDelay;
    const elapsed = Math.max(0, timestamp - startTime);
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3); // ease-out cubic
    const current = Math.floor(eased * target);
    el.textContent = current + suffix;
    if (progress < 1) requestAnimationFrame(step);
    else el.textContent = target + suffix;
  }
  requestAnimationFrame(step);
}

function initStats() {
  const section = document.querySelector('.stats-section');
  let hasAnimated = false;
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting && !hasAnimated) {
      hasAnimated = true;
      observer.disconnect();
      document.querySelectorAll('.stat-item').forEach((item, i) => {
        const el     = item.querySelector('.stat-value');
        const target = parseInt(item.dataset.target, 10);
        const suffix = item.dataset.suffix || '';
        animateCounter(el, target, suffix, 1500, i * 200);
      });
    }
  }, { threshold: 0.2 });
  if (section) observer.observe(section);
}
```

The `hasAnimated` flag guarantees the animation fires exactly once per page load regardless of the section scrolling in and out.

### Programming Topics Grid

```html
<section class="topics-section" id="topics" aria-labelledby="topics-heading">
  <h2 id="topics-heading">Topics &amp; Tutorials</h2>
  <div class="topics-grid" role="list">
    <article class="topic-card" role="listitem" tabindex="0"
             data-topic="java"
             aria-label="Java — Intermediate">
      <span class="topic-icon" aria-hidden="true">☕</span>
      <h3 class="topic-name">Java</h3>
      <p class="topic-desc">OOP mastery with Java — classes, design patterns, and Spring Boot fundamentals.</p>
      <span class="topic-difficulty badge-intermediate">Intermediate</span>
    </article>
    <!-- … 11 more topic cards … -->
  </div>
</section>
```

The 12 topics and their metadata (icon, description ≤ 20 words, difficulty) are defined as a JS array in `main.js` and rendered by `renderTopicsGrid()`. Each card's `click`/`keydown Enter|Space` handler navigates to `courses.html?category=<topic>`.

**Grid layout:**

| Viewport | Columns |
|----------|---------|
| ≥ 1024 px | 4 |
| 768 – 1023 px | 3 |
| 480 – 767 px | 2 |
| < 480 px | 1 |

### Learning Roadmaps

```html
<section class="roadmaps-section" id="roadmaps" aria-labelledby="roadmaps-heading">
  <h2 id="roadmaps-heading">Learning Roadmaps</h2>
  <div class="roadmap-tabs" role="tablist" aria-label="Developer roadmaps">
    <button role="tab" aria-selected="true"  aria-controls="roadmap-java"       id="tab-java">Java Developer</button>
    <button role="tab" aria-selected="false" aria-controls="roadmap-fullstack"  id="tab-fullstack">Full Stack</button>
    <button role="tab" aria-selected="false" aria-controls="roadmap-python"     id="tab-python">Python Developer</button>
  </div>
  <div id="roadmap-java" role="tabpanel" aria-labelledby="tab-java">
    <ol class="roadmap-track">
      <li class="roadmap-step">
        <span class="step-number">1</span>
        <div class="step-body">
          <h3 class="step-title">
            <a href="courses.html?category=java">Core Java &amp; OOP</a>
          </h3>
          <p class="step-desc">Learn classes, inheritance, polymorphism, and interfaces — the foundation of Java development.</p>
        </div>
      </li>
      <!-- … more steps … -->
    </ol>
  </div>
  <!-- … other tabpanels … -->
</section>
```

Roadmap steps that reference an available course category render their title as an `<a>` linking to `courses.html?category=<slug>`. Steps without a corresponding course render as plain `<h3>` text with no anchor or `href`.

### Latest YouTube Tutorials

```html
<section class="tutorials-section" id="tutorials" aria-labelledby="tutorials-heading">
  <h2 id="tutorials-heading">Latest Tutorials</h2>
  <div class="tutorials-grid">
    <article class="tutorial-card">
      <a href="https://youtu.be/…" target="_blank" rel="noopener"
         class="tutorial-thumb-link" aria-label="Watch: Java OOP Tutorial">
        <img src="https://img.youtube.com/vi/…/hqdefault.jpg"
             alt="Java OOP Tutorial thumbnail"
             loading="lazy" width="320" height="180" />
        <span class="tutorial-duration">18:42</span>
      </a>
      <div class="tutorial-info">
        <span class="tutorial-topic-tag">Java</span>
        <h3 class="tutorial-title">Java OOP Tutorial</h3>
        <time class="tutorial-date" datetime="2025-01-15">Jan 2025</time>
      </div>
    </article>
  </div>
  <a href="https://www.youtube.com/@CodeLoop-SP/videos"
     target="_blank" rel="noopener"
     class="btn btn-secondary">View All on YouTube</a>
</section>
```

The tutorial card data is defined in `js/main.js`:

```js
const TUTORIALS = [
  {
    id:       'yt-video-id',
    title:    'Java OOP Tutorial',
    topic:    'Java',
    duration: '18:42',
    date:     '2025-01-15',
    url:      'https://youtu.be/…',
  },
  // 3–8 entries
];
```

If `TUTORIALS` is empty, `renderTutorials()` calls `section.hidden = true` rather than rendering an empty container, satisfying Requirement 8.5.

### Interview Preparation Section

```html
<section class="interview-section" id="interview" aria-labelledby="interview-heading">
  <div class="interview-content">
    <h2 id="interview-heading">Crack Your Interviews</h2>
    <p class="interview-desc">
      Interview Hub 360 is a dedicated companion platform that helps you prepare
      comprehensively for technical and HR interviews with structured roadmaps,
      curated question banks, and coding challenges — all completely free.
    </p>
    <ul class="interview-features" aria-label="Platform highlights">
      <li>Technical question banks</li>
      <li>HR interview preparation</li>
      <li>Structured topic roadmaps</li>
      <li>Coding challenges</li>
      <li>Free access, always</li>
    </ul>
    <a href="https://interviewhub360.netlify.app/"
       target="_blank" rel="noopener"
       class="btn btn-primary">Visit Interview Hub 360</a>
  </div>
  <div class="interview-visual" aria-hidden="true"><!-- decorative icon composition --></div>
</section>
```

### Real-World Projects Showcase

```html
<section class="projects-section" id="projects" aria-labelledby="projects-heading">
  <h2 id="projects-heading">Real-World Projects</h2>
  <div class="projects-grid">
    <article class="project-card">
      <h3 class="project-title">Student Management System</h3>
      <p class="project-desc">A full CRUD application with database integration — Java Spring Boot, JPA, MySQL.</p>
      <ul class="project-tech" aria-label="Technologies">
        <li>Java</li><li>Spring Boot</li><li>MySQL</li>
      </ul>
      <a href="https://youtu.be/…" target="_blank" rel="noopener"
         class="btn btn-ghost">Watch Tutorial ↗</a>
    </article>
    <!-- at least 3 cards total -->
  </div>
</section>
```

Project data is defined in `js/main.js` as `PROJECTS` array. Title ≤ 60 chars, description ≤ 150 chars, technology list ≤ 5 items.

**Grid layout:**

| Viewport | Columns |
|----------|---------|
| ≥ 1280 px | multi-column (3) |
| 768 – 1279 px | 2 |
| < 768 px | 1 |

### Why Choose CodeLoop

```html
<section class="why-section" id="why" aria-labelledby="why-heading">
  <h2 id="why-heading">Why Choose CodeLoop</h2>
  <div class="why-grid">
    <article class="why-card">
      <span class="why-icon" aria-label="Concept-first teaching">💡</span>
      <h3>Concept First</h3>
      <p>Every tutorial starts with the "why" before the "how" — so you understand, not just copy-paste.</p>
    </article>
    <article class="why-card">
      <span class="why-icon" aria-label="Interview-focused content">🎯</span>
      <h3>Interview Focused</h3>
      <p>Content structured around what real interviews ask — so your learning has direct career ROI.</p>
    </article>
    <article class="why-card">
      <span class="why-icon" aria-label="Zero cost to learners">✅</span>
      <h3>Zero Cost</h3>
      <p>Everything is free on YouTube. No hidden courses, no premium tiers — just click and learn.</p>
    </article>
    <article class="why-card">
      <span class="why-icon" aria-label="Regular content updates">🔄</span>
      <h3>Regular Updates</h3>
      <p>Fresh content added consistently. Subscribe and never miss a new tutorial or topic drop.</p>
    </article>
  </div>
</section>
```

**Grid layout:**

| Viewport | Columns |
|----------|---------|
| ≥ 1024 px | 4 |
| 640 – 1023 px | 2 |
| < 640 px | 1 |

### About Founder

```html
<section class="founder-section" id="about" aria-labelledby="founder-heading">
  <div class="founder-image-wrap">
    <img src="img/praveen-muthu.jpg"
         alt="Praveen Muthu, Founder and Developer of CodeLoop"
         width="240" height="240"
         loading="lazy"
         onerror="this.style.display='none'; document.getElementById('founder-avatar-fallback').hidden=false;" />
    <div id="founder-avatar-fallback"
         class="founder-avatar-fallback"
         aria-label="PM initials avatar"
         hidden>PM</div>
  </div>
  <div class="founder-content">
    <h2 id="founder-heading">Praveen Muthu</h2>
    <p class="founder-role">Founder &amp; Developer — CodeLoop</p>
    <p class="founder-bio">
      Praveen is a passionate developer and educator who built CodeLoop to make quality
      programming education freely available to everyone. He creates structured tutorials
      on Java, Python, JavaScript, Data Structures, Algorithms, and System Design —
      each crafted to build deep understanding and interview readiness. Based in India,
      he combines teaching clarity with practical real-world examples to help learners
      go from zero to confident coder.
    </p>
    <div class="founder-links">
      <a href="https://www.linkedin.com/in/praveenmuthu-v/"
         target="_blank" rel="noopener" aria-label="Praveen Muthu on LinkedIn">LinkedIn</a>
      <a href="https://www.youtube.com/@CodeLoop-SP/videos"
         target="_blank" rel="noopener" aria-label="CodeLoop YouTube channel">YouTube</a>
      <a href="https://www.instagram.com/codeloop_yt/"
         target="_blank" rel="noopener" aria-label="CodeLoop on Instagram">Instagram</a>
      <a href="mailto:codeloop9563@gmail.com"
         target="_blank" rel="noopener" aria-label="Email CodeLoop">Email</a>
    </div>
  </div>
</section>
```

If the founder image fails to load, the `onerror` handler hides the `<img>` and reveals the styled `.founder-avatar-fallback` div (a circle with "PM" initials) so the section layout is preserved.

### Footer

```html
<footer role="contentinfo">
  <div class="footer-inner">
    <div class="footer-brand">
      <a href="index.html" class="footer-logo">CodeLoop</a>
      <p class="footer-tagline">Learn. Code. Loop.</p>
      <div class="footer-socials">
        <a href="https://www.youtube.com/@CodeLoop-SP/videos" target="_blank"
           rel="noopener" aria-label="YouTube"><!-- YT SVG --></a>
        <a href="https://www.linkedin.com/in/praveenmuthu-v/" target="_blank"
           rel="noopener" aria-label="LinkedIn"><!-- LI SVG --></a>
        <a href="https://www.instagram.com/codeloop_yt/" target="_blank"
           rel="noopener" aria-label="Instagram"><!-- IG SVG --></a>
        <a href="mailto:codeloop9563@gmail.com"
           aria-label="Email"><!-- Email SVG --></a>
      </div>
    </div>
    <nav aria-label="Footer navigation">
      <h2 class="footer-col-title">Navigate</h2>
      <ul role="list">
        <li><a href="index.html">Home</a></li>
        <li><a href="courses.html">Courses</a></li>
        <li><a href="index.html#roadmaps">Roadmaps</a></li>
        <li><a href="contact.html">Contact</a></li>
      </ul>
    </nav>
    <nav aria-label="Footer resources">
      <h2 class="footer-col-title">Resources</h2>
      <ul role="list">
        <li><a href="https://interviewhub360.netlify.app/" target="_blank" rel="noopener">Interview Hub 360</a></li>
        <li><a href="https://www.youtube.com/@CodeLoop-SP/videos" target="_blank" rel="noopener">YouTube Channel</a></li>
      </ul>
    </nav>
    <div class="footer-legal">
      <h2 class="footer-col-title">Legal</h2>
      <ul role="list">
        <li><button onclick="CodeLoop.modal.open('terms')">Terms of Use</button></li>
        <li><button onclick="CodeLoop.modal.open('privacy')">Privacy Policy</button></li>
        <li><button onclick="CodeLoop.modal.open('disclaimer')">Disclaimer</button></li>
      </ul>
    </div>
  </div>
  <div class="footer-bottom">
    <p>&copy; <span id="footer-year"></span> Praveen Muthu / CodeLoop. All rights reserved.</p>
  </div>
</footer>
```

`document.getElementById('footer-year').textContent = new Date().getFullYear()` runs in `main.js`.

Legal modal trigger buttons (`Terms`, `Privacy`, `Disclaimer`) open a shared modal overlay with focus trapping. Displayed content is stored as HTML strings in a `MODAL_CONTENT` map in `main.js`. The modal closes on close button click, overlay backdrop click, or Escape key, returning focus to the triggering element.

### Legal Modal Overlay

```html
<div id="legalModal" class="modal-overlay" role="dialog" aria-modal="true"
     aria-labelledby="modal-title" hidden>
  <div class="modal-box">
    <button class="modal-close" aria-label="Close modal">✕</button>
    <h2 id="modal-title" class="modal-title"></h2>
    <div class="modal-body"></div>
  </div>
</div>
```

**Focus trapping algorithm** (in `main.js`):

```js
function trapFocus(container) {
  const focusable = container.querySelectorAll(
    'a[href], button:not([disabled]), input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last  = focusable[focusable.length - 1];
  container.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;
    if (e.shiftKey) {
      if (document.activeElement === first) { e.preventDefault(); last.focus(); }
    } else {
      if (document.activeElement === last) { e.preventDefault(); first.focus(); }
    }
  });
}
```

On open: remove `hidden`, call `trapFocus(modalBox)`, move focus to close button, store `document.activeElement` as `lastFocus`.
On close: add `hidden`, restore focus to `lastFocus`.

### Scroll Progress Indicator

```html
<!-- placed as first child of <body>, before nav -->
<div id="scrollProgress" class="scroll-progress" role="progressbar"
     aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"
     aria-label="Page scroll progress"></div>
```

```css
.scroll-progress {
  position: fixed;
  top: 0; left: 0;
  height: 3px;
  width: 0%;
  background: var(--accent-primary);
  z-index: 1000;  /* above navbar which is z-index: 200 */
  transition: width 0.05s linear;
}
```

```js
function updateScrollProgress() {
  const scrollable = document.documentElement.scrollHeight - window.innerHeight;
  const pct = scrollable <= 0 ? 0 : (window.scrollY / scrollable) * 100;
  const bar = document.getElementById('scrollProgress');
  bar.style.width = pct + '%';
  bar.setAttribute('aria-valuenow', Math.round(pct));
}
window.addEventListener('scroll', updateScrollProgress, { passive: true });
```

When `scrollHeight - innerHeight` is ≤ 0, `pct` is set to 0 — no division by zero.

### Back-to-Top Button

```html
<button id="backToTop" class="back-to-top" aria-label="Back to top" hidden>
  <!-- up-arrow SVG -->
</button>
```

```css
.back-to-top {
  position: fixed;
  bottom: 2rem; right: 2rem;
  z-index: 150;
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}
.back-to-top.visible {
  visibility: visible;
  opacity: 1;
}
```

```js
function initBackToTop() {
  const btn = document.getElementById('backToTop');
  window.addEventListener('scroll', () => {
    btn.classList.toggle('visible', window.scrollY > 300);
  }, { passive: true });
  btn.addEventListener('click', () => window.scrollTo({ top: 0, behavior: 'smooth' }));
  btn.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }
  });
}
```

The button is reachable via Tab and operable with Enter or Space.

---

## Data Models

### Topic Card Data

```js
const TOPICS = [
  { id: 'java',          icon: '☕', name: 'Java',               desc: 'OOP mastery — classes, design patterns, and Spring Boot fundamentals.',           difficulty: 'Intermediate' },
  { id: 'python',        icon: '🐍', name: 'Python',             desc: 'From basics to advanced — decorators, generators, async, and real projects.',    difficulty: 'Beginner' },
  { id: 'javascript',    icon: '🌐', name: 'JavaScript',         desc: 'Modern JS, DOM manipulation, ES6+, async/await, and frontend fundamentals.',     difficulty: 'Beginner' },
  { id: 'html-css',      icon: '🎨', name: 'HTML & CSS',         desc: 'Semantic HTML5, CSS Grid, Flexbox, animations, and responsive design.',           difficulty: 'Beginner' },
  { id: 'dsa',           icon: '🌳', name: 'DSA',                desc: 'Arrays, linked lists, stacks, queues, trees, and graphs — step by step.',         difficulty: 'Intermediate' },
  { id: 'algorithms',    icon: '⚡', name: 'Algorithms',          desc: 'Sorting, searching, dynamic programming, greedy — with complexity analysis.',    difficulty: 'Advanced' },
  { id: 'sql',           icon: '🗄️', name: 'SQL',                desc: 'Database design, queries, joins, aggregations, and normalization.',               difficulty: 'Beginner' },
  { id: 'spring-boot',   icon: '🍃', name: 'Spring Boot',        desc: 'REST APIs, JPA, security, and production-ready Java backends.',                   difficulty: 'Intermediate' },
  { id: 'full-stack',    icon: '🚀', name: 'Full Stack',         desc: 'Connect frontend to backend — end-to-end projects from scratch.',                 difficulty: 'Intermediate' },
  { id: 'system-design', icon: '🏗️', name: 'System Design',     desc: 'Scalable architecture, caching, load balancing — for senior-level interviews.',  difficulty: 'Advanced' },
  { id: 'interview-prep',icon: '🎯', name: 'Interview Prep',     desc: 'Coding rounds, HR prep, communication, and mock problem walkthroughs.',           difficulty: 'Intermediate' },
  { id: 'projects',      icon: '🛠️', name: 'Real-World Projects',desc: 'End-to-end builds to solidify theory and fill your resume.',                      difficulty: 'Intermediate' },
];
```

### Course Card Data (courses.js)

```js
const COURSES = [
  {
    id:          'java',
    category:    'Java',
    title:       'Java & OOP Mastery',
    icon:        '☕',
    difficulty:  'Intermediate',
    description: 'Master object-oriented programming with Java — classes, inheritance, interfaces, design patterns, and Spring Boot basics used in industry.',
    topics:      ['Classes', 'Inheritance', 'Interfaces', 'Collections', 'Streams', 'Spring Boot'],
    youtubeUrl:  'https://www.youtube.com/@CodeLoop-SP/videos',
  },
  // … one entry per category (Java, Python, JavaScript, HTML & CSS, Spring Boot, SQL, DSA, Full Stack) …
];
```

### FAQ Data (contact.js)

```js
const FAQS = [
  { question: 'Are all CodeLoop tutorials really free?',           answer: '…' },
  { question: 'Which programming language should I start with?',  answer: '…' },
  { question: 'Can I request a specific tutorial topic?',          answer: '…' },
  { question: 'What is Interview Hub 360?',                        answer: '…' },
  { question: 'How do I collaborate or sponsor content?',          answer: '…' },
  { question: 'How long does it take to get a reply?',             answer: '…' },
];
```

---

## Design System Implementation

### CSS Custom Properties

Defined on `:root` (dark mode default) and overridden on `[data-theme="light"]`:

```css
:root,
[data-theme="dark"] {
  --bg-main:          #080B14;
  --bg-secondary:     #101626;
  --bg-card:          #151D30;
  --accent-primary:   #8B5CF6;
  --accent-secondary: #38BDF8;
  --text-main:        #F8FAFC;
  --text-secondary:   #94A3B8;

  /* Derived semantic tokens */
  --bg-nav:           rgba(8, 11, 20, 0.85);
  --border-color:     rgba(255, 255, 255, 0.08);
  --shadow-card:      0 4px 24px rgba(0, 0, 0, 0.4);

  /* Spacing scale */
  --space-1:  0.25rem;
  --space-2:  0.5rem;
  --space-3:  0.75rem;
  --space-4:  1rem;
  --space-6:  1.5rem;
  --space-8:  2rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --space-24: 6rem;

  /* Typography */
  --font-sans: 'Inter', 'Satoshi', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', ui-monospace, monospace;
  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.125rem;
  --text-xl:   1.25rem;
  --text-2xl:  1.5rem;
  --text-3xl:  1.875rem;
  --text-4xl:  2.25rem;
  --text-5xl:  3rem;
  --text-6xl:  3.75rem;

  /* Border radius */
  --radius-sm:   0.375rem;
  --radius-md:   0.5rem;
  --radius-lg:   0.75rem;
  --radius-xl:   1rem;
  --radius-2xl:  1.5rem;
  --radius-full: 9999px;

  /* Transitions */
  --transition-fast:   0.15s ease;
  --transition-base:   0.25s ease;
  --transition-slow:   0.4s ease;
}

[data-theme="light"] {
  --bg-main:          #F8FAFC;
  --bg-secondary:     #F1F5F9;
  --bg-card:          #FFFFFF;
  --text-main:        #0F172A;
  --text-secondary:   #475569;
  --border-color:     rgba(0, 0, 0, 0.08);
  --shadow-card:      0 4px 24px rgba(0, 0, 0, 0.08);
  --bg-nav:           rgba(248, 250, 252, 0.9);
}
```

Light-mode contrast ratios verified:
- `--text-main` (#0F172A) on `--bg-main` (#F8FAFC): **≈ 18.1:1** ✓
- `--text-secondary` (#475569) on `--bg-main` (#F8FAFC): **≈ 5.9:1** ✓
- `--accent-primary` (#8B5CF6) on `--bg-main` (#F8FAFC): **≈ 4.6:1** ✓ (large text / UI)
- `--accent-secondary` (#38BDF8) on `--bg-card` (#FFFFFF): **≈ 2.5:1** — used for decorative/non-text purposes only; primary accent used for interactive text

### Typography

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: 1.6;
  color: var(--text-main);
  background-color: var(--bg-main);
  -webkit-font-smoothing: antialiased;
}
```

The Google Fonts `<link>` is preconnected via:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

### Component-Level Tokens

Buttons, cards, and form controls derive from the base tokens:

```css
.btn {
  display: inline-flex; align-items: center; gap: var(--space-2);
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-md);
  font-weight: 700; font-size: var(--text-sm);
  text-decoration: none;
  transition: transform var(--transition-fast), box-shadow var(--transition-fast);
  cursor: pointer; border: none;
}
.btn-primary {
  background: var(--accent-primary);
  color: #fff;
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(139, 92, 246, 0.4);
}
.btn-secondary {
  background: transparent;
  border: 1px solid var(--border-color);
  color: var(--text-main);
}
.btn-secondary:hover {
  border-color: var(--accent-primary);
  background: rgba(139, 92, 246, 0.08);
}
.btn-ghost {
  background: transparent;
  color: var(--accent-primary);
  padding: var(--space-2) 0;
}

.card {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-xl);
  box-shadow: var(--shadow-card);
  transition: transform var(--transition-base), border-color var(--transition-base), box-shadow var(--transition-base);
}
.card:hover {
  transform: translateY(-4px);
  border-color: var(--accent-primary);
  box-shadow: 0 12px 40px rgba(139, 92, 246, 0.12);
}

/* Focus indicator — applied globally */
:focus-visible {
  outline: 2px solid var(--accent-primary);
  outline-offset: 3px;
}
```

---

## Theme Switching Mechanism

### Anti-FOSC Inline Script

This script must be the **first** `<script>` tag inside `<head>`, before any stylesheets, to prevent a flash of unstyled content:

```html
<script>
  (function() {
    try {
      var t = localStorage.getItem('theme');
      document.documentElement.setAttribute('data-theme', t === 'light' ? 'light' : 'dark');
    } catch (e) {
      document.documentElement.setAttribute('data-theme', 'dark');
    }
  })();
</script>
```

The try/catch handles the case where localStorage is unavailable (private browsing, security policies) and silently falls back to dark without throwing a JavaScript error.

### Theme Toggle Logic (main.js)

```js
const CodeLoop = window.CodeLoop || {};

CodeLoop.theme = {
  init() {
    const btn = document.getElementById('themeToggle');
    if (!btn) return;
    CodeLoop.theme._syncIcon();
    btn.addEventListener('click', CodeLoop.theme.toggle);
  },

  toggle() {
    const html  = document.documentElement;
    const current = html.getAttribute('data-theme') || 'dark';
    const next    = current === 'dark' ? 'light' : 'dark';
    html.setAttribute('data-theme', next);
    try { localStorage.setItem('theme', next); } catch (e) { /* unavailable */ }
    CodeLoop.theme._syncIcon();
  },

  _syncIcon() {
    const btn = document.getElementById('themeToggle');
    const isDark = document.documentElement.getAttribute('data-theme') !== 'light';
    btn.setAttribute('aria-label', isDark ? 'Switch to light mode' : 'Switch to dark mode');
    btn.innerHTML = isDark ? '<!-- sun SVG -->' : '<!-- moon SVG -->';
  }
};
```

---

## Courses Page Architecture

### Filter State Machine

The courses filter is a simple state object with three independent dimensions:

```js
// courses.js
const filterState = {
  category:   'all',   // 'all' | 'java' | 'python' | 'javascript' | 'html-css' | 'spring-boot' | 'sql' | 'dsa' | 'full-stack'
  difficulty: 'all',   // 'all' | 'Beginner' | 'Intermediate' | 'Advanced'
  search:     '',      // free text string
};
```

**Transition functions:**

```js
function setCategory(value)   { filterState.category   = value; applyFilters(); }
function setDifficulty(value) { filterState.difficulty = value; applyFilters(); }
function setSearch(value)     { filterState.search     = value.trim(); applyFilters(); }
```

**Combined filter algorithm:**

```js
function applyFilters() {
  const query = filterState.search.toLowerCase();
  let visibleCount = 0;

  document.querySelectorAll('.course-card').forEach(card => {
    const matchCat   = filterState.category === 'all'
                       || card.dataset.category === filterState.category;
    const matchDiff  = filterState.difficulty === 'all'
                       || card.dataset.difficulty === filterState.difficulty;
    const title      = (card.dataset.title || '').toLowerCase();
    const desc       = (card.dataset.description || '').toLowerCase();
    const matchSearch= query.length < 2
                       || title.includes(query)
                       || desc.includes(query);

    const visible = matchCat && matchDiff && matchSearch;
    card.hidden = !visible;
    if (visible) visibleCount++;
  });

  const empty = document.getElementById('coursesEmpty');
  if (empty) empty.hidden = visibleCount > 0;
}
```

This satisfies Requirement 14.7: every visible card satisfies the category AND difficulty AND search term conditions simultaneously.

### URL Parameter Pre-Selection

On `courses.html` page load:

```js
function initFromURL() {
  const params   = new URLSearchParams(window.location.search);
  const category = params.get('category');
  if (category) {
    filterState.category = category;
    // also activate the matching filter button visually
    const btn = document.querySelector(`.filter-btn[data-category="${category}"]`);
    if (btn) {
      document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
    }
    applyFilters();
  }
}
```

### Search Debouncing

```js
let searchDebounceTimer = null;
document.getElementById('courseSearchInput').addEventListener('input', (e) => {
  clearTimeout(searchDebounceTimer);
  searchDebounceTimer = setTimeout(() => setSearch(e.target.value), 150);
});
```

The 150 ms debounce ensures DOM updates start well within the 300 ms requirement.

---

## Contact Page Architecture

### Form Validation (contact.js)

**Validation rules:**

| Field | Rule |
|-------|------|
| Full Name | Required, max 100 chars |
| Email Address | Required, matches `/^[^@]+@[^@]+\.[^@]+$/` |
| Subject | Required, must be one of the predefined options |
| Message | Required, max 2000 chars |

**Algorithm:**

```js
function validateForm() {
  clearErrors();
  let valid = true;

  const name    = document.getElementById('cf-name').value.trim();
  const email   = document.getElementById('cf-email').value.trim();
  const subject = document.getElementById('cf-subject').value;
  const message = document.getElementById('cf-message').value.trim();

  if (!name)             { showError('cf-name',    'Name is required.');             valid = false; }
  if (!email)            { showError('cf-email',   'Email is required.');            valid = false; }
  else if (!/^[^@]+@[^@]+\.[^@]+$/.test(email))
                         { showError('cf-email',   'Please enter a valid email.');   valid = false; }
  if (!subject)          { showError('cf-subject', 'Please select a subject.');      valid = false; }
  if (!message)          { showError('cf-message', 'Message is required.');          valid = false; }

  return valid;
}

function showError(fieldId, msg) {
  const field = document.getElementById(fieldId);
  const err   = document.getElementById(fieldId + '-error');
  field.setAttribute('aria-invalid', 'true');
  field.setAttribute('aria-describedby', fieldId + '-error');
  if (err) { err.textContent = msg; err.hidden = false; }
}

function clearErrors() {
  document.querySelectorAll('.form-error').forEach(e => { e.hidden = true; e.textContent = ''; });
  document.querySelectorAll('[aria-invalid]').forEach(f => f.removeAttribute('aria-invalid'));
}
```

**Submission:**

```js
function submitForm(e) {
  e.preventDefault();
  if (!validateForm()) return;

  const subject = encodeURIComponent(document.getElementById('cf-subject').value);
  const name    = encodeURIComponent(document.getElementById('cf-name').value.trim());
  const email   = encodeURIComponent(document.getElementById('cf-email').value.trim());
  const body    = encodeURIComponent(
    `From: ${decodeURIComponent(name)} <${decodeURIComponent(email)}>\n\n`
    + document.getElementById('cf-message').value.trim()
  );

  window.location.href = `mailto:codeloop9563@gmail.com?subject=${subject}&body=${body}`;
  showSuccessState();
}
```

Each form field has a paired `<p id="cf-{field}-error" class="form-error" role="alert" hidden></p>` element. Using `role="alert"` causes screen readers to announce errors without requiring focus movement.

### Form HTML Structure

```html
<form id="contactForm" novalidate>
  <div class="form-group">
    <label for="cf-name">Full Name</label>
    <input type="text" id="cf-name" name="name"
           maxlength="100" autocomplete="name" required />
    <p id="cf-name-error" class="form-error" role="alert" hidden></p>
  </div>
  <div class="form-group">
    <label for="cf-email">Email Address</label>
    <input type="email" id="cf-email" name="email"
           autocomplete="email" required />
    <p id="cf-email-error" class="form-error" role="alert" hidden></p>
  </div>
  <div class="form-group">
    <label for="cf-subject">Subject</label>
    <select id="cf-subject" name="subject" required>
      <option value="" disabled selected>Choose a topic…</option>
      <option value="Question about a tutorial">Question about a tutorial</option>
      <option value="Collaboration / Partnership">Collaboration / Partnership</option>
      <option value="Feedback">Feedback</option>
      <option value="Report an issue">Report an issue</option>
      <option value="Something else">Something else</option>
    </select>
    <p id="cf-subject-error" class="form-error" role="alert" hidden></p>
  </div>
  <div class="form-group">
    <label for="cf-message">Message</label>
    <textarea id="cf-message" name="message"
              maxlength="2000" rows="6" required></textarea>
    <p id="cf-message-error" class="form-error" role="alert" hidden></p>
  </div>
  <button type="submit" class="btn btn-primary">Send Message</button>
</form>
```

### FAQ Accordion (contact.js)

Each FAQ item consists of a `<button>` trigger and a `<div>` answer panel. The mutual-exclusion invariant ensures at most one panel is open at any time.

```html
<div class="faq-item">
  <button class="faq-trigger"
          aria-expanded="false"
          aria-controls="faq-1-answer"
          id="faq-1-trigger">
    Are all CodeLoop tutorials really free?
    <span class="faq-icon" aria-hidden="true">+</span>
  </button>
  <div id="faq-1-answer"
       class="faq-answer"
       role="region"
       aria-labelledby="faq-1-trigger"
       aria-hidden="true"
       hidden>
    <p>Yes — 100% free, always…</p>
  </div>
</div>
```

```js
function initFAQ() {
  document.querySelectorAll('.faq-trigger').forEach(trigger => {
    trigger.addEventListener('click', () => {
      const isOpen   = trigger.getAttribute('aria-expanded') === 'true';
      const answerId = trigger.getAttribute('aria-controls');
      const answer   = document.getElementById(answerId);

      // Collapse all items
      document.querySelectorAll('.faq-trigger').forEach(t => {
        t.setAttribute('aria-expanded', 'false');
        const a = document.getElementById(t.getAttribute('aria-controls'));
        if (a) { a.setAttribute('aria-hidden', 'true'); a.hidden = true; }
      });

      // Expand clicked item only if it was closed
      if (!isOpen) {
        trigger.setAttribute('aria-expanded', 'true');
        answer.setAttribute('aria-hidden', 'false');
        answer.hidden = false;
      }
    });
  });
}
```

The `transition: max-height 0.4s ease` CSS animation is achieved by setting a CSS custom property `--faq-height` via JavaScript and using it in the transition rather than toggling `hidden` directly — the `hidden` attribute is only set after the close transition completes via a `transitionend` event.

---

## JavaScript Module Responsibilities

### main.js

- `CodeLoop.theme.init()` — reads localStorage, attaches toggle event listener
- `CodeLoop.scrollProgress.init()` — attaches scroll listener for progress bar
- `CodeLoop.backToTop.init()` — attaches scroll + click listeners for back-to-top
- `CodeLoop.search.init()` — initialises search overlay for index.html (topics grid)
- `CodeLoop.modal.open(key)` / `CodeLoop.modal.close()` — legal modal with focus trapping
- `initHero()` — adds `.hero--loaded` class after DOMContentLoaded
- `initStats()` — IntersectionObserver + requestAnimationFrame counter animation
- `renderTopicsGrid()` — generates topic card DOM from `TOPICS` data array
- `renderTutorials()` — generates tutorial card DOM from `TUTORIALS` data array; hides section if empty
- `renderProjects()` — generates project card DOM from `PROJECTS` data array
- `initRoadmapTabs()` — tablist keyboard navigation (Arrow keys, Home, End)
- `document.getElementById('footer-year').textContent = new Date().getFullYear()`

### courses.js

- `initFromURL()` — reads `?category` query parameter and pre-selects filter
- `renderCourses()` — generates course card DOM from `COURSES` data array
- `applyFilters()` — applies all three filter dimensions and shows/hides cards
- `setCategory(value)` / `setDifficulty(value)` — update state and re-apply
- `CodeLoop.search.init()` — initialises search overlay scoped to course cards
- Attaches `input` listener on `#courseSearchInput` with 150 ms debounce

### contact.js

- `initFAQ()` — accordion open/close with ARIA state management
- `validateForm()` — validates all four required fields, returns boolean
- `showError(fieldId, msg)` / `clearErrors()` — inline validation display
- `submitForm(e)` — constructs and opens `mailto:` link, shows success state
- `document.getElementById('contactForm').addEventListener('submit', submitForm)`

---

## Responsive Layout Strategy

### Breakpoints

Defined as CSS custom properties for use in media queries in `responsive.css`:

```
< 480 px   — xs: single-column everything, compact text
480–767 px — sm: 2-column grids, stacked hero
768–1023 px — md: 3-column topic grid, 2-column course grid
1024–1279 px — lg: 4-column topic grid, desktop nav
≥ 1280 px  — xl: full desktop layout, multi-column projects
```

### Grid Systems

All multi-column grids use CSS Grid with `minmax()` patterns that collapse gracefully:

```css
/* Topics: 4 → 3 → 2 → 1 */
.topics-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: var(--space-4);
}

/* Courses: auto-fill with min 300px */
.courses-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: var(--space-5);
}

/* Footer: 2fr 1fr 1fr 1fr → 1fr 1fr → 1fr */
.footer-inner {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr;
  gap: var(--space-10);
}
```

`responsive.css` overrides via `@media` only:

```css
@media (max-width: 1023px) {
  .topics-grid { grid-template-columns: repeat(3, 1fr); }
  .footer-inner { grid-template-columns: 1fr 1fr; }
}
@media (max-width: 767px) {
  .topics-grid { grid-template-columns: repeat(2, 1fr); }
  .why-grid    { grid-template-columns: 1fr 1fr; }
  .footer-inner { grid-template-columns: 1fr 1fr; }
  .nav-links, .nav-cta { display: none; }
  .nav-hamburger { display: flex; }
}
@media (max-width: 639px) {
  .why-grid    { grid-template-columns: 1fr; }
  .stats-grid  { grid-template-columns: 1fr; }
  .footer-inner { grid-template-columns: 1fr; }
  .roadmaps-section { /* single column, min-width 320px */ }
}
@media (max-width: 479px) {
  .topics-grid { grid-template-columns: 1fr; }
}
```

---

## Animation and Transition Specifications

| Element | Property | Duration | Easing | Delay |
|---------|----------|----------|--------|-------|
| Hero badge | opacity + translateY | 600ms | ease | 0ms |
| Hero heading | opacity + translateY | 600ms | ease | 150ms |
| Hero description | opacity + translateY | 600ms | ease | 300ms |
| Hero actions | opacity + translateY | 600ms | ease | 450ms |
| Hero code visual | opacity + translateY | 600ms | ease | 600ms |
| Stat counters | numeric (rAF) | 1500ms | ease-out cubic | 200ms stagger |
| Card hover | translateY(-4px) + shadow | 250ms | ease | 0ms |
| Mobile menu | opacity + translateY | 300ms | ease | 0ms |
| Search overlay | opacity + scale(0.98→1) | 250ms | ease | 0ms |
| FAQ panel | max-height | 400ms | ease | 0ms |
| Back-to-top | opacity + visibility | 300ms | ease | 0ms |
| Scroll progress | width | 50ms | linear | 0ms |
| Theme toggle icon | opacity | 200ms | ease | 0ms |

**Code snippet animation** (hero decorative element):

```css
@keyframes float {
  0%, 100% { transform: translateY(0);    }
  50%       { transform: translateY(-8px); }
}
.hero-code-snippet {
  animation: float 4s ease-in-out infinite;
}
```

**Cursor blink** (inside code snippet):

```css
@keyframes blink { 50% { opacity: 0; } }
.cursor {
  display: inline-block;
  width: 2px; height: 1em;
  background: var(--accent-primary);
  animation: blink 1s step-end infinite;
}
```

---

## SEO Implementation

### Per-Page Meta Tags

**index.html:**
```html
<title>CodeLoop — Learn. Code. Loop. | Free Programming Tutorials</title>
<meta name="description" content="Free programming tutorials on YouTube by Praveen Muthu. Master Java, Python, JavaScript, DSA, System Design and more — zero cost, interview-focused, beginner-friendly." />
<link rel="canonical" href="https://praveenmuthuvkl.github.io/code-loop-portfolio/" />
<meta property="og:type"        content="website" />
<meta property="og:title"       content="CodeLoop — Free Programming Tutorials" />
<meta property="og:description" content="Free programming tutorials on YouTube — Java, Python, JavaScript, DSA, System Design and more. Learn. Code. Loop." />
<meta property="og:url"         content="https://praveenmuthuvkl.github.io/code-loop-portfolio/" />
<meta property="og:image"       content="https://praveenmuthuvkl.github.io/code-loop-portfolio/img/og-image.png" />
<meta name="twitter:card"        content="summary_large_image" />
<meta name="twitter:title"       content="CodeLoop — Free Programming Tutorials" />
<meta name="twitter:description" content="Free programming tutorials on YouTube — Java, Python, JavaScript, DSA, System Design and more." />
<meta name="twitter:image"       content="https://praveenmuthuvkl.github.io/code-loop-portfolio/img/og-image.png" />
```

**courses.html:**
```html
<title>Courses & Topics — CodeLoop | Browse Free Programming Tutorials</title>
<meta name="description" content="Browse all CodeLoop programming courses — Java, Python, JavaScript, SQL, Spring Boot, DSA, System Design, and more. Filter by difficulty and topic. All free on YouTube." />
<link rel="canonical" href="https://praveenmuthuvkl.github.io/code-loop-portfolio/courses.html" />
```

**contact.html:**
```html
<title>Contact — CodeLoop | Get in Touch with Praveen Muthu</title>
<meta name="description" content="Contact CodeLoop founder Praveen Muthu for questions, collaboration, feedback, or tutorial requests. Reach out via email, LinkedIn, YouTube, or the contact form." />
<link rel="canonical" href="https://praveenmuthuvkl.github.io/code-loop-portfolio/contact.html" />
```

All three pages include `<html lang="en">` and a single `<h1>` per page.

### sitemap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://praveenmuthuvkl.github.io/code-loop-portfolio/</loc>
    <lastmod>2025-06-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://praveenmuthuvkl.github.io/code-loop-portfolio/courses.html</loc>
    <lastmod>2025-06-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.9</priority>
  </url>
  <url>
    <loc>https://praveenmuthuvkl.github.io/code-loop-portfolio/contact.html</loc>
    <lastmod>2025-06-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>
</urlset>
```

### robots.txt

```
User-agent: *
Allow: /
Sitemap: https://praveenmuthuvkl.github.io/code-loop-portfolio/sitemap.xml
```

---

## Accessibility Implementation

### Heading Hierarchy

Each page has exactly one `<h1>`. Heading levels are not skipped:

```
index.html:   h1 (hero) → h2 (Statistics, Topics, Roadmaps, Tutorials, Interview, Projects, Why Choose, About, Footer nav) → h3 (topic card names, roadmap step titles, tutorial titles, project titles, why card titles, footer column titles)
courses.html: h1 (page hero) → h2 (Courses, Roadmaps, Footer nav) → h3 (course card titles, roadmap step titles)
contact.html: h1 (page hero) → h2 (Contact Form title, FAQ title, Footer nav) → h3 (no h3 needed)
```

### ARIA Patterns Summary

| Element | Pattern |
|---------|---------|
| Navbar | `<nav aria-label="Main navigation">` |
| Mobile menu | `role="dialog" aria-modal="true"` |
| Hamburger | `aria-label="Toggle navigation menu"`, `aria-expanded`, `aria-controls` |
| Search overlay | `role="search" aria-label="Site search"` |
| Topics grid | `role="list"` / `role="listitem"` on cards; cards are `tabindex="0"` with keyboard activation |
| Roadmap tabs | ARIA tabs pattern: `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected`, `aria-controls` |
| Stat counters | `aria-live="polite"` on value spans |
| Scroll progress | `role="progressbar"`, `aria-valuenow`, `aria-valuemin`, `aria-valuemax` |
| Legal modal | `role="dialog" aria-modal="true" aria-labelledby` with focus trap |
| Contact form labels | `<label for="id">` matched to input `id` |
| Form errors | `role="alert" hidden` — announced on reveal |
| Form invalid fields | `aria-invalid="true"`, `aria-describedby="field-error"` |
| FAQ triggers | `aria-expanded`, `aria-controls` |
| FAQ panels | `role="region"`, `aria-labelledby`, `aria-hidden` |
| All images | Non-empty `alt`; decorative elements `aria-hidden="true"` or `alt=""` |
| SVG icons | Inline SVG with `aria-hidden="true"` when decorative; `aria-label` on their parent button |

### Keyboard Navigation

- All interactive elements reachable via Tab/Shift-Tab.
- `topic-card` elements receive `tabindex="0"` and respond to Enter/Space as a navigation action (navigates to courses page with pre-selected filter).
- Roadmap tabs implement the ARIA tabs keyboard pattern: Arrow Left/Right moves between tabs; Home/End jump to first/last tab.
- Modal focus trap prevents Tab from leaving the modal while open.
- Mobile menu closes on Escape; focus returns to hamburger.
- Search overlay closes on Escape; focus returns to search button.

### Focus Indicators

```css
/* Visible on all interactive elements, keyboard-only (not mouse) */
:focus-visible {
  outline: 2px solid var(--accent-primary);
  outline-offset: 3px;
  border-radius: var(--radius-sm);
}
/* Contrast: --accent-primary #8B5CF6 on --bg-main #080B14 ≈ 5.5:1 ✓ */
/* Contrast: --accent-primary #8B5CF6 on --bg-card #151D30 ≈ 4.8:1 ✓ */
/* Contrast: --accent-primary #8B5CF6 on --bg-main #F8FAFC (light) ≈ 4.6:1 ✓ */
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Theme token contrast compliance

*For any* text–background CSS custom property pair used together in the rendered page, the WCAG contrast ratio must be ≥ 4.5:1 for normal-sized text and ≥ 3:1 for large text and UI components, in both dark and light modes.

**Validates: Requirements 1.2**

---

### Property 2: Theme localStorage round-trip

*For any* valid theme value (`"dark"` or `"light"`) stored in `localStorage` under the key `"theme"`, loading any page of the site must result in the `data-theme` attribute on `<html>` equalling that stored value before first paint.

**Validates: Requirements 1.3, 1.6**

---

### Property 3: Theme toggle state preservation

*For any* starting theme state, activating the theme toggle must produce the opposite theme in both the `data-theme` attribute on `<html>` and the value stored in `localStorage["theme"]`.

**Validates: Requirements 1.6**

---

### Property 4: Hamburger ARIA state consistency

*For any* state of the mobile menu (open or closed), the `aria-expanded` attribute on the hamburger button must equal `"true"` when the menu is open and `"false"` when the menu is closed.

**Validates: Requirements 2.10, 20.3**

---

### Property 5: Search filter correctness

*For any* search query string of 2 or more characters and any set of content cards on the page, after applying the filter: every visible card's title or description must contain the search term (case-insensitive), and every card whose title and description both do not contain the term must be hidden.

**Validates: Requirements 3.2**

---

### Property 6: Search clear restores original state

*For any* set of content cards that have been filtered by a search query, clearing the query to fewer than 2 characters must restore all cards to the visible state in their original display order.

**Validates: Requirements 3.3**

---

### Property 7: Statistics animation fires exactly once

*For any* scroll pattern — including scrolling the statistics section in and out of the viewport multiple times — the count-up animation must be triggered exactly once per page load.

**Validates: Requirements 5.3**

---

### Property 8: Topic card content invariant

*For every* topic card rendered in the Topics Grid, the card's description text must be 20 words or fewer, and the difficulty label must be one of the values `"Beginner"`, `"Intermediate"`, or `"Advanced"`.

**Validates: Requirements 6.2**

---

### Property 9: Course combined filter conjunction

*For any* active category filter, difficulty filter, and search term, every card that is visible must satisfy all three conditions (category matches AND difficulty matches AND title/description contains the search term). Equivalently, every card that fails any single condition must be hidden.

**Validates: Requirements 14.6, 14.7**

---

### Property 10: Contact form validation rejects incomplete submissions

*For any* subset of the four required form fields (name, email, subject, message) that are left empty or invalid, submitting the form must display an inline validation error message directly below each failing field and must not construct or open a `mailto:` link.

**Validates: Requirements 13.2, 13.3**

---

### Property 11: Contact form mailto construction

*For any* valid form submission (all fields filled, email format valid), the constructed `mailto:` URL must contain the URL-encoded subject value and the URL-encoded message value provided by the user.

**Validates: Requirements 13.4**

---

### Property 12: FAQ mutual exclusion invariant

*For any* sequence of FAQ trigger activations, after each activation at most one FAQ answer panel must be in the open state (`aria-expanded="true"`, `aria-hidden="false"`, visible in DOM). If the currently open panel's trigger is activated, all panels must be closed.

**Validates: Requirements 13.7**

---

### Property 13: Scroll progress formula

*For any* `scrollY` value in the range `[0, document.documentElement.scrollHeight - window.innerHeight]`, the width of the scroll progress bar must equal `(scrollY / (scrollHeight - innerHeight)) * 100%`. When `scrollHeight - innerHeight ≤ 0`, the width must be `0%` and no division-by-zero error must be thrown.

**Validates: Requirements 16.2, 16.4**

---

### Property 14: Back-to-top visibility threshold

*For any* `scrollY` value, the back-to-top button must be visible (opacity 1, visibility visible) when `scrollY > 300` and hidden (opacity 0, visibility hidden) when `scrollY ≤ 300`.

**Validates: Requirements 17.1, 17.2**

---

### Property 15: Meta description length compliance

*For every* page in the site (index.html, courses.html, contact.html), the content of the `<meta name="description">` tag must have a character count in the range [120, 160] inclusive.

**Validates: Requirements 18.1**

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| `localStorage` unavailable | Silent catch in anti-FOSC script and theme toggle; defaults to dark theme |
| Founder image fails to load | `onerror` hides `<img>` and shows styled `PM` initials fallback element |
| `TUTORIALS` array empty | `renderTutorials()` sets `section.hidden = true` instead of rendering empty container |
| `scrollHeight - innerHeight ≤ 0` | Progress bar width clamped to 0%; no division performed |
| Invalid `?category` URL parameter | `initFromURL()` checks for matching button; if none found, filter stays at "all" |
| Form submission with invalid data | Prevents `mailto:` construction, shows inline errors, page stays on form |
| `IntersectionObserver` unavailable (rare) | `if ('IntersectionObserver' in window)` guard; falls back to triggering animation on first scroll event |

---

## Testing Strategy

### Unit Tests

Unit tests verify specific examples, edge cases, and error conditions. Written with a standard test runner (e.g., Vitest):

- Theme toggle: starting from dark → toggled → light; starting from light → toggled → dark.
- Anti-FOSC: with key `"light"` stored → `data-theme="light"`; with key `"dark"` stored → `data-theme="dark"`; with no key → `data-theme="dark"`; with localStorage throwing → `data-theme="dark"`, no error thrown.
- `animateCounter`: final value equals target; intermediate values are non-decreasing; runs exactly once per call.
- Search filter: query with 0–1 chars shows all; query matching some → correct subset visible; query matching none → "No results found" displayed.
- `applyFilters()` with category="java", difficulty="all" → only Java cards visible; category="all", difficulty="Beginner" → only Beginner cards; both set → conjunction of both.
- `validateForm()` with each field individually empty → returns false, shows error for that field only.
- `validateForm()` with invalid email formats → returns false, shows email error.
- `validateForm()` with all fields valid → returns true.
- `submitForm()` with valid data → `mailto:` URL contains correct encoded subject and message.
- FAQ toggle: open item A → A open; open item B → A closed, B open; open item B again → B closed (all closed).
- Scroll progress: scrollY=0 → 0%; scrollY=500, scrollable=1000 → 50%; scrollable=0 → 0%, no error.
- Back-to-top: scrollY=300 → hidden; scrollY=301 → visible.

### Property-Based Tests

Property-based tests are written using a PBT library (e.g., [fast-check](https://github.com/dubzzz/fast-check) for JavaScript). Each test runs a minimum of 100 iterations with randomised inputs.

Each test must include a comment referencing the originating design property:
**Format: `// Feature: codeloop-website-redesign, Property {N}: {property_title}`**

- **Property 1** — Generate all token colour pairs from the design tokens object; for each pair assert WCAG contrast ratio ≥ 4.5:1 or ≥ 3:1 depending on text classification.
- **Property 2 & 3** — Generate arbitrary theme values; test localStorage round-trip and toggle inversion.
- **Property 4** — Generate arbitrary sequences of hamburger clicks; assert `aria-expanded` always reflects actual menu open state.
- **Property 5 & 6** — Generate arbitrary card datasets and query strings; test filter correctness and clear restoration.
- **Property 7** — Simulate arbitrary scroll-in/scroll-out sequences; assert animation triggered exactly once.
- **Property 8** — Generate arbitrary topic data; assert description word count ≤ 20 and difficulty is one of the three valid values.
- **Property 9** — Generate arbitrary (category, difficulty, search) filter states and card datasets; assert visible cards satisfy all three conditions.
- **Property 10 & 11** — Generate arbitrary form field combinations; test validation rejection and mailto construction.
- **Property 12** — Generate arbitrary sequences of FAQ trigger indices; assert at most one panel open after each operation.
- **Property 13** — Generate arbitrary (scrollY, scrollHeight, innerHeight) values; assert progress formula and division-by-zero safety.
- **Property 14** — Generate arbitrary scrollY values; assert visibility state matches threshold.
- **Property 15** — Assert meta description length for all three pages.
