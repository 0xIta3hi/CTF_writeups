**Category:** Reverse Engineering

**Difficulty:** Easy/Medium

**Files:** `rollingsilence` (ELF64 executable, stripped)

## Challenge Overview

We are given a stripped ELF64 binary named `rollingsilence`. The prompt hints that the binary seems "silent and harmless" with no visible input or output, but performs a "rolling transformation" in memory.

Our goal is to reverse engineer the internal state changes to uncover a hidden flag.

---

## 1. Initial Reconnaissance

First, I checked the file type and basic properties to understand what I was dealing with.

Bash

```
$ file rollingsilence
rollingsilence: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped
```

Since the binary is **stripped**, we won't have helpful function names like `main`. I used `readelf` to find the entry point.

Bash

```
$ readelf -h rollingsilence
Entry point:                0x401000
```

There is only a single executable section (`.text`). This suggests a very small, hand-written assembly program rather than a standard C compilation.

---

## 2. Static Analysis & Assembly Logic

I disassembled the binary starting at the entry point `0x401000`.

Bash

```
$ objdump -d rollingsilence | head -25
```

The output reveals a tight loop. Here is the annotated breakdown of the assembly:

Code snippet

```
401000: 48 31 ff        xor    %rdi,%rdi       ; i = 0 (loop index)
401003: b0 89           mov    $0x89,%al       ; Initial state = 0x89

; --- LOOP START ---
401005: 48 83 ff 1a     cmp    $0x1a,%rdi      ; Compare i with 26 (Flag Length)
401009: 7d 14           jge    0x40101f        ; If i >= 26, jump to exit

; 1. Load Encrypted Byte
40100b: 8a 9f 00 20     mov    0x402000(%rdi),%bl  ; bl = encrypted[i]

; 2. Decrypt
401011: 30 c3           xor    %al,%bl         ; bl = bl ^ state (This is the flag byte!)

; 3. Update State for Next Iteration
401013: 40 00 f8        add    %dil,%al        ; state += i (lower 8 bits of index)
401016: d0 c0           rol    $1,%al          ; state = RotateLeft(state, 1)
401018: 34 a5           xor    $0xa5,%al       ; state ^= 0xA5

; 4. Increment and Continue
40101a: 48 ff c7        inc    %rdi            ; i++
40101d: eb e6           jmp    0x401005        ; Jump back to start
```

### The Logic

The binary implements a state-machine based decryption.

1. **Ciphertext Location:** The data is pulled from `0x402000`.
    
2. **Decryption:** The flag byte is simply the ciphertext byte XORed with the current `state`.
    
3. **State Evolution:** The `state` (held in the `AL` register) changes after every byte using three operations:
    
    - Add the current index.
        
    - Bitwise Rotate Left (ROL) by 1.
        
    - XOR with static key `0xA5`.
        

---

## 3. Extracting the Ciphertext

The assembly pointed to memory address `0x402000` as the source of the encrypted data. I used GDB to dump these bytes.

Bash

```
$ gdb ./rollingsilence
(gdb) x/26bx 0x402000
0x402000: 0xeb 0xc6 0xa9 0x48 0xbd 0x61 0xe9 0x83
0x402008: 0x19 0xc1 0xb5 0x70 0xde 0x58 0xb8 0x49
0x402010: 0x8e 0x28 0x16 0xf1 0x55 0xa5 0x09 0x2d
0x402018: 0x20 0x62
```

**Encrypted Array:** `[0xeb, 0xc6, 0xa9, 0x48, 0xbd, 0x61, 0xe9, 0x83, 0x19, 0xc1, 0xb5, 0x70, 0xde, 0x58, 0xb8, 0x49, 0x8e, 0x28, 0x16, 0xf1, 0x55, 0xa5, 0x09, 0x2d, 0x20, 0x62]`

---

## 4. The Solution Script

Since the logic is deterministic, we can reimplement the assembly in Python to recover the flag.

_Note: Python doesn't have a native `ROL` (Rotate Left) instruction for 8-bit integers, so we simulate it using bitwise shifts._



```python
#!/usr/bin/env python3

# The raw bytes extracted from 0x402000
encrypted = [
    0xeb, 0xc6, 0xa9, 0x48, 0xbd, 0x61, 0xe9, 0x83, 
    0x19, 0xc1, 0xb5, 0x70, 0xde, 0x58, 0xb8, 0x49, 
    0x8e, 0x28, 0x16, 0xf1, 0x55, 0xa5, 0x09, 0x2d, 
    0x20, 0x62
]

# Initial State from 0x401003
state = 0x89
flag = ""

print(f"[*] Starting decryption with initial state: {hex(state)}")

for i in range(len(encrypted)):
    # 1. Decrypt: ciphertext ^ state
    dec_byte = encrypted[i] ^ state
    flag += chr(dec_byte)
    
    # 2. Update State (Exact mirror of assembly)
    # 401013: add %dil, %al
    state = (state + i) & 0xFF 
    
    # 401016: rol $1, %al
    # ((state << 1) | (state >> 7)) simulates 8-bit rotation
    state = ((state << 1) | (state >> 7)) & 0xFF
    
    # 401018: xor $0xa5, %al
    state ^= 0xA5

print(f"\n[+] Flag Found: {flag}")
```

---

## 5. Result

Running the script decrypts the flag perfectly.

**Output:**

Plaintext

```
[*] Starting decryption with initial state: 0x89

[+] Flag Found: bpctf{registers_are_state}
```

The challenge demonstrated how "silent" binaries can hide logic purely in register state transitions without ever calling standard I/O functions.

**Final Flag:** `bpctf{registers_are_state}`