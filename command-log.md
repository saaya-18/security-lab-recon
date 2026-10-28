# Sanitized Command Log

**Date:** October 28-29, 2026
**Target:** OWASP Juice Shop at http://127.0.0.1:3000 (my own laptop, localhost only, in scope per `rules-of-engagement.md`)
**Sanitization:** Windows username and personal paths removed from all prompts.

## 1. Confirm target is inside the isolated lab
```
docker start juice-shop
docker ps
docker port juice-shop
```
Output:
```
juice-shop

CONTAINER ID   IMAGE                   COMMAND                  CREATED      STATUS         PORTS                      NAMES
84dfc689bf44   bkimminich/juice-shop   "/nodejs/bin/node /j..."  3 days ago   Up 6 minutes   127.0.0.1:3000->3000/tcp   juice-shop

3000/tcp -> 127.0.0.1:3000
```
The port is bound to 127.0.0.1 only, so the target is reachable only from this laptop (isolated lab).

## 2. Response headers
```
curl.exe -I http://127.0.0.1:3000
```
Output:
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 28 Oct 2026 18:29:46 GMT
ETag: W/"2700-1a183c67f48"
Content-Type: text/html; charset=UTF-8
Content-Length: 9984
Vary: Accept-Encoding
Date: Wed, 28 Oct 2026 18:29:58 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```

## 3. robots.txt (routes the site mentions)
```
curl.exe http://127.0.0.1:3000/robots.txt
```
Output:
```
User-agent: *
Disallow: /ftp
```

## 4. Application version
```
curl.exe http://127.0.0.1:3000/rest/admin/application-version
```
Output:
```
{"version":"20.3.0"}
```

## 5. Search input (visible input)
```
curl.exe "http://127.0.0.1:3000/rest/products/search?q=apple"
```
Output (first part only):
```
{"status":"success","data":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg", ...
(output truncated; the response also listed Apple Pomace and Pineapple Juice)
```

## 6. Exposed folder check
```
curl.exe -i http://127.0.0.1:3000/ftp
```
Output (end of response; the top of the output was not captured):
```
... <ul id="files" class="view-tiles">
quarantine (directory)
acquisitions.md
announcement_encrypted.md
coupons_2013.md.bak
eastere.gg
encrypt.pyc
incident-support.kdbx
legal.md
package-lock.json.bak
package.json.bak
suspicious_errors.yml
</ul> </div> </body> </html>
curl: (18) end of response with 12 bytes missing
```
Note: the response is an HTML directory listing of files, so the folder is browsable without logging in.
