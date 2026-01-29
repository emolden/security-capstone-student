# DREAD Risk Register - KeyKeeper

## Risk Assessment Table

| Risk ID | Title | Description | D | R | E | A | D | Total | Mitigation | Status |
|---------|-------|-------------|---|---|---|---|---|-------|------------|--------|
| R1 | Secrets Stored in Plaintext | Secrets are stored unencrypted in MongoDB database. If database is compromised, all secrets are immediately exposed in readable format. | 10 | 10 | 7 | 10 | 8 | **45** | Implement AES-256 encryption for all secrets at rest. Use separate encryption keys per environment. Store encryption keys securely (AWS KMS, HashiCorp Vault). | Open |
| R2 | IDOR on Secret Reveal | Users can reveal any secret by changing the ID in the URL (/items/:id/reveal). No ownership verification before revealing secret values. | 9 | 10 | 10 | 10 | 9 | **48** | Add server-side ownership checks before revealing. Verify req.user.id matches secret.owner or user has explicit share permission. Use UUIDs instead of sequential IDs. | Open |
| R3 | IDOR on Secret Revocation | Users can revoke any secret by changing the ID in the request (/items/:id/revoke). No ownership verification allows unauthorized revocation. | 8 | 10 | 10 | 10 | 9 | **47** | Add server-side ownership checks. Verify user owns secret or has admin role before allowing revocation. Log all revocation attempts. | Open |
| R4 | All Users Can View All Secrets | GET /items returns all secrets in the system to any authenticated user, regardless of ownership or permissions. | 10 | 10 | 10 | 10 | 8 | **48** | Filter GET /items query by ownership. Only return secrets where owner=req.user.id OR user in sharedWith array. Implement proper RBAC. | Open |
| R5 | No Rate Limiting on Login | Unlimited login attempts allowed. Enables brute force password attacks and credential stuffing attacks. | 7 | 10 | 10 | 10 | 10 | **47** | Implement rate limiting: 5 attempts per 15 minutes per username, 20 attempts per IP. Add CAPTCHA after 3 failures. Implement account lockout after 10 failures. | Open |
| R6 | No Step-Up Authentication for Reveals | Users can reveal secret values without re-entering password, even hours after initial login. Stolen or unattended sessions can reveal all secrets. | 8 | 10 | 10 | 10 | 7 | **45** | Require password re-entry before revealing secrets. Implement step-up authentication with 5-minute validity window. Consider biometric verification for high-value secrets. | Open |
| R7 | Weak Password Policy | Only 8 characters required with no complexity requirements. Passwords like 'password1' and '12345678' are accepted. | 7 | 10 | 10 | 10 | 10 | **47** | Require minimum 12 characters with uppercase, lowercase, numbers, and special characters. Implement password strength checking (zxcvbn). Reject common passwords. | Open |
| R8 | Email Enumeration via Registration | Registration endpoint returns 'Email already exists' error, allowing attackers to enumerate valid email addresses in the system. | 5 | 10 | 10 | 10 | 10 | **45** | Return generic message for existing emails. Implement same-length response times. Consider email verification flow that doesn't reveal account existence. | Open |
| R9 | JWT Stored in localStorage | Authentication tokens stored in browser localStorage instead of HttpOnly cookies. Vulnerable to XSS token theft even with current XSS protections. | 8 | 10 | 8 | 10 | 7 | **43** | Move JWT to HttpOnly, Secure, SameSite=Strict cookies. Remove localStorage usage. Implement CSRF protection with tokens. | Open |
| R10 | No Audit Logging on Secret Access | System doesn't log when secrets are viewed or revealed. No way to track unauthorized access or investigate security incidents. | 6 | 10 | 10 | 10 | 8 | **44** | Log all secret access including views and reveals with user ID, secret ID, timestamp, IP address, and action type. Make logs immutable and stored separately. | Open |
| R11 | NoSQL Injection Risk | Although current validation prevents injection, any changes to validation logic or new endpoints could introduce NoSQL injection vulnerabilities. | 9 | 7 | 6 | 10 | 7 | **39** | Maintain strict input validation using Zod schemas. Always use parameterized queries. Never interpolate user input into database queries. Regular security reviews of new endpoints. | Open |
| R12 | Missing Security Headers | No Content-Security-Policy, X-Frame-Options, HSTS, or other security headers. Increases risk of XSS, clickjacking, and downgrade attacks. | 6 | 10 | 9 | 10 | 10 | **45** | Install and configure helmet.js middleware. Set strict CSP policy. Enable HSTS with long max-age. Add X-Frame-Options: DENY. | Open |
| R13 | No Two-Factor Authentication | Only password required for login. Compromised passwords grant full access to all secrets without secondary verification. | 8 | 9 | 8 | 10 | 10 | **45** | Implement TOTP-based MFA using authenticator apps. Make MFA mandatory for admin accounts. Provide backup codes for account recovery. | Open |
| R14 | Overprivileged Database Account | API connects to MongoDB with account that has more permissions than needed. Compromise of API could lead to database-wide damage. | 7 | 8 | 6 | 10 | 6 | **37** | Create least-privilege database user for API with only necessary permissions (read/write on secrets collection only). Use separate admin account for maintenance. | Open |
| R15 | Mass Secret Creation | No limits on how many secrets users can create. Could fill storage or be used to hide malicious secrets among legitimate ones. | 5 | 10 | 10 | 8 | 9 | **42** | Implement storage quotas per user (e.g., 100 secrets per developer, 500 per devops). Add rate limiting on POST /items (10 creates per hour). | Open |
| R16 | Secrets Transmitted to Browser Unnecessarily | GET /items sends actual secret values to browser even when masked in UI. Browser memory, DevTools, and extensions can access plaintext secrets. | 9 | 10 | 10 | 10 | 8 | **47** | Modify GET /items to return only metadata (name, type, environment) without secret values. Only transmit secret values on explicit /reveal request after step-up auth. | Open |
| R17 | Database Not Logging Changes | MongoDB audit logging not enabled. Cannot track who modified or deleted data directly in database, only through API. | 6 | 8 | 7 | 8 | 5 | **34** | Enable MongoDB audit logging. Log all database operations including direct connections, administrative actions, and schema changes. | Open |
| R18 | Verbose Error Messages | Error messages may reveal system information like file paths, database details, or stack traces in production. | 5 | 8 | 9 | 10 | 10 | **42** | Implement generic error messages for users. Log detailed errors server-side only. Disable debug mode in production. Use custom error handlers. | Open |
| R19 | No Session Timeout | JWT tokens valid for 2 hours with no idle timeout. Unattended sessions remain active allowing unauthorized access. | 7 | 10 | 10 | 10 | 8 | **45** | Implement sliding session timeout (15 minutes idle). Force re-authentication after 2 hours regardless of activity. Add 'logout all devices' functionality. | Open |
| R20 | Database Backup Not Encrypted | Database backups stored without encryption. Backup compromise exposes all historical secrets. | 9 | 7 | 6 | 10 | 6 | **38** | Encrypt all database backups using AES-256. Store backup encryption keys separately from backup data. Implement secure backup retention and disposal policies. | Open |

---

## DREAD Scoring Legend

**D = Damage Potential** (0-10)
- How much damage would occur if exploited?
- 0 = No damage
- 10 = Complete system compromise

**R = Reproducibility** (0-10)
- How easy to reproduce the attack?
- 0 = Nearly impossible
- 10 = Works every time

**E = Exploitability** (0-10)
- How much skill/effort required?
- 0 = Nation-state level
- 10 = Anyone with basic knowledge

**A = Affected Users** (0-10)
- How many users impacted?
- 0 = No users
- 10 = All users

**D = Discoverability** (0-10)
- How easy to discover?
- 0 = Extremely difficult
- 10 = Obvious to anyone

**Total Score** = D + R + E + A + D (Maximum: 50)

---

## Risk Severity Classification

| Score Range | Severity | Action Required |
|-------------|----------|-----------------|
| 45-50 | **Critical** | Immediate action required (within 1 sprint) |
| 40-44 | **High** | Address within 1-2 sprints |
| 35-39 | **Medium** | Address within 2-3 sprints |
| 30-34 | **Low** | Ongoing improvement |
| Below 30 | **Minimal** | Monitor and review |

---

## Summary Statistics

**Total Risks:** 20

**By Severity:**
- Critical (45-50): 12 risks (60%)
- High (40-44): 4 risks (20%)
- Medium (35-39): 3 risks (15%)
- Low (30-34): 1 risk (5%)

**Average Risk Score:** 43.5/50 (87% - Very High Overall Risk)

**Top 5 Critical Risks:**
1. R2: IDOR on Secret Reveal (48)
2. R4: All Users Can View All Secrets (48)
3. R3: IDOR on Secret Revocation (47)
4. R5: No Rate Limiting on Login (47)
5. R7: Weak Password Policy (47)
