# CTF Walkthrough — VulnNet: dotpy

## 1. Overview

This writeup documents the end-to-end exploitation of the **VulnNet: dotpy** machine. The assessment focused on identifying a vulnerable Python Flask web application, confirming server-side template injection (SSTI), and leveraging it to achieve remote code execution and privilege escalation. The goal was to demonstrate a clear attack chain rather than isolated vulnerabilities.

---

## 2. Reconnaissance

### Tool Used
- **Nmap**

### Why This Tool
Initial reconnaissance was required to identify exposed services and technologies running on the target.

### Command
```bash
nmap -sC -sV -p 8080 <IP>
```

### Findings
- Port **8080** open
- HTTP service identified as **Werkzeug / Flask**
- Indicates a Python-based web application

This information suggested potential exposure to Python-specific vulnerabilities such as SSTI.

---

## 3. Enumeration

### Method
- Manual web application testing
- Request interception and modification using **Burp Suite**

### Why This Step
To understand how user input was processed and whether it was reflected, evaluated, or sanitized.

### Observations
- User-supplied input was reflected in server responses
- Similar behavior observed across multiple endpoints
- Responses appeared dynamically generated rather than static

These indicators suggested server-side template rendering of user-controlled input.

---

## 4. Exploitation

### Vulnerability Identified
- **Server-Side Template Injection (SSTI)** in **Jinja2**

### Confirmation Technique
Arithmetic expressions were injected into user-controllable parameters and evaluated by the server, confirming template execution instead of simple string reflection.

### Why It Worked
- Unsanitized user input was rendered directly inside a Jinja2 template
- No input validation or context escaping was applied
- Flask evaluated attacker-supplied template expressions on the server

### Exploitation Outcome
- Access to Python object traversal via Jinja2
- Ability to reach Python built-ins and OS interfaces
- **Arbitrary system command execution achieved**
- Reverse shell obtained as the web application user

### Tooling Used
- **Burp Suite** (request interception and payload manipulation)

---

## 5. Post-Exploitation

### Initial Access
- Shell obtained with limited privileges as the web application user

### Privilege Escalation
- Local enumeration revealed **misconfigured sudo permissions**
- Certain commands could be executed with elevated privileges without proper authentication
- Privilege escalation performed successfully

### Result
- Elevated access on the target system
- Full compromise achieved

---

## 6. Impact

- Arbitrary remote code execution via SSTI
- Full system compromise through privilege escalation
- Potential exposure of sensitive data
- Possibility of persistence, lateral movement, and further network compromise in real-world environments

---

## 7. Mitigation

- Never render untrusted user input directly in templates
- Apply strict input validation and context-aware escaping
- Disable or restrict dangerous template functionality
- Remove misconfigured sudo rules and enforce least privilege
- Regularly audit application and system-level permissions

---

## 8. Conclusion

The VulnNet: dotpy machine highlights how server-side template injection in Python Flask applications can lead to complete system compromise when combined with poor privilege separation. A structured attack chain — from reconnaissance to exploitation and escalation — was essential in achieving full control of the target.
