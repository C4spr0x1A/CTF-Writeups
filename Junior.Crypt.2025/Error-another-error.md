# Error, another error ... - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Error, another error ...
*   **Category:** Cryptography

---

### The Challenge

The challenge description, "My smart server stopped correcting data errors. Although before it could find a bad bit in a 7-bit block," immediately pointed towards Hamming codes. Specifically, the Hamming(7,4) code, which can detect and correct a single-bit error in a 7-bit message.

The task was to connect to a server, receive 20 corrupted 7-bit Hamming codes, correct them, and send back the error position and the original 4-bit data.

### The Solution

The core of the solution is to implement a Hamming(7,4) decoder. For each 7-bit code received, the decoder calculates three syndrome bits (`s1`, `s2`, `s3`). These bits, when combined, form a number that indicates the position of the error. If the position is 0, there is no error.

Once the error is found (or determined to be absent), the original 4 data bits can be extracted from their positions in the 7-bit code.

I wrote this Python script to solve it:

```python
#!/usr/bin/env python3
import socket
import re

def solve_hamming():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9057))
        
        for _ in range(20):
            # Receive the data from the server
            data = s.recv(1024).decode()
            
            # Extract the 7-bit code
            match = re.search(r'(\d{7})', data)
            if not match:
                continue
                
            code_str = match.group(1)
            bits = [int(b) for b in code_str]
            
            # Calculate syndrome bits
            p1, p2, d1, p3, d2, d3, d4 = bits
            s1 = p1 ^ d1 ^ d2 ^ d4
            s2 = p2 ^ d1 ^ d3 ^ d4
            s3 = p3 ^ d2 ^ d3 ^ d4
            
            # Determine error position (1-based)
            error_pos = (s3 * 4) + (s2 * 2) + s1
            
            # Correct the bit if there's an error
            if error_pos > 0:
                bits[error_pos - 1] ^= 1

            # Extract the original 4 data bits
            original_data = f"{bits[2]}{bits[4]}{bits[5]}{bits[6]}"
            
            # Send the result back (0-based error position)
            response = f"{error_pos - 1 if error_pos > 0 else 0}:{original_data}\n"
            s.sendall(response.encode())
            
        # Get the flag
        flag_data = s.recv(1024).decode()
        flag = re.search(r'grodno{.*?}', flag_data).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_hamming()

```

### The Flag

After successfully correcting all 20 codes, the server provides the flag:

```
grodno{fd5220870658230148241247122142403583402f2eda}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
