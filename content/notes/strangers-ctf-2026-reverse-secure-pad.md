---
title: "Stranger's CTF 2026 - SecurePad"
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "reverse engineering", "crypto"]
description: "Writeup for SecurePad from Stranger's CTF Techtrix '26."
---

**Challenge Name:** SecurePad  
**Category:** Reverse Engineering / Crypto  
**Description:** 
> SecurePad™ — "Security Through Obscurity… and Comic Sans." A custom encryption algorythm powered notepad.

---

### 1. Description
The challenge provides a Windows executable **[SecurePad.exe](/ctfs/files/strangers-ctf-2026/SecurePad.exe)** and an encrypted file **[flag.enc](/ctfs/files/strangers-ctf-2026/flag.enc)**. The goal is to reverse the "custom encryption algorythm" and decrypt the flag.

### 2. Analysis

#### Binary Overview
- **File Type:** PE32 executable for MS Windows (GUI)
- **Architecture:** Intel i386
- **Symbols:** None (Stripped)

Static analysis using `strings` revealed:
- References to `Comic Sans`.
- Usage of `SystemFunction036` (`RtlGenRandom`) for generating random keys.
- A fake flag `CTF{5011d_5n4k3}`.

#### Reverse Engineering
Disassembling the "Save" function at `0x4014eb` revealed a multi-step encryption process:

1.  **XOR Layer 1:** The plaintext is XORed with an 8-byte random key.
2.  **Custom RC4 Cipher:**
    - **S-box Init:** A custom loop initializes the S-box where `S[i(i+1)/2 mod 256] = i(i+1)/2 mod 256`. Indices not hit by the triangular sequence remain zero.
    - **KSA:** A standard RC4 Key Scheduling Algorithm using the 8-byte random key as the seed.
    - **PRGA:** A standard RC4 Pseudo-Random Generation Algorithm XORs the buffer with the keystream.
3.  **XOR Layer 2:** The buffer is XORed with the same 8-byte random key again.

**Note:** Since XOR is commutative and the key is applied twice, the two simple XOR layers effectively cancel out for the first 8 bytes (and XOR with 0 for the rest), leaving the custom RC4 cipher as the primary encryption.

#### File Format (`flag.enc`)
The file structure was identified as:
- **0x00 - 0x1B:** Ciphertext (28 bytes)
- **0x1C - 0x2F:** Padding (zeros)
- **0x30 - 0x33:** Plaintext length (28)
- **0x37 - 0x3E:** 8-byte random key used for the cipher
- **0x3F:** Random key length (8)

---

### 3. Solution

The decryption requires replicating the specific S-box initialization and then applying standard RC4 decryption using the key found in the file footer.

#### Exploitation Script

```python
import struct

def decrypt(filename):
    with open(filename, "rb") as f:
        data = f.read()
    
    length = 28
    ciphertext = data[:length]
    random_key = data[0x37:0x3f]
    
    # 1. Replicate S-box initialization (Half-zero triangular pattern)
    s = [0] * 256
    bl = 0
    for cl in range(256, 0, -1):
        bl = (bl - cl) & 0xFF
        s[bl] = bl
    
    # 2. RC4 KSA with extracted key
    j = 0
    for i in range(256):
        j = (j + s[i] + random_key[i % 8]) & 0xFF
        s[i], s[j] = s[j], s[i]
        
    # 3. RC4 PRGA Decryption
    res = bytearray()
    i = 0
    j = 0
    for buf_idx in range(length):
        i = (i + 1) & 0xFF
        j = (j + s[i]) & 0xFF
        s[i], s[j] = s[j], s[i]
        res.append(ciphertext[buf_idx] ^ s[(s[i] + s[j]) & 0xFF])
        
    return res

flag = decrypt("flag.enc")
print(flag.decode(errors='ignore'))
```

---

### 4. Flag
`CTF{r0ll_y0ur_0wn_b4d_c0d3}`
