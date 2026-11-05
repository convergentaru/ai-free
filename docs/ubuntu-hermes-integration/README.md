# AI Free + Hermes на Ubuntu 26.04

Практический журнал установки, настройки и диагностики AI Free 0.4.25 на Ubuntu 26.04.1 LTS и подключения его локального OpenAI-совместимого API к Hermes Agent.

Дата фиксации состояния: **5 ноября 2026 года**.

## Где находится этот проект

- Документация хранится в личном GitHub-репозитории: [convergentaru/ai-free](https://github.com/convergentaru/ai-free).
- Исходный проект разработчика: [Staks-sor/ai-free](https://github.com/Staks-sor/ai-free).
- Локальная установка на машине: `~/projects/ai-free`.
- Эта папка относится к пользовательской документации и не является патчем исходного кода AI Free.

## Среда

| Компонент | Зафиксированное значение |
|---|---|
| ОС | Ubuntu 26.04.1 LTS, Resolute |
| Сеанс рабочего стола | i3 / X11 |
| Node.js | v24.21.0 |
| AI Free | 0.4.25 |
| Локальный каталог | `~/projects/ai-free` |
| DeepSeek UI / window server | `http://127.0.0.1:4317` |

## Текущее состояние

| Направление | Статус на момент записи |
|---|---|
| Установка AI Free | Успешно после обновления Patchright и Playwright |
| Chromium | Установлен; системные зависимости Chromium установлены |
| DeepSeek | Авторизация подтверждена; запрос через Hermes хотя бы один раз успешно ответил |
| Hermes → AI Free | Настроено через OpenAI-compatible Chat Completions |
| Стабильность DeepSeek | Были последующие HTTP 500 с TLS/ECONNRESET до upstream на порту 443; сетевой маршрут ещё исследуется |
| ChatGPT | Появился как провайдер; импорт сессии подтвердился, но рабочая авторизация упиралась в повторяющуюся Cloudflare-проверку |
| Qwen | В логе была ошибка закрытия браузерного контекста; подключение не подтверждено |
| Parakeet V3 | Не установлен; необязательный компонент голосового ввода |
| Claude | Ещё не подключался; отдельная будущая задача |

## Содержание папки

- [SETUP-AND-HERMES.md](SETUP-AND-HERMES.md) — чистая установка, исправление Chromium и настройка Hermes.
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — Patchright, браузер, HTTP 500, прокси и Cloudflare.
- [CURRENT-STATE.md](CURRENT-STATE.md) — контрольная точка, подтверждённые результаты и следующие шаги.

## Важные правила

1. Не помещать в GitHub API-ключи, токены, cookies, файлы auth.json, экспортированные ChatGPT-сессии и содержимое .env.
2. Не менять существующий профиль Unsloth/Qwen в Hermes ради подключения AI Free. AI Free добавляется отдельным custom provider.
3. Не считать сообщение «подключён» подтверждением работающей авторизации. Проверять реальным запросом.
4. Не смешивать порт окна AI Free 4317 и порт отдельного API-процесса 4318.
5. При сетевой ошибке сначала смотреть журнал AI Free; не переустанавливать проект без причины.

## Первичные источники проекта

- [README AI Free](https://github.com/Staks-sor/ai-free/blob/main/README.md)
- [package.json](https://github.com/Staks-sor/ai-free/blob/main/package.json)
- [CLI dispatch](https://github.com/Staks-sor/ai-free/blob/main/src/cli/run.mjs)
- [OpenAI-compatible handler](https://github.com/Staks-sor/ai-free/blob/main/api/openai-handler.mjs)
- [ChatGPT browser login](https://github.com/Staks-sor/ai-free/blob/main/src/providers/chatgpt/browser-login.mjs)
