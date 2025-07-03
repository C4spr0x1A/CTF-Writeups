# Rainbow in White - Steganography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Rainbow in White
*   **Category:** Steganography

---

### The Challenge

The challenge, "Rainbow in White," provided a BMP image that appeared to be completely white. The description hinted that a "rainbow-colored watermark" was hidden inside, a reference to the fact that white light is a mixture of all colors.

### The Solution

At first glance, the image is just a white square. However, a closer look at the pixel data reveals that while most pixels are pure white (`#FFFFFF`), some are slightly off-white (e.g., `#FEFFFF`, `#FFFFFE`). These almost-white pixels hide the watermark.

My approach was to write a script that would create a new image—a mask—to make these differences visible. The script would iterate through every pixel of the original image. If a pixel was pure white, it would be colored black in the new image. If it was anything other than pure white, it would be colored white.

This process created a new black and white image where the hidden text was clearly legible.

Here's a conceptual Python script using the Pillow library that would do the job:

```python
from PIL import Image

# Open the original image
try:
    img = Image.open("RainbowInWhite.bmp")
    pixels = img.load()
except FileNotFoundError:
    print("Error: RainbowInWhite.bmp not found.")
    exit()

# Create a new image for the mask
mask = Image.new('RGB', img.size)
mask_pixels = mask.load()

# Iterate through each pixel
for i in range(img.width):
    for j in range(img.height):
        # If the pixel is not pure white, make it white in the mask.
        # Otherwise, make it black.
        if pixels[i, j] != (255, 255, 255):
            mask_pixels[i, j] = (255, 255, 255)
        else:
            mask_pixels[i, j] = (0, 0, 0)

# Save the mask
mask.save("watermark_revealed.png")
print("Mask saved as watermark_revealed.png")
```

### The Flag

Opening the generated `watermark_revealed.png` shows the flag written in plain text:

```
grodno{hihihi-hahaha-hohoho-kokoko}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*