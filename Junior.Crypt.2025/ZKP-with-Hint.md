# ZKP with Hint - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** ZKP with Hint
*   **Category:** Cryptography

---

### The Challenge

This challenge presented another flawed Schnorr Zero-Knowledge Proof. This time, the server was trying to be "helpful" by providing a hint: for each round, it would leak the lower 464 bits of the 512-bit secret nonce `r`. This hint is a critical vulnerability that allows an attacker to recover the full nonce, and ultimately, the secret `x`.

### The Solution

The 464-bit leak leaves only the upper 48 bits of `r` unknown. This reduces the problem of finding `r` to a "small-exponent" discrete logarithm problem, which is solvable.

**Step 1: Recovering the Full Nonce `r`**

Let `r_leak` be the known 464 bits, and `r_high` be the unknown 48 bits. The full nonce is `r = r_high * 2^464 + r_leak`.

The server gives us `C = g^r mod p`. We can rewrite this as:
*   `C ≡ g^(r_high * 2^464 + r_leak) (mod p)`
*   `C * g^(-r_leak) ≡ (g^(2^464))^r_high (mod p)`

Let `C_prime = C * mod_inverse(g^r_leak, p)` and `g_prime = g^(2^464) mod p`. The equation becomes `C_prime ≡ (g_prime)^r_high (mod p)`.

We now have a discrete logarithm problem where we need to find `r_high`, and we know it's less than `2^48`. This is small enough to be solved efficiently using the **Baby-Step Giant-Step (BSGS)** algorithm.

**Step 2: Recovering the Secret `x`**

The exploit proceeds in two rounds:

**Round 1: Find `x`**
1.  Connect to the server and get the public values (`p`, `g`, `y`) and the first round's values (`C1`, `leak1`, `e1`).
2.  Use the BSGS algorithm to solve for `r_high1` and construct the full nonce `r1`.
3.  Intentionally fail the proof by sending a dummy value for `s`. The server responds with the *correct* `s1`.
4.  Rearrange the Schnorr equation to solve for the secret `x`:
    *   `s1 = (r1 + e1 * x) mod (p-1)`
    *   `x = (s1 - r1) * mod_inverse(e1, p-1) mod (p-1)`

**Round 2: Forge the Proof**
1.  For the second round, the server gives us new values: `C2`, `leak2`, `e2`.
2.  Repeat the BSGS attack to find the full nonce `r2`.
3.  Since we now know both `x` (from Round 1) and `r2`, we can correctly calculate the proof for this round:
    *   `s2 = (r2 + e2 * x) mod (p-1)`
4.  Send the correct `s2`. The server will verify it and give us the flag.

### The Flag

```
grodno{4def9010456729462826404cdadfhghdfj467af9ee5}
```

This challenge was a great example of how even a small information leak can completely break a cryptographic protocol.

---

*Written by Mohamed Armaoui (C4spr0x1A)*