# Справочник OpenFang CLI (рус.)

Краткая справка по основным командам `openfang`.

## Режимы работы

- **Daemon mode** — при запущенном демоне CLI общается с ним по HTTP (рекомендуется для продакшена).
- **In-process mode** — если демон не найден, некоторые команды поднимают временное ядро в процессе (данные не сохраняются).

Запуск без подкоманды открывает TUI (ratatui) с полноэкранной панелью управления.

## Установка (кратко)

```bash
cargo install --path crates/openfang-cli
# или
cargo build --release -p openfang-cli
```

## Глобальные опции

- `--config <PATH>` — путь к файлу конфигурации
- `--help`, `--version`

Переменные окружения: `RUST_LOG`, `OPENFANG_AGENTS_DIR`, `EDITOR`/`VISUAL`.

## Часто используемые команды

```bash
openfang init            # Инициализация ~/.openfang/
openfang start           # Запуск демона
openfang status          # Статус демона
openfang doctor          # Диагностика окружения
openfang agent spawn <manifest.toml>  # Запуск агента
openfang agent list
openfang agent chat <id|name>
openfang hand list
openfang hand activate <name>
```

`openfang --help` и `openfang <cmd> --help` показывают полные опции для команд.

---

### openfang doctor

Проверяет работоспособность и конфигурацию окружения.

**Выполняемые проверки:**

1. **Директория OpenFang** — проверка существования `~/.openfang/`
2. **Файл .env** — существует и имеет правильные права доступа (0600 в Unix)
3. **Синтаксис Config TOML** — проверка корректности парсинга `config.toml`
4. **Статус демона** — запущен ли демон
5. **Доступность порта 4200** — если демон не запущен, проверяется, свободен ли порт
6. **Устаревший daemon.json** — остатки `daemon.json` от аварийно завершенного демона
7. **Файл базы данных** — проверка магических байтов SQLite
8. **Дисковое пространство** — предупреждение, если доступно менее 100 МБ (только для Unix)
9. **Манифесты агентов** — валидация всех `.toml` файлов в `~/.openfang/agents/`
10. **Ключи провайдеров LLM** — проверка переменных окружения для 10 провайдеров (Groq, OpenRouter, Anthropic, OpenAI, DeepSeek, Gemini, Google, Together, Mistral, Fireworks), выполнение живой валидации (обнаружение ошибок 401/403)
11. **Токены каналов** — валидация формата токенов Telegram, Discord, Slack
12. **Согласованность конфигурации** — проверка того, что ссылки `api_key_env` в конфигурации соответствуют фактическим переменным окружения
13. **Инструментарий Rust** — `rustc --version`

**Пример:**

```bash
openfang doctor

openfang doctor --repair

openfang doctor --json
```

---

### openfang dashboard

Открыть веб-панель управления в браузере по умолчанию.

```
openfang dashboard
```

**Поведение:**

- Требуется запущенный демон.
- Открывает URL демона (например, `http://127.0.0.1:4200/`) в системном браузере.
- Копирует URL в системный буфер обмена (использует PowerShell в Windows, `pbcopy` в macOS, `xclip`/`xsel` в Linux).

**Пример:**

```bash
openfang dashboard
```

---

### openfang completion

Генерация скриптов автодополнения для командной оболочки.

```
openfang completion <SHELL>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<SHELL>` | Целевая оболочка. Одна из: `bash`, `zsh`, `fish`, `elvish`, `powershell`. |

**Пример:**

```bash
# Bash
openfang completion bash > ~/.bash_completion.d/openfang

# Zsh
openfang completion zsh > ~/.zfunc/_openfang

# Fish
openfang completion fish > ~/.config/fish/completions/openfang.fish

# PowerShell
openfang completion powershell > openfang.ps1
```

---

## Команды агентов

### openfang agent new

Создать агента на основе встроенного шаблона.

```
openfang agent new [<TEMPLATE>]
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<TEMPLATE>` | Имя шаблона (например, `coder`, `assistant`, `researcher`). Если опущено, отображается интерактивный список всех доступных шаблонов. |

**Поведение:**

- Шаблоны обнаруживаются в: директории `agents/` репозитория (для разработчиков), `~/.openfang/agents/` (установленные) и `OPENFANG_AGENTS_DIR` (переопределение через переменные окружения).
- Каждый шаблон представляет собой директорию, содержащую манифест `agent.toml`.
- В режиме демона: отправляет `POST /api/agents` с манифестом. Агент является постоянным.
- В автономном режиме: запускает временное ядро в процессе. Агент является эфемерным.

**Пример:**

```bash
# Интерактивный выбор
openfang agent new

# Создание по имени
openfang agent new coder

# Создание по шаблону assistant
openfang agent new assistant
```

---

### openfang agent spawn

Создать агента из пользовательского файла манифеста.

```
openfang agent spawn <MANIFEST>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<MANIFEST>` | Путь к файлу манифеста агента в формате TOML. |

**Поведение:**

- Читает и парсит файл манифеста TOML.
- В режиме демона: отправляет необработанный TOML в `POST /api/agents`.
- В автономном режиме: запускает временное ядро и создает агента локально.

**Пример:**

```bash
openfang agent spawn ./my-agent/agent.toml
```

---

### openfang agent list

Список всех запущенных агентов.

```
openfang agent list [--json]
```

**Опции:**

| Опция | Описание |
|---|---|
| `--json` | Вывод в формате JSON массива для скриптов. |

**Колонки вывода:** ID, NAME, STATE, PROVIDER, MODEL (режим демона) или ID, NAME, STATE, CREATED (автономный режим).

**Пример:**

```bash
openfang agent list

openfang agent list --json | jq '.[].name'
```

---

### openfang agent chat

Начать интерактивную сессию чата с конкретным агентом.

```
openfang agent chat <AGENT_ID>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<AGENT_ID>` | UUID агента. Можно получить через `openfang agent list`. |

**Поведение:**

- Открывает цикл чата в стиле REPL.
- Вводите сообщения в приглашении `you>`.
- Ответы агента отображаются в приглашении `agent>`, после чего следует информация об использовании токенов и количестве итераций.
- Введите `exit`, `quit` или нажмите `Ctrl+C`, чтобы завершить сессию.

**Пример:**

```bash
openfang agent chat a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

---

### openfang agent kill

Завершить работу запущенного агента.

```
openfang agent kill <AGENT_ID>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<AGENT_ID>` | UUID агента для завершения. |

**Пример:**

```bash
openfang agent kill a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

---

## Команды воркфлоу (Workflow)

Все команды воркфлоу требуют запущенного демона.

### openfang workflow list

Список всех зарегистрированных воркфлоу.

```
openfang workflow list
```

**Колонки вывода:** ID, NAME, STEPS, CREATED.

---

### openfang workflow create

Создать воркфлоу из файла определения JSON.

```
openfang workflow create <FILE>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<FILE>` | Путь к JSON-файлу, описывающему шаги воркфлоу. |

**Пример:**

```bash
openfang workflow create ./my-workflow.json
```

---

### openfang workflow run

Выполнить воркфлоу по ID.

```
openfang workflow run <WORKFLOW_ID> <INPUT>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<WORKFLOW_ID>` | UUID воркфлоу. Можно получить через `openfang workflow list`. |
| `<INPUT>` | Входной текст для передачи в воркфлоу. |

**Пример:**

```bash
openfang workflow run abc123 "Analyze this code for security issues"
```

---

## Команды триггеров

Все команды триггеров требуют запущенного демона.

### openfang trigger list

Список всех триггеров событий.

```
openfang trigger list [--agent-id <ID>]
```

**Опции:**

| Опция | Описание |
|---|---|
| `--agent-id <ID>` | Фильтровать триггеры по UUID владеющего агента. |

**Колонки вывода:** TRIGGER ID, AGENT ID, ENABLED, FIRES, PATTERN.

---

### openfang trigger create

Создать триггер события для агента.

```
openfang trigger create <AGENT_ID> <PATTERN_JSON> [--prompt <TEMPLATE>] [--max-fires <N>]
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<AGENT_ID>` | UUID агента, которому принадлежит триггер. |
| `<PATTERN_JSON>` | Шаблон триггера в виде JSON-строки. |

**Опции:**

| Опция | По умолчанию | Описание |
|---|---|---|
| `--prompt <TEMPLATE>` | `"Event: {{event}}"` | Шаблон промпта. Используйте `{{event}}` как плейсхолдер для данных события. |
| `--max-fires <N>` | `0` (безлимитно) | Максимальное количество срабатываний триггера. |

**Примеры шаблонов:**

```bash
# Срабатывание на любое событие жизненного цикла
openfang trigger create <AGENT_ID> '{"lifecycle":{}}'

# Срабатывание при создании конкретного агента
openfang trigger create <AGENT_ID> '{"agent_spawned":{"name_pattern":"*"}}'

# Срабатывание при завершении работы агента
openfang trigger create <AGENT_ID> '{"agent_terminated":{}}'

# Срабатывание на все события (ограничено 10 разами)
openfang trigger create <AGENT_ID> '{"all":{}}' --max-fires 10
```

---

### openfang trigger delete

Удалить триггер по ID.

```
openfang trigger delete <TRIGGER_ID>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<TRIGGER_ID>` | UUID триггера для удаления. |

---

## Команды навыков (Skill)

### openfang skill list

Список всех установленных навыков.

```
openfang skill list
```

**Колонки вывода:** NAME, VERSION, TOOLS, DESCRIPTION.

Загружает навыки из `~/.openfang/skills/`, а также встроенные навыки, скомпилированные в бинарный файл.

---

### openfang skill install

Установить навык из локальной директории, URL git или маркетплейса FangHub.

```
openfang skill install <SOURCE>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<SOURCE>` | Имя навыка (FangHub), путь к локальной директории или URL git. |

**Поведение:**

- **Локальная директория:** Ищет `skill.toml` в директории. Если не найдено, проверяет наличие навыков в формате OpenClaw (SKILL.md с YAML frontmatter) и автоматически конвертирует их.
- **Удаленный источник (FangHub):** Загружает и устанавливает из маркетплейса FangHub. Навыки проходят проверку SHA256 и сканирование на наличие инъекций в промпты.

**Пример:**

```bash
# Установка из локальной директории
openfang skill install ./my-skill/

# Установка из FangHub
openfang skill install web-search

# Установка навыка в формате OpenClaw
openfang skill install ./openclaw-skill/
```

---

### openfang skill remove

Удалить установленный навык.

```
openfang skill remove <NAME>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<NAME>` | Имя навыка для удаления. |

**Пример:**

```bash
openfang skill remove web-search
```

---

### openfang skill search

Поиск навыков в маркетплейсе FangHub.

```
openfang skill search <QUERY>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<QUERY>` | Поисковый запрос. |

**Пример:**

```bash
openfang skill search "docker kubernetes"
```

---

### openfang skill create

Интерактивное создание каркаса нового проекта навыка.

```
openfang skill create
```

**Поведение:**

Запрашивает:
- Имя навыка
- Описание
- Среда выполнения (`python`, `node` или `wasm`; по умолчанию `python`)

Создает директорию в `~/.openfang/skills/<name>/` с:
- `skill.toml` — файл манифеста
- `src/main.py` (или `src/index.js`) — точка входа с шаблонным кодом

**Пример:**

```bash
openfang skill create
# Skill name: my-tool
# Description: A custom analysis tool
# Runtime (python/node/wasm) [python]: python
```

---

## Команды каналов (Channel)

### openfang channel list

Список настроенных каналов и их статус.

```
openfang channel list
```

**Колонки вывода:** CHANNEL, ENV VAR, STATUS.

Проверяет `config.toml` на наличие разделов конфигурации каналов и переменные окружения на наличие необходимых токенов. Статус может быть: `Ready`, `Missing env`, `Not configured`.

**Проверяемые каналы:** webchat, telegram, discord, slack, whatsapp, signal, matrix, email.

---

### openfang channel setup

Интерактивный мастер настройки интеграции канала.

```
openfang channel setup [<CHANNEL>]
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<CHANNEL>` | Имя канала. Если опущено, отображается интерактивный выбор. |

**Поддерживаемые каналы:** `telegram`, `discord`, `slack`, `whatsapp`, `email`, `signal`, `matrix`.

Каждый мастер:
1. Отображает пошаговые инструкции для получения учетных данных.
2. Запрашивает токены/учетные данные.
3. Сохраняет токены в `~/.openfang/.env` с правами доступа только для владельца.
4. Добавляет блок конфигурации канала в `config.toml` (запрашивает подтверждение).
5. Предупреждает о необходимости перезапуска демона, если он запущен.

**Пример:**

```bash
# Интерактивный выбор
openfang channel setup

# Прямая настройка
openfang channel setup telegram
openfang channel setup discord
openfang channel setup slack
```

---

### openfang channel test

Отправить тестовое сообщение через настроенный канал.

```
openfang channel test <CHANNEL>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<CHANNEL>` | Имя канала для тестирования. |

Требуется запущенный демон. Отправляет `POST /api/channels/<channel>/test`.

**Пример:**

```bash
openfang channel test telegram
```

---

### openfang channel enable

Включить интеграцию канала.

```
openfang channel enable <CHANNEL>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<CHANNEL>` | Имя канала для включения. |

В режиме демона: отправляет `POST /api/channels/<channel>/enable`. Без демона: выводит уведомление о том, что изменения вступят в силу при следующем запуске.

---

### openfang channel disable

Отключить канал без удаления его конфигурации.

```
openfang channel disable <CHANNEL>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<CHANNEL>` | Имя канала для отключения. |

В режиме демона: отправляет `POST /api/channels/<channel>/disable`. Без демона: выводит уведомление о необходимости отредактировать `config.toml`.

---

## Команды конфигурации (Config)

### openfang config show

Отобразить текущий файл конфигурации.

```
openfang config show
```

Выводит содержимое `~/.openfang/config.toml` с путем к файлу в виде комментария в заголовке.

---

### openfang config edit

Открыть файл конфигурации в вашем редакторе.

```
openfang config edit
```

Использует `$EDITOR`, затем `$VISUAL`, при их отсутствии переходит к `notepad` (Windows) или `vi` (Unix).

---

### openfang config get

Получить отдельное значение конфигурации по пути к ключу через точку.

```
openfang config get <KEY>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<KEY>` | Путь к ключу в структуре TOML через точку. |

**Пример:**

```bash
openfang config get default_model.provider
# groq

openfang config get api_listen
# 127.0.0.1:4200

openfang config get memory.decay_rate
# 0.05
```

---

### openfang config set

Установить значение конфигурации по пути к ключу через точку.

```
openfang config set <KEY> <VALUE>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<KEY>` | Путь к ключу через точку. |
| `<VALUE>` | Новое значение. Тип определяется на основе существующего значения (целое число, число с плавающей точкой, логическое значение или строка). |

**Предупреждение:** Эта команда повторно сериализует TOML-файл, что удаляет все комментарии.

**Пример:**

```bash
openfang config set default_model.provider anthropic
openfang config set default_model.model claude-sonnet-4-20250514
openfang config set api_listen "0.0.0.0:4200"
```

---

### openfang config set-key

Сохранить API-ключ провайдера LLM в `~/.openfang/.env`.

```
openfang config set-key <PROVIDER>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<PROVIDER>` | Имя провайдера (например, `groq`, `anthropic`, `openai`, `gemini`, `deepseek`, `openrouter`, `together`, `mistral`, `fireworks`, `perplexity`, `cohere`, `xai`, `brave`, `tavily`). |

**Поведение:**

- Запрашивает API-ключ в интерактивном режиме.
- Сохраняет в `~/.openfang/.env` как `<PROVIDER_NAME>_API_KEY=<value>`.
- Запускает живой тест валидации через API провайдера.
- Права доступа к файлу ограничены только владельцем в Unix.

**Пример:**

```bash
openfang config set-key groq
# Paste your groq API key: gsk_...
# [ok] Saved GROQ_API_KEY to ~/.openfang/.env
# Testing key... OK
```

---

### openfang config delete-key

Удалить API-ключ из `~/.openfang/.env`.

```
openfang config delete-key <PROVIDER>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<PROVIDER>` | Имя провайдера. |

**Пример:**

```bash
openfang config delete-key openai
```

---

### openfang config test-key

Протестировать соединение с провайдером, используя сохраненный API-ключ.

```
openfang config test-key <PROVIDER>
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<PROVIDER>` | Имя провайдера. |

**Поведение:**

- Читает API-ключ из окружения (загруженного из `~/.openfang/.env`).
- Обращается к эндпоинту моделей/здоровья провайдера.
- Сообщает `OK` (ключ принят) или `FAILED (401/403)` (ключ отклонен).
- Завершается с кодом 1 в случае неудачи.

**Пример:**

```bash
openfang config test-key groq
# Testing groq (GROQ_API_KEY)... OK
```

---

## Быстрый чат (Quick Chat)

### openfang chat

Быстрый псевдоним для запуска сессии чата.

```
openfang chat [<AGENT>]
```

**Аргументы:**

| Аргумент | Описание |
|---|---|
| `<AGENT>` | Необязательное имя агента или UUID. |

**Поведение:**

- **Режим демона:** Находит агента по имени или ID среди запущенных агентов. Если имя агента не указано, использует первого доступного агента. Если агентов нет, предлагает `openfang agent new`.
- **Автономный режим (без демона):** Запускает временное ядро и автоматически создает агента из шаблонов. Ищет агента, соответствующего указанному имени, затем переходит к `assistant`, затем к первому доступному шаблону.

Это самый простой способ начать общение — он работает как с демоном, так и без него.

**Пример:**

```bash
# Чат с агентом по умолчанию
openfang chat

# Чат с конкретным агентом по имени
openfang chat coder

# Чат с конкретным агентом по UUID
openfang chat a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

---

## Миграция

### openfang migrate

Миграция конфигурации и агентов из другого фреймворка агентов.

```
openfang migrate --from <FRAMEWORK> [--source-dir <PATH>] [--dry-run]
```

**Опции:**

| Опция | Описание |
|---|---|
| `--from <FRAMEWORK>` | Исходный фреймворк. Один из: `openclaw`, `langchain`, `autogpt`. |
| `--source-dir <PATH>` | Путь к исходному рабочему пространству. Автоматически определяется, если не задан (например, `~/.openclaw`, `~/.langchain`, `~/Auto-GPT`). |
| `--dry-run` | Показать, что будет импортировано, без внесения изменений. |

**Поведение:**

- Конвертирует конфигурации агентов, манифесты YAML и настройки из исходного фреймворка в формат OpenFang.
- Сохраняет импортированные данные в `~/.openfang/`.
- Записывает `migration_report.md` с кратким изложением того, что было импортировано.

**Пример:**

```bash
# Предварительный просмотр миграции из OpenClaw
openfang migrate --from openclaw --dry-run

# Миграция из OpenClaw (автоопределение источника)
openfang migrate --from openclaw

# Миграция из LangChain с указанием пути к источнику
openfang migrate --from langchain --source-dir /home/user/.langchain

# Миграция из AutoGPT
openfang migrate --from autogpt
```

---

## MCP-сервер

### openfang mcp

Запуск сервера MCP (Model Context Protocol) через stdio.

```
openfang mcp
```

**Поведение:**

- Предоставляет запущенных агентов OpenFang как инструменты MCP через JSON-RPC 2.0 по stdin/stdout с фреймингом Content-Length.
- Каждый агент становится вызываемым инструментом с именем `openfang_agent_<name>` (дефисы заменяются на подчеркивания).
- Подключается к запущенному демону через HTTP, если он доступен; в противном случае запускает временное ядро.
- Версия протокола: `2024-11-05`.
- Максимальный размер сообщения: 10 МБ (лимит безопасности).

**Поддерживаемые методы MCP:**

| Метод | Описание |
|---|---|
| `initialize` | Возвращает возможности и информацию сервера. |
| `tools/list` | Выводит список всех доступных инструментов агентов. |
| `tools/call` | Отправляет сообщение агенту и возвращает ответ. |

**Схема входных данных инструмента:**

Каждый инструмент агента принимает один аргумент `message` (строка).

**Интеграция с Claude Desktop / другими MCP-клиентами:**

Добавьте в конфигурацию вашего MCP-клиента:

```json
{
  "mcpServers": {
    "openfang": {
      "command": "openfang",
      "args": ["mcp"]
    }
  }
}
```

---

## Автоопределение демона

CLI использует двухэтапный механизм для обнаружения запущенного демона:

1. **Чтение `daemon.json`:** При запуске демон записывает в `~/.openfang/daemon.json` адрес прослушивания (например, `127.0.0.1:4200`). CLI читает этот файл, чтобы узнать, где находится демон.

2. **Проверка состояния (Health check):** CLI отправляет `GET http://<listen_addr>/api/health` с таймаутом в 2 секунды. Если проверка прошла успешно, демон считается запущенным, и CLI использует HTTP для связи с ним.

Если любой из этапов завершается неудачно (нет `daemon.json`, файл устарел, таймаут проверки состояния), CLI переходит в автономный режим для команд, которые его поддерживают. Команды, требующие демона (workflows, triggers, channel test/enable/disable, dashboard), завершатся с ошибкой и информативным сообщением.

**Жизненный цикл демона:**

```
openfang start          # Запускает демон, записывает daemon.json
                        # Другие экземпляры CLI обнаруживают daemon.json
openfang status         # Подключается к демону через HTTP
Ctrl+C                  # Демон завершает работу, daemon.json удаляется

openfang doctor --repair  # Очищает устаревший daemon.json после сбоев
```

---

## Файл окружения

OpenFang загружает `~/.openfang/.env` в окружение процесса при каждом вызове CLI. Переменные окружения системы имеют приоритет над значениями из `.env`.

Файл `.env` хранит API-ключи и секреты:

```bash
GROQ_API_KEY=gsk_...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AIza...
TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
```

Управляйте ключами с помощью команд `config set-key` / `config delete-key`, а не редактируйте файл напрямую, так как эти команды обеспечивают правильные права доступа.

---

## Коды выхода

| Код | Значение |
|---|---|
| `0` | Успех. |
| `1` | Общая ошибка (неверные аргументы, сбой операций, отсутствие демона, ошибки парсинга, ошибки создания агента). |
| `130` | Прервано вторым нажатием `Ctrl+C` (принудительный выход). |

---

## Примеры

### Первая настройка

```bash
# 1. Установите ваш API-ключ
export GROQ_API_KEY="gsk_your_key_here"

# 2. Инициализируйте OpenFang
openfang init --quick

# 3. Запустите демон
openfang start
```

### Ежедневное использование

```bash
# Быстрый чат (агент создается автоматически при необходимости)
openfang chat

# Чат с конкретным агентом
openfang chat coder

# Проверка того, что запущено
openfang status

# Открыть веб-панель управления
openfang dashboard
```

### Управление агентами

```bash
# Создание из шаблона
openfang agent new assistant

# Создание из пользовательского манифеста
openfang agent spawn ./agents/custom-agent/agent.toml

# Список запущенных агентов
openfang agent list

# Чат с агентом по UUID
openfang agent chat <UUID>

# Завершение работы агента
openfang agent kill <UUID>
```

### Автоматизация воркфлоу

```bash
# Создание воркфлоу
openfang workflow create ./review-pipeline.json

# Список воркфлоу
openfang workflow list

# Запуск воркфлоу
openfang workflow run <WORKFLOW_ID> "Review the latest PR"
```

### Триггеры событий

```bash
# Создание триггера, который срабатывает при создании агента
openfang trigger create <AGENT_ID> '{"agent_spawned":{"name_pattern":"*"}}' \
  --prompt "New agent spawned: {{event}}" \
  --max-fires 100

# Список всех триггеров
openfang trigger list

# Список триггеров для конкретного агента
openfang trigger list --agent-id <AGENT_ID>

# Удаление триггера
openfang trigger delete <TRIGGER_ID>
```

### Управление навыками

```bash
# Поиск в FangHub
openfang skill search "code review"

# Установка навыка
openfang skill install code-reviewer

# Список установленных навыков
openfang skill list

# Создание нового навыка
openfang skill create

# Удаление навыка
openfang skill remove code-reviewer
```

### Настройка каналов

```bash
# Интерактивный выбор канала
openfang channel setup

# Прямая настройка канала
openfang channel setup telegram

# Проверка статуса каналов
openfang channel list

# Тестирование канала
openfang channel test telegram

# Включение/выключение каналов
openfang channel enable discord
openfang channel disable slack
```

### Конфигурация

```bash
# Просмотр конфигурации
openfang config show

# Получение конкретного значения
openfang config get default_model.provider

# Смена провайдера
openfang config set default_model.provider anthropic
openfang config set default_model.model claude-sonnet-4-20250514
openfang config set default_model.api_key_env ANTHROPIC_API_KEY

# Управление API-ключами
openfang config set-key anthropic
openfang config test-key anthropic
openfang config delete-key openai

# Открыть в редакторе
openfang config edit
```

### Миграция из других фреймворков

```bash
# Предварительный просмотр миграции
openfang migrate --from openclaw --dry-run

# Запуск миграции
openfang migrate --from openclaw

# Миграция из LangChain
openfang migrate --from langchain --source-dir ~/.langchain
```

### Интеграция MCP

```bash
# Запуск MCP-сервера для Claude Desktop или других MCP-клиентов
openfang mcp
```

### Диагностика

```bash
# Запуск всех диагностических проверок
openfang doctor

# Автоматическое исправление проблем
openfang doctor --repair

# Машиночитаемая диагностика
openfang doctor --json
```

### Автодополнение оболочки

```bash
# Генерация и установка дополнений для вашей оболочки
openfang completion bash >> ~/.bashrc
openfang completion zsh > "${fpath[1]}/_openfang"
openfang completion fish > ~/.config/fish/completions/openfang.fish
```

---

## Поддерживаемые провайдеры LLM

Следующие провайдеры распознаются командами `openfang config set-key` и `openfang doctor`:

| Провайдер | Переменная окружения | Модель по умолчанию |
|---|---|---|
| Groq | `GROQ_API_KEY` | `llama-3.3-70b-versatile` |
| Gemini | `GEMINI_API_KEY` или `GOOGLE_API_KEY` | `gemini-2.5-flash` |
| DeepSeek | `DEEPSEEK_API_KEY` | `deepseek-chat` |
| Anthropic | `ANTHROPIC_API_KEY` | `claude-sonnet-4-20250514` |
| OpenAI | `OPENAI_API_KEY` | `gpt-4o` |
| OpenRouter | `OPENROUTER_API_KEY` | `openrouter/google/gemini-2.5-flash` |
| Together | `TOGETHER_API_KEY` | -- |
| Mistral | `MISTRAL_API_KEY` | -- |
| Fireworks | `FIREWORKS_API_KEY` | -- |
| Perplexity | `PERPLEXITY_API_KEY` | -- |
| Cohere | `COHERE_API_KEY` | -- |
| xAI | `XAI_API_KEY` | -- |

Дополнительные ключи провайдеров поиска/получения данных: `BRAVE_API_KEY`, `TAVILY_API_KEY`.
