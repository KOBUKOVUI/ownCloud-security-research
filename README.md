Vulnerability Classification

CWE-307: Improper Restriction of Excessive Authentication Attempts

Severity

CVSS v3.1: 6.5 (Medium)

AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N

The score reflects the impact observed during testing of the affected authentication flow.

Technical Description

The affected WebDAV endpoint accepts authentication credentials through the HTTP Authorization: Basic header.

Example request:

GET /remote.php/webdav/ HTTP/1.1
Host: target.example
Authorization: Basic <BASE64(username:password)>

The Base64 value represents:

username:password

Repeated authentication failures against the same account do not trigger the expected brute-force protection mechanisms.

During testing, multiple consecutive invalid authentication attempts were submitted without observing:

Rate limiting
Progressive authentication delays
Temporary account lockout
HTTP 429 Too Many Requests

A subsequent authentication attempt using valid credentials was still accepted.

Proof of Concept
Step 1 — Access the WebDAV Authentication Flow

Authenticate to ownCloud and obtain access to a private document/share that uses the affected WebDAV authentication flow.

When accessing the affected endpoint, the server presents an HTTP Basic Authentication challenge.

Step 2 — Observe the Authentication Request

The credentials are transmitted through the HTTP Authorization header:

Authorization: Basic <BASE64(username:password)>
Step 3 — Perform Repeated Authentication Attempts

Using a controlled test account, repeatedly submit authentication requests containing invalid passwords.

Example test credentials:

testuser:incorrect-password-01
testuser:incorrect-password-02
testuser:incorrect-password-03
...

The requests can be reproduced using Burp Suite Intruder or another HTTP testing tool.

Step 4 — Observe the Server Behavior

After multiple consecutive failed authentication attempts, the server continues accepting authentication requests without enforcing an effective brute-force protection mechanism.

No temporary lockout, progressive delay, rate limiting, or HTTP 429 Too Many Requests response is observed.

Step 5 — Verify Successful Authentication

A subsequent request using the correct credentials is accepted normally.

This demonstrates that repeated authentication failures do not sufficiently restrict subsequent password-guessing attempts through the affected WebDAV authentication mechanism.

A short PoC video demonstrating the complete test is included with this research.

Impact

An attacker able to reach the affected WebDAV authentication endpoint can repeatedly attempt passwords against user accounts without encountering effective application-level brute-force protections.

Successful password guessing could result in unauthorized access to the targeted ownCloud account and its associated resources, depending on the privileges and permissions of that account.

The vulnerability therefore increases the feasibility of online password-guessing attacks against accounts exposed through the affected authentication flow.

Comparison With Related Vulnerability

A similar issue has previously been reported in a related Nextcloud product and assigned:

CVE-2023-32319

Reference:

https://hackerone.com/reports/1879549

This reference is provided for comparison of the authentication brute-force protection issue and does not imply that the two vulnerabilities are identical.

Recommended Mitigation

The WebDAV authentication flow should be integrated with the application's existing brute-force protection mechanism.

Recommended controls include:

Track failed authentication attempts for each account and/or source.
Apply progressive delays after repeated authentication failures.
Enforce a configurable maximum number of consecutive failed attempts.
Temporarily reject further authentication attempts after the configured threshold is reached.
Return HTTP 429 Too Many Requests when rate limiting is triggered.
Apply the protection consistently across all authentication mechanisms, including HTTP Basic Authentication used by WebDAV.
Ensure that alternative authentication mechanisms or endpoints cannot be used to bypass the application's existing brute-force protection.

A possible policy could temporarily restrict authentication after a configurable number of consecutive failures, such as 10 attempts. The exact threshold and restriction period should be determined according to the application's security requirements.

Disclosure Timeline
Date	Event
2026-07-18	Vulnerability reported to CERT/CC
2026-07-18	CERT/CC received the report
2026-08-31	CERT/CC confirmed that the vendor had not responded and began the CVE reservation process
TBD	CVE assignment
TBD	Public disclosure
References
ownCloud Server: https://github.com/owncloud/core
Related vulnerability: CVE-2023-32319
HackerOne report: https://hackerone.com/reports/1879549

Researcher

Hoang

Security research and responsible disclosure.

Disclaimer

This research was conducted for security testing and responsible disclosure purposes.

Testing was performed against a controlled environment. No unauthorized access to third-party accounts or data was performed.
