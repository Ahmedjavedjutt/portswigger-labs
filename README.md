# PortSwigger Web Security Academy — Lab Write-ups

**Student:** Ahmed Javed
**Institute:** Boston Institute of Analytics (BIA) — Cybersecurity & Ethical Hacking
**Certifications:** ISC2 CC Domain 1-4 | BIA Cybersecurity Diploma (In Progress)
**Platform:** PortSwigger Web Security Academy (free)
**Total Labs:** 5 completed (3 Apprentice · 2 Practitioner)
**Tools:** Burp Suite Community Edition · Browser DevTools

---

## What I Practised

These labs cover real OWASP Top 10 vulnerabilities exploited hands-on:

- SQL Injection — login bypass, UNION attacks, blind time-based
- Server-Side Request Forgery (SSRF)
- Insecure Direct Object References (IDOR / Broken Access Control)

---

## Lab 1 — SQL Injection Login Bypass

**Difficulty:** Apprentice
**Link:** https://portswigger.net/web-security/sql-injection/lab-login-bypass

**Vulnerability:** Authentication bypass via SQL injection in the login form.

**What I did:**
1. Opened the login page
2. Entered `administrator'--` in the username field
3. Entered anything in the password field
4. Logged in as administrator — no valid password needed

**Payload:** `administrator'--`

**Why it works:** The backend SQL query becomes:
```
SELECT * FROM users WHERE username='administrator'--' AND password='...'
```
The `--` comments out the rest of the query. The password check is never evaluated.

**Key learning:** Never trust user input in SQL queries. Fix: use parameterized queries / prepared statements.

---

## Lab 2 — SQL Injection UNION Attack: Number of Columns

**Difficulty:** Practitioner
**Link:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

**Vulnerability:** UNION-based SQL injection in the product category filter.

**What I did:**
1. Intercepted the category request in Burp Suite Repeater
2. Injected `' ORDER BY 1--` then incremented to `2--`, `3--` until an error appeared
3. Confirmed 3 columns with `' UNION SELECT NULL,NULL,NULL--`

**Payload:** `' UNION SELECT NULL,NULL,NULL--`

**Why it works:** A UNION SELECT must return the same number of columns as the original query. NULL is compatible with any data type. Incrementing ORDER BY until an error reveals the column count.

**Key learning:** Determining column count is the first step in every UNION attack — it unlocks data extraction from other database tables.

---

## Lab 3 — Blind SQL Injection with Time Delays

**Difficulty:** Practitioner
**Link:** https://portswigger.net/web-security/sql-injection/blind/lab-time-delays

**Vulnerability:** Blind SQLi in the TrackingId cookie — no visible output, confirmed via response timing.

**What I did:**
1. Captured the request in Burp Suite
2. Modified the TrackingId cookie with a time-delay payload
3. Page took 10 seconds to respond — injection confirmed

**Payload:** `'; SELECT pg_sleep(10)--`

**Why it works:** When the app returns no error or data, you can still confirm injection by making the database pause. A 10-second delay proves the injection point is valid and executing.

**Key learning:** Blind SQLi is harder to detect but still fully exploitable. Time-based confirmation is a standard technique used in real penetration tests.

---

## Lab 4 — Basic SSRF Against Local Server

**Difficulty:** Apprentice
**Link:** https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost

**Vulnerability:** Server-Side Request Forgery — the server makes HTTP requests on my behalf.

**What I did:**
1. Intercepted the stock check request in Burp Suite
2. Found the `stockApi` parameter containing an external URL
3. Replaced the value with `http://localhost/admin`
4. The server fetched its own internal admin panel and returned it to me

**Payload:** `http://localhost/admin`

**Why it works:** The web server can reach its own internal admin panel which is blocked from outside. By controlling what URL the server requests, I make it fetch restricted internal resources for me.

**Key learning:** SSRF is OWASP Top 10 2021 A10. Internal services that trust localhost are fully exposed when SSRF is present. Fix: whitelist allowed URLs and block internal IP ranges.

---

## Lab 5 — Insecure Direct Object References (IDOR)

**Difficulty:** Apprentice
**Link:** https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references

**Vulnerability:** IDOR — accessing another user's file by changing an ID in the URL.

**What I did:**
1. Used the live chat feature and downloaded my transcript
2. Noticed the download URL contained `/download/2`
3. Changed it to `/download/1` in Burp Suite Repeater
4. Retrieved another user's chat transcript — which contained their plaintext password
5. Used that password to log into their account and complete the lab

**Payload:** Change `/download/2` to `/download/1`

**Why it works:** The server uses sequential numeric IDs for files but does not verify whether the requesting user owns that resource. No authorization check = any user accesses any file.

**Key learning:** IDOR falls under Broken Access Control — OWASP A01 (the #1 vulnerability in 2021). Always enforce server-side ownership checks on every single resource request.

---

## OWASP Top 10 Mapping

| Lab | Vulnerability | OWASP 2021 Category |
|-----|--------------|---------------------|
| 1 | SQL Injection login bypass | A03 Injection |
| 2 | UNION-based SQL Injection | A03 Injection |
| 3 | Blind SQL Injection (time-based) | A03 Injection |
| 4 | Server-Side Request Forgery | A10 SSRF |
| 5 | Insecure Direct Object Reference | A01 Broken Access Control |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Burp Suite Community | Request interception, Repeater, parameter editing |
| Browser DevTools | Initial inspection, URL manipulation |
| PortSwigger Academy | Lab environment |

## References

- [PortSwigger SQLi Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [OWASP Top 10 2021](https://owasp.org/www-project-top-ten/)
- [MITRE ATT&CK T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)

---

*Part of my cybersecurity portfolio → [github.com/Ahmedjavedjutt](https://github.com/Ahmedjavedjutt)*
