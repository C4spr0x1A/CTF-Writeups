# Intel order VS Network order - Forensics Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Intel order VS Network order
*   **Category:** Forensics

---

### The Challenge

This challenge provided a broken BMP image file named `task_for_001.bmp`. The title "Intel order VS Network order" was a huge clue, pointing directly to an issue with byte order (endianness). Intel processors use little-endian byte order, while network byte order is big-endian.

### The Solution

When I tried to open the image, it failed. Inspecting the file's hex data revealed the problem. A valid BMP file has a header that specifies its properties. At offset 14, there's a 4-byte value for the DIB header size, which should be 40 (or `0x28`).

The bytes in the file were `00 00 00 28` (big-endian), but a BMP reader expects `28 00 00 00` (little-endian).

The fix was to patch this single value. I used `dd` to overwrite the incorrect bytes with the correct ones:

```bash
# Create a copy to work with
cp task_for_001.bmp fixed.bmp

# Overwrite the bytes at offset 14 with the little-endian value for 40
printf '\x28\x00\x00\x00' | dd of=fixed.bmp bs=1 seek=14 conv=notrunc
```

After this simple patch, the `fixed.bmp` file opened perfectly in an image viewer.

### The Flag

The flag was written directly on the image:

```
grodno{y3s,_we_ch4ng3_h3ader_0f_bmp-f1le}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)* 