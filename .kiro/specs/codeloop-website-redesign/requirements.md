# Requirements Document

## Introduction

CodeLoop is a free programming education platform founded by Praveen Muthu, delivering tutorials on YouTube at https://www.youtube.com/@CodeLoop-SP/videos. The current website (index.html, courses.html, contact.html) is functional but lacks the visual polish, interactivity, and feature depth of a premium developer education brand. This project completely redesigns and upgrades the CodeLoop website into a professionally designed technology startup site inspired by Vercel, Linear, and freeCodeCamp, while remaining a plain HTML/CSS/JavaScript site compatible with GitHub Pages at base path /code-loop-portfolio/. All existing brand identity, founder information, external links, and educational content must be preserved.

## Glossary

- **Website**: The CodeLoop static website hosted on GitHub Pages, consisting of index.html, courses.html, and contact.html.
- **Design_System**: The set of colours, typography, spacing, and component patterns applied consistently across all pages.
- **Theme**: The active colour mode, either Dark or Light, stored in localStorage and respected on every page load.
- **Navbar**: The sticky top navigation bar present on all pages.
- **Hero**: The primary above-the-fold section on the homepage.
- **Statistics_Section**: The animated counter block displaying aggregate numbers about the channel and platform.
- **Topics_Grid**: The section displaying the 12 programming topic cards on the homepage.
- **Roadmap**: A visual learning path showing ordered steps for a developer specialisation.
- **Course_Card**: A single card on the Courses page representing one topic with metadata, description, and a link.
- **Search**: The global in-page search that filters visible content by keyword.
- **Contact_Form**: The HTML form on the contact page that validates inputs and composes a mailto link.
- **FAQ**: The accordion-style frequently-asked-questions section on the Contact page.
- **SEO_Meta**: The set of HTML head elements covering title, description, canonical URL, Open Graph, and Twitter Card tags.
- **Sitemap**: The sitemap.xml file listing all public URLs for search engine crawlers.
- **Robots**: The robots.txt file declaring crawler access rules.
- **Scroll_Progress**: A thin bar fixed at the top of the viewport indicating what fraction of the page has been scrolled.
- **Back_To_Top**: A button that appears after scrolling down and smoothly returns the user to the top of the page.
- **Founder**: Praveen Muthu, the creator and educator behind CodeLoop.
- **Interview_Hub**: The external companion platform at https://interviewhub360.netlify.app/ for interview preparation.
- **YouTube_Channel**: The CodeLoop YouTube channel at https://www.youtube.com/@CodeLoop-SP/videos.
- **Instagram**: The CodeLoop Instagram account at https://www.instagram.com/codeloop_yt/.
- **LinkedIn**: Praveen Muthu's LinkedIn profile at https://www.linkedin.com/in/praveenmuthu-v/.

---

## Requirements

### Requirement 1: Design System & Theme

**User Story:** As a visitor, I want a visually polished, consistent design with dark and light mode support, so that the site looks professional and I can read it comfortably in any lighting condition.

#### Acceptance Criteria

1. THE Website SHALL define the following dark-mode CSS custom properties on `:root` or `[data-theme="dark"]`: `--bg-main: #080B14`, `--bg-secondary: #101626`, `--bg-card: #151D30`, `--accent-primary: #8B5CF6`, `--accent-secondary: #38BDF8`, `--text-main: #F8FAFC`, `--text-secondary: #94A3B8`.
2. WHEN the light mode Theme is active, THE Website SHALL apply a `[data-theme="light"]` override on the `<html>` element that redefines the background and text tokens so that all text–background colour pairs achieve a minimum contrast ratio of 4.5:1 (normal text) or 3:1 (large text / UI components) per WCAG 2.1 SC 1.4.3 and 1.4.11.
3. WHEN a visitor loads any page, THE Website SHALL read the `theme` key from localStorage and set the `data-theme` attribute on `<html>` before first paint to prevent flash of unstyled content.
4. WHEN no `theme` key exists in localStorage, THE Website SHALL default to `data-theme="dark"`.
5. IF localStorage is unavailable (e.g., private browsing or security restriction), THE Website SHALL fall back to the dark Theme without throwing a JavaScript error.
6. WHEN the theme toggle control is activated, THE Website SHALL toggle the `data-theme` attribute on `<html>` between `"dark"` and `"light"` and persist the new value to localStorage.
7. THE Website SHALL use Inter or Satoshi as the primary font family, with a system sans-serif stack as the fallback.
8. THE Design_System SHALL define identical values for border-radius, box-shadow, and spacing scale in a single CSS file (`css/style.css`); no page-specific stylesheet SHALL redefine these tokens with a conflicting value.

---

### Requirement 2: Navbar

**User Story:** As a visitor, I want a sticky, responsive navigation bar on every page, so that I can always find my way around the site without scrolling back to the top.

#### Acceptance Criteria

1. THE Navbar SHALL be fixed to the top of the viewport on all three pages and remain visible during scroll.
2. THE Navbar SHALL display the CodeLoop logo/wordmark on the left, navigation links in the centre, and a search icon, theme toggle, and "Subscribe" CTA button on the right.
3. WHEN the viewport width is 768 px or narrower, THE Navbar SHALL hide the centre navigation links and CTA button and display a hamburger icon instead.
4. WHEN the hamburger icon is activated (click, Enter, or Space), THE Navbar SHALL open a mobile menu that covers 100% of the viewport height and width, contains all navigation links and the CTA button, and moves keyboard focus to the first focusable element inside the menu.
5. WHEN the mobile menu is open and the hamburger icon is activated again, THE Navbar SHALL close the mobile menu and return keyboard focus to the hamburger icon.
6. WHEN the mobile menu is open and a pointer or touch event is registered outside the menu panel, THE Navbar SHALL close the mobile menu and return keyboard focus to the hamburger icon.
7. WHEN a navigation link in the mobile menu is selected, THE Navbar SHALL close the mobile menu and navigate to the target page.
8. WHEN the Navbar subscribe button is activated, THE Website SHALL open the YouTube_Channel URL in a new tab.
9. THE Navbar SHALL provide keyboard focus indicators on all interactive elements such that the focus indicator has a minimum contrast ratio of 3:1 against the adjacent background colour, satisfying WCAG 2.1 SC 2.4.7.
10. THE Navbar hamburger icon SHALL have an `aria-label` of "Toggle navigation menu" and an `aria-expanded` attribute that reflects `true` when the menu is open and `false` when closed.

---

### Requirement 3: Global Search

**User Story:** As a visitor, I want to type a keyword and see matching content highlighted or filtered on the current page, so that I can quickly find the topic or tutorial I am looking for.

#### Acceptance Criteria

1. WHEN the search icon in the Navbar is clicked or tapped, THE Website SHALL expand a search input field overlaying the current page and move keyboard focus to the input.
2. WHEN the search input contains 2 or more characters, THE Website SHALL filter visible topic cards, course cards, or roadmap steps on the current page on each keystroke to show only items whose title or description contains the search term (case-insensitive).
3. WHEN the search input is cleared to fewer than 2 characters, THE Website SHALL restore all previously hidden items in their original display order.
4. WHEN the search overlay close control is activated, THE Website SHALL hide the search input field and restore all previously hidden items in their original display order.
5. WHEN no items match the current search term of 2 or more characters, THE Website SHALL display a "No results found" message in the area where the cards are shown.
6. WHEN the Escape key is pressed while the search overlay is open, THE Website SHALL close the overlay, hide the search input field, and restore all items in their original display order.

---

### Requirement 4: Hero Section

**User Story:** As a first-time visitor, I want an impactful hero section that communicates what CodeLoop offers and gives me clear next steps, so that I immediately understand the platform's value.

#### Acceptance Criteria

1. THE Hero SHALL display a heading that includes the brand name "CodeLoop" and the tagline "Learn. Code. Loop."
2. THE Hero SHALL display a description of 30 words or fewer communicating that all programming tutorials are free on YouTube.
3. THE Hero SHALL display a primary CTA button linking to the YouTube_Channel and a secondary CTA button linking to the Courses page.
4. THE Hero SHALL display an animated developer-themed visual element (e.g., a code snippet animation or a floating icon composition) positioned so that it does not overlap the text content at any supported viewport width.
5. WHEN the page loads, THE Hero SHALL animate its heading, description, and buttons into view using CSS transitions, with each element delayed 150 ms after the previous one and each transition lasting 600 ms.
6. THE Hero heading and description text SHALL achieve a contrast ratio of at least 4.5:1 against their background at every supported viewport width from 320 px to 1440 px.

---

### Requirement 5: Statistics Section

**User Story:** As a visitor, I want to see key numbers about CodeLoop at a glance, so that I can quickly gauge the scale and credibility of the platform.

#### Acceptance Criteria

1. THE Statistics_Section SHALL display four metrics: YouTube Videos, Programming Topics, Free Learning Resources, and Learning Opportunities; target values SHALL be defined in a JavaScript configuration object in `js/main.js`.
2. WHEN the Statistics_Section enters the viewport, THE Website SHALL animate each counter from zero to its configured target value over 1500 ms, with a 200 ms stagger between each counter start, using `requestAnimationFrame`.
3. THE Statistics_Section SHALL trigger the count-up animation exactly once per page load, even if the section scrolls out of and back into the viewport.
4. WHEN the viewport width is below 640 px, THE Statistics_Section SHALL stack the four counters vertically in a single column.

---

### Requirement 6: Programming Topics Grid

**User Story:** As a learner, I want to browse all available programming topics in one view, so that I can pick where to start or what to explore next.

#### Acceptance Criteria

1. THE Topics_Grid SHALL display exactly 12 topic cards: Java, Python, JavaScript, HTML & CSS, DSA, Algorithms, SQL, Spring Boot, Full Stack, System Design, Interview Prep, and Real-World Projects.
2. EACH topic card SHALL display an icon, a topic title, a description of 20 words or fewer, and a difficulty label (Beginner, Intermediate, or Advanced).
3. WHEN a topic card is hovered with a pointer device, THE Topics_Grid SHALL apply a highlight effect that visibly changes at least one of: background colour, border colour, or box-shadow compared to the default state.
4. WHEN a topic card is activated (click, Enter, or Space), THE Website SHALL navigate to the Courses page with a URL query parameter or hash that pre-selects the matching category filter.
5. THE Topics_Grid SHALL use a responsive CSS grid with 4 columns at viewports ≥ 1024 px, 3 columns at 768 px – 1023 px, 2 columns at 480 px – 767 px, and 1 column below 480 px.

---

### Requirement 7: Learning Roadmaps

**User Story:** As a learner, I want to see structured learning paths for specific developer roles, so that I have a clear step-by-step plan to follow.

#### Acceptance Criteria

1. THE Website SHALL display at least three Roadmaps: Java Developer, Full Stack Developer, and Python Developer.
2. EACH Roadmap SHALL be displayed as a vertical timeline or step-sequence with numbered steps, a step title, and a description of 30 words or fewer per step.
3. WHEN a Roadmap step references a topic available on the Courses page, THE Website SHALL render that step as an anchor linking to the relevant course category.
4. IF a Roadmap step does not have a corresponding course, THE Website SHALL render that step as plain non-linked text with no `<a>` element or `href` attribute.
5. WHEN the viewport width is below 640 px, THE Roadmaps section SHALL reflow to a single-column layout with a minimum readable width of 320 px.

---

### Requirement 8: Latest YouTube Tutorials

**User Story:** As a visitor, I want to see featured YouTube tutorials on the homepage, so that I can immediately watch content without leaving the site.

#### Acceptance Criteria

1. THE Website SHALL display between 3 and 8 featured YouTube tutorial cards on the homepage, each showing a video thumbnail, title, topic tag, and duration or publish-date label; the card data SHALL be defined in a JavaScript array in `js/main.js`.
2. ALL tutorial thumbnail images SHALL use `loading="lazy"` to defer loading until they enter the viewport.
3. WHEN a tutorial card is activated (click, Enter, or Space), THE Website SHALL open the corresponding YouTube video URL in a new tab.
4. THE tutorials section SHALL display a "View All on YouTube" button that opens the YouTube_Channel URL in a new tab.
5. IF the tutorial configuration array in `js/main.js` is empty, THE Website SHALL hide the entire tutorials section rather than rendering an empty container.

---

### Requirement 9: Interview Preparation Section

**User Story:** As a learner preparing for job interviews, I want a prominent section pointing me to the Interview Hub platform, so that I can access structured interview practice alongside tutorials.

#### Acceptance Criteria

1. THE Website SHALL include a dedicated Interview Preparation section on the homepage with a description of at least 20 words explaining the Interview_Hub platform.
2. THE section SHALL display a CTA button with a label between 3 and 40 characters that opens `https://interviewhub360.netlify.app/` in a new tab.
3. THE section SHALL list at least four feature highlights of the Interview_Hub platform, each between 2 and 8 words.

---

### Requirement 10: Real-World Projects Showcase

**User Story:** As a learner, I want to see examples of real-world projects taught on the channel, so that I understand what I can build after completing the tutorials.

#### Acceptance Criteria

1. THE Website SHALL display a Projects section on the homepage with at least 3 project showcase cards.
2. EACH project card SHALL include a project title (max 60 characters), a description (max 150 characters), a technology list (max 5 items), and a link to a YouTube tutorial or playlist that opens in a new tab.
3. WHEN the viewport width is ≥ 1280 px, THE Projects section SHALL display cards in a multi-column grid; at 768 px – 1279 px in a 2-column grid; below 768 px in a single-column list.

---

### Requirement 11: Why Choose CodeLoop

**User Story:** As a prospective learner, I want to see clear reasons why CodeLoop is worth my time compared to other resources, so that I can decide to subscribe and follow the content.

#### Acceptance Criteria

1. THE Website SHALL display a "Why Choose CodeLoop" section containing exactly four feature cards.
2. EACH feature card SHALL include an icon with a non-empty `alt` attribute (or `aria-label` if SVG), a title of 2–6 words, and a description of 1–2 sentences (10–40 words).
3. THE four cards SHALL each address one of the following distinct topics, one card per topic: concept-first teaching, interview-focused content, zero cost to learners, and regular content updates.
4. WHEN the viewport width is ≥ 1024 px, THE four cards SHALL display in a 4-column grid; at 640 px – 1023 px in a 2-column grid; below 640 px in a single-column list.

---

### Requirement 12: About Founder Section

**User Story:** As a visitor, I want to learn about the person behind CodeLoop, so that I can feel a personal connection and trust the platform.

#### Acceptance Criteria

1. THE Website SHALL display an About Founder section that names Praveen Muthu and states his role as Founder & Developer of CodeLoop.
2. THE section SHALL include a photo or avatar image with a descriptive `alt` attribute, a bio paragraph of 30–200 words, and links to LinkedIn, YouTube_Channel, Instagram, and `mailto:codeloop9563@gmail.com`.
3. IF the founder image fails to load, THE Website SHALL display a styled avatar placeholder that maintains the section layout.
4. WHEN any Founder social or email link is activated, THE Website SHALL open the target URL in a new tab.

---

### Requirement 13: Contact Section & Form

**User Story:** As a visitor, I want to send a message to the CodeLoop team directly from the website, so that I can ask questions or propose collaborations without navigating away.

#### Acceptance Criteria

1. THE Contact_Form SHALL include four fields: Full Name (max 100 chars), Email Address, Subject (dropdown with at least 4 predefined options), and Message (max 2000 chars).
2. WHEN the Contact_Form is submitted and any required field is empty, THE Contact_Form SHALL display an inline validation error message directly below the empty field and prevent form submission.
3. WHEN the Contact_Form is submitted and the Email Address field value does not match the pattern `[^@]+@[^@]+\.[^@]+`, THE Contact_Form SHALL display an inline validation error on that field and prevent submission.
4. WHEN the Contact_Form passes all validation checks, THE Website SHALL construct a `mailto:` link with the form Subject and Message pre-filled, open it, and replace the form with a success confirmation message.
5. THE Contact page SHALL display direct contact information: the channel email address, LinkedIn URL, YouTube_Channel URL, and Instagram URL as labelled links.
6. THE Contact page SHALL include a FAQ accordion with at least the questions and answers preserved from the existing contact.html.
7. WHEN a FAQ accordion trigger button is activated (click, Enter, or Space), THE page SHALL expand that item's answer panel and collapse any other currently open panel; the transition SHALL complete within 500 ms.

---

### Requirement 14: Courses Page

**User Story:** As a learner, I want to browse, search, and filter all available courses, so that I can find the right content for my skill level and area of interest.

#### Acceptance Criteria

1. THE Courses page SHALL display Course_Cards for at least the following categories: Java, Python, JavaScript, Frontend (HTML & CSS), Backend (Spring Boot), SQL, DSA, and Full Stack.
2. EACH Course_Card SHALL display: topic icon, category label, course title, difficulty badge (Beginner / Intermediate / Advanced), a short description of at most 160 characters, key topics covered (2–6 items), and a "Watch on YouTube" link.
3. THE Courses page SHALL include a text search input that filters Course_Cards within 300 ms of the first keystroke (case-insensitive match on title and description).
4. THE Courses page SHALL include category filter buttons; activating a category button shows only Course_Cards with a matching category label; the "All" button restores all cards.
5. THE Courses page SHALL include a difficulty filter control (All / Beginner / Intermediate / Advanced) that narrows displayed cards to the selected difficulty level.
6. WHEN both a category filter and a difficulty filter are active simultaneously, THE Courses page SHALL display only Course_Cards that satisfy both the category AND difficulty conditions.
7. WHEN a search term is active alongside category and/or difficulty filters, THE Courses page SHALL display only Course_Cards that satisfy all three conditions simultaneously.
8. WHEN no Course_Cards match the active search term and/or filters, THE Courses page SHALL display a "No courses found" empty-state message in the area where the cards are shown.
9. THE Courses page SHALL include the Learning Roadmap section also present on the homepage.

---

### Requirement 15: Footer

**User Story:** As a visitor at the bottom of any page, I want a comprehensive footer with links, social icons, and legal information, so that I can navigate to any section or find the site's policies.

#### Acceptance Criteria

1. THE Footer SHALL appear on all three pages and contain: the CodeLoop logo/wordmark and tagline, social links (YouTube, LinkedIn, Instagram, Email), a navigation column, a resources column (linking to Interview_Hub and YouTube_Channel), and a legal column (Terms of Use, Privacy Policy, Disclaimer).
2. THE Footer SHALL display a copyright line in the format "© [current year] Praveen Muthu / CodeLoop. All rights reserved."
3. WHEN a social link in the Footer is activated, THE Website SHALL open the target URL in a new tab.
4. WHEN a legal link (Terms, Privacy, Disclaimer) is activated, THE Website SHALL display the corresponding policy text in a modal overlay; the modal SHALL provide a close button and close when the Escape key is pressed.
5. WHEN the viewport width is below 640 px, THE Footer SHALL reflow so that all columns stack vertically in a single column.

---

### Requirement 16: Scroll Progress Indicator

**User Story:** As a visitor reading a long page, I want to see how far through the page I have scrolled, so that I can gauge how much content remains.

#### Acceptance Criteria

1. THE Website SHALL display a Scroll_Progress bar fixed at the very top of the viewport with a z-index that places it above the Navbar, on all three pages.
2. WHEN the page is scrolled, THE Scroll_Progress bar's width SHALL be updated to `(scrollY / (scrollHeight - innerHeight)) * 100%`; the update SHALL fire on every `scroll` event.
3. THE Scroll_Progress bar SHALL use the `--accent-primary` CSS custom property for its background colour.
4. IF the page content height is less than or equal to the viewport height (denominator ≤ 0), THE Scroll_Progress bar SHALL remain at 0% width and no division-by-zero error SHALL be thrown.

---

### Requirement 17: Back-to-Top Button

**User Story:** As a visitor who has scrolled far down a long page, I want a quick way to return to the top, so that I can access the navbar and hero without excessive scrolling.

#### Acceptance Criteria

1. THE Back_To_Top button SHALL have `visibility: hidden` and `opacity: 0` when the page `scrollY` is 300 px or less.
2. WHEN the page `scrollY` exceeds 300 px, THE Back_To_Top button SHALL transition to `visibility: visible` and `opacity: 1` over a CSS transition that completes within 300 ms.
3. WHEN the Back_To_Top button is activated (click, Enter, or Space), THE Website SHALL scroll the viewport to the top using `scroll-behavior: smooth`.
4. THE Back_To_Top button SHALL be reachable via keyboard Tab navigation and operable with both Enter and Space keys.

---

### Requirement 18: SEO & Performance

**User Story:** As a content owner, I want search engines to correctly index and represent all pages, and for pages to load fast on mobile devices, so that organic traffic grows and learners have a good experience.

#### Acceptance Criteria

1. EACH page (index.html, courses.html, contact.html) SHALL include a unique `<title>` tag and a `<meta name="description">` tag with a page-specific description of 120–160 characters.
2. EACH page SHALL include a `<link rel="canonical">` tag pointing to its own absolute GitHub Pages URL.
3. EACH page SHALL include Open Graph meta tags: `og:title`, `og:description`, `og:type`, `og:url`, and `og:image`.
4. EACH page SHALL include Twitter Card meta tags: `twitter:card`, `twitter:title`, `twitter:description`, and `twitter:image`.
5. THE Website root SHALL include a `sitemap.xml` file listing all three page URLs with `<lastmod>` dates in ISO 8601 format (YYYY-MM-DD).
6. THE Website root SHALL include a `robots.txt` file containing `User-agent: *`, `Allow: /`, and a `Sitemap:` directive pointing to the absolute sitemap URL.
7. ALL images below the initial viewport (roughly 360×640 px on mobile) SHALL use `loading="lazy"`.
8. THE Website SHALL use semantic HTML5 elements (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`, `<aside>`) throughout all pages.
9. THE `<html>` element on every page SHALL include `lang="en"`.

---

### Requirement 19: File Structure & GitHub Pages Compatibility

**User Story:** As the site owner, I want the codebase organised in a predictable, maintainable file structure that deploys correctly to GitHub Pages, so that the site is easy to update and always accessible at the published URL.

#### Acceptance Criteria

1. THE Website SHALL organise files as follows: `index.html`, `courses.html`, `contact.html`, `sitemap.xml`, `robots.txt`, `css/style.css`, `css/responsive.css`, `js/main.js`, `js/courses.js`, `js/contact.js`.
2. ALL internal links and asset `src`/`href` attributes SHALL use relative paths so the site resolves correctly under the `/code-loop-portfolio/` base path on GitHub Pages.
3. THE Website SHALL use no server-side code, no build steps, no npm packages, and no external JavaScript frameworks — only plain HTML, CSS, and vanilla JavaScript.
4. EACH JavaScript file SHALL be responsible for a distinct concern: `main.js` for homepage logic and shared utilities, `courses.js` for courses page filtering and search, `contact.js` for form validation and FAQ accordion.
5. `css/style.css` SHALL contain base styles and component definitions; `css/responsive.css` SHALL contain only `@media` query blocks and SHALL not define styles outside a media query.

---

### Requirement 20: Accessibility

**User Story:** As a visitor using keyboard navigation or a screen reader, I want all interactive elements to be reachable and understandable without a mouse, so that the site is usable regardless of ability.

#### Acceptance Criteria

1. ALL interactive elements (buttons, links, inputs, selects) SHALL display a `:focus` outline of at least 2 px styled with `--accent-primary` or `--accent-secondary`, achieving a minimum 3:1 contrast ratio against the adjacent background.
2. ALL images SHALL have a non-empty `alt` attribute describing their content; purely decorative images SHALL use `alt=""`.
3. THE Navbar mobile hamburger button SHALL have an `aria-label` of "Toggle navigation menu" and its `aria-expanded` attribute SHALL be `"true"` when the menu is open and `"false"` when closed.
4. THE Contact_Form SHALL associate every input and select with its visible label using matching `for` and `id` attribute pairs.
5. EACH FAQ accordion trigger button SHALL carry `aria-expanded` reflecting its open/closed state, and its associated answer panel SHALL carry `aria-hidden` set to the opposite boolean value.
6. THE modal overlay for legal pages SHALL cycle keyboard focus between all focusable elements inside the modal while it is open and SHALL return focus to the element that triggered the modal when the modal is closed.
7. EACH page SHALL contain exactly one `<h1>` element and SHALL not skip heading levels (e.g., `<h2>` SHALL not be followed by `<h4>` without an intervening `<h3>`).
