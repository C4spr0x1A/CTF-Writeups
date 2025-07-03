# White Cat in a White Room - Steganography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** White Cat in a White Room
*   **Category:** Steganography

---

### The Challenge

The title of this challenge, "White Cat in a White Room," and the description, "It will be easier to find a white cat in a white room if you paint the cat black," were perfect clues. I was given a BMP image file that appeared to be completely white.

### The Solution

My initial analysis showed that the file was a 1-bit monochrome BMP. A 1-bit image has a very simple color palette: one color for the `0` bits and one color for the `1` bits.

The trick was that the creator of the challenge had edited the BMP's color palette in the file header. Both colors were set to white (`#FFFFFF`). This meant that no matter what the actual pixel data was, the image would always render as a blank white canvas.

The solution was to "paint the cat black," just as the hint suggested. I used a hex editor to change one of the color entries in the palette from white to black.

Here's how I could do it on the command line with `dd`:

```bash
# Make a copy to work on
cp CatInRoom.bmp flag.bmp

# The color palette in a 1-bit BMP starts at offset 54 (0x36).
# The first color is at offset 54, the second is at 58.
# We will overwrite the first color (4 bytes) with black (0x00000000).
printf '\x00\x00\x00\x00' | dd of=flag.bmp bs=1 seek=54 conv=notrunc
```

After this patch, I opened `flag.bmp` in an image viewer, and the hidden flag was clearly visible as text.

### The Flag

```
grodno{4374575035158325358345436}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
