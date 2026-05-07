# AI Security Skill: React Hardening & Auto-Vulnerability Fixer
# Author: @vokhactam | vokhactam.user@gmail.com
# Version: 1.0.0
# Date: 2026-05-07

## Objectives
This skill provides the AI with the ability to automatically:
- Detect common security vulnerabilities in React code.
- Fix (patch) vulnerabilities during code generation or code review.
- Apply default security configurations and best practices for every React project.
- Ensure the generated application is secure against real-world attacks (XSS, CSRF, injection, RCE, dependency vulnerabilities...).

## Scope
- Frontend React code (JSX, functional components, hooks).
- Build configurations (Webpack, Vite, Next.js).
- Dependency management (`package.json`).
- Interaction with backend APIs (alerts and guidance).

## Security Golden Rules
1. **Never trust user data** – always escape, validate, and sanitize.
2. **Prohibit the use of `dangerouslySetInnerHTML`** unless absolutely necessary and sanitized using DOMPurify.
3. **Block `javascript:` protocol URLs** in `href`, `src`, and `action` attributes.
4. **Mandatory CSRF protection** if the API uses cookie-based authentication.
5. **Continuous dependency updates** – automatically audit and propose upgrades.
6. **No hardcoded secrets/keys** in client-side code.
7. **Use TypeScript** to prevent XSS through data typing.

## Auto-Fix Pipeline
When the AI generates code or receives a request to create a React website, it performs the following steps sequentially:

### Step 0: Check React version and dependencies
- Run virtual command: `npm list react react-dom next` → determine versions.
- If a version with critical vulnerabilities is detected (e.g., React 19.0.0 → 19.2.2):
    - Automatically upgrade in `package.json` to a patched version (19.0.3, 19.1.4, 19.2.3 or higher).
    - Add the `"audit": "npm audit fix"` script to `package.json`.
- Scan all dependencies using `npm audit` (simulated) – if high/critical vulnerabilities exist, automatically update to secure versions.

### Step 1: Detect and Fix XSS
**Patterns to look for in code:**
- `dangerouslySetInnerHTML={{ __html: ... }}` with unsanitized variables.
- `<a href={userInput}>`, `<img src={userInput}>` without validation.
- `eval(userInput)`, `new Function(userInput)`.

**Automatic actions:**
- Replace `dangerouslySetInnerHTML` with a secure component:
```javascript
import DOMPurify from 'dompurify';
const SafeHTML = ({ html }) => <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />;
```
- Add the `sanitizeUrl` function to check protocols:
```javascript
const sanitizeUrl = (url) => {
  const safe = ['http:', 'https:', 'mailto:', 'tel:'];
  try {
    const parsed = new URL(url);
    if (!safe.includes(parsed.protocol)) return '#';
  } catch { return '#'; }
  return url;
};
```
- Automatically wrap all `href={variable}` with `href={sanitizeUrl(variable)}`.

### Step 2: CSRF Protection
- If `fetch` or `axios` is detected using `credentials: 'include'`:
    - Automatically add the `X-CSRF-Token` header retrieved from cookies or meta tags.
    - Guide the backend to implement CSRF tokens (if the AI can also modify the backend).
- Add middleware: use libraries like `csurf` (Express) or `django.middleware.csrf`.

### Step 3: Prevent Indirect Injection
- While React frontend doesn't directly handle SQL, if the AI generates API endpoints:
    - Mandate the use of parameterized queries (e.g., do not use string concatenation like `"SELECT * FROM users WHERE id = " + userId`).
    - Use ORMs (Prisma, TypeORM) or secure query builders.
- If string concatenation is detected in fetch URLs: switch to using `URLSearchParams` or `encodeURIComponent`.

### Step 4: Strong Authentication and Authorization
- Always add logic to check JWT tokens or sessions for every protected request.
- Prevent IDOR: verify resource access permissions (e.g., `userId !== req.user.id`).
- Generate default code for `AuthContext` and `ProtectedRoute`.

### Step 5: Frontend API Security
- Proposed Rate Limiting: Add `axios-retry` and implement 429 error notifications.
- Validate all input on the client-side (using Zod or Yup), though this does not replace server-side validation.
- Automatically hide API keys – do not allow hardcoding; remind to use environment variables like `VITE_*` or `NEXT_PUBLIC_*` while warning that they are not 100% secure.

### Step 6: Patch React2Shell and RSC Vulnerabilities (if using Next.js App Router)
- Check `next.config.js`: is `experimental.reactCompiler` enabled or are Server Components used?
- If yes, mandate:
    - Upgrade to `react@19.0.3+`, `react-dom@19.0.3+`, `next@15.0.4+` (or corresponding patched versions).
    - Add CSP header configurations to mitigate impact if patching isn't immediate.
    - Do not generate code that sends user data directly into Server Actions without validation.

### Step 7: Dependency Auto-healing
- Automatically add to `package.json`:
```json
"scripts": {
  "security-check": "npm audit && npx snyk test",
  "preinstall": "npx only-allow npm"
}
```
- If a vulnerable package is detected (e.g., lodash 4.17.x with a CVE), automatically upgrade to the patched version.
- Use `overrides` in `package.json` to force secure versions when direct upgrades are not possible.

### Step 8: Final Security Report
After generating code, the AI outputs a `SECURITY_AUDIT_REPORT.md` file including:
- A list of detected and automatically fixed vulnerabilities.
- Versions of upgraded packages.
- Recommendations that couldn't be automated (e.g., setting up CSP headers, HTTPS redirection).

## Example: AI Automatically Fixing Code Vulnerabilities

**Input code (insecure):**
```javascript
function Comment({ userInput }) {
  return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
}
```

**Output after applying the skill:**
```javascript
import DOMPurify from 'dompurify';

function Comment({ userInput }) {
  const sanitized = DOMPurify.sanitize(userInput);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

**Input code (XSS via href):**
```javascript
<a href={userProvidedLink}>Click</a>
```

**Output:**
```javascript
import { sanitizeUrl } from './utils/security';

<a href={sanitizeUrl(userProvidedLink)}>Click</a>
```

## Default Configuration for a New React Project (Secure Template)
When the AI creates a new project, it automatically adds:
- `package.json` with security scripts.
- `.env.example` file containing sample environment variables (no real secrets).
- `utils/security.js` file containing `sanitizeUrl` and `sanitizeHtml`.
- `middleware/csrf.js` file if using Next.js.
- `security-headers.js` file (CSP, HSTS, X-Frame-Options) for the server-side (Next.js or Express).

## Final Checklist (Zero-trust checklist)
Before outputting code, the AI asks itself:
- [ ] Are all `dangerouslySetInnerHTML` instances wrapped with DOMPurify?
- [ ] Do all `href`/`src` attributes receiving user input pass through `sanitizeUrl`?
- [ ] Have dependencies been updated to versions without critical CVEs?
- [ ] Do Server Components use React versions earlier than 19.0.3?
- [ ] Are CSRF tokens sent in all data-modifying requests?
- [ ] Are there no hardcoded secrets in the client-side code?

If all boxes are checked → output code along with the security audit report.
