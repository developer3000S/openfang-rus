# Разработка навыков

Навыки (skills) — это подключаемые наборы инструментов, расширяющие возможности агентов в OpenFang. Навык объединяет один или несколько инструментов с их реализацией, позволяя агентам выполнять задачи, не предусмотренные встроенными средствами. Это руководство охватывает создание навыков, формат манифеста, среды выполнения Python и WASM, публикацию в FangHub и управление через CLI.

## Содержание

- [Обзор](#overview)
- [Формат навыка](#skill-format)
- [Навыки на Python](#python-skills)
- [Навыки на WASM](#wasm-skills)
- [Требования навыка](#skill-requirements)
- [Установка навыков](#installing-skills)
- [Публикация в FangHub](#publishing-to-fanghub)
- [Команды CLI](#cli-commands)
- [Совместимость с OpenClaw](#openclaw-compatibility)
- [Лучшие практики](#best-practices)

---

<a name="overview"></a>
## Обзор

Навык состоит из:

1. **Манифеста** (`skill.toml` или `SKILL.md`), который объявляет метаданные, тип среды выполнения, предоставляемые инструменты и требования.
2. **Точки входа** (скрипт Python, модуль WASM, модуль Node.js или Markdown только с промптами), которая реализует логику инструментов.

Навыки устанавливаются в директорию `~/.openfang/skills/` и становятся доступными для агентов через реестр навыков. OpenFang поставляется с **60 встроенными навыками**, которые скомпилированы в бинарный файл и доступны сразу.

### Поддерживаемые среды выполнения

| Среда | Язык | Песочница | Примечания |
|---------|----------|-----------|-------|
| `python` | Python 3.8+ | Нет (подпроцесс с `env_clear()`) | Проще всего в написании. Использует протокол JSON через stdin/stdout. |
| `wasm` | Rust, C, Go и др. | Да (Wasmtime) | Полная изоляция в песочнице. Лучший выбор для инструментов, критичных к безопасности. |
| `node` | JavaScript/TypeScript | Нет (подпроцесс) | Для совместимости с OpenClaw. |
| `prompt_only` | Markdown | Н/Д | Экспертные знания, внедряемые в системный промпт. Без выполнения кода. |
| `builtin` | Rust | Н/Д | Скомпилированы в бинарный файл. Только для основных инструментов. |

### 60 встроенных навыков

OpenFang включает 60 экспертных навыков, встроенных в систему (установка не требуется):

| Категория | Навыки |
|----------|--------|
| DevOps и инфраструктура | `ci-cd`, `ansible`, `prometheus`, `nginx`, `kubernetes`, `terraform`, `helm`, `docker`, `sysadmin`, `shell-scripting`, `linux-networking` |
| Облака | `aws`, `gcp`, `azure` |
| Языки | `rust-expert`, `python-expert`, `typescript-expert`, `golang-expert` |
| Фронтенд | `react-expert`, `nextjs-expert`, `css-expert` |
| Базы данных | `postgres-expert`, `redis-expert`, `sqlite-expert`, `mongodb`, `elasticsearch`, `sql-analyst` |
| API и Web | `graphql-expert`, `openapi-expert`, `api-tester`, `oauth-expert` |
| AI/ML | `ml-engineer`, `llm-finetuning`, `vector-db`, `prompt-engineer` |
| Безопасность | `security-audit`, `crypto-expert`, `compliance` |
| Инструменты разработчика | `github`, `git-expert`, `jira`, `linear-tools`, `sentry`, `code-reviewer`, `regex-expert` |
| Письмо | `technical-writer`, `writing-coach`, `email-writer`, `presentation` |
| Данные | `data-analyst`, `data-pipeline` |
| Коллаборация | `slack-tools`, `notion`, `confluence`, `figma-expert` |
| Карьера | `interview-prep`, `project-manager` |
| Продвинутые | `wasm-expert`, `pdf-reader`, `web-search` |

Это навыки типа `prompt_only` в формате SKILL.md — экспертные знания, которые добавляются в системный промпт агента.

### Формат SKILL.md

Формат SKILL.md (также используемый в OpenClaw) использует заголовок YAML и тело в формате Markdown:

```markdown
---
name: rust-expert
description: Экспертные знания в программировании на Rust
---

# Rust Expert

## Ключевые принципы
- Правила владения и заимствования (ownership and borrowing)...
- Аннотации времени жизни (lifetime annotations)...

## Распространенные паттерны
...
```

Файлы SKILL.md автоматически парсятся и конвертируются в навыки типа `prompt_only`. Все такие файлы проходят через автоматический **сканер инъекций**, который ищет попытки обхода ограничений или эксфильтрации данных.

---

<a name="skill-format"></a>
## Формат навыка

### Структура директории

```
my-skill/
  skill.toml          # Манифест (обязательно)
  src/
    main.py           # Точка входа (для навыков на Python)
  README.md           # Опциональная документация
```

### Манифест (skill.toml)

```toml
[skill]
name = "web-summarizer"
version = "0.1.0"
description = "Сжимает любую веб-страницу в список ключевых тезисов"
author = "openfang-community"
license = "MIT"
tags = ["web", "summarizer", "research"]

[runtime]
type = "python"
entry = "src/main.py"

[[tools.provided]]
name = "summarize_url"
description = "Загрузить URL и вернуть краткое саммари по пунктам"
input_schema = { type = "object", properties = { url = { type = "string", description = "URL для обработки" } }, required = ["url"] }

[[tools.provided]]
name = "extract_links"
description = "Извлечь все ссылки с веб-страницы"
input_schema = { type = "object", properties = { url = { type = "string" } }, required = ["url"] }

[requirements]
tools = ["web_fetch"]
capabilities = ["NetConnect(*)"]
```

---

<a name="python-skills"></a>
## Навыки на Python

Навыки на Python писать проще всего. Они запускаются как подпроцессы и взаимодействуют через JSON через stdin/stdout.

### Протокол

1. OpenFang отправляет JSON-объект в stdin скрипта:

```json
{
  "tool": "summarize_url",
  "input": {
    "url": "https://example.com"
  },
  "agent_id": "uuid-...",
  "agent_name": "researcher"
}
```

2. Скрипт обрабатывает входные данные и выводит JSON-результат в stdout:

```json
{
  "result": "- Пункт один\n- Пункт два\n- Пункт три"
}
```

В случае ошибки верните объект ошибки:

```json
{
  "error": "Не удалось загрузить URL: соединение отклонено"
}
```

### Пример реализации

`src/main.py`:

```python
#!/usr/bin/env python3
import json
import sys
import urllib.request

def summarize_url(url: str) -> str:
    """Загрузить URL и вернуть краткое описание."""
    req = urllib.request.Request(url, headers={"User-Agent": "OpenFang-Skill/1.0"})
    with urllib.request.urlopen(req, timeout=30) as resp:
        content = resp.read().decode("utf-8", errors="replace")

    # Простейшее извлечение: первые 500 символов
    text = content[:500].strip()
    return f"Summary of {url}:\n{text}..."

def main():
    payload = json.loads(sys.stdin.read())
    tool_name = payload["tool"]
    input_data = payload["input"]

    try:
        if tool_name == "summarize_url":
            result = summarize_url(input_data["url"])
        else:
            print(json.dumps({"error": f"Unknown tool: {tool_name}"}))
            return

        print(json.dumps({"result": result}))
    except Exception as e:
        print(json.dumps({"error": str(e)}))

if __name__ == "__main__":
    main()
```

---

<a name="wasm-skills"></a>
## Навыки на WASM

Навыки на WASM запускаются в изолированной среде Wasmtime. Они идеальны для операций, требующих высокого уровня безопасности, так как песочница ограничивает ресурсы и права доступа.

### Сборка навыка на WASM

1. Напишите код на Rust (или другом языке, компилируемом в WASM):

```rust
// src/lib.rs
use std::io::{self, Read};

#[no_mangle]
pub extern "C" fn _start() {
    let mut input = String::new();
    io::stdin().read_to_string(&mut input).unwrap();

    let payload: serde_json::Value = serde_json::from_str(&input).unwrap();
    let tool = payload["tool"].as_str().unwrap_or("");
    let input_data = &payload["input"];

    let result = match tool {
        "my_tool" => {
            let param = input_data["param"].as_str().unwrap_or("");
            format!("Processed: {param}")
        }
        _ => format!("Unknown tool: {tool}"),
    };

    println!("{}", serde_json::json!({"result": result}));
}
```

2. Скомпилируйте в WASM:

```bash
cargo build --target wasm32-wasi --release
```

3. Укажите путь к `.wasm` файлу в манифесте:

```toml
[runtime]
type = "wasm"
entry = "target/wasm32-wasi/release/my_skill.wasm"
```

---

<a name="skill-requirements"></a>
## Требования навыка

Навыки могут объявлять необходимые ресурсы в разделе `[requirements]`:

### Требования к инструментам

Если вашему навыку нужно вызывать встроенные инструменты (например, `web_fetch` для загрузки страницы перед её обработкой):

```toml
[requirements]
tools = ["web_fetch", "file_read"]
```

### Требования к возможностям (capabilities)

Если навыку нужны специфические права:

```toml
[requirements]
capabilities = ["NetConnect(*)", "ShellExec(python3)"]
```

---

<a name="installing-skills"></a>
## Установка навыков

### Из локальной директории

```bash
openfang skill install /path/to/my-skill
```

### Из FangHub

```bash
openfang skill install web-summarizer
```

### Из Git-репозитория

```bash
openfang skill install https://github.com/user/openfang-skill-example.git
```

### Список установленных навыков

```bash
openfang skill list
```

---

<a name="publishing-to-fanghub"></a>
## Публикация в FangHub

FangHub — это маркетплейс навыков сообщества OpenFang.

### Подготовка навыка

1. Убедитесь, что `skill.toml` содержит полные метаданные.
2. Добавьте `README.md` с инструкциями по использованию.
3. Протестируйте навык локально.

### Поиск в FangHub

```bash
openfang skill search "web scraping"
```

---

<a name="cli-commands"></a>
## Команды CLI

### Основные команды

```bash
# Установить навык
openfang skill install <source>

# Список всех установленных навыков
openfang skill list

# Удалить навык
openfang skill remove <name>

# Поиск в FangHub
openfang skill search <query>

# Создать заготовку нового навыка (интерактивно)
openfang skill create
```

---

<a name="openclaw-compatibility"></a>
## Совместимость с OpenClaw

OpenFang может устанавливать и запускать навыки в формате OpenClaw. Установщик автоматически обнаруживает такие навыки (по наличию `package.json` + `index.ts`/`index.js`) и конвертирует их.

---

<a name="best-practices"></a>
## Лучшие практики

1. **Фокус на одной задаче** — один навык должен хорошо делать одну вещь.
2. **Минимальные требования** — запрашивайте только те инструменты и права, которые действительно нужны.
3. **Понятные имена инструментов** — LLM использует имя и описание инструмента для принятия решения о его вызове.
4. **Четкие схемы ввода** — добавляйте описания для каждого параметра, чтобы LLM знала, что передавать.
5. **Обработка ошибок** — всегда возвращайте JSON-объект ошибки вместо аварийного завершения.
6. **Версионность** — используйте семантическое версионирование.
7. **Тестирование** — проверяйте работу навыка с разными агентами и провайдерами.
8. **Документация** — всегда включайте README с примерами использования.
