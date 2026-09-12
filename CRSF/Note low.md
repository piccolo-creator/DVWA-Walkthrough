# Cybersecurity Lab Documentation: DVWA CSRF (Low Level)

## Executive Summary
This document outlines the tactical walkthrough and verification analysis for the **Cross-Site Request Forgery (CSRF)** vulnerability module under the **Low Security Level** inside the **Damn Vulnerable Web Application (DVWA)** sandbox. The objective was to create a functional mock phishing page outside the target application to force an authenticated user's browser into executing an unauthorized administrative action (updating account credentials) without their direct interaction or consent.

---

## 🛠️ Lab Environment Setup
The exercises were executed across parallel browser sessions hosted on an isolated Linux virtualization platform. 

As captured in the primary application viewport (**Screenshot 1**), the core target service handles the password change request parameters via the insecure HTTP GET query string line, processing authentication tokens via automated session cookies.

<img width="1920" height="973" alt="CSRF 01" src="https://github.com/user-attachments/assets/e5bab969-80fd-438c-9d8e-c8e8e8a27c4c" />


---

## 🛑 Security Level 1: Low Security

### 1. Phishing Payload Development
To execute the Cross-Site Forgery vector completely from outside the web framework application, an external document was built using the system terminal editor. As documented via the `cat` command buffer output in **Screenshot 2**, the malicious file was written with the following structural layout:

```html
<html>
<body>
    <h1>Congratulations! You won a free Iphone</h1>
    <p>Click here to receive your prize!</p>

    <!-- Malicious code -->
    <form action="http://127.0.0" method="GET">
        <input type="hidden" name="password_new" value="hacker123" />
        <input type="hidden" name="password_conf" value="hacker123" />
        <input type="hidden" name="Change" value="Change" />
        <input type="submit" value="CLAIM IPHONE HERE!" />
    </form>
</body>
</html>
```

*Note: While the terminal draft in **Screenshot 2** captures an initial configuration process, the structural variables and paths were mapped into the browser workflow layout sequentially.*

<img width="571" height="276" alt="CSRF 02" src="https://github.com/user-attachments/assets/6168cf02-7345-4460-8835-e42ff8241a4e" />


---

### 2. Deploying the Malicious Payload
To bypass administrative root file access constraints inside the web rendering context, the payload script file was staged out of the root system tree into a shared user folder segment. 

As verified in the file-browser modal of **Screenshot 3**, the code script was placed within the workspace track belonging to the system operator profile:
* **Absolute File Path Storage Link:** `/home/victor/Desktop/attacker.html`

<img width="1920" height="974" alt="CSRF 03" src="https://github.com/user-attachments/assets/e3a2ea96-193d-4f1a-9636-48f6f0bfeac2" />


---

### 3. Exploitation Delivery Vector
A mock scenario was staged where an active user session navigates onto an external web layout via a separate browser tracking tab. 

As demonstrated via the Inspect Element DOM browser developer view in **Screenshot 4**, loading the local target asset `file:///home/victor/Desktop/attacker.html` renders a clean social engineering page. Hidden beneath the harmless structural design layout elements are the critical, pre-populated form components pointing directly back to the target web instance (`127.0.0.1:42001`).

<img width="1920" height="978" alt="CSRF 04" src="https://github.com/user-attachments/assets/d1c34c29-cd45-4bda-a776-11055c90c74b" />


---

### 4. Verification & Lab Results
The moment the mock action button (**"CLAIM IPHONE HERE!"**) was triggered from the rogue tab, the browser automatically packed the hidden credential payload and routed it to the backend server alongside the victim's active background authentication cookies.

As confirmed on the credential checking panel in **Screenshot 5**, the exploit succeeds seamlessly. Testing the backend authentication engine confirms that the existing account password has been compromised and overwritten by the rogue user inputs, verifying complete compromise of the identity control layer.

<img width="1305" height="561" alt="CSRF 05" src="https://github.com/user-attachments/assets/3405abdf-8c07-4819-9d8a-dcad89625814" />


