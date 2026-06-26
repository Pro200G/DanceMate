# 01_ROOT_STRUCTURE.md

> Проект: **DanceMate**
> Документ: **Структура корневого каталога проекта**
> Версия: **1.0**

---

# 1. Назначение документа

Документ описывает организацию репозитория проекта DanceMate.

Цель документа:

* определить единый стандарт хранения файлов;
* разделить зоны ответственности Backend и Frontend;
* определить место хранения архитектурной документации;
* определить место хранения инфраструктуры;
* обеспечить масштабируемость проекта.

Настоящий документ описывает **только верхний уровень проекта**.

Структура Backend подробно рассматривается в документе **02_BACKEND_STRUCTURE.md**.

---

# 2. Корневая структура проекта

```text
DanceMate/
│
├── backend/
├── frontend/
├── docs/
├── infrastructure/
├── .github/
│
├── docker-compose.yml
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CONTRIBUTING.md
├── .editorconfig
├── .gitignore
└── .gitattributes
```

---

# 3. Назначение каталогов

## backend/

Содержит серверную часть приложения.

Включает:

* REST API
* WebSocket
* бизнес-логику
* модели данных
* PostgreSQL
* Redis
* Celery
* SQL-проект
* миграции
* тесты Backend

Внутренняя структура описана отдельно.

Документ:

```
02_BACKEND_STRUCTURE.md
```

---

## frontend/

Содержит мобильное приложение Flutter.

Включает:

* Android
* iOS
* Web
* Assets
* UI
* State Management
* API Client

Подробности:

```
08_FRONTEND_STRUCTURE.md
```

---

## docs/

Архитектурная документация проекта.

Все документы проекта находятся только здесь.

Структура:

```text
docs/

PROJECT_STRUCTURE.md

01_ROOT_STRUCTURE.md

02_BACKEND_STRUCTURE.md

03_DATABASE_STRUCTURE.md

04_DOMAIN_LAYER.md

05_USE_CASES.md

06_INFRASTRUCTURE.md

07_API.md

08_FRONTEND_STRUCTURE.md

09_DEPENDENCIES.md

10_SOLID_RULES.md

11_NAMING.md

12_TESTING.md

13_DEPLOYMENT.md
```

Документация хранится рядом с кодом проекта.

Любое изменение архитектуры сопровождается обновлением документации.

---

## infrastructure/

Содержит инфраструктуру проекта.

Пример структуры:

```text
infrastructure/

docker/

nginx/

kubernetes/

terraform/

monitoring/

logging/

ssl/
```

Назначение:

* Docker
* Nginx
* Kubernetes
* Grafana
* Prometheus
* Terraform
* SSL-сертификаты

В инфраструктуре отсутствует бизнес-логика.

---

## .github/

Содержит автоматизацию GitHub.

Структура:

```text
.github/

workflows/

ISSUE_TEMPLATE/

PULL_REQUEST_TEMPLATE.md

CODEOWNERS
```

Используется для:

* CI
* CD
* Code Review
* автоматического тестирования

---

# 4. Назначение файлов

## README.md

Главный документ проекта.

Содержит:

* описание проекта;
* инструкции по запуску;
* ссылки на документацию;
* стек технологий;
* требования.

---

## LICENSE

Лицензия проекта.

---

## CHANGELOG.md

История изменений проекта.

Используется формат:

Semantic Versioning.

---

## CONTRIBUTING.md

Правила участия в разработке.

Содержит:

* оформление Pull Request;
* стиль кода;
* требования к тестам;
* требования к документации.

---

## docker-compose.yml

Главный compose-файл проекта.

Используется для локальной разработки.

Поднимает:

* PostgreSQL;
* Redis;
* MinIO;
* Backend;
* Frontend;
* Nginx.

Production использует отдельные compose-файлы.

---

## .gitignore

Определяет файлы, которые не попадают в Git.

Не должны попадать:

* .env
* **pycache**
* .idea
* .vscode
* uploads
* backups
* Docker volumes

---

## .editorconfig

Единые настройки IDE.

Используется всеми разработчиками.

---

## .gitattributes

Настройки Git.

Используется для:

* LF
* UTF-8
* diff
* merge

---

# 5. Правила организации проекта

Корневой каталог не содержит бизнес-логики.

Все файлы должны относиться к одной из категорий:

* Backend
* Frontend
* Documentation
* Infrastructure
* Automation

Другие каталоги допускаются только после архитектурного согласования.

---

# 6. Принципы масштабирования

При увеличении проекта новые сервисы добавляются как самостоятельные каталоги.

Например:

```text
DanceMate/

backend/

frontend/

admin-panel/

analytics/

notification-service/

docs/

infrastructure/
```

Каждый сервис имеет собственную архитектуру.

---

# 7. Правила именования

Все каталоги используют:

```
snake_case
```

Примеры:

```
backend

frontend

infrastructure

notification_service
```

Документы используют:

```
UPPER_CASE.md
```

Примеры:

```
PROJECT_STRUCTURE.md

DATABASE_STRUCTURE.md

API.md
```

---

# 8. Зависимости верхнего уровня

Допустимые зависимости:

```text
Frontend

↓

Backend

↓

Infrastructure

↓

Database
```

Документация не зависит ни от одного слоя.

Infrastructure не содержит бизнес-логики.

Backend не зависит от Frontend.

Frontend не имеет доступа к базе данных.

Все взаимодействие производится исключительно через API и WebSocket.

---

# 9. Ответственность каталогов

| Каталог            | Ответственность                |
| ------------------ | ------------------------------ |
| backend            | Бизнес-логика и API            |
| frontend           | Клиентское приложение          |
| docs               | Архитектурная документация     |
| infrastructure     | Инфраструктура и развёртывание |
| .github            | Автоматизация GitHub           |
| docker-compose.yml | Локальное окружение            |
| README.md          | Основная документация          |
| CHANGELOG.md       | История изменений              |
| CONTRIBUTING.md    | Правила разработки             |
