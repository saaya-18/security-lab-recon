# OWASP Web Testing Lab - Findings

**Target:** OWASP Juice Shop v20.3.0 at http://127.0.0.1:3000 (my own isolated lab, in scope per `rules-of-engagement.md`)
**Tester:** Saaya Sajan
**Date:** October 29, 2026

All testing was performed only against the target above. No other systems were accessed.

| # | Finding | Severity | Evidence | Root cause | Remediation |
|---|---------|----------|----------|-------------|--------------|
| 1 | SQL injection in login form allows authentication bypass | Critical | Entered `' OR 1=1--` in the email field with any password at `#/login`. The app logged me in as `admin@juice-sh.op` without a valid password, and displayed: "You successfully solved a challenge: Login Admin." | The login query builds a SQL statement by directly inserting user input (string concatenation) instead of using parameterized queries, so the injected `OR 1=1--` makes the WHERE clause always true. | Use parameterized queries or an ORM with bound parameters for all database lookups. Never build SQL by concatenating user input. |
| 2 | Reflected cross-site scripting (XSS) in the search field | High | Entered `<iframe src="javascript:alert('xss')">` into the search box. The app rendered it and executed it, popping up an alert box with "xss". | User input from the search field is inserted into the page's HTML without sanitizing or escaping it, so a script-bearing tag gets rendered and executed instead of treated as plain text. | Escape or sanitize all user input before rendering it in HTML. Apply a strict Content-Security-Policy to block inline script execution as a second layer of defense. |
| 3 | Admin panel correctly blocked for unauthenticated users | Informational (no vulnerability found) | Navigated directly to `http://127.0.0.1:3000/#/administration` while logged out. The app returned a 403 page: "You are not allowed to access this page!" | Access control on this route correctly checks the user's session before serving admin content. | None needed. Recommend re-testing this route while logged in as a low-privilege (non-admin) user to confirm the same check applies there, not just to logged-out visitors. |
| 4 | Missing security response headers / permissive CORS | Medium | `curl.exe -I http://127.0.0.1:3000` (see `command-log.md`, task 3) returned `Access-Control-Allow-Origin: *` and did not include `Content-Security-Policy` or `Strict-Transport-Security`. | The server does not set a restrictive CORS policy or a Content-Security-Policy, and does not enforce HTTPS via HSTS. | Set `Access-Control-Allow-Origin` to specific trusted domains instead of `*`. Add a `Content-Security-Policy` header restricting script sources. Add `Strict-Transport-Security` once the app is served over HTTPS. |

## Business impact summary
- **SQL injection (#1)** is the most severe: it lets anyone log in as an administrator without credentials, which on a real system would mean full account takeover and access to all user data and orders.
- **XSS (#2)** could let an attacker run code in another user's browser, for example to steal their session or redirect them to a fake login page.
- **CORS/headers (#4)** widen the attack surface and make issues like #2 easier to exploit from another site.
- **#3** is a positive result and included to show the check that was performed, not a flaw.
