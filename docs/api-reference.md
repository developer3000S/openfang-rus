# Справочник API

OpenFang предоставляет REST API, WebSocket и SSE-стриминг. По умолчанию API слушает `http://127.0.0.1:4200`.

Все ответы защищены заголовками безопасности (CSP, X-Frame-Options, X-Content-Type-Options, HSTS) и проходят через GCRA rate limiter. Система безопасности включает Merkle-audit, taint tracking, WASM metering, Ed25519 подписи, SSRF-защиту и secret zeroization.

## Основные разделы

- Аутентификация и health
- Эндпоинты агентов (создание, чат, управление)
- Workflows и triggers
- Память и сессии
- Каналы
- Каталог моделей и провайдеры
- OpenAI-совместимые эндпоинты (`/v1/chat/completions`)

## Аутентификация

Когда в `config.toml` указан `api_key`, защищенные маршруты требуют заголовок:

```
Authorization: Bearer <ваш-api-key>
```

Публичные маршруты (без auth): `GET /api/health`, `GET /` (WebChat UI).

---

## Эндпоинты агентов

`GET /api/agents` — список агентов

`GET /api/agents/{id}` — информация по агенту

`POST /api/agents` — создать агента (манифест в теле запроса)

`POST /api/agents/{id}/message` — отправить сообщение агенту

`POST /api/agents/{id}/stop` — остановить агента

`GET /api/agents/{id}/session` — история сессии агента

### GET /api/agents/{id}

Возвращает подробную информацию об одном агенте.

**Ответ** `200 OK`:

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "hello-world",
  "state": "Running",
  "created_at": "2025-01-15T10:30:00Z",
  "session_id": "s1b2c3d4-...",
  "model": {
    "provider": "groq",
    "model": "llama-3.3-70b-versatile"
  },
  "capabilities": {
    "tools": ["file_read", "file_list", "web_fetch"],
    "network": []
  },
  "description": "A friendly greeting agent",
  "tags": []
}
```

### POST /api/agents

Запустить нового агента из TOML-манифеста.

**Тело запроса** (JSON):

```json
{
  "manifest_toml": "name = \"my-agent\"\nversion = \"0.1.0\"\ndescription = \"Test agent\"\nauthor = \"me\"\nmodule = \"builtin:chat\"\n\n[model]\nprovider = \"groq\"\nmodel = \"llama-3.3-70b-versatile\"\n\n[capabilities]\ntools = [\"file_read\", \"web_fetch\"]\nmemory_read = [\"*\"]\nmemory_write = [\"self.*\"]\n"
}
```

**Ответ** `201 Created`:

```json
{
  "agent_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "my-agent"
}
```

### PUT /api/agents/{id}/update

Обновить конфигурацию агента во время выполнения.

**Тело запроса**:

```json
{
  "description": "Updated description",
  "system_prompt": "You are a specialized assistant.",
  "tags": ["updated", "v2"]
}
```

**Ответ** `200 OK`:

```json
{
  "status": "updated",
  "agent_id": "a1b2c3d4-..."
}
```

### PUT /api/agents/{id}/mode

Установить режим работы агента. Режим `Stable` закрепляет текущую модель и замораживает реестр навыков. Режим `Normal` восстанавливает поведение по умолчанию.

**Тело запроса**:

```json
{
  "mode": "Stable"
}
```

**Ответ** `200 OK`:

```json
{
  "status": "updated",
  "mode": "Stable",
  "agent_id": "a1b2c3d4-..."
}
```

### POST /api/agents/{id}/message

Отправить сообщение агенту и получить полный ответ.

**Тело запроса**:

```json
{
  "message": "Какие файлы находятся в текущей директории?"
}
```

**Ответ** `200 OK`:

```json
{
  "response": "Вот файлы в текущей директории:\n- Cargo.toml\n- README.md\n...",
  "input_tokens": 142,
  "output_tokens": 87,
  "iterations": 1
}
```

### GET /api/agents/{id}/session

Возвращает историю переписки агента.

**Ответ** `200 OK`:

```json
{
  "session_id": "s1b2c3d4-...",
  "agent_id": "a1b2c3d4-...",
  "message_count": 4,
  "context_window_tokens": 1250,
  "messages": [
    {
      "role": "User",
      "content": "Привет"
    },
    {
      "role": "Assistant",
      "content": "Привет! Чем я могу вам помочь?"
    }
  ]
}
```

### DELETE /api/agents/{id}

Завершить работу агента и удалить его из реестра.

**Ответ** `200 OK`:

```json
{
  "status": "killed",
  "agent_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

---

## Эндпоинты воркфлоу (Workflow)

### GET /api/workflows

Список всех зарегистрированных воркфлоу.

**Ответ** `200 OK`:

```json
[
  {
    "id": "w1b2c3d4-...",
    "name": "code-review-pipeline",
    "description": "Automated code review workflow",
    "steps": 3,
    "created_at": "2025-01-15T10:30:00Z"
  }
]
```

### POST /api/workflows

Создать новое определение воркфлоу.

**Тело запроса** (JSON):

```json
{
  "name": "code-review-pipeline",
  "description": "Review code changes with multiple agents",
  "steps": [
    {
      "name": "analyze",
      "agent_name": "coder",
      "prompt": "Analyze this code for potential issues: {{input}}",
      "mode": "sequential",
      "timeout_secs": 120,
      "error_mode": "fail",
      "output_var": "analysis"
    },
    {
      "name": "security-check",
      "agent_name": "security-auditor",
      "prompt": "Review this code analysis for security vulnerabilities: {{analysis}}",
      "mode": "sequential",
      "timeout_secs": 120,
      "error_mode": "skip"
    },
    {
      "name": "summarize",
      "agent_name": "writer",
      "prompt": "Write a concise code review summary based on: {{analysis}}",
      "mode": "sequential",
      "timeout_secs": 60,
      "error_mode": "fail"
    }
  ]
}
```

**Опции конфигурации шага:**

| Поле | Тип | Описание |
|-------|------|-------------|
| `name` | string | Имя шага |
| `agent_id` | string | UUID агента (используйте либо это, либо `agent_name`) |
| `agent_name` | string | Имя агента (используйте либо это, либо `agent_id`) |
| `prompt` | string | Шаблон промпта с заполнителями `{{input}}` и `{{output_var}}` |
| `mode` | string | `"sequential"`, `"fan_out"`, `"collect"`, `"conditional"`, `"loop"` |
| `timeout_secs` | integer | Тайм-аут на шаг (по умолчанию: 120) |
| `error_mode` | string | `"fail"`, `"skip"`, `"retry"` |
| `max_retries` | integer | Для режима ошибок `"retry"` (по умолчанию: 3) |
| `output_var` | string | Имя переменной для сохранения вывода для последующих шагов |
| `condition` | string | Для режима `"conditional"` |
| `max_iterations` | integer | Для режима `"loop"` (по умолчанию: 5) |
| `until` | string | Для режима `"loop"`: условие остановки |

**Ответ** `201 Created`:

```json
{
  "workflow_id": "w1b2c3d4-..."
}
```

### POST /api/workflows/{id}/run

Запустить воркфлоу.

**Тело запроса**:

```json
{
  "input": "Review this pull request: ..."
}
```

**Ответ** `200 OK`:

```json
{
  "run_id": "r1b2c3d4-...",
  "output": "Code review summary:\n- No critical issues found\n...",
  "status": "completed"
}
```

### GET /api/workflows/{id}/runs

Список истории запусков воркфлоу.

**Ответ** `200 OK`:

```json
[
  {
    "id": "r1b2c3d4-...",
    "workflow_name": "code-review-pipeline",
    "state": "Completed",
    "steps_completed": 3,
    "started_at": "2025-01-15T10:30:00Z",
    "completed_at": "2025-01-15T10:32:15Z"
  }
]
```

---

## Эндпоинты триггеров (Trigger)

### GET /api/triggers

Список всех триггеров. Опционально фильтруется по агенту.

**Параметры запроса:**
- `agent_id` (опционально): Фильтр по UUID агента

**Ответ** `200 OK`:

```json
[
  {
    "id": "t1b2c3d4-...",
    "agent_id": "a1b2c3d4-...",
    "pattern": {"lifecycle": {}},
    "prompt_template": "Event: {{event}}",
    "enabled": true,
    "fire_count": 5,
    "max_fires": 0,
    "created_at": "2025-01-15T10:30:00Z"
  }
]
```

### POST /api/triggers

Создать новый триггер событий.

**Тело запроса**:

```json
{
  "agent_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "pattern": {
    "agent_spawned": {
      "name_pattern": "*"
    }
  },
  "prompt_template": "A new agent was spawned: {{event}}. Review its capabilities.",
  "max_fires": 0
}
```

**Поддерживаемые типы шаблонов:**

| Шаблон | Описание |
|---------|-------------|
| `{"lifecycle": {}}` | Все события жизненного цикла |
| `{"agent_spawned": {"name_pattern": "*"}}` | События запуска агентов |
| `{"agent_terminated": {}}` | События завершения работы агентов |
| `{"all": {}}` | Все события |

**Ответ** `201 Created`:

```json
{
  "trigger_id": "t1b2c3d4-...",
  "agent_id": "a1b2c3d4-..."
}
```

### PUT /api/triggers/{id}

Обновить конфигурацию существующего триггера.

**Тело запроса**:

```json
{
  "prompt_template": "Updated template: {{event}}",
  "enabled": false,
  "max_fires": 10
}
```

**Ответ** `200 OK`:

```json
{
  "status": "updated",
  "trigger_id": "t1b2c3d4-..."
}
```

### DELETE /api/triggers/{id}

Удалить триггер.

**Ответ** `200 OK`:

```json
{
  "status": "removed",
  "trigger_id": "t1b2c3d4-..."
}
```

---

## Эндпоинты памяти (Memory)

### GET /api/memory/agents/{id}/kv

Список всех пар "ключ-значение" для агента.

**Ответ** `200 OK`:

```json
{
  "kv_pairs": [
    {"key": "preferences", "value": {"theme": "dark"}},
    {"key": "state", "value": {"step": 3}}
  ]
}
```

### GET /api/memory/agents/{id}/kv/{key}

Получить конкретную пару "ключ-значение".

**Ответ** `200 OK`:

```json
{
  "key": "preferences",
  "value": {"theme": "dark"}
}
```

**Ответ** `404 Not Found` (ключ не существует):

```json
{
  "error": "Key 'preferences' not found"
}
```

### PUT /api/memory/agents/{id}/kv/{key}

Установить пару "ключ-значение". Создает или перезаписывает.

**Тело запроса**:

```json
{
  "value": {"theme": "dark", "language": "en"}
}
```

**Ответ** `200 OK`:

```json
{
  "status": "stored",
  "key": "preferences"
}
```

### DELETE /api/memory/agents/{id}/kv/{key}

Удалить пару "ключ-значение".

**Ответ** `200 OK`:

```json
{
  "status": "deleted",
  "key": "preferences"
}
```

---

## Эндпоинты каналов (Channel)

### GET /api/channels

Список настроенных адаптеров каналов и их статус. Поддерживается 40 адаптеров каналов, включая Telegram, Discord, Slack, WhatsApp, Matrix, Email, Teams, Mattermost, IRC, Google Chat, Twitch, Rocket.Chat, Zulip, XMPP, LINE, Viber, Messenger, Reddit, Mastodon, Bluesky и другие.

**Ответ** `200 OK`:

```json
{
  "channels": [
    {
      "name": "telegram",
      "enabled": true,
      "has_token": true
    },
    {
      "name": "discord",
      "enabled": true,
      "has_token": false
    }
  ],
  "total": 2
}
```

---

## Эндпоинты шаблонов (Template)

### GET /api/templates

Список доступных шаблонов агентов из директории agents.

**Ответ** `200 OK`:

```json
{
  "templates": [
    {
      "name": "hello-world",
      "description": "A friendly greeting agent",
      "path": "/home/user/.openfang/agents/hello-world/agent.toml"
    },
    {
      "name": "coder",
      "description": "Expert coding assistant",
      "path": "/home/user/.openfang/agents/coder/agent.toml"
    }
  ],
  "total": 30
}
```

### GET /api/templates/{name}

Получить манифест конкретного шаблона и сырой TOML.

**Ответ** `200 OK`:

```json
{
  "name": "hello-world",
  "manifest": {
    "name": "hello-world",
    "description": "A friendly greeting agent",
    "module": "builtin:chat",
    "tags": [],
    "model": {
      "provider": "groq",
      "model": "llama-3.3-70b-versatile"
    },
    "capabilities": {
      "tools": ["file_read", "file_list", "web_fetch"],
      "network": []
    }
  },
  "manifest_toml": "name = \"hello-world\"\nversion = \"0.1.0\"\n..."
}
```

---

## Системные эндпоинты

### GET /api/health

Публичная проверка здоровья. Не требует аутентификации. Возвращает сокращенное подмножество статуса системы (без деталей базы данных или количества агентов).

**Ответ** `200 OK`:

```json
{
  "status": "ok",
  "uptime_seconds": 3600,
  "panic_count": 0,
  "restart_count": 0
}
```

Поле `status` имеет значение `"ok"`, когда все системы исправны, или `"degraded"`, когда база данных недоступна.

### GET /api/health/detail

Полная проверка здоровья со статусом всех зависимостей. Требует аутентификации. В отличие от публичного `/api/health`, этот эндпоинт включает информацию о подключении к базе данных и количестве агентов.

**Ответ** `200 OK`:

```json
{
  "status": "ok",
  "uptime_seconds": 3600,
  "panic_count": 0,
  "restart_count": 0,
  "agent_count": 3,
  "database": "connected",
  "config_warnings": []
}
```

### GET /api/status

Подробный статус ядра, включая всех агентов.

**Ответ** `200 OK`:

```json
{
  "status": "running",
  "agent_count": 2,
  "data_dir": "/home/user/.openfang/data",
  "default_provider": "groq",
  "default_model": "llama-3.3-70b-versatile",
  "uptime_seconds": 3600,
  "agents": [
    {
      "id": "a1b2c3d4-...",
      "name": "hello-world",
      "state": "Running",
      "created_at": "2025-01-15T10:30:00Z",
      "model_provider": "groq",
      "model_name": "llama-3.3-70b-versatile"
    }
  ]
}
```

### GET /api/version

Информация о сборке и версии.

**Ответ** `200 OK`:

```json
{
  "name": "openfang",
  "version": "0.1.0",
  "build_date": "2025-01-15",
  "git_sha": "abc1234",
  "rust_version": "1.82.0",
  "platform": "linux",
  "arch": "x86_64"
}
```

### POST /api/shutdown

Инициировать корректное завершение работы. Состояния агентов сохраняются в SQLite для восстановления при следующем запуске.

**Ответ** `200 OK`:

```json
{
  "status": "shutting_down"
}
```

### GET /api/profiles

Список доступных профилей агентов (предопределенные конфигурации для распространенных случаев использования).

**Ответ** `200 OK`:

```json
{
  "profiles": [
    {
      "name": "coder",
      "tier": "smart",
      "description": "Expert coding assistant"
    },
    {
      "name": "researcher",
      "tier": "frontier",
      "description": "Deep research and analysis"
    }
  ]
}
```

### GET /api/tools

Список всех доступных инструментов, которые могут использовать агенты.

**Ответ** `200 OK`:

```json
{
  "tools": [
    "file_read",
    "file_write",
    "file_list",
    "web_fetch",
    "web_search",
    "shell_exec",
    "kv_get",
    "kv_set",
    "agent_call"
  ],
  "total": 23
}
```

### GET /api/config

Получить текущую конфигурацию ядра (секреты скрыты).

**Ответ** `200 OK`:

```json
{
  "data_dir": "/home/user/.openfang/data",
  "default_provider": "groq",
  "default_model": "llama-3.3-70b-versatile",
  "listen_addr": "127.0.0.1:4200",
  "api_key_set": true,
  "channels_configured": 2,
  "mcp_servers": 1
}
```

### GET /api/peers

Список пиров сети OFP (OpenFang Protocol) и статус их подключения.

**Ответ** `200 OK`:

```json
{
  "peers": [
    {
      "node_id": "peer-1",
      "address": "192.168.1.100:4000",
      "state": "connected",
      "authenticated": true,
      "last_seen": "2025-01-15T10:30:00Z"
    }
  ]
}
```

### GET /api/sessions

Список всех активных сессий всех агентов.

**Ответ** `200 OK`:

```json
{
  "sessions": [
    {
      "id": "s1b2c3d4-...",
      "agent_id": "a1b2c3d4-...",
      "agent_name": "coder",
      "message_count": 12,
      "created_at": "2025-01-15T10:30:00Z"
    }
  ]
}
```

### DELETE /api/sessions/{id}

Удалить конкретную сессию и ее историю переписки.

**Ответ** `200 OK`:

```json
{
  "status": "deleted",
  "session_id": "s1b2c3d4-..."
}
```

---

## Эндпоинты каталога моделей (Model Catalog)

OpenFang поддерживает встроенный каталог из 51+ модели от 20 провайдеров. Эти эндпоинты позволяют просматривать доступные модели, проверять статус аутентификации провайдеров и разрешать алиасы моделей.

### GET /api/models

Список всего каталога моделей. Возвращает все известные модели с их провайдером, уровнем (tier), окном контекста и информацией о ценообразовании.

**Ответ** `200 OK`:

```json
{
  "models": [
    {
      "id": "claude-sonnet-4-20250514",
      "provider": "anthropic",
      "display_name": "Claude Sonnet 4",
      "tier": "frontier",
      "context_window": 200000,
      "input_cost_per_1m": 3.0,
      "output_cost_per_1m": 15.0,
      "supports_tools": true,
      "supports_vision": true,
      "supports_streaming": true
    },
    {
      "id": "gemini-2.5-flash",
      "provider": "gemini",
      "display_name": "Gemini 2.5 Flash",
      "tier": "smart",
      "context_window": 1048576,
      "input_cost_per_1m": 0.15,
      "output_cost_per_1m": 0.6,
      "supports_tools": true,
      "supports_vision": true,
      "supports_streaming": true
    }
  ],
  "total": 51
}
```

### GET /api/models/{id}

Получить подробную информацию о конкретной модели.

**Ответ** `200 OK`:

```json
{
  "id": "llama-3.3-70b-versatile",
  "provider": "groq",
  "display_name": "Llama 3.3 70B",
  "tier": "fast",
  "context_window": 131072,
  "input_cost_per_1m": 0.59,
  "output_cost_per_1m": 0.79,
  "supports_tools": true,
  "supports_vision": false,
  "supports_streaming": true
}
```

**Ответ** `404 Not Found`:

```json
{
  "error": "Model 'unknown-model' not found in catalog"
}
```

### GET /api/models/aliases

Список всех алиасов моделей. Алиасы предоставляют короткие имена, которые разрешаются в полные ID моделей (например, `sonnet` разрешается в `claude-sonnet-4-20250514`).

**Ответ** `200 OK`:

```json
{
  "aliases": {
    "sonnet": "claude-sonnet-4-20250514",
    "opus": "claude-opus-4-20250514",
    "haiku": "claude-3-5-haiku-20241022",
    "flash": "gemini-2.5-flash",
    "gpt4": "gpt-4o",
    "llama": "llama-3.3-70b-versatile",
    "deepseek": "deepseek-chat",
    "grok": "grok-2",
    "jamba": "jamba-1.5-large"
  },
  "total": 23
}
```

### GET /api/providers

Список всех известных провайдеров LLM и статус их аутентификации. Статус аутентификации определяется путем проверки наличия переменных окружения (секретные значения никогда не считываются).

**Ответ** `200 OK`:

```json
{
  "providers": [
    {
      "name": "anthropic",
      "display_name": "Anthropic",
      "auth_status": "configured",
      "env_var": "ANTHROPIC_API_KEY",
      "base_url": "https://api.anthropic.com",
      "model_count": 3
    },
    {
      "name": "groq",
      "display_name": "Groq",
      "auth_status": "configured",
      "env_var": "GROQ_API_KEY",
      "base_url": "https://api.groq.com/openai",
      "model_count": 4
    },
    {
      "name": "ollama",
      "display_name": "Ollama",
      "auth_status": "no_key_needed",
      "base_url": "http://localhost:11434",
      "model_count": 0
    }
  ],
  "total": 20
}
```

---

## Эндпоинты конфигурации провайдеров

Управляйте API-ключами провайдеров LLM во время выполнения без редактирования конфигурационных файлов или перезапуска демона.

### POST /api/providers/{name}/key

Установить API-ключ для провайдера. Ключ хранится безопасно и вступает в силу немедленно.

**Тело запроса**:

```json
{
  "api_key": "sk-..."
}
```

**Ответ** `200 OK`:

```json
{
  "status": "configured",
  "provider": "anthropic"
}
```

### DELETE /api/providers/{name}/key

Удалить API-ключ для провайдера. Агенты, использующие этого провайдера, перейдут на FallbackDriver или завершатся с ошибкой.

**Ответ** `200 OK`:

```json
{
  "status": "removed",
  "provider": "anthropic"
}
```

### POST /api/providers/{name}/test

Проверить соединение с провайдером, сделав минимальный вызов API. Проверяет, что настроенный API-ключ действителен и эндпоинт провайдера доступен.

**Ответ** `200 OK`:

```json
{
  "status": "ok",
  "provider": "anthropic",
  "latency_ms": 245,
  "model_tested": "claude-sonnet-4-20250514"
}
```

**Ответ** `401 Unauthorized`:

```json
{
  "status": "failed",
  "provider": "anthropic",
  "error": "Invalid API key"
}
```

---

## Эндпоинты навыков и маркетплейса (Skills & Marketplace)

Управление реестром навыков. Навыки расширяют возможности агентов с помощью модулей на Python, Node.js, WASM или текстовых модулей (prompt-only). Все установки навыков проходят проверку SHA256 и сканирование на наличие промпт-инъекций.

### GET /api/skills

Список всех установленных навыков.

**Ответ** `200 OK`:

```json
{
  "skills": [
    {
      "name": "github",
      "version": "1.0.0",
      "runtime": "prompt_only",
      "description": "GitHub integration for issues, PRs, and repos",
      "bundled": true
    },
    {
      "name": "docker",
      "version": "1.0.0",
      "runtime": "prompt_only",
      "description": "Docker container management",
      "bundled": true
    }
  ],
  "total": 60
}
```

### POST /api/skills/install

Установить навык из локального пути или по URL. Манифест навыка проверяется (контрольная сумма SHA256) и сканируется на наличие промпт-инъекций перед установкой.

**Тело запроса**:

```json
{
  "source": "/path/to/skill",
  "verify": true
}
```

**Ответ** `201 Created`:

```json
{
  "status": "installed",
  "skill": "my-custom-skill",
  "version": "1.0.0"
}
```

### POST /api/skills/uninstall

Удалить установленный навык. Встроенные (bundled) навыки нельзя удалить.

**Тело запроса**:

```json
{
  "name": "my-custom-skill"
}
```

**Ответ** `200 OK`:

```json
{
  "status": "uninstalled",
  "skill": "my-custom-skill"
}
```

### POST /api/skills/create

Создать новый навык из шаблона.

**Тело запроса**:

```json
{
  "name": "my-skill",
  "runtime": "python",
  "description": "A custom skill"
}
```

**Ответ** `201 Created`:

```json
{
  "status": "created",
  "skill": "my-skill",
  "path": "/home/user/.openfang/skills/my-skill"
}
```

### GET /api/marketplace/search

Поиск в маркетплейсе FangHub по навыкам сообщества.

**Параметры запроса:**
- `q` (обязательно): Строка поискового запроса
- `page` (опционально): Номер страницы (по умолчанию: 1)

**Ответ** `200 OK`:

```json
{
  "results": [
    {
      "name": "weather-api",
      "author": "community",
      "description": "Real-time weather data integration",
      "downloads": 1250,
      "version": "2.1.0"
    }
  ],
  "total": 1,
  "page": 1
}
```

---

## Эндпоинты ClawHub

Просмотр и установка навыков из ClawHub (совместимость с экосистемой OpenClaw). Все установки проходят через полный конвейер безопасности: проверку SHA256, сканирование безопасности SKILL.md и применение границ доверия.

### GET /api/clawhub/search

Поиск в ClawHub по совместимым навыкам.

**Параметры запроса:**
- `q` (обязательно): Поисковый запрос

**Ответ** `200 OK`:

```json
{
  "results": [
    {
      "slug": "data-pipeline",
      "name": "Data Pipeline",
      "description": "ETL data pipeline automation",
      "author": "clawhub-community",
      "version": "1.2.0"
    }
  ],
  "total": 1
}
```

### GET /api/clawhub/browse

Просмотр категорий ClawHub.

**Параметры запроса:**
- `category` (опционально): Фильтр по категории
- `page` (опционально): Номер страницы (по умолчанию: 1)

**Ответ** `200 OK`:

```json
{
  "skills": [
    {
      "slug": "data-pipeline",
      "name": "Data Pipeline",
      "category": "data",
      "description": "ETL data pipeline automation"
    }
  ],
  "total": 15,
  "page": 1
}
```

### GET /api/clawhub/skill/{slug}

Получить подробную информацию о конкретном навыке ClawHub.

**Ответ** `200 OK`:

```json
{
  "slug": "data-pipeline",
  "name": "Data Pipeline",
  "description": "ETL data pipeline automation",
  "author": "clawhub-community",
  "version": "1.2.0",
  "runtime": "python",
  "readme": "# Data Pipeline\n\nAutomated ETL...",
  "sha256": "a1b2c3d4..."
}
```

### POST /api/clawhub/install

Установить навык из ClawHub. Автоматически скачивает, проверяет контрольную сумму SHA256, сканирует на промпт-инъекции и конвертирует формат SKILL.md в skill.toml OpenFang.

**Тело запроса**:

```json
{
  "slug": "data-pipeline"
}
```

**Ответ** `201 Created`:

```json
{
  "status": "installed",
  "skill": "data-pipeline",
  "version": "1.2.0",
  "converted_from": "SKILL.md"
}
```

---

## Эндпоинты протоколов MCP и A2A

OpenFang поддерживает как Model Context Protocol (MCP) для интероперабельности инструментов, так и протокол Agent-to-Agent (A2A) для взаимодействия агентов между различными системами.

### GET /api/mcp/servers

Список настроенных и подключенных MCP-серверов с их доступными инструментами.

**Ответ** `200 OK`:

```json
{
  "servers": [
    {
      "name": "filesystem",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "connected": true,
      "tools": [
        {
          "name": "mcp_filesystem_read_file",
          "description": "Read a file from the filesystem"
        },
        {
          "name": "mcp_filesystem_write_file",
          "description": "Write content to a file"
        }
      ]
    }
  ],
  "total": 1
}
```

### POST /mcp

Эндпоинт HTTP-транспорта MCP. Принимает запросы JSON-RPC 2.0 и предоставляет инструменты OpenFang через протокол MCP внешним клиентам.

**Тело запроса** (JSON-RPC 2.0):

```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}
```

**Ответ** `200 OK`:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "tools": [
      {
        "name": "file_read",
        "description": "Read a file's contents",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {"type": "string"}
          }
        }
      }
    ]
  },
  "id": 1
}
```

### GET /.well-known/agent.json

Эндпоинт обнаружения карточки агента A2A. Возвращает карточку агента A2A сервера, которая описывает его возможности, поддерживаемые протоколы и доступных агентов.

**Ответ** `200 OK`:

```json
{
  "name": "OpenFang",
  "description": "OpenFang Agent Operating System",
  "url": "http://127.0.0.1:4200",
  "version": "0.1.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false
  },
  "skills": [
    {
      "id": "chat",
      "name": "Chat",
      "description": "General-purpose chat with any agent"
    }
  ]
}
```

### GET /a2a/agents

Список агентов, доступных по протоколу A2A.

**Ответ** `200 OK`:

```json
{
  "agents": [
    {
      "id": "a1b2c3d4-...",
      "name": "coder",
      "description": "Expert coding assistant",
      "skills": ["code-review", "debugging", "refactoring"]
    }
  ]
}
```

### POST /a2a/tasks/send

Отправить задачу агенту по протоколу A2A. Соответствует спецификации Google A2A для делегирования задач между агентами.

**Тело запроса**:

```json
{
  "agent_id": "a1b2c3d4-...",
  "message": {
    "role": "user",
    "parts": [
      {"text": "Review this code for security issues"}
    ]
  }
}
```

**Ответ** `200 OK`:

```json
{
  "task_id": "task-1234-...",
  "status": "completed",
  "result": {
    "role": "agent",
    "parts": [
      {"text": "I found 2 potential security issues..."}
    ]
  }
}
```

### GET /a2a/tasks/{id}

Получить статус и результат задачи A2A.

**Ответ** `200 OK`:

```json
{
  "task_id": "task-1234-...",
  "status": "completed",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:30:05Z",
  "result": {
    "role": "agent",
    "parts": [
      {"text": "Analysis complete..."}
    ]
  }
}
```

### POST /a2a/tasks/{id}/cancel

Отменить выполняющуюся задачу A2A.

**Ответ** `200 OK`:

```json
{
  "task_id": "task-1234-...",
  "status": "cancelled"
}
```

---

## Эндпоинты аудита и безопасности

OpenFang поддерживает журнал аудита в виде цепочки хешей Меркла для всех операций, имеющих отношение к безопасности. Эти эндпоинты позволяют просматривать и проверять целостность журнала аудита.

### GET /api/audit/recent

Получить последние записи журнала аудита.

**Параметры запроса:**
- `limit` (опционально): Количество возвращаемых записей (по умолчанию: 50, макс: 500)

**Ответ** `200 OK`:

```json
{
  "entries": [
    {
      "id": 1042,
      "timestamp": "2025-01-15T10:30:00Z",
      "event_type": "agent_spawned",
      "agent_id": "a1b2c3d4-...",
      "details": "Agent 'coder' spawned with model groq/llama-3.3-70b-versatile",
      "hash": "a1b2c3d4e5f6...",
      "prev_hash": "f6e5d4c3b2a1..."
    }
  ],
  "total": 1042
}
```

### GET /api/audit/verify

Проверить целостность журнала аудита на основе цепочки хешей Меркла. Проходит по всей цепочке и сообщает о любых разорванных звеньях.

**Ответ** `200 OK`:

```json
{
  "status": "valid",
  "chain_length": 1042,
  "first_entry": "2025-01-10T08:00:00Z",
  "last_entry": "2025-01-15T10:30:00Z"
}
```

**Ответ** `200 OK` (цепочка разорвана):

```json
{
  "status": "broken",
  "chain_length": 1042,
  "break_at": 847,
  "error": "Hash mismatch at entry 847"
}
```

### GET /api/security

Обзор состояния безопасности, показывающий состояние всех 16 систем безопасности.

**Ответ** `200 OK`:

```json
{
  "security_systems": {
    "merkle_audit_trail": "active",
    "taint_tracking": "active",
    "wasm_dual_metering": "active",
    "security_headers": "active",
    "health_redaction": "active",
    "subprocess_sandbox": "active",
    "manifest_signing": "active",
    "gcra_rate_limiter": "active",
    "secret_zeroization": "active",
    "path_traversal_prevention": "active",
    "ssrf_protection": "active",
    "capability_inheritance_validation": "active",
    "ofp_hmac_auth": "active",
    "prompt_injection_scanning": "active",
    "loop_guard": "active",
    "session_repair": "active"
  },
  "total_systems": 16,
  "all_active": true
}
```

---

## Эндпоинты использования и аналитики (Usage & Analytics)

Отслеживание использования токенов, затрат и загрузки моделей по всем агентам. Работает на базе движка учета с оценкой стоимости из каталога моделей.

### GET /api/usage

Получить общую статистику использования.

**Параметры запроса:**
- `period` (опционально): Период времени (`hour`, `day`, `week`, `month`; по умолчанию: `day`)

**Ответ** `200 OK`:

```json
{
  "period": "day",
  "total_input_tokens": 125000,
  "total_output_tokens": 87000,
  "total_cost_usd": 0.42,
  "request_count": 156,
  "active_agents": 5
}
```

### GET /api/usage/summary

Получить высокоуровневую сводку использования с информацией о квотах.

**Ответ** `200 OK`:

```json
{
  "today": {
    "input_tokens": 125000,
    "output_tokens": 87000,
    "cost_usd": 0.42,
    "requests": 156
  },
  "quota": {
    "hourly_token_limit": 1000000,
    "hourly_tokens_used": 45000,
    "hourly_reset_at": "2025-01-15T11:00:00Z"
  }
}
```

### GET /api/usage/by-model

Получить разбивку использования по моделям.

**Ответ** `200 OK`:

```json
{
  "models": [
    {
      "model": "llama-3.3-70b-versatile",
      "provider": "groq",
      "input_tokens": 80000,
      "output_tokens": 55000,
      "cost_usd": 0.09,
      "request_count": 120
    },
    {
      "model": "gemini-2.5-flash",
      "provider": "gemini",
      "input_tokens": 45000,
      "output_tokens": 32000,
      "cost_usd": 0.33,
      "request_count": 36
    }
  ]
}
```

---

## Эндпоинты миграции (Migration)

Импорт данных из OpenClaw или других фреймворков агентов. Движок миграции обрабатывает конвертацию манифестов из YAML в TOML, парсинг SKILL.md и импорт истории сессий.

### GET /api/migrate/detect

Автоматическое обнаружение источников миграции в системе. Сканирует стандартные места на наличие установок OpenClaw, конфигурационных файлов и данных агентов.

**Ответ** `200 OK`:

```json
{
  "sources": [
    {
      "type": "openclaw",
      "path": "/home/user/.openclaw",
      "version": "2.1.0",
      "agents_found": 12,
      "skills_found": 8
    }
  ]
}
```

### POST /api/migrate/scan

Сканировать конкретный путь на наличие данных для импорта.

**Тело запроса**:

```json
{
  "path": "/home/user/.openclaw"
}
```

**Ответ** `200 OK`:

```json
{
  "agents": [
    {
      "name": "my-agent",
      "format": "yaml",
      "convertible": true
    }
  ],
  "skills": [
    {
      "name": "custom-skill",
      "format": "SKILL.md",
      "convertible": true
    }
  ],
  "sessions": 45
}
```

### POST /api/migrate

Запустить миграцию. Конвертирует манифесты, импортирует навыки и опционально импортирует историю сессий.

**Тело запроса**:

```json
{
  "source": "/home/user/.openclaw",
  "import_agents": true,
  "import_skills": true,
  "import_sessions": false
}
```

**Ответ** `200 OK`:

```json
{
  "status": "completed",
  "agents_imported": 12,
  "skills_imported": 8,
  "sessions_imported": 0,
  "warnings": [
    "Skill 'legacy-plugin' uses unsupported runtime 'ruby', skipped"
  ]
}
```

---

## Эндпоинты управления сессиями

### POST /api/agents/{id}/session/reset

Сбросить сессию агента, удалив всю историю переписки.

**Ответ** `200 OK`:

```json
{
  "status": "reset",
  "agent_id": "a1b2c3d4-...",
  "new_session_id": "s5e6f7g8-..."
}
```

### POST /api/agents/{id}/session/compact

Запустить сжатие сессии на базе LLM. Переписка агента резюмируется с помощью LLM, сохраняются только самые последние сообщения плюс сгенерированное резюме.

**Ответ** `200 OK`:

```json
{
  "status": "compacted",
  "message": "Session compacted: 80 messages summarized, 20 kept"
}
```

**Ответ** `200 OK` (сжатие не требуется):

```json
{
  "status": "ok",
  "message": "Session does not need compaction (below threshold)"
}
```

### POST /api/agents/{id}/stop

Отменить текущий запуск LLM для агента. Прерывает любую выполняющуюся генерацию.

**Ответ** `200 OK`:

```json
{
  "status": "stopped",
  "message": "Agent run cancelled"
}
```

### PUT /api/agents/{id}/model

Переключить модель LLM агента во время выполнения.

**Тело запроса**:

```json
{
  "model": "claude-sonnet-4-20250514"
}
```

**Ответ** `200 OK`:

```json
{
  "status": "updated",
  "model": "claude-sonnet-4-20250514"
}
```

---

## Эндпоинты Cron/Планировщика

Управление повторяющимися и разовыми запланированными задачами. Задачи могут запускать ходы агентов, системные события или воркфлоу по расписанию.

### GET /api/cron/jobs

Список всех задач cron. Опционально фильтруется по агенту: `?agent_id=<uuid>`.

**Ответ** `200 OK`:

```json
{
  "jobs": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "agent_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
      "name": "daily-report",
      "enabled": true,
      "schedule": { "kind": "every", "every_secs": 3600 },
      "action": {
        "kind": "agent_turn",
        "message": "Generate the daily report",
        "timeout_secs": 120
      },
      "delivery": {
        "kind": "channel",
        "channel": "slack",
        "to": "#reports"
      },
      "created_at": "2026-03-15T10:30:00Z",
      "last_run": "2026-03-16T09:00:00Z",
      "next_run": "2026-03-16T10:00:00Z"
    }
  ],
  "total": 1
}
```

### POST /api/cron/jobs

Создать новую задачу cron.

**Тело запроса**:

```json
{
  "agent_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "name": "daily-report",
  "schedule": { "kind": "every", "every_secs": 3600 },
  "action": {
    "kind": "agent_turn",
    "message": "Generate the daily report",
    "timeout_secs": 120
  },
  "delivery": {
    "kind": "channel",
    "channel": "slack",
    "to": "#reports"
  }
}
```

**Ответ** `201 Created`:

```json
{
  "result": "{\"job_id\":\"550e8400-e29b-41d4-a716-446655440000\",\"status\":\"created\"}"
}
```

### DELETE /api/cron/jobs/{id}

Удалить задачу cron по ID.

**Ответ** `200 OK`:

```json
{ "status": "deleted" }
```

### PUT /api/cron/jobs/{id}/enable

Включить или выключить задачу cron.

**Тело запроса**:

```json
{ "enabled": false }
```

**Ответ** `200 OK`:

```json
{ "status": "updated", "enabled": false }
```

### GET /api/cron/jobs/{id}/status

Получить метаданные задачи, включая время последнего запуска, статус и историю ошибок.

**Ответ** `200 OK`:

```json
{
  "job": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "agent_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "name": "daily-report",
    "enabled": true,
    "schedule": { "kind": "every", "every_secs": 3600 },
    "action": {
      "kind": "agent_turn",
      "message": "Generate the daily report",
      "timeout_secs": 120
    },
    "delivery": { "kind": "none" },
    "created_at": "2026-03-15T10:30:00Z",
    "last_run": "2026-03-16T09:00:00Z",
    "next_run": "2026-03-16T10:00:00Z"
  },
  "one_shot": false,
  "last_status": "ok",
  "consecutive_errors": 0
}
```

### POST /api/cron/jobs/{id}/run

Запустить задачу cron немедленно. Задача выполняется асинхронно в фоновом режиме — этот эндпоинт возвращает ответ сразу, не дожидаясь завершения. Опрашивайте `GET /api/cron/jobs/{id}/status`, чтобы проверить результат.

**Ответ** `200 OK`:

```json
{
  "status": "triggered",
  "job_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Ответы с ошибками**:

- `400 Bad Request` — Неверный ID задачи или задача отключена
- `404 Not Found` — Задача не найдена

---

## Протокол WebSocket

### Подключение

```
GET /api/agents/{id}/ws
```

Переключает соединение на WebSocket для двустороннего чата с агентом в реальном времени. Возвращает `400`, если ID агента невалиден, или `404`, если агент не существует.

### Формат сообщений

Все сообщения представляют собой строки в кодировке JSON.

### От клиента к серверу

**Отправить сообщение:**

```json
{
  "type": "message",
  "content": "Какая сейчас погода?"
}
```

Обычный текст (не JSON) также принимается и обрабатывается как сообщение.

**Команды чата** (отправляются как сообщения с префиксом `/`):

| Команда | Описание |
|---------|-------------|
| `/new` | Начать новую сессию (очистить историю) |
| `/compact` | Запустить LLM-сжатие сессии |
| `/model <name>` | Переключить модель агента |
| `/stop` | Отменить текущий запуск LLM |
| `/usage` | Показать использование токенов и стоимость |
| `/think` | Переключить режим расширенного размышления |
| `/models` | Список доступных моделей |
| `/providers` | Список провайдеров LLM и статус их аутентификации |

**Ping:**

```json
{
  "type": "ping"
}
```

### От сервера к клиенту

**Подключение подтверждено** (отправляется сразу после подключения):

```json
{
  "type": "connected",
  "agent_id": "a1b2c3d4-..."
}
```

**Индикатор размышления** (отправляется, когда агент начинает обработку):

```json
{
  "type": "thinking"
}
```

**Дельта текста** (потоковый токен, отправляется по мере генерации вывода LLM):

```json
{
  "type": "text_delta",
  "content": "Погода"
}
```

**Запуск использования инструмента** (отправляется, когда агент вызывает инструмент):

```json
{
  "type": "tool_start",
  "tool": "web_fetch"
}
```

**Полный ответ** (отправляется по завершении работы агента, содержит итоговый агрегированный ответ):

```json
{
  "type": "response",
  "content": "Сегодня солнечная погода, максимум 22°C.",
  "input_tokens": 245,
  "output_tokens": 32,
  "iterations": 2,
  "cost_usd": 0.0012
}
```

**Ошибка:**

```json
{
  "type": "error",
  "content": "Agent not found"
}
```

**Обновление списка агентов** (отправляется каждые 5 секунд с текущими состояниями агентов):

```json
{
  "type": "agents_updated",
  "agents": [
    {
      "id": "a1b2c3d4-...",
      "name": "hello-world",
      "state": "Running",
      "model_provider": "groq",
      "model_name": "llama-3.3-70b-versatile"
    }
  ]
}
```

**Pong** (ответ на ping):

```json
{
  "type": "pong"
}
```

### Жизненный цикл соединения

1. Клиент подключается к `ws://host:port/api/agents/{id}/ws`.
2. Сервер отправляет `{"type": "connected"}`.
3. Клиент отправляет `{"type": "message", "content": "..."}`.
4. Сервер отправляет `{"type": "thinking"}`, затем ноль или более событий `{"type": "text_delta"}`, затем `{"type": "response"}`.
5. Сервер периодически отправляет `{"type": "agents_updated"}` каждые 5 секунд.
6. Клиент отправляет фрейм Close или отключается для завершения сессии.

---

## SSE-стриминг (Server-Sent Events)

### POST /api/agents/{id}/message/stream

Отправить сообщение и получить ответ в виде потока Server-Sent Events. Это позволяет реализовать потоковую передачу текста по токенам в реальном времени.

**Тело запроса** (JSON):

```json
{
  "message": "Объясни квантовые вычисления"
}
```

**Поток событий SSE:**

```
event: chunk
data: {"content":"Квантовые","done":false}

event: chunk
data: {"content":" вычисления","done":false}

event: chunk
data: {"content":" — это тип","done":false}

event: tool_use
data: {"tool":"web_search"}

event: tool_result
data: {"tool":"web_search","input":{"query":"основы квантовых вычислений"}}

event: done
data: {"done":true,"usage":{"input_tokens":150,"output_tokens":340}}
```

### Типы событий SSE

| Имя события | Описание |
|------------|-------------|
| `chunk` | Дельта текста от LLM. `"done": false` указывает на то, что ожидаются еще токенов. |
| `tool_use` | Агент вызывает инструмент. Содержит имя инструмента. |
| `tool_result` | Вызов инструмента завершен. Содержит имя инструмента и входные данные. |
| `done` | Финальное событие. Содержит `"done": true` и статистику использования токенов. |

---

## OpenAI-совместимый API

OpenFang предоставляет OpenAI-совместимый API для бесшовной интеграции с инструментами, поддерживающими формат OpenAI API (Cursor, Continue, Open WebUI и т. д.).

### POST /v1/chat/completions

Отправить запрос chat completion, используя формат сообщений OpenAI.

**Тело запроса**:

```json
{
  "model": "openfang:coder",
  "messages": [
    {"role": "system", "content": "Вы — полезный помощник."},
    {"role": "user", "content": "Привет!"}
  ],
  "stream": false,
  "temperature": 0.7,
  "max_tokens": 1024
}
```

**Разрешение моделей** (поле `model` сопоставляется с агентом OpenFang):

| Формат | Пример | Поведение |
|--------|---------|----------|
| `openfang:<name>` | `openfang:coder` | Поиск агента по имени |
| UUID | `a1b2c3d4-...` | Поиск агента по ID |
| Простая строка | `coder` | Попытка поиска по имени агента |
| Любой другой | `gpt-4o` | Возврат к первому зарегистрированному агенту |

**Поддержка изображений** — сообщения могут включать части с контентом изображений:

```json
{
  "model": "openfang:analyst",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "Опиши это изображение"},
        {"type": "image_url", "image_url": {"url": "data:image/png;base64,iVBOR..."}}
      ]
    }
  ]
}
```

**Ответ (непотоковый)** `200 OK`:

```json
{
  "id": "chatcmpl-a1b2c3d4-...",
  "object": "chat.completion",
  "created": 1708617600,
  "model": "coder",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Привет! Чем я могу вам сегодня помочь?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 12,
    "total_tokens": 37
  }
}
```

**Потоковая передача** — установите `"stream": true` для SSE:

```
data: {"id":"chatcmpl-...","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"role":"assistant","content":"Привет"},"finish_reason":null}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":"!"},"finish_reason":null}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","choices":[{"index":0,"delta":{},"finish_reason":"stop"}],"usage":{"prompt_tokens":25,"completion_tokens":12,"total_tokens":37}}

data: [DONE]
```

### GET /v1/models

Список доступных моделей (агентов) в формате OpenAI.

**Ответ** `200 OK`:

```json
{
  "object": "list",
  "data": [
    {
      "id": "openfang:coder",
      "object": "model",
      "created": 1708617600,
      "owned_by": "openfang"
    },
    {
      "id": "openfang:researcher",
      "object": "model",
      "created": 1708617600,
      "owned_by": "openfang"
    }
  ]
}
```

---

## Ответы с ошибками

Все ответы с ошибками используют единообразный формат JSON:

```json
{
  "error": "Описание того, что пошло не так"
}
```

### Коды состояния HTTP

| Код | Значение |
|------|---------|
| `200` | Успех |
| `201` | Создано (запуск агента, создание воркфлоу, создание триггера, установка навыка) |
| `400` | Неверный запрос (некорректный UUID, отсутствие обязательных полей, неверный формат TOML/JSON) |
| `401` | Не авторизован (отсутствует или недействителен заголовок `Authorization: Bearer`) |
| `404` | Не найдено (агент, воркфлоу, триггер, шаблон, модель, навык или ключ KV не существуют) |
| `429` | Слишком много запросов (превышен лимит скорости GCRA) |
| `500` | Внутренняя ошибка сервера (сбой цикла агента, ошибка базы данных, ошибка драйвера) |

### ID запросов (Request IDs)

Каждый ответ включает заголовок `x-request-id` с UUID для трассировки:

```
x-request-id: 550e8400-e29b-41d4-a716-446655440000
```

Используйте это значение при сообщении о проблемах или сопоставлении запросов в логах.

### Заголовки безопасности

Каждый ответ включает заголовки безопасности:

| Заголовок | Значение |
|--------|-------|
| `Content-Security-Policy` | `default-src 'self'` (с соответствующими директивами) |
| `X-Frame-Options` | `DENY` |
| `X-Content-Type-Options` | `nosniff` |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains` |
| `X-Request-Id` | Уникальный UUID на запрос |

### Ограничение скорости (Rate Limiting)

Rate limiter GCRA (Generic Cell Rate Algorithm) обеспечивает ограничение скорости на основе корзин токенов с учетом стоимости запросов, отслеживанием по IP и автоматической очисткой устаревших записей. Разные эндпоинты потребляют разное количество токенов (например, `/api/agents/{id}/message` стоит дороже, чем `/api/health`). При превышении лимита сервер возвращает `429 Too Many Requests`:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{"error": "Rate limit exceeded"}
```

Заголовок `Retry-After` указывает длительность окна в секундах.

---

## Сводка эндпоинтов

**Всего 76 эндпоинтов** в 15 группах.

| Метод | Путь | Описание |
|--------|------|-------------|
| **Система** | | |
| GET | `/` | Интерфейс WebChat |
| GET | `/api/health` | Проверка здоровья (без auth, сокращенно) |
| GET | `/api/health/detail` | Полная проверка здоровья (требуется auth) |
| GET | `/api/status` | Статус ядра |
| GET | `/api/version` | Информация о версии |
| POST | `/api/shutdown` | Корректное завершение работы |
| GET | `/api/profiles` | Список профилей агентов |
| GET | `/api/tools` | Список доступных инструментов |
| GET | `/api/config` | Конфигурация (секреты скрыты) |
| GET | `/api/peers` | Список пиров сети OFP |
| **Агенты** | | |
| GET | `/api/agents` | Список агентов |
| POST | `/api/agents` | Запустить агента |
| GET | `/api/agents/{id}` | Детали агента |
| PUT | `/api/agents/{id}/update` | Обновить конфиг агента |
| PUT | `/api/agents/{id}/mode` | Установить режим агента (Stable/Normal) |
| DELETE | `/api/agents/{id}` | Завершить работу агента |
| POST | `/api/agents/{id}/message` | Отправить сообщение (блокирующий вызов) |
| POST | `/api/agents/{id}/message/stream` | Отправить сообщение (поток SSE) |
| GET | `/api/agents/{id}/session` | Получить историю переписки |
| GET | `/api/agents/{id}/ws` | Чат через WebSocket |
| POST | `/api/agents/{id}/session/reset` | Сбросить сессию |
| POST | `/api/agents/{id}/session/compact` | Сжатие на базе LLM |
| POST | `/api/agents/{id}/stop` | Отменить текущий запуск |
| PUT | `/api/agents/{id}/model` | Переключить модель |
| **Воркфлоу** | | |
| GET | `/api/workflows` | Список воркфлоу |
| POST | `/api/workflows` | Создать воркфлоу |
| POST | `/api/workflows/{id}/run` | Запустить воркфлоу |
| GET | `/api/workflows/{id}/runs` | Список запусков воркфлоу |
| **Триггеры** | | |
| GET | `/api/triggers` | Список триггеров |
| POST | `/api/triggers` | Создать триггер |
| PUT | `/api/triggers/{id}` | Обновить триггер |
| DELETE | `/api/triggers/{id}` | Удалить триггер |
| **Память** | | |
| GET | `/api/memory/agents/{id}/kv` | Список пар KV |
| GET | `/api/memory/agents/{id}/kv/{key}` | Получить значение KV |
| PUT | `/api/memory/agents/{id}/kv/{key}` | Установить значение KV |
| DELETE | `/api/memory/agents/{id}/kv/{key}` | Удалить значение KV |
| **Каналы** | | |
| GET | `/api/channels` | Список каналов (40 адаптеров) |
| **Шаблоны** | | |
| GET | `/api/templates` | Список шаблонов |
| GET | `/api/templates/{name}` | Получить шаблон |
| **Сессии** | | |
| GET | `/api/sessions` | Список сессий |
| DELETE | `/api/sessions/{id}` | Удалить сессию |
| **Каталог моделей** | | |
| GET | `/api/models` | Полный каталог (51+ модель) |
| GET | `/api/models/{id}` | Детали модели |
| GET | `/api/models/aliases` | Список 23 алиасов моделей |
| GET | `/api/providers` | Список провайдеров со статусом auth |
| **Конфиг провайдеров** | | |
| POST | `/api/providers/{name}/key` | Установить API-ключ провайдера |
| DELETE | `/api/providers/{name}/key` | Удалить API-ключ провайдера |
| POST | `/api/providers/{name}/test` | Проверить соединение с провайдером |
| **Навыки и маркетплейс** | | |
| GET | `/api/skills` | Список навыков (60 встроенных) |
| POST | `/api/skills/install` | Установить навык |
| POST | `/api/skills/uninstall` | Удалить навык |
| POST | `/api/skills/create` | Создать новый навык |
| GET | `/api/marketplace/search` | Поиск в FangHub |
| **ClawHub** | | |
| GET | `/api/clawhub/search` | Поиск в ClawHub |
| GET | `/api/clawhub/browse` | Просмотр ClawHub |
| GET | `/api/clawhub/skill/{slug}` | Детали навыка |
| POST | `/api/clawhub/install` | Установить из ClawHub |
| **MCP и A2A** | | |
| GET | `/api/mcp/servers` | Соединения с MCP-серверами |
| POST | `/mcp` | HTTP-транспорт MCP (JSON-RPC 2.0) |
| GET | `/.well-known/agent.json` | Карточка агента A2A |
| GET | `/a2a/agents` | Список агентов A2A |
| POST | `/a2a/tasks/send` | Отправить задачу A2A |
| GET | `/a2a/tasks/{id}` | Статус задачи A2A |
| POST | `/a2a/tasks/{id}/cancel` | Отменить задачу A2A |
| **Аудит и безопасность** | | |
| GET | `/api/audit/recent` | Последние логи аудита |
| GET | `/api/audit/verify` | Проверить цепочку Меркла |
| GET | `/api/security` | Статус безопасности (16 систем) |
| **Использование и аналитика** | | |
| GET | `/api/usage` | Статистика использования |
| GET | `/api/usage/summary` | Сводка использования с квотой |
| GET | `/api/usage/by-model` | Использование с разбивкой по моделям |
| **Миграция** | | |
| GET | `/api/migrate/detect` | Обнаружить источники миграции |
| POST | `/api/migrate/scan` | Сканировать на наличие данных |
| POST | `/api/migrate` | Запустить миграцию |
| **OpenAI Compatible** | | |
| POST | `/v1/chat/completions` | OpenAI-совместимый чат |
| GET | `/v1/models` | Список моделей (OpenAI формат) |
