# Dolina Znaniy

Монорепозиторий платформы для репетиторов и учеников: **FastAPI** + **PostgreSQL** + **Redis** на backend и **Next.js 14 App Router** на frontend. Инфраструктура упакована в **Docker Compose** с отдельным профилем для продакшена и **Nginx** reverse proxy.

## Быстрый старт (локально)

1. Скопируйте переменные окружения:

   ```bash
   cp backend/.env.example backend/.env
   cp frontend/.env.local.example frontend/.env.local
   ```

   При необходимости поправьте пароли Postgres и секрет JWT в `backend/.env`.

2. Поднимите сервисы и примените миграции:

   ```bash
   make up
   make migrate
   ```

   Команды `Makefile`:

   | Команда | Что делает |
   | --- | --- |
   | `make up` | `docker compose up -d --build` (режим разработки) |
   | `make migrate` | `alembic upgrade head` внутри контейнера `backend` |
   | `make logs` | хвост логов Compose |
   | `make down` | остановка dev-стека |
   | `make prod-up` | прод профиль через `docker-compose.prod.yml --profile prod` |

3. Откройте браузер: `http://localhost:3000` (frontend). API доступно напрямую по `http://localhost:8000`; браузер ходит на backend через префикс `/api/v1` (прокси Next.js → переменная `INTERNAL_API_URL`).

4. Создайте учителя, создайте комнату → получите код → зарегистрируйте ученика и введите код на странице «Мои занятия».

## Продакшен (VPS + Docker)

```bash
export PUBLIC_BASE_URL=https://your-domain.tld
export CORS_ORIGINS=https://your-domain.tld
export NEXT_PUBLIC_API_BASE=/api/v1
docker compose -f docker-compose.prod.yml --profile prod up -d --build
docker compose -f docker-compose.prod.yml exec backend alembic upgrade head
```

- Nginx проксирует `/api/*` в FastAPI, `/uploads/` отдаёт из общего тома (`uploads_prod`), UI из standalone Next.js.
- Блок HTTPS в `deploy/nginx/default.conf` закомментирован: добавьте сертификаты **Let's Encrypt** и при необходимости смонтируйте `/etc/letsencrypt` (комментарии в конфиге).

## Особенности MVP

- Уведомления ученику — полинг каждые 5 секунд (`GET /users/notifications`).
- Загрузка файлов: dev — локальная директория `backend/uploads`, prod — см. переменную `UPLOAD_BACKEND` (`local`/`s3`) в `backend/.env.example`.

## Swagger и тесты

- Интерактивная документация: `http://localhost:8000/docs`.
- Автотесты backend см. `.github/workflows/ci.yml` и `backend/tests/` (PostgreSQL нужен локально или в Compose).

### Миграции

Alembic: директория `backend/migrations`, стартовая ревизия `0001_initial`.

## Структура каталогов

```
backend/
frontend/
deploy/nginx/
docker-compose.yml / docker-compose.prod.yml
Makefile
```

## Переезд Vercel → Amvera / VPS

Поднимите backend в Docker где угодно, затем установите переменную `NEXT_PUBLIC_API_BASE` (абсолютный URL вашего HTTPS API или `/api/v1` при том же домене, что у Nginx) и поправьте `INTERNAL_API_URL` у Next.js билдера так, чтобы прокси в dev/prod билде указывал на актуальный бекенд.

Не комитьте секретные `.env`-файлы (они исключены в `.gitignore`).
