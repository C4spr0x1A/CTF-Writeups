# CTF Writeup: Unstoppable force meets immovable object

**Author:** Mohamed Armaoui - C4spr0x1A

## Challenge Details
| Category       | Cryptography |
|----------------|--------------|
| Challenge Name | Unstoppable force meets immovable object |
| CTF Event      | Blitz CTF 2025 |
| Flag           | `blitz{60nn4_b3_4_b16_c0ll1510n_wh3n_un570pp4bl3_f0rc3_m3375_1mm0v4bl3_0bj3c7}` |

## Solution

This writeup details my approach to solving the "Unstoppable force meets immovable object" CTF challenge.

### Vulnerability Analysis

The challenge involved a Flask web application that used a custom hashing function called `immovable_object`. My goal was to provide a password that was *not* equal to a predefined `NOT_PASSWORD` ("P@ssword@123"), but when hashed by `immovable_object`, produced the same hash as `NOT_PASSWORD`.

The `immovable_object` function's logic is as follows:

1.  Padding: If the input data length is not a multiple of `block_size` (which is 32 bytes), it's padded with null bytes (`\0`) until it is.
2.  Block Processing: The padded data is then split into `block_size` (32-byte) blocks.
3.  Integer Conversion: Each 32-byte block is converted into a large integer using `int.from_bytes(block, "big")`.
4.  XOR Sum: All these converted integers are XORed (`^`) together to produce the final hash.

The vulnerability lies in the XOR operation. If I had a target hash `H`, and I knew `H = B1 ^ B2 ^ ... ^ Bn`, I could craft a new input `P'` such that `immovable_object(P') = H` even if `P' != P`. Specifically, for two blocks `B_a` and `B_b`, I knew that `B_a ^ B_b` would produce a certain hash. If I wanted this to equal `H`, and I chose an arbitrary `B_a'`, then the second block `B_b'` must satisfy `B_a' ^ B_b' = H`. This implied `B_b' = H ^ B_a'`.

In this challenge, `NOT_PASSWORD` ("P@ssword@123") is 14 bytes long. When passed to `immovable_object`, it's padded to 32 bytes and treated as a single block. I called the integer representation of this padded block `TARGET_HASH`.

My strategy was to create a new password `crafted_password` composed of two 32-byte blocks, `BLOCK1` and `BLOCK2`, such that `int(BLOCK1) ^ int(BLOCK2) = TARGET_HASH`. I could choose an arbitrary `BLOCK1` (e.g., 32 'A' characters), then calculate `BLOCK2` as `TARGET_HASH ^ int(BLOCK1)`.

### Exploitation Steps

1.  Analyze `main.py`: I understood the `immovable_object` function and the condition for obtaining the flag: `password != NOT_PASSWORD` and `immovable_object(password) == immovable_object(NOT_PASSWORD)`.

2.  Calculate `TARGET_HASH`: I determined the hash produced by `NOT_PASSWORD` ("P@ssword@123") using the `immovable_object` function. Since "P@ssword@123" is 14 bytes, it got padded with 18 null bytes to become a single 32-byte block.

3.  Craft `BLOCK1`: I chose an arbitrary 32-byte sequence for the first block. For simplicity, I used `b'A' * 32`.

4.  Calculate `BLOCK2`: I computed `BLOCK2_int = TARGET_HASH ^ int(BLOCK1)`. Then I converted `BLOCK2_int` back to a 32-byte sequence.

5.  Construct `crafted_password_bytes`: I concatenated `BLOCK1` and `BLOCK2` to form the full byte sequence of my crafted password: `crafted_password_bytes = BLOCK1_bytes + BLOCK2_bytes`.

6.  URL-encode the crafted password: Since the crafted password contained non-printable bytes, it was crucial to URL-encode it before sending it in a POST request. This ensured that the exact byte sequence was preserved during transmission.

The following Python script was used to generate the URL-encoded password:

```python
import urllib.parse

NOT_PASSWORD = 'P@ssword@123'.encode('utf-8')
block_size = 32

def immovable_object(data, block_size=32):
    if len(data) % block_size != 0:
        data += b'\0' * (block_size - (len(data) % block_size))
    h = 0
    for i in range(0, len(data), block_size):
        block = int.from_bytes(data[i : i + block_size], 'big')
        h ^= block
    return h

# Calculate the target hash from NOT_PASSWORD
target_hash = immovable_object(NOT_PASSWORD)

# Choose an arbitrary 32-byte block for block1. Using 'A'*32.
block1_bytes = b'A' * block_size
block1_int = int.from_bytes(block1_bytes, 'big')

# Calculate block2_int such that block1_int ^ block2_int = target_hash
block2_int = target_hash ^ block1_int

# Convert block2_int back to bytes
block2_bytes = block2_int.to_bytes(block_size, 'big')

# Combine block1 and block2 to form the new password bytes
crafted_password_bytes = block1_bytes + block2_bytes

# URL-encode the crafted password bytes
url_encoded_password = urllib.parse.quote_plus(crafted_password_bytes)

print(f'{url_encoded_password}')
```

Running this script produced the URL-encoded password:

`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA%11%01226.3%25%01psrAAAAAAAAAAAAAAAAAAAA`

7.  Send POST Request: I used curl to send a POST request with the crafted and URL-encoded password to the challenge URL.

    `curl -X POST -d "password=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA%11%01226.3%25%01psrAAAAAAAAAAAAAAAAAAAA" https://ufmio-n1sj9nsb.blitzhack.xyz/`

## Flag

The server responded with the flag:

`blitz{60nn4_b3_4_b16_c0ll1510n_wh3n_un570pp4bl3_f0rc3_m3375_1mm0v4bl3_0bj3c7}`
