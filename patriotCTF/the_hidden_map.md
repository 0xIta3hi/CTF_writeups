# Challenge 2: The Hidden Map (Python Decompilation)

### 1. The Scenario

Category: Reverse Engineering / Forensics

Input: A compiled Python file (.pyc) and a large array of negative integers (output = [-90, -42...]).

Goal: Decode the array to reveal a map, fix the formatting, and traverse the maze to find the flag.

### 2. Analysis & Reconnaissance

We started by inspecting the raw data in the `output` array.

- **Observation:** All numbers were negative.
    
- **Hypothesis:** These are negated ASCII codes. For example, `-90` $\rightarrow$ `90` $\rightarrow$ `'Z'`.
    
- **The Glitch:** When we simply converted them to characters and printed them, we got a "Collapsed" output—a few lines of a map followed by a single line of gibberish overwriting itself.
    

The Culprit: ASCII 13 (\r)

The array contained the number -13. In ASCII, 13 is a Carriage Return (CR). It moves the cursor to the start of the current line but does not advance to the next line. This caused the map to overwrite itself repeatedly.

### 3. The "Unfolding" Solution

To fix the map, we had to perform two repairs:

1. **Sanitization:** Replace all instances of `13` with `10` (`\n` or Newline) to properly stack the rows.
    
2. **Formatting:** The bottom half of the map was actually a single long line (because the original code missed the newlines entirely). We discovered the top half had a width of **28 characters**, so we forced the bottom half to wrap every 28 characters.
    

### 4. The Pathfinding

Once the map was reconstructed, it revealed a maze:

- **Walls:** `Z`, `*`, `[`
    
- **Path:** `'` (top half) and `&` (bottom half).
    
- **Objective:** Enter at the `0` (top left), follow the path, and collect every unique character encountered.
    

### 5. The Solver Script

This script performs the decoding, reconstruction, and automated pathfinding to extract the flag.

Python

```
# Challenge 2 Solver: The Hidden Map

import sys

# 1. The Raw Data (Truncated for brevity)
output = [-90, -42, -39, -42, -39, -39, ... , -91] 

# 2. Decode & Reconstruct Map
print("[*] Decoding and Reconstructing Map...")
raw_string = ""
for num in output:
    val = abs(num)
    # Fix Carriage Return (13) -> Newline (10)
    raw_string += "\n" if val == 13 else chr(val)

# Split lines and fix the "Collapsed" bottom section
lines = raw_string.split('\n')
grid_lines = []
for line in lines:
    if len(line) > 40:
        # Force wrap the collapsed line at width 28
        for i in range(0, len(line), 28):
            grid_lines.append(line[i:i+28])
    else:
        grid_lines.append(line)

# Pad to make a perfect rectangle for the solver
max_width = max(len(line) for line in grid_lines)
grid = [line.ljust(max_width, ' ') for line in grid_lines]

# 3. Pathfinding Logic (DFS)
# Walls: Z, *, [, ]
WALLS = {"Z", "*", "[", "]"}
visited = set()
path_log = []

# Find Start ('0')
start_pos = None
for r, line in enumerate(grid):
    if '0' in line:
        start_pos = (r, line.find('0'))
        break

def solve(pos):
    r, c = pos
    char = grid[r][c]
    
    # Collect character if it's not a path marker
    if char not in ["&", "'", "+", "|", "-"]:
        path_log.append(char)
        
    visited.add(pos)
    
    # Try moves: Left, Down, Right, Up
    moves = [(0, -1), (1, 0), (0, 1), (-1, 0)]
    for dr, dc in moves:
        nr, nc = r + dr, c + dc
        if 0 <= nr < len(grid) and 0 <= nc < len(grid[nr]):
            if grid[nr][nc] not in WALLS and (nr, nc) not in visited:
                solve((nr, nc))
                return # Follow the single valid path

solve(start_pos)
flag = "".join(path_log)
print(f"[*] Clean Flag: PCTF{{{flag}}}")
```

### 6. Key Takeaways

- **Data Sanitization:** Always check if your data contains Control Characters (`\r`, `\t`, `\b`) that might be hiding information.
    
- **Visualization:** When output looks wrong (one long line), check for fixed-width patterns. The "28 character width" was the key to unlocking the second half of the maze.
    
- **DFS (Depth First Search):** Simple maze challenges can be solved programmatically by treating characters as nodes in a graph.

* Note: Although i couldn't solve this challenge as the flag generated was wrong. Yet this is the closet to the flag that i got. :) 
