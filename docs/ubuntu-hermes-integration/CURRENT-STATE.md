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

## Известная нестабильность

Позднее AI Free вернул HTTP 500 при запросах Hermes. Лог показывает upstream TLS reset до установления защищённого соединения на порт 443 (`ECONNRESET`). При этом создание DeepSeek chat session до completion прошло с HTTP 200.

Node.js не видел переменных proxy. Системный proxy GNOME был отключён. Найден слушающий локальный порт 7897, который нужно проверить как HTTP proxy endpoint. Подробности — в TROUBLESHOOTING.md.

## ChatGPT

- ChatGPT отображается среди провайдеров AI Free.
- Импорт локального session JSON завершился сообщением об успехе.
- При попытке использовать ChatGPT AI Free просит пройти Cloudflare challenge и синхронизировать сессию.
- Во встроенной панели Cloudflare challenge зацикливается.
- Показанный UI статус «подключён» не является доказательством, что запросы ChatGPT выполняются.
- Проверка через отдельный видимый Chrome ещё не проведена.

### Следующий шаг ChatGPT

AI Free остановить. В отдельном терминале выполнить:

```bash
cd ~/projects/ai-free
CHATGPT_EMBED_IN_UI=0 npm run login-chatgpt
```

Ожидаемое поведение по исходному коду: открыть отдельное видимое окно браузера, вручную пройти challenge/вход, затем дождаться подтверждения сессии. Не запускать одновременно второй процесс AI Free с тем же ChatGPT browser profile.

## Следующие задачи

1. Проверить, принимает ли локальный порт 7897 HTTP proxy-запросы; затем направить Node/AI Free через подтверждённый прокси и проверить стабильность DeepSeek.
2. Завершить ChatGPT login через отдельное видимое окно Chrome; затем проверить реальный запрос ChatGPT в AI Free.
3. Только после реального успешного запроса ChatGPT определить, какой API model ID и отдельный API key AI Free выдаёт для ChatGPT, и добавить второй custom provider в Hermes. Не переиспользовать DeepSeek API key для ChatGPT.
4. Исследовать Claude отдельно: определить поддерживаемый официальный способ авторизации и API-маршрут. Пока не считать, что AI Free предоставляет Claude как встроенного провайдера.
5. Проверить сохранность существующего Unsloth/Qwen профиля Hermes и зафиксировать итоговые настройки после тестов.

## Секреты и локальные данные

Никогда не коммитить в этот репозиторий:

- `~/.deepseek-cli/auth.json`;
- `~/.chatgpt-cli/auth.json`;
- browser-profile каталоги;
- API keys из AI Free или Hermes .env;
- экспортированные ChatGPT session JSON;
- логи, содержащие cookies, токены или личные данные.

Этот файл — контрольная точка проекта, а не резервная копия секретов или конфигурационных файлов.
