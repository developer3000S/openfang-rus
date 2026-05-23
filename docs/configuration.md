# Справочник по конфигурации OpenFang

Полное руководство по файлу `config.toml`, охватывающее все настраиваемые поля OpenFang Agent OS.

---

## Содержание

- [Обзор](#overview)
- [Минимальная конфигурация](#minimal-configuration)
- [Полный пример](#full-example)
- [Справочник по разделам](#section-reference)
  - [Поля верхнего уровня](#top-level-fields)
  - [Открытие доступа к панели управления](#exposing-the-dashboard)
  - [\[default\_model\]](#default_model)
  - [\[memory\]](#memory)
  - [\[network\]](#network)
  - [\[web\]](#web)
  - [\[channels\]](#channels)
  - [\[\[mcp\_servers\]\]](#mcp_servers)
  - [\[a2a\]](#a2a)
  - [\[\[fallback\_providers\]\]](#fallback_providers)
  - [\[\[users\]\]](#users)
  - [Переопределения для каналов](#channel-overrides)
- [Переменные окружения](#environment-variables)
- [Валидация](#validation)

---

<a name="overview"></a>
## Обзор

OpenFang считывает свою конфигурацию из одного TOML-файла:

```
~/.openfang/config.toml
```

В Windows `~` разрешается в `C:\Users\<username>`. Если домашнюю директорию определить не удается, в качестве резервного варианта используется системная временная директория.

Домашняя директория определяется со следующим приоритетом:

1. Переменная окружения `OPENFANG_HOME` (например, `OPENFANG_HOME=/data` в официальном Docker-образе).
2. `~/.openfang` (по умолчанию).

Таким образом, внутри Docker-контейнера файл конфигурации должен находиться по пути `/data/config.toml` (так как в образе установлено `ENV OPENFANG_HOME=/data`). Размещение его в любом другом месте (например, `/opt/openfang/config.toml`) будет проигнорировано.

**Ключевые особенности:**

- Каждая структура в конфигурации использует `#[serde(default)]`, что означает, что **все поля являются необязательными**. Пропущенные поля получают значения по умолчанию.
- Разделы каналов (`[channels.telegram]`, `[channels.discord]` и т. д.) являются типами `Option<T>` — при их отсутствии адаптер канала **выключен**. Наличие заголовка раздела (даже пустого) включает адаптер с настройками по умолчанию.
- Секреты **никогда не хранятся в config.toml** напрямую. Вместо этого такие поля, как `api_key_env` и `bot_token_env`, содержат **имя** переменной окружения, в которой находится фактический секрет. Это предотвращает случайную утечку данных при использовании систем контроля версий.
- Конфиденциальные поля (`api_key`, `shared_secret`) автоматически скрываются в отладочном выводе и логах.

---

<a name="minimal-configuration"></a>
## Минимальная конфигурация

Для простейшей работающей конфигурации достаточно установить API-ключ провайдера LLM в переменной окружения. Если файл конфигурации отсутствует, OpenFang запускается с Anthropic в качестве провайдера по умолчанию:

```toml
# ~/.openfang/config.toml
# Минимальный вариант: просто переопределите модель, если хотите использовать не те, что по умолчанию.
# Установите ANTHROPIC_API_KEY в вашем окружении.

[default_model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
api_key_env = "ANTHROPIC_API_KEY"
```

Или для использования локального экземпляра Ollama без API-ключа:

```toml
[default_model]
provider = "ollama"
model = "llama3.2:latest"
base_url = "http://localhost:11434"
api_key_env = ""
```

---

<a name="full-example"></a>
## Полный пример

```toml
# ============================================================
# OpenFang Agent OS -- Полный справочник по конфигурации
# ============================================================

# --- Поля верхнего уровня ---
home_dir = "~/.openfang"             # Домашняя директория OpenFang
data_dir = "~/.openfang/data"        # Базы данных SQLite и файлы данных
log_level = "info"                   # trace | debug | info | warn | error
api_listen = "127.0.0.1:50051"      # Адрес для привязки HTTP/WS API
network_enabled = false              # Включить P2P сеть OFP
api_key = ""                         # API Bearer токен (пусто = без аутентификации)
mode = "default"                     # stable | default | dev
language = "ru"                      # Локаль для CLI/сообщений
usage_footer = "full"                # off | tokens | cost | full

# --- Провайдер LLM по умолчанию ---
[default_model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
api_key_env = "ANTHROPIC_API_KEY"
# base_url = "https://api.anthropic.com"  # Опциональное переопределение

# --- Резервные провайдеры (Fallback) ---
[[fallback_providers]]
provider = "ollama"
model = "llama3.2:latest"
api_key_env = ""
# base_url = "http://localhost:11434"  # Использует значение по умолчанию из каталога, если опущено

[[fallback_providers]]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"

# --- Память ---
[memory]
# sqlite_path = "~/.openfang/data/openfang.db"  # Автоматически определяется, если опущено
embedding_model = "all-MiniLM-L6-v2"
consolidation_threshold = 10000
decay_rate = 0.1

# --- Сеть (Протокол OFP) ---
[network]
listen_addresses = ["/ip4/0.0.0.0/tcp/0"]
bootstrap_peers = []
mdns_enabled = true
max_peers = 50
shared_secret = ""                   # Обязательно, если network_enabled = true

# --- Веб-инструменты ---
[web]
search_provider = "auto"             # auto | brave | tavily | perplexity | duck_duck_go
cache_ttl_minutes = 15

[web.brave]
api_key_env = "BRAVE_API_KEY"
max_results = 5
country = ""
search_lang = ""
freshness = ""

[web.tavily]
api_key_env = "TAVILY_API_KEY"
search_depth = "basic"               # basic | advanced
max_results = 5
include_answer = true

[web.perplexity]
api_key_env = "PERPLEXITY_API_KEY"
model = "sonar"

[web.fetch]
max_chars = 50000
max_response_bytes = 10485760        # 10 MB
timeout_secs = 30
readability = true

# --- MCP серверы ---
[[mcp_servers]]
name = "filesystem"
timeout_secs = 30
env = []
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]

[[mcp_servers]]
name = "remote-tools"
timeout_secs = 60
env = ["REMOTE_API_KEY"]
[mcp_servers.transport]
type = "sse"
url = "https://mcp.example.com/events"

# --- Протокол A2A ---
[a2a]
enabled = false
listen_path = "/a2a"

[[a2a.external_agents]]
name = "research-agent"
url = "https://agent.example.com/.well-known/agent.json"

# --- Пользователи RBAC ---
[[users]]
name = "Alice"
role = "owner"                       # owner | admin | user | viewer
api_key_hash = ""
[users.channel_bindings]
telegram = "123456"
discord = "987654321"

[[users]]
name = "Bob"
role = "user"
[users.channel_bindings]
slack = "U0123ABCDEF"

# --- Адаптеры каналов ---
# (См. раздел "Каналы" ниже для всех 40 адаптеров)

[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
allowed_users = []
# default_agent = "assistant"
poll_interval_secs = 1

[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
allowed_guilds = []
intents = 33280

[channels.slack]
app_token_env = "SLACK_APP_TOKEN"
bot_token_env = "SLACK_BOT_TOKEN"
allowed_channels = []
```

---

<a name="section-reference"></a>
## Справочник по разделам

<a name="top-level-fields"></a>
### Поля верхнего уровня

Эти поля находятся в корне `config.toml` (не внутри какой-либо секции `[section]`).

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `home_dir` | путь | `~/.openfang` | Домашняя директория OpenFang. Хранит конфигурацию, агентов, навыки. |
| `data_dir` | путь | `~/.openfang/data` | Директория для баз данных SQLite и постоянных данных. |
| `log_level` | строка | `"info"` | Детализация логов. Одно из: `trace`, `debug`, `info`, `warn`, `error`. |
| `api_listen` | строка | `"127.0.0.1:50051"` | Адрес привязки для сервера API (HTTP/WebSocket/SSE). Используйте `0.0.0.0:<порт>`, чтобы принимать подключения извне (LAN, Docker, удаленные клиенты). См. раздел [Открытие доступа к панели управления](#exposing-the-dashboard) ниже. Может быть переопределено переменной окружения `OPENFANG_LISTEN`. |
| `network_enabled` | bool | `false` | Включить уровень P2P сети OFP. |
| `api_key` | строка | `""` (пусто) | Ключ аутентификации API. Если установлен, все эндпоинты, кроме `/api/health`, требуют заголовок `Authorization: Bearer <key>`. Пустое значение означает отсутствие аутентификации (только для локальной разработки). |
| `mode` | строка | `"default"` | Режим работы ядра. См. ниже. |
| `language` | строка | `"en"` | Код языка/локали для вывода CLI и системных сообщений. |
| `usage_footer` | строка | `"full"` | Управляет информацией об использовании, добавляемой к ответам. См. ниже. |

**Значения `mode`:**

| Значение | Поведение |
|-------|----------|
| `stable` | Консервативный: без автообновлений, зафиксированные модели, замороженный реестр навыков. Использует `FallbackDriver`. |
| `default` | Сбалансированный: стандартная работа. |
| `dev` | Разработчик: включены экспериментальные функции. |

**Значения `usage_footer`:**

| Значение | Поведение |
|-------|----------|
| `off` | Информация об использовании не отображается. |
| `tokens` | Показывать только количество токенов. |
| `cost` | Показывать только оценочную стоимость. |
| `full` | Показывать и токены, и стоимость (по умолчанию). |

---

<a name="exposing-the-dashboard"></a>
### Открытие доступа к панели управления

По умолчанию OpenFang привязывает API и панель управления к `127.0.0.1` (только loopback), поэтому демон недоступен ниоткуда, кроме локальной машины. Чтобы принимать подключения из вашей локальной сети, хоста Docker или от удаленного клиента, вы должны явно разрешить привязку к внешним интерфейсам.

**Два способа изменить адрес привязки:**

1. Отредактируйте `config.toml`:

   ```toml
   api_listen = "0.0.0.0:4200"
   ```

2. Или установите переменную окружения `OPENFANG_LISTEN` (имеет приоритет над `config.toml`):

   ```bash
   export OPENFANG_LISTEN=0.0.0.0:4200
   ```

   Использование переменной окружения — рекомендуемый путь для Docker, так как это не требует монтирования файла конфигурации.

**Пример для Docker.** Официальный образ устанавливает `OPENFANG_HOME=/data` и открывает порт `4200`. Простейшая полная настройка:

```yaml
services:
  openfang:
    image: ghcr.io/rightnow-ai/openfang:latest
    ports:
      - "4200:4200"
    volumes:
      - openfang-data:/data
    environment:
      - OPENFANG_LISTEN=0.0.0.0:4200          # обязательно: привязка ко всем интерфейсам внутри контейнера
      - OPENFANG_API_KEY=${OPENFANG_API_KEY}  # настоятельно рекомендуется при открытии доступа
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY:-}
volumes:
  openfang-data:
```

Если вы предпочитаете монтировать `config.toml`, файл должен находиться по пути `/data/config.toml` внутри контейнера (из-за `OPENFANG_HOME=/data`). Файл по пути `/opt/openfang/config.toml` или любому другому не будет подхвачен. Порт в `api_listen` также должен соответствовать порту, указанному в `ports:` — в примере конфигурации `openfang.toml.example` указан порт `50051` для безопасности; измените его на `4200` (или любой другой, который вы пробрасываете) при запуске в Docker.

**Предупреждение по безопасности.** Как только вы выполняете привязку к адресу, отличному от loopback, любой, кому доступен этот адрес, сможет взаимодействовать с API. Промежуточное ПО OpenFang применяет политику "закрыто по умолчанию" для защищенных маршрутов:

- Если `api_key` пуст И аутентификация панели управления выключена И адрес привязки — не loopback, защищенные маршруты будут отклонять внешние запросы с ошибкой `401 Unauthorized`.
- Небольшой набор публичных маршрутов (проверка здоровья, статические ресурсы, колбэк OAuth) остаются доступными, чтобы панель управления могла отобразить страницу входа. Они не раскрывают данные агентов и не принимают команды.
- Чтобы запустить систему с полным открытым доступом (не рекомендуется), установите `OPENFANG_ALLOW_NO_AUTH=1`. Об этом будет выведено заметное предупреждение в логах.

Поддерживаемые способы безопасного открытия доступа к панели:

- Установите `api_key = "..."` в `config.toml` (или `OPENFANG_API_KEY=...`) и отправляйте заголовок `Authorization: Bearer <key>` в каждом запросе.
- Или включите раздел [`[auth]`](#auth), чтобы требовать логин и пароль в интерфейсе панели управления.
- Или оставьте `api_listen` на `127.0.0.1` и подключайтесь к панели через SSH-туннель или обратный прокси-сервер, который берет аутентификацию на себя.

---

<a name="default_model"></a>
### `[default_model]`

Настраивает основного провайдера LLM, который используется, если агенты не указывают свою собственную модель.

```toml
[default_model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
api_key_env = "ANTHROPIC_API_KEY"
# base_url = "https://api.anthropic.com"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `provider` | строка | `"anthropic"` | Имя провайдера. Поддерживаются: `anthropic`, `gemini`, `openai`, `groq`, `openrouter`, `deepseek`, `together`, `mistral`, `fireworks`, `ollama`, `vllm`, `lmstudio`, `perplexity`, `cohere`, `ai21`, `cerebras`, `sambanova`, `huggingface`, `xai`, `replicate`. |
| `model` | строка | `"claude-sonnet-4-20250514"` | Идентификатор модели. Алиасы, такие как `sonnet`, `haiku`, `gpt-4o`, `gemini-flash`, разрешаются через каталог моделей. |
| `api_key_env` | строка | `"ANTHROPIC_API_KEY"` | Имя переменной окружения, содержащей API-ключ. Сам ключ считывается из этой переменной во время работы и никогда не хранится в конфиге. |
| `base_url` | строка или null | `null` | Переопределение базового URL API. Полезно для прокси или self-hosted эндпоинтов. Если `null`, используется URL провайдера по умолчанию. |

---

<a name="memory"></a>
### `[memory]`

Настраивает подсистему памяти на базе SQLite, включая векторные эмбеддинги и затухание памяти.

```toml
[memory]
# sqlite_path = "/custom/path/openfang.db"
embedding_model = "all-MiniLM-L6-v2"
consolidation_threshold = 10000
decay_rate = 0.1
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `sqlite_path` | путь или null | `null` | Явный путь к файлу базы данных SQLite. Если `null`, используется `{data_dir}/openfang.db`. |
| `embedding_model` | строка | `"all-MiniLM-L6-v2"` | Имя модели, используемой для генерации векторных эмбеддингов для семантического поиска по памяти. |
| `consolidation_threshold` | u64 | `10000` | Количество сохраненных записей памяти, после которого запускается автоматическая консолидация для объединения и очистки старых записей. |
| `decay_rate` | f32 | `0.1` | Скорость затухания уверенности в памяти. `0.0` = без затухания (память никогда не стирается), `1.0` = агрессивное затухание. Значения от 0.0 до 1.0. |

---

<a name="network"></a>
### `[network]`

Настраивает уровень P2P сети OFP (OpenFang Protocol) с взаимной аутентификацией HMAC-SHA256.

```toml
[network]
listen_addresses = ["/ip4/0.0.0.0/tcp/0"]
bootstrap_peers = []
mdns_enabled = true
max_peers = 50
shared_secret = "my-cluster-secret"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `listen_addresses` | список строк | `["/ip4/0.0.0.0/tcp/0"]` | Мультиадреса libp2p для прослушивания. Порт `0` означает автоматическое назначение. |
| `bootstrap_peers` | список строк | `[]` | Мультиадреса бутстрап-узлов для поиска через DHT. |
| `mdns_enabled` | bool | `true` | Включить mDNS для автоматического обнаружения узлов в локальной сети. |
| `max_peers` | u32 | `50` | Максимальное количество одновременно подключенных узлов. |
| `shared_secret` | строка | `""` (пусто) | Общий секрет для взаимной аутентификации OFP HMAC-SHA256. **Обязательно**, если `network_enabled = true`. Обе стороны должны использовать один и тот же секрет. Скрывается в логах. |

---

<a name="auth"></a>
### `[auth]`

Настраивает вход в панель управления с аутентификацией по логину/паролю. По умолчанию выключено.

```toml
[auth]
enabled = true
username = "admin"
password_hash = "$argon2id$v=19$m=19456,t=2,p=1$..."  # сгенерируйте с помощью: openfang auth hash-password
session_ttl_hours = 168
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `enabled` | bool | `false` | Включить аутентификацию по логину/паролю для панели управления. |
| `username` | строка | `"admin"` | Имя пользователя администратора. |
| `password_hash` | строка | `""` (пусто) | Хеш пароля Argon2id в формате PHC. Сгенерируйте с помощью `openfang auth hash-password`. |
| `session_ttl_hours` | u64 | `168` (7 дней) | Время жизни токена сессии в часах. |

**Генерация хеша пароля:**

```bash
openfang auth hash-password
```

Команда запросит пароль и выведет строку PHC Argon2id для вставки в `config.toml`.

> **Важное изменение (v0.5.0):** Хеши паролей должны быть в формате Argon2id. Старые шестнадцатеричные хеши SHA256 из версий до v0.5.0 больше не принимаются. Запустите `openfang auth hash-password` повторно, чтобы создать новый хеш.

---

<a name="web"></a>
### `[web]`

Настраивает возможности веб-поиска и получения данных, используемые инструментами агентов.

```toml
[web]
search_provider = "auto"
cache_ttl_minutes = 15
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `search_provider` | строка | `"auto"` | Какую поисковую систему использовать. См. значения ниже. |
| `cache_ttl_minutes` | u64 | `15` | Длительность кэширования результатов поиска/получения в минутах. `0` = кэширование отключено. |

**Значения `search_provider`:**

| Значение | Описание |
|-------|-------------|
| `auto` | Каскадный резерв: пробует Tavily, затем Brave, затем Perplexity, затем SearXNG, затем DuckDuckGo, в зависимости от доступных ключей API/конфигураций. |
| `brave` | Brave Search API. Требуется `BRAVE_API_KEY`. |
| `tavily` | Tavily AI-native search. Требуется `TAVILY_API_KEY`. |
| `perplexity` | Perplexity AI search. Требуется `PERPLEXITY_API_KEY`. |
| `searxng` | Агрегатор поисковых систем (self-hosted). Ключ API не требуется, просто укажите адрес вашего экземпляра SearXNG. |
| `duck_duck_go` | Скрапинг HTML DuckDuckGo. Ключ API не требуется. |

#### `[web.brave]`

```toml
[web.brave]
api_key_env = "BRAVE_API_KEY"
max_results = 5
country = ""
search_lang = ""
freshness = ""
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `api_key_env` | строка | `"BRAVE_API_KEY"` | Имя переменной окружения с API-ключом Brave Search. |
| `max_results` | usize | `5` | Максимальное количество результатов поиска. |
| `country` | строка | `""` | Код страны для локализованных результатов (например, `"US"`, `"RU"`). Пусто = без фильтра. |
| `search_lang` | строка | `""` | Код языка (например, `"en"`, `"ru"`). Пусто = без фильтра. |
| `freshness` | строка | `""` | Фильтр свежести. `"pd"` = за последние сутки, `"pw"` = за неделю, `"pm"` = за месяц. Пусто = без фильтра. |

#### `[web.tavily]`

```toml
[web.tavily]
api_key_env = "TAVILY_API_KEY"
search_depth = "basic"
max_results = 5
include_answer = true
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `api_key_env` | строка | `"TAVILY_API_KEY"` | Имя переменной окружения с API-ключом Tavily. |
| `search_depth` | строка | `"basic"` | Глубина поиска: `"basic"` для быстрых результатов, `"advanced"` для глубокого анализа. |
| `max_results` | usize | `5` | Максимальное количество результатов поиска. |
| `include_answer` | bool | `true` | Включать ли в результаты краткий ответ, сгенерированный ИИ Tavily. |

#### `[web.perplexity]`

```toml
[web.perplexity]
api_key_env = "PERPLEXITY_API_KEY"
model = "sonar"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `api_key_env` | строка | `"PERPLEXITY_API_KEY"` | Имя переменной окружения с API-ключом Perplexity. |
| `model` | строка | `"sonar"` | Модель Perplexity для поисковых запросов. |

#### `[web.searxng]`

**SearXNG** — self-hosted агрегатор поисковых систем. Ключ API не требуется. Поддерживает более 30 категорий поиска и пагинацию.

```toml
[web.searxng]
url = "https://searxng.example.com"    # URL экземпляра SearXNG (обязательно)
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `url` | строка | (обязательно) | Полный URL вашего экземпляра SearXNG (например, `https://searxng.example.com`). Должен быть доступен. |

#### `[web.fetch]`

```toml
[web.fetch]
max_chars = 50000
max_response_bytes = 10485760
timeout_secs = 30
readability = true
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `max_chars` | usize | `50000` | Максимальное количество символов в полученном контенте. Контент сверх этого лимита обрезается. |
| `max_response_bytes` | usize | `10485760` (10 MB) | Максимальный размер тела HTTP-ответа в байтах. |
| `timeout_secs` | u64 | `30` | Таймаут HTTP-запроса в секундах. |
| `readability` | bool | `true` | Включить извлечение контента HTML-to-Markdown (режим чтения). Если `true`, полученный HTML преобразуется в чистый Markdown. |

---

<a name="channels"></a>
### `[channels]`

Все 40 адаптеров каналов настраиваются в разделе `[channels.<name>]`. Каждое поле канала имеет тип `Option<T>` — отсутствие раздела полностью отключает адаптер. Наличие заголовка (даже пустого) включает его со значениями по умолчанию.

Конфигурация каждого канала включает поле `default_agent` (необязательное имя агента для маршрутизации сообщений) и подтаблицу `overrides` (см. [Переопределения для каналов](#channel-overrides)).

#### `[channels.telegram]`

```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
allowed_users = []
# default_agent = "assistant"
poll_interval_secs = 1
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `bot_token_env` | строка | `"TELEGRAM_BOT_TOKEN"` | Переменная окружения с токеном Telegram Bot API. |
| `allowed_users` | список i64 | `[]` | ID пользователей Telegram, которым разрешено взаимодействие. Пусто = разрешить всем. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |
| `poll_interval_secs` | u64 | `1` | Интервал длинных опросов (long-polling) в секундах. |

#### `[channels.discord]`

```toml
[channels.discord]
bot_token_env = "DISCORD_BOT_TOKEN"
allowed_guilds = []
# default_agent = "assistant"
intents = 33280
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `bot_token_env` | строка | `"DISCORD_BOT_TOKEN"` | Переменная окружения с токеном Discord бота. |
| `allowed_guilds` | список u64 | `[]` | ID разрешенных гильдий (серверов). Пусто = разрешить все. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |
| `intents` | u64 | `33280` | Битовая маска интентов шлюза. По умолчанию = `GUILD_MESSAGES \| MESSAGE_CONTENT`. |

#### `[channels.slack]`

```toml
[channels.slack]
app_token_env = "SLACK_APP_TOKEN"
bot_token_env = "SLACK_BOT_TOKEN"
allowed_channels = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `app_token_env` | строка | `"SLACK_APP_TOKEN"` | Переменная окружения с токеном уровня приложения Slack (`xapp-`) для Socket Mode. |
| `bot_token_env` | строка | `"SLACK_BOT_TOKEN"` | Переменная окружения с токеном бота Slack (`xoxb-`) для REST API. |
| `allowed_channels` | список строк | `[]` | ID разрешенных каналов. Пусто = разрешить все. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |

#### `[channels.whatsapp]`

```toml
[channels.whatsapp]
access_token_env = "WHATSAPP_ACCESS_TOKEN"
verify_token_env = "WHATSAPP_VERIFY_TOKEN"
phone_number_id = ""
webhook_port = 8443
allowed_users = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `access_token_env` | строка | `"WHATSAPP_ACCESS_TOKEN"` | Переменная окружения с токеном доступа WhatsApp Cloud API. |
| `verify_token_env` | строка | `"WHATSAPP_VERIFY_TOKEN"` | Переменная окружения с токеном верификации вебхука. |
| `phone_number_id` | строка | `""` | ID номера телефона WhatsApp Business. |
| `webhook_port` | u16 | `8443` | Порт для прослушивания входящих колбэков вебхука. |
| `allowed_users` | список строк | `[]` | Разрешенные номера телефонов. Пусто = разрешить все. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |

#### `[channels.signal]`

```toml
[channels.signal]
api_url = "http://localhost:8080"
phone_number = ""
allowed_users = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `api_url` | строка | `"http://localhost:8080"` | URL для signal-cli REST API. |
| `phone_number` | строка | `""` | Зарегистрированный номер телефона бота. |
| `allowed_users` | список строк | `[]` | Разрешенные номера телефонов. Пусто = разрешить все. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |

#### `[channels.matrix]`

```toml
[channels.matrix]
homeserver_url = "https://matrix.org"
user_id = "@openfang:matrix.org"
access_token_env = "MATRIX_ACCESS_TOKEN"
allowed_rooms = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `homeserver_url` | строка | `"https://matrix.org"` | URL домашнего сервера Matrix. |
| `user_id` | строка | `""` | ID пользователя бота (например, `"@openfang:matrix.org"`). |
| `access_token_env` | строка | `"MATRIX_ACCESS_TOKEN"` | Переменная окружения с токеном доступа Matrix. |
| `allowed_rooms` | список строк | `[]` | ID комнат для прослушивания. Пусто = все комнаты, в которых состоит бот. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |

#### `[channels.email]`

```toml
[channels.email]
imap_host = "imap.gmail.com"
imap_port = 993
smtp_host = "smtp.gmail.com"
smtp_port = 587
username = "bot@example.com"
password_env = "EMAIL_PASSWORD"
poll_interval_secs = 30
folders = ["INBOX"]
allowed_senders = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `imap_host` | строка | `""` | Имя хоста IMAP-сервера. |
| `imap_port` | u16 | `993` | Порт IMAP-сервера (993 для TLS). |
| `smtp_host` | строка | `""` | Имя хоста SMTP-сервера. |
| `smtp_port` | u16 | `587` | Порт SMTP-сервера (587 для STARTTLS). |
| `username` | строка | `""` | Email адрес для IMAP и SMTP. |
| `password_env` | строка | `"EMAIL_PASSWORD"` | Переменная окружения с паролем от почты или паролем приложения. |
| `poll_interval_secs` | u64 | `30` | Интервал опроса IMAP в секундах. |
| `folders` | список строк | `["INBOX"]` | Папки IMAP для мониторинга. |
| `allowed_senders` | список строк | `[]` | Обрабатывать письма только от этих отправителей. Пусто = от всех. |
| `default_agent` | строка или null | `null` | Имя агента для маршрутизации сообщений. |

#### `[channels.teams]`

```toml
[channels.teams]
app_id = ""
app_password_env = "TEAMS_APP_PASSWORD"
webhook_port = 3978
allowed_tenants = []
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `app_id` | строка | `""` | Azure Bot App ID. |
| `app_password_env` | строка | `"TEAMS_APP_PASSWORD"` | Переменная окружения с паролем приложения Azure Bot Framework. |
| `webhook_port` | u16 | `3978` | Порт для входящего вебхука Bot Framework. |

---

<a name="mcp_servers"></a>
### `[[mcp_servers]]`

Подключения к серверам MCP (Model Context Protocol) обеспечивают интеграцию с внешними инструментами. Каждая запись является отдельным элементом массива `[[mcp_servers]]`.

```toml
[[mcp_servers]]
name = "filesystem"
timeout_secs = 30
env = []

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/docs"]
```

```toml
[[mcp_servers]]
name = "remote-api"
timeout_secs = 60
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]

[mcp_servers.transport]
type = "sse"
url = "https://mcp.example.com/sse"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `name` | строка | *обязательно* | Отображаемое имя MCP сервера. Инструменты будут доступны в пространстве имен `mcp_{name}_{tool}`. |
| `timeout_secs` | u64 | `30` | Таймаут запроса в секундах. |
| `env` | список строк | `[]` | Имена переменных окружения для передачи в подпроцесс (только для транспорта stdio). |

**Варианты транспорта** (размеченное объединение по полю `type`):

| `type` | Поля | Описание |
|--------|--------|-------------|
| `stdio` | `command` (строка), `args` (список строк, по умолчанию `[]`) | Запускает подпроцесс, общение идет через JSON-RPC через stdin/stdout. |
| `sse` | `url` (строка) | Подключается к HTTP эндпоинту Server-Sent Events. |

---

<a name="a2a"></a>
### `[a2a]`

Конфигурация протокола Agent-to-Agent, позволяющая агентам взаимодействовать между различными экземплярами OpenFang.

```toml
[a2a]
enabled = true
listen_path = "/a2a"

[[a2a.external_agents]]
name = "research-agent"
url = "https://agent.example.com/.well-known/agent.json"

[[a2a.external_agents]]
name = "code-reviewer"
url = "https://reviewer.example.com/.well-known/agent.json"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `enabled` | bool | `false` | Включен ли протокол A2A. |
| `listen_path` | строка | `"/a2a"` | Префикс пути URL для эндпоинтов A2A. |
| `external_agents` | список объектов | `[]` | Внешние A2A агенты для обнаружения и взаимодействия. |

**Записи в `external_agents`:**

| Поле | Тип | Описание |
|-------|------|-------------|
| `name` | строка | Отображаемое имя внешнего агента. |
| `url` | строка | URL эндпоинта карточки агента (обычно `/.well-known/agent.json`). |

---

<a name="fallback_providers"></a>
### `[[fallback_providers]]`

Цепочка резервных провайдеров. Если основной провайдер LLM (`[default_model]`) дает сбой, эти провайдеры пробуются по порядку.

```toml
[[fallback_providers]]
provider = "ollama"
model = "llama3.2:latest"
api_key_env = ""
# base_url = "http://localhost:11434"

[[fallback_providers]]
provider = "groq"
model = "llama-3.3-70b-versatile"
api_key_env = "GROQ_API_KEY"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `provider` | строка | `""` | Имя провайдера (например, `"ollama"`, `"groq"`, `"openai"`). |
| `model` | строка | `""` | Идентификатор модели для этого провайдера. |
| `api_key_env` | строка | `""` | Имя переменной окружения для API-ключа. Пусто для локальных провайдеров (ollama, vllm, lmstudio). |
| `base_url` | строка или null | `null` | Переопределение базового URL. Если null, используется значение по умолчанию из каталога. |

---

<a name="users"></a>
### `[[users]]`

Конфигурация многопользовательского доступа на основе ролей (RBAC). Пользователям можно назначать роли и привязывать их к идентификаторам на платформах каналов.

```toml
[[users]]
name = "Alice"
role = "owner"
api_key_hash = "sha256_hash_of_api_key"

[users.channel_bindings]
telegram = "123456"
discord = "987654321"
slack = "U0ABCDEFG"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `name` | строка | *обязательно* | Отображаемое имя пользователя. |
| `role` | строка | `"user"` | Роль пользователя в иерархии RBAC. |
| `channel_bindings` | карта строк | `{}` | Сопоставляет названия платформ каналов с идентификаторами пользователей на этих платформах. |
| `api_key_hash` | строка или null | `null` | SHA256 хеш личного API-ключа пользователя для доступа к API. |

**Иерархия ролей** (от высших привилегий к низшим):

| Роль | Описание |
|------|-------------|
| `owner` | Полный административный доступ. Может управлять всеми агентами, пользователями и конфигурацией. |
| `admin` | Может управлять агентами и большинством настроек. Не может изменять аккаунты владельцев. |
| `user` | Может взаимодействовать с агентами. Ограниченные возможности управления. |
| `viewer` | Доступ только для чтения. Может видеть ответы агентов, но не может отправлять сообщения. |

---

<a name="channel-overrides"></a>
### Переопределения для каналов

Адаптер каждого канала поддерживает подтаблицу `[channels.<name>.overrides]`, которая настраивает поведение агента именно для этого канала.

```toml
[channels.telegram.overrides]
model = "claude-haiku-4-5-20251001"
system_prompt = "Вы — лаконичный помощник в Telegram."
dm_policy = "respond"
group_policy = "mention_only"
rate_limit_per_user = 10
threading = true
output_format = "telegram_html"
usage_footer = "tokens"
```

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `model` | строка или null | `null` | Переопределение модели для этого канала. Если null, используется модель агента по умолчанию. |
| `system_prompt` | строка или null | `null` | Переопределение системного промпта для этого канала. |
| `dm_policy` | строка | `"respond"` | Как бот обрабатывает личные сообщения. См. ниже. |
| `group_policy` | строка | `"mention_only"` | Как бот обрабатывает сообщения в группах. См. ниже. |
| `rate_limit_per_user` | u32 | `0` | Макс. сообщений от пользователя в минуту. `0` = безлимитно. |
| `threading` | bool | `false` | Включить ответы в тредах (где поддерживается платформой). |
| `output_format` | строка или null | `null` | Переопределение форматирования вывода. См. ниже. |
| `usage_footer` | строка или null | `null` | Переопределение режима подвала использования для этого канала. Значения: `off`, `tokens`, `cost`, `full`. |

**Значения `dm_policy`:**

| Значение | Описание |
|-------|-------------|
| `respond` | Отвечать на все личные сообщения (по умолчанию). |
| `allowed_only` | Отвечать только пользователям из списка разрешенных. |
| `ignore` | Игнорировать все личные сообщения. |

**Значения `group_policy`:**

| Значение | Описание |
|-------|-------------|
| `all` | Отвечать на все сообщения в групповых чатах. |
| `mention_only` | Отвечать только при @упоминании бота (по умолчанию). |
| `commands_only` | Отвечать только на слеш-команды. |
| `ignore` | Игнорировать все групповые сообщения. |

**Значения `output_format`:**

| Значение | Описание |
|-------|-------------|
| `markdown` | Стандартный Markdown (по умолчанию). |
| `telegram_html` | Подмножество HTML Telegram (`<b>`, `<i>`, `<code>` и т.д.). |
| `slack_mrkdwn` | Формат mrkdwn Slack (`*bold*`, `_italic_`, `` `code` ``). |
| `plain_text` | Без разметки форматирования. |

---

<a name="environment-variables"></a>
## Переменные окружения

Полная таблица всех переменных окружения, на которые ссылается конфигурация. Сами по себе они не считываются файлом конфигурации — они считываются ядром и адаптерами каналов во время работы.

### Ключи провайдеров LLM

| Переменная | Используется в | Описание |
|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | `[default_model]` | API-ключ Anthropic (модели Claude). |
| `GEMINI_API_KEY` | Драйвер Gemini | API-ключ Google Gemini. Псевдоним: `GOOGLE_API_KEY`. |
| `OPENAI_API_KEY` | OpenAI-совм. драйвер | API-ключ OpenAI. |
| `GROQ_API_KEY` | Провайдер Groq | API-ключ Groq (быстрый инференс Llama). |
| `DEEPSEEK_API_KEY` | Провайдер DeepSeek | API-ключ DeepSeek. |
| `PERPLEXITY_API_KEY` | Провайдер Perplexity | API-ключ Perplexity (LLM и веб-поиск). |
| `OPENROUTER_API_KEY` | Провайдер OpenRouter | API-ключ OpenRouter. |
| `TOGETHER_API_KEY` | Провайдер Together AI | API-ключ Together AI. |
| `MISTRAL_API_KEY` | Провайдер Mistral | API-ключ Mistral AI. |
| `FIREWORKS_API_KEY` | Провайдер Fireworks | API-ключ Fireworks AI. |
| `COHERE_API_KEY` | Провайдер Cohere | API-ключ Cohere. |
| `AI21_API_KEY` | Провайдер AI21 | API-ключ AI21 Labs. |
| `CEREBRAS_API_KEY` | Провайдер Cerebras | API-ключ Cerebras. |
| `SAMBANOVA_API_KEY` | Провайдер SambaNova | API-ключ SambaNova. |
| `HUGGINGFACE_API_KEY` | Провайдер Hugging Face | API-ключ Hugging Face Inference. |
| `XAI_API_KEY` | Провайдер xAI | API-ключ xAI (Grok). |
| `REPLICATE_API_KEY` | Провайдер Replicate | API-ключ Replicate. |

### Ключи веб-поиска

| Переменная | Используется в | Описание |
|----------|---------|-------------|
| `BRAVE_API_KEY` | `[web.brave]` | API-ключ Brave Search. |
| `TAVILY_API_KEY` | `[web.tavily]` | API-ключ Tavily Search. |
| `PERPLEXITY_API_KEY` | `[web.perplexity]` | API-ключ Perplexity Search (общий с LLM). |

---

<a name="validation"></a>
## Валидация

`KernelConfig::validate()` запускается при старте системы и возвращает список **предупреждений** (некритичных). Ядро все равно запускается, но записывает каждое предупреждение в лог.

### Что проверяется

Для каждого **включенного канала** (т.е. раздел конфигурации присутствует в TOML) валидатор проверяет, что соответствующие переменные окружения установлены и не пусты.

Например, для Telegram проверяется `bot_token_env`, для Slack — оба токена (`app_token_env` и `bot_token_env`).

Для **провайдеров веб-поиска** проверяются ключи для `brave`, `tavily` и `perplexity`.

### Что НЕ проверяется

- `api_key_env` в `[default_model]` не проверяется функцией `validate()`. Отсутствие ключей LLM приведет к ошибкам во время работы, когда драйвер будет использован впервые.
- `shared_secret` в `[network]` не проверяется на соответствие `network_enabled`. Если сеть включена с пустым секретом, аутентификация не удастся в момент подключения.
- Конфигурации серверов MCP не проверяются при загрузке конфига. Ошибки подключения проявятся во время фоновой фазы подключения MCP.
- Манифесты агентов имеют свою собственную отдельную валидацию.

---

## Связанная конфигурация

Некоторые подсистемы имеют свою конфигурацию, которая не является частью `config.toml`, но о ней стоит знать:

### Сжатие сессии (в рантайме)

Настраивается внутренне через `CompactionConfig` (в данный момент не вынесено в `config.toml`):

- `threshold` (по умолчанию `80`): Сжимать, когда количество сообщений в сессии превышает это значение.
- `keep_recent` (по умолчанию `20`): Количество последних сообщений, сохраняемых дословно после сжатия.
- `max_summary_tokens` (по умолчанию `1024`): Максимальное количество токенов для саммари сжатых сообщений.

### Песочница WASM (в рантайме)

Настраивается внутренне через `SandboxConfig`:

- `fuel_limit` (по умолчанию `1000000`): Максимальный бюджет инструкций CPU. `0` = безлимитно.
- `max_memory_bytes` (по умолчанию `16 MB`): Максимальная линейная память WASM.
- `timeout_secs`: Таймаут реального времени (по умолчанию 30 с).

### Маршрутизация моделей (манифест агента)

Настраивается в манифестах агентов через `ModelRoutingConfig`: задаются модели для уровней `simple`, `medium` и `complex`, а также пороги токенов для классификации запроса.

### Монитор сердцебиения (Heartbeat)

Глобальные настройки сердцебиения в `[heartbeat]`:
- `default_timeout_secs` (по умолчанию `180`): Секунды бездействия перед тем, как пометить агента как не отвечающего.
