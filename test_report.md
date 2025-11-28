# Booking System – Phase 1  
## Penetration Testing Report (Registration Page)

---

# 1️⃣ Introduction

## Tester(s):
**Name:** Majeed Babatunde  

## Purpose:
To identify security vulnerabilities in the **user registration functionality** of the Booking System Phase 1 application, focusing on input validation, business logic, and server-side protections. Both manual and automated (OWASP ZAP) testing methods were used.

## Scope:
### Tested components:
- Registration page (`/register`)
- Registration form fields: username, email, password, age
- Server responses
- Basic security headers
- Input validation (client-side & server-side)
- Behavior under invalid or malicious input

### Exclusions:
- Login functionality  
- Reservation system  
- Admin features  
- Phase 2 improvements (not released yet)  

### Test Approach:
**Gray-box testing**  
Rationale: Application code not available, but controlled environment and system behavior known.

### Test Environment & Dates:
**Start:** 25.11.2025  
**End:** 28.11.2025  

**Environment Details:**
- OS: Kali Linux (VirtualBox)  
- Runtime: Local Docker container (Phase 1 Review Environment)  
- Browser: Firefox (ZAP proxy)  
- Tools: OWASP ZAP, manual testing, browser DevTools  
- Database: PostgreSQL (Docker container)

### Assumptions & Constraints:
- Only Phase 1 is available (Phase 2 comparison pending)  
- Limited to registration page  
- No credentials required  
- Local safe-to-test environment  
- Only allowed vulnerability scanning (no DoS, no destructive tests)

---

# 2️⃣ Executive Summary

### Short Summary:
Testing revealed **multiple critical vulnerabilities** in the registration page, including SQL Injection, improper input validation, missing security headers, and broken business logic (e.g., underage registration). The system is **not secure** in its current form.

### Overall Risk Level:
🟥 **Critical**

### Top 5 Immediate Actions:
1. Implement server-side input validation (strict allow-list).  
2. Fix SQL Injection by using parameterized queries.  
3. Enforce a strong password policy.  
4. Add essential security headers (CSP, X-Frame-Options, X-Content-Type-Options).  
5. Validate business logic (e.g., age must be ≥ 15, unique emails required).

---

# 3️⃣ Severity Scale & Definitions

| Severity | Description | Recommended Action |
|---------|-------------|--------------------|
| 🔴 High | Serious vulnerability allowing full system compromise (SQLi, Path Traversal) | Immediate fix required |
| 🟠 Medium | Significant issue (missing CSP, clickjacking risk) | Fix ASAP |
| 🟡 Low | Minor issue or configuration weakness | Fix soon |
| 🔵 Info | No direct risk but useful for hardening | Monitor / maintenance |

---

# 4️⃣ Key Findings (Top 5)

Below are the most important vulnerabilities discovered from manual + ZAP testing.

| ID | Severity | Finding | Description | Evidence / Proof |
|----|----------|----------|-------------|------------------|
| **F-01** | 🔴 High | SQL Injection in registration | `username` parameter vulnerable to boolean-based SQLi (`OR 1=1 --`) | ZAP evidence + manual payload success |
| **F-02** | 🔴 High | Path Traversal-like behavior | Application reacts unexpectedly to path-style inputs (`../`, `/register`) | ZAP alerts (3 instances) |
| **F-03** | 🟠 Medium | Missing security headers | No CSP, X-Frame-Options, or X-Content-Type-Options detected | Network tab & ZAP results |
| **F-04** | 🟠 Medium | Broken business logic (underage registration allowed) | Users under 15 can register; invalid ages accepted | Manual input testing |
| **F-05** | 🟡 Low | Weak password handling | Passwords like `1`, `aaa`, `<script>` accepted | Successful registration with weak passwords |

---

# 5️⃣ OWASP ZAP Test Report (Attachment)

### Purpose:
Automated scanning using OWASP ZAP was performed to identify additional vulnerabilities, confirm manual findings, and detect missing headers or injection issues.

### Attached Report:
**File:** `zap-report.md`  
**Contents:**  
- 2 High-risk alerts  
- 2 Medium alerts  
- 1 Low alert  
- 1 Informational alert  
- Detailed breakdown of SQL Injection, Path Traversal, and missing headers

ZAP by Checkmarx Scanning Report Summary of Alerts Risk Level Number of Alerts High 2 Medium 2 Low 1 Informational 1 Alerts Name Risk Level Number of Instances Path Traversal High 3 SQL Injection High 2 Content Security Policy (CSP) Header Not Set Medium 1 Missing Anti-clickjacking Header Medium 1 X-Content-Type-Options Header Missing Low 3 User Agent Fuzzer Informational 12 Alert Details Path Traversal Risk Level: High Description: Allows an attacker to access files, directories, and commands outside the web document root directory. Attackers manipulate URLs to reveal arbitrary files or execute commands. Affected URL(s): http://0.0.0.0:8000/register (POST - Parameter: username - 3 Instances) Solution: Implement strict input validation using an allow list, restrict access to known files, and use built-in path canonicalization functions. SQL Injection Risk Level: High Description: Allows an attacker to manipulate database queries through improperly validated input. Affected URL(s): http://0.0.0.0:8000/register (POST - Parameter: username - 2 Instances) Solution: Use prepared statements, avoid dynamic SQL concatenation, escape user input, and apply the principle of least privilege for database access. Content Security Policy (CSP) Header Not Set Risk Level: Medium Description: Missing CSP header allows attackers to execute malicious scripts (XSS) or data injection attacks. Affected URL(s): http://0.0.0.0:8000/register (GET) Solution: Configure the web server to include the Content-Security-Policy header. Missing Anti-clickjacking Header Risk Level: Medium Description: The absence of X-Frame-Options allows clickjacking attacks. Affected URL(s): http://0.0.0.0:8000/register (GET - Parameter: x-frame-options) Solution: Set X-Frame-Options to SAMEORIGIN or DENY, or use CSP’s frame-ancestors directive. X-Content-Type-Options Header Missing Risk Level: Low Description: Missing X-Content-Type-Options: nosniff allows MIME-type sniffing vulnerabilities in older browsers. Affected URL(s): http://0.0.0.0:8000/register (GET, POST - Parameter: x-content-type-options - 3 Instances) Solution: Set the X-Content-Type-Options header to nosniff. User Agent Fuzzer Risk Level: Informational Description: Detects differences in response based on User-Agent headers. Affected URL(s): http://0.0.0.0:8000/register (POST - Parameter: Header User-Agent - 12 Instances) Solution: Ensure consistent response handling regardless of the User-Agent. Conclusion This ZAP scanning report highlights multiple security vulnerabilities, including critical risks like SQL Injection and Path Traversal. Immediate remediation is recommended, particularly implementing proper input validation, enforcing security headers, and ensuring the application does not expose unnecessary file system access. Strengthening security controls can significantly reduce potential attack vectors.

