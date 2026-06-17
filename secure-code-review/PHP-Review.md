
# PHP Secure Code Review Notes

## Overview

This document summarizes common security issues identified during PHP source code reviews, vulnerability research, and application security assessments.

The objective is to help developers identify insecure coding patterns and adopt secure development practices.

---

# Review Scope

Typical review areas include:

* Authentication
* Authorization
* Input Validation
* Database Security
* File Upload Security
* Session Management
* Error Handling
* Sensitive Data Protection

---

# Finding Category 1: SQL Injection

## Vulnerable Pattern

```php
$id = $_GET['id'];

$query = "SELECT * FROM users WHERE id='$id'";

mysqli_query($conn, $query);
```

## Security Risk

User-controlled input becomes part of the SQL statement.

Potential impact:

* Authentication bypass
* Information disclosure
* Data manipulation
* Database compromise

## Secure Implementation

```php
$stmt = $conn->prepare(
"SELECT * FROM users WHERE id=?"
);

$stmt->bind_param(
"i",
$id
);

$stmt->execute();
```

---

# Finding Category 2: Cross-Site Scripting (XSS)

## Vulnerable Pattern

```php
echo $_GET['name'];
```

## Security Risk

Untrusted user input is rendered directly in HTML.

Potential impact:

* Session theft
* Account takeover
* Client-side attacks

## Secure Implementation

```php
echo htmlspecialchars(
$_GET['name'],
ENT_QUOTES,
'UTF-8'
);
```

---

# Finding Category 3: Insecure File Upload

## Vulnerable Pattern

```php
move_uploaded_file(
$_FILES['file']['tmp_name'],
"uploads/" .
$_FILES['file']['name']
);
```

## Security Risk

Attackers may upload:

* PHP webshells
* Malicious scripts
* Executable content

## Secure Implementation

Validate:

* File type
* File extension
* MIME type
* File size

Store uploads outside the web root whenever possible.

---

# Finding Category 4: Missing Authorization Checks

## Vulnerable Pattern

```php
$userId = $_GET['id'];

showProfile($userId);
```

## Security Risk

May result in:

* IDOR
* Unauthorized access
* Privacy violations

## Secure Implementation

```php
if (
$userId !== $_SESSION['user_id']
) {
    http_response_code(403);
    exit();
}
```

---

# Finding Category 5: Weak Session Management

## Vulnerable Pattern

```php
session_start();
```

without additional controls.

## Security Risk

* Session fixation
* Session hijacking

## Secure Implementation

```php
session_regenerate_id(true);
```

Additional protections:

* Secure cookies
* HttpOnly cookies
* SameSite cookies

---

# Finding Category 6: Sensitive Information Disclosure

## Vulnerable Pattern

```php
ini_set(
'display_errors',
1
);
```

## Security Risk

Error messages may reveal:

* SQL queries
* File paths
* Internal logic
* Configuration details

## Secure Implementation

```ini
display_errors = Off
```

Log errors internally.

---

# Finding Category 7: Weak Password Storage

## Vulnerable Pattern

```php
$password = md5($password);
```

## Security Risk

Weak hashing algorithms are vulnerable to cracking.

## Secure Implementation

```php
$passwordHash =
password_hash(
$password,
PASSWORD_DEFAULT
);
```

Verification:

```php
password_verify(
$password,
$passwordHash
);
```

---

# Finding Category 8: Command Injection

## Vulnerable Pattern

```php
system(
"ping " .
$_GET['host']
);
```

## Security Risk

Attackers may execute arbitrary operating system commands.

## Secure Implementation

Avoid shell execution whenever possible.

Use:

* Allowlist validation
* Escaping functions
* Native APIs

---

# PHP Secure Development Checklist

## Input Validation

* Validate all input
* Apply type checking
* Enforce length restrictions
* Use allowlists

## Database Security

* Use prepared statements
* Avoid dynamic SQL
* Apply least privilege

## Authentication

* Enforce strong passwords
* Use MFA where possible
* Secure password storage

## Authorization

* Verify ownership
* Enforce role checks
* Protect administrative actions

## Session Security

* Regenerate session IDs
* Use secure cookies
* Set HttpOnly and SameSite flags

## Error Handling

* Disable detailed errors
* Log internally
* Avoid information disclosure

---

# Common Review Methodology

During code review:

1. Identify user-controlled input.
2. Trace data flow.
3. Locate sensitive sinks.
4. Verify validation and authorization.
5. Assess impact.
6. Recommend remediation.

---

# Lessons Learned

Most PHP security issues arise from:

* Unsafe input handling
* Dynamic SQL construction
* Missing authorization checks
* Excessive trust in client input
* Weak security configuration

Secure coding practices and regular code reviews significantly reduce the likelihood of these vulnerabilities.

---

# References

* OWASP Top 10
* OWASP PHP Security Cheat Sheet
* PHP Security Guide
* CWE Top 25 Most Dangerous Software Weaknesses
* NIST Secure Software Development Framework (SSDF)

---

# Disclaimer

This document is intended for secure code review, defensive security research, software security education, and vulnerability remediation purposes only.
