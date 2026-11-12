# בסיס: Nginx על Alpine
FROM nginx:1.30-alpine
# קובצי האתר לתיקייה ש-Nginx מגיש ממנה
COPY index.html app.js style.css /usr/share/nginx/html/
# ההגדרות שלנו: הגשת האתר + reverse proxy ל-backend ול-stats
COPY nginx.conf /etc/nginx/conf.d/default.conf
# תיעוד: Nginx מאזין בפורט 80
EXPOSE 80
# אין CMD: יורשים אותו מה-image של nginx
