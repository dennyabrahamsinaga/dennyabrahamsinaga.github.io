---
title: "Stranger's CTF 2026 - The Kamchatka Taps"
date: 2026-03-27
draft: false
categories: ["CTF"]
tags: ["ctf", "reverse", "audio forensics"]
description: "Writeup for The Kamchatka Taps from Stranger's CTF Techtrix '26."
---

### Challenge Information
- **Category:** Reverse / Audio Forensics
- **Description:** 
> Joyce and Murray intercepted a faint audio signal originating from a high-security Russian prison in Kamchatka. It sounds like someone is rhythmically banging on a heating pipe. Decode the message to find out who is trapped inside.

### Files Provided
- **[kamchatka_taps.wav](/ctfs/files/strangers-ctf-2026/kamchatka_taps.wav)**

### Analysis
The "rhythmically banging" description is a clear hint for Morse code. The WAV file contains sharp, impulsive tapping sounds separated by varying intervals.

#### 1. Visualization
Generating a waveform plot confirms the rhythmic structure:

```bash
ffmpeg -i kamchatka_taps.wav -lavfi "showwavespic=s=2000x400:colors=red" waveform.png
```

#### 2. Automated Decoding
A Python script was used to:
1. Load the audio data and calculate the amplitude envelope.
2. Apply smoothing (moving average) to merge the sharp transients into measurable pulses.
3. Detect pulse durations and gaps.
4. Categorize pulses into dots and dashes, and gaps into letter and word separators.

### Solution
Solver:

```python
import numpy as np
from scipy.io import wavfile
from scipy.ndimage import uniform_filter1d

# Load the wav file
sample_rate, data = wavfile.read("/home/eph/CTF/2026/Stranger's CTF - Techtrix '26/reverse/The Kamchatka Taps/kamchatka_taps.wav")

if data.dtype == np.int16:
    data = data.astype(np.float32) / 32768.0
elif data.dtype == np.int32:
    data = data.astype(np.float32) / 2147483648.0

# Amplitude envelope
amplitude = np.abs(data)

# SMOOTHING: Moving average to join impulsive taps
# A typical dot should be ~50-100ms. 
# Window of 10ms (441 samples)
window_size = int(0.01 * sample_rate)
smoothed = uniform_filter1d(amplitude, size=window_size)

max_amp = np.max(smoothed)
threshold = 0.3 * max_amp

active = smoothed > threshold

# Find pulses
transitions = np.diff(active.astype(np.int8))
starts = np.where(transitions == 1)[0]
ends = np.where(transitions == -1)[0]

if len(active) > 0 and active[0]: starts = np.insert(starts, 0, 0)
if len(active) > 0 and active[-1]: ends = np.append(ends, len(active)-1)
min_len = min(len(starts), len(ends))
starts, ends = starts[:min_len], ends[:min_len]

pulses = list(zip(starts, ends))
pulse_lengths = [e - s for s, e in pulses]

print(f"Total smoothed pulses: {len(pulses)}")
if pulses:
    # Meaningful pulses > 500 samples (~11ms)
    filtered_indices = [i for i, l in enumerate(pulse_lengths) if l > 500]
    meaningful_lengths = [pulse_lengths[i] for i in filtered_indices]
    print(f"Meaningful pulses (>500 samples): {len(meaningful_lengths)}")
    
    if len(meaningful_lengths) > 5:
        sorted_m = sorted(meaningful_lengths)
        dot_val = sorted_m[int(len(sorted_m)*0.3)]
        dash_val = sorted_m[int(len(sorted_m)*0.8)]
        print(f"Estimated dot: {dot_val}, dash: {dash_val}")
        mid = (dot_val + dash_val) / 2
        
        morse_string = ""
        for idx in range(len(filtered_indices)):
            i = filtered_indices[idx]
            l = pulse_lengths[i]
            morse_string += "." if l < mid else "-"
            
            if idx < len(filtered_indices) - 1:
                next_i = filtered_indices[idx+1]
                gap = starts[next_i] - ends[i]
                # Morse Timing: element=1u, letter=3u, word=7u
                # Dot is approx 'dot_val'
                if gap > 1.5 * dot_val:
                    if gap > 4 * dot_val:
                        morse_string += "   "
                    else:
                        morse_string += " "
        
        print(f"Morse: {morse_string}")

        MORSE_CODE_DICT = { 'A':'.-', 'B':'-...', 'C':'-.-.', 'D':'-..', 'E':'.', 'F':'..-.', 'G':'--.', 'H':'....', 'I':'..', 'J':'.---', 'K':'-.-', 'L':'.-..', 'M':'--', 'N':'-.', 'O':'---', 'P':'.--.', 'Q':'--.-', 'R':'.-.', 'S':'...', 'T':'-', 'U':'..-', 'V':'...-', 'W':'.--', 'X':'-..-', 'Y':'-.--', 'Z':'--..', '1':'.----', '2':'..---', '3':'...--', '4':'....-', '5':'.....', '6':'-....', '7':'--...', '8':'---..', '9':'----.', '0':'-----'}

        words = []
        for w in morse_string.split("   "):
            chars = ""
            for c in w.split(" "):
                if not c: continue
                found = False
                for k, v in MORSE_CODE_DICT.items():
                    if v == c:
                        chars += k
                        found = True
                        break
                if not found: chars += "?"
            words.append(chars)
        print(f"Decoded: {' '.join(words)}")
```


The processed signal revealed the following Morse sequence:

`-.-. - ..-.   - .... .   .- -- . .-. .. -.-. .- -.   .. ...   .- .-.. .. ...- .`

Decoded text:
`CTF THE AMERICAN IS ALIVE`

The message identifies the trapped prisoner as "THE AMERICAN" (Hopper). Following the challenge format:

### Flag
`CTF{THE AMERICAN IS ALIVE}`
