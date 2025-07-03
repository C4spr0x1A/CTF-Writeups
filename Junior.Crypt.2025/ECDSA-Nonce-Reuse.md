# ECDSA Nonce Reuse - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Signature ECDSA
*   **Category:** Cryptography

---

### The Challenge

This challenge demonstrated a classic cryptographic mistake: reusing a nonce in ECDSA signatures. The server signed two different messages but used the same random nonce `k` for both. This is a fatal flaw that allows for the complete recovery of the private key.

The server provided all the necessary components of the two signatures (`r`, `s1`, `s2`) and the hashes of the two messages (`h1`, `h2`).

### The Solution

The ECDSA signature algorithm involves a series of equations. When the nonce `k` is reused, we can set up a system of two equations and solve for the private key `d`.

1.  **The two signature equations are:**
    *   `s1 = k_inv * (h1 + r * d) mod n`
    *   `s2 = k_inv * (h2 + r * d) mod n`

2.  **Subtract the two equations to solve for `k`:**
    *   `s1 - s2 = k_inv * (h1 - h2) mod n`
    *   `k = (h1 - h2) * mod_inverse(s1 - s2, n) mod n`

3.  **Substitute `k` back into one of the original equations to solve for `d`:**
    *   `d = (s1 * k - h1) * mod_inverse(r, n) mod n`

I implemented this with the following Python script:

```python
#!/usr/bin/env python3

# Values from the challenge
n = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141
r = 0xe37ce11f44951a60da61977e3aadb42c5705d31363d42b5988a8b0141cb2f50d
s1 = 0xdf88df0b8b3cc27eedddc4f3a1ecfb55e63c94739e003c1a56397ba261ba381d
h1 = 0x315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3
s2 = 0x2291d4ab9e8b0c412d74fb4918f57580b5165f8732fd278e65c802ff8be86f61
h2 = 0xa6ab91893bbd50903679eb6f0d5364dba7ec12cd3ccc6b06dfb04c044e43d300

# Step 1: Recover the nonce k
s_diff_inv = pow(s1 - s2, -1, n)
k = ((h1 - h2) * s_diff_inv) % n

# Step 2: Recover the private key d
r_inv = pow(r, -1, n)
d = ((s1 * k - h1) * r_inv) % n

# The private key is the flag in hex
flag_hex = hex(d)
flag = bytes.fromhex(flag_hex[2:]).decode('utf-8')

print(f"[!] Private key (as hex): {flag_hex}")
print(f"[!] Flag: {flag}")
```

### The Flag

Running the script reveals the private key, which, when converted from hex to ASCII, is the flag:

```
grodno{My_pr1vate_key_f0r_ECDSA}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*