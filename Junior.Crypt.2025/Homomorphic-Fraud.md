# Homomorphic Fraud - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Homomorphic Fraud
*   **Category:** Cryptography

---

### The Challenge

A bank server uses the Paillier homomorphic encryption scheme to protect customer balances. The server allows two actions: viewing the encrypted balance and increasing it. The vulnerability lies in the second option: the server blindly accepts any encrypted value from the client and adds it to the current balance, without verifying what that value is.

Paillier's additive homomorphic property means: `Enc(m1) * Enc(m2) = Enc(m1 + m2)`.

The goal was to set the account balance to exactly 1,000,000.

### The Solution

My strategy was a two-step process:

1.  **Reset the balance to zero:** I requested the current encrypted balance, `Enc(balance)`, and calculated its modular inverse. Sending this inverse back to the server resulted in `Enc(balance) * Enc(-balance) = Enc(0)`. The balance was now at an encrypted zero.

2.  **Set the balance to 1,000,000:** I created a new ciphertext for the number 1,000,000. Since I didn't have the full private key, I used a randomness factor `r=1`. Sending this to the server resulted in `Enc(0) * Enc(1000000) = Enc(1000000)`.

Here is the Python script that automates the attack:

```python
#!/usr/bin/env python3
import socket
import re

def solve_homomorphic_fraud():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect(('ctf.mf.grsu.by', 9054))
        
        # --- Step 1: Reset balance to zero ---
        
        # Receive the banner and get public key and encrypted balance
        initial_data = s.recv(4096).decode()
        n = int(re.search(r'\((\d+),', initial_data).group(1))
        g = int(re.search(r', (\d+)\)', initial_data).group(1))
        enc_balance = int(re.search(r'Encrypted Balance = (\d+)', initial_data).group(1))
        n2 = n * n
        
        # Calculate and send the modular inverse to "add" -balance
        inv_enc = pow(enc_balance, -1, n2)
        s.sendall(f"{inv_enc}\n".encode())
        
        # --- Step 2: Set balance to 1,000,000 ---
        
        # Wait for the next prompt
        s.recv(4096)
        
        # Create a ciphertext for 1,000,000 with r=1 and send it
        target_amount = 1000000
        enc_target = pow(g, target_amount, n2)
        s.sendall(f"{enc_target}\n".encode())
        
        # Receive the flag
        flag_response = s.recv(4096).decode()
        flag = re.search(r'grodno{.*?}', flag_response).group(0)
        print(f"\n[!] Flag: {flag}")

if __name__ == "__main__":
    solve_homomorphic_fraud()
```

### The Flag

The server decrypted the final balance, confirmed it was 1,000,000, and sent back the flag:

```
grodno{6de150435442404854572843420932985434f0ed5}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)* 