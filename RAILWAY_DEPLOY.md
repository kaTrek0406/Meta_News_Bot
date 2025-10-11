# 🚀 Деплой бота на Railway

Этот гайд поможет задеплоить Meta News Bot на Railway через GitHub.

## 📋 Подготовка

### 1. Создайте репозиторий на GitHub

1. Зайдите на [GitHub](https://github.com) и создайте новый репозиторий (можно приватный)
2. Скопируйте URL репозитория (например: `https://github.com/yourusername/meta_news_bot.git`)

### 2. Загрузите код в GitHub

```bash
# В директории проекта выполните:
cd D:\meta_news_bot_json_patched\meta_news_bot_json_railway

# Добавьте удалённый репозиторий
git remote add origin https://github.com/ваш-username/ваш-репозиторий.git

# Отправьте код
git branch -M main
git push -u origin main
```

## 🚂 Деплой на Railway

### 1. Создайте аккаунт на Railway

1. Зайдите на [Railway.app](https://railway.app)
2. Войдите через GitHub

### 2. Создайте новый проект

1. Нажмите **"New Project"**
2. Выберите **"Deploy from GitHub repo"**
3. Выберите ваш репозиторий `meta_news_bot`
4. Railway автоматически обнаружит Python проект

### 3. Настройте переменные окружения

В Railway перейдите в раздел **Variables** и добавьте следующие переменные из вашего `.env` файла:

#### Обязательные переменные:

```bash
TELEGRAM_BOT_TOKEN=ваш_токен_бота
TELEGRAM_CHAT_ID=ваш_chat_id
OPENROUTER_API_KEY=ваш_openrouter_ключ
```

#### Опциональные переменные:

```bash
# OpenRouter настройки
OPENROUTER_SITE_URL=https://meta-news-bot.local
OPENROUTER_SITE_TITLE=Meta News Bot
LLM_MODEL=openai/gpt-4o-mini
LLM_TEMPERATURE=0.2
LLM_MAX_TOKENS=1800
LLM_REQUEST_TIMEOUT=45
LLM_RETRY_ATTEMPTS=3
LLM_RETRY_BACKOFF_SECONDS=2
PAID_FALLBACK_MAX_PER_RUN=0

# Планировщик
DAILY_DISPATCH_TIME=09:00
TZ=UTC
DAILY_DEV_ONLY=1

# Автоперевод
AUTO_TRANSLATE_DIFFS=1
MAX_NOTIFY_CHARS=1400

# Ограничения LLM
MAX_INPUT_CHARS=9000
CHUNK_CHARS=1800
MAX_CHUNKS=4
CHUNK_MAX_TOKENS=400

# Dev Chat ID (опционально)
TELEGRAM_DEV_CHAT_ID=527824690

# Логи
LOGLEVEL=INFO
```

### 4. Деплой

Railway автоматически начнёт деплой после настройки переменных окружения.

## ✅ Проверка работы

1. В логах Railway вы должны увидеть: `"Бот запущен. Ожидаю команды…"`
2. Напишите боту команду `/start` в Telegram
3. Бот должен ответить списком категорий

## 🔄 Автоматические обновления

После настройки каждый `git push` в main ветку будет автоматически деплоить изменения на Railway.

```bash
# Внесите изменения в код
git add .
git commit -m "Описание изменений"
git push
```

Railway автоматически пересоберёт и перезапустит бота.

## 📊 Мониторинг

- **Логи**: В Railway откройте вкладку "Deployments" → выберите активный деплой → "View Logs"
- **Метрики**: Вкладка "Metrics" покажет использование CPU и памяти
- **Рестарты**: Бот настроен на автоматический перезапуск при ошибках (до 10 попыток)

## 💰 Тарифы Railway

- **Free Tier**: $5 в месяц бесплатных ресурсов (достаточно для небольшого бота)
- **Developer Plan**: $5/месяц за $5 ресурсов + $0.000231/GB-hour, $0.000463/vCPU-hour

## 🐛 Troubleshooting

### Бот не запускается

1. Проверьте логи в Railway
2. Убедитесь, что все обязательные переменные окружения установлены
3. Проверьте, что `TELEGRAM_BOT_TOKEN` корректен

### Бот не отвечает

1. Проверьте, что бот запущен (логи должны показывать "Бот запущен")
2. Убедитесь, что вы используете правильный `TELEGRAM_CHAT_ID`
3. Проверьте интернет-соединение Railway (обычно всегда работает)

### Ошибки с данными

Railway использует эфемерное хранилище. При каждом деплое данные в папке `data/` будут сброшены. Для постоянного хранилища рассмотрите:
- Railway Volumes (платная опция)
- Внешнее хранилище (S3, PostgreSQL и т.д.)

## 📚 Полезные ссылки

- [Railway Documentation](https://docs.railway.app/)
- [Railway Discord](https://discord.gg/railway)
- [Python on Railway](https://docs.railway.app/guides/python)
