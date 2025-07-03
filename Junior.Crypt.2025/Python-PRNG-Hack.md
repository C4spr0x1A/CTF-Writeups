# Python PRNG Hack - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Python PRNG Hack
*   **Category:** Cryptography

---

### The Challenge

This challenge targeted the pseudo-random number generator (PRNG) in Python's `random` module. The server would generate random 32-bit numbers, and after requesting up to 1000 of them, the goal was to predict the very next one.

The vulnerability lies in the fact that Python's `random` module uses the Mersenne Twister (MT19937) algorithm, which is not cryptographically secure. After observing 624 consecutive outputs, its internal state can be fully recovered, making all future "random" numbers completely predictable.

### The Solution

My plan was to collect 624 random numbers from the server and then use a known MT19937 state recovery script to predict the 625th number.

Here is the Python script I used. It leverages a class `MT19937Predictor` (not shown here, but widely available) that can reconstruct the generator's state.

```python
#!/usr/bin/env python3
import socket
import re
from mt19937predictor import MT19937Predictor # A known implementation for MT19937 state recovery

def solve_prng():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9043))
        
        # We need exactly 624 numbers to predict the state.
        numbers = []
        for _ in range(624):
            s.sendall(b"1\n") # Request a number
            response = s.recv(1024).decode()
            numbers.append(int(re.search(r'(\d+)', response).group(1)))

        # Use the predictor to find the next number.
        predictor = MT19937Predictor()
        for num in numbers:
            predictor.setrandbits(num, 32)
            
        prediction = predictor.getrandbits(32)
        
        # Send the prediction to get the flag.
        s.sendall(b"2\n") # Choose to guess
        s.recv(1024) # Consume the prompt
        s.sendall(f"{prediction}\n".encode())
        
        flag_response = s.recv(1024).decode()
        flag = re.search(r'grodno{.*?}', flag_response).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_prng()
```

### The Flag

The prediction was correct, and the server responded with the flag:

```
grodno{8d3f5043644adfecb56464646adef0ede}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
