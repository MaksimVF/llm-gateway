# LLM Gateway

Продвинутый REST API-сервис на FastAPI для проксирования запросов к серверу инференса LLM с комплексной системой биллинга, rate limiting и мониторинга токенов.

## 🚀 Возможности

- ✅ **API Key Authentication** - Безопасная аутентификация по API-ключам
- ✅ **Rate Limiting** - Ограничение запросов (30 req/min) с Redis/fallback
- ✅ **Token Tracking** - Точный учёт использования токенов
- ✅ **Billing System** - Дневные/месячные лимиты с SQLite хранением
- ✅ **Tariff Management** - Гибкая система тарифов для разных ключей
- ✅ **Health Monitoring** - Проверка состояния всех компонентов
- ✅ **Fallback Storage** - Автоматическое переключение при недоступности Redis
- ✅ **Comprehensive Logging** - Детальное логирование всех операций
- ✅ **CORS Support** - Поддержка веб-интерфейсов

## 📁 Структура проекта

```
llm-gateway/
├── main.py                         # Запуск FastAPI-приложения  
├── gateway/
│   ├── __init__.py
│   ├── routes.py                   # Основные API маршруты
│   ├── auth.py                     # Аутентификация API-ключей
│   ├── client.py                   # Проксирование к LLM серверу
│   └── limiter.py                  # Rate limiting с Redis
├── billing/
│   ├── __init__.py
│   ├── models.py                   # SQLAlchemy модели
│   ├── tracker.py                  # Биллинг и статистика
│   ├── limits.py                   # Проверка лимитов
│   ├── token_tracker.py            # Отслеживание токенов
│   ├── tariffs.py                  # Система тарифов
│   ├── utils.py                    # Утилиты для работы с ключами
│   └── keys.json                   # Хранилище API-ключей
├── tools/                          # CLI-утилиты
│   ├── __init__.py
│   ├── cli.py                      # CLI для управления ключами
│   └── key_utils.py                # Утилиты для CLI
├── .env                            # Конфигурация
├── requirements.txt                # Зависимости
├── BILLING.md                      # Документация биллинга
├── RATE_LIMITING.md                # Документация rate limiting
├── TOKEN_TRACKING.md               # Документация токен трекинга
├── CLI.md                          # Документация CLI-утилиты
└── README.md                       # Основная документация
```

## Установка и запуск

### 1. Установка зависимостей

```bash
pip install -r requirements.txt
```

### 2. Настройка переменных окружения

Отредактируйте файл `.env`:

```env
# Список разрешённых API-ключей (разделённых запятой)
ALLOWED_API_KEYS=sk-test-key-1,sk-test-key-2,sk-prod-key-123

# URL сервера LLM для проксирования запросов
LLM_SERVER_URL=http://localhost:8080/v1/completions

# Настройки логирования
LOG_LEVEL=INFO

# Лимиты биллинга (глобальные по умолчанию)
API_DAILY_LIMIT=50000
API_MONTHLY_LIMIT=1000000

# База данных для биллинга
DATABASE_URL=sqlite:///./billing.db

# Настройки Redis для rate limiting и token tracking
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=

# Настройки rate limiting
RATE_LIMIT_REQUESTS=30
RATE_LIMIT_WINDOW=60

# Тарифы по умолчанию
DEFAULT_MAX_TOKENS=50000
```

### 3. Запуск сервера

```bash
# Запуск через Python
python main.py

# Или через uvicorn
uvicorn main:app --host 0.0.0.0 --port 12000 --reload
```

Сервер будет доступен по адресу: `http://localhost:12000`

## API Endpoints

### POST /v1/completions

Основной эндпоинт для генерации текста. Проксирует запросы к серверу LLM.

**Заголовки:**
- `Authorization: Bearer <your-api-key>`
- `Content-Type: application/json`

**Пример запроса:**

```bash
curl -X POST "http://localhost:12000/v1/completions" \
  -H "Authorization: Bearer sk-test-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Привет! Как дела?",
    "max_tokens": 100,
    "temperature": 0.7
  }'
```

### GET /usage

Получение комплексной статистики использования для API-ключа.

```bash
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/usage
```

**Ответ включает:**
- Статистику биллинга (дневная/месячная)
- Данные TokenTracker
- Информацию о тарифе
- Состояние rate limiting

### GET /limits

Получение информации о лимитах для API-ключа.

```bash
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/limits
```

### GET /tariff

Информация о тарифе и использовании токенов.

```bash
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/tariff
```

**Ответ:**
```json
{
  "tariff": {
    "max_tokens": 10000,
    "name": "Test Basic",
    "description": "Базовый тестовый тариф",
    "api_key": "sk-test-..."
  },
  "usage": {
    "used_tokens": 1500,
    "remaining_tokens": 8500,
    "usage_percentage": 15.0
  }
}
```

### GET /rate-limit

Текущее состояние rate limiting для API-ключа.

```bash
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/rate-limit
```

**Ответ:**
```json
{
  "api_key": "sk-test-...",
  "limit": 30,
  "window_seconds": 60,
  "current_count": 5,
  "remaining": 25,
  "storage": "redis"
}
```

### GET /health

Проверка состояния сервиса.

```bash
curl http://localhost:12000/health
```

### GET /

Информация о сервисе и доступных эндпоинтах.

```bash
curl http://localhost:12000/
```

## 🔧 CLI-утилита управления ключами

Проект включает мощную CLI-утилиту для управления API-ключами и тарифами.

### Основные команды

```bash
# Создание нового API-ключа
cd tools
python cli.py create-key --limit 100000 --name "Premium Plan"

# Просмотр всех ключей
python cli.py list-keys --stats

# Обновление лимита токенов
python cli.py set-limit --key sk-test-key-1 --limit 50000

# Удаление ключа
python cli.py revoke-key --key sk-old-key --force

# Статистика
python cli.py stats

# Резервное копирование
python cli.py backup --path backup.json

# Восстановление
python cli.py restore --path backup.json
```

### Пример вывода

```
📋 Список API-ключей (4 шт.):
================================================================================
🔑 sk-prod-key-123
   📊 Лимит: 1,000,000 токенов
   🏷️  Название: Production Premium

📈 Статистика:
   Всего ключей: 4
   Общий лимит: 1,115,000 токенов
   Средний лимит: 278,750 токенов
```

Подробная документация: [CLI.md](CLI.md)

## Безопасность

- API-ключи проверяются через заголовок `Authorization: Bearer <key>`
- Поддерживается несколько ключей в переменной `ALLOWED_API_KEYS`
- Все ошибки аутентификации логируются
- Чувствительная информация не выводится в логах
- CLI маскирует ключи в выводе для безопасности

## 🧪 Тестирование

### Запуск тестов биллинга

```bash
python test_billing.py
```

### Запуск тестов расширенных функций

```bash
python test_advanced_features.py
```

### Тестирование API

```bash
# Проверка rate limiting
for i in {1..5}; do curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/tariff; done

# Проверка биллинга
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/usage

# Проверка тарифов
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/tariff
```

## 📊 Мониторинг

### Проверка состояния компонентов

```bash
# Общее состояние
curl http://localhost:12000/health

# Состояние Redis
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/rate-limit

# Статистика использования
curl -H "Authorization: Bearer sk-test-key-1" http://localhost:12000/usage
```

### Логирование

Система логирует:
- Успешные и неуспешные запросы
- Rate limiting события
- Превышения биллинговых лимитов
- Ошибки подключения к Redis/LLM
- Переключения на fallback хранилище

## ⚠️ Обработка ошибок

Сервис обрабатывает следующие типы ошибок:

- **401 Unauthorized** - API ключ не предоставлен
- **403 Forbidden** - Недопустимый API ключ  
- **429 Too Many Requests** - Превышен rate limit или биллинговые лимиты
- **400 Bad Request** - Ошибка в формате запроса
- **502 Bad Gateway** - Ошибка подключения к LLM серверу
- **504 Gateway Timeout** - Таймаут при обращении к LLM серверу

## Логирование

Сервис логирует:
- Входящие запросы (с маскированием API ключей)
- Ошибки подключения к LLM серверу
- HTTP ошибки и таймауты
- Успешные операции

## Разработка

### Запуск в режиме разработки

```bash
uvicorn main:app --host 0.0.0.0 --port 12000 --reload
```

### Документация API

FastAPI автоматически генерирует документацию:
- Swagger UI: `http://localhost:12000/docs`
- ReDoc: `http://localhost:12000/redoc`

## Примеры использования

### Простой запрос

```bash
curl -X POST "http://localhost:12000/v1/completions" \
  -H "Authorization: Bearer sk-test-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Напиши короткий рассказ о роботе",
    "max_tokens": 200,
    "temperature": 0.8
  }'
```

### Проверка здоровья сервиса

```bash
curl http://localhost:12000/health
```

### Получение информации о сервисе

```bash
curl http://localhost:12000/
```

## 📚 Документация

### Основные документы

- **[BILLING.md](BILLING.md)** - Подробная документация системы биллинга
- **[RATE_LIMITING.md](RATE_LIMITING.md)** - Документация по rate limiting
- **[TOKEN_TRACKING.md](TOKEN_TRACKING.md)** - Документация по отслеживанию токенов

### API документация

FastAPI автоматически генерирует интерактивную документацию:
- **Swagger UI**: `http://localhost:12000/docs`
- **ReDoc**: `http://localhost:12000/redoc`

### Архитектурные решения

#### Хранилище данных
- **SQLite** - для биллинговой статистики (персистентное)
- **Redis** - для rate limiting и token tracking (с TTL)
- **Memory fallback** - резервное хранилище при недоступности Redis

#### Безопасность
- Маскирование API ключей в логах
- Валидация всех входящих данных
- Изоляция данных между ключами
- Защита от DDoS через rate limiting

#### Производительность
- Асинхронная обработка запросов
- Эффективное кэширование в Redis
- Минимальные накладные расходы на проверки
- Автоматическое переключение на fallback

## 🚀 Развертывание

### Docker (рекомендуется)

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 12000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "12000"]
```

### Systemd сервис

```ini
[Unit]
Description=LLM Gateway
After=network.target

[Service]
Type=simple
User=llm-gateway
WorkingDirectory=/opt/llm-gateway
ExecStart=/opt/llm-gateway/venv/bin/uvicorn main:app --host 0.0.0.0 --port 12000
Restart=always

[Install]
WantedBy=multi-user.target
```

### Nginx прокси

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:12000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## 🤝 Вклад в проект

1. Fork репозитория
2. Создайте feature branch (`git checkout -b feature/amazing-feature`)
3. Commit изменения (`git commit -m 'Add amazing feature'`)
4. Push в branch (`git push origin feature/amazing-feature`)
5. Откройте Pull Request

## 📄 Лицензия

Этот проект распространяется под лицензией MIT. См. файл `LICENSE` для подробностей.

## 📞 Поддержка

Если у вас есть вопросы или проблемы:
1. Проверьте [Issues](https://github.com/MaksimVF/llm-gateway/issues)
2. Создайте новый Issue с подробным описанием
3. Приложите логи и конфигурацию (без чувствительных данных)