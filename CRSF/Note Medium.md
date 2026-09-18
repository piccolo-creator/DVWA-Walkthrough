# DVWA Cross-Site Request Forgery (CSRF) — Medium Level Walkthrough

This guide details the methodology, analysis, and execution steps used to bypass the **Medium Level CSRF** defense mechanism in the Damn Vulnerable Web Application (DVWA) using **Burp Suite**.

---

## 1. Initial Assessment & The Defended Environment

In the **Medium Level** CSRF challenge, the application introduces a security validation check utilizing the `HTTP_REFERER` header. The server verifies if the incoming request originated from its own server domain or IP address (`127.0.0.1` or `localhost`). 

### The Reflected XSS Roadblock
* **Initial Strategy:** An exploitation attempt was made using a Reflected Cross-Site Scripting (XSS) payload injected into the `XSS (Reflected)` input domain to force an inline execution of an asynchronous background `GET` request via an `XMLHttpRequest` JavaScript sequence.
* **The Problem:** Modern browser security rules (such as **SameSite Cookie policies**) block or strip authentication cookies during cross-site JavaScript script handling. This restriction caused the execution chain to fail when relying on simple injection methods directly within the browser interface.

---

## 2. Exploitation Framework with Burp Suite

To circumvent client-side browser restrictions and manipulate network-level headers directly, **Burp Suite Community Edition** was integrated as an intercepting proxy.

### Step 1: Traffic Capture (Proxy Intercept)
1. Burp Suite's integrated Chromium browser was launched to channel application traffic through the loopback proxy.
2. The DVWA challenge environment security was configured to **Medium**.
3. Under the **Proxy** -> **Intercept** module, intercept controls were enabled (**Intercept is on**).
4. A password submission request was initialized on the `CSRF` panel, freezing the execution context and capturing the raw upstream transaction state.

```http
GET /vulnerabilities/csrf/?password_current=password&password_new=hacker123&password_conf=hacker123&Change=Change&user_token=46fd5c3ffdff940ae5bbda72a40adaf HTTP/1.1
Host: 127.0.0.1:42001
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36
Referer: http://127.0.0
Cookie: security=medium; PHPSESSID=e858fe512532cb96cb2cb25d03a17c12b
```

### Step 2: Request Manipulation (Repeater Module)
The captured frame was routed to the **Repeater** sub-system (`Ctrl + R`) to isolate the request parameters and test the server-side authentication rules.

1. **Analyzing the Flaw:** The underlying PHP code implementation evaluates the `Referer` string globally rather than asserting a strict prefix match anchor. It simply looks for the literal value `127.0.0.1` anywhere within the payload string.
2. **Forging the Header:** The `Referer` line was modified from its legitimate internal state to point to an untrusted endpoint while appending the matching token parameter structure onto the end of the request domain:

```http
Referer: http://hacked.com
```

3. **Transmission:** Clicking **Send** executed the forged header directly against the target host backend.

---

## 3. Vulnerability Verification & Results

### Sever-Side Response Processing
Upon analyzing the data returned in the **Response** preview screen inside Burp Suite, looking up the term `changed` confirmed that the spoofed parameter successfully bypassed the domain filtering architecture.

The server responded with an `HTTP/1.1 200 OK` header, displaying the application message:
```html
<pre>Password Changed.</pre>
```

### Post-Exploit Verification Failures
When testing credentials using third-party credential assertion dialogs or mismatched environment cookies, an validation failure will trigger a `Wrong password for 'admin'` exception error notice. Ensure that the active test execution matches your target session parameters and matching target token rules.

---

## 4. Real-World Attack Scenario Translation

While this exploit was executed via Burp Suite for testing validation behaviors within an administrative context, a live-threat implementation translates outside of an interactive proxy:

1. **Malicious Domain Staging:** An attacker purchases or compromises a domain space and designs a path containing the trusted string segment, such as: `http://attacker-controlled-space.com`
2. **Exploit Payload Delivery:** The landing directory hosts an index page with an embedded hidden structure (like an iframe or auto-submitting form vector) targeting the DVWA password execution path.
3. **Automatic Referer Spreading:** When a logged-in target user visits the malicious path, the user's browser natively sets the request header to `Referer: http://attacker-controlled-space.com`.
4. **Passive Execution:** The vulnerable backend verifies the presence of the string array element `127.0.0.1`, validates the query as secure, and switches the administrative account credentials silently.

