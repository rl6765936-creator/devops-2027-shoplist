# בסיס: Node.js 24 על Alpine (קטן)
FROM node:24-alpine
# תיקיית העבודה בתוך ה-container
WORKDIR /app
# הקבצים של השירות; אין npm install כי אין חבילות חיצוניות
COPY package.json server.js ./
# תיעוד: stats מאזין בפורט 3000
EXPOSE 3000
# הפעלת השרת, בצורת exec
CMD ["node", "server.js"]
