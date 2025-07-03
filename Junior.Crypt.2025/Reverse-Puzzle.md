# Reverse Puzzle - Reverse Engineering Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Reverse Puzzle
*   **Category:** Reverse Engineering

---

### The Challenge

This challenge gave me a Python script with a function called `puzzle`. This function takes a string and rearranges it a given number of times. The goal was to find the input string that, after being processed by `puzzle(input, 5)`, would result in a specific target string.

The `puzzle` function works by taking all the characters at even indices, followed by all the characters at odd indices, and joining them together.

### The Solution

To solve this, I had to reverse-engineer the `puzzle` function. If the forward function takes `even + odd`, the reverse function must take the result, split it in half, and then "weave" the two halves back together into their original even and odd positions.

I wrote a `reverse_puzzle` function to do just that. Then, I applied this reverse function 5 times to the target string provided in the challenge.

Here is the Python script I used to find the flag:

```python
#!/usr/bin/env python3

def puzzle(s: str, step: int) -> str:
    """The forward puzzle function from the challenge."""
    if step == 0:
        return s
    return puzzle(s[::2] + s[1::2], step - 1)

def reverse_puzzle_step(s: str) -> str:
    """Reverses a single step of the puzzle."""
    n = len(s)
    mid = (n + 1) // 2
    
    first_half = s[:mid]
    second_half = s[mid:]
    
    res = [''] * n
    # Weave the first half into even positions
    for i, char in enumerate(first_half):
        res[i * 2] = char
    # Weave the second half into odd positions
    for i, char in enumerate(second_half):
        res[i * 2 + 1] = char
        
    return "".join(res)

def reverse_puzzle(s: str, steps: int) -> str:
    """Reverses the full puzzle function for a given number of steps."""
    current_s = s
    for _ in range(steps):
        current_s = reverse_puzzle_step(current_s)
    return current_s

if __name__ == "__main__":
    target = '789603251257384214725442633'
    
    # Reverse the target string 5 times to get the original flag content
    flag_content = reverse_puzzle(target, 5)
    
    print(f"[*] Target string: {target}")
    print(f"[+] Reversed content: {flag_content}")
    print(f"\n[!] Full flag: grodno{{{flag_content}}}")
    
    # Verify the result
    assert puzzle(flag_content, 5) == target
    print("\n[*] Verification successful!")

```

### The Flag

The script successfully reverses the puzzle and reveals the flag:

```
grodno{774248325798612643250235431}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*