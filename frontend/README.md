# LLM Gateway Frontend

Современный фронтенд для LLM Gateway, построенный с использованием React, TypeScript, Vite и TailwindCSS.

## 🚀 Технологии

- **React 18** - Библиотека для создания пользовательских интерфейсов
- **TypeScript** - Типизированный JavaScript
- **Vite** - Быстрый сборщик и dev-сервер
- **TailwindCSS** - Utility-first CSS фреймворк
- **React Router** - Маршрутизация
- **Zustand** - Управление состоянием
- **React Hook Form + Zod** - Формы и валидация
- **Axios** - HTTP клиент
- **Lucide React** - Иконки

## 📁 Структура проекта

```
frontend/
├── src/
│   ├── components/          # Компоненты
│   │   ├── ui/             # Базовые UI компоненты
│   │   ├── layout/         # Layout компоненты
│   │   └── auth/           # Компоненты аутентификации
│   ├── pages/              # Страницы приложения
│   ├── store/              # Zustand stores
│   ├── lib/                # Утилиты и API клиент
│   ├── hooks/              # Кастомные хуки
│   ├── types/              # TypeScript типы
│   ├── styles/             # Стили
│   └── assets/             # Статические ресурсы
├── public/                 # Публичные файлы
└── dist/                   # Собранное приложение
```

## 🛠 Установка и запуск

### Предварительные требования

- Node.js 18+ 
- npm или yarn

### Установка зависимостей

```bash
cd frontend
npm install
```

### Переменные окружения

Создайте файл `.env` в корне папки frontend:

```env
# API Configuration
VITE_API_URL=http://localhost:12000

# App Configuration
VITE_APP_NAME=LLM Gateway
VITE_APP_VERSION=1.0.0

# Environment
VITE_NODE_ENV=development

# Features
VITE_ENABLE_TELEGRAM_AUTH=false
VITE_ENABLE_OAUTH=false
VITE_ENABLE_ANALYTICS=false

# Debug
VITE_DEBUG=true
```

### Запуск в режиме разработки

```bash
npm run dev
```

Приложение будет доступно по адресу: http://localhost:12001

### Сборка для продакшена

```bash
npm run build
```

### Предварительный просмотр сборки

```bash
npm run preview
```

## 🎨 Компоненты

### UI Компоненты

- **Button** - Кнопки с различными вариантами стилей
- **Input** - Поля ввода с валидацией
- **Card** - Карточки для группировки контента
- **Modal** - Модальные окна
- **Badge** - Бейджи для статусов
- **Spinner** - Индикаторы загрузки
- **Toast** - Уведомления

### Layout Компоненты

- **Header** - Шапка сайта с навигацией
- **Footer** - Подвал сайта
- **Layout** - Основной layout wrapper

## 📱 Страницы

- **HomePage** - Главная страница с описанием сервиса
- **LoginPage** - Страница входа
- **RegisterPage** - Страница регистрации
- **DashboardPage** - Панель управления (в разработке)
- **ApiKeysPage** - Управление API ключами (в разработке)
- **PlaygroundPage** - Тестирование API (в разработке)
- **DocsPage** - Документация (в разработке)
- **PricingPage** - Тарифы (в разработке)

## 🔐 Аутентификация

Приложение использует JWT токены для аутентификации:

- Токены сохраняются в localStorage
- Автоматическое обновление токенов
- Защищенные маршруты
- Перенаправление неавторизованных пользователей

## 🎯 Состояние приложения

Управление состоянием осуществляется через Zustand stores:

- **authStore** - Аутентификация и данные пользователя
- **notificationStore** - Уведомления и тосты

## 🌐 API Интеграция

API клиент настроен для работы с backend:

- Автоматическое добавление токенов авторизации
- Обработка ошибок и повторных запросов
- Типизированные методы для всех эндпоинтов

## 🎨 Стилизация

Используется TailwindCSS с кастомной конфигурацией:

- Кастомная цветовая палитра
- Готовые компоненты в globals.css
- Responsive дизайн
- Темная тема (планируется)

## 🔧 Разработка

### Добавление новой страницы

1. Создайте компонент в `src/pages/`
2. Добавьте маршрут в `App.tsx`
3. Обновите навигацию в `Header.tsx`

### Добавление нового API эндпоинта

1. Добавьте типы в `src/types/index.ts`
2. Создайте методы в `src/lib/api.ts`
3. Используйте в компонентах

### Добавление нового UI компонента

1. Создайте компонент в `src/components/ui/`
2. Экспортируйте из `src/components/ui/index.ts`
3. Добавьте стили в TailwindCSS

## 📦 Сборка и деплой

### Docker

```bash
# Сборка образа
docker build -t llm-gateway-frontend .

# Запуск контейнера
docker run -p 80:80 llm-gateway-frontend
```

### Nginx

Пример конфигурации для Nginx:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    root /var/www/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://backend:12000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🐛 Отладка

### Включение debug режима

Установите `VITE_DEBUG=true` в `.env` файле.

### Просмотр состояния

Используйте React DevTools и Zustand DevTools для отладки состояния.

### Логирование API запросов

API клиент автоматически логирует запросы в development режиме.

## 🤝 Вклад в проект

1. Форкните репозиторий
2. Создайте feature ветку
3. Внесите изменения
4. Добавьте тесты (если необходимо)
5. Создайте Pull Request

## 📄 Лицензия

MIT License - см. файл LICENSE для деталей.

## 🆘 Поддержка

- GitHub Issues: [Создать issue](https://github.com/MaksimVF/llm-gateway/issues)
- Email: support@llmgateway.dev
- Telegram: @llmgateway_support

## 🗺 Roadmap

- [ ] Темная тема
- [ ] Интернационализация (i18n)
- [ ] PWA поддержка
- [ ] Тесты (Jest + Testing Library)
- [ ] Storybook для компонентов
- [ ] Telegram авторизация
- [ ] OAuth провайдеры
- [ ] Аналитика и метрики
- [ ] Офлайн режим