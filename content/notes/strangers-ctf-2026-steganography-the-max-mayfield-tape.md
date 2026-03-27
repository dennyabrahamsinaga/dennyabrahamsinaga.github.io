---
title: "Stranger's CTF 2026 - The Max Mayfield Tape"
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "steganography"]
description: "Writeup for The Max Mayfield Tape from Stranger's CTF Techtrix '26."
---

### Challenge Information
- **Category:** Steganography
- **Description:** 
> Max was listening to her favorite song on loop to keep Vecna out of her mind. However, we intercepted her walkman and noticed a strange, high-pitched anomaly in the background of the track. What is Vecna's message?

### Files Provided
- **[cursed_tape.wav](/ctfs/files/strangers-ctf-2026cursed_tape.wav)**

### Analysis
The challenge description hints at a "high-pitched anomaly" in the background of a WAV file. This is a common indicator for audio spectrogram steganography, where data is hidden visually in the frequency domain.

#### 1. Metadata Inspection
First, we check the audio properties:

```bash
exiftool cursed_tape.wav
```

```text
ExifTool Version Number         : 13.25
File Name                       : cursed_tape.wav
Directory                       : steganography/The Max Mayfield Tape
File Size                       : 2.7 MB
File Modification Date/Time     : 2026:03:26 08:11:50+07:00
File Access Date/Time           : 2026:03:27 22:24:31+07:00
File Inode Change Date/Time     : 2026:03:26 10:10:55+07:00
File Permissions                : -rwxrwxrwx
File Type                       : WAV
File Type Extension             : wav
MIME Type                       : audio/x-wav
Encoding                        : Microsoft PCM
Num Channels                    : 1
Sample Rate                     : 44100
Avg Bytes Per Sec               : 88200
Bits Per Sample                 : 16
Duration                        : 0:00:31
```
The file is a standard 16-bit Mono PCM WAV file at 44.1kHz.

#### 2. Spectrogram Generation
To visualize the frequencies, we can use `ffmpeg` to generate a high-resolution linear spectrogram:

```bash
ffmpeg -i cursed_tape.wav -lavfi showspectrumpic=s=2048x1024:legend=0:scale=lin spectrogram.png
```

### Solution
Upon inspecting the generated `spectrogram.png`, the flag is clearly visible in the mid-to-high frequency range, superimposed over the audio signal.

![spectogram](/images/ctf/2026/strangers-ctf-2026/spectrogram.png)

#### Flag
`CTF{V3cn4_1s_L1st3n1ng}`
