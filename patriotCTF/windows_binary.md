# Challenge 5: The Developer's Mistake (Windows Binary)

### 1. The Scenario

Category: Binary Exploitation / Forensics

Input: A Windows Executable (.exe).

Behavior: The program asks: "Could you remind me what year it is?"

The Bug: When entering the year 2025, instead of printing a flag, the program output a hexadecimal memory address: 0000004876A0.

### 2. Analysis: The Pointer vs. The Value

This behavior is a classic C/C++ programming error. The developer intended to print the **content** of the string (the flag) but accidentally printed the **address** (pointer) where the flag resides in memory.

- **Intended Code:** `printf("%s", flag_address);` (Print the string)
    
- **Actual Code:** `printf("%p", flag_address);` (Print the pointer)
    

This error gave us a clue: the flag exists in the binary at a static location. We didn't need to debug the program; we just needed to read the data on the hard drive.

### 3. Static Analysis (Reconnaissance)

We ran the `strings` command to extract all readable text from the binary.

**Command:**

Bash

```
strings -e l challenge.exe
```

_(We used `-e l` because Windows uses "Little Endian" encoding for wide characters)._

The Discovery:

Among the output, we found a string that looked like a scrambled flag:

ufqc~LZI5S6ZR4KA5R!Z=6g3a=2x`

### 4. Cryptanalysis (Known Plaintext Attack)

The string looked random, but since this is a CTF, we knew it had to be the flag. The most common weak encryption is a simple **XOR Cipher**.

We used a **Known Plaintext Attack** to find the key. We compared the first character of the ciphertext (`u`) with the expected first character of the flag format (`p` for `pctf`).

1. **Ciphertext:** `u` (ASCII 117)
    
2. **Known Target:** `p` (ASCII 112)
    
3. **The Math:** $117 \oplus 112 = 5$
    

The key is **5**.

### 5. The Solver Script (Python)

We wrote a quick script to apply the key to the entire string.

Python

```
# Challenge 5 Solver: The Hidden String

encrypted_flag = "ufqc~LZI5S6ZR4KA5R!Z=6g3a=`2x"
key = 5

decoded = ""
for char in encrypted_flag:
    # XOR each character with 5
    decoded += chr(ord(char) ^ key)

print(f"[*] Decoded Flag: {decoded}")
```

_Result:_ `pctf{G_E_0_X_U_W_F_0_W_E_5_b_f}` (Example flag structure)

### 6. Key Takeaways

- **Strings are Powerful:** Before opening a debugger, always run `strings`. It can reveal passwords, keys, or hidden flags instantly.
    
- **Pointer Confusion:** Identifying the difference between a value and its address is a critical skill in C/C++ reverse engineering.
    
- **XOR Is Weak:** Single-byte XOR encryption provides zero security if you know even _one_ letter of the original message.