# PASTA Template (lightweight)

## 1. Objectives
- Provide developers and DevOps teams a centralized system to store and manage API keys, passwords, and credentials
- Reduce risk of secrets being stored in code repositories or shared insecurely via email/Slack
- Enable secure secret sharing across teams while maintaining audit trails
  
## 2. Technical scope
- React frontend (Vite/React client)
- Node/Express backend
- MongoDB database
- Authentication system (JWT-based)
- User roles (developer, devops, admin)

## 3. Decomposition
- External Entity: Users (Developer/DevOps/Admin)
- Client Trust Boundary: Browser (Vite/React)
- API Trust Boundary: Express API Server
- Database Trust Boundary: MongoDB

## 4. Threat analysis
**Information Disclosure (20+ threats) - HIGHEST PRIORITY**
- Secrets stored in plaintext in database
- All users can view all secrets regardless of ownership
- Secrets transmitted to browser unnecessarily
- JWT tokens stored in localStorage (accessible to XSS)

**Elevation of Privilege (15+ threats)**
- IDOR vulnerabilities on all /items endpoints
- No authorization checks for secret access, modification, or deletion
- Users can access/modify/delete secrets belonging to other users

**Spoofing (10+ threats)**
- Weak password requirements (only 8 characters, no complexity)
- No two-factor authentication
- Brute force attacks possible (no rate limiting)
- No account lockout mechanisms

**Denial of Service (10+ threats)**
- No rate limiting on any endpoints
- No pagination (could exhaust memory)
- Unlimited login attempts

**Tampering (Addressed)**
- Input validation prevents NoSQL injection (Zod schemas)
- XSS protection working (React escaping)

**Repudiation (5+ threats)**
- Incomplete audit logging
- Access logs not visible to users/admins

## 5. Vulnerability analysis
1. **Plaintext Secret Storage**
   - Secrets stored unencrypted in MongoDB
   - Database compromise = immediate secret exposure

2. **Broken Access Control - All Secrets Visible**
   - GET /items returns all secrets to any authenticated user
   - No filtering by ownership or permissions

3. **IDOR on Secret Operations**
   - GET /items/:id - view any secret by changing ID
   - POST /items/:id/reveal - reveal any secret
   - POST /items/:id/revoke - revoke any secret
   - No ownership verification

4. **Insecure Token Storage**
   - JWT stored in localStorage (not HttpOnly cookie)
   - Vulnerable to XSS token theft

5. **No Rate Limiting**
   - Unlimited login attempts (brute force possible)
   - Unlimited API requests (DoS possible)

6. **Weak Password Policy**
   - Only 8 characters required
   - No complexity requirements
   - Weak passwords like "password1" accepted

7. **Missing HTTP Security Headers**
   - No Content-Security-Policy
   - No X-Frame-Options
   - No HSTS

8. **No Step-Up Authentication**
   - Secret reveals don't require password re-entry
   - High-risk operations have same auth as low-risk

9. **Incomplete Audit Logging**
   - Access logs exist in database but not exposed
   - No logging for secret views/reveals

10. **No Two-Factor Authentication**
    - Single factor (password only)
    - Compromised password = complete access

## 6. Attack modeling (narratives)
- External malicious actor, intermediate technical skills
- Disgruntled employee (developer role), about to be fire
- External attacker, advanced skillsd
- actor using using leaked credentials
- developer accidentally discovers vulnerability

## 7. Risk & impact + mitigations
| Risk | Likelihood | Impact | Overall Risk | Priority |
|------|-----------|---------|--------------|----------|
| Plaintext secret storage | Medium | Critical | HIGH | 1 |
| IDOR vulnerabilities | High | Critical | HIGH | 2 |
| Broken access control (all users see all secrets) | High | Critical | HIGH | 3 |
| Insecure JWT storage (localStorage) | Medium | High | HIGH | 4 |
| No password rate limiting (brute force) | High | High | HIGH | 5 |
| Weak password policy | High | Medium | MEDIUM | 6 |
| No passwrod MFA | Medium | High | MEDIUM | 7 |
| Missing security headers | Medium | Medium | MEDIUM | 8 |
| No secret reveal authentication | Low | Medium | LOW | 9 |
