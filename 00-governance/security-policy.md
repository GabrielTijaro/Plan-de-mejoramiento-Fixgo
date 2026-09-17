# Security Policy — FixGo

> Adapted from the generic framework template to FixGo's actual roles and stack
> (Firebase Auth + JWT, MySQL, Firebase Realtime Database).

---

## Security principles

1. **Defense in Depth:** Multiple security layers. If one fails, the others contain the damage.
2. **Least Privilege:** Each component has only the minimum necessary permissions.
3. **Fail Secure:** In case of error, the system denies access, does not allow it.
4. **Security by Design:** Security controls are designed from the start, not added at the end.
5. **Zero Trust:** Always verify, never implicitly trust, even within the internal network.

---

## Authentication

- Firebase Authentication issues the identity token; the backend (`auth-service`, Java /
  Spring Boot) validates it and issues its own JWT for API access.
- Access token expiration: 1 hour. Refresh token: 7 days, rotated on each use.
- Passwords are never handled directly by FixGo services — Firebase Auth owns credential
  storage. `auth-service` only stores the resulting `user_id`, role, and profile data.


### Refresh Token

- Stored in the database (with bcrypt hash)
- Mandatory rotation on each use (one refresh token = one use)
- Invalidated on logout and on password change
- ALL active tokens invalidated if use of a revoked token is detected

---

## Authorization

## Authorization — RBAC

| Role | Description | Permissions |
|------|-------------|------------|
| `DRIVER` | Registered user requesting roadside assistance | `vehicles:create`, `vehicles:read` (own), `requests:create`, `requests:read` (own) |
| `MECHANIC` | Verified workshop/technician receiving dispatch requests | `requests:read` (assigned), `requests:update` (status/diagnostic), `profile:update` (own) |
| `ADMIN` | Platform operator | `mechanics:verify`, `requests:read` (all), `audit-logs:read` |

**Permission model:**

```text
Permission: resource: action

Examples for FixGo:
  vehicles:create
  vehicles:read
  vehicles:update
  vehicles:delete
  requests:create
  requests:read
  requests:update
  mechanics:verify
  audit-logs:read

**Validation:**
- The API Gateway validates the JWT (signature and expiration)

- Each service validates the role permissions for the specific operation

- Roles are included in the JWT as claim roles: ["DRIVER", "MECHANIC", "ADMIN"]

---

## Secure communication

### Transmission

- **HTTPS mandatory** in all environments except local
- TLS 1.2 minimum; TLS 1.3 recommended
- Certificates: Let's Encrypt (staging) / Corporate CA (production)
- HSTS enabled in production

### Internal service-to-service communication

- mTLS for service-to-service communication in production (if possible with service mesh)
- Bearer token or internal API key for services that do not support mTLS

---

## Secret management

```
✗ NEVER in source code
✗ NEVER in committed .env
✗ NEVER in logs
✗ NEVER in client error messages
✓ Environment variables (injected by the orchestrator)
✓ Vault (HashiCorp Vault, AWS Secrets Manager, etc.)
✓ Kubernetes Secrets (encrypted with KMS)
```

**Secret rotation:**
- API keys: every 90 days
- TLS certificates: 60 days before expiration
- DB passwords: every 6 months or immediately if compromise is suspected

---

## Input validation and sanitization

### General rules

1. **Never trust user input.** Validate at the edge (controller) before processing.
2. **Whitelist, not blacklist.** Define what is allowed, not only what is prohibited.
3. **Reject early.** If input is invalid, respond 400 and do not process further.

### SQL Injection — Prevention

```typescript
// ✗ VULNERABLE
const result = await db.query(`SELECT * FROM users WHERE email = '${userInput}'`);

// ✓ SAFE — always use prepared parameters
const result = await db.query('SELECT * FROM users WHERE email = $1', [userInput]);
```

### XSS — Prevention

```typescript
// ✗ VULNERABLE — rendering HTML without escaping
element.innerHTML = userProvidedContent;

// ✓ SAFE — use textContent or sanitize
element.textContent = userProvidedContent;
// or with library: DOMPurify.sanitize(userProvidedContent)
```

### Validation with Zod / Joi

```typescript
// Explicit validation schema in the controller
const CreateServiceRequestSchema = z.object({
  driverId: z.string().uuid(),
  vehicleId: z.string().uuid(),
  issueDescription: z.string().min(10).max(500),
  location: z.object({
    latitude: z.number().min(-90).max(90),
    longitude: z.number().min(-180).max(180),
  }),
}); 
```

---

## OWASP Top 10 — Review checklist

| Vulnerability | Implemented control |
|---------------|-------------------|
| A01: Broken Access Control | RBAC + permission validation in each service |
| A02: Cryptographic Failures | TLS 1.2+, bcrypt for passwords, secrets in vault |
| A03: Injection | Prepared parameters in SQL, schema validation |
| A04: Insecure Design | Threat modeling in design, Security review |
| A05: Security Misconfiguration | IaC for configuration, review of defaults |
| A06: Vulnerable Components | Dependabot / Snyk for automatic updates |
| A07: Authentication Failures | JWT with rotation, brute-force protection |
| A08: Software Integrity Failures | Verify dependency checksums, SBOM |
| A09: Logging Failures | Logs without PII, centralized, with alerts |
| A10: SSRF | Whitelist of external URLs, do not follow redirects automatically |

---

## Audit and security logs

### Events that are ALWAYS recorded

```typescript
// Security events — store in a separate log, with retention > 1 year
const SECURITY_EVENTS = [
  'auth.login.success',
  'auth.login.failure',
  'auth.login.brute_force_detected',
  'auth.password.changed',
  'auth.token.revoked',
  'auth.unauthorized_access_attempt',
  'data.pii.accessed',
  'admin.role.changed',
  'admin.user.deleted',
];
```

**Required fields in security logs:**
- `userId` (or `ANONYMOUS` if not authenticated)
- `sourceIp`
- `action`
- `resource`
- `result` (SUCCESS / FAILURE)
- `timestamp`

---

## Vulnerability process

If a vulnerability is found in this documentation or in a future implementation:

1. Do not commit sensitive details (keys, tokens, real user data) to this public
   academic repository.
2. Report it privately to the instructor and, if the team repository is affected, to the
   FixGo team lead (Johan Andrés Liñan Esquivel).
3. Document the fix as an ADR if it changes an architectural decision (e.g. a change in
   how JWTs are validated).

### Remediation SLAs

| Severity | Remediation time |
|----------|----------------|
| Critical (CVSS 9-10) | 24 hours |
| High (CVSS 7-8.9) | 1 week |
| Medium (CVSS 4-6.9) | 1 month |
| Low (CVSS < 4) | Next security review |

---

## Correlations

- Non-functional security requirements → `04-requirements/non-functional.md`
- Data ownership and encryption at rest → `06-data/models.md`