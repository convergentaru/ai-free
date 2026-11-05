# Диагностика AI Free на Ubuntu 26.04

В этом файле разделены подтверждённые причины и гипотезы, которые ещё нужно проверить.

## 1. Patchright: Ubuntu 26.04 не распознаётся

### Симптом

Во время первого `npm install`:

```text
ERROR: Patchright does not support chromium on ubuntu26.04-x64
```

### Подтверждённое решение в этой установке

```bash
npm update patchright playwright --ignore-scripts
npm ls patchright playwright playwright-core --depth=2
npm install
```

Результат: Patchright 1.64.3 и Playwright 1.64.1 установились, Chromium/Headless Shell/FFmpeg скачались, postinstall завершился успешно.

После этого дополнительно выполнено:

```bash
sudo npx playwright install-deps chromium
```

## 2. DeepSeek: HTTP 500 в Hermes

### Наблюдаемый симптом

Hermes получал:

```text
HTTP 500: fetch failed
Provider said: HTTP 500: fetch failed
```

В журнале AI Free `~/.ai-free/logs/ai-free.log` последовательность была следующей:

1. `POST /api/v0/chat_session/create` на стороне DeepSeek вернул `statusCode: 200`.
2. После создания сессии AI Free отправил upstream completion request.
3. Node fetch завершился ошибкой `ECONNRESET`.
4. Cause: `Client network socket disconnected before secure TLS connection was established`, порт 443.
5. Локальный `POST /v1/chat/completions` завершился кодом 500.

Это указывает на сетевой сбой при установлении TLS-соединения с upstream, а не на доказанную ошибку формата запроса Hermes. Сам по себе HTTP 500 не доказывает, что сервис DeepSeek целиком недоступен.

### Сведения, собранные на машине

Node.js: v24.21.0.

Переменные окружения Node были пусты:

- HTTP_PROXY — unset
- HTTPS_PROXY — unset
- ALL_PROXY — unset
- NO_PROXY — unset
- нижний регистр тех же переменных — unset

Системный прокси GNOME:

- mode — none
- HTTP host — пусто
- HTTP port — 8080 (значение настройки при отключённом режиме)
- HTTPS host — пусто
- HTTPS port — 0

При проверке локальных портов через ss обнаружен listener `*:7897`. Пользователь назвал используемое proxy-приложение «Калла». Пока не доказано, что порт 7897 — именно HTTP proxy endpoint; до проверки нельзя автоматически задавать его как HTTPS_PROXY.

### Следующая безопасная проверка — ещё не выполнена

После подтверждения, что локальный порт 7897 принимает HTTP proxy, можно проверить его так:

```bash
curl -sS -o /dev/null -w 'HTTP: %{http_code} | TLS: %{time_appconnect}s\\n' \\
  --proxy http://127.0.0.1:7897 \\
  --connect-timeout 10 --max-time 25 \\
  https://chat.deepseek.com
```

HTTP-код сам по себе не означает ошибку прокси: важны результат установления TLS и отсутствие connection reset.

Только если HTTP proxy на 7897 подтверждён, отдельно тестировать запуск Node с соответствующим HTTPS_PROXY и включённой поддержкой proxy env. Не менять системный прокси и не редактировать постоянные конфиги на первом тесте.

### Команда чтения лога

```bash
tail -n 50 ~/.ai-free/logs/ai-free.log
```

Перед публикацией логов проверять, что в них нет токенов, cookies, authorization headers или персональных данных.

## 3. Окно Chrome / Playwright закрывается

В предыдущем логе при попытке входа Qwen был запущен системный Chrome с профилем `~/.qwen-cli/browser-profile`, затем процесс завершился с кодом 0. Playwright выдал:

```text
Target page, context or browser has been closed
```

Это отдельный симптом Qwen-браузера; он не доказывает неисправность DeepSeek.

## 4. ChatGPT: Cloudflare challenge

### Неудачный путь

- ChatGPT виден среди провайдеров AI Free.
- Встроенная панель 🧠 → Браузер → ChatGPT циклически показывала проверку Cloudflare («подтвердите, что вы человек»).
- Нажатие «Синхронизировать» не завершало вход.
- Обычная отметка «подключён» в UI не доказывала, что сессия пригодна для API.
- Однократный импорт JSON с accessToken и sessionToken сообщил об успехе, но проблема Cloudflare сохранялась. Импорт не решил проблему.

### Рабочий путь: отдельный видимый Google Chrome

При остановленном AI Free удалён только каталог ChatGPT-профиля и запущена штатная команда входа:

```bash
rm -rf ~/.chatgpt-cli
cd ~/projects/ai-free
CHATGPT_EMBED_IN_UI=0 npm run login-chatgpt
```

Результат, подтверждённый терминалом:

```text
Открываем окно ChatGPT (chatgpt.com).
Используется обычный Google Chrome, не Playwright Chromium.
✅ Успешный вход в ChatGPT! Окно браузера остаётся открытым.
```

Пользователь сообщил, что прошёл Cloudflare, вошёл в ChatGPT и сессия заработала. Для этой машины следует предпочитать этот отдельный видимый Chrome-путь вместо встроенного окна, которое зацикливало challenge.

### Что означает «окно браузера остаётся открытым»

Команда сама сохраняет авторизацию в `~/.chatgpt-cli/auth.json` и оставляет браузерное окно открытым. Не нужно вручную передавать/переносить cookies или повторно импортировать JSON после успешного входа.

### Безопасность

Не добавлять в GitHub экспортированные токены, cookies или JSON-сессии. Экспорт сессии из DevTools содержит секреты авторизации аккаунта. Такие файлы должны оставаться только локально и удаляться после разового импорта. Если файл с токенами опубликован или передан третьим лицам, следует отозвать соответствующую сессию.

## 5. Parakeet V3

Статус «Not Installed» относится к дополнительному голосовому вводу. Для обычного чата, DeepSeek и Hermes API он не требуется. Отдельная установка Rust и модели Parakeet V3 отложена.

## 6. Чего пока не делать

- Не удалять `~/.deepseek-cli/` — там находится авторизация и общее состояние/чаты AI Free.
- Не удалять `~/.ai-free/` без отдельного решения — там локальная память и skills.
- Не удалять весь проект `~/projects/ai-free` ради ошибки HTTP 500.
- Не считать «Connected» или наличие модели в списке достаточным тестом ChatGPT.
- Не менять настройки Hermes/Unsloth для устранения upstream TLS reset.
