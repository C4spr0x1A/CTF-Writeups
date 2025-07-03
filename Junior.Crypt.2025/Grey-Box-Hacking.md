# Grey Box Hacking - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Grey Box Hacking
*   **Category:** Cryptography

---

### The Challenge

This was a fun one. I was given the base64-encoded ciphertext of a flag and a Python script showing how it was encrypted. The encryption method was a 16-bit Linear-Feedback Shift Register (LFSR), but with a catch: both the initial state and the feedback mask were unknown. My job was to reverse-engineer these two values to decrypt the flag.

### The Solution

This was a multi-step process that involved a few different cryptographic techniques.

1.  **Keystream Recovery:** I started by decoding the base64 ciphertext into a stream of bits. In a stream cipher like this, `Ciphertext = Plaintext ⊕ Keystream`. Since I didn't know the plaintext, I couldn't get the full keystream. However, I could make an educated guess about parts of it.

2.  **Finding the Feedback Mask with Berlekamp-Massey:** The Berlekamp-Massey algorithm is a fantastic tool for finding the feedback polynomial of an LFSR if you have a piece of its output stream. I fed parts of my recovered keystream into it until I found a polynomial of degree 16. This polynomial gave me the feedback mask: `0x801F`.

3.  **Brute-Forcing the Initial State:** With the mask now known, the only missing piece was the 16-bit initial state. A 16-bit value is small enough to brute-force (only 65,536 possibilities). I knew the flag started with `grodno{`, so I wrote a script to test every possible initial state. For each state, I would generate the first 7 bytes of the keystream and XOR it with the first 7 bytes of the ciphertext. If the result was `grodno{`, I had found the correct initial state. This revealed the state to be `0xCAF1`.

4.  **Decrypting the Flag:** With the initial state (`0xCAF1`) and the feedback mask (`0x801F`) both known, I could now perfectly replicate the keystream generator. I generated the full keystream, XORed it with the ciphertext, and the flag was revealed.

### The Flag

```
grodno{LFSRs_have_l0ng_been_us3d_as_pseud0-rand0m_number_g3n3rators_for_use_1n_stream_c1phers}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
