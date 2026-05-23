# Миграция на OpenFang

Это руководство описывает процесс миграции с OpenClaw (и других фреймворков) на OpenFang. Движок миграции обрабатывает конвертацию конфигурации, импорт агентов, перенос памяти, перенастройку каналов и сканирование навыков.

## Содержание

- [Быстрая миграция](#быстрая-миграция)
- [Что переносится](#что-переносится)
- [Шаги ручной миграции](#шаги-ручной-миграции)
- [Различия в формате конфигурации](#различия-в-формате-конфигурации)
- [Сопоставление имен инструментов](#сопоставление-имен-инструментов)
- [Сопоставление провайдеров](#сопоставление-провайдеров)
- [Сравнение функций](#сравнение-функций)

---

## Быстрая миграция

Запустите одну команду, чтобы перенести весь ваш воркспейс OpenClaw:

```bash
openfang migrate --from openclaw
```

Команда автоматически обнаружит ваш воркспейс OpenClaw в `~/.openclaw/` и импортирует все в `~/.openfang/`.

### Опции

```bash
# Указать кастомную исходную директорию
openfang migrate --from openclaw --source-dir /path/to/openclaw/workspace

# Пробный запуск — посмотреть, что будет импортировано, без внесения изменений
openfang migrate --from openclaw --dry-run
```

### Отчет о миграции

После успешной миграции в `~/.openfang/` сохраняется файл `migration_report.md` с кратким изложением всего, что было импортировано, пропущено или требует ручного вмешательства.

### Другие фреймворки

Планируется поддержка миграции с LangChain и AutoGPT:

```bash
openfang migrate --from langchain   # Скоро
openfang migrate --from autogpt     # Скоро
```

---

## Что переносится

| Элемент | Источник (OpenClaw) | Назначение (OpenFang) | Статус |
|------|-------------------|------------------------|--------|
| **Конфиг** | `~/.openclaw/config.yaml` | `~/.openfang/config.toml` | Полностью автоматизировано |
| **Агенты** | `~/.openclaw/agents/*/agent.yaml` | `~/.openfang/agents/*/agent.toml` | Полностью автоматизировано |
| **Память** | `~/.openclaw/agents/*/MEMORY.md` | `~/.openfang/agents/*/imported_memory.md` | Полностью автоматизировано |
| **Каналы** | `~/.openclaw/messaging/*.yaml` | `~/.openfang/channels_import.toml` | Автоматизировано (ручное слияние) |
| **Навыки** | `~/.openclaw/skills/` | Просканировано и отражено в отчете | Ручная переустановка |
| **Сессии** | `~/.openclaw/agents/*/sessions/` | Не переносится | Рекомендуется начать с чистого листа |
| **Файлы воркспейса** | `~/.openclaw/agents/*/workspace/` | Не переносится | Скопируйте вручную при необходимости |

### Примечание по импорту каналов

Конфигурации каналов (Telegram, Discord, Slack) экспортируются в файл `channels_import.toml`. Вы должны вручную перенести раздел `[channels]` в ваш `~/.openfang/config.toml`.

### Примечание по навыкам

Навыки OpenClaw (Node.js) обнаруживаются и перечисляются в отчете о миграции, но не конвертируются автоматически. После миграции переустановите навыки с помощью:

```bash
openfang skill install <skill-name-or-path>
```

OpenFang автоматически обнаруживает навыки в формате OpenClaw и конвертирует их во время установки.

---

## Шаги ручной миграции

Если вы предпочитаете переносить данные вручную (или вам нужно обработать особые случаи), выполните следующие шаги:

### 1. Инициализация OpenFang

```bash
openfang init
```

Это создаст директорию `~/.openfang/` с `config.toml` по умолчанию.

### 2. Конвертация конфигурации

Переведите ваш `config.yaml` в `config.toml`:

**OpenClaw** (`~/.openclaw/config.yaml`):
```yaml
provider: anthropic
model: claude-sonnet-4-20250514
api_key_env: ANTHROPIC_API_KEY
temperature: 0.7
memory:
  decay_rate: 0.05
```

**OpenFang** (`~/.openfang/config.toml`):
```toml
[default_model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"
api_key_env = "ANTHROPIC_API_KEY"

[memory]
decay_rate = 0.05

[network]
listen_addr = "127.0.0.1:4200"
```

### 3. Конвертация манифестов агентов

Переведите каждый `agent.yaml` в `agent.toml`:

**OpenClaw** (`~/.openclaw/agents/coder/agent.yaml`):
```yaml
name: coder
description: A coding assistant
provider: anthropic
model: claude-sonnet-4-20250514
tools:
  - read_file
  - write_file
  - execute_command
tags:
  - coding
  - dev
```

**OpenFang** (`~/.openfang/agents/coder/agent.toml`):
```toml
name = "coder"
version = "0.1.0"
description = "A coding assistant"
author = "openfang"
module = "builtin:chat"
tags = ["coding", "dev"]

[model]
provider = "anthropic"
model = "claude-sonnet-4-20250514"

[capabilities]
tools = ["file_read", "file_write", "shell_exec"]
memory_read = ["*"]
memory_write = ["self.*"]
```

### 4. Конвертация конфигураций каналов

**OpenClaw** (`~/.openclaw/messaging/telegram.yaml`):
```yaml
type: telegram
bot_token_env: TELEGRAM_BOT_TOKEN
default_agent: coder
allowed_users:
  - "123456789"
```

**OpenFang** (добавьте в `~/.openfang/config.toml`):
```toml
[channels.telegram]
bot_token_env = "TELEGRAM_BOT_TOKEN"
default_agent = "coder"
allowed_users = ["123456789"]
```

### 5. Импорт памяти

Скопируйте любые файлы `MEMORY.md` из агентов OpenClaw в директории агентов OpenFang:

```bash
cp ~/.openclaw/agents/coder/MEMORY.md ~/.openfang/agents/coder/imported_memory.md
```

Ядро поглотит их при первой загрузке.

---

## Различия в формате конфигурации

| Аспект | OpenClaw | OpenFang |
|--------|----------|----------|
| Формат | YAML | TOML |
| Расположение конфига | `~/.openclaw/config.yaml` | `~/.openfang/config.toml` |
| Определение агента | `agent.yaml` | `agent.toml` |
| Конфиг каналов | Отдельные файлы для каждого канала | Объединены в `config.toml` |
| Разрешения инструментов | Неявные (список инструментов) | На основе возможностей (инструменты, память, сеть, шелл) |
| Конфиг моделей | Плоский (поля верхнего уровня) | Вложенный (раздел `[model]`) |
| Модуль агента | Неявный | Явный (`module = "builtin:chat"` / `"wasm:..."` / `"python:..."`) |
| Планирование | Не поддерживается | Встроено (раздел `[schedule]`: реактивное, непрерывное, периодическое, проактивное) |
| Ресурсные квоты | Не поддерживаются | Встроены (раздел `[resources]`: токены/час, память, процессорное время) |
| Сеть | Не поддерживается | Протокол OFP (раздел `[network]`) |

---

## Сопоставление имен инструментов

Имена инструментов были изменены между OpenClaw и OpenFang для единообразия. Движок миграции обрабатывает это автоматически.

| Инструмент OpenClaw | Инструмент OpenFang | Примечания |
|---------------|---------------|-------|
| `read_file` | `file_read` | Именование "существительное-первым" |
| `write_file` | `file_write` | |
| `list_files` | `file_list` | |
| `execute_command` | `shell_exec` | Ограничено возможностями |
| `web_search` | `web_search` | Без изменений |
| `fetch_url` | `web_fetch` | |
| `browser_navigate` | `browser_navigate` | Без изменений |
| `memory_search` | `memory_recall` | |
| `memory_recall` | `memory_recall` | |
| `memory_save` | `memory_store` | |
| `memory_store` | `memory_store` | |
| `sessions_send` | `agent_send` | |
| `agent_message` | `agent_send` | |
| `agents_list` | `agent_list` | |
| `agent_list` | `agent_list` | |

### Новые инструменты в OpenFang

У этих инструментов нет эквивалента в OpenClaw:

| Инструмент | Описание |
|------|-------------|
| `agent_spawn` | Запустить нового агента изнутри другого агента |
| `agent_kill` | Завершить работу другого агента |
| `agent_find` | Поиск агентов по имени, тегу или описанию |
| `memory_store` | Сохранить данные "ключ-значение" в общей памяти |
| `memory_recall` | Извлечь данные "ключ-значение" из общей памяти |
| `task_post` | Разместить задачу на общей доске задач |
| `task_claim` | Взять доступную задачу |
| `task_complete` | Отметить задачу как выполненную |
| `task_list` | Список задач по статусу |
| `event_publish` | Опубликовать кастомное событие в шине событий |
| `schedule_create` | Создать запланированную задачу |
| `schedule_list` | Список запланированных задач |
| `schedule_delete` | Удалить запланированную задачу |
| `image_analyze` | Проанализировать изображение |
| `location_get` | Получить информацию о местоположении |

### Профили инструментов

Профили инструментов OpenClaw сопоставляются с явными списками инструментов:

| Профиль OpenClaw | Инструменты OpenFang |
|------------------|----------------|
| `minimal` | `file_read`, `file_list` |
| `coding` | `file_read`, `file_write`, `file_list`, `shell_exec`, `web_fetch` |
| `messaging` | `agent_send`, `agent_list`, `memory_store`, `memory_recall` |
| `research` | `web_fetch`, `web_search`, `file_read`, `file_write` |
| `full` | Все 10 основных инструментов |

---

## Сопоставление провайдеров

| Имя OpenClaw | Имя OpenFang | Переменная окружения API-ключа |
|---------------|---------------|-----------------|
| `anthropic` | `anthropic` | `ANTHROPIC_API_KEY` |
| `claude` | `anthropic` | `ANTHROPIC_API_KEY` |
| `openai` | `openai` | `OPENAI_API_KEY` |
| `gpt` | `openai` | `OPENAI_API_KEY` |
| `groq` | `groq` | `GROQ_API_KEY` |
| `ollama` | `ollama` | (не требуется) |
| `openrouter` | `openrouter` | `OPENROUTER_API_KEY` |
| `deepseek` | `deepseek` | `DEEPSEEK_API_KEY` |
| `together` | `together` | `TOGETHER_API_KEY` |
| `mistral` | `mistral` | `MISTRAL_API_KEY` |
| `fireworks` | `fireworks` | `FIREWORKS_API_KEY` |

### Новые провайдеры в OpenFang

| Провайдер | Описание |
|----------|-------------|
| `vllm` | Собственный сервер инференса vLLM |
| `lmstudio` | Локальные модели LM Studio |

---

## Сравнение функций

| Функция | OpenClaw | OpenFang |
|---------|----------|----------|
| **Язык** | Node.js / TypeScript | Rust |
| **Формат конфига** | YAML | TOML |
| **Манифесты агентов** | YAML | TOML |
| **Многоагентность** | Базовая (передача сообщений) | Первоклассная (запуск, завершение, поиск, воркфлоу, триггеры) |
| **Планирование агентов** | Вручную | Встроено (реактивное, непрерывное, периодическое, проактивное) |
| **Память** | Markdown-файлы | SQLite + KV-хранилище + семантический поиск + граф знаний |
| **Управление сессиями** | JSONL-файлы | SQLite с отслеживанием окна контекста |
| **Провайдеры LLM** | ~5 | 11 (Anthropic, OpenAI, Groq, OpenRouter, DeepSeek, Together, Mistral, Fireworks, Ollama, vLLM, LM Studio) |
| **Модели для каждого агента** | Нет | Да (переопределение провайдера и модели для агента) |
| **Безопасность** | Отсутствует | На основе возможностей (инструменты, память, сеть, шелл, запуск агента) |
| **Ресурсные квоты** | Отсутствуют | Лимиты токенов/час, памяти и процессорного времени для каждого агента |
| **Движок воркфлоу** | Отсутствует | Встроенный (последовательный, fan-out, сбор, условный, цикличный) |
| **Триггеры событий** | Отсутствуют | Триггеры событий с сопоставлением шаблонов и промптами на основе шаблонов |
| **Песочница WASM** | Отсутствует | Исполнение в песочнице на базе Wasmtime |
| **Python рантайм** | Отсутствует | Исполнение Python-агентов через подпроцессы |
| **Сеть** | Отсутствует | Одноранговая сеть OFP (OpenFang Protocol) |
| **Сервер API** | Базовый REST | REST + WebSocket + потоковая передача SSE |
| **Интерфейс WebChat** | Отдельно | Встроен в демон |
| **Адаптеры каналов** | Telegram, Discord | Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Email |
| **Навыки/Плагины** | пакеты npm | TOML + Python/WASM/Node.js, маркетплейс FangHub |
| **CLI** | Базовый | Полноценный CLI с автоопределением демона, сервер MCP |
| **Поддержка MCP** | Нет | Встроенный сервер MCP (stdio) |
| **Процесс-супервизор** | Отсутствует | Мониторинг здоровья, отслеживание паник/перезапусков |
| **Постоянство данных** | Файловое | SQLite (агенты сохраняются после перезапуска) |

---

## Устранение неполадок

### Ошибка миграции "Source directory not found"

Движок миграции по умолчанию ищет `~/.openclaw/`. Если ваш воркспейс OpenClaw находится в другом месте:

```bash
openfang migrate --from openclaw --source-dir /path/to/your/workspace
```

### Агент не запускается после миграции

Проверьте конвертированный `agent.toml` на наличие:
- Валидных имен инструментов (см. таблицу [Сопоставление имен инструментов](#сопоставление-имен-инструментов))
- Валидного имени провайдера (см. таблицу [Сопоставление провайдеров](#сопоставление-провайдеров))
- Правильного поля `module` (должно быть `"builtin:chat"` для стандартных LLM-агентов)

### Навыки не работают

Навыки OpenClaw Node.js должны быть переустановлены:

```bash
openfang skill install /path/to/openclaw/skills/my-skill
```

Установщик автоматически определит формат OpenClaw и конвертирует манифест навыка.

### Канал не подключается

После миграции каналы экспортируются в `channels_import.toml`. Вы должны объединить их с вашим `config.toml` вручную:

```bash
cat ~/.openfang/channels_import.toml
# Скопируйте разделы [channels.*] в ~/.openfang/config.toml
```

Затем перезапустите демон:

```bash
openfang start
```
