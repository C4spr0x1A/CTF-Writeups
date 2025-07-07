# CTF Writeup: Operator

**Author:** Mohamed Armaoui - C4spr0x1A

## Challenge Details
| Category       | Reverse Engineering |
|----------------|---------------------|
| Challenge Name | Operator            |
| CTF Event      | Cube CTF 2025       |
| Flag           | `cube{c00l_0p3r4t0rs_us3_mult1_st4g3_p4yl04ds_8ab49338}` |

## Solution

This write-up details my approach to solving the "Operator" CTF challenge, which involved analyzing a pcap file to find a hidden flag.

### 1. Initial Analysis

The challenge provided a single file, `operator.pcap`. My first step was to analyze this file to understand the network traffic it contained. Using `tshark`, I identified two significant TCP streams on port 2025:

*   Stream 7: Appeared to be a file transfer.
*   Stream 29: Contained what looked like encrypted data.

### 2. Investigating the Encrypted Data (Stream 29)

My initial focus was on the encrypted data in stream 29. I extracted its hex data and made several unsuccessful decryption attempts. I tried keys based on the "cube{...}" flag format and the challenge name "operator," and also performed a single-byte XOR bruteforce. None of these methods yielded readable plaintext, leading to the belief that I was missing the correct key or decryption algorithm.

### 3. Reconstructing the Executable (Stream 7)

I then shifted my attention to stream 7, hypothesizing that it contained the executable responsible for the encryption. My initial attempts to extract the binary using standard tools like `tshark` and `tcpflow` resulted in a corrupted file.

To correctly reconstruct the binary, I followed these steps:

1.  Extract the raw hex payload from stream 7:
        tshark -r operator.pcap -Y 'tcp.stream eq 7' -T fields -e tcp.payload | tr -d '\n' > stream7.hex
    
2.  Convert the hex dump to a binary file:
        xxd -r -p stream7.hex xcat.bin
    
This process successfully created a valid ELF executable named `xcat.bin`.

### 4. Analyzing the xcat.bin Binary

With a working binary, I analyzed it to find the encryption logic and key.

1.  Disassembly: Using `objdump`, I found a function named `xor_encrypt`.
        objdump -d xcat.bin
    
    The disassembly of the `xor_encrypt` function showed a simple XOR operation.

2.  Finding the Key: I discovered a static 16-byte key (`040717764269b00bde1823221eedf7ae`) in the `.data` section at address `0x4010`.

### 5. The Breakthrough: Per-Packet Decryption

Armed with the correct key, I updated my Python decryption script. When I ran it on the entire data from stream 29 (concatenated), it only decrypted the first line of the communication: "Hi, is this a secure line?". The rest of the data remained garbled.

Further analysis of the binary's chat function showed that the same static key was used for both sending and receiving data within a loop. The encryption index was reset for each `send` and `recv` call. This indicated that the XOR cipher was applied to each message (each TCP packet's payload) individually, rather than continuously across the entire stream.

The solution was to decrypt each packet's payload from stream 29 as a separate message.

### 6. The Solution

I created a final Python script (`solve4.py`) to implement this per-packet decryption logic.

```python
def solve():
    hex_messages = [
        "4c6e3b562b1a907fb67150027fcd84cb677265136205d965bb2729",
        "4d276403300c9063b16846026d82fd",
        "506f72052749d179bb38504d7388d7d861756e56310cde78b76c4a547bcd99c17062645631069042fe6f424c6acd83c124657256311cc26efe6c4b4767ca85cb24697802620cc87bb16b464614",
        "45696e0123109c2bb67d5147399ed7c37d2763193249c36ebd6a46563e8499c86b757a173600df65e412",
        "67727513390a803bb24713522d9fc3da34756429371a8354b36d4f562fb284da30602429325dc967ee2c475141d596cc303e24457a14ba",
        "4d277f19320c9065b17a4c4667cd91c76a6364563601d17ff0360d28"
    ]
    
    key = bytes.fromhex("040717764269b00bde1823221eedf7ae")
    print(f"Key length: {len(key)}")

    for i, hex_data in enumerate(hex_messages):
        data = bytes.fromhex(hex_data)
        decrypted_data = bytearray()
        for j in range(len(data)):
            decrypted_data.append(data[j] ^ key[j % len(key)])
        
        print(f"--- Decrypted message {i+1} ---")
        print(decrypted_data.decode('utf-8', errors='ignore'))

if __name__ == "__main__":
    solve()
```

Running this script produced the full decrypted conversation:

```
Key length: 16
--- Decrypted message 1 ---
Hi, is this a secure line?

--- Decrypted message 2 ---
I sure hope so

--- Decrypted message 3 ---
These are some very sensitive notes so I want to be sure they're not exposed

--- Decrypted message 4 ---
Anyway, here's my top secret information:

--- Decrypted message 5 ---
cube{c00l_0p3r4t0rs_us3_mult1_st4g3_p4yl04ds_8ab49338}

--- Decrypted message 6 ---
I hope nobody finds that...
```