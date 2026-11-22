# שיעור 7 — תשובות

## 1. מה ההבדל בין chart ל-release? תנו דוגמה מהשיעור.
chart הוא התבנית: התיקייה `helm/shoplist` עם `Chart.yaml`, `values.yaml` ו-`templates/`. release הוא התקנה אחת של ה-chart ב-cluster, עם שם משלה. בשיעור: ה-chart הוא `shoplist`, וה-release הוא `shop`, ב-namespace `shop`. מאותו chart אפשר להתקין כמה releases, למשל `shop2`.

## 2. למה הסרנו את `namespace.yml` מה-chart, ומה מחליף אותו?
כי ה-namespace לא אמור להיות קבוע בתוך ה-chart: בוחרים אותו בזמן ההתקנה, עם `-n shop`. במקום הקובץ, בתבניות כתוב `{{ .Release.Namespace }}`, ואת ה-namespace יוצר הדגל `--create-namespace`. כך אותו chart יכול לרוץ בכל namespace, בלי לשנות קבצים.

## 3. בד5 כתבתי ב-`values-big.yaml` רק שני ערכים. מאיפה הגיעו כל השאר, למשל `nodePort`?
מ-`values.yaml`. Helm תמיד טוען קודם את `values.yaml` כבסיס, ואז את הקובץ שב-`-f`, שדורס רק את הערכים שכתובים בו (`backend.replicas: 3`, `stats.replicas: 2`). כל שאר הערכים, כולל `nodePort: 30080`, נשארים מהבסיס.

## 4. בד6, האפליקציה עבדה גם כשה-image היה שבור. למה?
כי Kubernetes מעדכן ב-rolling update: הוא מסיר pod ישן רק אחרי שה-pod החדש מוכן. ה-pod החדש של backend נתקע ב-`ErrImageNeverPull`, ולכן אף pod ישן לא הוסר, ושלושת ה-pods הישנים המשיכו לרוץ. ה-Service שולח בקשות רק ל-pods מוכנים (readinessProbe משיעור 6), כך שכל הבקשות הגיעו ל-pods התקינים.

## 5. אחרי ה-rollback יש 4 revisions ולא 2. למה Helm לא פשוט חוזר ל-revision 2?
Helm לא מוחק היסטוריה, הוא רק מוסיף לה. rollback יוצר revision חדש (4) עם ההגדרות של revision 2, והתיאור שלו הוא `Rollback to 2`. כך רואים בדיוק מה קרה ומתי, כולל הגרסה השבורה (3). אפשר גם לחזור אחורה שוב, ואפילו לבטל את ה-rollback עצמו.

## 6. מה היה קורה לשלושת המוצרים בד7 בלי השורה `helm.sh/resource-policy: keep`?
הם היו נמחקים. `helm uninstall` מוחק את כל מה שה-release יצר, כולל ה-PVC שבו postgres שומר את הנתונים. בהתקנה מחדש היה נוצר PVC חדש וריק, והרשימה הייתה ריקה. עם הפתק, Helm השאיר את `postgres-pvc` (וכתב: `These resources were kept due to the resource policy`), ו-postgres החדש התחבר אליו ומצא את המוצרים.

## 7. חברה כתבה בטעות `{{ .Values.backend.replica }}`. Helm לא נתן שגיאה. מה כן קרה, ואיך היא הייתה מגלה?
אין ב-`values.yaml` מפתח בשם `replica`, ולכן Helm מילא את החור בערך ריק, בלי שגיאה. בקובץ הסופי נוצרה שורה `replicas:` בלי מספר, ו-Kubernetes השתמש בברירת המחדל, 1. כך השינוי ב-`values.yaml` (למשל 3 ב-`values-big.yaml`) לא משפיע. מגלים את זה עם `helm template`: מחפשים שורה ריקה אחרי `replicas:`. או עם `kubectl get pods`, שמראה רק pod אחד במקום 3.

## 8. מה יקרה ל-postgres אם ישתמש ביותר מ-512Mi זיכרון? ומה אם ישתמש ביותר מליבה אחת של CPU? למה ההתנהגות שונה?
זיכרון: ה-container נהרג (`OOMKilled`), ו-Kubernetes מפעיל אותו מחדש. CPU: הוא לא נהרג, אלא מואט (throttling), ומקבל רק את החלק שלו. ההבדל: CPU הוא זמן, ואפשר פשוט לתת פחות ממנו, ותהליך שמחכה יכול להמשיך אחר כך. זיכרון כבר תפוס, ואי אפשר "לקחת אותו בחזרה" בלי לעצור את התהליך.

## 9. למה הגיבוי הוא CronJob ולא Deployment? מה היה קורה אם הייתי כותבת אותו כ-Deployment?
גיבוי הוא משימה שצריכה לרוץ ולהסתיים, בזמן קבוע. CronJob יוצר Job לפי שעון (`0 2 * * *`), וה-Job מריץ את הגיבוי עד שהוא מסתיים בהצלחה (`Complete`, `1/1`). Deployment דואג שה-pod ירוץ תמיד: ברגע שהגיבוי היה מסתיים, Deployment היה מקים את ה-pod מחדש, והגיבוי היה רץ שוב ושוב בלי הפסקה, וממלא את הארונית בקבצים.