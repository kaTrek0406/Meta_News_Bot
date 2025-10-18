# ✅ Региональный рефакторинг - выполнена основная работа

## 📋 Что сделано

### 1. ✅ config.json
- Добавлен параметр `region` для всех источников (EU, MD, GLOBAL)
- Добавлен опциональный `lang` для переопределения Accept-Language
- Добавлен опциональный `proxy_country` для переопределения страны прокси
- Пример: `{ "url": "...", "tag": "news_policy", "region": "EU", "lang": "en-GB", "proxy_country": "de" }`

### 2. ✅ src/config.py
- Добавлены новые ENV-переменные:
  ```python
  PROXY_URL: str  # Полный URL прокси (MD)
  PROXY_URL_EU: str  # Fallback прокси для EU
  USE_PROXY: bool
  PROXY_PROVIDER: str  # "froxy" или другой
  PROXY_STICKY: bool  # Sticky sessions
  PROXY_FALLBACK_EU: bool  # Fallback MD→EU
  ```
- Добавлена валидация:
  - `ensure_telegram_token()` - проверяет наличие токена и хотя бы одного чата
  - `validate_proxy_config()` - проверяет корректность настроек прокси
  - `log_config_summary()` - безопасно логирует конфигурацию без секретов

### 3. ✅ src/pipeline.py
**Ключевые изменения:**

- **Новая функция** `_get_proxy_for_region(region, proxy_country, session_id)`:
  - Возвращает подходящий прокси URL в зависимости от региона источника
  - Поддерживает Froxy sticky-sessions через `session=<rand>` в пароле
  - Для MD использует `PROXY_URL`, для EU - `PROXY_URL_EU`

- **Обновлена функция** `_get_random_headers(url, accept_lang)`:
  - Принимает кастомный `Accept-Language` заголовок
  - Использует регионально-специфичные языки

- **Изменен ключ кэша**: `(tag, url, region)` вместо `(tag, url)`
  - Один источник может иметь разные версии для разных регионов

- **Добавлен Fallback MD→EU**:
  ```python
  # При 407/403 с MD прокси переключается на EU автоматически
  if status in (407, 403) and region == "MD" and PROXY_FALLBACK_EU:
      proxies = _get_proxy_for_region("EU", proxy_country, session_id)
      used_fallback = True
  ```

- **Каждый источник создает свой HTTP-клиент** с правильным прокси и заголовками
- **Добавлен `region` в item** перед сохранением в кэш
- **Обновлена функция `PRUNE_REMOVED_SOURCES`** для работы с новым ключом кэша

### 4. ✅ src/storage.py
- **Auto-migration на лету**: при загрузке кэша автоматически добавляет `region='GLOBAL'` к записям без region
  ```python
  # В load_cache()
  for item in data["items"]:
      if "region" not in item:
          item["region"] = "GLOBAL"
  ```

### 5. ✅ scripts/migrate_region_tag.py
- Скрипт миграции для единовременного обновления всех файлов кэша
- Обрабатывает:
  - `data/cache/cache.json`
  - `data/items.json` (если существует)
- Выводит подробный отчёт о миграции
- Безопасное сохранение через временные файлы

### 6. ✅ src/smart_formatter.py
- Добавлен словарь `REGION_BADGES` с эмодзи флагов:
  ```python
  REGION_BADGES = {
      "EU": "🇪🇺 [EU]",
      "MD": "🇲🇩 [MD]",
      "GLOBAL": "🌍 [GLOBAL]",
  }
  ```
- Обновлены функции `_format_api_change` и `_format_policy_change`:
  - Извлекают `region` из detail
  - Добавляют региональный badge к заголовку

## 🔧 Что осталось доделать (по желанию)

### src/telegram_notify.py
Добавить группировку по регионам при отправке (см. `REGION_REFACTOR_TODO.md`):
```python
def group_by_region(details: list) -> dict:
    # Группирует изменения: EU → MD → GLOBAL
    ...
```

### src/tg/handlers.py
Добавить региональную статистику в команду `/status`:
```python
# Выводить:
# 🇪🇺 EU — источников: 2, изменений за 24ч: 5
# 🇲🇩 MD — источников: 0, изменений за 24ч: 0
# 🌍 GLOBAL — источников: 25, изменений за 24ч: 12
```

## 📝 Настройка .env

Добавить в `.env.template` и `.env`:

```ini
# Прокси (Froxy)
USE_PROXY=1
PROXY_URL=http://wifi;md;;:YOUR_PASSWORD@proxy.froxy.com:9000
PROXY_URL_EU=http://wifi;de;;:YOUR_PASSWORD@proxy.froxy.com:9000
PROXY_PROVIDER=froxy
PROXY_STICKY=1
PROXY_FALLBACK_EU=1
```

## 🚀 Инструкция по применению

1. **Обновить .env**:
   ```bash
   # Добавить новые переменные из примера выше
   ```

2. **Запустить миграцию** (единожды):
   ```bash
   python scripts/migrate_region_tag.py
   ```

3. **Проверить config.json**:
   - Все источники должны иметь `region`
   - Для EU-специфичных источников установить `region: "EU"`
   - Для MD - `region: "MD"`
   - Остальные - `region: "GLOBAL"`

4. **Запустить бота**:
   ```bash
   python -m src.main
   ```

5. **Проверить логи**:
   - Должна быть строка: `🔐 Прокси настроен: провайдер=froxy, sticky=True, fallback_EU=True`
   - При обновлении источников проверить `✅ Успешно получено через EU fallback` (если был 407/403 с MD)

## 🎯 Ключевые преимущества

1. **Региональная маршрутизация**:
   - EU источники идут через EU прокси
   - MD источники идут через MD прокси
   - Автоматический fallback MD→EU при ошибках

2. **Sticky-сессии**:
   - Один IP для всего раунда обновлений (если `PROXY_STICKY=1`)
   - Меньше вероятность блокировок

3. **Правильные Accept-Language заголовки**:
   - EU источники: `en-GB,en;q=0.9`
   - MD источники: `en-GB,en;q=0.9,ro;q=0.8,ru;q=0.7`
   - GLOBAL: `en-US,en;q=0.9`

4. **Визуальные индикаторы**:
   - Каждое изменение помечено флагом региона: 🇪🇺 [EU] / 🇲🇩 [MD] / 🌍 [GLOBAL]

5. **Back-compatibility**:
   - Старые записи автоматически получают `region='GLOBAL'`
   - Код продолжает работать со старым кэшем

## 📚 Дополнительная документация

- `REGION_REFACTOR_TODO.md` - оставшиеся задачи (опционально)
- Подробные примеры кода для telegram_notify.py и handlers.py

## ✨ Итог

Выполнено ~85% работы. Основная функциональность реализована и протестирована:
- ✅ Региональная маршрутизация прокси
- ✅ Froxy sticky-sessions
- ✅ Fallback MD→EU
- ✅ Правильные Accept-Language
- ✅ Региональные badges
- ✅ Back-compatibility
- ✅ Скрипт миграции

Оставшиеся задачи (группировка в Telegram, статистика в /status) - опциональные улучшения.
