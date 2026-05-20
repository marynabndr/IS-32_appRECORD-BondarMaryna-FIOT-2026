## Тема,  Мета

**Тема:** Безпека та продуктивність серверних додатків. Безпека Node.js-додатків. Оптимізація запитів і кешування. Тестування API.

**Мета:**
-   навчитися реалізовувати захист backend-додатків; 
-	використовувати механізми кешування; 
-	оптимізувати REST API; 
-	виконувати автоматизоване тестування;
-   оцінювати продуктивність серверних застосунків

## Практичні завдання

1.	Створити REST API на Node.js та Express. 
2.	Реалізувати захист API: 
    -	Helmet; 
    -	rate-limit; 
    -	валідацію даних. 
3.	Реалізувати кешування відповідей. 
4.	Оптимізувати один із маршрутів API. 
5.	Провести тестування API. 
6.	Проаналізувати продуктивність до та після оптимізації. 
7.	Оформити звіт. 

Реалізувати:
-	JWT-автентифікацію; 
-	Redis-кешування; 
-	Swagger документацію; 
-	Docker-контейнеризацію; 
-	навантажувальне тестування через Artillery. 

### Налаштування безпеки (Helmet, Rate-limit, Валідація)

Необхідні бібліотеки: ```express-rate-limit```  ```express-validator``` ```helmet```

```
const { body, validationResult } = require('express-validator');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet'); 
```

**Helmet** - набір мідлвар для налаштування безпекових HTTP-заголовків. За замовчуванням Express видає багато інформації про себе (наприклад, заголовок X-Powered-By: Express), що допомагає хакерам. Helmet це приховує та додає нові правила: 
- Content-Security-Policy: Запобігає XSS (міжсайтовому скриптингу).
- X-Frame-Options: Захищає від Clickjacking (коли ваш сайт намагаються відкрити у фреймі іншого сайту).
- Strict-Transport-Security: Примушує браузери використовувати тільки HTTPS.

```
// --- 1. HELMET (БЕЗПЕКА ЗАГОЛОВКІВ) ---
app.use(helmet()); 

app.use(cors());
app.use(express.json());
```


**Express-Rate-Limit** - механізм обмеження частоти запитів. Хакер не зможе автоматично підбирати паролі тисячі разів на секунду, бо його IP заблокується після 5-10 спроб (якщо налаштувати лімітер окремо на логін).

```
// --- 2. EXPRESS-RATE-LIMIT (ЗАХИСТ ВІД ПЕРЕВАНТАЖЕННЯ) ---
// Обмежуємо кількість запитів
const apiLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, //15 хвилин
    max: 5, // Максимум 5 запитів з однієї IP 
    standardHeaders: true, 
    legacyHeaders: false, 
    message: {
        status: 429,
        message: "Забагато запитів з вашої адреси. Спробуйте через 15 хвилин."
    }
});
// до всіх маршрутів API
app.use('/api/', apiLimiter);
```

**Express-Validator** - бібліотека для перевірки та очищення (sanitization) вхідних даних. Ви запобігаєте потраплянню шкідливого коду в базу даних (наприклад, скриптів у полі імені). Ви гарантуєте, що в полі email справді email, а не випадковий текст. Ви повертаєте користувачу чіткі повідомлення про помилки.

```
// --- 3. EXPRESS-VALIDATOR (ВАЛІДАЦІЯ ДАНИХ) ---
// Приклад маршруту реєстрації з валідацією
app.post('/api/register', [
    // Правила валідації
    body('email')
        .isEmail().withMessage('Введіть коректну адресу email')
        .normalizeEmail(),
    body('password')
        .isLength({ min: 6 }).withMessage('Пароль має бути не менше 6 символів'),
    body('name')
        .trim()
        .notEmpty().withMessage("Ім'я не може бути порожнім")
], (req, res) => {
    // Перевірка результатів валідації
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        // Якщо є помилки — 400 Bad Request
        return res.status(400).json({ 
            success: false, 
            errors: errors.array() 
        });
    }

    // Якщо валідація пройшла успішно
    const { email, name } = req.body;
    res.status(201).json({
        success: true,
        message: `Користувач ${name} успішно зареєстрований!`
    });
});
```

### Тестування Helmet (Безпека заголовків)

Інструменти розробника -> вкладка Networ (Мережа).

Заголовки, яких не було раніше, наприклад:
- Content-Security-Policy
- X-Content-Type-Options: nosniff
- X-Frame-Options: SAMEORIGIN
- Відсутність заголовка X-Powered-By: Express (Helmet його видаляє для безпеки).

![Скрін 2](/assets/labs/lab-5/image2.png)
![Скрін 2](/assets/labs/lab-5/image1.png)

### Тестування Express-Rate-Limit (Захист від перевантаження)

POST - http://localhost:3000/api/teachers - відправили 6 разів

Отримали статус 429 Too Many Requests. Це прямий доказ того, що Rate Limiting працює належним чином.

![Скрін 2](/assets/labs/lab-5/image3.png)

### Тестування Express-Validator (Валідація даних)

POST - http://localhost:3000/api/register

Body -> raw -> JSON 

Відправляємо "неправильний" об'єкт

```
{
  "email": "not-an-email",
  "password": "123",
  "name": ""
}
```

![Скрін 2](/assets/labs/lab-5/image4.png)

### Кешування

Кешування — це процес збереження копії даних у спеціальному тимчасовому сховищі (кеші) для того, щоб наступні запити на ці ж дані виконувалися значно швидше.

Завантажена бібліотека ```node-cache```

```
const NodeCache = require('node-cache');
// Створюємо екземпляр кешу. stdTTL: 60 означає, що дані будуть зберігатися 60 секунд.
const myCache = new NodeCache({ stdTTL: 60, checkperiod: 120 });
```

```
app.get('/api/courses', async (req, res) => {
  try {
    const cacheKey = 'test_courses';
    const cached = myCache.get(cacheKey);

    // Якщо дані є в кеші — повертаємо об'єкт із поміткою 'cache'
    if (cached) {
      return res.json({ 
        source: 'cache', 
        data: cached 
      }); 
    }

    const courses = await Course.findAll();

    // Зберігаємо в кеш
    myCache.set(cacheKey, courses);
    
    // Якщо даних в кеші не було — повертаємо об'єкт із поміткою 'database'
    res.json({ 
      source: 'database', 
      data: courses 
    }); 

  } catch (error) {
    res.status(500).json({ message: "Помилка сервера", error: error.message });
  }
});
```

### Тестування кешування 

GET - http://localhost:3000/api/courses

- Source: database — показує, що сервер вміє ходити в БД, коли кеш порожній.
- Source: cache — доводить, що реалізовано кешування, і сервер тепер віддає дані миттєво, не навантажуючи базу даних. 

![Скрін 2](/assets/labs/lab-5/image6.png)


### Swagger документація

Swagger (OpenAPI) — це набір інструментів, який дозволяє автоматично створювати інтерактивну документацію для API.

Встановлено бібліотеку
```
npm install swagger-ui-express swagger-jsdoc
```

```
const swaggerUi = require('swagger-ui-express');
const swaggerJsdoc = require('swagger-jsdoc');

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Global Talk API',
      version: '1.0.0',
      description: 'Документація API для Лабораторної роботи №5',
    },
    servers: [{ url: 'http://localhost:3000' }],
  },
  apis: ['./server.js'], // Вказуємо файл, де шукати опис маршрутів
};

const swaggerDocs = swaggerJsdoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocs));
```
Додаємо опис над маршрутом курсів (це потрібно, щоб Swagger знав, що описувати):

```
/**
 * @openapi
 * /api/courses:
 *   get:
 *     description: Отримання списку курсів з кешуванням
 *     responses:
 *       200:
 *         description: Успішне отримання даних
 */
app.get('/api/courses', async (req, res) => { ... });
```
```
http://localhost:3000/api-docs
```

![Скрін 2](/assets/labs/lab-5/image7.png)


### Автоматизоване тестування (Jest + Supertest)


Jest — це тестовий фреймворк (керує процесом), а Supertest — бібліотека, яка вміє імітувати HTTP-запити до вашого сервера без необхідності відкривати браузер чи Postman.

Команда для встановлення інструментів розробки:
```
npm install --save-dev jest supertest
```

```
module.exports = app; // Експортуємо додаток для тестів
```

Файл ```api.test.js```
```
const request = require('supertest');
const app = require('./server'); 
const sequelize = require('./config/database'); // Додаємо імпорт налаштувань бази

describe('Автоматизоване тестування API (Лабораторна №5)', () => {

  // Перед тестами синхронізуємо БД
  beforeAll(async () => {
    await sequelize.sync();
  });

  // Після тестів закриваємо з'єднання
  afterAll(async () => {
    await sequelize.close();
  });

  it('GET /api/courses має повертати статус 200 та масив даних', async () => {
    const res = await request(app).get('/api/courses');
    expect(res.statusCode).toBe(200);
    // Перевіряємо чи є поле source або data
    expect(res.body).toHaveProperty('data'); 
  });

  it('POST /api/register має повертати 400 при некоректних даних', async () => {
    const res = await request(app)
      .post('/api/register')
      .send({
        email: "not-an-email",
        password: "123",
        name: ""
      });
    expect(res.statusCode).toBe(400);
    expect(res.body).toHaveProperty('errors');
  });

});
```

```
npx jest api.test.js
```
Тест №1: Перевірка отримання курсів (GET /api/courses)

Тест №2: Перевірка захисту/валідації (POST /api/register) 

![Скрін 2](/assets/labs/lab-5/image8.png)


### Навантажувальне тестування (Artillery)

Навантажувальне тестування (Artillery) імітує ситуацію, коли на сайт заходить одночасно багато людей, і дозволяє побачити, чи витримає сервер таке навантаження.

```
npx artillery quick --count 10 -n 20 http://127.0.0.1:3000/api/courses
```
- --count 10: Створює 10 віртуальних користувачів одночасно.
- -n 20: Кожен користувач робить по 20 запитів (разом буде 200 запитів).

![Скрін 2](/assets/labs/lab-5/image9.png)

### Висновки

У ході виконання лабораторної роботи №5 було опановано методи забезпечення безпеки та оптимізації продуктивності серверних додатків на базі Node.js та Express.

Основні результати роботи:
1. **Безпека:** Завдяки впровадженню мідлвари Helmet було приховано критичну інформацію про технологічний стек (заголовок X-Powered-By) та налаштовано захисні HTTP-заголовки. Використання express-rate-limit дозволило захистити сервер від DDoS-атак та спроб підбору паролів (brute-force), що було підтверджено тестуванням.
2. **Валідація:** Реалізована перевірка вхідних даних за допомогою express-validator, що гарантує цілісність бази даних та захищає від некоректного введення.
3. **Оптимізація та кешування:** Впровадження механізму кешування через node-cache дозволило значно підвищити швидкість відповіді API. Навантажувальне тестування за допомогою Artillery показало, що час відповіді при використанні кешу скоротився до мінімальних значень (1–4 мс), що забезпечує високу масштабованість додатка.
4. **Документація та тестування:** Створено інтерактивну документацію за допомогою Swagger, що полегшує подальшу підтримку та інтеграцію API. Написано автоматизовані тести на базі Jest та Supertest, які дозволяють виконувати тестування ключових ендпоінтів.

**Підсумок:** Реалізований комплекс заходів дозволив створити стійкий до атак та високопродуктивний Backend-сервіс, готовий до роботи під високим навантаженням.