---
title: "Stranger's CTF 2026 - Recovered Logs"
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "web"]
description: "Writeup for Recovered Logs from Stranger's CTF Techtrix '26."
---

**Challenge Name:** RECOVERED LOGS - HAWKINS NATIONAL LABORATORY  
**Category:** Web  
**Target:** `nc 140.245.25.63 8005` (Accessible via `curl --http0.9`)

---

### 1. Description
The challenge provides a Dr. Brenner themed terminal service. Attempts to access via standard `curl` fail with an `HTTP/0.9` error, indicating it's a raw TCP service or uses a legacy protocol. 

### 2. Analysis

#### Service Investigation
Connecting to the terminal reveals a restricted menu:
1.  View Previous Messages (Logs)
2.  Send Message to Command
3.  Initiate Upside Down Protocol (Admin)
4.  Disconnect

#### Log Analysis
Using option `1` to view logs provides the following critical information:
-   **User Owens:** "The new token system is a joke. You just take our username, stick the year the lab was founded ('1983') at the end, and run it through a basic MD5 hash?"
-   **Dr. Brenner:** "No one knows my Operator ID is exactly 'DrBrenner' anyway."

From this, the administrative token generation logic is:
`Token = MD5(Username + "1983")`

For Dr. Brenner:
`Token = MD5("DrBrenner1983")`

---

### 3. Solution

#### Step 1: Calculate Admin Token
```bash
echo -n "DrBrenner1983" | md5sum
# Result: 43095647eb9196088480eab17e08605a
```

#### Step 2: Gain Admin Access
1.  Connect via `nc 140.245.25.63 8005`.
2.  Enter Operator ID: `DrBrenner`.
3.  Select option `3` (Initiate Upside Down Protocol).
4.  Provide the calculated MD5 token: `43095647eb9196088480eab17e08605a`.

#### Step 3: Retrieve Flag
Once authenticated as `DrBrenner`, select option `1` (View Previous Messages) again to see "CLASSIFIED NINA LOGS".

```bash
--- CLASSIFIED NINA LOGS ---
Brenner: The tear in spacetime is expanding.
Owens: We need more power to the containment grid.
System: WARNING - Subject 011 has escaped. Protocol override. CTF{Dr_n0_br3nn3R}
```
---
### 4. Final Flag 
`CTF{Dr_n0_br3nn3R}`
