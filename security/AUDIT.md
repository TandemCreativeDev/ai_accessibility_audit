# Next.js Security Audit Information Sheet

## Methodology

**Never flag an issue without confirming it actually exists:**

✅ **Framework Context**: Next.js Server Components and built-in protections invalidate many traditional web vulnerabilities
✅ **Environment Variables**: `NEXT_PUBLIC_` prefixed variables are intentionally client-exposed
✅ **Test Dynamic Functionality**: Server Actions, middleware chains, authentication flows require runtime validation
✅ **Distinguish Risk Levels**: Business logic flaws vs configuration issues vs theoretical vulnerabilities

## Process

Before producing output, present each violation with specifics to me, and your proposal for rectifying it, one violation at a time. I will approve, request changes or disapprove it. Once we have run through all, you can produce final json output.

## Core Patterns

### 1. Server Component Authentication

```javascript
export default async function ProtectedPage() {
  const session = await auth();
  if (!session?.user) redirect("/login");
}
```

**Violations:** Missing server-side auth checks, client-only authentication
**⚠️ FALSE POSITIVE:** Client Component auth when server auth already exists
**✅ Check:** Verify `auth()` called in Server Component, not just Client

### 2. Server Actions CSRF Protection

```javascript
"use server";
export async function updateProfile(formData: FormData) {
  const session = await auth();
  if (!session) throw new Error("Unauthorized");
}
```

**Violations:** Custom API routes without CSRF protection, bypassing Server Actions
**⚠️ FALSE POSITIVE:** Missing CSRF tokens in Server Actions (automatically protected in Next.js 14+)

### 3. Environment Variable Security

```javascript
// ❌ Exposed secrets
const API_KEY = process.env.NEXT_PUBLIC_SECRET_KEY;
// ✅ Properly scoped
const API_KEY = process.env.SECRET_API_KEY; // server-only
```

**Violations:** Sensitive data in `NEXT_PUBLIC_` variables, hardcoded secrets
**⚠️ FALSE POSITIVE:** Public config correctly prefixed with `NEXT_PUBLIC_`

### 4. SQL Injection Prevention

```javascript
// ✅ Safe with Prisma
const posts = await prisma.post.findMany({
  where: { title: { contains: userInput } },
});
```

**Violations:** String concatenation in queries, `$queryRawUnsafe` usage
**⚠️ FALSE POSITIVE:** Prisma parameterised queries, properly structured `$queryRaw`

### 5. XSS Prevention in SSR

```javascript
// ✅ Auto-escaped
return <h1>{user.name}</h1>;
// ❌ Dangerous
return <div dangerouslySetInnerHTML={{ __html: userContent }} />;
```

**Violations:** `dangerouslySetInnerHTML` without sanitisation
**⚠️ FALSE POSITIVE:** React's automatic escaping in JSX expressions

### 6. Input Validation

```javascript
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});
const data = schema.parse(formData);
```

**Violations:** No input validation on Server Actions/API routes
**⚠️ FALSE POSITIVE:** Validation in display-only components

### 7. API Route Authentication

```javascript
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session) return NextResponse.json({}, { status: 401 });
}
```

**Violations:** Missing auth checks in data-modifying endpoints
**⚠️ FALSE POSITIVE:** Public endpoints that shouldn't require authentication

### 8. Rate Limiting

```javascript
export async function middleware(request: NextRequest) {
  const { success } = await rateLimit.limit(request.ip);
  if (!success) return new NextResponse("Rate limited", { status: 429 });
}
```

**Violations:** No rate limiting on auth/sensitive endpoints
**⚠️ FALSE POSITIVE:** Rate limiting on static assets or cached responses

### 9. Secure Cookie Configuration

```javascript
cookies().set("session", token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "lax",
});
```

**Violations:** Missing `httpOnly`, `secure`, or `sameSite` attributes
**⚠️ FALSE POSITIVE:** Development-only insecure cookies

### 10. File Upload Security

```javascript
const ALLOWED_TYPES = ["image/jpeg", "image/png"];
if (!ALLOWED_TYPES.includes(file.type)) return error;
const buffer = await file.arrayBuffer();
if (!isValidImageFile(buffer)) return error;
```

**Violations:** No file type/size validation, missing magic byte checks
**⚠️ FALSE POSITIVE:** Type checks in preview/display components

### 11. Authorization Beyond Authentication

```javascript
const session = await auth();
if (!session?.user?.role === "admin") redirect("/unauthorized");
```

**Violations:** Authentication without role/permission checks
**⚠️ FALSE POSITIVE:** Public pages accessible to all authenticated users

### 12. Content Security Policy

```javascript
const cspHeader = `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`;
response.headers.set("Content-Security-Policy", cspHeader);
```

**Violations:** Missing CSP, `unsafe-inline`, overly permissive policies
**⚠️ FALSE POSITIVE:** Development CSP with `unsafe-eval`

### 13. SSRF Prevention

```javascript
const ALLOWED_DOMAINS = ["api.github.com", "api.stripe.com"];
if (!ALLOWED_DOMAINS.includes(urlObj.hostname)) {
  throw new Error("Domain not allowed");
}
```

**Violations:** User-controlled URLs in fetch operations
**⚠️ FALSE POSITIVE:** Hardcoded, legitimate external API calls

### 14. Error Information Disclosure

```javascript
} catch (error) {
  // ❌ Exposes internals
  return NextResponse.json({ error: error.message });
  // ✅ Safe
  return NextResponse.json({ error: 'Invalid request' }, { status: 400 });
}
```

**Violations:** Exposing stack traces, database errors, file paths
**⚠️ FALSE POSITIVE:** User-friendly validation error messages

### 15. Session Management

```javascript
export async function createSession(userId: string) {
  const session = await encrypt({ userId, expiresAt });
  cookies().set("session", session, {
    httpOnly: true,
    expires: expiresAt,
  });
}
```

**Violations:** Sessions in localStorage, missing expiration, weak tokens
**⚠️ FALSE POSITIVE:** UI state in sessionStorage (non-auth data)

### 16. Security Headers

```javascript
// next.config.js
async headers() {
  return [{
    source: '/(.*)',
    headers: [
      { key: 'X-Frame-Options', value: 'DENY' },
      { key: 'X-Content-Type-Options', value: 'nosniff' }
    ]
  }];
}
```

**Violations:** Missing security headers in production
**⚠️ FALSE POSITIVE:** Headers set via CDN or middleware instead

### 17. Route Parameter Validation

```javascript
export default async function PostPage({ params }: { params: { id: string } }) {
  const { id } = z.object({ id: z.string().uuid() }).parse(params);
  const post = await getPost(id);
}
```

**Violations:** Direct use of params in database queries
**⚠️ FALSE POSITIVE:** Simple display logic with route params

### 18. Dependency Vulnerabilities

**Commands:** `npm audit --audit-level=moderate`, `npx snyk test`
**Violations:** High/Critical vulnerabilities with available fixes
**⚠️ FALSE POSITIVE:** Low severity in dev dependencies, scanner false positives

## Quick Audit Priorities

1. Server Component auth verification (`await auth()`)
2. API route authentication on protected endpoints
3. Environment variable scope (`NEXT_PUBLIC_` misuse)
4. Input validation on Server Actions/API routes
5. SQL injection via string concatenation
6. File upload validation (type, size, content)
7. Rate limiting on auth endpoints
8. Security headers in production
9. Session cookie security attributes
10. Error message information disclosure
11. Authorization checks beyond authentication
12. XSS via `dangerouslySetInnerHTML`
13. SSRF in user-controlled fetch operations
14. CSP implementation with nonces
15. Dependency audit (`npm audit`)

## Output Format

```json
{
  "issue": "unique-id",
  "severity": "Critical|Serious|Moderate|Minor",
  "location": "file:line",
  "description": "brief description",
  "commands": ["npm install bcrypt"],
  "fix": {
    "before": "vulnerable code",
    "after": "secure code",
    "effort": "Low|Medium|High"
  }
}
```
