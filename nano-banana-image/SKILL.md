---
name: nano-banana-image
description: |
  יצירה ועריכת תמונות וסרטונים ודפי נחיתה עם Nano Banana Pro + Veo 3 + Remotion דרך Google AI Studio API.
  השתמש בסקיל הזה כאשר המשתמש רוצה: ליצור תמונה לעסק, לצור סרטון שיווקי, טקסט מונפש, אנימציה שיווקית, רילס, לשפר prompt, לערוך תמונה קיימת.
  
  הפעל תמיד כאשר המשתמש אומר: "צור תמונה", "תייצר לי תמונה", "ננו בננה", "nano banana", "generate image", "create image", "ערוך תמונה", "שנה תמונה", "תמונה מטקסט", "צור סרטון", "סרטון שיווקי", "veo", "רימושיין", "remotion", "טקסט מונפש", "אנימציה", "רילס", "דף נחיתה", "landing page".
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



## שלב 9 — דף נחיתה מקצועי (Landing Page)

אחרי שיש סרטון (Veo 3) + לוגו + מחקר על העסק — הצע ליצור **דף נחיתה מלא**.

### מה לשאול לפני הבנייה

שאל את המשתמש רק על מה שחסר (לא לשאול על מה שכבר גילית מהמחקר):

```
✅ בדוק שיש לך — לפני ששואל:
- שם העסק ותחום ← מהמחקר
- כאבי לקוח + פתרונות ← מהמחקר
- שירותים ← מהמחקר
- תמונה/סרטון ← כבר יצרנו
- לוגו ← כבר הורדנו
- צבעי מותג ← מהמחקר

❓ שאל רק אם חסר:
- טלפון / ווצאפ (חובה לדף!)
- כתובת / עיר שירות
- שם המומחה/ית
- שאלות נפוצות (FAQ)
- המלצות לקוחות (testimonials)
- כמה זמן עסק קיים / כמה לקוחות
- קישור לאינסטגרם / פייסבוק
```

> **כלל:** שאל בשאלה אחת מרוכזת, לא 8 שאלות נפרדות.
> "חסר לי: טלפון/ווצאפ, כתובת, ואם יש המלצות לקוחות — כל השאר יש לי מהמחקר."

---

### מבנה דף הנחיתה

בנה HTML קובץ יחיד (`.html`) עם CSS ו-JS מובנים. כולל:

```
1. HERO         ← כותרת + תת-כותרת + כפתור CTA + סרטון ברקע או בצד
2. PROBLEM      ← "האם אתה סובל מ...?" — כאבי הלקוח
3. SOLUTION     ← הפתרון שלנו + יתרונות
4. SERVICES     ← כרטיסי שירותים עם אייקונים
5. ABOUT        ← מי המומחה, ניסיון, הסמכות
6. TESTIMONIALS ← המלצות לקוחות (3 לפחות)
7. CTA          ← "השאר פרטים" / כפתור ווצאפ
8. FOOTER       ← לוגו, קישורים, טלפון
```

---

### תמונות לדף הנחיתה — היררכיה + בדיקה ויזואלית חובה!

**היררכיית מקורות תמונה:**

1. **Nano Banana Pro** — אם הקוטה זמינה (AI מותאם לעסק)
2. **Pexels** — אם Nano Banana מחזיר שגיאה 429 (Quota exceeded)
3. **תמונות מאתר הלקוח** — רק אחרי בדיקה שמתאימות לנושא

**חיפוש ב-Pexels (כשיש שגיאת 429):**

גלוש לחיפוש Pexels עם מילות מפתח **ספציפיות לנושא הדף**, לדוגמא:
- פדיקור רפואי: `https://www.pexels.com/search/medical+pedicure+clinic/`
- ייעוץ עסקי: `https://www.pexels.com/search/business+consultant+office/`
- שיפוץ בתים: `https://www.pexels.com/search/home+renovation+professional/`

```javascript
// הורדה מ-Pexels (הרץ מתוך דף pexels.com):
const imgId = 'REPLACE_WITH_ID'; // מה-URL של התמונה
const imgUrl = `https://images.pexels.com/photos/${imgId}/pexels-photo-${imgId}.jpeg?auto=compress&cs=tinysrgb&w=800&h=1000&fit=crop`;
fetch(imgUrl)
  .then(r => r.blob())
  .then(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'img_[section].jpg';
    document.body.appendChild(a); a.click();
  });
```

**⚠️ חובה — בדיקה ויזואלית אחרי כל הורדה:**

```
שלב 1: הורד את התמונה לתיקיית Downloads
שלב 2: השתמש ב-Read tool לצפייה ויזואלית בתמונה עצמה:
        Read("C:\\Users\\koby-\\Downloads\\[name]_img.jpg")
שלב 3: Claude רואה את התמונה — בדוק שהיא מדברת לנושא:
        ✅ מתאים: כלים רפואיים, עבודה מקצועית, סצנה רלוונטית
        ❌ לא מתאים: נוף טבע, פריטים לא קשורים, אנשים ברקע שגוי
שלב 4: אם לא מתאים → חפש ב-Pexels עם מילות מפתח אחרות + חזור לשלב 1
```

> **כלל קריטי:** לעולם אל תכניס תמונה לדף הנחיתה בלי לראות אותה קודם עם `Read` tool ולאשר שהיא רלוונטית לנושא.

---

### עקרונות עיצוב (חובה!)

**עיצוב ברמת ui-ux-pro-max:**

1. **פונטים:** Heebo (עברית) מ-Google Fonts — `<link>` בראש הדף
2. **כיוון:** `direction: rtl; text-align: right` על ה-body
3. **צבעים:** השתמש בצבעי מותג שחולצו מהאתר. הגדר CSS variables:
```css
:root {
  --brand-primary: #[צבע ראשי];
  --brand-secondary: #[צבע משני];
  --brand-accent: #[הדגשה];
}
```
4. **אנימציות:** Scroll-triggered reveals עם Intersection Observer
5. **Hero:** gradient overlay על הסרטון/תמונה — לא רקע ריק
6. **כפתורי CTA:** גדולים, בצבע מותג, עם hover animation
7. **Mobile-first:** responsive בכל גודל מסך

---

### וידאו בדף

הסרטון של Veo 3 שיצרנו — הטמע כ-`<video>` element:

```html
<!-- אופציה A: ברקע ה-Hero -->
<div class="hero">
  <video autoplay muted loop playsinline class="hero-video">
    <source src="./[name]_pedicure.mp4" type="video/mp4">
  </video>
  <div class="hero-overlay"></div>
  <div class="hero-content">
    <h1>כותרת ראשית</h1>
    <p>תת-כותרת</p>
    <a href="https://wa.me/972XXXXXXX" class="cta-btn">דברו איתנו עכשיו</a>
  </div>
</div>
```

---

### כפתור ווצאפ צף (חובה!)

```html
<a href="https://wa.me/972XXXXXXXXX?text=היי, אשמח לקבל פרטים" 
   class="whatsapp-float" target="_blank">💬</a>

<style>
.whatsapp-float {
  position: fixed; bottom: 24px; left: 24px;
  z-index: 999; background: #25D366;
  width: 60px; height: 60px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 28px; box-shadow: 0 4px 20px rgba(37,211,102,0.4);
  animation: pulse 2s infinite; text-decoration: none;
}
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.08); }
}
</style>
```

---

### מצב עריכה מובנה (Edit Mode) — חובה בכל דף!

כל דף נחיתה חייב לכלול **מצב עריכה מלא** — מאפשר למשתמש לדייק טקסטים, תמונות וקישורים בלי לנגוע בקוד.

**הוסף לסוף ה-HTML (לפני `</body>`):**

```html
<!-- EDIT MODE -->
<button id="edit-toggle" onclick="EditMode.toggle()" style="position:fixed;top:16px;left:16px;z-index:9999;background:#6C3DE0;color:#fff;border:none;border-radius:8px;padding:10px 18px;font-size:14px;cursor:pointer;font-family:Heebo,sans-serif;box-shadow:0 4px 15px rgba(108,61,224,.4);">✏️ עריכה</button>
<input type="file" id="img-file-input" accept="image/*" style="display:none">
<div id="link-popup" style="display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);background:#fff;border-radius:12px;padding:24px;z-index:10000;box-shadow:0 20px 60px rgba(0,0,0,.3);min-width:320px;direction:rtl;">
  <h3 style="margin:0 0 16px;font-family:Heebo">עריכת קישור</h3>
  <label style="font-family:Heebo;font-size:14px">טקסט:<br><input id="link-text-input" type="text" style="width:100%;margin:6px 0 12px;padding:8px;border:1px solid #ddd;border-radius:6px;font-family:Heebo"></label>
  <label style="font-family:Heebo;font-size:14px">קישור:<br><input id="link-href-input" type="text" style="width:100%;margin:6px 0 12px;padding:8px;border:1px solid #ddd;border-radius:6px"></label>
  <div style="display:flex;gap:8px;justify-content:flex-end">
    <button onclick="EditMode.closeLinkPopup()" style="padding:8px 16px;background:#eee;border:none;border-radius:6px;cursor:pointer;font-family:Heebo">ביטול</button>
    <button onclick="EditMode.saveLink()" style="padding:8px 16px;background:#6C3DE0;color:#fff;border:none;border-radius:6px;cursor:pointer;font-family:Heebo">שמור</button>
  </div>
</div>
<script>
const EditMode = (() => {
  let active=false, _curImg=null, _curLink=null;
  const TEXT_SEL='h1,h2,h3,h4,p,span,.service-title,.service-desc,.stat-number,.stat-label,.testimonial-text,.testimonial-author,.hero-title,.hero-subtitle,.section-title,.section-subtitle,.about-text,.about-name,.about-credentials';
  const IMG_SEL='section img,.problem-image img,.about-visual img';
  const LINK_SEL='a.cta-btn,a.whatsapp-float,.footer a,button.cta';
  function toggle() {
    active=!active;
    document.body.classList.toggle('edit-mode',active);
    document.getElementById('edit-toggle').textContent=active?'💾 שמור ויצא':'✏️ עריכה';
    if(active) enable(); else { saveHTML(); disable(); }
  }
  function enable() {
    document.querySelectorAll(TEXT_SEL).forEach(el=>{el.dataset.ed='1';el.contentEditable='true';el.style.outline='2px dashed #6C3DE0';});
    document.querySelectorAll(IMG_SEL).forEach(img=>{img.style.cursor='pointer';img.style.outline='2px dashed #FF6B35';img.addEventListener('click',onImgClick);});
    document.querySelectorAll(LINK_SEL).forEach(el=>{el.dataset.linkEd='1';el.addEventListener('click',onLinkClick);});
  }
  function disable() {
    document.querySelectorAll('[data-ed]').forEach(el=>{el.contentEditable='false';el.style.outline='';delete el.dataset.ed;});
    document.querySelectorAll(IMG_SEL).forEach(img=>{img.style.cursor='';img.style.outline='';img.removeEventListener('click',onImgClick);});
    document.querySelectorAll('[data-link-ed]').forEach(el=>{el.removeEventListener('click',onLinkClick);delete el.dataset.linkEd;});
  }
  function onImgClick(e){if(!active)return;e.preventDefault();e.stopPropagation();_curImg=e.currentTarget;document.getElementById('img-file-input').click();}
  document.getElementById('img-file-input').addEventListener('change',function(){
    if(!_curImg||!this.files[0])return;
    const reader=new FileReader();
    reader.onload=ev=>{_curImg.src=ev.target.result;};
    reader.readAsDataURL(this.files[0]);this.value='';
  });
  function onLinkClick(e){if(!active)return;e.preventDefault();e.stopPropagation();_curLink=e.currentTarget;document.getElementById('link-text-input').value=_curLink.textContent.trim();document.getElementById('link-href-input').value=_curLink.href||'';document.getElementById('link-popup').style.display='block';}
  function saveLink(){if(_curLink){_curLink.textContent=document.getElementById('link-text-input').value;_curLink.href=document.getElementById('link-href-input').value;}closeLinkPopup();}
  function closeLinkPopup(){document.getElementById('link-popup').style.display='none';_curLink=null;}
  function saveHTML(){const html='<!DOCTYPE html>\n'+document.documentElement.outerHTML;const blob=new Blob([html],{type:'text/html; charset=utf-8'});const a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download=document.title.replace(/\s+/g,'_')+'_edited.html';a.click();URL.revokeObjectURL(a.href);}
  return{toggle,saveLink,closeLinkPopup};
})();
</script>
```

**מה מאפשר מצב העריכה:**
- לחיצה על כל טקסט → עריכה ישירה
- לחיצה על תמונה → בחירת קובץ חדש מהמחשב
- לחיצה על כפתור/קישור → שינוי טקסט + URL
- לחיצה שנייה על "עריכה" → **שמירה אוטומטית** של הדף הערוך כקובץ HTML

---

### שמירה ופתיחה

שמור בתיקיית Downloads לצד הסרטון:
`C:\Users\koby-\Downloads\[business-name]_landing.html`

תן למשתמש קישור: `[פתח דף נחיתה](computer://C:\Users\koby-\Downloads\[name]_landing.html)`

---

### שאל אחרי הבנייה

> "רוצה שאתאים את הצבעים / הטקסטים / המבנה? אפשר לשנות כל דבר."

## כללי ברזל

1. לוגו תמיד — כל תמונה יוצאת עם לוגו. אין יוצאים מן הכלל.
2. סרטון/Remotion רק באישור — אל תייצר בלי שהמשתמש אמר מפורשות "כן".
3. עברית בתמונות — תמיד לכלול הקשר עברי/ישראלי בפרומפט.
4. מחקר לפני פרומפט — חוקר את העסק לפני שבונה prompt.
5. API דרך Chrome בלבד — הרץ fetch() דרך mcp__Claude_in_Chrome__javascript_tool.
6. פולינג לסרטון — Veo הוא async. חכה, פלל, אל תניח שנכשל לפני 3 דקות.
7. Remotion = חינם — אין עלות. ניתן לרנדר כמה פעמים שרוצים.
8. **בדוק תמונות** — לאחר כל הורדת תמונה, השתמש ב-Read tool לצפייה בתמונה לפני הכנסתה לדף.
9. **Edit Mode** — כל דף נחיתה חייב לכלול את בלוק ה-Edit Mode המלא.
10. **Pexels fallback** — אם Nano Banana מחזיר 429, עבור מידית ל-Pexels עם מילות מפתח ספציפיות לנושא.

---

## ניהול מפתח API

מיקום: C:\Users\koby-\Downloads\nano-banana-config.txt
NANO_BANANA_API_KEY=AIzaSy...

קרא עם Read tool בתחילת כל שיחה.
