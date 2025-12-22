# Challenge 6: The License Key (Linux Binary)

### 1. The Scenario

**Category:** Reverse Engineering **Input:** A Linux Executable (ELF binary). **Behavior:** The program prompts for a "License Key" from page 3 of a booklet. **The Prompt:** `Please enter the license key in the format xxxx-xxxx-xxxx`

### 2. Static Analysis (The Blueprint)

We started with the `strings` command. It revealed a crucial Format String that contradicted the user prompt.

**Discovery:** `%4s-%d-%10s` This `scanf` format string gave us the exact structure of the key:

1. **Part 1:** A 4-character string (`%4s`).
    
2. **Part 2:** A decimal integer (`%d`).
    
3. **Part 3:** A 10-character string (`%10s`).
   

We also found two candidate strings in the binary: `PTE1` (4 chars) and `PatriotCTF` (10 chars). This gave us a working theory for the key structure: `PTE1-[INT]-PatriotCTF`.

### 3. Dynamic Analysis (The Trap)

We ran `ltrace` (Library Trace) to watch the program check the password.

- It confirmed the `scanf` call reading our input.
    
- However, it **did not** show any `strcmp` or check for the integer.
    
- **Conclusion:** The integer validation is done via internal CPU math instructions, not library functions.
    

### 4. Deep Dive: GDB & Assembly

We opened the binary in **GDB** and disassembled the `main` function (`disas main`). This revealed two secrets that static analysis missed.

**Secret A: The Real First String** At `main+92`, we saw the code manually checking characters one by one against hex values:

- `cmp al, 0x43` ('C')
    
- `cmp al, 0x41` ('A')
    
- `cmp al, 0x43` ('C')
    
- `cmp al, 0x49` ('I') The real first string was **`CACI`**, not `PTE1`.
    

**Secret B: The Math Equation** At `main+154`, we found a block of assembly performing arithmetic on the integer input (`[rbp-0x18]`). It wasn't a simple comparison; it was an equation:

1. **LHS:** `(Input + 22) % 1738`
    
2. **RHS:** `(((Input * 2) % 2000) * 6) + 9`
    
3. **Check:** `cmp ecx, eax` (Are they equal?)
    

### 5. The Solver Script (Python)

Since solving `(x+22)%1738 == ...` manually is painful, we wrote a script to brute-force the integer within the bounds defined by the assembly (`-5000` to `10000`).

Python

```
# Challenge 6 Solver: The Integer Math

print("[*] Brute-forcing the License Key Integer...")

# Range defined in assembly: -5000 to 10000
for x in range(-5000, 10001):
    
    # Logic derived from Assembly instructions
    lhs = (x + 22) % 1738
    
    # Note: imul and shifts in assembly translated to this modulo logic
    rhs = (((x * 2) % 2000) * 6) + 9
    
    if lhs == rhs:
        print(f"[*] Match Found: {x}")
        print(f"[*] Final Key: CACI-{x}-PatriotCTF")
        break
```

### 6. Key Takeaways

- **Trust Code, Not Prompts:** The user prompt said `xxxx-xxxx-xxxx`, but the code actually enforced `String-Int-String`.
    
- **Red Herrings:** The binary contained `PTE1` (likely a leftover string), but the actual check looked for `CACI`. Only assembly analysis revealed the truth.
    
- **The `ltrace` Blind Spot:** Library tracing is powerful but cannot see internal math. When `ltrace` goes silent, you must switch to GDB.
    
- **Assembly Math:** You don't need to be a mathematician. You just need to translate assembly instructions (`add`, `imul`, `idiv`) into a Python script and let the computer solve it.