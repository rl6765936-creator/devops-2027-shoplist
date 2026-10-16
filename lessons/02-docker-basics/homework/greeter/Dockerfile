# שימוש בסביבת Python בגרסה 3.11
FROM python:3.11-slim
# הגדרת תיקיית העבודה בתוך הקונטיינר
WORKDIR /app
# העתקת קובץ התלויות מהמחשב לתיקיית העבודה בקונטיינר
COPY requirements.txt .
# התקנת הספריות המוגדרות בקובץ התלויות ללא שמירת קבצי מטמון
RUN pip install --no-cache-dir -r requirements.txt
# העתקת קובץ השרת לתיקיית העבודה בקונטיינר
COPY app.py .
# הצהרה שהשרת מאזין לפורט 8000 בתוך הקונטיינר
EXPOSE 8000
# הפעלת שרת ה-Python כאשר הקונטיינר מתחיל לרוץ
CMD ["python", "app.py"]
