# Next.js + Tailwind Accessibility Audit

## Core Patterns

### 1. Unique Page Titles

`export const metadata = { title: "Page-specific title | Website title" }`
**Violations:** Generic titles, missing metadata
**✅ Check:** `useEffect(() => { document.title = ... }, [language])` also works

### 2. Heading Hierarchy

`h1 -> h2 -> h3` (exactly one h1 per page, no skipped levels)
**Violations:** Multiple h1s, skipped levels, missing h1

### 3. Main Landmark

`<main className="container mx-auto">{children}</main>` in layout.tsx
**Violations:** Pages without main element, inconsistent layout

### 4. Colour Contrast

Min 4.5:1 normal text, 3:1 large text/UI
**API:** `https://webaim.org/resources/contrastchecker/?fcolor=HEXCOLOR&bcolor=HEXCOLOR&api`
**⚠️ MUST calculate actual ratios before flagging - don't assume failures**
**API:** Calculate luminance manually or use contrast tools
**Violations:** Insufficient ratios, colour-only information

### 5. Alt Text

Descriptive: `<Image alt="User profile showing completion status" />`
Decorative: `<Image alt="" />`
**Violations:** Missing alt, generic text, identical alt/captions

### 6. Skip Links

```jsx
<a
  href="#main"
  className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4"
>
  Skip to main content
</a>
```

**⚠️ Only flag if:** Complex navigation, sidebars, lengthy headers
**Don't flag:** Thin headers (<100px) with immediate main content access
**Violations:** Missing skip links, non-functional targets, invisible when focused

### 7. Touch Targets (WCAG 2.2)

`min-h-[24px] min-w-[24px]` or adequate spacing
**⚠️ WCAG 2.2 standard is NOW 24px minimum (not 44px previously used)**
**Violations:** <24px targets, insufficient spacing

### 8. Video Captions

`<track kind="subtitles" src="/subtitles.vtt">` for informational content
**Violations:** Missing captions/transcripts on informational videos (not video with `aria-hidden`), auto-playing audio, autoplay without controls
**Check:** If video without controls, check that alternative controls are not provided outside element before flagging

### 9. Time Limits

User controls for auto-playing content, timeout warnings
**Violations:** Auto-play without controls, hard time limits

### 10. Semantic HTML & Components

Replace divs: `<section>`, `<article>`, `<nav>`, `<aside>`, `<ul>/<li>`, `<time>`, `<address>`
**Reusable patterns:** Consistent ARIA labelling, focus management
**Violations:** Deep div nesting (>3 levels), missed semantic opportunities

### 11. Dynamic Language

```jsx
useEffect(() => {
  document.documentElement.lang = language;
}, [language]);
```

**⚠️ Check dependency array works before flagging as broken**
**Violations:** Language switching without lang updates

### 12. Forms

```jsx
<input aria-labelledby="label-id" aria-describedby="error-id" />
<fieldset><legend>Group label</legend></fieldset>
```

Browser validation: `formRef.current.checkValidity()` + `setCustomValidity()`
**⚠️ Browser validation with `onInvalid/onInput` IS WCAG compliant for simple forms**
**Violations:** Unlabeled inputs, missing fieldset/legend

### 13. Live Regions & Focus Management

```jsx
<div aria-live="polite" className="sr-only">
  {updateText}
</div>
```

Focus traps: `focus-trap-react` with `returnFocusOnDeactivate: true`
**Targets:** Carousels, search results, tab content, modals
**Violations:** Dynamic content without announcements, missing focus traps

### 14. Motion Preferences

`motion-reduce:hidden`, `motion-reduce:block`, `motion-reduce:static`, `motion-safe:bl;ock`
**⚠️ Don't flag when Tailwind `motion-reduce:` classes already exist**
**Violations:** Auto-playing animations without motion-reduce alternatives

## WCAG 2.2 Requirements

### 15. Focus Not Obscured

**Violations:** Sticky headers/modals covering focus indicators
**Fix:** scroll-padding-top, margin adjustments

### 16. Redundant Entry

**Violations:** Multi-step forms requiring duplicate information
**Fix:** Session storage, autocomplete attributes

### 17. Accessible Authentication

**Violations:** Cognitive CAPTCHAs without alternatives
**Fix:** Email/SMS verification, biometric options

### 18. Consistent Help

**Violations:** Help resources in different locations per page
**Fix:** Global help component in layout

## Next.js Specific

### 19. SSR/Hydration & Client Routing

```jsx
useEffect(() => {
  // Client-only features
}, []);
```

Route focus management: Built-in announcer + manual focus reset
**Violations:** Hydration mismatches, focus remaining on previous page elements

### 20. Progressive Enhancement

Core functionality works without JavaScript
**Violations:** Blank pages during JS loading, broken accessibility without JS

## Tailwind Specific

### 21. Dynamic Classes & Dark Mode

Complete class names: `text-blue-500` not `text-${color}-500`
Dark mode contrast: Test all combinations, explicit dark: classes
**Violations:** Purged classes, poor dark mode contrast

## Testing Patterns

### 22. Component Testing

`@testing-library/jest-dom`, `jest-axe`
Test ARIA states, keyboard navigation, screen reader text

### 23. CI/CD Integration

`axe-core`, `pa11y-ci`, `lighthouse-ci`
Fail builds on accessibility regressions

## Quick Audit Priorities

1. Missing unique page titles
2. Multiple h1s/skipped heading levels
3. Missing main landmark
4. **Calculated** colour contrast failures
5. Missing/inappropriate alt text
6. **Complex layouts** without skip links
7. Touch targets <24px
8. **Informational videos** without captions
9. Deep div nesting vs semantic HTML
10. Dynamic content without live regions
11. **Actual missing** motion-reduce alternatives
12. Unlabeled forms
13. Missing focus traps in modals
14. Hydration accessibility mismatches
15. Missing route focus management

## Output Format

```json
{
  "issue": "unique-id",
  "wcag": ["2.4.7"],
  "severity": "Critical|Serious|Moderate|Minor",
  "location": "components/Button.tsx:42",
  "description": "Missing focus indicator",
  "fix": {
    "before": "<button className=\"bg-blue-500\">",
    "after": "<button className=\"bg-blue-500 focus-visible:ring-2\">",
    "effort": "Low"
  }
}
```
