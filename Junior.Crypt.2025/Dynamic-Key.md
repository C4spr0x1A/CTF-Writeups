# Dynamic Key - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Dynamic Key
*   **Category:** Cryptography

---

### The Challenge

This challenge involved a custom encryption scheme where the key was "dynamic," meaning it was generated based on the current system time. I was given the Python script that performed the encryption and a static, encrypted version of the flag's content. The goal was to find the original flag.

### The Solution

The "dynamic key" was a bit of a red herring. While the key was generated from the timestamp, the formula `((int(time.time()) % 256) ^ 0xFF) & 0x7F` meant that the key could only be one of 128 possible values (0-127).

Since the encrypted flag was static and the key space was so small, I didn't need to worry about the time at all. I could simply brute-force all 128 keys.

My plan was to:
1.  Implement a `decrypt` function that reversed the provided `encrypt` function.
2.  Loop through every possible key from 0 to 127.
3.  For each key, decrypt the target ciphertext and print the result.
4.  Visually inspect the output for something that looked like a flag.

Here's the Python script I used:

```python
#!/usr/bin/env python3

def decrypt(encrypted_bytes, key):
    """Reverses the encryption function from the challenge."""
    decrypted_chars = []
    for i, byte_val in enumerate(encrypted_bytes):
        # Reverse the original formula: (c + key) ^ (i * 2) = byte_val
        original_char_code = ((byte_val ^ (i * 2)) - key) & 0xFF
        decrypted_chars.append(chr(original_char_code))
    return ''.join(decrypted_chars)

# The target ciphertext from the challenge script
expected = b'\x74\xab\x9a\x62\x95\x6b\x9f\x81\x6b\x87\xbd\x99\x81\xb9\x93\x98\xb5\x80\x8d\xa9\x5b\x4a\xb1\x8e\xac\xa7\x9c\xb9\xa9\xa4\xa8\xb1\x39\xdc\xd7\x26\xd5\xea\xee\xdb\xc8\xc7\xca\xf5\x39\xc8\xc0\xcb'

print("[*] Brute-forcing all 128 possible keys...")
for key in range(128):
    decrypted = decrypt(expected, key)
    # A simple check for printable characters and underscores
    if all(c.isprintable() or c == '_' for c in decrypted):
        print(f"  [+] Key {key}: grodno{{{decrypted}}}")

```

### The Flag

Running the script produced a few candidates, but only one made sense. For key `48`, the output was the flag:

```
grodno{Dyn4m1c_Key_is_Very_C0mplex_and_Inc0mprehens1ble}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
