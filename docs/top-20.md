### Топ-20 часто используемых компонентов и шаблонов кода

Ниже представлен рейтинг ключевых компонентов/функциональных блоков, регулярно встречающихся в приложениях Node.js/TypeScript и Python. Для каждого компонента указано, *что он делает*, **типичный код/инструменты**, **языки** (где особенно распространён) и **насколько часто встречается**.

1. **Аутентификация и управление пользователями** – обеспечение входа пользователей, регистрации, управления сессиями и правами доступа. *Описание:* практически все приложения с индивидуальными пользователями включают механизм логина. Этот компонент отвечает за проверку учётных данных, хранение паролей (хэширование), выдачу токенов (JWT, OAuth) или создание сессии. *Типичный код:* определение моделей пользователя, эндпоинт `/login` для проверки пароля, генерация JWT токена или установка cookie-сессии. Например, код на Express с использованием **Passport.js** или **jsonwebtoken** для JWT, на Python – встроенная система **Django auth** или библиотека **flask-login**:

   ```js
   // Node.js + Express (JWT пример):
   app.post('/login', async (req, res) => {
       const { username, password } = req.body;
       const user = await Users.findOne({ username });
       if (!user || !user.validatePassword(password)) {
           return res.status(401).send('Unauthorized');
       }
       const token = jwt.sign({ id: user.id }, SECRET_KEY);
       res.json({ token });
   });
   ```

   ```python
   # Python Flask example:
   from flask import request, jsonify, session
   from werkzeug.security import check_password_hash
   @app.route('/login', methods=['POST'])
   def login():
       data = request.get_json()
       user = User.query.filter_by(username=data['username']).first()
       if not user or not check_password_hash(user.password_hash, data['password']):
           return jsonify({'error': 'Unauthorized'}), 401
       session['user_id'] = user.id  # using server-side session
       return jsonify({'message': 'Login successful'})
   ```

   *Инструменты и язык:* Node (Passport, JWT, Bcrypt), Python (Django auth, Flask-Login, FastAPI OAuth2). *Частота:* Исключительно высокая в веб-приложениях – по оценкам, **аутентификация требуется практически всегда**. Например, библиотека Passport.js загружена >1.7 млн раз в неделю (npm), а в Django аутентификация встроена по умолчанию. На GitHub тысячи проектов содержат реализацию JWT-авторизации.

2. **CRUD-операции и REST API эндпоинты** – стандартные операции создания, чтения, обновления, удаления сущностей + соответствующие маршруты API. *Описание:* CRUD (Create, Read, Update, Delete) – базовый функционал приложений, позволяющий пользователям управлять записями (посты блога, товары, учетные записи и пр.). Реализуется через REST API (или GraphQL) с методами POST/GET/PUT/DELETE. *Типичный код:* набор маршрутов или контроллеров, связанный с моделью данных. Например, REST-контроллер для сущности `Article`:

   ```js
   // Node/Express example for "articles" CRUD:
   app.get('/articles/:id', async (req, res) => {
       const article = await Article.findById(req.params.id);
       if (!article) return res.status(404).send('Not found');
       res.json(article);
   });
   app.post('/articles', async (req, res) => {
       const newArticle = new Article(req.body);
       await newArticle.save();
       res.status(201).json(newArticle);
   });
   // (Similarly PUT for update, DELETE for deletion)
   ```

   ```python
   # Flask example:
   @app.route('/articles/<int:id>', methods=['GET'])
   def get_article(id):
       article = Article.query.get_or_404(id)
       return jsonify(article.to_dict())
   @app.route('/articles', methods=['POST'])
   def create_article():
       data = request.get_json()
       article = Article(**data)
       db.session.add(article)
       db.session.commit()
       return jsonify(article.to_dict()), 201
   ```

   *Инструменты:* веб-фреймворки (Express, Django REST Framework, FastAPI, etc.), ORMs (Mongoose, SQLAlchemy) для удобной реализации CRUD. *Частота:* Крайне высокая. **CRUD – основа почти каждого веб-приложения**. Большинство внутренних инструментов состоят из однотипных форм и таблиц, реализующих CRUD. Например, Retool отмечает, что “большинство бизнес-ПО – это базовые CRUD-приложения”. Таким образом, компоненты CRUD повторяются повсеместно – от простых скриптов до сложных систем.

3. **Подключение к базе данных и ORM-модели** – шаблоны кода для взаимодействия с базами данных, определения моделей данных. *Описание:* Почти все приложения хранят состояние в базе данных (SQL или NoSQL). Компонент включает код инициализации соединения к БД, описание схемы (таблиц, коллекций) и выполнение запросов. *Типичный код:*

   * В Node.js: подключение с помощью драйвера или ORM (например, **Mongoose** для MongoDB, **TypeORM** или **Sequelize** для SQL).
   * В Python: настройка SQLAlchemy или Django ORM, описание классов-моделей.
     Например, модель и подключение в SQLAlchemy:

   ```python
   # models.py
   from flask_sqlalchemy import SQLAlchemy
   db = SQLAlchemy()
   class User(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       username = db.Column(db.String(150), unique=True, nullable=False)
       email = db.Column(db.String(200))
       password_hash = db.Column(db.String(128))
   # app.py (init)
   app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
   db.init_app(app)
   ```

   Или в Node (Mongoose):

   ```js
   const mongoose = require('mongoose');
   mongoose.connect(process.env.MONGO_URI);
   const UserSchema = new mongoose.Schema({
       username: { type: String, unique: true },
       email: String,
       passwordHash: String
   });
   const User = mongoose.model('User', UserSchema);
   ```

   *Инструменты:* ORMs и драйверы: Mongoose (Node, MongoDB), Sequelize/Prisma/TypeORM (Node, SQL), SQLAlchemy, Django ORM (Python). *Частота:* Очень высокая для приложений, работающих с БД. В списке топ-зависимостей npm есть **mongoose** (\~18 тыс. упоминаний), что свидетельствует о массе Node-приложений с MongoDB. Аналогично, Django ORM используется во всех проектах на Django (тысячи репозиториев), SQLAlchemy – в большинстве Flask/FastAPI проектов, etc. Этот компонент универсален для любого хранилища данных.

4. **Валидация ввода и обработка ошибок** – общие фрагменты для проверки корректности пользовательских данных и единообразной обработки исключений. *Описание:* Веб-приложения получают ввод (формы, JSON запросы) и должны проверить его перед использованием. Повторно используется код валидации полей (например, не пустое поле, соответствие email формату) и отклонения некорректных запросов с сообщениями об ошибках. Кроме того, компоненты глобальной обработки ошибок (например, middleware в Express или обработчик `@app.errorhandler` во Flask) повторяются практически во всех проектах. *Типичный код:*

   * Использование библиотек валидации: **Joi** или **Yup** в Node, **pydantic** в FastAPI или WTForms в Flask.
   * Глобальный try-catch: в Express часто добавляют *error-handling middleware* для централизованного логирования и ответа.

   ```js
   // Express validation with Joi:
   const schema = Joi.object({ username: Joi.string().required(), age: Joi.number().min(0) });
   app.post('/signup', (req, res, next) => {
       const { error, value } = schema.validate(req.body);
       if (error) return res.status(400).json({ error: error.details[0].message });
       // ... proceed to create user
   });
   // Global error handler:
   app.use((err, req, res, next) => {
       console.error(err.stack);
       res.status(500).send('Internal Server Error');
   });
   ```

   ```python
   # Pydantic validation in FastAPI automatically via model schemas:
   from pydantic import BaseModel, Field, ValidationError
   class Item(BaseModel):
       name: str = Field(..., min_length=1)
       price: float = Field(..., gt=0)
   @app.post("/items")
   def create_item(item: Item):
       return {"item": item.dict()}
   # Flask manual error handler:
   @app.errorhandler(Exception)
   def handle_error(e):
       app.logger.error(f"Unhandled error: {e}")
       return {"error": "Internal server error"}, 500
   ```

   *Инструменты:* Joi/Express-Validator (Node), pydantic/cerberus/Marshmallow (Python), built-in validation in Django Forms/DRF serializers. *Частота:* Валидация присутствует в большинстве приложений, хотя иногда неявно (например, FastAPI автоматически валидирует с помощью pydantic схем). Типовой код для ошибок также практически всегда есть. Таким образом, данный компонент – один из “скрытых” но вездесущих блоков.

5. **Логирование и мониторинг** – настройка логов приложения, запись событий, иногда интеграция с системами мониторинга. *Описание:* Компонент обеспечивает ведение журналов (logs) – кто и когда обращался к API, какие произошли ошибки, отладочные сообщения. Обычно включает инициализацию логгера (например, Winston в Node или стандартный `logging` в Python) и использование его в ключевых местах. В более продвинутых случаях – отправку логов во внешние системы (ELK, CloudWatch), метрики Prometheus, трассировку запросов (OpenTelemetry). *Типичный код:*

   ```js
   // Node.js (Winston logger setup):
   const winston = require('winston');
   const logger = winston.createLogger({
       level: 'info',
       format: winston.format.json(),
       transports: [
           new winston.transports.Console(),
           new winston.transports.File({ filename: 'app.log' })
       ]
   });
   // Usage:
   app.get('/status', (req, res) => {
       logger.info('Status check'); 
       res.json({ status: 'ok' });
   });
   ```

   ```python
   import logging
   logging.basicConfig(level=logging.INFO, filename='app.log',
                       format='%(asctime)s %(levelname)s:%(message)s')
   logger = logging.getLogger(__name__)
   # Usage
   logger.info("Application started")
   ```

   *Инструменты:* Winston, Bunyan, pino (Node); logging модуль, Loguru (Python); Opentelemetry SDK для трассировки. *Частота:* Практически во всех приложениях есть базовое логирование. Например, **debug** (библиотека для логов) входит в топ-5 зависимостей npm-проектов. В Python `logging` используется по умолчанию (Django настраивает логирование). Компонент не виден пользователям, но очень распространён.

6. **Конфигурация и переменные окружения** – код для загрузки конфигурационных параметров (настройки БД, API-ключи, и пр.) из файлов или переменных среды. *Описание:* Лучшие практики (12-Factor App) диктуют хранить конфиг отдельно от кода. Шаблонный код этого компонента: чтение файла `.env` или `config.yaml` при старте приложения, маппинг в переменные и использование их по коду. *Типичный код:*

   * В Node: использование пакета **dotenv** или конфиг-модуля (nconf, convict). Например:

   ```js
   require('dotenv').config();
   const PORT = process.env.PORT || 3000;
   const dbUrl = process.env.DATABASE_URL;
   // ... use PORT in app.listen, dbUrl in DB connection
   ```

   * В Python: пакет **python-dotenv** для Flask, или настройки Django (берутся из переменных окружения через `os.environ`). Например:

   ```python
   from dotenv import load_dotenv
   import os
   load_dotenv()  # reads .env
   DB_URL = os.getenv("DATABASE_URL")
   DEBUG_MODE = os.getenv("DEBUG", "false").lower() == "true"
   ```

   *Инструменты:* dotenv, nconf (Node); python-dotenv, Dynaconf, Pydantic Settings (Python). *Частота:* Очень высокая. По данным анализа GitHub, распространённый способ внедрения конфигурации – через **Environment variables и dotenv**. В списке npm-зависимостей **dotenv** имел \~12 тыс. упоминаний на GitHub. Практически каждый проект использует что-то подобное, даже если неявно (например, Heroku через переменные окружения).

7. **Интеграция внешних API и HTTP-клиенты** – повторяемые фрагменты для вызова сторонних веб-сервисов из приложения. *Описание:* Многие приложения обращаются к внешним API: платежные шлюзы (Stripe, PayPal), сервисы отправки почты (SendGrid, SMTP), соц. платформы (Facebook, Slack), геокодирование, и пр. Код интеграции обычно включает формирование HTTP-запроса, обработку ответа и ошибок. *Типичный код:*

   * На Node.js часто применяется **axios** или `node-fetch`:

   ```js
   const axios = require('axios');
   async function verifyCaptcha(token) {
       try {
           const res = await axios.post(`https://www.google.com/recaptcha/api/siteverify`, null, {
               params: { secret: SECRET_KEY, response: token }
           });
           return res.data.success;
       } catch (err) {
           console.error("Captcha verification failed", err);
           return false;
       }
   }
   ```

   * В Python – библиотека **requests** (которая является одной из самых скачиваемых и используемых):

   ```python
   import requests
   def send_sms(phone, msg):
       url = "https://api.twilio.com/send_sms"
       payload = {"to": phone, "message": msg}
       headers = {"Authorization": f"Bearer {API_TOKEN}"}
       resp = requests.post(url, json=payload, headers=headers)
       resp.raise_for_status()
       return resp.json()
   ```

   *Инструменты:* axios, node-fetch, superagent (Node); requests, httpx (Python). Также SDK: boto3 (AWS), google-api-python-client, etc. *Частота:* Очень распространено – практически каждое веб-приложение имеет хотя бы интеграцию отправки email или webhook. **Requests** – один из самых популярных Python-пакетов (≈40 млн загрузок/месяц). В Node **axios** тоже крайне популярен. Значительная доля кода (особенно ботов, интеграционных сервисов) состоит именно из вызовов внешних API.

8. **Реал-тайм обновления и коммуникация (чаты, нотификации)** – компоненты для обмена сообщениями в режиме реального времени, обычно через WebSocket. *Описание:* Функциональность мгновенного оповещения – будь то чат между пользователями, трансляция обновлений (например, прогресс задачи) или уведомления – требует поддержания постоянного соединения или долгого опроса. Повторно используются шаблоны: настройка WebSocket-сервера, рассылка сообщений подписчикам, обработка событий соединения/отключения. *Типичный код:*

   * В Node.js стандартом де-факто является **Socket.io**, который облегчает работу с WebSocket:

   ```js
   const httpServer = require('http').createServer(app);
   const io = require('socket.io')(httpServer);
   io.on('connection', (socket) => {
       console.log('a user connected:', socket.id);
       socket.on('chat message', (msg) => {
           io.emit('chat message', msg); // broadcast to all
       });
       socket.on('disconnect', () => {
           console.log('user disconnected');
       });
   });
   httpServer.listen(3000);
   ```

   * В Python для реального времени используют **Django Channels** или библиотеки вроде **websockets**/**Socket.IO-client**:

   ```python
   # Example using websockets library (async)
   import asyncio, websockets
   connected = set()
   async def handler(websocket, path):
       connected.add(websocket)
       try:
           async for message in websocket:
               # broadcast to all connected
               await asyncio.wait([ws.send(message) for ws in connected if ws != websocket])
       finally:
           connected.remove(websocket)
   start_server = websockets.serve(handler, "0.0.0.0", 8765)
   asyncio.get_event_loop().run_until_complete(start_server)
   asyncio.get_event_loop().run_forever()
   ```

   *Инструменты:* Socket.io (Node и клиент JS), ws (низкоуровневый WebSocket), Django Channels, Flask-SocketIO (на базе eventlet). *Частота:* Чат-функции часто приводятся в примерах и учебных приложениях, что стимулирует их копирование. По данным GitHub, Socket.io используется в **\~15 тысячах** проектов. Не каждое приложение имеет realtime, но многие внутренние дашборды требуют live-обновлений, а мессенджеры/чаты – популярный тип приложений (часто реализуемый по шаблону). Таким образом, компонент достаточно распространён.

9. **Очереди задач и фоновые задания** – шаблоны для выполнения длительных задач асинхронно, вне основного запроса. *Описание:* Если действие занимает много времени (обработка изображения, отправка email, генерация отчёта), его выносят в фоновую задачу, чтобы не блокировать пользовательский запрос. Для этого служат очереди сообщений (RabbitMQ, Redis) и воркеры. Компонент включает код постановки задачи в очередь и код обработчика задачи. *Типичный код:*

   * В Node.js распространены библиотеки **Bull** (Redis queue) или использование облачных очередей. Пример постановки задачи:

   ```js
   const Queue = require('bull');
   const emailQueue = new Queue('emailQueue', { redis: { host: '127.0.0.1', port: 6379 }});
   // Producer: adding a job
   app.post('/send-email', (req, res) => {
       const { to, subject, body } = req.body;
       emailQueue.add({ to, subject, body });
       res.json({ status: 'queued' });
   });
   // Consumer (worker.js):
   emailQueue.process(async job => {
       const { to, subject, body } = job.data;
       await sendEmail(to, subject, body);  // imaginary sendEmail function
       return Promise.resolve();
   });
   ```

   * В Python очень популярна связка **Celery + Redis/RabbitMQ**:

   ```python
   # tasks.py
   from celery import Celery
   app = Celery(broker='redis://localhost:6379/0')
   @app.task
   def send_email(to, subject, body):
       # ... код отправки email
       print(f"Email sent to {to}")
   # Вызов задачи из основного приложения:
   send_email.delay(user.email, "Welcome!", "Thank you for joining...")
   ```

   *Инструменты:* Bull/BullMQ, Agenda (Node); Celery, RQ, Huey (Python); Sidekiq аналоги (в других ЯП). *Частота:* В крупных приложениях – повсеместно. Даже в простых – часто есть отложенная отправка почты при регистрации, что уже требует фоновой задачи. Код постановки задач часто шаблонен. Например, Celery используется тысячами проектов (свыше 10 млн загрузок/год), Bull и похожие – стандарт для Node-бэкендов. Для микросервисов на Node также часто используют Kafka/RabbitMQ прямо, но это выходит за рамки нашего анализа.

10. **Конвейеры CI/CD и скрипты развертывания** – конфигурации и скрипты для автоматической сборки, тестирования и деплоя приложения. *Описание:* Хотя формально это не часть прикладного кода, **pipeline**-скрипты (файлы YAML, Dockerfile, Shell-скрипты) живут вместе с кодом и имеют повторяющиеся шаблоны. Например, `.github/workflows/ci.yml` с установкой зависимостей и запуском тестов – в разных проектах отличается лишь именами. Dockerfile для Node или Python веб-приложения тоже следует типовым рецептам (берутся стандартные базовые образы, копируется `package.json`/`requirements.txt`, и т.д.). *Типичный код:*

* **GitHub Actions Workflow (Node.js пример):**

```yaml
name: CI
on: [push, pull_request]
jobs:
  build_test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm install
      - run: npm run build --if-present
      - run: npm test
```

* **Dockerfile (Python Flask пример):**

```Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
ENV FLASK_APP=app.py
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0"]
```

*Инструменты:* Сервисы CI (GitHub Actions, GitLab CI, Travis CI), контейнеры Docker, инструмент Terraform (для инфраструктуры). *Частота:* По оценкам исследований, **не менее 70% популярных проектов используют CI**, и эта доля растёт. Файл конфигурации CI/CD стал нормой даже для маленьких библиотек. Docker также очень распространён для деплоя: шаблоны Dockerfile повторяются из проекта в проект (например, почти все Node Dockerfile основаны на официальном образе Node и выполняют npm install + npm start). В целом, данный компонент – один из самых переиспользуемых (через копирование или генераторы) вне самого кода приложения.

11. **Инфраструктура как код (IaC)** – скрипты описания инфраструктуры (например, Terraform, CloudFormation) внутри репозитория. *Описание:* DevOps-практики приводят к тому, что конфигурация облачных ресурсов хранится в виде кода рядом с приложением. Это могут быть Terraform-файлы, Kubernetes-манифесты, Ansible playbooks. Они также имеют шаблонный характер: например, Terraform-модуль для деплоя веб-приложения на AWS (создание ECS-кластера, базы данных и пр.) будет схож от проекта к проекту. *Типичный код:*

* **Terraform (фрагмент):**

```hcl
provider "aws" {
  region = var.region
}
resource "aws_ecs_cluster" "app_cluster" {
  name = "${var.project_name}-cluster"
}
resource "aws_ecs_task_definition" "app_task" {
  family = "${var.project_name}-task"
  container_definitions = jsonencode([
    {
      name = "app",
      image = var.docker_image,
      memory = 512,
      portMappings = [{ containerPort = 5000, hostPort = 5000 }]
    }
  ])
}
# ... (service, load balancer, etc.)
```

*Инструменты:* Terraform, Serverless Framework (описание functions), Kubernetes YAML, Pulumi (на TS/Python), AWS CloudFormation (JSON/YAML). *Частота:* На open-source арене несколько ниже, чем CI, но быстро растёт. По опросам \~90% компаний хотя бы частично внедрили IaC-практики. В публичных репозиториях встречается реже (не все выкладывают инфраструктуру), но **шаблоны IaC** определённо широко переиспользуются (есть популярные boilerplate для AWS, Kubernetes charts шаблонные и т.д.). Мы включаем этот компонент как перспективный, хотя и менее заметный в анализе исходников.

12. **Тесты и образцы (fixtures)** – шаблоны кода для модульных и интеграционных тестов. *Описание:* Автоматические тесты стали неотъемлемой частью разработки. Тестовые файлы часто повторяют одни и те же конструкции: инициализация тестового приложения, набор типовых проверок (например, “должен вернуть 200 OK при GET /health”). Этот компонент включает базовые тесты (health-check, авторизация) и заготовки данных (fixtures). *Типичный код:*

* **JavaScript (Jest, Mocha):**

```js
const request = require('supertest');
const app = require('../app');
describe('GET /status', () => {
  it('should return 200 and status ok', async () => {
    const res = await request(app).get('/status');
    expect(res.statusCode).toBe(200);
    expect(res.body).toHaveProperty('status', 'ok');
  });
});
```

* **Python (PyTest):**

```python
import pytest
from app import app as flask_app
@pytest.fixture
def client():
    with flask_app.test_client() as client:
        yield client
def test_status_endpoint(client):
    resp = client.get('/status')
    assert resp.status_code == 200
    data = resp.get_json()
    assert data.get('status') == 'ok'
```

*Инструменты:* Jest, Mocha, SuperTest (Node); PyTest, unittest (Python); Selenium/Playwright для энд-ту-энд. *Частота:* В активных проектах – высокая, хотя многие репозитории без тестов. Но генераторы (например, Cookiecutter Django) создают проекты **со 100% покрытием тестами на старте**, предлагая шаблон тестового набора. Общие тесты (на доступность главной страницы, на создание объекта) очень похожи во многих проектах, т.е. этот код тоже во многом шаблонный.

13. **Микросервисные шаблоны (API Gateway, сервисные вызовы)** – повторяемые паттерны в распределённых системах. *Описание:* При разбиении приложения на микросервисы возникает необходимость в общих компонентах: **API Gateway** (единая точка входа, маршрутизация к сервисам, часто реализуется на Express/NestJS или специальном фреймворке), **сервис-дискавери**, межсервисная авторизация, **health-check** endpoint для оркестратора. Эти элементы часто берутся из типовых решений. *Типичный код:*

* **API Gateway** на Express – фактически прокси:

```js
const { createProxyMiddleware } = require('http-proxy-middleware');
// Проксируем /api/auth/* на сервис аутентификации
app.use('/api/auth', createProxyMiddleware({ target: AUTH_SERVICE_URL, changeOrigin: true }));
// Проксируем /api/data/* на сервис данных
app.use('/api/data', createProxyMiddleware({ target: DATA_SERVICE_URL, changeOrigin: true }));
```

* **Health-check** (для Kubernetes/LB):

```js
app.get('/health', (req, res) => res.status(200).send('OK'));
```

*Инструменты:* http-proxy-middleware, Zuul/Kong (внешние), service registry (Consul, etcd), gRPC clients. *Частота:* С ростом микросервисов – повсеместно: **74% организаций используют микросервисную архитектуру**, а значит, у них есть общий код для gateway и коммуникации. В OSS многие используют готовые API Gateway (Kong, Traefik), но и самописные встречаются. Health-check endpoint присутствует практически во всех сервисах. Таким образом, шаблоны микросервисов также очень распространены, хотя и специфичны для продвинутых проектов.

14. **Отправка email и уведомлений** – компонент для рассылки писем, SMS, push-уведомлений пользователям. *Описание:* Типичная задача – отправить письмо подтверждения при регистрации, или уведомление о событии. Код включает шаблоны писем, вызов SMTP или внешнего API, обработку ошибок. *Типичный код:*

* Использование сервисов: Node – пакет **nodemailer** для SMTP:

```js
const nodemailer = require('nodemailer');
const transporter = nodemailer.createTransport({ service: 'Gmail', auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS } });
async function sendWelcomeEmail(to, userName) {
    await transporter.sendMail({
        from: '"MyApp" <noreply@myapp.com>', to,
        subject: 'Welcome to MyApp',
        text: `Hello ${userName}, thanks for joining!`
    });
}
```

* Python – например, **smtplib** или интеграция с API (SendGrid):

```python
import smtplib
from email.message import EmailMessage
def send_email_smtp(to, subject, body):
    msg = EmailMessage()
    msg['Subject'] = subject
    msg['From'] = 'noreply@myapp.com'
    msg['To'] = to
    msg.set_content(body)
    with smtplib.SMTP('smtp.gmail.com', 587) as smtp:
        smtp.starttls()
        smtp.login(SMTP_USER, SMTP_PASS)
        smtp.send_message(msg)
```

*Инструменты:* nodemailer, AWS SES SDK, Twilio API (SMS), Push libraries; Django имеет встроенный email backend. *Частота:* Почти все приложения, где есть пользователи, отправляют email (подтверждения, восстановление пароля). Код отличается деталями, но общая структура одна. Поэтому многие разработчики копируют готовые примеры или используют стандартные функции (например, `django.core.mail.send_mail`). Данный компонент – часто переиспользуемый, хотя занимает не много строк.

15. **Управление ролями и доступом (RBAC)** – компонент расширяющий аутентификацию: разграничение прав пользователей, проверка ролей (admin, user и т.д.). *Описание:* Во многих системах есть различия в правах – например, администраторы могут делать больше. Шаблонный код: хранить роль пользователя, при доступе к ресурсу проверять, имеет ли он право. *Типичный код:*

```js
// Middleware для проверки роли
function requireRole(role) {
    return (req, res, next) => {
        const user = req.user; // предположим, мидлварь аутентификации уже положила пользователя
        if (!user || user.role !== role) {
            return res.status(403).send('Forbidden');
        }
        next();
    };
}
// Применение:
app.delete('/admin/users/:id', requireRole('admin'), adminController.deleteUser);
```

Или на уровне маршрутов (например, в Django через декораторы или проверки в представлении). *Инструменты:* Casbin, Oso (библиотеки авторизации), встроенные фреймворковые возможности (например, `@login_required` и ручная проверка `request.user.is_staff` в Django). *Частота:* В системах с двумя и более ролями – высокая. Код проверки обычно пишется вручную (2-3 строчки), но встречается повсеместно. Многие проекты просто реализуют это самостоятельно, копируя из предыдущих. Таким образом, хотя компонент небольшой, он типовой.

16. **Кеширование данных** – шаблоны кода для кеширования часто используемых данных (в памяти или распределённо). *Описание:* Чтобы повысить производительность, часто сохраняют результат тяжёлых запросов в кеш (Redis, memory) и потом быстро отдают из кеша. Код компонента включает проверку наличия данных в кеше, иначе – получение с источника и сохранение в кеш. *Типичный код:*

```js
const redis = require('redis');
const client = redis.createClient();
async function getUserProfile(userId) {
    const cacheKey = `user:${userId}`;
    const cached = await client.get(cacheKey);
    if (cached) {
        return JSON.parse(cached);
    }
    const user = await db.getUserById(userId);
    client.set(cacheKey, JSON.stringify(user), { EX: 3600 }); // кеш на 1 час
    return user;
}
```

*Инструменты:* Redis (через `redis` для Node, `redis-py` для Python), мемкеш, кэширование запросов (HTTP caching headers). *Частота:* Во многих продуктивных системах – да, но в простых может отсутствовать. Тем не менее, шаблон *кеширование с Redis* очень популярен: достаточное число проектов включает подобный код (Redis фигурирует в dependency многих Node/Python проектов).

17. **Работа с файлами и загрузка файлов (Upload)** – компонент для приёма файлов от пользователей (аватары, документы) и хранения их на сервере или облаке. *Описание:* Повторяемый код: HTML-форма или API для загрузки, сервер принимает файл, сохраняет на диске или в облачное хранилище (S3, Google Cloud Storage), возвращает ссылку. Также часто есть типовой код по выдаче статических файлов или их обработке. *Типичный код:*

* Express с **multer**:

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });
app.post('/upload', upload.single('file'), (req, res) => {
    // файл сохранён как req.file.path
    res.json({ filename: req.file.filename });
});
```

* Python Flask:

```python
from werkzeug.utils import secure_filename
@app.route('/upload', methods=['POST'])
def upload():
    file = request.files['file']
    filename = secure_filename(file.filename)
    file.save(os.path.join('uploads', filename))
    return {"filename": filename}
```

*Инструменты:* multer (Node), Flask-Uploads или просто Werkzeug, Django Storages. *Частота:* Встречается регулярно, хотя не в каждом приложении. Загрузка аватара, экспорт файла – популярные задачи. Часто разработчики берут пример кода из документации (как выше).

18. **Шаблоны HTML/рендеринг страниц** – (в Node реже, чаще front-end, но в Python/Django крайне типично) шаблонный код генерации HTML на сервере. *Описание:* Этот компонент особенно характерен для традиционных веб-приложений: использование шаблонизаторов (Jinja2, EJS, Handlebars) для формирования страниц с данными. Код компонента – маршрут, который собирает данные и возвращает отрендеренный шаблон. *Типичный код:*

* Django view:

```python
from django.shortcuts import render, get_object_or_404
def article_detail(request, article_id):
    article = get_object_or_404(Article, pk=article_id)
    return render(request, 'article_detail.html', {'article': article})
```

* Express с шаблонизатором EJS:

```js
app.set('view engine', 'ejs');
app.get('/articles/:id', async (req, res) => {
    const article = await Article.findById(req.params.id);
    if (!article) return res.status(404).send('Not found');
    res.render('article_detail', { article });
});
```

*Инструменты:* Jinja2/Django template language, EJS/Pug/Handlebars (Node). *Частота:* Несмотря на рост SPA, серверный рендеринг все еще в ходу, особенно в Python экосистеме (Django admin, шаблоны писем). Потому подобные views и шаблоны – тоже повторяемый элемент.

19. **Миграции базы данных** – скрипты/классы для изменения схемы базы данных (создание таблиц, изменение полей), которые часто генерируются. *Описание:* ORM обычно включает систему миграций (Django migrations, Alembic для SQLAlchemy, Sequelize migrations). Хотя пишется не вручную, миграции – часть репозитория, и их структура типична (есть “скелет” файла миграции с функциями *upgrade/downgrade* или списком операций). Например, миграция Alembic:

```python
# revision identifiers, used by Alembic.
revision = '34b1b2cdefgh'
down_revision = '12a3bc4defg'
def upgrade():
    op.create_table('user',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('username', sa.String(50), nullable=False)
    )
def downgrade():
    op.drop_table('user')
```

*Инструменты:* Django migrations (автогенерируемые .py файлы), Alembic (Python), Sequelize CLI (Node), Flyway etc. *Частота:* В проектах с БД – высокая. Шаблон миграции повторяется благодаря инструментам. Этот компонент скорее автоматически генерируется, но мы его отмечаем, так как он присутствует в репозиториях и является повторяющимся паттерном кода.

20. **Документация API (OpenAPI/Swagger)** – код для автогенерации или предоставления спецификации API. *Описание:* Многие REST API включают интерактивную документацию (Swagger UI) или по крайней мере JSON-файл OpenAPI. В Node часто подключают **swagger-ui-express** с готовым YAML-файлом, а в Python FastAPI генерирует `/docs` автоматически. *Типичный код:*

* Node + swagger-ui:

```js
const swaggerUi = require('swagger-ui-express');
const swaggerDoc = require('./openapi.json');
app.use('/docs', swaggerUi.serve, swaggerUi.setup(swaggerDoc));
```

*Инструменты:* Swagger UI, Redoc, FastAPI docs (Starlette), Sphinx/MkDocs for general docs. *Частота:* Растёт с популярностью API-first подхода. Повторяется если используется один и тот же генератор. Например, **OpenAPI спецификация** присутствует во множестве API-репозиториев. Этот компонент – важный с точки зрения полного цикла разработки, хотя может отсутствовать в простых проектах.

*(Выше перечислены наиболее значимые общие компоненты; список можно продолжить компонентами: международзация (i18n) и локализация; обработка платежей (Stripe SDK); загрузка конфигурации фронтенда (Webpack configs) – однако их влияние менее универсально.)*