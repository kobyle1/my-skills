---
name: nano-banana-image
description: |
  Create and edit images with Nano Banana 2 (Gemini Image Generation) via Google AI Studio API.
  Use when the user wants to: create an image from text, edit an existing image, enhance a prompt, or save images.

  ALWAYS invoke when the user says: "create image", "generate image", "nano banana", "edit image", "image from text".
---

# Nano Banana 2 - Image Generation Skill

Create and edit images with Nano Banana 2 (Gemini Imagen API).

---

## API Settings

- **Generation model:** imagen-3.0-generate-002
- **Editing model:** gemini-2.0-flash-preview-image-generation
- **Base URL:** https://generativelanguage.googleapis.com/v1beta/models
- **API key:** Read from config file at start of each session

---

## API Key Management

Read the API key from the config file (never ask in chat):

Config file location: C:\Users\[username]\Downloads\nano-banana-config.txt

Python code to read key:
```python
from pathlib import Path

config_path = Path.home() / "Downloads" / "nano-banana-config.txt"
api_key = None

if config_path.exists():
    for line in config_path.read_text().splitlines():
        if line.startswith("NANO_BANANA_API_KEY="):
            api_key = line.split("=", 1)[1].strip()
            break

if not api_key:
    print("API key not found. Open nano-banana-config.txt and add your key.")
    exit(1)
```

---

## Mode 1: Generate Image from Text

```python
import google.generativeai as genai
import sys
from pathlib import Path

api_key = sys.argv[1]
prompt = sys.argv[2]
output_path = sys.argv[3]

genai.configure(api_key=api_key)

response = genai.ImageGenerationModel("imagen-3.0-generate-002").generate_images(
    prompt=prompt,
    number_of_images=1,
    aspect_ratio="1:1",  # 1:1, 9:16, 16:9, 3:4, 4:3
    safety_filter_level="block_few",
    person_generation="allow_adult",
)

image_data = response.images[0]._image_bytes
Path(output_path).write_bytes(image_data)
```

**Supported aspect ratios:**
- 1:1 -- Square (default)
- 9:16 -- Story / Portrait
- 16:9 -- Landscape / Slides
- 3:4 -- Portrait
- 4:3 -- Landscape

---

## Mode 2: Edit Existing Image

```python
import google.generativeai as genai
import base64, sys
from pathlib import Path

api_key = sys.argv[1]
image_path = sys.argv[2]
instruction = sys.argv[3]
output_path = sys.argv[4]

genai.configure(api_key=api_key)

image_bytes = Path(image_path).read_bytes()
image_b64 = base64.b64encode(image_bytes).decode()

model = genai.GenerativeModel("gemini-2.0-flash-preview-image-generation")
response = model.generate_content([
    {"text": instruction},
    {"inline_data": {"mime_type": "image/jpeg", "data": image_b64}}
])

for part in response.candidates[0].content.parts:
    if hasattr(part, 'inline_data') and part.inline_data:
        Path(output_path).write_bytes(base64.b64decode(part.inline_data.data))
        break
```

---

## Mode 3: Hebrew Prompt Builder

When the user describes in Hebrew what they want, build a professional English prompt.

**Professional prompt format:**
[Subject] -- [Action/Scene] -- [Style] -- [Lighting] -- [Camera] -- [Details]

**Examples:**

| Hebrew | Professional Prompt |
|--------|---------------------|
| "dog on beach" | "A golden retriever sitting on a sandy beach at sunset, warm golden hour lighting, photorealistic, shallow depth of field" |
| "futuristic city" | "Futuristic cityscape at night, neon lights, cyberpunk aesthetic, aerial view, cinematic lighting, ultra-detailed" |
| "coffee logo" | "Minimalist coffee shop logo, vintage style, warm brown tones, clean white background, professional branding" |

**Ask the user:**
- Style? (realistic / painting / vector / photographic)
- Use? (social media / website / print)
- Aspect ratio?

---

## Mode 4: Auto-Save

Save every generated image with a descriptive filename:

**Format:** nanob2_YYYY-MM-DD_HH-MM_[short-description].png

**Default folder:** User's Downloads folder unless requested otherwise.

---

## Error Handling

| Error | Solution |
|-------|---------|
| API_KEY_INVALID | Ask for new key |
| SAFETY | Prompt blocked -- suggest rephrasing |
| QUOTA_EXCEEDED | User reached limit -- explain and wait |
| File not saved | Check folder permissions |

---

## Installation

```bash
pip install google-generativeai --break-system-packages
```
