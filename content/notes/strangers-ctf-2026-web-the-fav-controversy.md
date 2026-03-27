---
title: "Stranger's CTF 2026 - The Fav Controversy  "
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "web"]
description: "Writeup for The Fav Controversy from Stranger's CTF Techtrix '26."
---

**Challenge Name:** The Fav Controversy  
**Category:** Web  
**Target:** `http://140.245.25.63:8003/`

---

### 1. Description
The challenge description mentions: *"Every time you search for a 'controversial' topic, the site whisks you through a series of hidden doorways before showing you a blank result. The flag isn't at the destination...it's scattered across the journey itself."* It also hints at checking the browser history.

### 2. Analysis

#### Redirection Chain
When a "controversial" search term (like `everything`, `truth`, or `admin`) is entered into the search bar, the server initiates a chain of HTTP 302 redirects instead of directly serving the results.

By using `curl -vL` or monitoring the network history, the following redirect locations were observed:
1.  `/next-page/id=Q1RGe3JjQ19rNF8=`
2.  `/next-page/id=ZCE1NV9MbUEwfQ==`

These `id` parameters appeared to be Base64 encoded strings.

---

### 3. Solution

#### Step 1: Identify the Redirects
Sending a search request for a controversial term:
```bash
curl -v "http://140.245.25.63:8003/search?q=everything"
```
The first redirect points to:
`Location: /next-page/id=Q1RGe3JjQ19rNF8=`

Following the first redirect:
```bash
curl -v "http://140.245.25.63:8003/next-page/id=Q1RGe3JjQ19rNF8="
```
The second redirect points to:
`Location: /next-page/id=ZCE1NV9MbUEwfQ==`

#### Step 2: Decode the Segments
Decoding the Base64 strings found in the URLs:
-   `Q1RGe3JjQ19rNF8=` -> `CTF{rcC_k4_`
-   `ZCE1NV9MbUEwfQ==` -> `d!55_LmA0}`

#### Step 3: Assemble the Segments
Combining the segments results in the final flag.

---

### 4. Flag
`CTF{rcC_k4_d!55_LmA0}`
