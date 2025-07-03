# Interesting WAV - Steganography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Interesting WAV
*   **Category:** Steganography

---

### The Challenge

The description for this challenge was, "sometimes the PreseNt is deeper - not everythinG that sounds needs to be listened to." The capitalized letters `N`, `T`, and `G` immediately hinted at the `PNG` file format, which was a great starting point.

I was given a `stego.wav` file to analyze.

### The Solution

My first step was to run `binwalk` on the `.wav` file to see if anything was hidden inside.

```bash
binwalk stego.wav
```

And just like that, `binwalk` found a JPEG image embedded inside the audio file.

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
44            0x2C            JPEG image data, JFIF standard 1.01
```

Knowing the image was embedded at offset 44, I used `dd` to extract it:

```bash
dd if=stego.wav of=extracted_image.jpg bs=1 skip=44
```

Opening up `extracted_image.jpg` revealed the flag.

### The Flag

```
grodno{WAV_t0_PNG_0r_PNG_t0_WAV}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*
