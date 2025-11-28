# Запуск проекта через Docker Compose

Этот проект настроен для запуска через Docker Compose, включая все необходимые сервисы.

## Структура сервисов

- **db** - PostgreSQL база данных
- **minio** - MinIO для хранения файлов
- **backend** - FastAPI бэкенд приложения
- **frontend** - React фронтенд приложения

## Требования

- Docker
- Docker Compose
- OpenRouter Api Key

## Запуск проекта

1. Перейдите в корневую директорию проекта:
```bash
cd <repository-name>
```

В файл backend/.env подложите свой API ключ от Openrouter (можно получить бесплатно по [ссылке](https://openrouter.ai/settings/keys) после авторизации)

2. Запустите все сервисы:
```bash
docker-compose up --build
```

Для запуска в фоновом режиме:
```bash
docker-compose up -d --build
```

## Что происходит при запуске

1. **База данных (PostgreSQL)** запускается первой и ждет готовности
2. **MinIO** запускается и ждет готовности
3. **Backend** запускается после готовности БД и MinIO:
   - Выполняет скрипт `migrations/init_db.py` для инициализации базы данных
   - Выполняет скрипт `migrations/migrate_json_to_db.py` для миграции данных из JSON
   - Запускает FastAPI приложение на порту 8000
4. **Frontend** запускается после готовности backend на порту 3000

## Порты

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **MinIO Console**: http://localhost:9001 (minioadmin/minioadmin)
- **MinIO API**: http://localhost:9000
- **PostgreSQL**: localhost:5432

## Остановка проекта

```bash
docker-compose down
```

Для остановки и удаления volumes (удалит все данные):
```bash
docker-compose down -v
```

## Просмотр логов

Все сервисы:
```bash
docker-compose logs -f
```

Конкретный сервис:
```bash
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f db
docker-compose logs -f minio
```

## Переменные окружения

Основные переменные окружения можно изменить в файле `docker-compose.yml`:

### Backend
- `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` - настройки БД
- `SECRET_KEY` - секретный ключ для JWT (важно изменить в продакшене!)
- `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` - настройки MinIO

### Frontend
- `REACT_APP_API_URL` - URL бэкенд API

## Данные для входа администратора

После первого запуска создается администратор:
- **Email**: admin@gooddeeds.ru
- **Пароль**: admin123

⚠️ **ВАЖНО**: Смените пароль администратора после первого входа!

## Устранение проблем

### Backend не может подключиться к БД
Убедитесь, что БД полностью запустилась (проверьте логи):
```bash
docker-compose logs db
```

### Скрипты миграции не выполняются
Проверьте логи backend:
```bash
docker-compose logs backend
```

### Frontend не подключается к Backend
Убедитесь, что:
1. Backend запущен и доступен на порту 8000
2. В `frontend/src/services/api.js` указан правильный URL (по умолчанию `http://localhost:8000`)

## Пересборка после изменений

Если вы внесли изменения в код:
```bash
docker-compose up --build
```

Для пересборки конкретного сервиса:
```bash
docker-compose build backend
docker-compose up -d backend
```

