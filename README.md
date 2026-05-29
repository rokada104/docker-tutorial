# 🚀 Полный практический гайд: Docker, PostgreSQL и Go-приложение с HTTP-сервисом

Этот гайд поможет вам с нуля собрать полностью рабочее окружение для практической разработки:
- Поймёте базовые команды Docker.
- Поднимете PostgreSQL в контейнере.
- Создадите приложение на Go, которое пишет данные в базу через HTTP-запрос.
- Настроите volume для сохранения данных после удаления контейнеров.
- Сделаете сборку приложения в продакшен-стиле: multi-stage build, запуск не от root-пользователя.

💻 Для практики можете использовать онлайн-песочницу [iximiuz labs playground](https://labs.iximiuz.com/playgrounds/docker), если Docker локально не установлен.

📝 Рекомендуемый клиент для локальной работы с базой: [DBeaver](https://dbeaver.io/download/)

📦 Официальный образ Postgres с документацией:  
[https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres)

Автор урока: **Олег Козырев**  
- [Telegram канал](https://t.me/olezhek28go)
- [YouTube канал](https://www.youtube.com/@olezhek28go)

---

## 🎉 Понравился урок? Продолжи изучение!

🔗 Следующий шаг — разберись с Docker Compose!

👉 [Урок по Docker Compose: автоматизация запуска приложения и базы](https://github.com/olezhek28/docker-compose-tutorial)

Там мы:
- Поднимаем Postgres и приложение в одной команде.
- Настраиваем `.env` переменные.
- Подключаем миграции через Goose.
- И делаем проект готовым для продакшена!

---

## 🖼️ Схема работы Docker

[![Диаграмма Docker](https://kinsta.com/wp-content/uploads/2022/10/Docker-Diagram.png)](https://kinsta.com/blog/what-is-docker/)

> Источник изображения: [kinsta.com](https://kinsta.com/blog/what-is-docker/)

## 📦 Проверяем, что Docker работает

Запускаем тестовый контейнер:

```bash
docker run hello-world
```

Docker скачает тестовый образ и выведет сообщение, если всё настроено правильно.

Посмотреть список запущенных контейнеров:

```bash
docker ps
```

Посмотреть список локальных образов:

```bash
docker images
```

---

## 🌐 Поднимаем Nginx в контейнере

```bash
docker run -d -p 80:80 --name mynginx nginx
```

Теперь можно открыть в браузере: [http://localhost](http://localhost)  
➡️ Nginx успешно работает!

---

## 🗄️ Поднимаем PostgreSQL в контейнере

```bash
docker run -d -p 5432:5432 --name mypostgres -e POSTGRES_USER=demo -e POSTGRES_PASSWORD=demo postgres:15
```

Теперь база доступна на порту 5432, и пользователь demo готов к работе.

---

## 🐚 Заходим внутрь контейнера с базой

Если в контейнере есть bash:

```bash
docker exec -it mypostgres bash
```

Если bash нет, используем sh:

```bash
docker exec -it mypostgres sh
```

---

## 🧩 Подключаемся к Postgres внутри контейнера

```bash
psql -U demo -d postgres
```

- `-U demo` — имя пользователя
- `-d postgres` — база данных по умолчанию

Полезные команды внутри psql:
- Посмотреть все таблицы: `\dt`
- Посмотреть все объекты: `\d`
- Посмотреть все базы данных: `\l`

Создаём таблицу пользователей:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username text,
  email text,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Добавляем данные вручную:

```sql
INSERT INTO users (username, email) VALUES ('alice', 'alice@example.com');
```

Проверяем данные:

```sql
SELECT * FROM users;
```

---

## 🧹 Убираем за собой

Останавливаем контейнер:

```bash
docker stop mypostgres
```

Удаляем контейнер и образ:

```bash
docker rmi -f mypostgres 1b92e7a80c02
```

Чистим все неиспользуемые ресурсы Docker:

```bash
docker system prune
```

> ⚠️ Внимание: команда очистит все остановленные контейнеры, образы и кэш.

---

## 💻 Переходим к локальной разработке на своей машине

- Устанавливаем Docker Desktop: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- Устанавливаем DBeaver для подключения к базе: [https://dbeaver.io/download/](https://dbeaver.io/download/)

---

## 🛠️ Создаём Go-приложение с HTTP endpoint для записи в базу

Приложение принимает POST-запрос с username и email и сохраняет их в таблицу users.

Добавляем файл `.dockerignore`, чтобы не попадали лишние файлы в контейнер при сборке.

Создаём multi-stage Dockerfile:
- Этап сборки — heavy image с Go SDK.
- Этап финальный — лёгкий продакшен образ на Alpine.
- Собираем статически слинкованный бинарник.

Собираем приложение под нужную архитектуру:

```bash
docker build --platform linux/amd64 -t my-go-server:v1.0.0 .
```

Для полной пересборки без кэша:

```bash
docker build --no-cache --platform linux/amd64 -t my-go-server:v1.0.0 .
```

Проверяем архитектуру образа:

```bash
docker image inspect my-go-server:v1.0.0 --format='{{.Architecture}}/{{.Os}}'
```

Запускаем приложение:

```bash
docker run -p 8080:8080 my-go-server:v1.0.0
```

---

## 🔗 Создаём сеть для общения контейнеров

Создаём сеть:

```bash
docker network create app-network
```

Запускаем Postgres в сети:

```bash
docker run -d --name mypostgres --network app-network -p 5432:5432 -e POSTGRES_USER=demo -e POSTGRES_PASSWORD=demo postgres:15
```

Запускаем приложение в той же сети:

```bash
docker run -p 8080:8080 --network app-network my-go-server:v1.0.0
```

---

## 🧩 Подключаемся к базе через DBeaver

Создаём новое подключение:
- Хост: `localhost`
- Порт: `5432`
- База данных: `postgres`
- Пользователь: `demo`
- Пароль: `demo`

Создаём таблицу в DBeaver:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username text,
  email text,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🌐 Тестируем приложение через curl

Отправляем POST-запрос для создания пользователя:

```bash
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "email": "alice@example.com"}'
```

Проверяем в базе: пользователь появился 🎉

---

## 💡 Проверяем сохранность данных

Останавливаем контейнер с базой:

```bash
docker stop mypostgres
```

Стартуем снова:

```bash
docker start mypostgres
```

✅ Данные остаются!

Удаляем контейнер:

```bash
docker rm mypostgres
```

Запускаем заново без volume — ❌ данные пропали.

---

## 💾 Добавляем volume для сохранения данных

Запускаем контейнер с volume:

```bash
docker run -d --name mypostgres --network app-network -p 5432:5432 \
  -e POSTGRES_USER=demo \
  -e POSTGRES_PASSWORD=demo \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
```

Создаём таблицу, добавляем пользователя через curl.

Останавливаем контейнер:

```bash
docker stop mypostgres
```

Запускаем снова:

```bash
docker start mypostgres
```

✅ Данные сохранены!

---

## 🧑‍💻 Дорабатываем Dockerfile приложения для продакшен-уровня безопасности

Добавляем запуск приложения не от root-пользователя:

```Dockerfile
# Создаём непривилегированного пользователя и группу
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Меняем владельца файлов на созданного пользователя
RUN chown -R appuser:appgroup /app

# Переключаемся на непривилегированного пользователя
USER appuser
```

Теперь контейнер безопасно работает без root-доступа.

---

## 🎉 Поздравляю! Вы собрали полноценное приложение с Docker, Postgres и Go.

Если вам понравился урок — заглядывайте:
- [Telegram канал](https://t.me/olezhek28go) — полезные заметки, лайф стайл и обсуждение волнующих вопросов в мире IT.
- [YouTube канал](https://www.youtube.com/@olezhek28go) — технические и софтовые видео про разработку.

## 🧩 Как работает концепция слоёв в Docker и зачем нужен multi-stage build

Когда мы собираем образ Docker, он строится как "слоёный пирог".  
Каждая инструкция в Dockerfile — это новый слой. Например:
- `FROM` — базовый слой образа (например, Alpine или Go SDK).
- `COPY`, `RUN` — добавляют новые слои с изменениями.
- `CMD`, `EXPOSE` — тоже фиксируются как слои.

Docker кэширует слои: если что-то в слое не поменялось, он просто использует уже готовый слой, чтобы ускорить сборку.

### 📦 Проблема обычной сборки

Если собирать Go-приложение без multi-stage build:
- Внутри финального образа окажется весь SDK Go, исходные файлы и временные артефакты.
- Это раздувает размер образа и может нести лишние риски безопасности (внутри много лишнего).

### 🚀 Решение — Multi-Stage Build

Multi-stage build позволяет:
- В одной стадии использовать тяжёлый образ для сборки (например, `golang:1.23.1`).
- А во второй стадии взять только результат сборки — бинарный файл — и положить его в минимальный образ (например, `alpine:3.21.3`).

Такой подход даёт сразу несколько плюсов:
✅ Минимальный размер финального образа.  
✅ Нет лишних инструментов внутри контейнера.  
✅ Быстрая и надёжная сборка.  
✅ Повышенная безопасность (меньше attack surface).

Автор: **Олег Козырев**
