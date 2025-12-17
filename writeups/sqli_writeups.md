# SQL Injection Assessment – Writeup

## 1. Overview

This assessment focused on identifying and exploiting SQL Injection vulnerabilities across intentionally vulnerable web applications. Multiple PortSwigger SQL Injection labs were completed, covering WHERE clause manipulation, authentication bypass, UNION-based data extraction, and database fingerprinting. In addition, the TryHackMe _Advanced SQL Injection_ room was used to validate advanced scenarios such as filter evasion, stored/second-order SQL injection, out-of-band (OOB) data exfiltration, and controlled automation using sqlmap. The objective was to demonstrate practical exploitation workflow and attacker decision-making rather than theoretical understanding.

---

## 2. Target Description

The assessment was conducted against intentionally vulnerable web applications provided by PortSwigger SQL Injection labs and the TryHackMe _Advanced SQL Injection_ room.

Manual testing was performed using browser-based interaction and Burp Suite to identify injection points. Vulnerabilities were discovered in both GET- and POST-based parameters where user-controlled input was directly concatenated into backend SQL queries without effective server-side sanitization.

Identified attack surfaces included search and filtering functionality, authentication workflows, and database-driven update operations, enabling exploitation through Boolean-based, UNION-based, stored, and out-of-band SQL injection techniques.

---

## 3. Vulnerability Analysis

### 3.1 SQL Injection — WHERE Clause Manipulation

**Injection Point:**  
`GET /filter?category=Gifts`

**Payload Used:**

```sql
Gifts' OR 1=1--
```

**Why it worked:**

- The injected condition forced the WHERE clause to evaluate as TRUE.
- The SQL comment operator truncated the remaining query logic.
- Application logic enforcing visibility constraints (e.g., `released = 1`) was bypassed.

**Result:**  
Hidden and unreleased products were returned in the application response.

**Evidence:**  
Screenshot confirming the display of hidden products.

---

### 3.2 Authentication Bypass via SQL Injection

**Injection Point:**  
`POST /login` (username parameter)

**Payload Used:**

```sql
administrator' OR 1=1--
```

**Why it worked:**

- Authentication logic relied on unsanitized dynamic SQL.
- Password verification was bypassed by logical condition manipulation.
- The database returned the first matching user record.

**Impact:**  
Unauthorized administrative access without valid credentials.

---

### 3.3 UNION-Based SQL Injection

**Techniques Used:**

- `ORDER BY` enumeration to determine column count
- `UNION SELECT NULL` testing
- Identification of string-compatible columns

**Final Payload:**

```sql
' UNION SELECT NULL,'T7yGh9',NULL--
```

**Result:**  
Attacker-controlled data was reflected in the response, confirming full UNION-based data extraction capability.

---

### 3.4 Stored / Second-Order SQL Injection

**Injection Scenario:**  
User-supplied input was stored in the database through an update or insert operation and later reused in a separate SQL query without proper sanitization.

**Why it worked:**

- The initial request did not immediately trigger SQL execution.
- The injected payload was persisted in the database.
- Execution occurred when the stored value was later consumed by another query in a different context.

**Impact:**

- SQL injection executed indirectly at a later stage.
- Demonstrates risk beyond direct input-to-query injection.
- Highlights insufficient sanitization of stored data.

### 3.5 Advanced SQL Injection Validation

Advanced SQL injection techniques were validated in controlled lab environments as part of the TryHackMe _Advanced SQL Injection_ room.

This included:

- **Filter evasion**, where common SQL keywords were blocked and bypassed using case manipulation, inline comments, and encoding techniques.
- **Second-order SQL injection**, where injected input was stored in the database and later executed when reused in a different SQL context.
- **Out-of-band (OOB) SQL injection**, where direct query output was unavailable and data exfiltration was confirmed through database-triggered external callbacks.

These scenarios were validated through observable application behavior and external interaction rather than direct in-band query responses, reflecting real-world conditions where error messages and output are restricted.

**Key takeaway:**  
Advanced SQL injection relies on understanding data flow and execution context rather than payload memorization.

---

## 4. SQLmap Exploitation

Manual validation was performed prior to automation.

**Confirmed Injectable Parameter:**

```text
id=1
```

**Database Enumeration:**

```bash
sqlmap -u "http://127.0.0.1/vulnerable.php?id=1" --batch --dbs
```

**Result:**  
Databases were successfully enumerated.

**Table Extraction:**

```bash
sqlmap -u "http://127.0.0.1/vulnerable.php?id=1" -D dvwa -T users --dump
```

**Outcome:**  
User credentials were extracted, confirming full database compromise following successful manual exploitation.

---

## 5. Impact

- Unauthorized access to sensitive application data, including hidden and unreleased records.
- Authentication bypass via SQL injection in the login workflow, resulting in unauthorized administrative access.
- Ability to enumerate databases and extract user credentials following manual validation, confirming the potential for full database compromise.
- In a real-world production environment, these vulnerabilities could lead to data exfiltration, account takeover, and complete application compromise.

---

## 6. Mitigation

- Use prepared statements and parameterized queries to prevent user input from being interpreted as executable SQL.
- Enforce strict server-side input validation and avoid reliance on client-side encoding or filtering.
- Apply least-privilege principles to database accounts to reduce impact.
- Sanitize and validate stored data before reuse in subsequent queries to prevent second-order SQL injection.
- Monitor and alert on abnormal query patterns indicative of SQL injection attempts.
