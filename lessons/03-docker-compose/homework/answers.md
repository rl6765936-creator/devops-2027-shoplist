# שיעור 3 — תשובות

**1. כמה containers ירוצו? כמה images ייבנו?**
ירוצו 2 containers, אחד לכל service. ייבנו 2 images, כי Compose בונה image נפרד לכל service שיש לו `build` (השמות יהיו בערך `homework-greeter-he` ו-`homework-greeter-en`). אבל הם בנויים מאותו Dockerfile, ולכן ה-build השני כמעט כולו `CACHED` והשכבות משותפות, כך שבפועל זה לא תופס מקום כפול.

**2. מה יחזירו הכתובות?**
- `http://localhost:9001/hello/Rivka` מחזירה `{"message": "Shalom, Rivka!"}`
- `http://localhost:9002/hello/Rivka` מחזירה `{"message": "Hello, Rivka!"}`

**3. למה אי אפשר אותו פורט במחשב, אבל אפשר אותו פורט בתוך ה-container?**
פורט במחשב יכול להיות תפוס רק על ידי תהליך אחד, ולכן אם שני services יבקשו את 9001, השני ייכשל. לעומת זאת, לכל container יש רשת משלו (כתובת IP משלו), אז פורט 8000 בתוך כל אחד מהם הוא פורט נפרד ואין התנגשות.

**4. מה מוסיפים בשביל greeter בצרפתית?**
עוד service מתחת ל-`services:`:

```yaml
  # greeter בצרפתית: מחזיר "Bonjour", נגיש במחשב בפורט 9003
  greeter-fr:
    build:
      context: ../../02-docker-basics/homework/greeter
    environment:
      GREETING: Bonjour
    ports:
      - "9003:8000"
```

**5. למה greeter לא צריך volume, ו-postgres כן?**
greeter לא שומר שום מידע: כל תשובה מחושבת מחדש מהבקשה וממשתנה הסביבה. אם מוחקים את ה-container לא הולך לאיבוד כלום. postgres שומר נתונים (המוצרים), ומה שנכתב בתוך container נמחק יחד איתו, ולכן צריך volume ששומר את הנתונים מחוץ ל-container.
