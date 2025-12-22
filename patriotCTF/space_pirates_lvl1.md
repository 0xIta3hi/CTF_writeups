
# 🚩 Challenge 1: Space Pirates - Transmission Decoder (Level 1)

### 1. The Scenario

**Category:** Reverse Engineering **Input:** A C source file (`decoder.c`) containing an encrypted byte array (`TARGET`) and the encryption logic. **Goal:** Decrypt the `TARGET` array to retrieve the flag.

### 2. Static Analysis (Reading the Code)

We started by reading the `main` function in the C code. We identified that the input flag goes through **four distinct operations** to become the encrypted `TARGET`.

The encryption flow in the code was:

1. **XOR** with a rotating 5-byte key.
    
2. **Swap** adjacent bytes (0&1, 2&3, etc.).
    
3. **Add** a magic constant (`0x2A`) modulo 256.
    
4. **XOR** with the current position index.
    

### 3. The Solution Logic

To get the flag, we simply have to reverse the flow. In Reverse Engineering, this means doing the **Inverse Operations** in the **Reverse Order** (Last In, First Out).

![Image of cryptography encryption decryption flow diagram](https://encrypted-tbn2.gstatic.com/licensed-image?q=tbn:ANd9GcSthDVJIIPyYlY-vxT2y0_YdsJkhIVrvCFuobBhsDO_4g7uzQEoIvtYPdzavFVdQGWxZgYizFKeU_keBPTrJldIPa1eXC2rE6FvJqXf3_kTbbAvrZ4)

**The Decryption Stack:**

1. **Reverse Step 4:** `XOR` with Position. _(Inverse of XOR is XOR)_.
    
2. **Reverse Step 3:** `Subtract` the Magic Constant (`0x2A`). _(Inverse of Addition is Subtraction)_.
    
3. **Reverse Step 2:** `Swap` adjacent bytes. _(Swapping twice restores original order)_.
    
4. **Reverse Step 1:** `XOR` with the Rotating Key. _(Inverse of XOR is XOR)_.
    

### 4. The Solver Script (Python)

Here is the Python script that implements the inverse logic:

Python

```
# Space Pirates Level 1 Solver

# 1. Extracted Data
TARGET = [
    0x5A, 0x3D, 0x5B, 0x9C, 0x98, 0x73, 0xAE, 0x32, 0x25, 0x47, 
    0x48, 0x51, 0x6C, 0x71, 0x3A, 0x62, 0xB8, 0x7B, 0x63, 0x57, 
    0x25, 0x89, 0x58, 0xBF, 0x78, 0x34, 0x98, 0x71, 0x68, 0x59
]

XOR_KEY = [0x42, 0x73, 0x21, 0x69, 0x37]
MAGIC_ADD = 0x2A
flag_len = len(TARGET)
buffer = list(TARGET)

print("[*] Reversing Space Pirate Encryption...")

# STEP 1: Reverse the Position XOR (Last operation becomes first)
# Logic: ciphertext ^ index = intermediate
for i in range(flag_len):
    buffer[i] ^= i

# STEP 2: Reverse the Addition (Magic Constant)
# Logic: (x + 0x2A) -> (y - 0x2A)
for i in range(flag_len):
    buffer[i] = (buffer[i] - MAGIC_ADD) & 0xFF

# STEP 3: Reverse the Byte Swaps
# Logic: Swap pairs (0,1), (2,3) back to original
for i in range(0, flag_len, 2):
    buffer[i], buffer[i+1] = buffer[i+1], buffer[i]

# STEP 4: Reverse the Rotating Key XOR
# Logic: intermediate ^ key = plaintext
for i in range(flag_len):
    buffer[i] ^= XOR_KEY[i % 5]

# Convert Integers to ASCII
flag = "".join([chr(x) for x in buffer])
print(f"[*] Flag Found: {flag}")
```

### 5. Key Takeaways

- **Symmetry:** XOR and Swap operations are their own inverses.
    
- **Order Matters:** You must reverse the order of operations (LIFO), not just the operations themselves.
    
- **Mod 256:** When reversing addition, we use `(x - y) & 0xFF` to ensure we handle the byte wrap-around correctly in Python.