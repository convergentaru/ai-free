# Установка AI Free и подключение Hermes

Проверено на Ubuntu 26.04.1 LTS, Node.js v24.21.0, AI Free 0.4.25 — 5 ноября 2026 года.

## 1. Получение исходников

Исходный проект клонировался из репозитория разработчика:

```bash
mkdir -p ~/projects
git clone https://github.com/Staks-sor/ai-free.git ~/projects/ai-free
```

Рабочая папка: `~/projects/ai-free`.

## 2. Исправление установки Chromium на Ubuntu 26.04

Первый обычный запуск `npm install` завершился ошибкой:

```text
patchright install chromium
ERROR: Patchright does not support chromium on ubuntu26.04-x64
```

Установленные тогда версии:

- Patchright 1.60.2
- Playwright 1.60.0
- playwright-core 1.60.0

В проектном package.json были диапазоны версий с caret, допускающие обновление минорных/патч-релизов. В этой установке сработала последовательность:

```bash
cd ~/projects/ai-free
npm update patchright playwright --ignore-scripts
npm ls patchright playwright playwright-core --depth=2
npm install
```

После обновления фактически установились:

- Patchright 1.64.3
- Playwright 1.64.1
- playwright-core 1.64.1

Повторный `npm install` прошёл успешно. Были скачаны Chrome for Testing 156.0.8078.4, Chromium Headless Shell и FFmpeg. Это — подтверждённое решение именно для зафиксированной среды и версии на дату записи; при будущих обновлениях сначала проверять совместимость новых версий.

## 3. Системные зависимости Chromium

После успешной установки npm выполнено:

```bash
sudo npx playwright install-deps chromium
```

Команда завершилась без ошибки. Она установила недостающие шрифты и X-системные пакеты, в том числе xvfb и xserver-xephyr, и обновила несколько X-серверных пакетов. Дополнительное занятое место по сообщению apt — около 80,9 МБ.

## 4. Первый запуск и меню провайдеров

В package.json доступны отдельные scripts:

- `npm start` — запускает window mode: `node ./bin/deepseek.mjs --window`.
- `npm run welcome` — явно вызывает welcome-меню: `node ./bin/deepseek.mjs --welcome`.
- `npm run check` — проверяет DeepSeek-авторизацию созданием сессии.
- `npm run login-chatgpt` — отдельная команда входа ChatGPT.
- `npm run import-chatgpt -- <file>` — импорт JSON-сессии ChatGPT.

В этой установке `npm start` сразу открыл окно чатов и показал сообщение, что DeepSeek token отсутствует и вход следует выполнить через Settings. Для явного вызова меню провайдеров предусмотрена команда:

```bash
cd ~/projects/ai-free
npm run welcome
```

README проекта описывает welcome-меню для первого запуска, когда ни один провайдер не подключён. На практике важно помнить, что `npm start` жёстко передаёт `--window`, а CLI-код не показывает welcome-меню в window mode. Если нужен именно экран выбора провайдера, используйте явный `npm run welcome`.

Обычный запуск рабочего окна:

```bash
cd ~/projects/ai-free
npm start
```

Окно чатов открылось по адресу `http://127.0.0.1:4317`.

## 5. Проверка авторизации DeepSeek

Выполнялась команда:

```bash
cd ~/projects/ai-free
npm run check
```

Подтверждённый результат:

```text
OK: authenticated, created session <session-id>
```

Сам идентификатор сессии не требуется сохранять в документации. Команда подтверждает, что AI Free смог создать DeepSeek chat session; это не гарантирует, что все последующие upstream-запросы будут проходить без сетевых сбоев.

## 6. Подключение AI Free к Hermes

Подключение добавлялось через:

```bash
hermes model
```

В мастере настройки Custom OpenAI-compatible endpoint были использованы:

| Поле | Значение |
|---|---|
| Base URL | `http://127.0.0.1:4317/v1` |
| API key | Локальный ключ AI Free из его настроек API; секрет не записывать в Git |
| API compatibility | `2. Chat Completions` |
| Model | `deepseek-v4-flash` |
| Context length | Оставлено пустым для auto-detect |
| Display name | `AI Free — DeepSeek` |
| Reasoning effort | `medium` |

Hermes сообщил, что endpoint проверен через `/v1/models`, и отобразил пять моделей:

1. `deepseek-v4-flash`
2. `deepseek-v4-pro`
3. `deepseek-v4-vision`
4. `deepseek-chat`
5. `deepseek-reasoner`

После сохранения Hermes сообщил, что контекст auto-detected как 1M tokens. Это только значение, которое сообщил мастер Hermes; не следует автоматически считать его подтверждённым рабочим контекстным окном модели.

Первый проверочный запрос в Hermes получил ответ от модели `deepseek-v4-flash`. Таким образом, связка была реально проверена запросом, а не только обнаружением списка моделей.

## 7. Порты: не перепутать

В текущем README AI Free описан отдельный OpenAI/Anthropic API на `localhost:4318`, который запускается отдельной командой `npm run api`.

Одновременно window server на `127.0.0.1:4317` в этой установке также обслужил `/v1/models` и `/v1/chat/completions`: Hermes проверил список моделей на порту 4317, а журнал AI Free записал входящие запросы на `/v1/chat/completions` с этого процесса.

Итого:

- `4317` — рабочее окно / window server; именно этот адрес уже проверен в Hermes.
- `4318` — отдельный API mode из README; его самостоятельный запуск здесь отдельно не проверялся.

Не менять Base URL Hermes на 4318, пока отдельный API-сервер действительно не запущен и не проверен.

## 8. Эксплуатационные правила

- AI Free должен быть запущен, пока Hermes использует его API.
- Не закрывать процесс `npm start` во время работы Hermes через порт 4317.
- DeepSeek добавлен в Hermes как отдельный custom provider. Существующий профиль Unsloth/Qwen не удалялся и не заменялся.
- В случае HTTP 500 смотреть `~/.ai-free/logs/ai-free.log`, а не переустанавливать зависимости.
