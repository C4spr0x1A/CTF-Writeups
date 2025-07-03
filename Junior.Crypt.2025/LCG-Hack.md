# LCG Hack - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** LCG Hack
*   **Category:** Cryptography

---

### The Challenge

This challenge involved breaking a Linear Congruential Generator (LCG), a common algorithm for generating pseudo-random numbers. The server would provide up to 50 consecutive numbers from an LCG, and the goal was to predict the next one.

The LCG formula is: `X_n+1 = (a * X_n + b) % m`

The key vulnerability was in the server's implementation: the initial seed, `X_0`, was set to be equal to the modulus, `m`. This meant the very first number the server sent was the modulus itself.

### The Solution

With the modulus `m` known, and the constants `a` and `b` provided in the server's source code, predicting the rest of the sequence was trivial.

I wrote a Python script to:
1.  Connect to the server.
2.  Request the first three numbers to be sure. The first number is our modulus `m`.
3.  Use the last received number and the known `a`, `b`, and `m` to calculate the next number in the sequence.
4.  Send the predicted number back to the server to get the flag.

Here is the script:

```python
#!/usr/bin/env python3
import socket
import re

def solve_lcg():
    # Constants from the server's source code
    a = 2**15 - 1
    b = 2**51 - 1

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9042))
        
        # We only need one number to get the modulus, but we'll get a few
        # to confirm our theory.
        s.sendall(b"3\n")
        response = s.recv(1024).decode()
        
        numbers = [int(n) for n in re.findall(r'\d+', response)]
        
        # The first number is the modulus
        m = numbers[0]
        
        # The last number we received
        last_number = numbers[-1]
        
        # Predict the next number
        prediction = (a * last_number + b) % m
        
        # Send the prediction
        s.sendall(f"{prediction}\n".encode())
        
        # Get the flag
        flag_response = s.recv(1024).decode()
        flag = re.search(r'grodno{.*?}', flag_response).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_lcg()
```

### The Flag

The server accepted the prediction and returned the flag:

```
grodno{3d3d50436448239239fe54646adbf0ed5}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
