# Next.js + Tailwind Architecture Audit

AI-assisted architecture auditing for Next.js applications using Tailwind CSS. This repository provides a comprehensive audit framework designed specifically for LLM consumption to identify, prioritise, and fix architectural issues systematically.

## Quick Start

### 1. Upload Your Project

Share your Next.js project files with an LLM along with the `audit.xml` document.

### 2. Run the Audit

The `audit.xml` contains everything needed to perform a comprehensive architecture audit. Simply upload both your codebase and the XML file - no additional prompts required.

The LLM will automatically:

- Apply the structured audit methodology
- Present each violation individually for review
- Provide specific architecture principle guidance
- Output structured JSON for implementation

### 3. Review Violations and Fixes

The LLM will present each architectural issue one at a time with suggested fixes. You can review, verify the LLM has interpreted the codebase correctly, and approve or modify the proposed solutions.

### 4. Process Results

At the end, the LLM will output structured JSON listing each issue that can be fed directly into implementation agents or development workflows.

### 5. Implementation Agent

Feed a prompt like this into your implementation agent (recommend to use [OpenAI Codex](https://openai.com/index/introducing-codex/)):

```markdown
Apply required changes listed in JSON architecture audit below:
[PASTE COMPLETE JSON OUTPUT]

Work through each fix systematically, maintaining code quality and testing.
```

## What Gets Audited

### Component Architecture (5 Areas)

- **Abstraction Timing**: AHA principle compliance, premature optimisation detection
- **Component Boundaries**: Server vs Client component placement, state co-location
- **Business Logic**: Separation of concerns, presentation vs logic boundaries
- **Component Responsibility**: DOTADIW principle, single responsibility assessment
- **Composition Patterns**: Inheritance vs composition, component API design

### Code Organisation (3 Areas)

- **Import Structure**: Consistent import ordering, dependency management
- **File Structure**: Naming conventions, folder organisation patterns
- **TypeScript Usage**: Appropriate complexity, type safety without over-engineering

### React Patterns (9 Areas)

- **Component Extraction**: Tailwind component timing, reusability assessment
- **State Management**: Context vs prop drilling, state complexity evaluation
- **Form Handling**: Server Actions vs manual state, validation patterns
- **Error Handling**: Consistent error strategies, boundary implementation
- **Performance**: Memoisation usage, render optimisation, code splitting
- **Hook Dependencies**: Effect dependencies, stale closure prevention
- **Component APIs**: Prop design, interface clarity, composability
- **Server Actions**: Input validation, error handling, revalidation patterns
- **Focus Management**: Route changes, modal interactions, keyboard navigation

### Code Quality (6 Areas)

- **Naming Conventions**: Consistent patterns, component declarations
- **Bundle Management**: Dynamic imports, code splitting strategies
- **State Complexity**: Reducer vs useState, synchronisation patterns
- **File Hygiene**: Component per file, size management, export patterns
- **Data Co-location**: Static data placement, configuration management
- **Type Strategy**: Schema integration, validation alignment, DRY principles

## Audit Output Format

Each issue returns structured JSON:

```json
{
  "issue": "unique-id",
  "severity": "Critical|Serious|Moderate|Minor",
  "location": "components/UserForm.tsx:42",
  "description": "Premature abstraction violation",
  "fix": {
    "before": "function getDisplayName(user, options) { /* complex logic */ }",
    "after": "const displayName = `${user.firstName} ${user.lastName}`;",
    "effort": "Low"
  }
}
```

## Implementation Workflow

### 1. Receive Audit Results

Collect all JSON issue objects from the LLM audit.

### 2. Prioritise Fixes

Issues are automatically scored using:

```
Score = Severity × Maintainability Impact × Team Productivity × Technical Debt
```

**Fix Priority:**

- **Immediate**: Critical architecture violations blocking development
- **Sprint**: Serious maintainability issues affecting team velocity
- **Backlog**: Moderate improvements enhancing code quality
- **Nice-to-have**: Minor optimisations and consistency improvements

### 3. Implement Fixes

Use the audit output to instruct implementation agents:

```
Apply this architecture fix:
- Issue: State managed too high in component tree
- Location: components/Dashboard.tsx:15
- Principle: State co-location
- Change: Move modal state from Dashboard to Modal component
- Effort: Medium

Verify state remains functional after moving closer to usage point.
```

### 4. Verify Implementation

Run validation tools for verification:

- **ESLint**: Component patterns, import organisation
- **TypeScript**: Type safety, compilation checks
- **Bundle Analyser**: Code splitting effectiveness
- **Manual testing**: Component functionality, performance assessment

## Quick Wins Identified

The audit automatically flags these common, high-impact issues:

1. Server Component overuse without interactivity
2. Premature abstractions with fewer than 3 usage sites
3. State managed above necessary scope
4. Business logic mixed in presentation components
5. Inconsistent import patterns across files
6. Mixed organisational patterns within projects
7. Over-engineered TypeScript for simple use cases
8. Component extraction too early in development
9. Manual form state instead of Server Actions
10. Inconsistent error handling patterns
11. Expensive operations in render without memoisation
12. Missing hook dependencies causing stale closures
13. Component APIs with too many props
14. Inconsistent naming conventions
15. Large components not code split appropriately

## Coverage & Limitations

**Automated Detection**: ~70% of common architecture issues via pattern matching
**Manual Assessment Required**: ~30% for complex business logic evaluation
**False Positives**: Minimised through context-aware validation

**Not Covered:**

- Domain-specific business logic assessment
- Performance profiling under load
- Team workflow optimisation
- Legacy code migration strategies

## Integration Options

### CI/CD Pipeline

```yaml
- name: Architecture Audit
  run: |
    # Upload codebase + audit.xml to LLM
    # Process JSON output
    # Fail build on Critical/Serious architecture violations
```

### Development Workflow

1. Feature branch creation
2. LLM audit on changed components
3. Fix implementation via agents
4. Automated pattern verification
5. Manual review for complex architectural decisions

## Best Practices

- **Component-First**: Design reusable patterns before implementation
- **Progressive Enhancement**: Build functionality incrementally without premature optimisation
- **Team Standards**: Establish consistent patterns across the development team
- **Regular Audits**: Run on feature branches to catch issues early
- **Architecture Documentation**: Maintain decision records for complex architectural choices

---

This framework transforms architecture auditing from subjective code reviews into systematic, AI-assisted quality assurance that scales with your development process.
