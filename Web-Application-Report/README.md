# Web Application Penetration Testing Project

This repository documents a **security assessment of a deliberately vulnerable web application (OWASP Juice Shop lab environment)** conducted for learning and practical skill development.

---

## Assessment Scope

- Authenticated testing  
- Black-box methodology  
- Web application + REST APIs  
- Focus on access control, authentication, and input handling weaknesses  

---

## Key Security Findings

| Category | Vulnerability | Risk |
|---------|---------------|------|
| Authentication | Missing Rate Limiting | Enables brute-force attacks |
| User Management | User Enumeration | Allows account discovery |
| Access Control | Insecure Direct Object References (IDOR) | Unauthorized data access |
| Session Security | Session Invalidation Failure | Session reuse after logout |
| Server-Side | Server-Side Request Forgery (SSRF) | Internal resource access |
| Error Handling | Stack Trace Disclosure | Information leakage |

---

## Attack Chain Demonstrated

This project shows how individual low-level issues can be chained:

**User Enumeration → Brute Force → Account Compromise → IDOR Exploitation → Session Persistence Abuse → SSRF**

This demonstrates real-world attacker methodology rather than isolated vulnerability testing.

---

## Report

The full structured penetration test report is available in the `/report` directory.

---

## Evidence

Screenshots and proof-of-concept artifacts are stored in the `/evidence` folder, organized by finding.

---

## Disclaimer

This testing was performed **only in a controlled lab environment**. No real systems were targeted.
