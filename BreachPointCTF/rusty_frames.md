# Writeup: RustyFrames

**Category:** Reverse Engineering / Basic Analysis **Difficulty:** medium 
**Files:** `rusty_frame_S6NV0Zx` (Executable)

## Challenge Overview

We are provided with a binary executable named `rusty_frame_S6NV0Zx`. The goal is to find the correct passphrase to unlock the binary and retrieve the flag. The challenge name "RustyFrames" hints that the binary might be written in Rust, which often results in larger binaries with unique string structures.

---

## 1. Static Analysis: Digging for Strings

As a first step in any binary challenge, I checked for human-readable strings embedded inside the executable. Since Rust binaries can be cluttered, I filtered the output to look for likely keywords, such as "let" (common in Rust syntax or passphrases).

Bash

```
$ strings rusty_frame_S6NV0Zx | grep "let"
let_me_iH3
```

**Observation:** The command revealed an interesting string: `let_me_iH3`. At first glance, this looks like a passphrase, but the suffix `_iH3` seems slightly off. It appears to be a leet-speak or obfuscated version of the phrase "let me in".

---

## 2. Dynamic Analysis: Execution

Based on the static analysis, I decided to run the binary. Instead of using the literal string found (`let_me_iH3`), I hypothesized that the actual password might be the plain English version `let_me_in`, or that the binary was checking for input related to that phrase.

I executed the binary and provided the guess as input.

Bash

```
$ ./rusty_frame_S6NV0Zx
let_me_in
bpctf{stand_proud_mate}
```

**Success!** The binary accepted the input `let_me_in` and immediately printed the flag. It seems `let_me_iH3` was either a red herring left in the strings or a slight variation of the actual key.

---

## 3. Conclusion

The challenge relied on basic string analysis and a small amount of intuition to normalize the found string into a valid passphrase.

**Final Flag:** `bpctf{stand_proud_mate}`