# 🚩 CTF Write-ups & Exploitation Scripts

**Author:** 0xIta3hi 
**Focus:** Web Security | Reverse Engineering | Binary Exploitation (Pwn) | Cloud Security

---

## 📖 About This Repository

This repository serves as a comprehensive archive of my journey through various Capture The Flag (CTF) competitions. It contains detailed write-ups, custom solver scripts, and post-mortem analysis of challenges I have solved.

The goal is not just to store flags, but to document the **methodology**, **tooling**, and **thought processes** used to break security mechanisms. 

### 🏆 Recent Highlights
* **PatriotCTF / Space Pirates:** Served as Lead Reverse Engineer, automating asymmetric cipher decryption and analyzing binaries to help propel the team rank from **#790 to #410**.

---

## 📂 Repository Structure

The repository is organized by CTF Event, then by Category:

```text
├── CTF_Event_Name/
│   ├── Web/
│   │   ├── Challenge_Name/
│   │   │   ├── solver.py       # Automated exploit script
│   │   │   ├── analysis.md     # Detailed write-up
│   │   │   └── artifacts/      # Source code or relevant files
│   │   └── ...
│   ├── Reverse_Engineering/
│   │   ├── Challenge_Name/
│   │   │   ├── decoder.py      # Decryption logic
│   │   │   └── notes.md        # Static/Dynamic analysis logs
│   │   └── ...
│   └── Pwn/
│       └── ...
└── Scripts/                    # General purpose helper tools
````

---

## 🛠️ Technical Competencies

### 🕸️ Web Exploitation

- **Vulnerability Analysis:** Injection (SQLi, XSS, SSTI), Broken Access Control, and IDOR.
    
- **Authentication Bypass:** JWT manipulation, session fixation, and logic flaws.
    
- **Tooling:** Burp Suite Professional, OWASP ZAP, Python `requests`.
    

### ⚙️ Reverse Engineering (Rev)

- **Static Analysis:** Disassembling binaries (ELF/PE) using **Ghidra** and **IDA** to reconstruct logic flow.
    
- **Algorithmic Reversal:** Identifying and inverting custom symmetric encryption (XOR, Bitwise Rotation, Modular Arithmetic).
    
- **File format Analysis:** Analyzing format strings and memory structures in C, Rust, and Go binaries.
    

### 💥 Binary Exploitation (Pwn)

- **Memory Corruption:** Stack buffer overflows, format string vulnerabilities.
    
- **Dynamic Debugging:** Runtime analysis using **GDB (GEF/Pwndbg)** and **ltrace** to inspect registers and memory states.
    
- **Exploit Development:** Writing Python scripts with `pwntools` to automate payload delivery and interaction.
    

---

## 🧰 The Toolkit

The following tools are heavily utilized across the write-ups in this repo:

|**Category**|**Tools**|
|---|---|
|**Recon & Web**|`nmap`, `Burp Suite`, `ffuf`, `curl`|
|**Reverse Engineering**|`Ghidra`, `strings`, `uncompyle6` (Python), `DnSpy` (.NET)|
|**Debugging & Pwn**|`GDB` (with GEF), `ltrace`, `strace`, `Checksec`|
|**Scripting**|`Python 3`, `Pwntools`, `Z3 Solver` (Constraint Solving)|

---

## 📝 Featured Write-ups

### [PatriotCTF / Space Pirates]

- **Level 3 (Go Vault):** Reversing a custom 6-stage bitwise cipher in Go using Python.
    
- **License Key (Linux):** Using GDB to uncover hidden algebraic constraints in an ELF binary that `ltrace` missed.
    
- **Memory Leak (Windows):** Exploiting a format string error to read raw memory pointers.
    

_(More write-ups added regularly)_

---

## ⚠️ Disclaimer

All information and scripts in this repository are for **educational purposes only**. They are intended to help security researchers and developers understand vulnerabilities to better secure systems. Do not use these techniques on systems where you do not have explicit permission.