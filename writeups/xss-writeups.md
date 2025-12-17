# Cross-Site Scripting (XSS) Assessment – Writeup

## Overview

This assessment documents the identification and exploitation of Cross-Site Scripting (XSS) vulnerabilities across multiple intentionally vulnerable web application labs. The work covers **Reflected XSS**, **Stored XSS**, and **DOM-based XSS**, with emphasis on understanding vulnerable sinks, execution context, and why specific payloads succeeded. The objective was to demonstrate exploitation reasoning and browser-side behavior rather than payload dumping.

---

## 1. Reflected XSS

### Vulnerability Summary

The application reflected user-controlled input directly into the HTML response without proper output encoding. This allowed injected JavaScript to execute in the victim’s browser when a crafted URL was accessed.

### Injection Point / Sink

- URL parameter reflected into the HTML response
- HTML attribute / tag context
- No server-side encoding applied

### Payload Used

```html
"><script>alert(1)</script>
```

### Why It Executed

- The payload broke out of the existing HTML context using a quote.
- A new `<script>` tag was injected and executed by the browser.
- The application failed to encode special characters before rendering.

### Result

JavaScript execution was confirmed via a browser alert on page load.

---

## 2. Stored XSS

### Vulnerability Summary

User input submitted through a persistent field was stored in the backend database and rendered to all users without sanitization. This caused script execution whenever the affected page was viewed.

### Injection Point / Sink

- Persistent input field (e.g., comments or messages)
- Stored value rendered directly into HTML output

### Payload Used

```html
<img src=x onerror=alert(1)>
```

### Why It Executed

- The payload was stored verbatim in the database.
- When rendered, the browser attempted to load an invalid image source.
- The `onerror` event handler executed attacker-controlled JavaScript.

### Result

JavaScript executed automatically on page load for any user viewing the affected content.

---

## 3. DOM-Based XSS (document.write Sink)

### Vulnerability Summary

Client-side JavaScript used `document.write()` with data sourced from `location.search`, allowing attacker-controlled input to be written directly into the DOM.

### Injection Point / Sink

- Source: `location.search`
- Sink: `document.write()`
- No client-side validation or sanitization

### Payload Used

```html
"><svg/onload=alert(1)>
```

### Why It Executed

- `document.write()` inserted raw HTML into the DOM.
- The injected SVG element executed immediately via the `onload` handler.
- No server-side interaction was required.

### Result

JavaScript execution occurred as soon as the page loaded with the crafted URL.

---

## 4. DOM-Based XSS (innerHTML Sink)

### Vulnerability Summary

The application assigned user-controlled input from the URL directly to `.innerHTML`, allowing HTML and JavaScript injection into the DOM.

### Injection Point / Sink

- Source: `location.search`
- Sink: `.innerHTML` assignment

### Payload Used

```html
" onmouseover=alert(1) a="
```

### Why It Executed

- The payload broke out of the existing attribute context.
- A new event handler was injected into the DOM.
- JavaScript executed upon user interaction (mouseover).

### Result

JavaScript execution was confirmed when the user interacted with the injected element.

---

## Impact

- Execution of arbitrary JavaScript in victim browsers
- Session hijacking and credential theft potential
- Persistent compromise of application users (Stored XSS)
- Complete client-side trust boundary violation

---

## Mitigation

- Encode output based on context (HTML, attribute, JavaScript).
- Avoid dangerous sinks such as `document.write()` and `.innerHTML` for untrusted data.
- Use safe DOM APIs like `textContent` or `innerText`.
- Sanitize user input using trusted libraries (e.g., DOMPurify).
- Enforce Content Security Policy (CSP) to reduce script execution impact.

---

## Conclusion

This assessment demonstrated multiple XSS classes across server-side and client-side contexts. By focusing on execution context and vulnerable sinks rather than payload volume, the findings highlight how improper input handling and unsafe DOM manipulation lead to reliable XSS exploitation in real-world applications.
