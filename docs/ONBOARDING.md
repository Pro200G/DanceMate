# ONBOARDING.md

# Онбординг разработчика

**Проект:** DanceMate
**Версия:** 1.0

---

## 1. Назначение документа

Документ предназначен для новых разработчиков, присоединяющихся к проекту DanceMate.

Цели:
- быстрый старт разработки;
- настройка локального окружения;
- понимание архитектуры и процессов;
- снижение порога входа в проект.

После прохождения онбординга разработчик должен:
- иметь рабочее локальное окружение;
- понимать архитектуру проекта;
- знать процессы разработки;
- уметь создавать и тестировать код.

---

## 2. Быстрый старт (5 минут)

### 2.1 Клонирование репозитория

```bash
git clone https://github.com/dancemate/dancemate.git
cd dancemate
```

### 2.2 Настройка окружения

```bash
# Создание виртуального окружения
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# или .venv\Scripts\activate  # Windows

# Установка зависимостей
pip install -e ".[dev]"
```

### 2.3 Настройка переменных окружения

```bash
cp .env.example .env
# Отредактируйте .env под ваше окружение
```

### 2.4 Запуск через Docker Compose

```bash
docker-compose up -d
```

### 2.5 Применение миграций

```bash
alembic upgrade head
```

### 2.6 Запуск приложения

```bash
uvicorn app.main:app --reload
```

### 2.7 Проверка

Откройте браузер:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- Health Check: http://localhost:8000/health

---

## 3. Требования к окружению

### 3.1 Обязательное ПО

| Компонент | Версия | Назначение |
|-----------|--------|------------|
| **Python** | 3.13+ | Основной язык |
| **Docker** | 24.0+ | Контейнеризация |
| **Docker Compose** | 2.20+ | Оркестрация контейнеров |
| **Git** | 2.40+ | Система контроля версий |
| **Make** | 3.81+ | Автоматизация задач (опционально) |

### 3.2 Рекомендуемое ПО

| Компонент | Назначение |
|-----------|------------|
| **PyCharm Professional** | IDE (поддержка FastAPI, Docker) |
| **VS Code** | Альтернативная IDE |
| **Postman** | Тестирование API |
| **TablePlus** | Работа с PostgreSQL |
| **Redis Insight** | Работа с Redis |

### 3.3 Python зависимости

Все зависимости описаны в `pyproject.toml`:

```bash
# Установка всех зависимостей
pip install -e ".[dev,test]"
```

Основные группы:
- `default` — основные зависимости (FastAPI, SQLAlchemy, и т.д.)
- `dev` — инструменты разработки (Ruff, mypy, pre-commit)
- `test` — тестирование (pytest, pytest-asyncio)

---

## 4. Настройка окружения

### 4.1 Структура переменных окружения

Создайте файл `.env` в корне проекта:

```env
# Приложение
APP_NAME=DanceMate
APP_ENV=development
APP_DEBUG=true
APP_SECRET_KEY=dev-secret-key-change-me

# База данных
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/dancemate
DATABASE_POOL_SIZE=5

# Redis
REDIS_URL=redis://localhost:6379/0

# MinIO
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=dancemate

# JWT
JWT_SECRET_KEY=dev-jwt-secret-change-me
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=30

# OTP (для SMS)
SMS_PROVIDER=mock  # mock, twilio, sms_ru
SMS_MOCK_VERIFICATION_CODE=123456

# Email
EMAIL_PROVIDER=mock  # mock, smtp, sendgrid
SMTP_HOST=localhost
SMTP_PORT=1025

# Платежи
PAYMENT_PROVIDER=mock  # mock, stripe, yookassa

# Логирование
LOG_LEVEL=DEBUG
LOG_JSON_FORMAT=false
```

### 4.2 Запуск инфраструктуры

#### Вариант 1: Docker Compose (рекомендуемый)

```bash
# Запуск всех сервисов
docker-compose up -d

# Проверка статуса
docker-compose ps

# Просмотр логов
docker-compose logs -f backend
```

Сервисы будут доступны:
- PostgreSQL: `localhost:5432`
- Redis: `localhost:6379`
- MinIO: `http://localhost:9000` (console: `http://localhost:9001`)
- Backend: `http://localhost:8000`

#### Вариант 2: Локальный запуск (без Docker)

```bash
# PostgreSQL
brew install postgresql@17  # Mac
brew services start postgresql@17

# Redis
brew install redis
brew services start redis

# MinIO
brew install minio/stable/minio
minio server ./data
```

---

## 5. Структура проекта

### 5.1 Основные каталоги

```
dancemate/
├── backend/                 # Backend приложение
│   ├── app/
│   │   ├── api/            # API Layer (REST, WebSocket)
│   │   ├── application/    # Application Layer (Use Cases)
│   │   ├── domain/         # Domain Layer (Ядро)
│   │   └── infrastructure/ # Infrastructure Layer (PostgreSQL, Redis)
│   ├── tests/              # Тесты
│   └── migrations/         # Alembic миграции
├── frontend/               # Flutter приложение (разрабатывается отдельно)
├── database/               # SQL проекты и схемы
├── docker/                 # Docker конфигурации
├── docs/                   # Архитектурная документация
└── scripts/                # Утилиты и скрипты
```

### 5.2 Архитектурные слои

```
API Layer (FastAPI)
    ↓
Application Layer (Use Cases)
    ↓
Domain Layer (Entities, Aggregates)
    ↑
Infrastructure Layer (SQLAlchemy, Redis)
```

**Важно:** Зависимости направлены внутрь. Domain Layer не знает о существовании других слоёв.

### 5.3 Ключевые файлы

| Файл | Назначение |
|------|------------|
| `app/main.py` | Точка входа FastAPI |
| `app/api/router.py` | Регистрация всех роутеров |
| `app/domain/base.py` | Базовые классы (Entity, AggregateRoot) |
| `app/infrastructure/database/session.py` | Настройка SQLAlchemy |
| `app/infrastructure/database/base.py` | Базовый класс ORM моделей |
| `alembic.ini` | Настройка Alembic |
| `pyproject.toml` | Зависимости и настройки проекта |
| `docker-compose.yml` | Конфигурация контейнеров |

---

## 6. Работа с базой данных

### 6.1 Миграции (Alembic)

```bash
# Создание новой миграции
alembic revision -m "add_users_table"

# Применение миграций
alembic upgrade head

# Откат миграции
alembic downgrade -1

# Просмотр состояния
alembic current
```

### 6.2 Работа с ORM

```python
# Пример использования SQLAlchemy в репозитории
from app.infrastructure.database.session import async_session
from app.domain.entities import User

async def get_user(user_id: UUID):
    async with async_session() as session:
        result = await session.execute(
            select(UserModel).where(UserModel.id == user_id)
        )
        return result.scalar_one_or_none()
```

### 6.3 Работа с PostgreSQL

```bash
# Подключение к БД через Docker
docker exec -it dancemate-postgres psql -U postgres -d dancemate

# Основные команды
\dt                # список таблиц
\d users           # структура таблицы
SELECT * FROM users LIMIT 10;
```

---

## 7. Работа с Redis

```bash
# Подключение к Redis через Docker
docker exec -it dancemate-redis redis-cli

# Основные команды
KEYS *             # все ключи
GET key            # получить значение
TTL key            # время жизни
DEL key            # удалить ключ
```

Использование в коде:

```python
from app.infrastructure.cache.redis import redis_client

# Кеширование
await redis_client.set("user:123", user_json, ex=3600)

# Получение
user_data = await redis_client.get("user:123")
```

---

## 8. Разработка

### 8.1 Создание новой фичи

1. **Создайте ветку** от `develop`:
   ```bash
   git checkout develop
   git pull
   git checkout -b feature/your-feature-name
   ```

2. **Опишите Domain Layer:**
   - Добавьте Entity в `app/domain/entities/`
   - Добавьте Value Object в `app/domain/value_objects/`
   - Добавьте Domain Event в `app/domain/events/`
   - Добавьте Repository Interface в `app/domain/repositories/`

3. **Реализуйте Application Layer:**
   - Создайте Command/Query в `app/application/commands/queries`
   - Создайте Handler в `app/application/command_handlers/query_handlers`
   - Добавьте Use Case в `app/application/use_cases/`

4. **Реализуйте Infrastructure Layer:**
   - Создайте ORM модель в `app/infrastructure/database/models/`
   - Реализуйте Repository в `app/infrastructure/repositories/`
   - Добавьте миграцию (если меняется схема)

5. **Добавьте API Layer:**
   - Создайте роутер в `app/api/v1/`
   - Добавьте DTO в `app/api/schemas/`
   - Добавьте зависимости в `app/api/dependencies/`

6. **Напишите тесты:**
   - Unit Tests в `tests/unit/`
   - Integration Tests в `tests/integration/`
   - API Tests в `tests/api/`

7. **Обновите документацию**

### 8.2 Тестирование

```bash
# Запуск всех тестов
pytest

# Запуск конкретного теста
pytest tests/unit/test_user.py

# Запуск с покрытием
pytest --cov=app --cov-report=html

# Открыть отчёт о покрытии
open htmlcov/index.html
```

### 8.3 Качество кода

```bash
# Форматирование
ruff format .

# Проверка стиля
ruff check .

# Проверка типов
mypy app

# Безопасность
bandit -r app

# Запуск всех проверок
pre-commit run --all-files
```

---

## 9. CI/CD

### 9.1 Процесс разработки

1. Создайте ветку `feature/*` от `develop`
2. Вносите изменения
3. Создайте Pull Request в `develop`
4. Дождитесь прохождения CI/CD
5. Получите Code Review
6. Объедините изменения

### 9.2 Pull Request Checklist

- [ ] Код соответствует архитектуре (слои не нарушены)
- [ ] Написаны тесты
- [ ] Тесты проходят
- [ ] Покрытие ≥ 90%
- [ ] Ruff и mypy не имеют ошибок
- [ ] Документация обновлена
- [ ] Миграции созданы (при необходимости)
- [ ] Нет `TODO`, `FIXME`, `print()`

### 9.3 GitHub Actions

При создании PR автоматически запускаются:
1. Проверка форматирования
2. Статический анализ
3. Unit Tests
4. Integration Tests
5. API Tests
6. Проверка безопасности
7. Проверка миграций

---

## 10. Полезные команды

### 10.1 Docker

```bash
# Запуск всех сервисов
docker-compose up -d

# Остановка всех сервисов
docker-compose down

# Пересборка контейнеров
docker-compose up -d --build

# Просмотр логов конкретного сервиса
docker-compose logs -f backend

# Очистка неиспользуемых контейнеров
docker system prune -a
```

### 10.2 Разработка

```bash
# Запуск сервера разработки
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Запуск с профилированием
python -m cProfile -o profile.stats app/main.py

# Создание суперпользователя (тестовый скрипт)
python scripts/create_admin.py

# Генерация тестовых данных
python scripts/seed_dev_data.py
```

### 10.3 База данных

```bash
# Создание миграции
alembic revision -m "description"

# Применение миграций
alembic upgrade head

# Откат на одну миграцию
alembic downgrade -1

# Просмотр истории миграций
alembic history

# Создание резервной копии
pg_dump -U postgres dancemate > backup.sql
```

---

## 11. Решение типичных проблем

### 11.1 Ошибка подключения к PostgreSQL

```bash
# Проверьте, запущен ли контейнер
docker ps | grep postgres

# Если нет, запустите
docker-compose up -d postgres

# Проверьте URL в .env
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/dancemate
```

### 11.2 Ошибка миграций

```bash
# Сброс базы (для разработки)
docker-compose down -v
docker-compose up -d postgres
alembic upgrade head
```

### 11.3 Ошибка импортов

```bash
# Убедитесь, что проект установлен как пакет
pip install -e .

# Проверьте PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:${PWD}"
```

### 11.4 Медленные тесты

```bash
# Запуск только нужных тестов
pytest tests/unit/test_specific.py

# Запуск без покрытия
pytest --no-cov

# Параллельный запуск
pytest -n auto
```

---

## 12. Архитектурная документация

Все архитектурные решения документированы. Начните с этих документов:

| Документ | Назначение |
|----------|------------|
| `01_PROJECT_STRUCTURE.md` | Общая структура и навигация |
| `02_SYSTEM_ARCHITECTURE.md` | Системная архитектура |
| `05_DOMAIN_LAYER.md` | Бизнес-логика (ядро) |
| `06_APPLICATION_LAYER.md` | Прикладной слой |
| `07_INFRASTRUCTURE_LAYER.md` | Инфраструктурный слой |
| `08_API_LAYER.md` | REST API и WebSocket |
| `09_SECURITY_ARCHITECTURE.md` | Безопасность |
| `12_CODING_STANDARDS.md` | Стандарты кодирования |
| `GLOSSARY.md` | Глоссарий терминов |
| `ERROR_CODES.md` | Коды ошибок API |

---

## 13. Коммуникация в команде

### 13.1 Каналы связи

- **Telegram** — основная оперативная коммуникация
- **GitHub Issues** — задачи и баги
- **GitHub Discussions** — обсуждение архитектуры
- **Email** — официальные уведомления

### 13.2 Встречи

- **Daily Standup** — ежедневно, 10:00, 15 минут
- **Sprint Planning** — раз в 2 недели, 1 час
- **Architecture Review** — по запросу
- **Code Review** — асинхронно (каждый PR)

### 13.3 Ответственность

- **Team Lead** — архитектура, технические решения
- **Senior Developers** — код-ревью, mentorship
- **Developers** — разработка, тестирование, документация
- **DevOps** — инфраструктура, CI/CD, мониторинг

---

## 14. Полезные ссылки

### 14.1 Технологии

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/20/)
- [Pydantic V2](https://docs.pydantic.dev/latest/)
- [Alembic](https://alembic.sqlalchemy.org/)
- [Docker](https://docs.docker.com/)
- [PostgreSQL 17](https://www.postgresql.org/docs/17/)

### 14.2 Архитектура

- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
- [Clean Architecture by Robert Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)

### 14.3 Инструменты

- [Ruff](https://docs.astral.sh/ruff/)
- [Mypy](https://mypy-lang.org/)
- [Pre-commit](https://pre-commit.com/)
- [Pytest](https://docs.pytest.org/)

---

## 15. Заключение

После прохождения этого онбординга вы готовы к работе над проектом DanceMate.

**Ключевые моменты:**
1. Архитектура — строгая, не нарушайте слои.
2. Domain Layer — ядро, не знает о внешнем мире.
3. Тесты — обязательны, покрытие ≥ 90%.
4. Code Review — обязателен для всех изменений.
5. Документация — всегда актуальна, обновляйте её.

**Добро пожаловать в команду DanceMate! 🎉**

---

## 16. Обратная связь

Если вы нашли ошибку в этом документе или хотите его улучшить:

1. Создайте Issue в GitHub
2. Или создайте Pull Request с правками
3. Обсудите с командой