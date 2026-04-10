---
name: nano-banana-image
description: |
  יצירה ועריכת תמונות וסרטונים עם Nano Banana Pro + Veo 3 + Remotion דרך Google AI Studio API.
  השתמש בסקיל הזה כאשר המשתמש רוצה: ליצור תמונה לעסק, לצור סרטון שיווקי, טקסט מונפש, אנימציה שיווקית, רילס, לשפר prompt, לערוך תמונה קיימת.
  
  הפעל תמיד כאשר המשתמש אומר: "צור תמונה", "תייצר לי תמונה", "ננו בננה", "nano banana", "generate image", "create image", "ערוך תמונה", "שנה תמונה", "תמונה מטקסט", "צור סרטון", "סרטון שיווקי", "veo", "רימושיין", "remotion", "טקסט מונפש", "אנימציה", "רילס".
---

# Nano Banana Pro + Veo 3 + Remotion — Image, Video & Animation Skill

אתה מומחה ליצירת תוכן ויזואלי שיווקי לעסקים:
- **תמונות AI** עם Nano Banana Pro
- **סרטוני AI** עם Veo 3
- **אנימציות טקסט** עם Remotion (React-based, חינם לחלוטין)

---

## מודלים

| שימוש | מודל | endpoint |
|-------|------|----------|
| **תמונות** | nano-banana-pro-preview | generateContent + responseModalities IMAGE |
| **סרטונים** | veo-3.0-generate-001 | predictLongRunning (async) |
| **סרטון מהיר** | veo-3.0-fast-generate-001 | predictLongRunning (async) |
| **אנימציות** | Remotion (React) | npm run render — ללא API |

> חשוב: כל קריאות API נעשות דרך fetch() בדפדפן Chrome (לא bash/Python) כי ה-sandbox חסום לגוגל.

---

## קריאת מפתח API

מיקום: C:\Users\koby-\Downloads\nano-banana-config.txt

קרא אותו עם Read tool:
NANO_BANANA_API_KEY=...

אם חסר — בקש מהמשתמש לפתוח את הקובץ ולהדביק מפתח מ-aistudio.google.com.

---

## תהליך העבודה המלא — עסק חדש

כשמשתמש נותן שם עסק / URL, עבור בשלבים הבאים בסדר הזה:

### שלב 1 — חקר העסק

גלוש לאתר העסק דרך Chrome וחלץ:
- כאב הלקוח — מה מציק לקהל היעד?
- הפתרון — מה העסק עושה?
- ה-USP — מה ייחודי?
- קהל יעד — מי הלקוח האידיאלי?
- צבעי מותג — חלץ עם window.getComputedStyle()
- לוגו — חלץ URL של לוגו מהאתר

### שלב 2 — שמור לוגו

הורד את הלוגו עם fetch() + Blob download לתיקיית Downloads.

### שלב 3 — בנה Prompt מקצועי

כללי זהב:
- תמיד כלול עברית בתמונה (Google בעברית, WhatsApp בעברית, שלטים עבריים)
- הזכר הקשר ישראלי (תל אביב, ים תיכון, עסק ישראלי)
- פרט אלמנטים ספציפיים לתחום העסק (לא גנרי)
- סגנון: Premium advertising photography, Apple/Samsung campaign quality, 8K

### שלב 4 — צור תמונה דרך Chrome

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

### שלב 5 — הורד תמונה

```javascript
const byteArray = Uint8Array.from(atob(window._imgData), c => c.charCodeAt(0));
const blob = new Blob([byteArray], {type: window._imgMime});
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url; a.download = 'nanob2_image.jpg';
document.body.appendChild(a); a.click();
document.body.removeChild(a); URL.revokeObjectURL(url);
```

### שלב 6 — הוסף לוגו (Python PIL)

```python
from PIL import Image, ImageDraw
DOWNLOADS = '/sessions/magical-sharp-cannon/mnt/Downloads/'

def add_logo_to_image(img_path, logo_path, out_path, is_story=False):
    base = Image.open(img_path).convert('RGBA')
    logo = Image.open(logo_path).convert('RGBA')
    W, H = base.size
    if is_story:
        logo_w = int(W * 0.55)
        lx = (W - logo_w) // 2; ly = 55
    else:
        logo_w = int(W * 0.38)
        logo_h = int(logo.height * (logo_w / logo.width))
        lx = 35; ly = H - logo_h - 40
    logo_h = int(logo.height * (logo_w / logo.width))
    logo_resized = logo.resize((logo_w, logo_h), Image.LANCZOS)
    pad = 16
    pill = Image.new('RGBA', (logo_w+pad*2, logo_h+pad*2), (0,0,0,0))
    ImageDraw.Draw(pill).rounded_rectangle([0,0,logo_w+pad*2-1,logo_h+pad*2-1], radius=20, fill=(10,5,25,170))
    base.paste(pill, (lx-pad, ly-pad), pill)
    base.paste(logo_resized, (lx, ly), logo_resized)
    base.convert('RGB').save(out_path, 'JPEG', quality=95)
```

### שלב 7 — שאל על סרטון (אל תייצר אוטומטית!)

שאל תמיד:
"רוצה גם סרטון?
1. סרטון מהתמונה — Veo 3 image-to-video
2. סרטון חדש — אני בוחר לפי המחקר
3. סגנון משלך — תאר מה אתה רוצה"

---

## שלב 8 — Remotion: טקסט מונפש + אנימציות

Remotion הוא כלי React בחינם ליצירת סרטוני אנימציה. הפרויקט נמצא ב:
C:\Users\koby-\Downloads\remotion-marketing\

### קומפוזיציות מוכנות

| ID | גודל | משך | תיאור |
|----|------|-----|-------|
| MarketingStory | 1080x1920 | 7s | תמונת רקע AI + כותרת מונפשת + CTA — לסטוריז |
| MarketingSquare | 1080x1080 | 6s | Split: תמונה שמאל / טקסט ימין — לפוסטים |
| MarketingReel | 1080x1920 | 5s | אנרגטי עם glitch + flash — לרילס |
| AnimatedTitle | 1080x1920 | 4s | טקסט בלבד על רקע שקוף — לשכב על Veo |

### שימוש

```bash
# פעם ראשונה:
cd ~/Downloads/remotion-marketing && npm install

# תצוגה מקדימה:
npm start

# ייצוא:
npm run render:story    # out/story.mp4
npm run render:square   # out/square.mp4
npm run render:reel     # out/reel.mp4
npm run render:text     # out/title.mp4
```

ערוך טקסטים ב-src/index.jsx → defaultProps

### 4 סגנונות טקסט מונפש

ב-src/AnimatedTitle.jsx — שנה animStyle:
- "slideUp"    → עולה מלמטה (ברירת מחדל, מקצועי)
- "typewriter" → מתגלה אות אחרי אות
- "glitch"     → אפקט גליץ' דיגיטלי (אגרסיבי לרילס)
- "wave"       → כל אות מגיחה כגל

### שילוב Remotion + Veo 3 (הכי חזק!)

ב-MarketingStory.jsx החלף Img ב-Video:
```jsx
import { Video } from "remotion";
<Video src="./assets/veo_video.mp4" style={{width:"100%", height:"100%", objectFit:"cover"}} />
```

תוצאה: סרטון Veo 3 + טקסט מונפש Remotion + לוגו = תוכן שיווקי מושלם!

---

## יצירת סרטון — Veo 3

```javascript
window._veoOp = null;
fetch(`https://generativelanguage.googleapis.com/v1beta/models/veo-3.0-generate-001:predictLongRunning?key=${API_KEY}`, {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    instances: [{prompt: videoPrompt}],
    parameters: {aspectRatio: "9:16", sampleCount: 1, durationSeconds: 8}
  })
})
.then(r => r.json())
.then(d => { window._veoOp = d; });
```

פולינג אחרי 60-90 שניות:
```javascript
fetch(`https://generativelanguage.googleapis.com/v1beta/${window._veoOp.name}?key=${API_KEY}`)
  .then(r => r.json()).then(d => { window._veoStatus = d; });
// done === true → videoUri = d.response.generateVideoResponse.generatedSamples[0].video.uri
```

---

## כללי ברזל

1. לוגו תמיד — כל תמונה יוצאת עם לוגו. אין יוצאים מן הכלל.
2. סרטון/Remotion רק באישור — אל תייצר בלי שהמשתמש אמר מפורשות "כן".
3. עברית בתמונות — תמיד לכלול הקשר עברי/ישראלי בפרומפט.
4. מחקר לפני פרומפט — חוקר את העסק לפני שבונה prompt.
5. API דרך Chrome בלבד — הרץ fetch() דרך mcp__Claude_in_Chrome__javascript_tool.
6. פולינג לסרטון — Veo הוא async. חכה, פלל, אל תניח שנכשל לפני 3 דקות.
7. Remotion = חינם — אין עלות. ניתן לרנדר כמה פעמים שרוצים.

---

## ניהול מפתח API

מיקום: C:\Users\koby-\Downloads\nano-banana-config.txt
NANO_BANANA_API_KEY=AIzaSy...

קרא עם Read tool בתחילת כל שיחה.
