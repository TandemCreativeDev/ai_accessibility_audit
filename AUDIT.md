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

**Check:** Interactive elements minimum 24x24px CSS pixels
**Pattern:** `min-h-[24px] min-w-[24px]` or adequate spacing between targets
**Implementation:** Use padding for touch area, not just visual size
**Flag:** Small touch targets, insufficient spacing (<24px) between interactive elements
**Note:** Need not change actual icon sizes, only touch target around it

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

## New WCAG 2.2 Requirements (Often Missed)

### 17. Focus Not Obscured (Minimum)

**Check:** Focused elements remain at least partially visible
**Pattern:** Avoid sticky headers/footers that cover focus indicators
**Flag:** Modal overlays, fixed elements obscuring keyboard focus
**Fix:** Add scroll-padding-top or margin adjustments

### 18. Redundant Entry

**Check:** Previously entered information persists without re-entry
**Pattern:** Form data persistence, auto-fill support
**Flag:** Multi-step forms requiring duplicate information
**Fix:** Session storage, proper autocomplete attributes

### 19. Accessible Authentication (Minimum)

**Check:** Authentication doesn't rely solely on cognitive tests
**Pattern:** Alternative authentication methods, no puzzle CAPTCHAs
**Flag:** Complex cognitive CAPTCHAs without alternatives
**Fix:** Email/SMS verification, biometric options

### 20. Consistent Help

**Check:** Help mechanisms appear in same location across pages
**Pattern:** Consistent chat widget, help link placement
**Flag:** Help resources in different locations per page
**Fix:** Global help component in layout

## Next.js Specific Accessibility Checks

### 21. SSR/Hydration Accessibility

**Check:** No hydration mismatches breaking accessibility
**Pattern:** Conditional rendering with `useEffect` for client-only features
**Flag:** Time-based rendering, browser API usage in SSR
**Fix:** Proper hydration boundaries, suppressHydrationWarning

### 22. Client-Side Routing Focus

**Check:** Focus management during route transitions
**Pattern:** Route announcer, focus reset to main content
**Flag:** Focus remains on previous page elements
**Fix:** Next.js built-in route announcer, manual focus management

### 23. Progressive Enhancement

**Check:** Core functionality works without JavaScript
**Pattern:** SSG/SSR for critical content, progressive enhancement
**Flag:** Blank pages during JS loading, broken accessibility features
**Fix:** Server-rendered accessible defaults

## Tailwind-Specific Patterns

### 24. Dynamic Class Accessibility

**Check:** No dynamic class generation breaking purge
**Pattern:** Complete class names, not string concatenation
**Flag:** `text-${color}-500` patterns, conditional partial classes
**Fix:** Class maps, complete conditional classes

### 25. Dark Mode Contrast

**Check:** Maintain contrast ratios in both themes
**Pattern:** Test all color combinations in light/dark modes
**Flag:** Poor contrast in dark mode, invisible focus indicators
**Fix:** Theme-aware color systems, explicit dark mode classes

## Automated Testing Integration

### 26. Component-Level Testing

**Check:** Accessibility tests for each component
**Tools:** `@testing-library/jest-dom`, `jest-axe`
**Pattern:** Test ARIA states, keyboard navigation, screen reader text
**Flag:** Components without accessibility test coverage

### 27. CI/CD Integration

**Check:** Automated accessibility testing in pipelines
**Tools:** `axe-core`, `pa11y-ci`, `lighthouse-ci`
**Pattern:** Fail builds on accessibility regressions
**Flag:** No automated accessibility checks

## AI-Optimized Audit Output Format

```json
{
  "issue": {
    "id": "unique-identifier",
    "wcag": ["2.4.7", "2.5.5"],
    "severity": "Critical|Serious|Moderate|Minor",
    "component": "Button|Form|Navigation|etc",
    "location": {
      "file": "components/Button.tsx",
      "line": 42,
      "selector": "button.primary-action"
    }
  },
  "context": {
    "description": "Clear description of the issue",
    "userImpact": "How this affects real users",
    "affectedUsers": ["Screen reader", "Keyboard", "Low vision"]
  },
  "fix": {
    "effort": "Low|Medium|High",
    "code": {
      "before": "// Current implementation",
      "after": "// Fixed implementation"
    },
    "tailwindClasses": ["focus:ring-2", "focus:ring-offset-2"],
    "testing": "How to verify the fix"
  },
  "aiPrompt": "Fix button component missing focus indicator: Add Tailwind focus-visible:ring-2 focus-visible:ring-blue-500 to primary buttons in components/Button.tsx"
}
```

## Prioritization Matrix

### Severity Levels

- **Blocker:** Prevents access to content/functionality
- **Critical:** Significant barriers to primary tasks
- **Serious:** Major usability issues
- **Moderate:** Noticeable problems
- **Minor:** Polish issues

### Impact Scoring

```
Score = Severity × Frequency × User Impact × Legal Risk
```

### Fix Priority

1. **Immediate:** Blockers + high legal risk
2. **Sprint:** Critical issues on key pages
3. **Backlog:** Moderate issues
4. **Nice-to-have:** Minor enhancements

## Quick Wins to Flag

1. **Missing unique page titles**
2. **Multiple h1s or skipped heading levels**
3. **Pages without main landmark from layout**
4. **Color contrast failures (use API for verification)**
5. **Missing or inappropriate alt text**
6. **Missing or non-functional skip links**
7. **Touch targets below 24x24px**
8. **Informational videos without captions**
9. **Deep div nesting instead of semantic HTML**
10. **Language switching without `lang` updates**
11. **Dynamic content without live regions**
12. **Missing motion-reduce alternatives**
13. **Forms without proper labelling/validation**
14. **Focus traps missing from modals**
15. **Hydration mismatches affecting accessibility**
16. **Missing focus management on route changes**
17. **Dark mode contrast failures**
18. **Dynamic Tailwind classes breaking**

## Automated Tool Coverage

- **axe-core:** ~57% of WCAG issues, zero false positives
- **Pa11y:** Command-line friendly, CI/CD integration
- **Lighthouse:** Quick assessment, subset of axe tests
- **WAVE API:** Visual overlay, 10k free scans/month
- **Manual Testing:** Required for remaining ~43% of issues

Focus on systematic patterns rather than individual violations. Identify reusable component improvements that scale across the application.
