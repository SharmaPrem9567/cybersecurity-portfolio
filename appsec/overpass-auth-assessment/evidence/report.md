# Overpass --- Application Security Assessment

**Vulnerability Class:** Authentication Bypass (Broken Authentication)\
**Risk Rating:** Critical (CVSS 3.1: 9.8 --- AV:N / AC:L / PR:N / UI:N /
S:U / C:H / I:H / A:H)

> **Scope / Authorization**\
> Testing was conducted in a controlled, authorized lab environment.\
> No production systems, real data, or external services were accessed.

---

## Table of Contents

1.  Executive Summary\
2.  Application Overview\
3.  Threat Model\
4.  Attack Hypotheses\
5.  Vulnerability Analysis\
6.  Steps to Reproduce\
7.  Evidence Summary\
8.  Impact\
9.  Remediation\
10. Detection & Hardening\
11. Lessons Learned

---

## 1️⃣ Executive Summary

An assessment of the Overpass web application identified a **critical
authentication bypass** caused by trusting client-supplied
authentication cookies without server-side validation.

An unauthenticated attacker can supply an arbitrary cookie value and be
treated as a logged-in user, gaining access to sensitive resources and
administrative functions **without credentials**.

This flaw compromises the integrity of the authentication model and
enables account takeover, data exposure, and full application
compromise.

**Severity:** Critical\
**Business Risk:** Loss of confidentiality, loss of integrity,
regulatory exposure, reputational damage.\
**Exploitability:** Trivial --- no credentials or special tools
required.

---

## 2️⃣ Application Overview

Overpass is a self-hosted password-management application with a web UI
and backend service layer.

### Key Assets Protected

- Stored credentials and user secrets\
- Authenticated session state\
- Administrative functionality\
- Application configuration data

### Authentication Overview

- Custom cookie-based authentication\
- Token integrity not verified server-side\
- Authorization decisions derived from client data

The system **assumes** tokens are valid instead of **verifying** them.

---

## 3️⃣ Threat Model

**Attacker Profile** - External, unauthenticated user\

- No credentials or insider access\
- Basic HTTP interception capability

**Objective** - Access protected resources\

- Escalate privileges\
- Extract sensitive data

**Broken Assumptions** - Client-provided cookies are trustworthy\

- Tokens represent legitimate, authenticated sessions\
- Validation happens somewhere else (it does not)

Result: authentication integrity collapses.

---

## 4️⃣ Attack Hypotheses

1.  Custom token logic might allow forgery.\
2.  Server may trust any token that "looks" valid.\
3.  Session validation may not occur server-side.\
4.  Sensitive endpoints might be reachable without authentication.

These guided targeted testing on authentication and session handling.

---

## 5️⃣ Vulnerability Analysis

### Description

The application performs authentication using a cookie that indicates
session state.\
The server **does not verify**:

- who issued the token\
- whether it is valid\
- whether it belongs to the requesting user

Presence of the cookie alone triggers authenticated status.

### Root Cause

- Absence of cryptographic signing\
- Missing server-side validation\
- Authorization logic tied to attacker-controlled input

### Classification

- Identification & Authentication Failures (OWASP A07:2021)\
- Broken Authentication (OWASP ASVS V2 & V3)

---

## 6️⃣ Steps to Reproduce (Exact, Repeatable)

**Component:** Cookie-based session handling

1.  Navigate to the application as an unauthenticated user.\

2.  Intercept the request (Burp / browser dev tools).\

3.  Add a cookie named `session` (or modify the existing one) with any
    non-empty value:

        Cookie: SessionToken=admin

4.  Request a protected endpoint (e.g., `/admin` or `/secrets`).

**Observed Result:**\
The server treats the request as authenticated and grants access.

**Expected Result:**\
Server rejects the request and redirects to login.

No credentials, bruteforce, or payloads required.

---

## 7️⃣ Evidence Summary

- Screenshot: accessing administrative panel without login\
- Screenshot: viewing protected content with arbitrary cookie

(Sensitive values redacted.)

---

## 8️⃣ Impact

This vulnerability enables:

- **Account takeover** --- impersonation without credentials\
- **Data exposure** --- stored secrets and sensitive records\
- **Privilege escalation** --- administrative control\
- **Lateral compromise** --- chaining to other systems using leaked
  credentials

In production, this commonly leads to **reportable breaches**.

---

## 9️⃣ Remediation (Practical & Enforceable)

### Design Changes

- Replace custom authentication with a vetted framework (OIDC, OAuth2,
  or standard session libraries).
- Treat all client-supplied session tokens as untrusted.

### Technical Controls

- Validate session tokens server-side on every request\
- Sign tokens using HMAC or asymmetric keys\
- Bind sessions to user identity, device, and expiry\
- Invalidate tokens on logout / privilege change\
- Store secrets in environment variables or dedicated secrets vault

### Standards Mapping

- OWASP Top 10 --- Identification & Authentication Failures\
- OWASP ASVS
  - V2 --- Authentication\
  - V3 --- Session Management

This requires **architectural redesign**, not patching.

---

## 🔍 10️⃣ Detection & Hardening

**Monitoring** - Alert on admin access without prior login event\

- Detect malformed / identical tokens across users\
- Log and review rejected tokens vs. accepted tokens

**Preventive Safeguards** - Disable direct access to admin endpoints
from unauthenticated contexts\

- Add rate limits and anomaly detection on session creation\
- Enforce secure cookie flags (HttpOnly, Secure, SameSite)

---

## 1️⃣1️⃣ Lessons Learned

- Custom authentication mechanisms are inherently risky.\
- Authentication and session controls must be validated at design
  time.\
- Any token accepted without cryptographic verification is a breach
  waiting to happen.

**Summary:** authentication integrity must be enforced server-side ---
always.
