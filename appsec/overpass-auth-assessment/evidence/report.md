# Overpass --- Web Application Security Assessment

**Finding:** Authentication Bypass (Broken Authentication)\
**Risk:** Critical (CVSS 9.8)

> Testing was performed in a controlled lab environment.\
> No production systems or real user data were accessed.

---

## 1. Executive Summary

The application incorrectly treats any session cookie as valid
authentication.\
Because tokens are not verified server‑side, an attacker can access
administrative pages and stored secrets **without logging in**.

This weakness exposes credentials and configuration data and could
enable compromise of other systems that reuse those credentials. The
issue is simple to exploit and must be addressed immediately.

**Priority:** Immediate remediation\
**Business Impact:** Loss of confidentiality and integrity; potential
regulatory exposure\
**Likelihood:** High --- no credentials or specialized tools required

### Business Decision Summary

- **Risk:** Critical --- unauthorized admin access and credential
  exposure.\
- **Cost if ignored:** Possible lateral compromise, incident response
  costs, loss of trust, and potential compliance issues.\
- **Recommended action:** Replace custom auth or enforce verified
  server‑side sessions immediately, then monitor and retest after
  deployment.

---

## 2. Scope & Methodology

Testing followed the OWASP Web Security Testing Guide (WSTG):

- Information gathering and reconnaissance\
- Authentication and session testing\
- Authorization and access‑control validation\
- Input and logic testing\
- Verification and retest

**Tools:** Burp Suite (manual proxy), browser developer tools, raw HTTP
requests.

**Assumptions:**

- Application is running in a lab instance\
- Admin interface is expected to be protected\
- No WAF or reverse‑proxy controls were present

---

---

## 2.1 Environment & Testing Dates

**Platform:** TryHackMe (Overpass lab — simulated environment)

**Instance type:** Isolated training VM (no production systems)

**Tester:** Security assessment performed by security analyst

**Testing window:** 2025-12-15 (lab executed during dedicated learning session)

**Data sensitivity:** Non‑production, synthetic data only

> These details are included to maintain transparency and mirror real client assessment structure.

### 2.2 Scope Boundaries & Exclusions

**In-scope**

- Web application front-end

- Authentication and session behavior

- Authorization and administrative interfaces

- Read‑only observation of stored secrets

**Out of scope / not performed**

- Denial‑of‑Service testing

- Brute‑force or credential‑stuffing attacks

- Pivoting into external infrastructure

- Social engineering or phishing simulations

Documenting these constraints prevents misinterpretation of coverage and reflects real‑world engagements.

## 3. Application Overview

Overpass is a self-hosted password-management web application.

**Key Assets**

- stored secrets and credentials\
- authenticated session state\
- administrative functions\
- application configuration and keys

**Authentication Behavior**

- custom cookie-based authentication\
- token integrity is not verified server-side\
- authorization decisions depend on client-controlled values

The system assumes cookies are legitimate instead of validating them.

---

## 4. Threat Model

### Authentication and Trust Boundary Flow

**High‑Level Trust Flow Diagram**

```
Browser → Web Server → Auth Logic (broken) → Secrets DB
```

This makes it clear that authentication is trusted too early, and downstream components rely on that broken assumption.

**Trust boundary:** everything beyond the web server implicitly trusts
the cookie value.\
Because the cookie is attacker-controlled, authentication fails
application-wide.

---

## 5. Attack Hypotheses & Results

| Hypothesis                            | Result   | Evidence                         |
| ------------------------------------- | -------- | -------------------------------- |
| Forged cookie may be accepted         | Accepted | `session=abc123` authenticates   |
| Server validates sessions server‑side | Rejected | Any cookie grants access         |
| Access control tied to cookie role    | Accepted | `/admin/` accessible directly    |
| Logout invalidates session            | Rejected | Same cookie works after logout   |
| Sensitive data is isolated            | Accepted | Access to stored SSH private key |

These hypotheses guided testing rather than being written after the
fact.

---

## 6. Vulnerability Analysis

### Description

Authentication is implemented using a cookie labeled as a session token.
The application performs no validation of:

- the issuer of the token\
- token validity or expiry\
- ownership of the token by the requesting user

Presence of any cookie value flips the application into an authenticated
state.

### Exact Enforcement Failure

Authentication behaves as: **cookie exists → user is logged in.**\
The server does not maintain session state, making revocation impossible
and bypass trivial.

### Root Cause

- no cryptographic signing of tokens\
- no server-side session lookup\
- authorization logic trusts client-controlled input\
- logout does not invalidate a real session (because none exists)

**Classification:**\

- OWASP A07:2021 --- Identification & Authentication Failures\
- ASVS V2/V3\
- **CWE‑287: Improper Authentication**\
- **CWE‑306: Missing Authentication for Critical Function**

---

## 7. Affected Components

| Component         | Endpoint   | Impact                              |
| ----------------- | ---------- | ----------------------------------- |
| Admin panel       | `/admin`   | Full administrative access          |
| Secrets storage   | `/secrets` | Disclosure of sensitive credentials |
| Session lifecycle | `/logout`  | Session remains valid after logout  |

---

## 8. Steps to Reproduce (Repeatable Proof)

### Raw HTTP Example

    GET /admin/ HTTP/1.1
    Host: overpass.local
    Cookie: session=abc123

**Observed:** request succeeds (authenticated context)\
**Expected:** HTTP 401 or redirect to login

### Results Matrix

| Cookie Scenario | Endpoint | Expected | Actual                       |
| --------------- | -------- | -------- | ---------------------------- |
| (none)          | /admin/  | Blocked  | Blocked                      |
| session=abc123  | /admin/  | Blocked  | Allowed                      |
| session=abc123  | /secrets | Blocked  | Allowed                      |
| blank cookie    | /admin/  | Blocked  | Not tested                   |
| after logout    | /admin/  | Blocked  | Allowed (logout ineffective) |

---

## 9. Exploit Chain (Realistic Risk Path)

1.  Attacker fabricates cookie\
2.  Gains admin access\
3.  Extracts secrets / SSH key\
4.  Uses secrets to pivot to other systems **(projected — not executed in lab)**\
5.  Maintains persistence via configuration changes

Result: wider infrastructure compromise is **plausible (projected)** based on exposed credentials.

---

## 10. Validation and Retest (What "fixed" should look like)

After server-side validation is implemented:

- forged cookie → 401\
- logout → session invalidated\
- reused token → rejected\
- admin pages → inaccessible without valid session

In production, retest should confirm all outcomes above.

---

## 11. Impact (Real-World Abuse Paths)

- attacker fabricates cookie → gains admin access\
- retrieves stored credentials and keys\
- reuses secrets against other systems **(projected)**\
- modifies configuration to maintain persistence\
- operators lose the ability to revoke access

Bottom line: authentication cannot be trusted and application integrity
collapses.

---

## 12. Remediation

### Replace custom authentication or implement proper sessions

- server-side session store (database or Redis)\
- cryptographically signed tokens (HMAC/JWT with verification)\
- bind session to user identity and expiry\
- invalidate sessions on logout\
- decouple authorization decisions from cookie contents

### Temporary risk reductions (until fixed)

- restrict admin interface by IP\
- add WAF rule blocking unauthenticated admin requests\
- monitor anomalous cookie usage patterns

---

## 13. CVSS Justification

- AV:N --- remotely exploitable\
- AC:L --- trivial to perform\
- PR:N --- no credentials required\
- UI:N --- no victim interaction\
- C/I/A:H --- full administrative control\
- S:U --- impact limited to the application boundary

**Score: 9.8 (Critical)**

---

## 14. Detection and Logging

Log per request:

- user or anonymous ID\
- session hash\
- decision (ALLOW/DENY)\
- endpoint\
- IP and User-Agent

Alert on:

- ALLOW without a corresponding session lookup\
- the same session hash across multiple IP addresses\
- post-logout session reuse

---

---

## 15. False Positives & Limitations

- Findings were validated manually and confirmed reproducible.

- Because the environment is a training lab, real third‑party integrations were not present —
  some downstream risks are therefore **projected** rather than demonstrated.

- Session‑related behavior was tested without load or concurrency; race‑condition defects may remain unobserved.

- No credential brute‑forcing or denial‑of‑service testing was executed due to scope limits.

Clear disclosure of limitations prevents over‑ or under‑estimating risk and mirrors real consulting expectations.

## 16. Lessons Learned

Custom authentication is fragile and should be avoided when possible.\
All authentication must be validated on the server for every request.

---

## Appendix A — Evidence & Artifacts

### A.1 Screenshots

- Admin access granted with fabricated cookie (`/admin`)

- Secrets page displaying protected credentials (`/secrets`)

- Post‑logout request still allowed

_(Screenshots intentionally omitted here. In a real engagement, each screenshot would include timestamp, scope ID, and request correlation so developers can reproduce exactly.)_

### A.2 Burp / HTTP Artifacts

```
GET /admin/ HTTP/1.1
Host: overpass.local
Cookie: session=abc123
```

```
GET /secrets HTTP/1.1
Host: overpass.local
Cookie: session=abc123
```

Each artifact above is tied to findings and supports repeatable verification.
