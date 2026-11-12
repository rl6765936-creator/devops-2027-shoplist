# בסיס: Python 3.11 בגרסה מצומצמת
FROM python:3.11-slim
# תיקיית העבודה בתוך ה-container
WORKDIR /app
# קודם רק קובץ התלויות, כדי שה-cache של ההתקנה יישמר כשהקוד משתנה
COPY requirements.txt .
# התקנת התלויות; trusted-host בגלל הרשת של נטפרי
RUN pip install --no-cache-dir --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.txt
# רק עכשיו הקוד עצמו
COPY app.py .
# תיעוד: ה-backend מאזין בפורט 5000
EXPOSE 5000
# הפעלת השרת, בצורת exec
CMD ["python", "app.py"]
