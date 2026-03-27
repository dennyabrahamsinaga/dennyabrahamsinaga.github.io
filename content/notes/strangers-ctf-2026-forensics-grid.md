---
title: "Stranger's CTF 2026 - Grid"
date: 2026-03-27
draft: true
categories: ["CTF"]
tags: ["ctf", "forensics"]
description: "Writeup for Grid from Stranger's CTF Techtrix '26."
---

### Challenge Information
- Name: Grid
- Category: Forensics
- Difficulty: Easy
- Description:
> Easy once you figure out the trick :)
---

### Initial Analysis

The provided file (1f407) does not have a recognizable extension. Running basic file identification tools does not immediately reveal its format.

However, attempting to load it with NumPy works:

```python
import numpy as np
arr = np.load("1f407")
print(arr.shape)
```

Output:
```shell
(484, 484)
```

This indicates the file is a NumPy array.


### Observations
The array is square (484 × 484). Values are in the set { -1, 0, 1 } and the matrix is symmetric with a zero diagonal

This strongly suggests the matrix represents a graph adjacency or relation matrix.

### Key Insight

Compute the degree (number of connections) for each row:

```python
deg = (arr == 1).sum(axis=1)
print(set(deg))

```
Result:

```shell
{223, 259}
```

There are exactly two degree classes, which implies:
- The nodes are divided into two distinct groups.
- This is effectively a binary labeling.

We can treat this as bits (0/1).

### Recovering the Grid

Since:

484 = 22 × 22

We can reshape the labels into a 22×22 grid.

```python
labels = (deg == 259).astype(int)
grid = labels.reshape(22, 22)
```

### Visualization

To visualize the grid:

```python
from PIL import Image

img = Image.fromarray((grid * 255).astype('uint8'))
img = img.resize((440, 440), Image.NEAREST)
img.save("grid.png")
```

The resulting image reveals a structured barcode-like pattern.

### Identifying the Encoding

The pattern has:
- Solid border on two sides
- Alternating border on the other two sides

This matches the Data Matrix (ECC200) barcode structure.

### Decoding

Remove the outer border:

```python
dm = grid[1:-1, 1:-1]
```

This leaves a 20×20 Data Matrix symbol.

Using any Data Matrix decoder (e.g., zxing, libdmtx, or online tools), the message decodes to:

```bash
CTF{5t4rc0urt_fdCMr}
```

---

### Full Solver
```python
import numpy as np
from PIL import Image

arr = np.load("1f407")

# Compute degree
deg = (arr == 1).sum(axis=1)

# Extract binary labels
labels = (deg == max(deg)).astype(int)

# Reshape into 22x22 grid
grid = labels.reshape(22, 22)

# Save image
img = Image.fromarray((grid * 255).astype('uint8'))
img = img.resize((440, 440), Image.NEAREST)
img.save("grid.png")

print("Flag: CTF{5t4rc0urt_fdCMr}")
```

![Grid](/images/ctf/2026/strangers-ctf-2026/grid.png)

---

Final Flag:  
`CTF{5t4rc0urt_fdCMr}`