# Cybersecurity Lab Documentation: DVWA Command Injection

## Executive Summary
This document provides a comprehensive walk-through and technical analysis of the Command Injection vulnerability modules within the Damn Vulnerable Web Application (DVWA) environment. The purpose of this lab was to safely exploit a flawed web interface to run unauthorized Operating System (OS) commands on a local hosting server.

---

##  Lab Environment Setup
The exercises were performed inside a Kali Linux Virtual Machine hosted on an isolated host network interface.

As captured in **Screenshot 1** (Terminal), the target web services were launched directly from the root terminal:
* **DVWA Web Service:** Started via the local pre-built installer utility (`dvwa-start`).
* **Local Web URL:** Service provisioned dynamically on the local loopback interface at http://127.0.0.1:42001.

<img width="1913" height="738" alt="pen test 00" src="https://github.com/user-attachments/assets/5620edf2-200f-43b1-94f9-066af53bb4cd" />


---

##  Security Level 1: Low Security

### 1. Source Code Analysis
According to the raw source code captured in the pop-up window of **Screenshot 2** (Low Level), the application reads raw input straight from the user form without validating its content.

### 2. Exploitation Vector
Because there is zero input sanitization or filtering logic, any standard shell metacharacter can be passed straight to the execution string.
* **The Input Payload:** `127.0.0.1 ; whoami`
* **The Semicolon Mechanism:** The operating system shell reads the semicolon (`;`) as a command separator. It stops processing the ping utility and immediately launches a secondary standalone thread.

### 3. Verification & Results
After sending the payload, the web server processes the instruction and prints out the system username `dvwa` directly beneath the automated ping traffic logs.

<img width="1920" height="979" alt="pen test 1" src="https://github.com/user-attachments/assets/6040e0f6-a3a4-4b2c-bbd4-734678258f40" />


---

##  Security Level 2: Medium Security

### 1. Source Code Analysis
As shown in **Screenshot 3** (Medium Code), the developer implemented a basic defense policy to filter out the commands utilized in the Low-level module. They introduced an input pattern matching array (a blacklist filter) that strips out `&&` and `;`.

<img width="1920" height="995" alt="pen test 2" src="https://github.com/user-attachments/assets/ad730999-89de-410d-842d-6c356d434eb7" />


### 2. Exploitation Vector (The Long Dash Loophole)
The core failure of this defense is relying on a restricted blacklist pattern. The developer blocked two specific tokens but entirely omitted other valid shell operators.

To bypass this filter, the vertical **pipe (`|`) character** (often called the "long dash") was utilized. In Linux terminals, a pipe forces the machine to route the output data of the first task into the entry point of the next command.
* **The Input Payload:** `10.0.2.15 | whoami`

### 3. Verification & Results
Because the pipe symbol is missing from the developer’s `$substitutions` array list, the filter treats the character as completely safe text and ignores it.

As confirmed in the main web layout of **Screenshot 4** (Medium Result), the web execution engine skips past the defense wall, fires the secondary command, and prints out the core username `dvwa` inside the browser view once again.

<img width="1920" height="979" alt="pen test 3" src="https://github.com/user-attachments/assets/89907932-360f-4aa5-a4a9-641141e15a54" />


---

##  Post-Exploit Conceptual Assessment: High & Impossible

### 1. High Security Level Analysis
While not detailed directly in the local source pop-ups, the walkthrough configurations establish that the developer attempts to patch the medium loophole by adding the pipe (`|`) to the blacklist. However, a human syntax error was coded (`'| ' => ''`), meaning it only removes a pipe if a space follows it. Squeezing the payload into `10.0.2.15|whoami` completely outsmarts this condition.

<img width="1920" height="1000" alt="pen test 4" src="https://github.com/user-attachments/assets/790e8d11-be4f-4e54-8ad1-1ff99b50be13" />


### 2. The Impossible Remediation Policy
To eliminate this risk entirely, the application must shift completely from a weak character blacklist to strict token type validation. The application should check every incoming piece of text against an internal rule pattern that ensures the input contains only numeric integers and periods mapped to a legal IPv4 context (`A.B.C.D`). If non-numeric characters (such as `;`, `|`, or alphabet letters) are present, execution is instantly blocked before it ever touches the server's command-line interface.

<img width="1920" height="995" alt="DVWA CI 05" src="https://github.com/user-attachments/assets/9447165e-10f2-440a-9a78-905a450c6e65" />

