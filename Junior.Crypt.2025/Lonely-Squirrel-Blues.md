# Lonely Squirrel Blues - Steganography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Lonely Squirrel Blues
*   **Category:** Steganography

---

### The Challenge

This challenge provided an animated GIF of a squirrel playing a banjo. The title, "Lonely Squirrel Blues," and the hint, "Haven't heard of this?" were clever red herrings designed to make you think about audio steganography. However, the key was in the animation itself.

### The Solution

My first step was to examine the GIF's properties. I discovered it was an animation with 5 frames. This was a strong indicator that the data might be split across the frames.

I used `convert` from ImageMagick to extract the 5 frames into individual PNG files:

```bash
convert squirrel_banjo.gif frame_%02d.png
```

Next, I ran `zsteg` on each frame. `zsteg` is a great tool for detecting steganography in PNG and BMP files, and it immediately found hidden data in the Least Significant Bits (LSB) of the image data.

Each frame contained a piece of a Base32-encoded string:

*   **frame_00.png:** `part1:M5ZG6ZDON55US3S7ORUGKX3GNFSWYZC7N5TF62L:`
*   **frame_01.png:** `part2:OMZXXE3LBORUW63S7ONSWG5LSNF2HSLC7JRSWC4:`
*   **frame_02.png:** `part3:3UL5JWSZ3ONFTGSY3BNZ2F6QTJORZV6KCMKNBCS`
*   **frame_03.png:** `part4:X3BNRTW64TJORUG2X3JONPWC3S7NFWXA33SORQW`
*   **frame_04.png:** `part5:45C7ORXXA2LDL5XWMX3ENFZWG5LTONUW63T5`

I concatenated these parts and decoded the resulting Base32 string:

```bash
echo "M5ZG6ZDON55US3S7ORUGKX3GNFSWYZC7N5TF62LOMZXXE3LBORUW63S7ONSWG5LSNF2HSLC7JRSWC43UL5JWSZ3ONFTGSY3BNZ2F6QTJORZV6KCMKNBCSX3BNRTW64TJORUG2X3JONPWC3S7NFWXA33SORQW45C7ORXXA2LDL5XWMX3ENFZWG5LTONUW63T5" | base32 -d
```

### The Flag

The decoded string was the flag:

```
grodno{In_the_field_of_information_security,_Least_Significant_Bits_(LSB)_algorithm_is_an_important_topic_of_discussion}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*