## Адаптеры каналов

OpenFang подключается к платформам обмена сообщениями через **40 адаптеров каналов**, что позволяет взаимодействовать с агентами на большинстве популярных платформ. Адаптеры охватывают потребительские мессенджеры, корпоративные решения, социальные сети, сообщества, протоколы для приватности и универсальные вебхуки.

Все адаптеры реализуют общие принципы: аккуратное завершение через `watch::channel`, экспоненциальный бэкофф при падениях соединения, `Zeroizing<String>` для секретов, автоматическое разбиение сообщений по лимитам платформ, переопределения модели/подсказки на уровне канала, политики DM/group, лимиты по пользователям и форматирование вывода (Markdown, TelegramHTML, SlackMrkdwn, PlainText).

## Содержание

- [Все 40 каналов](#all-40-channels)
- [Конфигурация каналов](#channel-configuration)
- [Переопределения каналов](#channel-overrides)
- [Форматтер, лимитер и политики](#formatter-rate-limiter-and-policies)
- [Telegram](#telegram)
- [Discord](#discord)
- [Slack](#slack)
- [WhatsApp](#whatsapp)
- [Feishu / Lark](#feishu--lark)
- [Signal](#signal)
- [Matrix](#matrix)
- [Email](#email)
- [WebChat](#webchat-built-in)
- [Маршрутизация агентов](#agent-routing)
- [Написание собственных адаптеров](#writing-custom-adapters)

---

## Все 40 каналов

Список каналов распределён по категориям: Core, Enterprise, Social, Community, Self-hosted, Privacy, Workplace, Notification, Integration. См. оригинал для полного справочника (таблицы переменных окружения и вариантов).

## Конфигурация каналов

Все конфигурации каналов находятся в `~/.openfang/config.toml` в секции `[channels]`. Каждый канал описывается своей подсекцией, например:

```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
default_agent = "assistant"
allowed_users = ["123456789"]

[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
default_agent = "coder"
```

### Общие поля

- `bot_token_env` / `token_env` — имя переменной окружения с токеном.
- `default_agent` — агент, который получает сообщения по умолчанию.
- `allowed_users` — опциональный список ID пользователей, которым разрешено взаимодействовать.
- `overrides` — секция для переопределений поведения на уровне канала.

---

## Переопределения каналов

Каждый адаптер поддерживает `ChannelOverrides`, позволяющие переопределять модель, системный prompt, политику DM/group, лимиты и формат вывода без изменения манифеста агента. Пример:

```toml
[channels.telegram.overrides]
model = "gemini-2.5-flash"
system_prompt = "Вы — лаконичный помощник в Telegram. Ответы не должны превышать 200 слов."
dm_policy = "respond"
group_policy = "mention_only"
rate_limit_per_user = 10
threading = true
output_format = "telegram_html"
```

---

## Форматтер, лимитер и политики

### Форматтер вывода

Модуль `formatter` (`openfang-channels/src/formatter.rs`) преобразует Markdown-вывод от LLM в нативные форматы платформ:

| OutputFormat | Цель | Примечания |
|-------------|--------|-------|
| `Markdown` | Стандартный Markdown | По умолчанию; передается как есть. |
| `TelegramHtml` | Подмножество HTML Telegram | Преобразует `**жирный**` в `<b>`, `` `код` `` в `<code>` и т.д. |
| `SlackMrkdwn` | Slack mrkdwn | Преобразует `**жирный**` в `*жирный*`, ссылки в `<url\|текст>` и т.д. |
| `PlainText` | Простой текст | Удаляет всю разметку. |

### Лимитер частоты запросов на пользователя

`ChannelRateLimiter` (`openfang-channels/src/rate_limiter.rs`) использует `DashMap` для отслеживания количества сообщений от каждого пользователя. Если в переопределениях канала установлено `rate_limit_per_user`, лимитер применяет ограничение скользящего окна (N сообщений в минуту). При превышении лимита пользователь получает вежливый отказ.

### Политика DM (Личные сообщения)

Управляет тем, как адаптер обрабатывает прямые сообщения:

| DmPolicy | Поведение |
|----------|----------|
| `Respond` | Отвечать на все ЛС (по умолчанию). |
| `AllowedOnly` | Отвечать только пользователям из `allowed_users`. |
| `Ignore` | Молча игнорировать все ЛС. |

### Политика Group (Групповые чаты)

Управляет обработкой сообщений в групповых чатах, каналах и комнатах:

| GroupPolicy | Поведение |
|-------------|----------|
| `All` | Отвечать на каждое сообщение в группе. |
| `MentionOnly` | Отвечать только при @упоминании бота (по умолчанию). |
| `CommandsOnly` | Отвечать только на сообщения, начинающиеся с `/команды`. |
| `Ignore` | Молча игнорировать все групповые сообщения. |

Применение политики происходит в `dispatch_message()` до того, как сообщение попадет в цикл агента. Это означает, что игнорируемые сообщения не потребляют токены LLM.

---

## Telegram

### Предварительные условия

- Токен бота Telegram (получите у [@BotFather](https://t.me/botfather))

### Настройка

1. Откройте Telegram и напишите `@BotFather`.
2. Отправьте `/newbot` и следуйте инструкциям для создания нового бота.
3. Скопируйте токен бота.
4. Установите переменную окружения:

```bash
export TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

5. Добавьте в конфиг:

```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
default_agent = "assistant"
# Опционально: ограничить доступ конкретными ID пользователей
# allowed_users = ["123456789"]

[channels.telegram.overrides]
# Опционально: нативное HTML-форматирование Telegram
# output_format = "telegram_html"
# group_policy = "mention_only"
```

6. Перезапустите демон:

```bash
openfang start
```

### Как это работает

Адаптер Telegram использует технологию long-polling через API `getUpdates`. Он опрашивает сервер каждые несколько секунд с 30-секундным таймаутом ожидания. При ошибках API применяется экспоненциальная задержка (от 1 до 60 секунд). Завершение работы координируется через `watch::channel`.

Сообщения от авторизованных пользователей преобразуются в события `ChannelMessage` и направляются настроенному агенту. Ответы отправляются через API `sendMessage`. Длинные ответы автоматически разбиваются на несколько сообщений, чтобы соответствовать лимиту Telegram в 4096 символов, используя общую утилиту `split_message()`.

### Идентификация пользователя (`telegram_user_id`)

Каждое входящее сообщение Telegram содержит числовой ID отправителя в поле `message.metadata["telegram_user_id"]` в виде строки. Это стабильный, постоянный идентификатор из `message.from.id`.

Отображаемые имена (display names) не уникальны и могут меняться, поэтому агенты, которым требуется детерминированное поведение для каждого пользователя (RBAC, персональные рабочие пространства), должны ориентироваться на `telegram_user_id`, а не на имя отправителя.

Мост также вставляет ID в префикс промпта, если не установлен `sender_email`:

```
[From: Alena (tg_id:554772934)] Hello!
```

---

## Discord

### Предварительные условия

- Приложение и бот Discord (создайте в [Discord Developer Portal](https://discord.com/developers/applications))

### Настройка

1. Перейдите в [Discord Developer Portal](https://discord.com/developers/applications).
2. Нажмите "New Application" и дайте ему имя.
3. Перейдите в раздел **Bot** и нажмите "Add Bot".
4. Скопируйте токен бота.
5. В разделе **Privileged Gateway Intents** включите:
   - **Message Content Intent** (необходимо для чтения текста сообщений)
6. Перейдите в **OAuth2 > URL Generator**:
   - Выберите scopes: `bot`
   - Выберите permissions: `Send Messages`, `Read Message History`
   - Скопируйте сгенерированный URL и откройте его в браузере, чтобы пригласить бота на свой сервер.
7. Установите переменную окружения:

```bash
export DISCORD_BOT_TOKEN=MTIzNDU2Nzg5.ABCDEF.ghijklmnop
```

8. Добавьте в конфиг:

```toml
[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
default_agent = "coder"
```

9. Перезапустите демон.

### Как это работает

Адаптер Discord подключается к Discord Gateway через WebSocket (v10). Он слушает события `MESSAGE_CREATE` и пересылает сообщения агенту. Ответы отправляются через REST API эндпоинт `channels/{id}/messages`. Адаптер автоматически обрабатывает переподключения, сердцебиение (heartbeating) и возобновление сессий.

---

## Slack

### Предварительные условия

- Приложение Slack с включенным Socket Mode

### Настройка

1. Перейдите в [Slack API](https://api.slack.com/apps) и нажмите "Create New App" > "From Scratch".
2. Включите **Socket Mode** (Settings > Socket Mode):
   - Сгенерируйте App-Level Token с разрешением `connections:write`.
   - Скопируйте токен (`xapp-...`).
3. Перейдите в **OAuth & Permissions** и добавьте Bot Token Scopes:
   - `chat:write`
   - `app_mentions:read`
   - `im:history`
   - `im:read`
   - `im:write`
4. Установите приложение в ваше рабочее пространство (workspace).
5. Скопируйте Bot User OAuth Token (`xoxb-...`).
6. Установите переменные окружения:

```bash
export SLACK_APP_TOKEN=xapp-1-...
export SLACK_BOT_TOKEN=xoxb-...
```

7. Добавьте в конфиг:

```toml
[channels.slack]
bot_token_env = "SLACK_BOT_TOKEN"
app_token_env = "SLACK_APP_TOKEN"
default_agent = "ops"
```

8. Перезапустите демон.

### Как это работает

Адаптер Slack использует Socket Mode, который устанавливает WebSocket-соединение с серверами Slack. Это избавляет от необходимости иметь публичный URL для вебхука. Адаптер получает события (упоминания приложения, личные сообщения) и направляет их агенту. Ответы публикуются через Web API `chat.postMessage`. При включении `threading = true` ответы отправляются в тред сообщения через `thread_ts`.

---

## WhatsApp

### Предварительные условия

- Аккаунт Meta Business с доступом к WhatsApp Cloud API

### Настройка

1. Перейдите на [Meta for Developers](https://developers.facebook.com/).
2. Создайте Business App.
3. Добавьте продукт WhatsApp.
4. Настройте тестовый номер телефона (или используйте рабочий).
5. Скопируйте:
   - Phone Number ID
   - Permanent Access Token
   - Придумайте Verify Token (любая строка)
6. Установите переменные окружения:

```bash
export WA_PHONE_ID=123456789012345
export WA_ACCESS_TOKEN=EAABs...
export WA_VERIFY_TOKEN=my-secret-verify-token
```

7. Добавьте в конфиг:

```toml
[channels.whatsapp]
mode = "cloud_api"
phone_number_id_env = "WA_PHONE_ID"
access_token_env = "WA_ACCESS_TOKEN"
verify_token_env = "WA_VERIFY_TOKEN"
webhook_port = 8443
default_agent = "assistant"
```

8. Настройте вебхук в панели Meta, указав публичный URL вашего сервера:
   - URL: `https://your-domain.com:8443/webhook/whatsapp`
   - Verify Token: значение, выбранное выше
   - Подписка на: `messages`

### Как это работает

Адаптер WhatsApp запускает HTTP-сервер (на порту `webhook_port`), который принимает входящие вебхуки от WhatsApp Cloud API. Он обрабатывает верификацию вебхука (GET) и прием сообщений (POST). Ответы отправляются через эндпоинт `messages` Cloud API.

---

## Feishu / Lark

### Предварительные условия

- Приложение Feishu/Lark, созданное на [open.feishu.cn](https://open.feishu.cn/)
- App ID и App Secret

### Настройка

1. Создайте кастомное приложение на Feishu Open Platform.
2. Включите подписку на события IM-сообщений.
3. Установите переменную окружения:

```bash
export FEISHU_APP_SECRET=cli_xxx_secret
```

4. Добавьте в конфиг (по умолчанию режим `websocket`):

```toml
[channels.feishu]
app_id = "cli_xxx"
app_secret_env = "FEISHU_APP_SECRET"
mode = "websocket"
default_agent = "assistant"
```

5. Перезапустите демон.

### Как это работает

- **Режим websocket**: OpenFang получает эндпоинт от Feishu и принимает события через постоянное соединение (публичный вебхук не требуется).
- **Режим webhook**: OpenFang запускает HTTP-сервер для приема push-событий от Feishu.
- В обоих режимах исходящие сообщения отправляются через Feishu OpenAPI HTTP `im/v1/messages`.

---

## Signal

### Предварительные условия

- Установленный signal-cli, привязанный к номеру телефона

### Настройка

1. Установите [signal-cli](https://github.com/AsamK/signal-cli).
2. Зарегистрируйте или привяжите номер телефона.
3. Добавьте в конфиг:

```toml
[channels.signal]
signal_cli_path = "/usr/local/bin/signal-cli"
phone_number = "+1234567890"
default_agent = "assistant"
```

4. Перезапустите демон.

### Как это работает

Адаптер Signal запускает `signal-cli` как подпроцесс в режиме демона и взаимодействует через JSON-RPC. Входящие сообщения считываются из потока вывода `signal-cli` и направляются агенту.

---

## Matrix

### Предварительные условия

- Аккаунт на homeserver Matrix и токен доступа

### Настройка

1. Создайте аккаунт бота на вашем сервере Matrix.
2. Сгенерируйте токен доступа (access token).
3. Установите переменную окружения:

```bash
export MATRIX_TOKEN=syt_...
```

4. Добавьте в конфиг:

```toml
[channels.matrix]
homeserver_url = "https://matrix.org"
access_token_env = "MATRIX_TOKEN"
user_id = "@openfang-bot:matrix.org"
default_agent = "assistant"
```

5. Пригласите бота в комнаты, которые он должен мониторить.
6. Перезапустите демон.

### Как это работает

Адаптер Matrix использует Matrix Client-Server API. Он синхронизируется с сервером с помощью long-polling (`/sync`) и обрабатывает новые сообщения из комнат. Ответы отправляются через эндпоинт `/rooms/{roomId}/send`.

---

## Email

### Предварительные условия

- Аккаунт электронной почты с доступом по IMAP и SMTP

### Настройка

1. Для Gmail создайте [пароль приложения](https://myaccount.google.com/apppasswords).
2. Установите переменную окружения:

```bash
export EMAIL_PASSWORD=abcd-efgh-ijkl-mnop
```

3. Добавьте в конфиг:

```toml
[channels.email]
imap_host = "imap.gmail.com"
imap_port = 993
smtp_host = "smtp.gmail.com"
smtp_port = 587
username = "you@gmail.com"
password_env = "EMAIL_PASSWORD"
poll_interval = 30
default_agent = "email-assistant"
```

4. Перезапустите демон.

### Как это работает

Адаптер email опрашивает входящий ящик IMAP с заданным интервалом. Новые письма парсятся (тема + тело) и направляются агенту. Ответы отправляются через SMTP, сохраняя тему письма для поддержки цепочек сообщений.

---

## WebChat (Встроенный)

Интерфейс WebChat встроен в демон и не требует настройки. Когда демон запущен, он доступен по адресу:

```
http://127.0.0.1:4200/
```

Особенности:
- Чат в реальном времени через WebSocket.
- Потоковые ответы (стриминг текста).
- Выбор агента среди запущенных.
- Отображение использования токенов.
- На localhost аутентификация не требуется (защищено CORS).

---

## Маршрутизация агентов

`AgentRouter` определяет, какой агент получит входящее сообщение. Логика маршрутизации такова:

1. **Привязки (Bindings)** — от более специфичных к общим. Декларативные правила `[[bindings]]` в `config.toml` сопоставляют атрибуты сообщения (канал, channel_id, peer_id, guild_id, account_id, роли) с агентами.
2. **Значение по умолчанию для канала**: Поле `default_agent` в конфиге канала.
3. **Привязка пользователь-агент**: Если пользователь ранее был ассоциирован с конкретным агентом (через команды или конфиг).
4. **Префикс команды**: Пользователи могут переключить агента, отправив команду `/agent coder`. Последующие сообщения будут направлены агенту "coder".
5. **Резерв (Fallback)**: Если ни одно правило не подошло, сообщение идет первому доступному агенту.

### Привязки (Bindings)

Привязка состоит из `agent` (цель) и `match_rule` (критерии). Все непустые поля в правиле должны совпадать.

```toml
# Направить конкретный канал Discord выделенному агенту.
[[bindings]]
agent = "researcher-medical"
match_rule = { channel = "discord", channel_id = "1234567890" }

# Общее правило для того же пользователя на любом другом канале.
[[bindings]]
agent = "assistant"
match_rule = { channel = "discord", peer_id = "user_discord_id" }
```

**`peer_id` vs `channel_id`** — их легко перепутать, но разница важна:
- `peer_id` соответствует **пользователю** (Discord user ID, Slack user ID и т.д.).
- `channel_id` соответствует **каналу/беседе** (текстовый канал Discord, чат Telegram).

Используйте `peer_id` для "сообщений от этого человека". Используйте `channel_id` для "сообщений в этой комнате".

---

## Написание собственных адаптеров

Чтобы добавить поддержку новой платформы обмена сообщениями, реализуйте трейт `ChannelAdapter`, определенный в `crates/openfang-channels/src/types.rs`.

### Трейт ChannelAdapter

```rust
pub trait ChannelAdapter: Send + Sync {
    fn name(&self) -> &str;
    fn channel_type(&self) -> ChannelType;
    async fn start(&self) -> Result<Pin<Box<dyn Stream<Item = ChannelMessage> + Send>>, Box<dyn std::error::Error>>;
    async fn send(&self, user: &ChannelUser, content: ChannelContent) -> Result<(), Box<dyn std::error::Error>>;
    async fn stop(&self) -> Result<(), Box<dyn std::error::Error>>;
    // ... дополнительные методы для типизации, статуса и тредов
}
```

### Основные этапы:

1. **Определите адаптер**: Создайте новый файл в `crates/openfang-channels/src/`. Используйте `Zeroizing<String>` для секретов и `watch::Receiver<bool>` для управления завершением.
2. **Зарегистрируйте модуль**: Добавьте его в `lib.rs` крейта каналов.
3. **Подключите к мосту**: В `crates/openfang-api/src/channel_bridge.rs` добавьте логику инициализации вашего адаптера.
4. **Добавьте поддержку конфига**: В `openfang-types` добавьте структуру конфигурации для новой платформы.
5. **Добавьте мастер настройки CLI**: В `crates/openfang-cli/src/main.rs` добавьте пошаговую инструкцию для вашей платформы.
6. **Протестируйте**: Напишите интеграционные тесты, используя `ChannelMessage` для симуляции входящих данных.
