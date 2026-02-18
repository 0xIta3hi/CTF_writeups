#  Crypto Write-Up: _Biased Stream Cipher (DockerJail Crypto)_

## 📌 Challenge Summary

The challenge provided a Python script that outputs a hex-encoded `output.bin`.  
At first glance, the ciphertext looked random and secure, but the description hinted at a **biased stream** and **known beginnings**, suggesting a weak stream cipher.

---

## 🔍 Given Code Analysis

```python
class A:
    def __init__(s):
        s.s = 0x37
    
    def g(s):
        t = s.s
        u = (t >> 7) & 1
        s.s = ((t << 1) ^ ((0xB8) & (-u))) & 255
        return s.s
```

This is an **8-bit Linear Feedback Shift Register (LFSR)**:

- State size: 8 bits
    
- Initial seed: `0x37`
    
- Feedback polynomial: `0xB8`
    
- Output depends on the **MSB (most significant bit)**
    

LFSRs are **not cryptographically secure**, especially when:

- The output is biased
    
- Only one bit of the state is leaked
    
- The keystream is reused or predictable
    

---

## 🧠 Key Insight

The cipher does **not output full random bytes**.  
Instead, it leaks **only the MSB**, which is then:

- Expanded to a full byte (`0x00` or `0xFF`)
    
- XORed with the plaintext
    

This creates a **biased keystream**, which is a well-known cryptographic weakness.

---

## 🧪 Attack Strategy

We reverse the encryption step-by-step:

### 1️⃣ Rebuild the LFSR

```python
class LFSR:
    def __init__(self):
        self.s = 0x37

    def step(self):
        t = self.s
        out = (t >> 7) & 1
        self.s = ((t << 1) ^ (0xB8 if out else 0)) & 0xFF
        return out
```

---

### 2️⃣ Handle Ambiguities

There were **three unknowns**:

|Variable|Reason|
|---|---|
|Polarity|`0 → 0x00` or `0 → 0xFF`|
|Offset|Stream may start mid-cycle|
|Extra XOR|Obfuscation layer|

So we brute-forced:

- Both polarities
    
- All 8 offsets
    
- A final single-byte XOR
    

---

### 3️⃣ Detect Valid Plaintext

We filtered results by:

- Printable ASCII ratio
    
- Presence of `flag{}` or `ctf{}`
    

---

## ✅ Final Solver Script

```python
import string


class LFSR:
    def __init__(self):
        self.s = 0x37

    def step(self):
        t = self.s
        out = (t >> 7) & 1
        self.s = ((t << 1) ^ (0xB8 if out else 0)) & 0xFF
        return out


def printable_ratio(b):
    return sum(chr(c) in string.printable for c in b) / len(b)


with open("output.bin", "rb") as f:
    ct = f.read()

for polarity in [0, 1]:
    for offset in range(8):
        gen = LFSR()
        for _ in range(offset):
            gen.step()

        keystream = bytes(
            ((gen.step() ^ polarity) * 0xFF)
            for _ in range(len(ct))
        )

        layer1 = bytes(c ^ k for c, k in zip(ct, keystream))

        if printable_ratio(layer1) < 0.6:
            continue

        for k in range(256):
            pt = bytes(c ^ k for c in layer1)
            text = pt.decode(errors="ignore").lower()

            if "flag{" in text or "ctf{" in text:
                print(pt.decode(errors="replace"))
                exit(0)
```

---

## 🏁 Final Result

Running the script reveals the flag:

```
BPCTF{biased_stream_leakage}
```

🎉 **Challenge solved.**

