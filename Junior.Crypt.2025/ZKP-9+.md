# ZKP 9+ - Zero-Knowledge Proof Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** ZKP 9+
*   **Category:** Cryptography

---

### The Challenge

This challenge required implementing the Schnorr Zero-Knowledge Proof (ZKP) protocol. The goal was to prove knowledge of a secret `x` (a discrete logarithm) without revealing `x` itself.

The server provides the public values `p`, `g`, and `y`, where `y = g^x mod p`. The vulnerability here was that `p` was a very small prime (347), which meant I could easily brute-force `x`.

### The Solution

My approach was to first find the secret `x` by testing all possible values, and then use that `x` to complete the three rounds of the Schnorr protocol.

Here's the Python script I used to solve the challenge:

```python
#!/usr/bin/env python3
import socket
import random
import re

def find_discrete_log(g, y, p):
    """Finds x such that g^x ≡ y (mod p) via brute-force."""
    for x in range(1, p):
        if pow(g, x, p) == y:
            return x
    return None

def solve_zkp():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9049))
        
        # Read the initial banner to get public values
        intro = s.recv(1024).decode()
        p = int(re.search(r'p = (\d+)', intro).group(1))
        g = int(re.search(r'g = (\d+)', intro).group(1))
        y = int(re.search(r'y = (\d+)', intro).group(1))

        # Brute-force the secret 'x'
        x = find_discrete_log(g, y, p)
        print(f"[*] Found secret: x = {x}")

        for i in range(3):
            print(f"\n--- Round {i+1} ---")
            
            # 1. Commitment
            r = random.randint(1, p - 2)
            C = pow(g, r, p)
            s.sendall(f"{C}\n".encode())
            print(f"  > Sent commitment C = {C}")

            # 2. Challenge
            challenge_prompt = s.recv(1024).decode()
            e = int(re.search(r'e = (\d+)', challenge_prompt).group(1))
            print(f"  < Received challenge e = {e}")

            # 3. Response
            s_val = (r + e * x) % (p - 1)
            s.sendall(f"{s_val}\n".encode())
            print(f"  > Sent response s = {s_val}")

        # Receive the flag
        final_response = s.recv(1024).decode()
        flag = re.search(r'grodno{.*?}', final_response).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_zkp()

```

### The Flag

The script successfully completes the three rounds and prints the flag:

```
grodno{6d1c2033a9248bc375df6e7c2fded9}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
