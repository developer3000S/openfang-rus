# Начало работы с OpenFang

Это руководство поможет вам установить OpenFang, настроить вашего первого провайдера LLM, создать агента и начать с ним общение.

## Содержание

- [Установка](#installation)
- [Конфигурация](#configuration)
- [Создание вашего первого агента](#spawn-your-first-agent)
- [Чат с агентом](#chat-with-agent)
- [Запуск демона](#start-the-daemon)
- [Использование веб-чата](#using-the-webchat-ui)
- [Следующие шаги](#next-steps)

---

<a name="installation"></a>
## Установка

### Вариант 1: Десктоп-приложение (Windows / macOS / Linux)

Скачайте установщик для вашей платформы из [последнего релиза](https://github.com/RightNow-AI/openfang/releases/latest):

| Платформа | Файл |
|---|---|
| Windows | установщик `.msi` |
| macOS | образ диска `.dmg` |
| Linux | `.AppImage` или `.deb` |

Десктоп-приложение включает полную систему OpenFang с нативным окном, системным треем, автообновлениями и уведомлениями ОС. Обновления устанавливаются автоматически в фоновом режиме.

### Вариант 2: Shell-установщик (Linux / macOS)

```bash
curl -sSf https://openfang.sh | sh
```

Эта команда скачивает последний бинарный файл CLI и устанавливает его в `~/.openfang/bin/`.

### Вариант 3: PowerShell-установщик (Windows)

```powershell
irm https://openfang.sh/install.ps1 | iex
```

Скачивает последний бинарный файл CLI, проверяет его контрольную сумму SHA256 и добавляет его в переменную окружения PATH.

### Вариант 4: Установка через Cargo (любая платформа)

Требуется Rust 1.75+:

```bash
cargo install --git https://github.com/RightNow-AI/openfang openfang-cli
```

Или сборка из исходников:

```bash
git clone https://github.com/RightNow-AI/openfang.git
cd openfang
cargo install --path crates/openfang-cli
```

### Вариант 5: Docker

```bash
docker pull ghcr.io/RightNow-AI/openfang:latest

docker run -d \
  --name openfang \
  -p 4200:4200 \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  -v openfang-data:/data \
  ghcr.io/RightNow-AI/openfang:latest
```

Или используйте Docker Compose:

```bash
git clone https://github.com/RightNow-AI/openfang.git
cd openfang
# Установите ваши API-ключи в переменных окружения или в файле .env
docker compose up -d
```

**Доступ к сервисам хоста из контейнера.** Если вы запускаете локальную LLM (Ollama, whisper.cpp, vLLM) на хосте и хотите, чтобы агент мог обращаться к ней, добавьте мост host-gateway. Это необходимо в Linux и colima:

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_HOST=http://host.docker.internal:11434 \
  -p 4200:4200 \
  ghcr.io/rightnow-ai/openfang:latest
```

Для Compose добавьте `extra_hosts: ["host.docker.internal:host-gateway"]` в конфигурацию сервиса. Подробности см. в [Устранение неполадок → Подключение к сервисам хоста из Docker](troubleshooting.md#connecting-to-host-services-from-docker).

### Проверка установки

```bash
openfang --version
```

---

<a name="configuration"></a>
## Конфигурация

### Инициализация

Запустите команду инициализации, чтобы создать директорию `~/.openfang/` и файл конфигурации по умолчанию:

```bash
openfang init
```

Это создаст:

```
~/.openfang/
  config.toml    # Основная конфигурация
  data/          # База данных и данные рантайма
  agents/        # Манифесты агентов (опционально)
```

### Настройка API-ключа

OpenFang требует API-ключ хотя бы одного провайдера LLM. Установите его как переменную окружения:

```bash
# Anthropic (Claude)
export ANTHROPIC_API_KEY=sk-ant-...

# Или OpenAI
export OPENAI_API_KEY=sk-...

# Или Groq (есть бесплатный уровень)
export GROQ_API_KEY=gsk_...
```
Добавьте команду export в профиль вашей оболочки (`~/.bashrc`, `~/.zshrc` и т. д.), чтобы настройки сохранялись.

### Редактирование конфига

По умолчанию используется Anthropic. Чтобы сменить провайдера, отредактируйте `~/.openfang/config.toml`:

```toml
[default_model]
provider = "groq"                      # anthropic, openai, groq, ollama и т. д.
model = "llama-3.3-70b-versatile"      # Идентификатор модели провайдера
api_key_env = "GROQ_API_KEY"           # Переменная окружения с ключом

[memory]
decay_rate = 0.05                      # Скорость затухания уверенности в памяти

[network]
listen_addr = "127.0.0.1:4200"        # Адрес прослушивания OFP
```

### Проверка настроек

```bash
openfang doctor
```

Эта команда проверяет наличие конфига, установленные API-ключи и доступность инструментов.

---

<a name="spawn-your-first-agent"></a>
## Создание вашего первого агента

### Использование встроенного шаблона

OpenFang поставляется с 30 шаблонами агентов. Запустите агента hello-world:

```bash
openfang agent spawn agents/hello-world/agent.toml
```

Вывод:

```
Agent spawned successfully!
  ID:   a1b2c3d4-e5f6-...
  Name: hello-world
```

### Использование собственного манифеста

Создайте файл `my-agent.toml`:

```toml
name = "my-assistant"
version = "0.1.0"
description = "Полезный помощник"
author = "вы"
module = "builtin:chat"

[model]
provider = "groq"
model = "llama-3.3-70b-versatile"

tools = ["file_read", "file_list", "web_fetch"]
memory_read = ["*"]
memory_write = ["self.*"]
```

Затем создайте агента:

```bash
openfang agent spawn my-agent.toml
```

### Список запущенных агентов

```bash
openfang agent list
```

Вывод:

```
ID                                     NAME             STATE      PROVIDER     MODEL
-----------------------------------------------------------------------------------------------
a1b2c3d4-e5f6-...                     hello-world      Running    groq         llama-3.3-70b-versatile
```

---

<a name="chat-with-agent"></a>
## Чат с агентом

Начните интерактивную сессию чата, используя ID агента:

```bash
openfang agent chat a1b2c3d4-e5f6-...
```

Или используйте команду быстрого чата (выбирает первого доступного агента):

```bash
openfang chat
```

Или укажите агента по имени:

```bash
openfang chat hello-world
```

Пример сессии:

```
Chat session started (daemon mode). Type 'exit' or Ctrl+C to quit.

you> Привет! Что ты умеешь?

agent> Я агент hello-world, работающий на OpenFang. Я умею:
- Читать файлы из файловой системы
- Загружать веб-страницы

Попробуй попросить меня прочитать файл или найти что-нибудь в сети!
  [tokens: 142 in / 87 out | iterations: 1]

you> Покажи список файлов в текущей директории

agent> Вот список файлов в текущей директории:
- Cargo.toml
- Cargo.lock
- README.md
- agents/
- crates/
- docs/
...

you> exit
Chat session ended.
```

---

<a name="start-the-daemon"></a>
## Запуск демона

Для работы постоянных агентов, многопользовательского доступа и интерфейса WebChat запустите демон:

```bash
openfang start
```

Демон предоставляет:
- **REST API** по адресу `http://127.0.0.1:4200/api/`
- **WebSocket** эндпоинт `ws://127.0.0.1:4200/api/agents/{id}/ws`
- **WebChat UI** по адресу `http://127.0.0.1:4200/`
- **P2P сеть OFP** на порту 4200

### Остановка демона

Нажмите `Ctrl+C` в терминале с запущенным демоном или используйте команды управления процессами.

---

<a name="using-the-webchat-ui"></a>
## Использование веб-интерфейса чата

Когда демон запущен, откройте браузер и перейдите по адресу:

```
http://127.0.0.1:4200/
```

Встроенный WebChat позволяет:
- Видеть всех запущенных агентов.
- Общаться с любым агентом в реальном времени (через WebSocket).
- Видеть потоковые ответы по мере их генерации.
- Отслеживать использование токенов для каждого сообщения.

---

<a name="next-steps"></a>
## Следующие шаги

Теперь, когда OpenFang запущен:

- **Изучите шаблоны агентов**: В директории `agents/` находятся 30 готовых агентов (программист, исследователь, писатель, сисадмин, аналитик, аудитор безопасности и другие).
- **Создавайте собственных агентов**: Пишите свои манифесты `agent.toml`. Подробности о возможностях и планировании см. в [Руководстве по архитектуре](architecture.md).
- **Настройте каналы**: Подключите любую из 40 платформ обмена сообщениями (Telegram, Discord, Slack, WhatsApp, LINE, Mastodon и еще 34). См. [Адаптеры каналов](channel-adapters.md).
- **Используйте встроенные навыки**: 60 экспертных навыков уже предустановлены (GitHub, Docker, Kubernetes, аудит безопасности, промпт-инжиниринг и т. д.). См. [Разработка навыков](skill-development.md).
- **Разрабатывайте свои навыки**: Расширяйте возможности агентов с помощью Python, WASM или чисто текстовых навыков. См. [Разработка навыков](skill-development.md).
- **Используйте API**: 76 REST/WS/SSE эндпоинтов, включая OpenAI-совместимый `/v1/chat/completions`. См. [Справочник API](api-reference.md).
- **Меняйте провайдеров LLM**: Поддерживается 20 провайдеров (Anthropic, OpenAI, Gemini, Groq, DeepSeek, xAI, Ollama и другие). Возможно переопределение модели для каждого агента.
- **Настраивайте воркфлоу**: Объединяйте нескольких агентов в цепочки. Используйте `openfang workflow create` с определением воркфлоу в формате TOML.
- **Используйте MCP**: Подключайтесь к внешним инструментам через Model Context Protocol. Настройте в `config.toml` в разделе `[[mcp_servers]]`.
- **Мигрируйте с OpenClaw**: Запустите `openfang migrate --from openclaw`. См. [MIGRATION.md](../MIGRATION.md).
- **Десктоп-приложение**: Запустите `cargo tauri dev` для нативной работы с системным треем.
- **Проведите диагностику**: `openfang doctor` проверит всю вашу установку.

### Шпаргалка по командам

```bash
openfang init                          # Инициализация ~/.openfang/
openfang start                         # Запуск демона
openfang status                        # Проверка статуса демона
openfang doctor                        # Запуск диагностических проверок

openfang agent spawn <manifest.toml>   # Создание агента
openfang agent list                    # Список всех агентов
openfang agent chat <id>               # Чат с агентом
openfang workflow run <id> <input>     # Запуск воркфлоу
openfang trigger list                  # Список триггеров событий
openfang trigger create <args>         # Создание триггера
openfang trigger delete <id>           # Удаление триггера

openfang skill install <source>        # Установка навыка
openfang mcp                           # Запуск MCP сервера (stdio)
```
