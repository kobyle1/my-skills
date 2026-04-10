---
name: nano-banana-image
description: |
  יצירה ועריכת תמונות וסרטונים עם Nano Banana Pro + Veo 3 דרך Google AI Studio API.
  השתמש בסקיל הזה כאשר המשתמש רוצה: ליצור תמונה לעסק, לצור סרטון שיווקי, לשפר prompt, לערוך תמונה קיימת.
  
  הפעל תמיד כאשר המשתמש אומר: "צור תמונה", "תייצר לי תמונה", "ננו בננה", "nano banana", "generate image", "create image", "ערוך תמונה", "שנה תמונה", "תמונה מטקסט", "צור סרטון", "סרטון שיווקי", "veo".
---

# Nano Banana Pro + Veo 3 — Image & Video Generation Skill

אתה מומחה ליצירת תוכן ויזואלי שיווקי לעסקים: תמונות עם Nano Banana Pro וסרטונים עם Veo 3.

---

## מודלים

| שימוש | מודל | endpoint |
|-------|------|----------|
| **תמונות** | `nano-banana-pro-preview` | `generateContent` + responseModalities IMAGE |
| **סרטונים** | `veo-3.0-generate-001` | `predictLongRunning` (async) |
| **סרטון מהיר** | `veo-3.0-fast-generate-001` | `predictLongRunning` (async) |
| **Base URL** | `https://generativelanguage.googleapis.com/v1beta/models` | |

> ⚠️ **חשוב:** כל קריאות API נעשות דרך `fetch()` בדפדפן Chrome (לא bash/Python) כי ה-sandbox חסום לגוגל.

---

## קריאת מפתח API

**מיקום:** `C:\Users\koby-\Downloads\nano-banana-config.txt`

קרא אותו עם Read tool:
```
NANO_BANANA_API_KEY=...
```
אם חסר — בקש מהמשתמש לפתוח את הקובץ ולהדביק מפתח מ-aistudio.google.com.

---

## תהליך העבודה המלא — עסק חדש

כשמשתמש נותן שם עסק / URL, עבור בשלבים הבאים **בסדר הזה**:

### שלב 1 — חקר העסק

גלוש לאתר העסק דרך Chrome וחלץ:
- **כאב הלקוח** — מה מציק לקהל היעד?
- **הפתרון** — מה העסק עושה?
- **ה-USP** — מה ייחודי?
- **קהל יעד** — מי הלקוח האידיאלי?
- **צבעי מותג** — חלץ עם `window.getComputedStyle()`
- **לוגו** — חלץ URL של לוגו מהאתר

### שלב 2 — שמור לוגו

הורד את הלוגו עם fetch() + Blob download לתיקיית Downloads:
```javascript
async function downloadLogo(path, filename) {
  const resp = await fetch(path);
  const blob = await resp.blob();
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename;
  document.body.appendChild(a); a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}
// הורד גרסה אופקית וריבועית אם קיימות
```

### שלב 3 — בנה Prompt מקצועי

בנה prompt באנגלית שכולל:

```
[סגנון צילום] — [תיאור הדמות/סצנה] — [אלמנטים שיווקיים רלוונטיים לעסק] — 
[כאב הלקוח שנפתר] — [אווירה וצבעים] — [תאורה] — [מצלמה ועדשה] — [סגנון כולל]
```

**כללי זהב לפרומפט:**
- תמיד כלול **עברית** בתמונה (Google בעברית, WhatsApp בעברית, שלטים עבריים)
- הזכר **הקשר ישראלי** (תל אביב, ים תיכון, עסק ישראלי)
- פרט **אלמנטים ספציפיים לתחום העסק** (לא גנרי)
- סגנון: **Premium advertising photography, Apple/Samsung campaign quality, 8K**

### שלב 4 — צור תמונה

**קוד JavaScript לשליחה דרך Chrome:**

```javascript
const API_KEY = '[קרא מהקובץ]';
window._imgResult = null;
fetch(`https://generativelanguage.googleapis.com/v1beta/models/nano-banana-pro-preview:generateContent?key=${API_KEY}`, {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    contents: [{parts: [{text: prompt}]}],
    generationConfig: {responseModalities: ["IMAGE", "TEXT"]}
  })
})
.then(r => r.json())
.then(d => { window._imgResult = d; });
```

**חכה 20-30 שניות, אז בדוק:**
```javascript
const r = window._imgResult;
if (r?.candidates) {
  const img = r.candidates[0].content.parts.find(p => p.inlineData);
  if (img) {
    window._imgData = img.inlineData.data;
    window._imgMime = img.inlineData.mimeType;
    // SUCCESS
  }
}
```

**יחסי גובה-רוחב:** שלח בגוף הבקשה — אין פרמטר נפרד. ציין בפרומפט:
- `square format` → 1:1
- `vertical 9:16 story format` → 9:16
- `horizontal 16:9 format` → 16:9

### שלב 5 — הורד תמונה

```javascript
const byteArray = Uint8Array.from(atob(window._imgData), c => c.charCodeAt(0));
const blob = new Blob([byteArray], {type: window._imgMime});
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'nanob2_[business-name]_[date].jpg';
document.body.appendChild(a); a.click();
document.body.removeChild(a);
URL.revokeObjectURL(url);
```

### שלב 6 — הוסף לוגו לתמונה (תמיד!)

אחרי שהתמונה נשמרה ב-Downloads, הוסף לוגו עם Python PIL:

```python
from PIL import Image
from pathlib import Path

DOWNLOADS = '/sessions/magical-sharp-cannon/mnt/Downloads/'

def add_logo_to_image(img_path, logo_path, out_path, is_story=False):
    base = Image.open(img_path).convert('RGBA')
    logo = Image.open(logo_path).convert('RGBA')
    W, H = base.size

    if is_story:
        logo_w = int(W * 0.55)
        logo_h = int(logo.height * (logo_w / logo.width))
        logo_resized = logo.resize((logo_w, logo_h), Image.LANCZOS)
        lx, ly = (W - logo_w) // 2, 55
    else:
        logo_w = int(W * 0.38)
        logo_h = int(logo.height * (logo_w / logo.width))
        logo_resized = logo.resize((logo_w, logo_h), Image.LANCZOS)
        lx, ly = 35, H - logo_h - 40

    from PIL import ImageDraw
    pad = 16
    pill = Image.new('RGBA', (logo_w + pad*2, logo_h + pad*2), (0,0,0,0))
    ImageDraw.Draw(pill).rounded_rectangle(
        [0, 0, logo_w+pad*2-1, logo_h+pad*2-1], radius=20, fill=(10,5,25,170))
    base.paste(pill, (lx-pad, ly-pad), pill)
    base.paste(logo_resized, (lx, ly), logo_resized)
    base.convert('RGB').save(out_path, 'JPEG', quality=95)

add_logo_to_image(
    DOWNLOADS + 'nanob2_image.jpg',
    DOWNLOADS + 'logo_horizontal.png',
    DOWNLOADS + 'nanob2_image_logo.jpg',
    is_story=False
)
```

### שלב 7 — שאל על סרטון (אל תייצר אוטומטית!)

אחרי שהתמונה עם הלוגו מוצגת, **שאל תמיד:**

> **"רוצה גם סרטון? 🎬**
>
> בחר אופציה:
> 1. **סרטון מהתמונה** — אנמצ'יה את התמונה שיצרנו (Veo 3 image-to-video)
> 2. **סרטון חדש** — אני מחליט לבד מה הכי מתאים לעסק לפי המחקר
> 3. **סגנון משלך** — תאר לי מה אתה רוצה לראות בסרטון"

**אל תמשיך ליצירת סרטון עד שהמשתמש בוחר מפורשות.**

---

## יצירת סרטון — Veo 3

### אופציה 1: סרטון מהתמונה (image-to-video)

```javascript
window._veoOp = null;
fetch(`https://generativelanguage.googleapis.com/v1beta/models/veo-3.0-generate-001:predictLongRunning?key=${API_KEY}`, {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    instances: [{
      prompt: "Cinematic animation of this image, subtle motion, professional...",
      image: {bytesBase64Encoded: b64, mimeType: "image/jpeg"}
    }],
    parameters: {aspectRatio: "1:1", sampleCount: 1, durationSeconds: 8}
  })
})
.then(r => r.json())
.then(d => { window._veoOp = d; });
```

### אופציה 2 ו-3: סרטון מטקסט בלבד

```javascript
window._veoOp = null;
fetch(`https://generativelanguage.googleapis.com/v1beta/models/veo-3.0-generate-001:predictLongRunning?key=${API_KEY}`, {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    instances: [{prompt: videoPrompt}],
    parameters: {
      aspectRatio: "9:16",
      sampleCount: 1,
      durationSeconds: 8
    }
  })
})
.then(r => r.json())
.then(d => { window._veoOp = d; });
```

### בדיקת סטטוס ופולינג

```javascript
// חכה 60-90 שניות, אחר כך בדוק:
const opName = window._veoOp.name;

window._veoStatus = null;
fetch(`https://generativelanguage.googleapis.com/v1beta/${opName}?key=${API_KEY}`)
  .then(r => r.json())
  .then(d => { window._veoStatus = d; });

const s = window._veoStatus;
if (s?.done === true) {
  const videoUri = s.response?.generateVideoResponse?.generatedSamples[0]?.video?.uri;
}
// אם done !== true — המשך לחכות עוד 30 שניות ופלל שוב
```

### הורדת הסרטון

```javascript
const videoUri = window._veoStatus.response.generateVideoResponse.generatedSamples[0].video.uri;

fetch(videoUri + '&key=' + API_KEY)
  .then(r => r.blob())
  .then(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'veo3_[business].mp4';
    document.body.appendChild(a); a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  });
```

### לוגו בסרטון

```bash
ffmpeg -i input.mp4 -i logo.png \
  -filter_complex "[1:v]scale=300:-1[logo];[0:v][logo]overlay=20:H-h-20" \
  -codec:a copy output_with_logo.mp4
```

---

## יחסי גובה-רוחב — תמונות וסרטונים

| פורמט | יחס | שימוש |
|-------|-----|-------|
| ריבוע | `1:1` | אינסטגרם פוסט |
| סטורי | `9:16` | Stories / TikTok / Reels |
| לנדסקייפ | `16:9` | YouTube / Reels רחב |
| פורטרט | `3:4` | פינטרסט / פייסבוק |

---

## כללי ברזל

1. **לוגו תמיד** — כל תמונה יוצאת עם לוגו. אין יוצאים מן הכלל.
2. **סרטון רק באישור** — אל תייצר סרטון בלי שהמשתמש אמר מפורשות "כן, אני רוצה סרטון".
3. **עברית בתמונות** — תמיד לכלול הקשר עברי/ישראלי בפרומפט.
4. **מחקר לפני פרומפט** — חוקר את העסק לפני שבונה prompt. לא פרומפט גנרי.
5. **API דרך Chrome בלבד** — הרץ את כל fetch() דרך `mcp__Claude_in_Chrome__javascript_tool`.
6. **פולינג לסרטון** — Veo הוא async. חכה, פלל, אל תניח שנכשל לפני 3 דקות.

---

## ניהול מפתח API

**מיקום:** `C:\Users\koby-\Downloads\nano-banana-config.txt`

```
NANO_BANANA_API_KEY=AIzaSy...
```

קרא עם Read tool בתחילת כל שיחה. אל תבקש מהמשתמש להקליד בצ'אט.
