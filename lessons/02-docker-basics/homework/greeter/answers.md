1
docker build -t greeter:v1 lessons/02-docker-basics/homework/greeter
2
docker run -d -p 9000:8000 --name greeter greeter:v1
3
docker run -d -p 9000:8000 --name greeter -e GREETING=Shalom greeter:v1
4
{"message":"Shalom, Rivka!"}

                            
                                                                                                                                                                              5
אם נשנה רק את app.py, השלבים שלפני העתקת הקוד יישארו CACHED, כולל התקנת Flask.

השלב COPY app.py יתבצע מחדש, וגם ההוראות שאחריו (EXPOSE ו־CMD) יעובדו מחדש.

הסיבה היא שהתקנת הספריות נמצאת לפני העתקת קוד האפליקציה, ולכן שינוי בקוד לא מחייב התקנה חוזרת של הספריות.
