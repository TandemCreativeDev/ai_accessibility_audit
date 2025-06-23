# Next.js Security Audit

AI-assisted security auditing for Next.js applications. This repository provides a comprehensive audit framework designed specifically for LLM consumption to identify, prioritise, and fix security vulnerabilities systematically.

## Quick Start

### 1. Upload Your Project

Share your Next.js project files with an LLM along with this `AUDIT.md` document.

### 2. Run the Audit

Use this prompt template:

```
Using the `AUDIT.md` guide, perform a comprehensive security audit of this Next.js project. Focus on the 19 core security areas and output findings in the specified JSON format for each vulnerability discovered.

Prioritise:
- Authentication and authorisation flaws
- Input validation vulnerabilities
- Environment variable exposure
- SQL injection and XSS risks
- Critical production security gaps

Provide actionable fixes with specific code examples.

Before producing output, present each violation with specifics to me, and your proposal for rectifying it, one violation at a time. I will approve, request changes or disapprove it. Once we have run through all, you can produce final json output.
```

### 3. Review Violations and Fixes

The LLM will present each vulnerability, one at a time, with suggested fixes. You can verify the LLM has correctly identified actual security issues and approve appropriate remediation approaches.

### 4. Process Results

At the end, the LLM will output a structured JSON listing each vulnerability that can be fed directly into implementation agents or development workflows.

## 5. Implementation Agent

Feed a prompt like this into your implementation agent:

```markdown
Apply required security fixes listed in JSON audit below:
[PASTE COMPLETE JSON OUTPUT]

Work through each fix systematically, maintaining code quality and testing.
```

## What Gets Audited

### Authentication & Authorisation (4 Areas)

- **Server-side Authentication**: Proper auth checks in Server Components
- **API Route Protection**: Authentication on protected endpoints
- **Authorisation Logic**: Role and permission validation beyond authentication
- **Session Management**: Secure session handling and cookie configuration

### Input & Data Security (6 Areas)

- **Input Validation**: Server Action and API route validation
- **SQL Injection Prevention**: Parameterised queries and safe ORM usage
- **XSS Prevention**: Safe content rendering and sanitisation
- **File Upload Security**: Type validation and content verification
- **Route Parameter Validation**: Secure handling of dynamic route parameters
- **Error Information Disclosure**: Preventing sensitive data leakage

### Infrastructure Security (5 Areas)

- **Environment Variables**: Proper secret management and scoping
- **Security Headers**: CSP, HSTS, and other protective headers
- **Rate Limiting**: Protection against abuse and brute force attacks
- **CSRF Protection**: Server Actions vs API route security patterns
- **SSRF Prevention**: Validation of user-controlled external requests

### Third-Party Integration (4 Areas)

- **Webhook Security**: Signature verification and payload validation
- **Dependency Vulnerabilities**: Automated vulnerability scanning
- **Content Security Policy**: Nonce-based script execution control
- **Third-party API Security**: Secure external service integration

## Audit Output Format

Each vulnerability returns structured JSON:

```json
{
  "issue": "unique-id",
  "severity": "Critical|Serious|Moderate|Minor",
  "location": "file:line",
  "description": "brief description",
  "fix": {
    "before": "vulnerable code",
    "after": "secure code",
    "effort": "Low|Medium|High"
  }
}
```

## Implementation Workflow

### 1. Receive Audit Results

Collect all JSON vulnerability objects from the LLM audit.

### 2. Prioritise Fixes

Vulnerabilities are automatically scored using:

```
Score = Severity × Exploitability × Business Impact × Compliance Risk
```

**Fix Priority:**

- **Immediate**: Critical auth bypasses, SQL injection, data exposure
- **Sprint**: Serious vulnerabilities on production endpoints
- **Backlog**: Moderate security improvements
- **Nice-to-have**: Minor hardening enhancements

### 3. Implement Fixes

Use the audit output to instruct implementation agents:

```
Apply this security fix:
- Issue: Missing authentication check
- Location: app/api/users/route.ts:15
- Severity: Critical
- Change: Add "const session = await auth(); if (!session) return 401;"
- Effort: Low

Verify the fix prevents unauthorised access to the endpoint.
```

### 4. Verify Implementation

Run security validation tools:

- **npm audit**: Dependency vulnerability scanning
- **Snyk**: Advanced vulnerability detection
- **OWASP ZAP**: Dynamic security testing
- **Manual testing**: Authentication bypass attempts

## Quick Wins Identified

The audit automatically flags these common, high-impact vulnerabilities:

1. Missing server-side authentication checks
2. API routes without authorisation
3. Secrets exposed via `NEXT_PUBLIC_` variables
4. Input validation gaps in Server Actions
5. SQL injection via string concatenation
6. XSS through `dangerouslySetInnerHTML`
7. Unvalidated file uploads
8. Missing rate limiting on auth endpoints
9. Insecure session cookie configuration
10. Error messages exposing internal details
11. Missing security headers in production
12. Unvalidated webhook endpoints
13. SSRF in user-controlled fetch operations
14. High-severity dependency vulnerabilities
15. Missing Content Security Policy
16. Direct route parameter usage in queries

## Coverage & Limitations

**Automated Detection**: ~70% of common vulnerabilities via pattern matching
**Manual Testing Required**: ~30% for complex business logic flaws
**False Positives**: Minimised through context-aware validation

**Not Covered:**

- Advanced persistent threats
- Social engineering vulnerabilities
- Infrastructure-specific misconfigurations
- Complex business logic flaws

## Integration Options

### CI/CD Pipeline

```yaml
- name: Security Audit
  run: |
    npm audit --audit-level=high
    # Upload codebase + AUDIT.md to LLM
    # Process JSON output
    # Fail build on Critical/Serious issues
```

### Development Workflow

1. Feature branch creation
2. LLM security audit on changed components
3. Fix implementation via agents
4. Automated vulnerability scanning
5. Manual penetration testing for critical paths

## Best Practices

- **Server-First Security**: Verify all security checks happen server-side
- **Defence in Depth**: Layer multiple security controls
- **Least Privilege**: Grant minimum necessary permissions
- **Input Validation**: Validate all user inputs at API boundaries
- **Regular Audits**: Run on feature branches and scheduled intervals
- **Security Training**: Use audit results to educate developers on secure patterns

---

This framework transforms security auditing from manual penetration testing into systematic, AI-assisted vulnerability assessment that scales with your development process.
