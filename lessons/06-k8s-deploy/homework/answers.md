# שיעור 6 — תשובות

## 1. למה בנינו את ה-images עם `-t shoplist-backend:local` ולא השתמשנו בשמות של Compose (`docker-backend`)? מה היה קורה בלי `imagePullPolicy: Never`?

תשובה:
Compose ממציא את השמות בעצמו, לפי שם התיקייה (`docker`) ושם הסרוויס, ולכן הם לא קבועים. אם התיקייה תשנה שם, ישתנה גם שם ה-image. עם `-t` אנחנו קובעות שם ותג קבועים וברורים, והם השמות שכתובים בקבצי ה-Deployment.
בלי `imagePullPolicy: Never`, אם ה-image לא נמצא במחשב, Kubernetes ינסה להוריד אותו מ-Docker Hub. image כזה לא קיים שם, וה-pod ייתקע בשגיאה `ErrImagePull` / `ImagePullBackOff`. עם `Never` הוא משתמש רק ב-image שכבר בנוי אצלנו במחשב, ואף פעם לא מנסה להוריד.

## 2. פענוח `POSTGRES_PASSWORD` מה-Secret: איזו פקודה הרצתי? מה זה אומר על האבטחה של Secret?

פקודה:
`[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("c2hvcHBhc3M="))`
התוצאה: `shoppass`

תשובה:
base64 הוא רק קידוד, לא הצפנה. כל אחד יכול לפענח אותו בפקודה אחת, בלי מפתח. גם `kubectl exec -n shoplist deployment/backend -- env` מציג את `DB_PASSWORD=shoppass` גלוי. כלומר Secret לא באמת מגן על הסיסמה. הוא רק מפריד אותה מקבצי ה-Deployment, ומאפשר לתת הרשאות נפרדות. לכן לא עושים commit לקובץ Secret עם סיסמה אמיתית.

## 3. מחקתי את ה-pod של postgres והמוצרים נשארו. איזה קובץ אחראי לזה, ואיזו פקודה כן הייתה מוחקת אותם?

תשובה:
`postgres-pvc.yml`. ה-PVC שומר את הנתונים מחוץ ל-pod, והוא מחובר ל-`/var/lib/postgresql/data` ב-`postgres-deployment.yml`. כשה-pod נמחק, ה-Deployment מקים pod חדש שמתחבר לאותו PVC, והנתונים עדיין שם.
הפקודה שכן הייתה מוחקת אותם: `kubectl delete pvc postgres-pvc -n shoplist`. גם `kubectl delete namespace shoplist` מוחקת אותם, כי היא מוחקת את כל מה שב-namespace, כולל ה-PVC.

## 4. ב-Compose היה `depends_on`, ב-Kubernetes אין. מה קורה אם ה-backend עולה לפני postgres? מה דואג שהוא לא יקבל בקשות לפני שהוא מוכן?

תשובה:
אם backend עולה לפני postgres, הוא לא מצליח להתחבר למסד הנתונים. אם הוא נופל, Kubernetes מפעיל אותו מחדש שוב ושוב, עד ש-postgres מוכן. רואים את זה בעמודה RESTARTS.
מה שדואג שהוא לא יקבל בקשות הוא ה-`readinessProbe`, שבודק את `/health` בפורט 5000. כל עוד הבדיקה לא עוברת, ה-pod לא מסומן כמוכן (`0/1`), וה-Service לא שולח אליו בקשות: הוא לא מופיע ב-endpoints.

## 5. ד8 (מדבקה שגויה `app: backnd`): מה ראיתי בדפדפן? האם ה-pods של backend נפלו? מה הראה `get endpoints`, ולמה?

מה ראיתי בדפדפן:
הנוריות הפכו לאדומות, והאפליקציה לא הצליחה לטעון או להוסיף מוצרים: ה-frontend לא הצליח להגיע ל-backend.

מצב ה-pods:
ה-pods של backend לא נפלו. שניהם נשארו `1/1 Running`.

הפלט של `kubectl get endpoints backend -n shoplist`:
`backend   <none>`
וב-`kubectl describe service backend` הופיע `Selector: app=backnd`, ושורת `Endpoints:` הייתה ריקה.

ההסבר:
ה-Service מוצא את ה-pods שלו לפי label, לא לפי שם. ה-selector שלו חיפש `app=backnd`, אבל ל-pods יש label `app=backend`. אף pod לא התאים, ולכן לא היו ל-Service כתובות, והבקשות לא הגיעו לאף pod, למרות שה-pods עצמם תקינים. אחרי התיקון ל-`app: backend` חזרו שתי כתובות: `10.1.0.31:5000,10.1.0.33:5000`.

## 6. שיניתי ערך ב-`app-config` והרצתי `apply`, אבל ה-backend עדיין רואה את הערך הישן. למה, ומה הפתרון?

תשובה:
משתני סביבה נקראים רק פעם אחת, כשה-pod עולה. `apply` משנה את ה-ConfigMap, אבל לא את ה-pods שכבר רצים. הפתרון הוא להפעיל את ה-pods מחדש:
`kubectl rollout restart deployment/backend -n shoplist`
ה-pods החדשים קוראים את הערך העדכני.

## 7. כמה פעמים כתבתי `shoplist` ב-12 הקבצים? אם נרצה עותק שני ב-namespace `shoplist-test`, כמה קבצים נצטרך לשנות?

תשובה:
`namespace: shoplist` כתוב 12 פעמים, פעם בכל קובץ. בנוסף `name: shoplist` מופיע ב-`namespace.yml`. בשביל `shoplist-test` צריך לשנות את כל 13 הקבצים, אחד אחד, ובכל אחד קל לשכוח או לטעות. הבעיה: אותו ערך חוזר בהרבה קבצים בלי משתנה אחד שמרכז אותו. את זה פותר Helm: כותבים את ה-namespace פעם אחת כמשתנה, ומשנים רק במקום אחד.