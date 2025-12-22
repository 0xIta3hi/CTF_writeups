# PatriotCTF '25
Role: Lead Reverse Engineer

Team Impact: Rank Improvement (790 $\to$ 410)

Focus Area: Binary Analysis, Cryptanalysis, Algorithm Reversal

## 1. Executive Summary

This extensive engagement involved dissecting multiple architectures (C, Rust, Go, Python, x86_64 Assembly) to reverse-engineer custom encryption algorithms and bypass authentication mechanisms. The primary challenges centered on "Symmetric Cryptography" (reversible logic) rather than Asymmetric Cryptography (public/private keys). Success relied on a hybrid approach of **Static Analysis** (reading code/assembly) and **Dynamic Analysis** (runtime manipulation).

## 2. Technical Inventory: Skills Acquired

### A. Cryptography & Mathematics

- **Symmetric Encryption:** Mastered the concept of "Inversion." If the code does $X$, the solver must do $X^{-1}$ in reverse order ($LIFO$).
    
- **Bitwise Mechanics:**
    
    - **XOR ($\oplus$):** Exploited the "Self-Inverse" property for encryption and decryption.
        
    - **Rotation (ROL/ROR):** Differentiated between logical Shifts (`<<`) and Circular Rotations, implementing custom Python solvers to replicate 8-bit rotation logic.
        
- **Modular Arithmetic:** Solved "Clock Math" constraints, specifically reversing modulo additions and handling byte wrapping (`& 0xFF`).
    
- **Constraint Solving:** Translated x86 assembly arithmetic instructions (`imul`, `add`, `mod`) into algebraic equations to brute-force integer keys.
    

### B. Static Analysis (Reconnaissance)

- **Binary Forensics:** Utilized `strings` to extract hidden flags and identify compiler artifacts (Little Endian encoding).
    
- **Decompilation:** Used **Ghidra** to translate raw machine code into readable pseudo-C logic.
    
- **Format String Analysis:** Deduced input requirements (`%4s-%d-%10s`) by analyzing format specifiers in the binary's data section.
    
- **Data Sanitization:** Identified and repaired "Data Corruption" caused by ASCII Control Characters (Carriage Returns `\r` overwriting output).
    

### C. Dynamic Analysis (Runtime Engineering)

- **Library Tracing:** Deployed `ltrace` to intercept calls between the binary and the OS (specifically `scanf` and `strcmp`).
    
- **Instruction-Level Debugging:** Used **GDB** to inspect CPU registers (`EAX`, `ECX`) at critical execution points.
    
- **Assembly Literacy:** Identified Control Flow patterns:
    
    - `CMP` (Compare values)
        
    - `JE / JNE` (Conditional Jumps)
        
    - `IMUL` (Complex multiplication/division optimization)
        

## 3. Role Analysis: The Reverse Engineer

Your contribution to the team can be defined by three key activities:

1. **The Translator:** You took "black box" files (EXEs, ELFs, Compiled Python) and translated them into "white box" logic (Python scripts, math equations).
    
2. **The Toolsmith:** Instead of guessing passwords, you built custom tools (solvers) that guaranteed the correct answer mathematically.
    
3. **The Fixer:** When the challenge developers made mistakes (like the memory pointer bug in the Year 2025 challenge), you identified the bug and bypassed it to find the raw data.
    

## 4. Strategic Learnings for Future CTFs

- **"Trust the Assembly, Not the Prompt":** In the Linux challenge, the user prompt lied (`xxxx-xxxx-xxxx`), but the Assembly told the truth (`String-Int-String`).
    
- **"The Truth is in Memory":** When static analysis fails, dynamic analysis (GDB/ltrace) reveals what the computer is _actually_ thinking.
    
- **"Automation Wins":** Manually reversing a 30-byte array is impossible. Writing a script to do it allows for instant adjustments when parameters change (as seen in the jump from Level 2 to Level 3).