# Service and Attack-Surface Inventory

**Target:** OWASP Juice Shop v20.3.0, http://127.0.0.1:3000 (isolated local lab)
**Scope:** Only this target. Evidence comes from `command-log.md` and from browsing the app.

Risk hypotheses are untested ideas about what could be weak. They are not confirmed findings.

| # | Observation | Evidence | Risk hypothesis |
|---|-------------|----------|-----------------|
| 1 | Web service on 127.0.0.1:3000 over plain HTTP, no TLS | `curl.exe -I` returned `HTTP/1.1 200 OK`; no `Strict-Transport-Security` header | Traffic is unencrypted. Fine on localhost, but credentials would be exposed on a real network. |
| 2 | Application version is publicly exposed: 20.3.0 | `/rest/admin/application-version` returned `{"version":"20.3.0"}` | Version disclosure lets an attacker look up known weaknesses for that release. |
| 3 | Wide-open CORS policy | `Access-Control-Allow-Origin: *` in response headers | Any website could read responses from the API in a user's browser. |
| 4 | Some security headers present, others not seen | Present: `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `Feature-Policy: payment 'self'`. Not seen: `Content-Security-Policy`, `Strict-Transport-Security` | Missing CSP could make cross-site scripting easier to exploit. |
| 5 | Custom header points to a route | `X-Recruiting: /#/jobs` | Headers can leak hidden routes or internal hints. |
| 6 | `robots.txt` lists a disallowed path | `Disallow: /ftp` | Listing a path in robots.txt points directly at an area the owner wanted hidden. |
| 7 | `/ftp` directory listing is reachable without login | `curl.exe -i /ftp` returned an HTML file listing | Directory listing exposes file names and lets anyone browse the folder. |
| 8 | Backup, config and credential-like files in `/ftp` | Listing includes `package.json.bak`, `package-lock.json.bak`, `coupons_2013.md.bak`, `incident-support.kdbx`, `encrypt.pyc`, `suspicious_errors.yml`, `announcement_encrypted.md` | Backup files can leak dependencies and source details. A `.kdbx` file is a password database and would be highly sensitive on a real system. |
| 9 | Search input sends data to `/rest/products/search?q=` and returns JSON | `curl.exe` search for `apple` returned `{"status":"success","data":[...]}` with product records | User-supplied input reaches a backend query, so possible injection or cross-site scripting. |
| 10 | Login form (email, password) | Seen in the browser at `#/login` | Possible weak authentication, injection in the login, or no brute-force protection. |
| 11 | Registration form (email, password, repeat password, security question and answer) | Seen in the browser at `#/register` | Weak password rules, user enumeration, or insecure account recovery. |
| 12 | Basket with quantity input | Seen in the browser under Your Basket | Possible quantity tampering or access to another user's basket. |

## Visible inputs
- Search box (`q` parameter)
- Login: email, password
- Registration: email, password, repeat password, security question and answer
- Basket quantity
