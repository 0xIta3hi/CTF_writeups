# Challenge 4: Space Pirates - The Vault (Level 3)

### 1. The Scenario

Category: Reverse Engineering

Input: A Go source file (vault.go) containing a 30-byte encrypted array (target) and the "Ultimate" cipher logic.

Goal: Decrypt the Pirate King's vault combination.

### 2. Analysis (Reading the Go Code)

The Pirate King used the same 6-stage pipeline concept as Level 2 but changed **every single parameter** to throw us off.

**The Upgrades:**

- **Key Length:** Increased to 7 bytes (Prime number, creates better mixing).
    
- **Chunk Size:** Increased to 6 bytes.
    
- **Math:** The position-based XOR became $(i^2 + i) \pmod{256}$.
    
- **Rotation:** A new 8-byte rotation pattern.
    

The Encryption Flow:

1. **XOR** with 7-byte Key.
    
2. **Rotate Left** (Variable amount).
    
3. **Swap** Adjacent Pairs.
    
4. **Subtract** `0x93`.
    
5. **Reverse** Chunks of 6.
    
6. **XOR** with Position Formula $(i^2 + i)$.
    

### 3. The Logic (The Inverse Function)

Despite the upgrades, the strategy remains the same: **LIFO (Last In, First Out)**.

**The Decryption Stack:**

1. **Reverse Step 6:** XOR with $(i^2 + i)$. _(XOR is self-inverse)_.
    
2. **Reverse Step 5:** Reverse Chunks of 6. _(Reversing twice restores order)_.
    
3. **Reverse Step 4:** Add `0x93`. _(Inverse of Subtraction)_.
    
4. **Reverse Step 3:** Swap Adjacent Pairs. _(Self-inverse)_.
    
5. **Reverse Step 2:** Rotate Right. _(Inverse of Rotate Left)_.
    
6. **Reverse Step 1:** XOR with 7-byte Key. _(Self-inverse)_.
    

### 4. The Solver Script (Python)

We updated our generic solver to handle the new parameters.

Python

```
# Space Pirates Level 3 Solver

# 1. Constants
TARGET = [
    0x60, 0x6D, 0x5D, 0x97, 0x2C, 0x04, 0xAF, 0x7C, 0xE2, 0x9E, 
    0x77, 0x85, 0xD1, 0x0F, 0x1D, 0x17, 0xD4, 0x30, 0xB7, 0x48, 
    0xDC, 0x48, 0x36, 0xC1, 0xCA, 0x28, 0xE1, 0x37, 0x58, 0x0F
]
XOR_KEY = [0xC7, 0x2E, 0x89, 0x51, 0xB4, 0x6D, 0x1F]
ROTATION_PATTERN = [7, 5, 3, 1, 6, 4, 2, 0]
MAGIC_SUB = 0x93
CHUNK_SIZE = 6

# Helper: Rotate Right
def ror(val, n):
    n = n % 8
    return ((val >> n) | (val << (8 - n))) & 0xFF

buffer = list(TARGET)
length = len(buffer)

print("[*] Cracking the Pirate King's Vault...")

# STEP 1: Reverse Position Formula XOR
for i in range(length):
    val = ((i * i) + i) % 256
    buffer[i] ^= val

# STEP 2: Reverse Chunk Reversal (Size 6)
for i in range(0, length, CHUNK_SIZE):
    end = min(i + CHUNK_SIZE, length)
    buffer[i:end] = buffer[i:end][::-1]

# STEP 3: Reverse Subtraction (Addition)
for i in range(length):
    buffer[i] = (buffer[i] + MAGIC_SUB) & 0xFF

# STEP 4: Reverse Swaps
for i in range(0, length - 1, 2):
    buffer[i], buffer[i+1] = buffer[i+1], buffer[i]

# STEP 5: Reverse Rotation (Rotate Right)
for i in range(length):
    rot = ROTATION_PATTERN[i % 8]
    buffer[i] = ror(buffer[i], rot)

# STEP 6: Reverse Key XOR
for i in range(length):
    buffer[i] ^= XOR_KEY[i % 7]

flag = "".join([chr(x) for x in buffer])
print(f"[*] Flag: {flag}")
```

### 5. Key Takeaways

- **Complexity vs. Difficulty:** The challenge looked harder because the numbers were bigger and the math scarier, but the _structure_ was identical to Level 2. Once you solve the logic, changing parameters is trivial.
    
- **Mathematical Signatures:** The use of $n^2 + n$ ensures the result is always even. This is a common property used in hashing and checksums.
    
- **Go Syntax:** We learned that Go handles byte slices similarly to Python lists, making the translation straightforward.