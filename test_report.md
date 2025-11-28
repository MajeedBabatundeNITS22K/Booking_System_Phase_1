
## Penetration Testing Report  
---
# 1. Introduction

This report documents the penetration testing performed on the **Phase 1 Booking System registration page**.  
The goal was to identify vulnerabilities related to:

- Input validation  
- Authentication and registration logic  
- Session handling  
- Error messaging  
- Security headers  
- Data processing (e.g., age validation, email validation)

**Tools Used:**  
- OWASP ZAP (Passive + Active Scan)  
- Manual testing (browser input manipulation, SQL payloads, logic testing)  
- Docker local environment (Phase 1 deployment)  

# 2. Manual Testing

Manual penetration testing was performed before running ZAP. Several significant vulnerabilities were discovered

# 3. ZAP by Checkmarx Scanning Report

## Summary of Alerts

| Risk Level       | Number of Alerts |
|-----------------|-----------------|
| High           | 2               |
| Medium         | 2               |
| Low            | 1               |
| Informational  | 1               |

## Alerts

| Name                                         | Risk Level    | Number of Instances |
|---------------------------------------------|--------------|--------------------|
| Path Traversal                              | High         | 3                  |
| SQL Injection                               | High         | 2                  |
| Content Security Policy (CSP) Header Not Set | Medium       | 1                  |
| Missing Anti-clickjacking Header           | Medium       | 1                  |
| X-Content-Type-Options Header Missing       | Low          | 3                  |
| User Agent Fuzzer                           | Informational| 12                 |

## Alert Details

### Path Traversal
*Risk Level:* High  
*Description:* Allows an attacker to access files, directories, and commands outside the web document root directory. Attackers manipulate URLs to reveal arbitrary files or execute commands.  
*Affected URL(s):*
- http://0.0.0.0:8000/register (POST - Parameter: username - 3 Instances)
*Solution:* Implement strict input validation using an allow list, restrict access to known files, and use built-in path canonicalization functions.

### SQL Injection
*Risk Level:* High  
*Description:* Allows an attacker to manipulate database queries through improperly validated input.  
*Affected URL(s):*
- http://0.0.0.0:8000/register (POST - Parameter: username - 2 Instances)
*Solution:* Use prepared statements, avoid dynamic SQL concatenation, escape user input, and apply the principle of least privilege for database access.

### Content Security Policy (CSP) Header Not Set
*Risk Level:* Medium  
*Description:* Missing CSP header allows attackers to execute malicious scripts (XSS) or data injection attacks.  
*Affected URL(s):*
- http://0.0.0.0:8000/register (GET)
*Solution:* Configure the web server to include the Content-Security-Policy header.

### Missing Anti-clickjacking Header
*Risk Level:* Medium  
*Description:* The absence of X-Frame-Options allows clickjacking attacks.
*Affected URL(s):*
- http://0.0.0.0:8000/register (GET - Parameter: x-frame-options)
*Solution:* Set X-Frame-Options to SAMEORIGIN or DENY, or use CSP’s frame-ancestors directive.

### X-Content-Type-Options Header Missing
*Risk Level:* Low  
*Description:* Missing X-Content-Type-Options: nosniff allows MIME-type sniffing vulnerabilities in older browsers.  
*Affected URL(s):*
- http://0.0.0.0:8000/register (GET, POST - Parameter: x-content-type-options - 3 Instances)
*Solution:* Set the X-Content-Type-Options header to nosniff.

### User Agent Fuzzer
*Risk Level:* Informational  
*Description:* Detects differences in response based on User-Agent headers.
*Affected URL(s):*
- http://0.0.0.0:8000/register (POST - Parameter: Header User-Agent - 12 Instances)
*Solution:* Ensure consistent response handling regardless of the User-Agent.

## Conclusion
This ZAP scanning report highlights multiple security vulnerabilities, including critical risks like SQL Injection and Path Traversal. Immediate remediation is recommended, particularly implementing proper input validation, enforcing security headers, and ensuring the application does not expose unnecessary file system access. Strengthening security controls can significantly reduce potential attack vectors.

