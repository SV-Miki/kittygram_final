# Kittygram

Kittygram — full-stack приложение для публикации информации о котах, их фотографий, цвета, года рождения и достижений.

Backend и frontend приложения были предоставлены как основа учебного проекта. В рамках работы были реализованы контейнеризация проекта, настройка PostgreSQL и Nginx, Docker Compose, CI/CD в GitHub Actions, публикация Docker-образов и production deployment.

## Возможности

- регистрация и авторизация пользователей
- добавление и редактирование информации о котах
- загрузка изображений котов
- указание цвета и года рождения
- добавление достижений
- работа с API через Django REST Framework
- хранение данных в PostgreSQL
- раздача frontend, static и media через Nginx
- запуск приложения в Docker-контейнерах
- автоматическая проверка backend и frontend в GitHub Actions
- сборка и публикация Docker-образов в Docker Hub
- ручной production deployment через GitHub Actions
- выполнение миграций и `collectstatic` при deployment
- Telegram-уведомление после успешного deployment

## Технологии

### Backend

- Python 3.12
- Django 5.1.1
- Django REST Framework 3.15.2
- Djoser 2.3.1
- PostgreSQL 13
- Gunicorn 23.0.0
- Pillow 11.0.0

### Frontend

- React
- Node.js 18

### Infrastructure

- Docker
- Docker Compose
- Nginx
- GitHub Actions
- Docker Hub
- Ruff

## Структура проекта

```text
kittygram/
├── .github/
│   └── workflows/
│       └── main.yml                    # CI/CD workflow
├── backend/
│   ├── cats/                           # Модели, serializers и API Kittygram
│   │   ├── migrations/
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── tests.py
│   │   └── views.py
│   ├── kittygram_backend/              # Конфигурация Django-проекта
│   │   ├── settings.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   ├── .dockerignore
│   ├── Dockerfile                      # Docker-образ backend
│   ├── manage.py
│   └── requirements.txt
├── frontend/
│   ├── public/
│   ├── src/
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── package.json
│   └── package-lock.json
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf                      # Gateway и reverse proxy
├── .env.example                        # Пример переменных окружения
├── .gitignore
├── docker-compose.yml                  # Локальный запуск
├── docker-compose.production.yml       # Production Compose
├── LICENSE
├── pyproject.toml                      # Конфигурация Ruff
└── README.md
```

## Архитектура

Проект запускается через Docker Compose и состоит из четырёх сервисов:

- `db` — PostgreSQL 13
- `backend` — Django API, запущенный через Gunicorn
- `frontend` — сборка React-приложения
- `gateway` — Nginx, который раздаёт frontend, static и media и проксирует запросы к backend

Используются три Docker volume:

- `pg_data` — данные PostgreSQL
- `static` — статические файлы Django и frontend
- `media` — загруженные пользователями изображения

Основные маршруты Nginx:

```text
/                   → frontend
/api/               → backend
/admin/             → Django Admin
/static_backend/    → Django static
/media/             → media-файлы
```

## Установка и запуск

Клонируйте репозиторий:

```bash
git clone git@github.com:SV-Miki/kittygram.git
cd kittygram
```

Создайте `.env` на основе примера:

```bash
cp .env.example .env
```

Заполните необходимые переменные окружения.

Запустите проект:

```bash
docker compose up --build -d
```

Примените миграции:

```bash
docker compose exec backend python manage.py migrate
```

Соберите статические файлы Django:

```bash
docker compose exec backend python manage.py collectstatic --noinput
```

После запуска приложение доступно по адресу:

```text
http://localhost:9000
```

Остановить контейнеры:

```bash
docker compose down
```

## Переменные окружения

Пример конфигурации находится в `.env.example`.

```env
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_HOST=db
DB_PORT=5432

SECRET_KEY=your-secret-key
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=

DOCKER_USERNAME=your-dockerhub-username
```

`SECRET_KEY`, пароль PostgreSQL и production-настройки не должны храниться в репозитории.

## API и авторизация

API доступно по префиксу:

```text
/api/
```

В проекте используется Token Authentication Django REST Framework.

Основные ресурсы:

```text
/api/cats/
/api/achievements/
/api/users/
/api/auth/token/
```

Для работы с защищёнными эндпоинтами требуется авторизация.

## CI/CD

GitHub Actions workflow находится в:

```text
.github/workflows/main.yml
```

При push в ветку `main` выполняются:

- установка backend-зависимостей
- проверка backend с Ruff
- запуск backend tests
- установка frontend-зависимостей
- запуск frontend tests
- сборка Docker-образов backend, frontend и gateway
- публикация Docker-образов в Docker Hub

Публикуются образы:

```text
<dockerhub_username>/kittygram_backend:latest
<dockerhub_username>/kittygram_frontend:latest
<dockerhub_username>/kittygram_gateway:latest
```

## Production deployment

Production Compose находится в:

```text
docker-compose.production.yml
```

Deployment запускается вручную через `workflow_dispatch`.

При запуске workflow:

- `docker-compose.production.yml` копируется на удалённый сервер
- `.env` формируется из GitHub Secrets
- актуальные Docker-образы загружаются из Docker Hub
- контейнеры перезапускаются
- выполняются миграции
- выполняется `collectstatic`
- после успешного deployment отправляется Telegram-уведомление

Для production используются GitHub Secrets:

```text
DOCKER_USERNAME
DOCKER_PASSWORD

HOST
USER
SSH_KEY

POSTGRES_DB
POSTGRES_USER
POSTGRES_PASSWORD

SECRET_KEY
ALLOWED_HOSTS
CSRF_TRUSTED_ORIGINS

TELEGRAM_CHAT_ID
TELEGRAM_TOKEN
```

## Проверка backend

Проверка конфигурации Django:

```bash
python backend/manage.py check
```

Проверка кода Ruff:

```bash
python -m ruff check backend/
```

## Проверка frontend

Установите зависимости:

```bash
cd frontend
npm ci
```

Запустите тесты:

```bash
npm test -- --watchAll=false
```

## Автор

Владислав Шилов
