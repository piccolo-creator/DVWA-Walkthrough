# Cybersecurity Lab Documentation: DVWA Command Injection

## Executive Summary
This document provides a comprehensive walk-through and technical analysis of the **Command Injection vulnerability** modules within the **Damn Vulnerable Web Application (DVWA)** environment. The primary objective of this lab was to safely exploit input processing flaws in a web application interface to run unauthorized Operating System (OS) terminal commands on the hosting server back-end.

---

## 🛠️ Lab Environment Setup
The exercises were conducted inside a **Kali Linux Virtual Machine** configured on an isolated internal network host segment. 

As captured in the primary terminal environment screen (**Screenshot 1**), the core web services were deployed directly from the root terminal path:
* **DVWA Web Service Deployment:** Successfully initiated using the local automation script command: `# dvwa-start`.
* **Network Target Mapping:** The web platform was served dynamically over the system loopback adapter path at **`http://127.0.0.1:42001`**.

<img width="1913" height="738" alt="pen test 00" src="https://github.com/user-attachments/assets/eedbfd21-c72b-4f08-b417-c10b7be47d7e" />


---

## 🛑 Security Level 1: Low Security

### 1. Source Code Analysis
According to the raw source code panel captured in **Screenshot 2**, the web application architecture accepts parameters directly from the client form structure without running any filtering or verification logic on the incoming data string:

```php
if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        $cmd = shell_exec( 'ping ' . $target );
    }
    else {
        $cmd = shell_exec( 'ping -c 4 ' . $target );
    }
}
```

### 2. Exploitation Vector
Because the application passes the `\(target` variable straight to a system command shell without cleaning out special characters, the server terminal environment can be hijacked using standard command breaks.  * **Testing Payload utilized:** `127.0.0.1 ; whoami` * **The Semicolon Break Feature:** The Linux shell treats the semicolon (`;`) as an explicit command delimiter line. It processes the initial `ping` directive, stops, and immediately triggers the secondary standalone `whoami` command right after it.  ### 3. Verification & Lab Results As verified in the main browser output window of **Screenshot 2**, the web server successfully executes the command injection payload. The system bypasses the web app interface controls entirely and displays the back-end web user identity account name **`dvwa`** printed explicitly onto the webpage interface.  *(Place `screenshot_2_low.png` here)*  ---  ## ⚠️ Security Level 2: Medium Security  ### 1. Source Code Analysis As displayed in the code review interface of **Screenshot 3**, the system developer attempted to lock down the command execution flaw by building a basic string substitution array (a character blacklist):  \%\%MAGIT_PARSER_PROTECT\%\%```php // Set blacklist $substitutions = array(     '&&' => '',     ';'  => '', );  // Remove any of the characters in the array (blacklist). $target = str_replace( array_keys( $substitutions ), $substitutions, $target ); \%\%MAGIT_PARSER_PROTECT\%\%```  If the program runs into a semicolon (`;`) or a double-ampersand (`&&`), it wipes the characters away completely and changes them to an empty string format (`''`).  ### 2. Exploitation Vector (The Long Dash Loophole) The central flaw in this defense configuration is the strategic reliance on a limited character blacklist. The developer only accounted for two specific command breaks, forgetting other valid shell terminal characters.  To break this filter layer, the vertical **pipe (`\vert{}`) symbol** was used. In Linux architectures, a pipe takes the output string generated from the left command and streams it straight as input into the right command.  * **Testing Payload utilized:** `10.0.2.15 \vert{} whoami`  ### 3. Verification & Lab Results Because the pipe symbol character is not listed in the developer's `\)substitutions` array logic, the sanitization filter treats it as regular string input and ignores it completely. 

As validated on the main page view of **Screenshot 3**, the web application engine accepts the payload, forwards it directly to the system background shell, and outputs the user profile tag **`dvwa`** to the screen again.

*(Place `screenshot_3_medium.png` here)*

---

## 🔒 Security Level 3: High Security

### 1. Source Code Analysis
As documented in the code view window of **Screenshot 4**, the developer expanded the blacklist parameters significantly to target a much larger array of control characters, including pipes, ampersands, variables, and bracket structures:

```php
// Set blacklist
$substitutions = array(
    '||' => '',
    '&'  => '',
    ';'  => '',
    '| ' => '', // <-- Human Syntax Error
    '-'  => '',
    '$'  => '',
    '('  => '',
    ')'  => '',
    '`'  => '',
    '||' => '',
);
```

### 2. Exploitation Vector (The Space Elimination Trick)
The high security layer fails due to a micro syntax error hidden within the substitution array rule: `'| ' => ''`. The developer accidentally left a trailing empty space next to the pipe character string indicator. 

This means the application is strictly searching for a pipe symbol *immediately followed by a space*. If an entry does not contain that space, it will slip through the filter completely untouched.

* **Testing Payload utilized:** `10.0.2.15|whoami`

### 3. Verification & Lab Results
By squeezing the payload completely together and **removing the space** surrounding the character, the target string bypasses the filter match rules. The pipe command breaks out of the context loop, runs the payload, and displays the **`dvwa`** username target on the display sheet, as confirmed in **Screenshot 4**.

*(Place `screenshot_4_high.png` here)*

---

## 🛡️ Secure Defensive Remediation: The Impossible Level
To fully secure a application environment against Command Injection risks, developers must abandon the use of weak signature-based blacklists entirely. 

### The White-Listing Solution
On the **Impossible** level, input text validation must be rigidly enforced. The application must break down user entries into individual octets and verify that the input corresponds *solely* to a valid, clean numeric IPv4 template layout structure (`A.B.C.D`). If any invalid alphabetic characters, semicolons, or pipe symbols are detected during validation, the thread is dropped instantly before the variable ever makes contact with a system shell execution process.

