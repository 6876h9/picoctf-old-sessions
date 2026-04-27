# Technical Writeup — Old Sessions

## Vulnerability Class

**Improper Session Expiration / Session Fixation / Sensitive Data Exposure**

CWE References:
- CWE-613: Insufficient Session Expiration
- CWE-200: Exposure of Sensitive Information to an Unauthorized Actor

---

## Application Behavior Analysis

The web application is built on Flask (Python). Flask's session management uses signed cookies by default, but the application appears to also maintain a server-side session store, which was exposed at the `/sessions` endpoint.

The session objects in the store had the following structure:

```json
{
  "_permanent": true,
  "key": "<username>"
}
```

`_permanent: True` in Flask sets the session cookie to persist beyond the browser session (i.e., it survives tab or browser closure). The critical missing control here is `PERMANENT_SESSION_LIFETIME`, which when left at its default or not enforced server-side, allows sessions to remain valid indefinitely.

---

## Attack Chain

```
Register account
      |
      v
Observe homepage comments --> /sessions path disclosed
      |
      v
Navigate to /sessions --> admin session token exposed
      |
      v
Open DevTools --> Application --> Cookies
      |
      v
Replace session cookie value with admin token
      |
      v
Refresh page --> Authenticated as admin --> Flag rendered
```

---

## Why This Works

Flask verifies the **signature** of its default cookie-based sessions. However, if the application is using a **server-side session store** (such as Flask-Session with filesystem or Redis backend), the cookie value is simply a session ID — a lookup key into the store. There is no cryptographic protection on the ID itself that prevents substitution.

Since the server-side store exposed all valid session IDs at `/sessions`, substituting the `admin` ID into the browser cookie was sufficient to hijack the session. The server looked up the ID, found a valid `admin` session, and authenticated accordingly.

---

## Secure Configuration (Flask)

```python
from datetime import timedelta
from flask import Flask

app = Flask(__name__)

# Set a strong secret key
app.config['SECRET_KEY'] = os.urandom(32)

# Enforce session lifetime
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(minutes=30)

# Secure cookie flags
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['SESSION_COOKIE_SECURE'] = True   # HTTPS only
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'
```

Additionally, the `/sessions` endpoint must be removed from any production deployment entirely.

---

*6876h9*
