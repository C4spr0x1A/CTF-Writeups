# NotGeometry - Forensics Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** S = H * W ?
*   **Category:** Forensics

---

### The Challenge

This challenge, titled "S = H * W ?", provided a BMP file named `NotGeometry.bmp`. The title was a dead giveaway: the relationship between the image's size, height, and width was incorrect.

Inspecting the file's properties revealed that the reported dimensions (247x235) would result in an image size of about 58KB, but the actual file contained over 121KB of pixel data. The header was lying.

### The Solution

The technique here is a form of steganography where the "hidden" data is not in the pixels themselves, but in the file's structure. Most image viewers read the header, see the incorrect dimensions, and either fail to render the image or show a cropped/distorted version.

My approach was to:
1.  Extract the raw pixel data from the BMP, ignoring the header.
2.  Determine the correct dimensions by factoring the size of the raw pixel data (121,768 bytes). A bit of trial and error showed that the dimensions were 248x491.
3.  Create a new, valid BMP header with the correct dimensions (248x491).
4.  Combine the new header with the original pixel data to create a valid BMP file.

Here is a Python script that would perform the header-patching:

```python
import struct

# Read the original file
with open('NotGeometry.bmp', 'rb') as f:
    data = f.read()

# The pixel data starts after the header, at offset 1078
header = data[:1078]
pixels = data[1078:]

# The actual image size is the length of the pixel data
actual_size = len(pixels) # 121768

# Find the correct dimensions (248x491)
new_width = 248
new_height = 491

# Create a mutable byte array for the new header
new_header = bytearray(header)

# Pack the correct width and height into the header (little-endian)
struct.pack_into('<I', new_header, 18, new_width)  # Width at offset 18
struct.pack_into('<I', new_header, 22, new_height) # Height at offset 22

# Write the new header and original pixels to a new file
with open('corrected.bmp', 'wb') as f:
    f.write(new_header)
    f.write(pixels)

print("Corrected BMP saved as corrected.bmp")
```

### The Flag

Opening `corrected.bmp` in any image viewer reveals the flag written in the image:

```
grodno{0100101001010101001010}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*