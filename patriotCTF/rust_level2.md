# Challenge 3: Space Pirates - Treasure Map (Level 2)

### 1. The Scenario

**Category:** Reverse Engineering **Input:** A Rust source file (`decoder.rs`) containing an encrypted 32-byte array (`TARGET`) and the logic to decrypt it. **Goal:** Reverse the "Advanced Pirate Cipher" to recover the map coordinates.

### 2. Analysis (Reading the Rust)

The Rust code implemented a 6-stage encryption pipeline. Unlike C, Rust is memory-safe and strict, but the logic remains sequential.

The encryption flow was:

1. **XOR** with a rotating 5-byte key.
    
2. **Rotate Left** by a variable amount (based on position mod 7).
    
3. **Swap** adjacent byte pairs.
    
4. **Subtract** a magic constant (`0x5D`).
    
5. **Reverse** bytes in chunks of 5.
    
6. **XOR** with Position Squared (`i*i % 256`).
    

### 3. The Logic (The Inverse Function)

To solve this, we applied the standard "Inverse Order, Inverse Logic" principle.

**The Decryption Stack:**

1. **Reverse Step 6:** `XOR` with Position Squared. _(Self-inverse)_.
    
2. **Reverse Step 5:** `Reverse` chunks of 5. _(Reversing a list twice restores it)_.
    
3. **Reverse Step 4:** `Add` the Magic Constant (`0x5D`). _(Inverse of Subtraction)_.
    
4. **Reverse Step 3:** `Swap` adjacent byte pairs. _(Self-inverse)_.
    
5. **Reverse Step 2:** `Rotate Right`. _(The inverse of Rotate Left is Rotate Right)_.
    
6. **Reverse Step 1:** `XOR` with the Rotating Key. _(Self-inverse)_.
    

**The "Gotcha": Bitwise Rotation** Standard Python doesn't have a `rotate` function for integers. We had to implement a custom helper to simulate 8-bit rotation: `((val >> n) | (val << (8 - n))) & 0xFF`

### 4. The Solver Script (Python)

Python

```
# Space Pirates Level 2 Solver

# 1. Constants
TARGET = [
    0x15, 0x5A, 0xAC, 0xF6, 0x36, 0x22, 0x3B, 0x52, 0x6C, 0x4F, 
    0x90, 0xD9, 0x35, 0x63, 0xF8, 0x0E, 0x02, 0x33, 0xB0, 0xF1, 
    0xB7, 0x69, 0x42, 0x67, 0x25, 0xEA, 0x96, 0x63, 0x1B, 0xA7, 
    0x03, 0x0B
]
XOR_KEY = [0x7E, 0x33, 0x91, 0x4C, 0xA5]
ROTATION_PATTERN = [1, 3, 5, 7, 2, 4, 6]
MAGIC_SUB = 0x5D

# Helper: Rotate Right (Inverse of Rotate Left)
def ror(val, n):
    n = n % 8 
    return ((val >> n) | (val << (8 - n))) & 0xFF

buffer = list(TARGET)
length = len(buffer)

print("[*] Decrypting Level 2...")

# STEP 1: Reverse Position XOR
for i in range(length):
    buffer[i] ^= (i * i) % 256

# STEP 2: Reverse Chunk Reversal (Size 5)
for i in range(0, length, 5):
    end = min(i + 5, length)
    buffer[i:end] = buffer[i:end][::-1]

# STEP 3: Reverse Subtraction (Addition)
for i in range(length):
    buffer[i] = (buffer[i] + MAGIC_SUB) & 0xFF

# STEP 4: Reverse Swaps
for i in range(0, length, 2):
    buffer[i], buffer[i+1] = buffer[i+1], buffer[i]

# STEP 5: Reverse Rotation (Rotate Right)
for i in range(length):
    rot = ROTATION_PATTERN[i % 7]
    buffer[i] = ror(buffer[i], rot)

# STEP 6: Reverse Key XOR
for i in range(length):
    buffer[i] ^= XOR_KEY[i % 5]

flag = "".join([chr(x) for x in buffer])
print(f"[*] Flag: {flag}")
```

### 5. Key Takeaways

- **Bitwise Operations:** Understanding how data bits move (Shift vs. Rotation) is critical for low-level reverse engineering.
    
- **Custom Helpers:** When your language (Python) lacks a feature present in the target language (Rust's `rotate_left`), you must implement the mathematical equivalent yourself.
    
- **Chunking:** Encryption often processes data in blocks. Recognizing block patterns (like "every 5 bytes") helps break down the problem.