# Next.js + Tailwind Accessibility Audit Guide

## Core Audit Areas

### 1. Page Structure & Metadata

**Check:** Each page has unique title in metadata/head
**Pattern:** Page-specific `export const metadata` or dynamic `generateMetadata()`
**Flag:** Same title across multiple pages, missing/generic titles
**Required:** Descriptive, unique page titles that reflect content

### 2. Heading Hierarchy

**Check:** Exactly one `<h1>` per page, no skipped levels
**Pattern:** `h1 -> h2 -> h3` (never skip from h1 to h3)
**Flag:** Multiple h1s, skipped heading levels, missing h1
**Location:** h1 should be first heading in main content area

### 3. Layout Consistency

**Check:** `layout.tsx` provides consistent `<main>` element on all pages
**Pattern:** `<main className="container mx-auto...">{children}</main>`
**Flag:** Pages without main landmark, inconsistent layout structure
**Required:** Main element wrapping page content, consistent across app

### 4. Color Contrast Requirements

**Check:** Minimum 4.5:1 ratio for normal text, 3:1 for large text/UI components
**API:** Use WebAIM API for precise calculations: `https://webaim.org/resources/contrastchecker/?fcolor=HEXCOLOR&bcolor=HEXCOLOR&api`
**Pattern:** Test all text/background combinations, including hover/focus states
**Flag:** Insufficient contrast ratios, relying solely on color for information
**Tailwind:** Ensure custom color combinations meet standards

### 5. Alternative Text (Alt Text) Audit

**Check:** All images have appropriate alt text, decorative images use `alt=""`
**Pattern:** Descriptive, concise, context-appropriate alt text
**Flag:** Missing alt attributes, generic text like "image" or "photo", identical alt text and captions
**Required:** Empty alt for decorative images: `<Image alt="" />`

### 6. Skip Links Implementation

**Check:** "Skip to main content" link as first focusable element
**Pattern:** `<a href="#main" className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4">Skip to main content</a>`
**Flag:** Missing skip links, non-functional skip targets, invisible when focused
**Required:** Visible when focused, functional target element with proper ID

### 7. Touch Target Size (WCAG 2.2)

**Check:** Interactive elements minimum 44x44px (complex pages only)
**Pattern:** `min-height: 44px; min-width: 44px` for buttons/links, or adequate spacing
**Implementation:** Enlarge touch area, not visual appearance: `padding` vs `font-size`
**Flag:** Small touch targets, insufficient spacing between interactive elements
**Exception:** Simple pages with minimal interactive content may not require this

### 8. Video Captions and Transcripts

**Check:** Video content has captions, audio has transcripts (informational content only)
**Pattern:** `<track kind="subtitles" src="/subtitles.vtt">` for video elements
**Flag:** Informational media without captions/transcripts, auto-playing audio
**Exception:** Decorative videos don't require captions if purely atmospheric

### 9. Time Limits and Auto-playing Content

**Check:** Users can disable/extend time limits, pause auto-playing content
**Pattern:** User controls for media, warnings before timeouts
**Flag:** Content that auto-plays without user control, hard time limits without options

### 10. Semantic HTML Optimisation

**Check:** Minimal div nesting, semantic elements used correctly
**Replace:** Generic divs with `<section>`, `<article>`, `<nav>`, `<aside>`
**Pattern:** Collections as `<ul>/<li>`, time elements as `<time>`, addresses as `<address>`
**Flag:** Deep div nesting (>3 levels), missed semantic opportunities

### 11. Dynamic Language Support

**Check:** Does language switching update `document.documentElement.lang`?
**Pattern:** `useEffect(() => { document.documentElement.lang = language; }, [language])`
**Flag:** Language toggles without programmatic `lang` attribute updates

### 12. Form Accessibility and Validation

**Check:** Comprehensive form support with proper validation strategy
**Pattern:** Browser-first validation with `formRef.current.checkValidity()` + `setCustomValidity()`
**Required:** `aria-labelledby`, `aria-describedby`, proper `<fieldset>/<legend>`
**Enhanced:** `role="group"`, associated labels, clear error messaging
**Flag:** Unlabeled inputs, aggressive custom validation, missing form structure

### 13. Live Regions for Dynamic Content

**Check:** All dynamic content has `aria-live` announcements
**Targets:** Carousels, filtered lists, search results, tab content, form submissions
**Pattern:** `<div aria-live="polite" className="sr-only">{updateText}</div>`
**Flag:** Dynamic content changes without screen reader announcements

### 14. Motion Preferences

**Check:** All animations respect `prefers-reduced-motion`
**Tailwind:** `motion-reduce:hidden`, `motion-reduce:block`, `motion-reduce:static`
**Pattern:** Separate experiences, not reduced animations
**Flag:** Auto-playing videos/animations without motion-reduce alternatives

### 15. Focus Management and ARIA

**Check:** Modal/drawer components trap focus properly, appropriate ARIA usage
**Library:** `focus-trap-react` with `returnFocusOnDeactivate: true`
**Pattern:** `focus-visible:ring-2`, `focus:outline-none` for custom styling
**ARIA:** Use appropriately, not overused or conflicting with semantics
**Flag:** Modals without focus trapping, unnecessary ARIA on semantic HTML, invalid ARIA combinations

### 16. Component Architecture

**Check:** Accessibility patterns are reusable across components
**Pattern:** Shared components for forms, navigation, announcements
**Required:** Consistent ARIA labelling, focus management, semantic structure
**Flag:** Repeated accessibility implementations, inconsistent patterns

## Tailwind Accessibility Classes to Verify

- `sr-only` / `focus:not-sr-only` - Screen reader content with focus visibility
- `focus-visible:ring-2` - Keyboard-only focus indicators
- `motion-reduce:hidden/block` - Motion preference support
- `aria-*` attributes properly implemented
- Custom focus styling with `focus:outline-none focus:ring-2`

## Quick Wins to Flag

1. **Missing unique page titles**
2. **Multiple h1s or skipped heading levels**
3. **Pages without main landmark from layout**
4. **Color contrast failures (use API for verification)**
5. **Missing or inappropriate alt text**
6. **Missing or non-functional skip links**
7. **Small touch targets on complex interfaces**
8. **Informational videos without captions**
9. **Deep div nesting instead of semantic HTML**
10. **Language switching without `lang` updates**
11. **Dynamic content without live regions**
12. **Missing motion-reduce alternatives**
13. **Forms without proper labelling/validation**
14. **Focus traps missing from modals**

## Audit Output Format

**Issue:** [Specific problem]
**Impact:** [Screen reader/keyboard/motion sensitivity impact]
**Fix:** [Tailwind classes and React patterns to implement]
**API Reference:** [For contrast issues, include WebAIM API results]
**Priority:** [High/Medium/Low based on WCAG AA compliance]

Focus on systematic patterns rather than individual violations. Identify reusable component improvements that scale across the application.
