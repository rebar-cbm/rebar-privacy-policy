# מדריך פריסה מפורט — Phase 1

הפריסה ב-7 שלבים. כל שלב עומד בפני עצמו ואפשר לעצור בין אחד לשני. זמן כולל משוער: 45–60 דקות אם הכל זורם.

## שלב 0 — גיבוי (לפני שנוגעים בכלום)

**למה:** אם משהו מתפוצץ, אתה יכול לחזור אחורה ב-3 דקות במקום לשחזר ידנית.

**פעולות:**

1. **גיבוי Firestore:**
   - Firebase Console → Firestore Database → הטאב Import/Export.
   - לחץ Export → בחר collection `users` (ואם יש גם `kashim_users`) → בחר Cloud Storage bucket → Export.
   - זמן: 1–2 דקות. חכה שתראה "Export completed".
2. **גיבוי הקבצים הנוכחיים מ-GitHub:**
   - הורד את הגרסה הנוכחית של `login.html` ו-`admin.html` למחשב (ב-GitHub לחץ על הקובץ → Download Raw). שמור בתיקייה `cbm-backup-2026-04-20/`.
3. **העתק את Firestore Rules הנוכחיים:**
   - Firebase Console → Firestore Database → הטאב Rules.
   - העתק את כל הטקסט הנוכחי לקובץ טקסט מקומי בשם `firestore.rules.backup.txt`.

**בדיקה:** יש לך 3 גיבויים פיזיים על המחשב. אם לא — חזור.

---

## שלב 1 — העלאת Firestore Rules החדשים

**למה ראשון:** הכללים החדשים סבלניים לקוד ישן (מותרים עדכונים של admin ושל בעל החשבון על ה-projects). אחרי שהם בתוקף, אפשר להתחיל להעלות קוד חדש בלי שייצור דאטה במבנה לא מאובטח.

**פעולות:**

1. פתח את `firestore.rules` שהורדת במחשב.
2. Firebase Console → Firestore Database → Rules → מחק את הכל → הדבק את התוכן של הקובץ החדש → Publish.
3. המתן להודעת "Rules published successfully".

**בדיקת שפיות (חובה):**

- פתח את האתר בחלון גלישה פרטית.
- התחבר עם משתמש קיים (לא חדש — אל תירשם עדיין).
- ודא שנכנסת ל-Rebar/Kashim כרגיל, שרואה פרויקטים קיימים, ושאפשר לשמור פרויקט חדש.

**אם נשבר:** Firebase Console → Rules → היסטוריה (אייקון הזמן מימין) → בחר את הגרסה הקודמת → Rollback. זה מיידי.

---

## שלב 2 — Migration של משתמשים קיימים

**למה:** משתמשים שנרשמו לפני היום לא חיים במבנה v1. הכלי מוסיף להם את השדות החסרים (profile/access/plan/status) עם ברירות מחדל הגיוניות.

**פעולות:**

1. העלה את `admin-migrate.html` לשורש האתר (דרך GitHub או FTP — איך שנוח לך).
2. רוקן את Cloudflare cache לנתיב הזה (Cloudflare Dashboard → Caching → Configuration → Purge Cache → Custom Purge → הכנס `https://www.rebar-cbm.com/admin-migrate.html`).
3. פתח `https://www.rebar-cbm.com/admin-migrate.html` בחלון פרטי.
4. היכנס עם מייל אדמין מורשה (`wael_andrea@hotmail.com` או `wael@tmteam.co.il`).
5. לחץ **🔍 סרוק (Dry Run)** → הכלי יראה לך כמה משתמשים תקינים וכמה legacy. עדיין לא משנה כלום.
6. עיין בלוג — ודא שהשמות והמיילים נראים הגיוניים.
7. אם הכל תקין — לחץ **🚀 בצע Migration**.
8. חכה שיסיים (שנייה אחת לכל משתמש — 50 משתמשים = דקה).

**בדיקת שפיות:**

- Firebase Console → Firestore → collection `users` → פתח מסמך של משתמש ישן → ודא שיש לו עכשיו `profile`, `access`, `plan`, `status`.
- בפרט ודא ש-`status.blocked = false` בכולם.

**חשוב מאוד:** אחרי שה-migration עובר בהצלחה, **מחק את `admin-migrate.html` מהשרת**. אל תשאיר כלי migration פומבי גם אם הוא מוגן בסיסמה.

---

## שלב 3 — העלאת login.html החדש

**למה:** מעכשיו, כל רישום חדש ייצור אוטומטית מסמך v1 מלא. כל התחברות תעדכן `lastLogin` ותבדוק חסימה.

**פעולות:**

1. העלה את `login.html` החדש דרך GitHub (Commit ישירות על הקובץ הקיים `login.html`).
2. רוקן את Cloudflare cache לנתיב `/login.html` (וגם לוריאנטים `/login-en.html`, `/login-ar.html` אם אתה מעדכן גם אותם — כרגע לא צריך, רק `login.html`).
3. חכה 30 שניות לפריסה.

**בדיקת שפיות — עם משתמש טסט חדש:**

- פתח חלון פרטי → `https://www.rebar-cbm.com/login.html`.
- הירשם עם מייל חדש שיש לך גישה אליו (`wael+test@...` או מייל Gmail משני).
- קבל את מייל האימות, לחץ על הקישור.
- חזור ל-login, התחבר.
- Firebase Console → Firestore → `users/` → חפש את ה-UID החדש → ודא שהמסמך קיים עם כל 4 הקטעים (profile/access/plan/status) וש-`status.lastLogin` התעדכן לעכשיו.

**בדיקת חסימה (חשוב):**

- ב-Firebase Console, עבור למסמך של משתמש הטסט → ערוך את `status.blocked` ל-`true` → שמור.
- בחלון הפרטי, התנתק → נסה להתחבר שוב → אמור לראות הודעה "⛔ חשבונך חסום".
- אם זה עבד — החזר `blocked: false`.

**אם נשבר:** חזור לגיבוי ב-`cbm-backup-2026-04-20/login.html` והעלה אותו במקום.

---

## שלב 4 — העלאת admin.html החדש

**פעולות:**

1. העלה את `admin.html` החדש ל-GitHub במקום הקיים.
2. רוקן Cloudflare cache ל-`/admin.html`.

**בדיקת שפיות — סיור בכל הטאבים:**

- פתח `https://www.rebar-cbm.com/admin.html` בחלון פרטי → היכנס עם מייל אדמין.
- **טאב Overview:** ודא שהמספרים נראים הגיוניים (סה"כ משתמשים, Pro, Free, חסומים).
- **טאב Users:** פתח מישהו → ודא שהפרטים נטענים, המתגים של הגישה עובדים.
- **בדיקה פעילה:** לחץ על מתג 📋 מכרזים של משתמש מסוים → אמור להשתנות מיידית ל-ON → רענן את הדף → המצב נשמר.
- **בדיקת חסימה:** לחץ על "⛔ חסום משתמש" של משתמש טסט → ודא שהופיע badge "חסום" וש-Firebase Console מראה `status.blocked: true`.
- **בדיקת audit log:** Firebase Console → collection `admin_audit` → אמור להיות מסמך חדש עם הפעולות שעשית.

**אם נשבר:** חזור לגיבוי ב-`cbm-backup-2026-04-20/admin.html`.

---

## שלב 5 — העלאת auth-guard.js לנתיב המשותף

**פעולות:**

1. ב-GitHub, צור תיקייה חדשה `shared/` בשורש (דרך יצירת קובץ בנתיב `shared/auth-guard.js`).
2. הדבק את התוכן של `auth-guard.js`.
3. Commit.
4. ודא שהוא נגיש: פתח `https://www.rebar-cbm.com/shared/auth-guard.js` בדפדפן → אמור לראות את הקוד.

זהו. הקובץ עצמו לא עושה כלום עד שכלי יקרא לו. עוברים להפעלה בכלי עצמו.

---

## שלב 6 — חיבור auth-guard לכלי אחד (Rebar בלבד לבדיקה)

**למה להתחיל מ-Rebar:** זה הכלי הכי נפוץ, נוח לבדיקה, ואם משהו נשבר אתה מזהה מיד.

**פעולות:**

1. פתח את `rebar/app.html` או `rebar/index.html` — ספציפית את הקובץ שמכיל היום את קוד ה-auth.
2. מצא את בלוק ה-Firebase SDK בראש (שורות עם `firebase-app-compat.js`, `firebase-auth-compat.js`). ודא שיש גם:
   ```html
   <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>
   ```
   אם אין — הוסף אותה.
3. מיד אחרי הטעינה של Firebase, הוסף:
   ```html
   <script src="/shared/auth-guard.js"></script>
   ```
4. מצא את הקוד הנוכחי שמבצע `auth.onAuthStateChanged(...)` ומאמת את המשתמש (כנראה ליד ראש הסקריפט). עטוף אותו או החלף אותו ב:
   ```html
   <script>
     CBMAuthGuard.protect({
       tool: 'rebar',
       loginUrl: '/login.html',
       homeUrl: '/',
       onAuthorized: (user, userDoc) => {
         window.currentUser = user;
         window.currentUserDoc = userDoc;
         // כאן הקוד שהיה רץ עד היום אחרי שהמשתמש אומת:
         initApp();  // או איך שקראת לפונקציית האתחול
       }
     });
   </script>
   ```
5. ודא ש-`initApp()` (או איך שנקראת פונקציית האתחול) אכן קיימת ומכילה את כל מה שצריך לרוץ אחרי האימות.

> ⚠ אם אתה לא בטוח בדיוק איפה להכניס את זה בקובץ הקיים של Rebar — שלח לי את הקוד הנוכחי של קטע ה-auth שלו, ואני אומר לך בדיוק מה להחליף ובמה.

**בדיקת שפיות:**

- חלון פרטי → `/rebar/app.html` בלי להיות מחובר → אמור לעשות redirect ל-login.
- התחבר עם משתמש רגיל → אמור להיכנס ל-Rebar.
- ב-admin, סגור לו את `access.rebar` → נסה להיכנס שוב → אמור לקבל הודעת "אין לך הרשאת גישה" ו-redirect הביתה.
- החזר את `access.rebar` → אמור לחזור לעבוד.

**אם עובד** — עבור לשלב 7. **אם לא** — חזור לגיבוי, תכתוב לי מה קרה.

---

## שלב 7 — חיבור auth-guard ל-Kashim ו-Tender

חזור על שלב 6 במדויק, פעם אחת לכל כלי:

- **Kashim:** `tool: 'kashim'` — נתיב `kashim/app.html`.
- **Tender:** `tool: 'tender'` — נתיב `tender/app.html` (או איפה שהוא).

אותה בדיקה לכל אחד — כניסה ללא הרשאה, חסימה ב-admin, שחזור.

---

## סיכום מה הושג אחרי 7 השלבים

- סכמת `users/` מאוחדת בכל הפלטפורמה.
- Admin אחד שולט ב-**כל** הכלים (לא רק Kashim).
- חסימת משתמש מרחוק עובדת ומיידית.
- מתג גישה פר-כלי פר-משתמש.
- Plan + תוקף נתמכים (בסיס לשכבת billing עתידית).
- Audit log לכל פעולת admin.
- אכיפה ברמת Firestore Rules — לא רק UI.

---

## נקודות קריטיות לזכור

- **אל תדלג על שלב 0 (גיבוי).** זה ההבדל בין "נפל, 3 דקות חזרנו" לבין "נפל, 3 שעות שחזור ידני".
- **Cloudflare cache** — בכל פעם שאתה מעלה קובץ HTML חדש, רוקן את הנתיב הספציפי. אחרת תחשוב שזה לא נפרס והכל עובד.
- **משתמש טסט נפרד** — אל תבדוק על המשתמש האישי שלך. צור `wael+test@...` וחבל עליו כמה שרוצה.
- **אל תשאיר `admin-migrate.html` על השרת** אחרי שסיימת.
