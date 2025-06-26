# Next.js + Tailwind Architecture & Code Quality Audit

## Methodology

**Never flag an issue without confirming it actually exists:**

- ✅ Test component boundaries - verify Server vs Client Components are correctly placed
- ✅ Check abstraction timing - confirm violations of AHA principle (3+ duplications before abstraction)
- ✅ Validate component composition - ensure violations aren't just different legitimate patterns
- ✅ Verify co-location violations - check if code placement actually hinders maintainability
- ✅ Confirm TypeScript overengineering - distinguish between necessary complexity and gold-plating

## Process

This audit examines architectural patterns, component design, and code organisation following functional programming principles and Next.js App Router best practices. Focus on systematic detection of patterns that impact maintainability, performance, and developer experience.

**Interactive Review Process:**

1. Present each violation individually with specific code evidence
2. Explain the architectural concern and proposed solution
3. Allow approval, modification, or rejection before proceeding
4. Export validated findings as structured JSON for implementation agents

## Core Patterns

### 1. Premature Abstraction (AHA Principle)

```typescript
// ❌ Hasty abstraction after first duplication
function getDisplayName(
  user,
  options: { honorific?: boolean; lastName?: boolean; formal?: boolean }
) {
  // Complex conditional logic handling all variations
}
```

**Violations:** Abstractions created before 3+ similar implementations exist, over-engineered generic functions, complex option objects
**⚠️ FALSE POSITIVE:** Legitimate utility functions with clear single purpose, library wrapper functions, shared configuration objects
**✅ Check:** Count actual usage sites - only flag if abstraction exists with <3 usage patterns

### 2. Component Single Responsibility (DOTADIW)

```typescript
// ✅ Clear single responsibility
const validateUser = (userData: UserData): void => {
  /* validation only */
};
const normalizeUser = (userData: UserData): NormalizedUser => {
  /* transformation only */
};
const processUser = (userData: UserData): NormalizedUser => {
  validateUser(userData);
  const normalized = normalizeUser(userData);
  return normalized;
};
```

**Violations:** Functions mixing validation, transformation, and side effects; components handling multiple concerns; mixed business logic and presentation
**⚠️ FALSE POSITIVE:** Related operations grouped logically, React component lifecycle methods, event handlers with necessary side effects
**✅ Check:** Can the function be described in a single sentence without "and"?

### 3. Server vs Client Component Boundaries

```typescript
// ❌ Unnecessary Client Component
"use client";
export default function PostsList({ posts }: { posts: Post[] }) {
  return (
    <div>
      {posts.map((post) => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

**Violations:** Static components marked with 'use client', data presentation components requiring client boundary, auth checks in Client Components only
**⚠️ FALSE POSITIVE:** Components using hooks, event handlers, browser APIs, components with legitimate interactivity
**✅ Check:** Remove 'use client' and verify component still functions correctly

### 4. State Co-location Violations

```typescript
// ❌ State managed far from usage
// In root component:
const [isModalOpen, setIsModalOpen] = useState(false);
// Passed through 3+ component levels to reach modal
```

**Violations:** State managed above necessary scope, props drilling through multiple levels, global state for local UI concerns
**⚠️ FALSE POSITIVE:** Truly shared state, URL-managed state, server state, theme/auth context
**✅ Check:** Can state be moved closer to usage without breaking functionality?

### 5. Business Logic in Components

```typescript
// ❌ Business logic mixed with presentation
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    // Complex business logic here
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        const processedUser = {
          ...data,
          displayName: data.firstName + " " + data.lastName,
          initials: data.firstName[0] + data.lastName[0],
        };
        setUser(processedUser);
      });
  }, [userId]);
}
```

**Violations:** Data fetching logic in components, business rules in JSX, complex transformations in render functions
**⚠️ FALSE POSITIVE:** Simple display calculations, formatting functions, basic event handlers
**✅ Check:** Can business logic be extracted to custom hooks or utility functions?

### 6. Import Organisation and Dependencies

```typescript
// ❌ Disorganised imports
import { Button } from "./ui/Button";
import React, { useState } from "react";
import { User } from "@/types";
import Link from "next/link";
import { cn } from "@/lib/utils";
```

**Violations:** Random import order, relative imports for shared modules, importing entire libraries for single functions
**⚠️ FALSE POSITIVE:** Alphabetical sorting within groups, consistent patterns across team, auto-formatted imports
**✅ Check:** Follow pattern: React → Next.js → External → Internal → Relative → Types

### 7. File and Folder Structure Inconsistency

```
// ❌ Inconsistent structure
components/
├── ui/Button.tsx
├── forms/
│   └── UserForm/
│       ├── index.tsx
│       └── UserForm.tsx
└── Header.tsx
```

**Violations:** Mixed file extensions, inconsistent folder patterns, components scattered across multiple patterns
**⚠️ FALSE POSITIVE:** Legacy code being migrated, different patterns for different component types, team conventions
**✅ Check:** Does project follow a consistent pattern for similar component types?

### 8. TypeScript Over-engineering

```typescript
// ❌ Unnecessary complexity
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface ComponentProps<T extends Record<string, unknown> = {}>
  extends Omit<React.HTMLAttributes<HTMLElement>, keyof T>,
    T {}
```

**Violations:** Complex generic types for simple use cases, deep utility types without clear benefit, over-constrained type definitions
**⚠️ FALSE POSITIVE:** Domain-specific type safety, proven utility types, types preventing real runtime errors
**✅ Check:** Can simpler types achieve the same safety and clarity?

### 9. Tailwind Component Extraction Timing

```tsx
// ❌ Too early component extraction
const ButtonWrapper = ({ children }: { children: React.ReactNode }) => (
  <div className="flex items-center gap-2">{children}</div>
);
```

**Violations:** Components created for single-use styling, wrapper components with only styling purpose, @apply usage for one-off patterns
**⚠️ FALSE POSITIVE:** Reusable design system components, components with multiple responsibilities, accessibility wrappers
**✅ Check:** Is component used in 3+ places or has non-styling responsibilities?

### 10. Prop Drilling vs Context Overuse

```typescript
// ❌ Context for simple state
const ThemeContext = createContext<{
  primaryColor: string;
  secondaryColor: string;
  fontSize: number;
  setPrimaryColor: (color: string) => void;
  // ... 15 more theme properties
}>({});
```

**Violations:** Context for frequently changing values, context replacing simple prop passing, multiple unrelated values in single context
**⚠️ FALSE POSITIVE:** Auth context, theme context, router context, stable configuration values
**✅ Check:** Does context value change frequently or get consumed by many components?

### 11. Form Handling Patterns

```typescript
// ❌ Manual form state management
function CreatePostForm() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    // Manual validation logic...
    // Manual API call...
    // Manual error handling...
  };
}
```

**Violations:** Manual form state for simple forms, reinventing validation logic, not using Server Actions where appropriate
**⚠️ FALSE POSITIVE:** Complex multi-step forms, forms requiring real-time validation, forms with dynamic fields
**✅ Check:** Can form use native FormData with Server Actions or established form library?

### 12. Error Handling Patterns

```typescript
// ❌ Inconsistent error handling
async function getUser(id: string) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error("Failed to fetch user");
    }
    return await response.json();
  } catch (error) {
    console.error(error);
    return null; // Sometimes returns null, sometimes throws
  }
}
```

**Violations:** Inconsistent error return patterns, mixing throw/return error strategies, generic error messages
**⚠️ FALSE POSITIVE:** Different error handling for different error types, domain-specific error handling
**✅ Check:** Is error handling pattern consistent across similar functions?

### 13. Component Composition vs Inheritance

```typescript
// ❌ Inheritance-like patterns
interface BaseButtonProps {
  variant: "primary" | "secondary";
  size: "sm" | "md" | "lg";
}

interface IconButtonProps extends BaseButtonProps {
  icon: React.ReactNode;
}

interface TextButtonProps extends BaseButtonProps {
  text: string;
}
```

**Violations:** Deep inheritance hierarchies, forced prop inheritance, inflexible component hierarchies
**⚠️ FALSE POSITIVE:** Shared prop patterns, TypeScript utility types for common patterns, component variants
**✅ Check:** Can components be composed rather than inherited?

### 14. Server Action Implementation

```typescript
// ❌ Missing validation and error handling
"use server";
export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  const content = formData.get("content") as string;

  await db.post.create({
    data: { title, content, authorId: "hardcoded-id" },
  });

  redirect("/posts");
}
```

**Violations:** Missing input validation, no error boundaries, hardcoded values, missing revalidation
**⚠️ FALSE POSITIVE:** Simple operations with trusted input, internal-only actions, prototype code
**✅ Check:** Does action validate input and handle errors appropriately?

### 15. Performance Anti-patterns

```typescript
// ❌ Unnecessary re-renders
function ExpensiveComponent({ items }: { items: Item[] }) {
  const processedItems = items
    .filter((item) => item.active)
    .sort((a, b) => a.name.localeCompare(b.name));

  return (
    <div>
      {processedItems.map((item) => (
        <ItemCard
          key={item.id}
          item={item}
          onClick={() => console.log(item.id)}
        />
      ))}
    </div>
  );
}
```

**Violations:** Expensive calculations in render, new function creation in render, missing memoization for expensive operations
**⚠️ FALSE POSITIVE:** Simple calculations, operations that don't cause performance issues, premature optimisation
**✅ Check:** Use React DevTools Profiler to confirm performance impact before flagging

### 16. Hook Dependencies and Effects

```typescript
// ❌ Missing or incorrect dependencies
useEffect(() => {
  fetchUserData(userId, filters);
}, [userId]); // Missing 'filters' dependency

const memoizedValue = useMemo(() => {
  return expensiveCalculation(data);
}, []); // Missing 'data' dependency
```

**Violations:** Missing dependencies in useEffect/useMemo/useCallback, exhaustive-deps ESLint rule disabled, stale closure bugs
**⚠️ FALSE POSITIVE:** Intentionally omitted dependencies with comments explaining why, custom hook patterns that manage dependencies
**✅ Check:** Enable exhaustive-deps rule and verify warnings are addressed

### 17. Component API Design

```typescript
// ❌ Unclear component APIs
function DataTable({
  data,
  showPagination,
  paginationPosition,
  sortable,
  sortBy,
  sortDirection,
  onSort,
  filterable,
  filterBy,
  onFilter,
}: // ... 20 more props
DataTableProps) {}
```

**Violations:** Too many props, unclear prop relationships, boolean props that could be enums, missing required/optional clarity
**⚠️ FALSE POSITIVE:** Complex components with legitimate configuration needs, props matching external API requirements
**✅ Check:** Can component API be simplified through composition or configuration objects?

### 18. Naming Conventions

```typescript
// ❌ Inconsistent naming
const userInfo = getUserData();
const userData = processUserInfo();
const user_profile = createUserProfile();
const GetUserComponent = () => {};
```

**Violations:** Mixed naming conventions, unclear variable purposes, non-descriptive names, incorrect casing for types/functions
**⚠️ FALSE POSITIVE:** Domain-specific terminology, established team conventions, external API naming requirements
**✅ Check:** Are naming patterns consistent within the project scope?

### 19. Code Splitting and Dynamic Imports

```typescript
// ❌ Missing code splitting for large components
import AdminPanel from "@/components/AdminPanel"; // Large component
import Chart from "@/components/Chart"; // Heavy charting library
import VideoPlayer from "@/components/VideoPlayer"; // Large media component

export default function Dashboard() {
  return (
    <div>
      <AdminPanel />
      <Chart />
      <VideoPlayer />
    </div>
  );
}
```

**Violations:** Large components not dynamically imported, admin-only features in main bundle, heavy libraries loaded unconditionally
**⚠️ FALSE POSITIVE:** Small components, components always needed, components with SSR requirements
**✅ Check:** Use bundle analyzer to identify components >50kB that could be code split

### 20. State Management Complexity

```typescript
// ❌ Over-engineered state management
const useComplexState = () => {
  const [state, dispatch] = useReducer(complexReducer, initialState);
  const [derivedState1, setDerivedState1] = useState();
  const [derivedState2, setDerivedState2] = useState();
  // Multiple useEffect hooks synchronising state
  // Custom selectors and memoisation
};
```

**Violations:** useReducer for simple state, multiple related useState calls, complex state synchronisation logic
**⚠️ FALSE POSITIVE:** Complex forms, state machines, legitimate complex business logic
**✅ Check:** Can state be simplified to fewer useState calls or moved to URL/server?

## Quick Audit Priorities

1. **Server Component overuse** - Client boundaries without interactivity
2. **Premature abstractions** - Generic functions with <3 usage sites
3. **State placement** - Local UI state managed globally
4. **Business logic location** - Complex logic mixed in components
5. **Import organisation** - Inconsistent import patterns across files
6. **File structure** - Mixed organisational patterns
7. **TypeScript complexity** - Over-engineered types for simple use cases
8. **Component extraction timing** - Tailwind components created too early
9. **Form handling** - Manual state for simple forms vs Server Actions
10. **Error handling consistency** - Mixed error patterns across similar functions
11. **Performance issues** - Expensive operations in render without memoisation
12. **Hook dependencies** - Missing dependencies causing stale closures
13. **Component APIs** - Too many props or unclear interfaces
14. **Naming conventions** - Inconsistent patterns within project scope
15. **Bundle size** - Large components not code split appropriately

## Output Format

```json
{
  "issue": "unique-id",
  "severity": "Critical|Serious|Moderate|Minor",
  "location": "file:line",
  "description": "brief description",
  "fix": {
    "before": "problematic code",
    "after": "improved code",
    "effort": "Low|Medium|High"
  }
}
```
