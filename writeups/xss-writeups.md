# XSS Exploitation Writeup — Day 7

## Overview

This assessment documents the identification and exploitation of Cross-Site Scripting (XSS) vulnerabilities across multiple contexts using intentionally vulnerable web application labs. The focus is on understanding data flow, execution context, and vulnerable client-side behavior rather than payload dumping.

The following XSS classes were successfully exploited:

- Reflected XSS
- Stored XSS
- DOM-based XSS (`document.write`)
- DOM-based XSS (`innerHTML`)

All findings were validated through observable JavaScript execution in the browser.

---

## 1. Reflected XSS — HTML Context

### Injection Point
```
GET /?search=<payload>
```

### Source
```
User-controlled search parameter (processed server-side)
```

### Sink
```
Reflected directly into HTML response without output encoding
```

### Payload Used
```html
<img src=x onerror=alert(1)>
```

### Why This Worked
- The server embedded untrusted user input directly into the HTML response.
- No context-aware output encoding was applied.
- The browser rendered the injected `<img>` element and executed the `onerror` handler.

### Result
JavaScript execution confirmed via browser alert upon page render.

---

## 2. Stored XSS — Persistent Injection

### Injection Point
```
POST /comment
```

### Source
```
Persistent comment input field
```

### Sink
```
Stored value rendered into HTML using unsafe output handling
```

### Payload Used
```html
<img src=x onerror="alert(1)">
```

### Why This Worked
- Malicious input was stored in the backend database without sanitization.
- Stored data was rendered for every user viewing the affected page.
- The browser executed the payload automatically on page load.

### Impact
- Persistent XSS affecting all visitors.
- Enables session hijacking, credential theft, and malicious page manipulation.

---

## 3. DOM XSS — document.write Sink

### Source
```javascript
location.search   // read directly by client-side JavaScript
```

### Sink
```javascript
document.write(location.search);
```

### Payload Used
```html
"><svg onload=alert(1)>
```

### Why This Worked
- Client-side JavaScript read attacker-controlled data directly from the URL.
- `document.write()` injected the data into the DOM as raw HTML.
- The browser parsed and executed the injected SVG `onload` handler.

### Key Observation
- The server never processed or reflected the payload.
- The vulnerability existed entirely in client-side JavaScript.

---

## 4. DOM XSS — innerHTML Sink

### Source
```javascript
location.search   // read directly by client-side JavaScript
```

### Sink
```javascript
element.innerHTML = location.search;
```

### Payload Used
```html
<svg onload=alert(1)>
```

### Why This Worked
- The application assigned attacker-controlled input directly to `innerHTML`.
- No sanitization or encoding was applied before DOM insertion.
- JavaScript executed immediately when the DOM was updated.

### Difference vs document.write
- `innerHTML`-based DOM XSS is more common in modern applications.
- Often harder to detect with traditional server-side scanning tools.

---

## Impact

- Execution of arbitrary JavaScript in victim browsers.
- Account takeover through session or token theft.
- Unauthorized actions performed on behalf of authenticated users.
- Persistent compromise of application trust boundaries.

---

## Mitigation

- Apply context-aware output encoding on all server-rendered content.
- Avoid dangerous DOM sinks such as `document.write()` and `innerHTML` for untrusted data.
- Use safe DOM APIs like `textContent` or `innerText`.
- Sanitize user input with trusted libraries such as DOMPurify.
- Implement Content Security Policy (CSP) to reduce exploitation impact.

---

## Final Assessment

This assessment demonstrated multiple XSS classes across both server-side and client-side execution paths. By validating reflected, stored, and DOM-based XSS through controlled payload execution, the findings highlight how improper input handling and unsafe DOM manipulation can lead to reliable, real-world exploitation.
