# Homomorphic Weak - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Homomorphic Weak
*   **Category:** Cryptography

---

### The Challenge

This challenge involved a Paillier homomorphic cryptosystem. The server provided the public key `(n, g)` and a ciphertext `c`. The vulnerability was in the key generation: the two primes, `p` and `q`, used to create the modulus `n` were too close to each other. This weakness allows for the factorization of `n` using Fermat's factorization method.

### The Solution

Once `n` is factored into `p` and `q`, the private key can be reconstructed, and the ciphertext can be decrypted. I wrote a Python script to automate this process.

The script connects to the server, parses the public key and ciphertext, factors `n` to find `p` and `q`, and then uses them to decrypt the message and retrieve the flag.

Here is the full script:

```python
#!/usr/bin/env python3

import socket
import re
from Crypto.Util.number import long_to_bytes
from math import gcd, isqrt

def connect_and_get_values():
    """Connect to the server and extract n, g, and c."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9055))
        data = s.recv(4096).decode('utf-8', errors='ignore')
        
        n = int(re.search(r'n = (\d+)', data).group(1))
        g = int(re.search(r'g = (\d+)', data).group(1))
        c = int(re.search(r'c = (\d+)', data).group(1))
        
        return n, g, c

def fermat_factor(n):
    """
    Factor a number n into p and q using Fermat's factorization method.
    This is efficient when p and q are close.
    """
    a = isqrt(n)
    if a * a < n:
        a += 1
    
    while True:
        b2 = a * a - n
        if b2 >= 0:
            b = isqrt(b2)
            if b * b == b2:
                p = a - b
                q = a + b
                return p, q
        a += 1

def paillier_decrypt(c, n, p, q):
    """Decrypt a Paillier-encrypted ciphertext."""
    
    g = n + 1
    l = (p - 1) * (q - 1) // gcd(p - 1, q - 1)
    
    # L(x) = (x - 1) // n
    u = pow(g, l, n * n)
    L_u = (u - 1) // n
    mu = pow(L_u, -1, n)
    
    m_pow = pow(c, l, n * n)
    L_m = (m_pow - 1) // n
    
    m = (L_m * mu) % n
    return m

if __name__ == "__main__":
    n, g, c = connect_and_get_values()
    
    print(f"[*] Factoring n = {n}")
    p, q = fermat_factor(n)
    print(f"[+] Found factors:\n  p = {p}\n  q = {q}")
    
    decrypted_message = paillier_decrypt(c, n, p, q)
    flag = long_to_bytes(decrypted_message)
    
    print(f"\n[!] Flag: {flag.decode()}")

```

### The Flag

Running the script gives the flag:

```
grodno{Crypto_Weak_Modulus_Factorization}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
