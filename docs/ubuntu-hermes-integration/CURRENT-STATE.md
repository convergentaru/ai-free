# Контрольная точка: AI Free + Hermes

**Дата:** 2026-11-05  
**Цель:** использовать DeepSeek, ChatGPT и позднее Claude через единый рабочий контур Hermes, не разрушая существующую конфигурацию Unsloth/Qwen.

## Что уже сделано

### AI Free

- Официальный upstream-код скачан в `~/projects/ai-free`.
- Версия проекта — 0.4.25.
- Ошибка установки Chromium на Ubuntu 26.04 исправлена обновлением Patchright/Playwright:
  - Patchright 1.64.3;
  - Playwright 1.64.1;
  - playwright-core 1.64.1.
- `npm install` после обновления завершился успешно.
- Chromium, Headless Shell и FFmpeg скачаны.
- Выполнена установка системных зависимостей Chromium через `sudo npx playwright install-deps chromium`.
- Window server запустился на `http://127.0.0.1:4317`.

### DeepSeek

- Авторизация подтверждена командой `npm run check`.
- AI Free выдал локальный OpenAI-compatible API key; секрет не фиксируется в этом документе.
- Hermes добавил custom endpoint с названием `AI Free — DeepSeek`.
- Протокол — Chat Completions.
- Base URL — `http://127.0.0.1:4317/v1`.
- Модель — `deepseek-v4-flash`.
- Hermes успешно получил первый ответ и сообщил модель `deepseek-v4-flash`.
- В Hermes сохранён отдельный custom provider; профиль Unsloth/Qwen не удалялся.

## Известная нестабильность DeepSeek

Позднее AI Free вернул HTTP 500 при запросах Hermes. Лог показывает upstream TLS reset до установления защищённого соединения на порт 443 (`ECONNRESET`). При этом создание DeepSeek chat session до completion прошло с HTTP 200.

Node.js не видел переменных proxy. Системный proxy GNOME был отключён. Найден слушающий локальный порт 7897, который нужно проверить как HTTP proxy endpoint. Подробности — в TROUBLESHOOTING.md.

## ChatGPT

### Успешная авторизация

- ChatGPT отображается среди провайдеров AI Free.
- Первоначальный встроенный вход через 🧠 → Браузер → ChatGPT зацикливался на Cloudflare challenge.
- Импорт JSON-сессии один раз сообщил об успешном импорте токенов, но последующее использование всё равно показывало Cloudflare challenge. Этот способ не считать подтверждённым рабочим входом.
- Удаление `~/.chatgpt-cli` использовалось для чистого сброса ChatGPT-профиля; DeepSeek и общие чаты не удалялись.
- **Подтверждённый способ:** при остановленном AI Free выполнено:
  
  ```bash
  rm -rf ~/.chatgpt-cli
  cd ~/projects/ai-free
  CHATGPT_EMBED_IN_UI=0 npm run login-chatgpt
  ```

- Эта команда открыла обычный Google Chrome, не встроенный браузер Playwright/Patchright.
- Пользователь вручную прошёл Cloudflare и вход ChatGPT.
- Терминал подтвердил: `✅ Успешный вход в ChatGPT! Окно браузера остаётся открытым.`
- Следовательно, отдельный видимый Chrome с `CHATGPT_EMBED_IN_UI=0` — подтверждённый успешный способ авторизации для этой машины.

### API → Hermes ещё не подключён

Исходный код AI Free содержит:
- отдельный ChatGPT client в `api/openai-handler.mjs`;
- модельный каталог ChatGPT в `packages/core/src/providers/model-catalog.mjs`;
- provider-specific API keys для `deepseek`, `qwen` и `chatgpt`;
- endpoint локального window server `POST /api/settings/openai-key`, который принимает `{"provider":"chatgpt"}` и создаёт отдельный ключ.

В текущем UI Settings → API список кнопок визуально включает DeepSeek и Qwen, но не ChatGPT. Поэтому если кнопка ChatGPT отсутствует, ключ можно создать через локальный endpoint после запуска AI Free. Точный шаг см. в SETUP-AND-HERMES.md.

### Предстоящая проверка ChatGPT

1. Убедиться, что команда входа завершилась и авторизация сохранена локально; не читать и не публиковать токены.
2. Запустить AI Free через `npm start`.
3. Создать отдельный ключ ChatGPT через `POST http://127.0.0.1:4317/api/settings/openai-key` с телом `{"provider":"chatgpt"}`.
4. Добавить в Hermes новый Custom OpenAI-compatible endpoint, используя тот же Base URL `http://127.0.0.1:4317/v1`, но именно ChatGPT API key.
5. Убедиться, что Hermes видит ChatGPT-модели; выбрать модель из фактического ответа `/v1/models` и выполнить тестовый запрос.

Не переиспользовать DeepSeek API key для ChatGPT: ключи разделены по provider ID.

## Следующие задачи

1. Проверить, принимает ли локальный порт 7897 HTTP proxy-запросы; затем направить Node/AI Free через подтверждённый прокси и проверить стабильность DeepSeek.
2. Завершить ChatGPT → Hermes через отдельный API key и подтвердить реальный ответ модели в Hermes.
3. Исследовать Claude отдельно: определить поддерживаемый официальный способ авторизации и API-маршрут. Пока не считать, что AI Free предоставляет Claude как встроенного провайдера.
4. Проверить сохранность существующего Unsloth/Qwen профиля Hermes и зафиксировать итоговые настройки после тестов.

## Секреты и локальные данные

Никогда не коммитить в этот репозиторий:

- `~/.deepseek-cli/auth.json`;
- `~/.chatgpt-cli/auth.json`;
- browser-profile каталоги;
- API keys из AI Free или Hermes .env;
- экспортированные ChatGPT session JSON;
- логи, содержащие cookies, токены или личные данные.

Этот файл — контрольная точка проекта, а не резервная копия секретов или конфигурационных файлов.
