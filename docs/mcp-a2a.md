# Руководство по интеграции MCP и A2A

OpenFang реализует как **Model Context Protocol (MCP)**, так и протокол **Agent-to-Agent (A2A)**, обеспечивая глубокую совместимость с внешними инструментами, IDE и другими фреймворками агентов.

---

## Содержание

- [Часть 1: MCP (Model Context Protocol)](#part-1-mcp-model-context-protocol)
  - [Обзор](#mcp-overview)
  - [Клиент MCP — подключение к внешним серверам](#mcp-client)
  - [Сервер MCP — предоставление доступа к OpenFang через MCP](#mcp-server)
  - [Примеры конфигурации](#mcp-configuration-examples)
  - [Конечные точки API](#mcp-api-endpoints)
- [Часть 2: A2A (Agent-to-Agent Protocol)](#part-2-a2a-agent-to-agent-protocol)
  - [Обзор](#a2a-overview)
  - [Карточка агента (Agent Card)](#agent-card)
  - [Сервер A2A](#a2a-server)
  - [Клиент A2A](#a2a-client)
  - [Жизненный цикл задачи](#task-lifecycle)
  - [Конечные точки API](#a2a-api-endpoints)
  - [Конфигурация](#a2a-configuration)
- [Безопасность](#security)

---

## Часть 1: MCP (Model Context Protocol)

### Обзор MCP

Model Context Protocol (MCP) — это протокол на базе JSON-RPC 2.0, который стандартизирует способы обнаружения и вызова инструментов приложениями LLM. OpenFang поддерживает MCP в обоих направлениях:

- **Как клиент**: OpenFang подключается к внешним серверам MCP (GitHub, файловая система, базы данных, Puppeteer и т. д.) и делает их инструменты доступными для всех агентов.
- **Как сервер**: OpenFang предоставляет своих собственных агентов в качестве инструментов MCP, поэтому такие IDE, как Cursor, VS Code и Claude Desktop, могут вызывать агентов OpenFang напрямую.

OpenFang реализует протокол MCP версии `2024-11-05`.

**Исходные файлы:**
- Клиент: `crates/openfang-runtime/src/mcp.rs`
- Обработчик сервера: `crates/openfang-runtime/src/mcp_server.rs`
- Сервер CLI: `crates/openfang-cli/src/mcp.rs`
- Типы конфигурации: `crates/openfang-types/src/config.rs` (`McpServerConfigEntry`, `McpTransportEntry`)

---

### Клиент MCP

Клиент MCP (`McpConnection` в `openfang-runtime`) позволяет OpenFang подключаться к любому MCP-совместимому серверу и использовать его инструменты так, как если бы они были встроенными.

#### Конфигурация

Серверы MCP настраиваются в `config.toml` с использованием массива `[[mcp_servers]]`:

```toml
[[mcp_servers]]
name = "github"
timeout_secs = 30
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

Каждая запись соответствует структуре `McpServerConfigEntry`:

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `name` | `String` | обязательно | Отображаемое имя, используемое в пространстве имен инструментов |
| `transport` | `McpTransportEntry` | обязательно | Способ подключения (stdio или SSE) |
| `timeout_secs` | `u64` | `30` | Таймаут запроса JSON-RPC |
| `env` | `Vec<String>` | `[]` | Переменные окружения для передачи в подпроцесс |

#### Типы транспорта

OpenFang поддерживает два транспорта MCP, определенных в `McpTransport`:

**Stdio** — запускает подпроцесс и взаимодействует через stdin/stdout с использованием JSON-RPC с разделителями-строками:

```toml
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

**SSE** — подключается к удаленной конечной точке HTTP и отправляет JSON-RPC через POST:

```toml
[mcp_servers.transport]
type = "sse"
url = "https://mcp.example.com/api"
```

#### Пространство имен инструментов

Все инструменты, обнаруженные на серверах MCP, получают пространство имен по шаблону `mcp_{server}_{tool}` для предотвращения конфликтов со встроенными инструментами или инструментами с других серверов. Имена нормализуются к нижнему регистру, а дефисы заменяются на подчеркивания.

Примеры:
- Сервер `github`, инструмент `create_issue` становится `mcp_github_create_issue`
- Сервер `my-server`, инструмент `do_thing` становится `mcp_my_server_do_thing`

Вспомогательные функции (экспортируемые из `openfang_runtime::mcp`):
- `format_mcp_tool_name(server, tool)` — создает имя с пространством имен
- `is_mcp_tool(name)` — проверяет, начинается ли имя инструмента с `mcp_`
- `extract_mcp_server(tool_name)` — извлекает имя сервера из имени инструмента

#### Автоматическое подключение при загрузке ядра

При запуске ядра (`start_background_agents()`) оно проверяет `config.mcp_servers`. Если настроены какие-либо серверы, запускается фоновая задача, вызывающая `connect_mcp_servers()`. Этот метод:

1. Перебирает каждую запись `McpServerConfigEntry` в конфигурации.
2. Преобразует `McpTransportEntry` уровня конфигурации в `McpTransport` времени выполнения.
3. Вызывает `McpConnection::connect()`, которая:
   - Запускает подпроцесс (stdio) или создает HTTP-клиент (SSE).
   - Отправляет рукопожатие `initialize` с информацией о клиенте.
   - Отправляет уведомление `notifications/initialized`.
   - Вызывает `tools/list` для обнаружения всех доступных инструментов.
   - Присваивает каждому инструменту пространство имен `mcp_{server}_{tool}`.
4. Кэширует обнаруженные записи `ToolDefinition` в `kernel.mcp_tools`.
5. Сохраняет активное соединение `McpConnection` в `kernel.mcp_connections`.

После подключения ядро записывает в лог общее количество доступных инструментов MCP.

#### Обнаружение и перечисление инструментов

Инструменты MCP объединяются с набором доступных инструментов агента через `available_tools()`:

```
встроенные инструменты (23) + инструменты навыков + инструменты MCP = полный список инструментов
```

Когда агент вызывает инструмент MCP в своем цикле, исполнитель инструментов распознает префикс `mcp_`, находит соответствующее соединение `McpConnection`, удаляет префикс пространства имен и перенаправляет запрос `tools/call` внешнему серверу MCP.

#### Жизненный цикл соединения

Структура `McpConnection` управляет временем жизни соединения:

```rust
pub struct McpConnection {
    config: McpServerConfig,
    tools: Vec<ToolDefinition>,
    transport: McpTransportHandle,  // Stdio или SSE
    next_id: u64,                   // Счетчик запросов JSON-RPC
}
```

Когда соединение разрывается, подпроцессы stdio автоматически завершаются через `Drop`:

```rust
impl Drop for McpConnection {
    fn drop(&mut self) {
        if let McpTransportHandle::Stdio { ref mut child, .. } = self.transport {
            let _ = child.start_kill();
        }
    }
}
```

---

### Сервер MCP

OpenFang также может выступать в роли сервера MCP, предоставляя своих агентов как вызываемые инструменты для внешних клиентов MCP.

#### Как это работает

Каждый агент OpenFang становится инструментом MCP с именем `openfang_agent_{name}` (дефисы заменяются на подчеркивания). Инструмент принимает один строковый параметр `message` и возвращает ответ агента.

Например, агент с именем `code-reviewer` становится инструментом MCP `openfang_agent_code_reviewer`.

#### CLI: `openfang mcp`

Основной способ запуска сервера MCP — команда `openfang mcp`, которая запускает сервер MCP на базе stdio:

```bash
openfang mcp
```

Эта команда:
1. Проверяет, запущен ли демон OpenFang (через `find_daemon()`).
2. Если найден, проксирует все вызовы инструментов демону через его HTTP API.
3. Если демон не запущен, загружает ядро внутри процесса в качестве запасного варианта.
4. Читает сообщения JSON-RPC с заголовком Content-Length из stdin.
5. Записывает ответы JSON-RPC с заголовком Content-Length в stdout.

Сервер MCP использует `McpBackend`, который поддерживает два режима:
- `McpBackend::Daemon` — пересылает запросы запущенному демону OpenFang через HTTP.
- `McpBackend::InProcess` — загружает полное ядро, когда демон недоступен.

#### Конечная точка HTTP MCP

OpenFang также предоставляет конечную точку MCP через HTTP по адресу `POST /mcp`. В отличие от сервера stdio (который предоставляет только агентов), конечная точка HTTP предоставляет полный набор инструментов (встроенные + навыки + инструменты MCP) и выполняет инструменты через конвейер ядра `execute_tool()`. Это означает, что HTTP-конечная точка MCP поддерживает:

- Все 23 встроенных инструмента (file_read, web_fetch и т. д.).
- Все установленные инструменты навыков.
- Все подключенные инструменты серверов MCP.

#### Поддерживаемые методы JSON-RPC

| Метод | Описание |
|--------|-------------|
| `initialize` | Рукопожатие; возвращает возможности и информацию о сервере |
| `notifications/initialized` | Подтверждение клиента; без ответа |
| `tools/list` | Возвращает все доступные инструменты с именами, описаниями и схемами ввода |
| `tools/call` | Выполняет инструмент и возвращает результат |

Неизвестные методы получают ошибку `-32601` (Method not found).

#### Детали протокола

**Обрамление сообщений** (режим stdio):

```
Content-Length: 123\r\n
\r\n
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}
```

Размер сообщений ограничен 10 МБ (`MAX_MCP_MESSAGE_SIZE`). Сообщения, превышающие этот размер, отбрасываются и отклоняются.

**Рукопожатие инициализации:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {},
    "clientInfo": { "name": "cursor", "version": "1.0" }
  }
}
```

Ответ:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": { "tools": {} },
    "serverInfo": { "name": "openfang", "version": "0.1.0" }
  }
}
```

**Вызов инструмента:**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "openfang_agent_code_reviewer",
    "arguments": {
      "message": "Review this Python function for security issues..."
    }
  }
}
```

Ответ:

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [{
      "type": "text",
      "text": "I found 3 potential security issues..."
    }]
  }
}
```

#### Подключение из IDE

**Cursor / VS Code (с расширением MCP):**

Добавьте в файл конфигурации MCP (например, `.cursor/mcp.json` или настройки VS Code MCP):

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

**Claude Desktop:**

Добавьте в `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "openfang": {
      "command": "openfang",
      "args": ["mcp"],
      "env": {}
    }
  }
}
```

После настройки все агенты OpenFang появятся в IDE как инструменты. Например, вы можете попросить Claude Desktop: "use the openfang code-reviewer agent to review this file."

---

### Примеры конфигурации MCP

#### Сервер GitHub (инструменты для работы с файлами, задачами и PR)

```toml
[[mcp_servers]]
name = "github"
timeout_secs = 30
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
```

#### Сервер файловой системы

```toml
[[mcp_servers]]
name = "filesystem"
timeout_secs = 10
env = []

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
```

#### Сервер PostgreSQL

```toml
[[mcp_servers]]
name = "postgres"
timeout_secs = 30
env = ["DATABASE_URL"]

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
```

#### Puppeteer (автоматизация браузера)

```toml
[[mcp_servers]]
name = "puppeteer"
timeout_secs = 60

[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-puppeteer"]
```

#### Удаленный сервер SSE

```toml
[[mcp_servers]]
name = "remote-tools"
timeout_secs = 30

[mcp_servers.transport]
type = "sse"
url = "https://tools.example.com/mcp"
```

#### Несколько серверов

```toml
[[mcp_servers]]
name = "github"
env = ["GITHUB_PERSONAL_ACCESS_TOKEN"]
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]

[[mcp_servers]]
name = "filesystem"
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]

[[mcp_servers]]
name = "postgres"
env = ["DATABASE_URL"]
[mcp_servers.transport]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
```

---

### Конечные точки API MCP

| Метод | Путь | Описание |
|--------|------|-------------|
| `GET` | `/api/mcp/servers` | Список настроенных и подключенных серверов MCP с их инструментами |
| `POST` | `/mcp` | Обработка запросов JSON-RPC MCP через HTTP (полное выполнение инструментов) |

Ответ **GET /api/mcp/servers**:

```json
{
  "configured": [
    {
      "name": "github",
      "transport": { "type": "stdio", "command": "npx", "args": [...] },
      "timeout_secs": 30,
      "env": ["GITHUB_PERSONAL_ACCESS_TOKEN"]
    }
  ],
  "connected": [
    {
      "name": "github",
      "tools_count": 12,
      "tools": [
        { "name": "mcp_github_create_issue", "description": "[MCP:github] Create a GitHub issue" },
        { "name": "mcp_github_search_repos", "description": "[MCP:github] Search repositories" }
      ],
      "connected": true
    }
  ]
}
```

---

## Часть 2: A2A (Agent-to-Agent Protocol)

### Обзор A2A

Протокол Agent-to-Agent (A2A), первоначально разработанный Google, обеспечивает совместимость агентов из разных фреймворков. Он позволяет агентам, созданным с помощью различных инструментов, обнаруживать возможности друг друга и обмениваться задачами.

OpenFang реализует A2A в обоих направлениях:

- **Как сервер**: Публикует карточки агентов (Agent Cards), описывающие возможности каждого агента, принимает задачи и отслеживает их жизненный цикл.
- **Как клиент**: Обнаруживает внешних агентов A2A во время загрузки, отправляет им задачи и опрашивает результаты.

**Исходные файлы:**
- Типы протокола и логика: `crates/openfang-runtime/src/a2a.rs`
- Маршруты API: `crates/openfang-api/src/routes.rs`
- Типы конфигурации: `crates/openfang-types/src/config.rs` (`A2aConfig`, `ExternalAgent`)

---

### Карточка агента (Agent Card)

Карточка агента — это JSON-документ, который описывает идентификатор агента, его возможности и поддерживаемые режимы взаимодействия. Она доступна по общеизвестному пути `/.well-known/agent.json` в соответствии со спецификацией A2A.

Структура `AgentCard`:

```rust
pub struct AgentCard {
    pub name: String,
    pub description: String,
    pub url: String,                         // URL конечной точки (напр., "http://host/a2a")
    pub version: String,                     // версия протокола
    pub capabilities: AgentCapabilities,
    pub skills: Vec<AgentSkill>,             // дескрипторы навыков A2A
    pub default_input_modes: Vec<String>,    // напр., ["text"]
    pub default_output_modes: Vec<String>,   // напр., ["text"]
}
```

**AgentCapabilities:**

```rust
pub struct AgentCapabilities {
    pub streaming: bool,                 // true — OpenFang поддерживает потоковую передачу
    pub push_notifications: bool,        // false — в настоящее время не реализовано
    pub state_transition_history: bool,  // true — доступна история статусов задач
}
```

**AgentSkill** (не путать с навыками OpenFang — это дескрипторы возможностей A2A):

```rust
pub struct AgentSkill {
    pub id: String,           // совпадает с именем инструмента OpenFang
    pub name: String,         // понятное имя (подчеркивания заменены пробелами)
    pub description: String,
    pub tags: Vec<String>,
    pub examples: Vec<String>,
}
```

Карточки агентов создаются на основе манифестов агентов OpenFang через `build_agent_card()`. Каждый инструмент в списке возможностей агента становится дескриптором навыка A2A. Пример карточки:

```json
{
  "name": "code-reviewer",
  "description": "Reviews code for bugs, security issues, and style",
  "url": "http://127.0.0.1:50051/a2a",
  "version": "0.1.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false,
    "stateTransitionHistory": true
  },
  "skills": [
    {
      "id": "file_read",
      "name": "file read",
      "description": "Can use the file_read tool",
      "tags": ["tool"],
      "examples": []
    }
  ],
  "defaultInputModes": ["text"],
  "defaultOutputModes": ["text"]
}
```

---

### Сервер A2A

OpenFang обслуживает запросы A2A через REST API. Реализация на стороне сервера включает:

1. **Публикацию карточки агента** по адресу `/.well-known/agent.json`.
2. **Список агентов** по адресу `/a2a/agents`.
3. **Отправку и отслеживание задач** через `A2aTaskStore`.

#### A2aTaskStore

`A2aTaskStore` — это ограниченное хранилище в памяти для отслеживания жизненного цикла задач A2A:

```rust
pub struct A2aTaskStore {
    tasks: Mutex<HashMap<String, A2aTask>>,
    max_tasks: usize,  // по умолчанию: 1000
}
```

Основные свойства:
- **Ограниченность**: Когда хранилище достигает `max_tasks`, оно удаляет самую старую завершенную/неудачную/отмененную задачу (FIFO).
- **Потокобезопасность**: Использует `Mutex<HashMap>` для параллельного доступа.
- **Поле ядра**: Сохраняется как `kernel.a2a_task_store`.

Методы `A2aTaskStore`:
- `insert(task)` — добавить новую задачу, удалив старые при заполнении.
- `get(task_id)` — получить задачу по ID.
- `update_status(task_id, status)` — изменить статус задачи.
- `complete(task_id, response, artifacts)` — пометить как завершенную с ответом.
- `fail(task_id, error_message)` — пометить как неудачную с ошибкой.
- `cancel(task_id)` — пометить как отмененную.

#### Поток отправки задачи

При вызове `POST /a2a/tasks/send`:

1. Извлекается текст сообщения из формата запроса A2A (части с типом "text").
2. Находится целевой агент (в настоящее время используется первый зарегистрированный агент).
3. Создается `A2aTask` со статусом `Working` и вставляется в хранилище задач.
4. Сообщение отправляется агенту через `kernel.send_message()`.
5. В случае успеха: задача завершается с ответом агента.
6. В случае ошибки: задача помечается как неудачная с сообщением об ошибке.
7. Возвращается конечное состояние задачи.

---

### Клиент A2A

Структура `A2aClient` обнаруживает внешних агентов A2A и взаимодействует с ними:

```rust
pub struct A2aClient {
    client: reqwest::Client,  // таймаут 30 секунд
}
```

**Методы:**

- `discover(url)` — получает `{url}/.well-known/agent.json` и парсит карточку агента.
- `send_task(url, message, session_id)` — отправляет запрос JSON-RPC на выполнение задачи.
- `get_task(url, task_id)` — опрашивает статус задачи.

#### Автообнаружение при загрузке

Когда ядро запускается и A2A включен с настроенными внешними агентами, запускается фоновая задача, вызывающая `discover_external_agents()`. Эта функция:

1. Создает `A2aClient`.
2. Перебирает каждого настроенного `ExternalAgent`.
3. Получает карточку каждого агента по адресу `{url}/.well-known/agent.json`.
4. Записывает успешные обнаружения в лог (имя, URL, количество навыков).
5. Сохраняет обнаруженные пары `(name, AgentCard)` в `kernel.a2a_external_agents`.

Неудачные попытки обнаружения записываются как предупреждения, но не препятствуют загрузке.

#### Отправка задач внешним агентам

```rust
let client = A2aClient::new();
let task = client.send_task(
    "https://other-agent.example.com/a2a",
    "Analyze this dataset for anomalies",
    Some("session-123"),
).await?;
println!("Task {}: {:?}", task.id, task.status);
```

Клиент отправляет запрос JSON-RPC:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tasks/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{ "type": "text", "text": "Analyze this dataset..." }]
    },
    "sessionId": "session-123"
  }
}
```

---

### Жизненный цикл задачи

`A2aTask` отслеживает полный жизненный цикл взаимодействия между агентами:

```rust
pub struct A2aTask {
    pub id: String,
    pub session_id: Option<String>,
    pub status: A2aTaskStatus,
    pub messages: Vec<A2aMessage>,
    pub artifacts: Vec<A2aArtifact>,
}
```

#### Статусы задач

| Статус | Описание |
|--------|-------------|
| `Submitted` | Задача получена, но еще не запущена |
| `Working` | Задача активно обрабатывается агентом |
| `InputRequired` | Агенту требуется дополнительная информация от вызывающей стороны |
| `Completed` | Задача успешно завершена |
| `Cancelled` | Задача была отменена вызывающей стороной |
| `Failed` | В ходе выполнения задачи возникла ошибка |

#### Формат сообщения

Сообщения используют специфичный для A2A формат с типизированными частями контента:

```rust
pub struct A2aMessage {
    pub role: String,          // "user" или "agent"
    pub parts: Vec<A2aPart>,
}

pub enum A2aPart {
    Text { text: String },
    File { name: String, mime_type: String, data: String },  // base64
    Data { mime_type: String, data: serde_json::Value },
}
```

#### Артефакты

Задачи могут создавать артефакты (файлы, структурированные данные) вместе с сообщениями:

```rust
pub struct A2aArtifact {
    pub name: String,
    pub parts: Vec<A2aPart>,
}
```

---

### Конечные точки API A2A

| Метод | Путь | Аутентификация | Описание |
|--------|------|------|-------------|
| `GET` | `/.well-known/agent.json` | Публичная | Карточка для основного агента |
| `GET` | `/a2a/agents` | Публичная | Список всех карточек агентов |
| `POST` | `/a2a/tasks/send` | Публичная | Отправить задачу агенту |
| `GET` | `/a2a/tasks/{id}` | Публичная | Получить статус задачи и сообщения |
| `POST` | `/a2a/tasks/{id}/cancel` | Публичная | Отменить выполняемую задачу |

#### GET /.well-known/agent.json

Возвращает карточку первого зарегистрированного агента. Если ни один агент не запущен, возвращает карточку-заглушку.

#### GET /a2a/agents

Выводит список всех зарегистрированных агентов в виде карточек агентов:

```json
{
  "agents": [
    {
      "name": "code-reviewer",
      "description": "Reviews code for bugs and security issues",
      "url": "http://127.0.0.1:50051/a2a",
      "version": "0.1.0",
      "capabilities": { "streaming": true, "pushNotifications": false, "stateTransitionHistory": true },
      "skills": [...],
      "defaultInputModes": ["text"],
      "defaultOutputModes": ["text"]
    }
  ],
  "total": 1
}
```

#### POST /a2a/tasks/send

Отправить задачу. Тело запроса соответствует формату JSON-RPC 2.0:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tasks/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{ "type": "text", "text": "Review this code for security issues" }]
    },
    "sessionId": "optional-session-id"
  }
}
```

Ответ (завершенная задача):

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "sessionId": "optional-session-id",
  "status": "completed",
  "messages": [
    {
      "role": "user",
      "parts": [{ "type": "text", "text": "Review this code for security issues" }]
    },
    {
      "role": "agent",
      "parts": [{ "type": "text", "text": "I found 2 potential issues..." }]
    }
  ],
  "artifacts": []
}
```

#### GET /a2a/tasks/{id}

Опрос статуса задачи. Возвращает `404`, если задача не найдена или была удалена.

#### POST /a2a/tasks/{id}/cancel

Отменить выполняемую задачу. Устанавливает статус `Cancelled`. Возвращает `404`, если задача не найдена.

---

### Конфигурация A2A

A2A настраивается в `config.toml` в разделе `[a2a]`:

```toml
[a2a]
enabled = true
listen_path = "/a2a"

[[a2a.external_agents]]
name = "research-agent"
url = "https://research.example.com"

[[a2a.external_agents]]
name = "data-analyst"
url = "https://data.example.com"
```

Структура `A2aConfig`:

| Поле | Тип | По умолчанию | Описание |
|-------|------|---------|-------------|
| `enabled` | `bool` | `false` | Активны ли конечные точки A2A |
| `listen_path` | `String` | `"/a2a"` | Базовый путь для конечных точек A2A |
| `external_agents` | `Vec<ExternalAgent>` | `[]` | Внешние агенты для обнаружения при загрузке |

Каждый `ExternalAgent`:

| Поле | Тип | Описание |
|-------|------|-------------|
| `name` | `String` | Отображаемое имя для этого внешнего агента |
| `url` | `String` | Базовый URL, где опубликована карточка агента |

Если `a2a` равно `None` (отсутствует в конфигурации), все функции A2A отключены. Конечные точки A2A всегда регистрируются в маршрутизаторе, но для работы обнаружения и хранилища задач требуется `enabled = true`.

---

## Безопасность

### Безопасность MCP

**Изоляция подпроцессов**: Серверы MCP stdio запускаются с `env_clear()` — среда подпроцесса полностью очищается. Передаются только явно разрешенные переменные окружения (перечисленные в поле `env`), а также `PATH`. Это предотвращает утечку секретов в недоверенные процессы серверов MCP.

**Предотвращение обхода путей**: Путь команды для серверов MCP stdio проверяется на отсутствие последовательностей `..`.

**Защита от SSRF**: URL-адреса транспорта SSE проверяются на соответствие известным конечным точкам метаданных (169.254.169.254, metadata.google) для предотвращения атак SSRF.

**Таймаут запроса**: Все запросы MCP имеют настраиваемый таймаут (по умолчанию 30 секунд) для предотвращения зависания соединений.

**Ограничение размера сообщений**: Сервер MCP stdio устанавливает максимальный размер сообщения в 10 МБ для предотвращения атак с исчерпанием памяти. Сообщения, превышающие этот размер, отбрасываются и отклоняются.

### Безопасность A2A

**Ограничение частоты запросов**: Конечные точки A2A проходят через тот же ограничитель частоты GCRA, что и все остальные конечные точки API.

**Аутентификация API**: Когда в конфигурации ядра установлен `api_key`, все конечные точки API (включая A2A) требуют заголовок `Authorization: Bearer <key>`. Исключение составляют `/.well-known/agent.json` и конечная точка работоспособности (health), которые обычно являются публичными.

**Ограничения хранилища задач**: `A2aTaskStore` ограничен (по умолчанию 1000 задач) с удалением FIFO завершенных/неудачных/отмененных задач, что предотвращает исчерпание памяти из-за накопления задач.

**Обнаружение внешних агентов**: `A2aClient` использует 30-секундный таймаут и отправляет заголовок `User-Agent: OpenFang/0.1 A2A`. Неудачные попытки обнаружения записываются в лог, но не блокируют загрузку ядра.

### Защита на уровне ядра

Оба потока выполнения инструментов MCP и A2A проходят через тот же конвейер безопасности, что и все остальные вызовы инструментов:
- Контроль доступа на основе возможностей (агенты получают только те инструменты, на которые они авторизованы).
- Обрезка результатов инструментов (жесткое ограничение в 50 000 символов).
- Универсальный 60-секундный таймаут выполнения инструмента.
- Обнаружение циклических вызовов (блокирует повторяющиеся шаблоны вызовов инструментов).
- Отслеживание "грязных" данных (taint tracking) между инструментами.
