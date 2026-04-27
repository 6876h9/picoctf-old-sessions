# Challenge Notes — Old Sessions

## Key Observations

- The `/sessions` endpoint required no authentication to access.
- Both session entries had `_permanent: True`, confirming the sessions would never expire on their own.
- The admin session token was stored in plaintext in the server-side session store.
- No secondary validation (IP binding, user-agent check) was in place.
- The endpoint path was leaked through a public user comment — not found through directory brute-forcing.

## What to Check in Similar Challenges

- Review all user-generated content on the page for hints pointing to internal paths.
- Check `/sessions`, `/debug`, `/admin`, `/config`, `/.env` as low-hanging fruit on Flask/Django apps.
- Inspect cookies immediately after login — note the cookie name, value format, and flags (HttpOnly, Secure, SameSite).
- If the app uses server-side sessions, the cookie value is a session ID and can be substituted if another valid ID is known.

## Tools That Would Also Work

- `curl -b "session=<token>" <target_url>` — command-line session test
- Burp Suite — intercept and modify the cookie in the request before it hits the server

## Time to Solve

Approximately 10 minutes from instance launch to flag capture.
