# שיעור 4 — תשובות

**1. ל-backend אין `ports`. איך stats מגיע אליו? ואיך הדפדפן מגיע אליו?**
stats נמצא באותה רשת של Compose, ופונה ל-backend לפי שם ה-service: `http://backend:5000`. Compose הופך את השם לכתובת. הדפדפן לא מגיע ל-backend ישירות: הוא פונה ל-Nginx ב-`localhost:8080`, ו-Nginx מעביר כל בקשה שמתחילה ב-`/api/` ל-`backend:5000` בתוך הרשת (reverse proxy).

**2. `down` ואז `up`: המוצרים נשארו? ומה עם `down -v`?**
אחרי `down` ו-`up` המוצרים נשארים, כי הם שמורים ב-named volume `postgres-data`, ו-`down` מוחק רק containers ורשת. `down -v` מוחק גם את ה-named volumes, ולכן המוצרים נמחקים, ובהפעלה הבאה המסד נוצר מחדש ריק.

**3. `../sql/init.sql:/docker-entrypoint-initdb.d/init.sql`: איזה סוג volume? מה אם משנים את `init.sql` אחרי שהמסד נוצר?**
זה bind mount. יודעים לפי הצד השמאלי: הוא נתיב במחשב (מתחיל ב-`.` או ב-`/`), ולא שם של volume. postgres מריץ קבצים מ-`docker-entrypoint-initdb.d` רק בהפעלה הראשונה, כשתיקיית הנתונים ריקה. לכן שינוי ב-`init.sql` אחרי שהמסד כבר נוצר לא ישפיע, עד שמוחקים את ה-volume (`down -v`) ומעלים מחדש.

**4. שם הרשת ושם ה-volume:**
- רשת: `docker_default`
- volume: `docker_postgres-data`

השמות מורכבים משם הפרויקט ועוד שם. שם הפרויקט הוא כברירת מחדל שם התיקייה שבה נמצא קובץ ה-compose, כאן `docker`. לרשת Compose מוסיף `_default`, ול-volume את השם שנתנו לו בקובץ (`postgres-data`).

**5. `Dockerfile.multistage`: מה יש ב-image הסופי ומה אין? למה אין `CMD`? למה ל-stats לא צריך multi-stage?**
ב-image הסופי יש רק Nginx, את הקבצים המוכנים מ-`dist` ואת `nginx.conf`. אין בו Node, אין את `build.js` ואין את קובצי המקור. רק השלב האחרון נכנס ל-image, ו"המטבח" נזרק. אין `CMD` כי השלב השני מתחיל מ-`nginx:1.30-alpine` ויורש ממנו את ה-`CMD` שמפעיל את Nginx. ל-stats לא צריך multi-stage כי אין לו שלב בנייה: את אותו `server.js` שכתבנו Node מריץ גם בזמן ריצה. Node נחוץ גם בהרצה, ואין כלים שאפשר להשאיר מאחור.

**6. הגדלים מ-ד8:**
| image | גודל |
|---|---|
| `single` | 101MB (תוכן 28.5MB) |
| `multi` | 101MB (תוכן 28.5MB) |
| `kitchen` | 244MB (תוכן 62.5MB) |

`multi` כמעט בגודל של `single` כי שניהם מבוססים על אותו `nginx:1.30-alpine`, וההבדל ביניהם הוא רק כמה קבצים קטנים של האתר. `kitchen` גדול בהרבה כי הוא מבוסס על `node:24-alpine`, שכולל את כל סביבת Node.

**7. `nginx.conf`: למה `/api/stats` מגיע ל-stats? ומה קורה ל-`/api/products` בגלל הלוכסן?**
Nginx בוחר את ה-`location` עם הקידומת הארוכה ביותר שמתאימה. `/api/stats` ארוך יותר מ-`/api/`, ולכן הוא מנצח והבקשה עוברת ל-stats. הלוכסן בסוף `http://backend:5000/` גורם ל-Nginx להחליף את החלק שהתאים (`/api/`) ב-`/`, ולכן `/api/products` מגיע ל-backend בתור `/products`, הכתובת שה-backend מכיר.

---

**בונוס (ד11):** הסיסמה נמצאת בקובץ `docker/.env`, שמופיע ב-`.gitignore` ולכן לא עולה ל-GitHub. בקובץ ה-compose כתבתי `${POSTGRES_PASSWORD:-shoppass}`, כלומר עם ערך ברירת מחדל. ככה גם ב-clone נקי (למשל במחשב של הכיתה), שאין בו `.env`, postgres עדיין עולה.
