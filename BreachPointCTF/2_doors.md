## 🚪 Writeup: 2 Door (Misc/OSINT)

### 1. The Initial Lead

We were provided with a ChatGPT share link: `https://chatgpt.com/share/6990cbe8-60d4-8010-ac07-7e9c6eb9d98f`

However, the initial link was a **Dead End**. It likely led to a 404 page or a generic conversation that didn't contain the flag.

### 2. The URL Manipulation

**Hint:** "Swap the last two letters." The last two characters of the ID were `8f`. By swapping them to `f8`, we modified the unique resource identifier:

- **Original:** `...ac07-7e9c6eb9d98f`
    
- **Modified:** `...ac07-7e9c6eb9d98f8` (Wait, the instructions said swap, so `f8`).
    
- **Corrected Link:** `https://chatgpt.com/share/6990cbe8-60d4-8010-ac07-7e9c6eb9d9f8`
    

This new link successfully "unlocked" the second door, leading to a hidden conversation page.

### 3. The Digital Trail

The hidden chat provided specific coordinates for a target on **Discord**:

- **Target User:** Seshan
    
- **Role:** Moderator
    
- **Handle:** `vividverse1625`
    
- **Instruction:** Check the Profile Picture (Avatar).
    

### 4. Steganography / Visual Inspection

Following the trail to Discord, we located the user `vividverse1625`. In OSINT challenges, flags are often hidden in the **Profile Picture (PFP)** using one of two methods:

1. **Visual Inspection:** The flag is written in tiny text or hidden in the background colors of the image.
    
2. **Metadata/Steganography:** The image file itself contains the flag in the strings or EXIF data.
    

By inspecting the avatar of **Seshan**, the flag was revealed.

**Flag:** `BPCTF{birthday_gift_by_atlee}`
