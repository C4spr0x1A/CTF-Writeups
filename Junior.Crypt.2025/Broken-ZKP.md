# Broken ZKP - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Broken ZKP. I
*   **Category:** Cryptography

---

### The Challenge

This challenge involved a "Broken" Zero-Knowledge Proof. It used the Schnorr ZKP protocol, but the implementation had a fatal flaw: the server reused the same secret nonce `r` for every round of the proof. In a secure implementation, `r` must be a freshly generated random number for each round. This reuse makes it possible to recover the secret `x` that the protocol is supposed to be protecting.

### The Solution

The reuse of `r` allows an attacker to perform an algebraic attack. By participating in two rounds of the protocol, I can set up a system of two equations with two unknowns (`r` and `x`).

The equations for two rounds are:
1.  `s1 = (r + e1 * x) mod (p-1)`
2.  `s2 = (r + e2 * x) mod (p-1)`

By subtracting the two equations, I can eliminate `r` and solve for `x`:
*   `s1 - s2 = (e1 - e2) * x mod (p-1)`
*   `x = (s1 - s2) * mod_inverse(e1 - e2, p-1) mod (p-1)`

The clever part of the exploit is how to get the `s1` and `s2` values. I can just send *any* incorrect value for `s` for the first two rounds. The server will reject my proof, but in its error message, it helpfully tells me what the *correct* `s` value should have been.

So, the plan is:
1.  Connect and get the public parameters (`p`, `g`, `y`).
2.  Start the first round. Send a commitment `C`. Receive the challenge `e1`. Send a dummy response for `s`.
3.  Parse the server's failure message to get the correct `s1`.
4.  Do the same for the second round to get `e2` and `s2`.
5.  Use the formula above to calculate `x`.
6.  Complete the remaining rounds of the protocol (failing them is fine) and then submit the calculated `x` to get the flag.

Here's the Python script to do it:

```python
#!/usr/bin/env python3
import socket
import re

def solve_broken_zkp():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9050))

        # Get public parameters
        intro = s.recv(1024).decode()
        p = int(re.search(r'p = (\d+)', intro).group(1))
        
        # --- Round 1: Get s1 and e1 ---
        s.sendall(b'1\n') # Send dummy commitment
        round1_prompt = s.recv(1024).decode()
        e1 = int(re.search(r'e = (\d+)', round1_prompt).group(1))
        s.sendall(b'1\n') # Send dummy s
        round1_fail = s.recv(1024).decode()
        s1 = int(re.search(r's = (\d+)', round1_fail).group(1))
        
        # --- Round 2: Get s2 and e2 ---
        s.sendall(b'1\n') # Send dummy commitment
        round2_prompt = s.recv(1024).decode()
        e2 = int(re.search(r'e = (\d+)', round2_prompt).group(1))
        s.sendall(b'1\n') # Send dummy s
        round2_fail = s.recv(1024).decode()
        s2 = int(re.search(r's = (\d+)', round2_fail).group(1))

        # --- Calculate x ---
        p_minus_1 = p - 1
        s_diff = (s1 - s2) % p_minus_1
        e_diff_inv = pow(e1 - e2, -1, p_minus_1)
        x = (s_diff * e_diff_inv) % p_minus_1

        # --- Get the flag ---
        # We can just fail the rest of the rounds
        for _ in range(3):
            s.sendall(b'1\n')
            s.recv(1024)
            s.sendall(b'1\n')
            s.recv(1024)
            
        # Submit the calculated x
        final_prompt = s.recv(1024).decode()
        s.sendall(f"{x}\n".encode())
        
        flag_response = s.recv(1024).decode()
        flag = re.search(r'grodno{.*?}', flag_response).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_broken_zkp()
```

### The Flag

```
grodno{3dc080adcbsyjfjrieei9943cj23efeeec}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
