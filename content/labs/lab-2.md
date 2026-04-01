## Тема, Мета

**Тема:** СТВОРЕННЯ БАЗИ ДАНИХ У MYSQL. ПІДКЛЮЧЕННЯ NODE.JS ДО MYSQL. РОБОТА З ORM SEQUELIZE

**Мета:** 
- Навчитися створювати базу даних у MySQL
- Освоїти виконання SQL-запитів (SELECT, INSERT, UPDATE, DELETE)
- Підключати серверну програму на Node.js до бази даних
- Використовувати ORM Sequelize для роботи з БД
- Реалізувати зв’язок One-to-Many між таблицями

## Завдання
## №1. Створити базу даних, яка буде використовуватись вашим вебдодатком.

Для забезпечення роботи вебдодатку «Global Talk» було використано реляційну систему керування базами даних (RDBMS) — MySQL. 

За допомогою інструментарію XAMPP (phpMyAdmin) було створено базу даних з назвою backend_globaltalk.


![створення бд](/assets/labs/lab-2/db.jpg)
![xampp](/assets/labs/lab-2/xampp.jpg)


## №2. Створити таблиці у вашій базі даних.

Згідно з функціональними вимогами проєкту, у базі даних було реалізовано 5 основних таблиць. Структура таблиць розроблена з урахуванням типів даних та необхідних зв'язків:

- Users: зберігає дані для авторизації клієнтів та адміністраторів.

- Teachers: містить інформацію про викладацький склад школи.

- Courses: основна таблиця з переліком мовних напрямків, рівнів та вартості навчання.

- Applications: фіксує звернення клієнтів через форму зворотного зв'язку для запису на пробне заняття.

- Reviews: зберігає зворотний зв'язок від студентів з оцінкою від 1 до 5.

![tables](/assets/labs/lab-2/tables.jpg)

![structure](/assets/labs/lab-2/structure.jpg)

```
CREATE TABLE Users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('client', 'admin') DEFAULT 'client'
);

CREATE TABLE Teachers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    full_name VARCHAR(255) NOT NULL,
    qualification VARCHAR(255),
    experience VARCHAR(100)
);

CREATE TABLE Courses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    language VARCHAR(100) NOT NULL,
    level VARCHAR(50),
    price DECIMAL(10, 2) NOT NULL,
    duration VARCHAR(100),
    teacherId INT,
    FOREIGN KEY (teacherId) REFERENCES Teachers(id) ON DELETE SET NULL ON UPDATE CASCADE
);

CREATE TABLE Applications (
    id INT AUTO_INCREMENT PRIMARY KEY,
    userId INT,
    courseId INT,
    phone VARCHAR(20) NOT NULL,
    status VARCHAR(50) DEFAULT 'Нова',
    FOREIGN KEY (userId) REFERENCES Users(id) ON DELETE CASCADE,
    FOREIGN KEY (courseId) REFERENCES Courses(id) ON DELETE CASCADE
);

CREATE TABLE Reviews (
    id INT AUTO_INCREMENT PRIMARY KEY,
    client_name VARCHAR(255) NOT NULL,
    text TEXT NOT NULL,
    rating INT CHECK (rating >= 1 AND rating <= 5)
);
```

## №3. Виконати SQL-запити

    - INSERT
```
INSERT INTO Users (name, email, password, role) VALUES 
('Марина Бондар', 'maryna@example.com', 'pass123', 'client'),
('Олексій Швець', 'oleksii@example.com', 'secure456', 'client'),
('Юлія Бєлова', 'yulia@example.com', 'member789', 'client'),
('Адміністратор', 'admin@globaltalk.com', 'admin_globaltalk', 'admin');
```

```
INSERT INTO Teachers (full_name, qualification, experience) VALUES 
('Олена Петренко', 'English C2 Expert', '8 років'),
('Маркус Шмідт', 'Native Speaker (German)', '10 років'),
('Анна Ковальська', 'Polish Philology Master', '5 років');
```

```
INSERT INTO Courses (title, language, level, price, duration, teacherId) VALUES 
('Англійська для IT', 'English', 'B2', 500.00, '3 місяці', 1),
('Бізнес-німецька', 'German', 'C1', 600.00, '4 місяці', 2),
('Польська для вступу', 'Polish', 'B1', 400.00, '2 місяці', 3);
```

```
INSERT INTO Applications (userId, courseId, phone, status) 
VALUES (1, 1, '+380991112233', 'Нова');

INSERT INTO Applications (userId, courseId, phone, status) 
VALUES (2, 2, '+380504445566', 'Опрацьовано');
```
```
INSERT INTO Reviews (client_name, text, rating) 
VALUES ('Марина Бондар', 'Чудова школа! Завдяки курсу англійської для IT змогла впевнено пройти співбесіду', 5);

INSERT INTO Reviews (client_name, text, rating) 
VALUES ('Олексій Швець', 'Неймовірний викладач німецької. Матеріал подається дуже зрозуміло та цікаво.', 5);
```

    - SELECT
    
    Перегляд усіх курсів із іменами викладачів (JOIN)
```
SELECT 
    Courses.title AS 'Назва курсу', 
    Courses.level AS 'Рівень', 
    Courses.price AS 'Вартість', 
    Teachers.full_name AS 'Викладач'
FROM Courses
JOIN Teachers ON Courses.teacherId = Teachers.id;
```
![select1](/assets/labs/lab-2/select1.jpg)

Пошук курсів конкретної мови (WHERE)
```
SELECT title, level, price 
FROM Courses 
WHERE language = 'English';
```
![select2](/assets/labs/lab-2/select2.jpg)

Перегляд заявок із деталями курсу та клієнта
```
SELECT 
    Applications.id AS '№ Заявки',
    Users.name AS 'Клієнт', 
    Applications.phone AS 'Телефон', 
    Courses.title AS 'Обраний курс',
    Applications.status AS 'Статус'
FROM Applications
JOIN Users ON Applications.userId = Users.id
JOIN Courses ON Applications.courseId = Courses.id;
```

![select3](/assets/labs/lab-2/select3.jpg)

    - UPDATE
   
Оновлення ціни курсу
```
UPDATE Courses 
SET price = 540.00, 
    title = 'Бізнес-німецька (АКЦІЯ)'
WHERE id = 2;
```

![update1](/assets/labs/lab-2/update1.jpg)

Оновлення досвіду викладача

```
UPDATE Teachers 
SET experience = '9 років' 
WHERE full_name = 'Олена Петренко';
```
![update2](/assets/labs/lab-2/update2.jpg)

    - DELETE 

Видалення конкретного відгуку

```
DELETE FROM Reviews 
WHERE id = 1;
```
![delete1](/assets/labs/lab-2/delete1.jpg)
![delete2](/assets/labs/lab-2/delete2.jpg)

## №4, 6. Підключити Node.js до MySQL через пакет mysql2. Використати ORM Sequelize

**Підготовка проекту**

Команда ```node -v```: переконалися, що середовище виконання JavaScript встановлено коректно і готове до роботи.

Команда ```npm init -y``` автоматично створила файл ```package.json.```

![task4](/assets/labs/lab-2/task4.jpg)


Команда ```npm install sequelize mysql2 express``` завантажила необхідні інструменти:

```express```: каркас для створення вебсервера.

```mysql2```: драйвер, який дозволяє Node.js фізично "спілкуватися" з базою даних MySQL у XAMPP.

```sequelize```: ORM-система, яка дозволяє працювати з таблицями бази даних як з об'єктами в коді.

Після виконання команди встановлення пакетів ```npm install```, менеджер пакетів автоматично сформував дерево залежностей у папці ```node_modules``` та створив файл ```package-lock.json``` для фіксації версій бібліотек. Це забезпечує цілісність та відтворюваність середовища розробки.

![task4(1)](/assets/labs/lab-2/task4(1).jpg)

**Конфігурація бази даних**

Після встановлення пакетів необхідно було створити конфігураційний файл, який вказує Sequelize, як саме знайти базу в XAMPP.

Створено файл ```config/database.js.```
Ми ініціалізували об'єкт Sequelize, передавши йому назву БД (backend_globaltalk), стандартного користувача (root) та хост (localhost). Тепер програма має "ключі" від бази даних і готова надсилати туди команди через драйвер mysql2.

```
const { Sequelize } = require('sequelize');

// Налаштування підключення до БД
const sequelize = new Sequelize('backend_globaltalk', 'root', '', {
  host: 'localhost',
  dialect: 'mysql',
  logging: false
});

// Перевірка підключення 
sequelize.authenticate()
  .then(() => console.log('З’єднання з БД встановлено успішно.'))
  .catch(err => console.error('Помилка підключення:', err));

module.exports = sequelize;
```

Замість того, щоб писати складні текстові SQL-запити, ми перейшли до об'єктно-реляційного відображення (ORM).

Sequelize дозволяє сприймати таблиці бази даних як звичайні JavaScript-класи.

Це захищає  проєкт від помилок у синтаксисі SQL та робить код набагато зрозумілішим. Тепер будь-яка зміна в коді автоматично відображається в структурі таблиць завдяки синхронізації.

**Основний файл сервера та синхронізація**

```server.js``` - основний файл сервера та синхронізація
Тут відбувається запуск програми та фізичне створення таблиць у базі даних.

```
const express = require('express');
const cors = require('cors');
const path = require('path');
const sequelize = require('./config/database');
const { User, Course, Teacher, Application, Review } = require('./models/index'); 

const app = express();
const PORT = 3000;

// Мідлвари 
app.use(cors());
app.use(express.json()); // Дозволяє читати JSON, який надсилає JS
app.use(express.static(__dirname)); // Дозволяє браузеру бачити CSS та JS файли

// Головна сторінка
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'index.html'));
});

// --- API МАРШРУТИ ---
// курси
app.get('/api/courses', async (req, res) => {
  try {
    const courses = await Course.findAll({ include: [Teacher] });
    res.json(courses);
  } catch (error) {
    res.status(500).json({ message: "Помилка курсів", error });
  }
});

// викладачі
app.get('/api/teachers', async (req, res) => {
  try {
    const teachers = await Teacher.findAll();
    res.json(teachers);
  } catch (error) {
    res.status(500).json({ message: "Помилка викладачів", error });
  }
});

// відгуки
app.get('/api/reviews', async (req, res) => {
  try {
    const reviews = await Review.findAll();
    res.json(reviews);
  } catch (error) {
    res.status(500).json({ message: "Помилка відгуків", error });
  }
});

// вхід 
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  try {
    const user = await User.findOne({ where: { email, password } });
    if (user) {
      res.json({ success: true, user: { id: user.id, name: user.name, role: user.role } });
    } else {
      res.status(401).json({ success: false, message: "Невірний email або пароль" });
    }
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// реєстарція
app.post('/api/register', async (req, res) => {
  try {
    const { name, email, password } = req.body;

    const candidate = await User.findOne({ where: { email } });
    if (candidate) {
      return res.status(400).json({ success: false, message: "Цей email вже зайнятий" });
    }

    // створення нового користувача
    const newUser = await User.create({
      name,
      email,
      password,
      role: 'client'
    });

    res.status(201).json({ 
      success: true, 
      user: { id: newUser.id, name: newUser.name, role: newUser.role } 
    });
  } catch (error) {
    console.error("Error during registration:", error); // Побачите помилку в терміналі
    res.status(500).json({ success: false, message: "Помилка на сервері при реєстрації", error: error.message });
  }
});

// заявка
app.post('/api/apply', async (req, res) => {
  const { userId, courseId, phone } = req.body;
  try {
    const newApp = await Application.create({ userId, courseId, phone, status: 'Нова' });
    res.status(201).json({ message: "Успішно!", data: newApp });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// старт сервера
async function startServer() {
  try {
    await sequelize.sync({ alter: true }); 
    console.log("Базу даних синхронізовано.");
    app.listen(PORT, () => console.log(`Сервер: http://localhost:${PORT}`));
  } catch (error) {
    console.error("Помилка старту:", error);
  }
}

startServer();
```
![server](/assets/labs/lab-2/server.png)


## №7,8. Створити моделі User та Course. Реалізувати зв’язок One-to-Many

Модель — це програмна "схема" таблиці.
Створюємо два окремі файли в папці models/, які визначають структуру даних для мовної школи.

Описуємо структуру таблиці Users як JavaScript-об’єкт.

```
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const User = sequelize.define('User', {
  name: { type: DataTypes.STRING, allowNull: false },
  email: { type: DataTypes.STRING, allowNull: false, unique: true },
  password: { type: DataTypes.STRING, allowNull: false },
  role: { 
    type: DataTypes.ENUM('client', 'admin'), 
    defaultValue: 'client' 
  }
}, { timestamps: false }); 

module.exports = User;
```

Створюємо модель для курсів, яка буде пов’язана з користувачем.

```
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Course = sequelize.define('Course', {
  title: { type: DataTypes.STRING, allowNull: false },
  language: { type: DataTypes.STRING, allowNull: false },
  level: { type: DataTypes.STRING },
  price: { type: DataTypes.DECIMAL(10, 2), allowNull: false },
  duration: { type: DataTypes.STRING }
}, { timestamps: false });

module.exports = Course;
```
Файл  ```models/index.js``` збирає всі моделі до купи та встановлює правила їхньої взаємодії.

```
const User = require('./User');
const Teacher = require('./Teacher');
const Course = require('./Course');
const Application = require('./Application');
const Review = require('./Review');

//Один вчитель має багато курсів (One-to-Many) 
Teacher.hasMany(Course, { foreignKey: 'teacherId' });
Course.belongsTo(Teacher, { foreignKey: 'teacherId' }); 

// Користувач може мати багато заявок (One-to-Many) 
User.hasMany(Application, { foreignKey: 'userId' });
Application.belongsTo(User, { foreignKey: 'userId' });

// Курс може мати багато заявок (One-to-Many)
Course.hasMany(Application, { foreignKey: 'courseId' });
Application.belongsTo(Course, { foreignKey: 'courseId' });

module.exports = {
  User,
  Teacher,
  Course,
  Application,
  Review
};
```

## №5. Виконати SQL-запити з Node.js. 

```test-db.js``` - скрипт для демонстрації CRUD та SQL запитів

```
const { User, Course, Review } = require('./models/index');
const sequelize = require('./config/database');

async function demo() {
  try {
    console.log("--- ПОЧАТОК ДЕМОНСТРАЦІЇ CRUD ---");

    // INSERT 
    const user1 = await User.create({
      name: 'Іван Петренко',
      email: 'ivan@gmail.com',
      password: 'password123',
      role: 'client'
    });

    const user2 = await User.create({
      name: 'Марія Ковальчук',
      email: 'maria@gmail.com',
      password: 'securepass',
      role: 'client'
    });
    console.log("Створено двох користувачів: Іван та Марія.");


    const englishCourse = await Course.create({
      title: 'Англійська для початківців',
      language: 'English',
      level: 'A1',
      price: 1000.00,
      duration: '3 місяці'
    });
    console.log("Додано курс:", englishCourse.title);


    const newReview = await Review.create({
      client_name: 'Іван Петренко',
      text: 'Чудовий курс, все зрозуміло!',
      rating: 5
    });
    console.log("Додано відгук від:", newReview.client_name);

    // SELECT
    const allUsers = await User.findAll();
    console.log(`Всього користувачів у базі: ${allUsers.length}`);

    const allCourses = await Course.findAll();
    console.log(`Доступні курси: ${allCourses.map(c => c.title).join(', ')}`);

    // UPDATE
    await Course.update({ price: 1250.50 }, {
      where: { id: englishCourse.id }
    });
    console.log("Ціну курсу 'Англійська для початківців' оновлено до 1250.50.");

    // DELETE 
    //const firstReview = await Review.findOne();
    //if (firstReview) {
    //  await Review.destroy({ where: { id: firstReview.id } });
    //  console.log(`🗑️ Відгук від ${firstReview.client_name} видалено.`);
   // }

    console.log("--- ДЕМОНСТРАЦІЮ ЗАВЕРШЕНО ---");
    process.exit();
  } catch (error) {
    console.error("Помилка під час виконання запитів:", error);
    process.exit(1);
  }
}

demo();
```

![test](/assets/labs/lab-2/test.jpg)

## Висновки

У ході роботи було опановано створення реляційної бази даних у MySQL та її інтеграцію з Node.js. На практиці реалізовано об’єктно-реляційне відображення за допомогою ORM Sequelize, що дозволило працювати з таблицями як з JavaScript-об’єктами.

Основні досягнення:
- Конфігурація: налаштовано підключення до БД через драйвер mysql2 та Sequelize.

- Моделювання: створено моделі User, Course, Teacher, Review та Application.

- Зв'язки: реалізовано зв’язок One-to-Many, зокрема між викладачами та курсами.

- CRUD-операції: виконано повний цикл маніпуляції даними (INSERT, SELECT, UPDATE, DELETE) через методи Sequelize (create, findAll, update, destroy).Використання ORM спростило розробку, забезпечило автоматичну синхронізацію моделей із БД та підвищило безпеку й швидкість написання серверного коду.