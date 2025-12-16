# CTF Walkthrough – VulnNet: dotpy

## 1. Overview

This writeup documents the exploitation of the VulnNet: dotpy machine.
The target exposed a Python Flask web application vulnerable to server-side template injection.
Successful exploitation resulted in remote code execution and elevated access within the system.

## 2. Target Information

- Target: VulnNet: dotpy
- Entry Point: User-controlled HTTP request parameter modified via Burp Suite
- Attack Surface: Flask-based HTTP service running on port 8080

## 3. Enumeration

- Tool: Nmap
- Command: nmap -sC -sV -p 8080 <IP>
- Key Finding: HTTP service identified as Werkzeug / Flask, indicating a Python-based web application

Manual enumeration revealed identical dynamic responses across multiple
endpoints when user input was modified, suggesting server-side template
processing rather than static content rendering.

## 4. Exploitation

- Vulnerability: Server-Side Template Injection (SSTI) in Jinja2

During testing, user-controlled input was reflected in server responses
and evaluated dynamically, indicating server-side template rendering.

To confirm SSTI, arithmetic template expressions were injected and
successfully evaluated by the application, confirming template execution
rather than simple string reflection.

After confirmation, Jinja2 object traversal techniques were used to
access Python built-in functions and the underlying operating system
interface.

- Why it Worked:
  The Flask application rendered unsanitized user input directly within
  a Jinja2 template context, allowing template expression evaluation.

- Result:
  Arbitrary system command execution was achieved, resulting in a reverse
  shell as the application user and enabling further post-exploitation.

  Tooling used: Burp Suite (request interception and modification)

## 5. Impact

Successful exploitation allowed arbitrary command execution on the
target system. This could lead to full system compromise, sensitive
data exposure, lateral movement, and persistence if exploited in a
real-world environment. Misconfigured sudo permissions further increased
the severity by enabling privilege escalation.

## 6. Post-Exploitation Notes

- Initial Shell Context:
  Remote code execution provided a shell running as the web application
  user with limited privileges.

- Privilege Escalation:
  Enumeration revealed misconfigured sudo permissions, allowing execution
  of privileged commands without proper authentication, leading to
  elevated access.

- Key Lesson Learned:
  Server-side template injection vulnerabilities can result in complete
  system compromise when combined with poor privilege separation and
  misconfigured sudo rules.

## 7. Mitigation

- Avoid rendering unsanitized user input within templates
- Apply strict input validation and proper context escaping
- Restrict sudo privileges and remove NOPASSWD rules for system utilities
- Enforce the principle of least privilege for application users

## 8. Conclusion

The VulnNet: dotpy machine demonstrated how SSTI vulnerabilities in Python web applications can lead to full system compromise. Manual request manipulation and understanding of template engines were critical to successful exploitation.
