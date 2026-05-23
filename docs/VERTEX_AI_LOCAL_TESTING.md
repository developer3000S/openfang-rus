# Руководство по локальному тестированию Vertex AI

## Предварительные условия

1. **JSON сервисного аккаунта GCP** по адресу `C:\Users\at384\Downloads\osc\dbg-grcit-dev-e1-c79e5571a5a7.json`
2. **gcloud CLI** установлен и добавлен в PATH
3. **Rust toolchain** с установленным cargo

## Быстрый старт (рекомендуется)

### Вариант 1: Использование пакетного файла

```batch
# Запустите это из директории openfang:
start-vertex.bat
```

Это автоматически:
- Очистит настройки прокси.
- Установит `GOOGLE_APPLICATION_CREDENTIALS`.
- Получит токен OAuth через `gcloud auth print-access-token`.
- Установит переменную окружения `VERTEX_AI_ACCESS_TOKEN`.
- Запустит OpenFang.

### Вариант 2: Ручная настройка PowerShell

```powershell
# 1. Завершите работу всех существующих экземпляров
taskkill /F /IM openfang.exe 2>$null

# 2. Установите переменные окружения (ВАЖНО: очистите прокси!)
$env:HTTPS_PROXY = ""
$env:HTTP_PROXY = ""
$env:GOOGLE_APPLICATION_CREDENTIALS = "C:\Users\at384\Downloads\osc\dbg-grcit-dev-e1-c79e5571a5a7.json"

# 3. Предварительно получите токен OAuth (ВАЖНО: позволяет избежать проблем с подпроцессами в Windows)
$env:VERTEX_AI_ACCESS_TOKEN = gcloud auth print-access-token

# 4. Запустите OpenFang
cd C:\Users\at384\Downloads\osc\dllm\openfang
.\target\debug\openfang.exe start
```

## Тестирование API

### Создание агента

```powershell
$env:HTTPS_PROXY = ""
$env:HTTP_PROXY = ""

# Создание агента с провайдером Vertex AI по умолчанию (из config.toml)
$body = '{"manifest_toml":"name = \"test-agent\"\nmode = \"assistant\""}'
Invoke-RestMethod -Uri "http://127.0.0.1:50051/api/agents" -Method POST -ContentType "application/json" -Body $body
```

### Отправка запроса в чат

```powershell
$env:HTTPS_PROXY = ""
$env:HTTP_PROXY = ""

$body = '{"model":"test-agent","messages":[{"role":"user","content":"What is 2+2?"}]}'
$response = Invoke-RestMethod -Uri "http://127.0.0.1:50051/v1/chat/completions" -Method POST -ContentType "application/json" -Body $body -TimeoutSec 120
Write-Host $response.choices[0].message.content
```

### Прямой тест Vertex AI (минуя OpenFang)

```powershell
$env:HTTPS_PROXY = ""
$env:HTTP_PROXY = ""

$token = gcloud auth print-access-token
$project = "dbg-grcit-dev-e1"
$region = "us-central1"
$model = "gemini-2.0-flash"
$url = "https://$region-aiplatform.googleapis.com/v1/projects/$project/locations/$region/publishers/google/models/$($model):generateContent"

$body = @{contents = @(@{role = "user"; parts = @(@{text = "Hello!"})})} | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri $url -Method POST -Headers @{Authorization = "Bearer $token"} -ContentType "application/json" -Body $body
```

## Конфигурация

### ~/.openfang/config.toml

```toml
[default_model]
provider = "vertex-ai"
model = "gemini-2.0-flash"

[memory]
decay_rate = 0.05

[network]
listen_addr = "127.0.0.1:4200"
```

## Переменные окружения

| Переменная | Назначение | Обязательно |
|----------|---------|----------|
| `GOOGLE_APPLICATION_CREDENTIALS` | Путь к JSON сервисного аккаунта | Да |
| `VERTEX_AI_ACCESS_TOKEN` | Предварительно полученный токен OAuth (обходит подпроцесс gcloud) | Рекомендуется для Windows |
| `GOOGLE_CLOUD_PROJECT` | Переопределение ID проекта | Нет (автоматически определяется из JSON) |
| `GOOGLE_CLOUD_REGION` / `VERTEX_AI_REGION` | Переопределение региона | Нет (по умолчанию us-central1) |
| `HTTPS_PROXY` / `HTTP_PROXY` | **Должны быть пустыми** для локального тестирования | Критично |

## Устранение неполадок

### "Agent processing failed" (Ошибка 500)

**Причина:** Подпроцесс gcloud не работает должным образом в Windows.

**Решение:** Предварительно получите токен:
```powershell
$env:VERTEX_AI_ACCESS_TOKEN = gcloud auth print-access-token
```

### "Connection refused" (В соединении отказано)

**Причина:** OpenFang не запущен или указан неверный порт.

**Решение:** Убедитесь, что сервер запущен на порту 50051:
```powershell
Get-NetTCPConnection -LocalPort 50051 -ErrorAction SilentlyContinue
```

### Токен истек

**Причина:** Токены OAuth истекают примерно через 1 час.

**Решение:** Получите токен повторно:
```powershell
$env:VERTEX_AI_ACCESS_TOKEN = gcloud auth print-access-token
```

## Команды сборки

```powershell
cd C:\Users\at384\Downloads\osc\dllm\openfang
$env:PATH = "$env:USERPROFILE\.cargo\bin;$env:PATH"

# Дебаг-сборка (более быстрая компиляция)
cargo build -p openfang-cli

# Запуск тестов
cargo test -p openfang-runtime --lib vertex

# Проверка форматирования
cargo fmt --check -p openfang-runtime

# Запуск clippy
cargo clippy -p openfang-runtime --lib -- -W warnings
```

## Конечные точки API

| Эндпоинт | Метод | Назначение |
|----------|--------|---------|
| `http://127.0.0.1:50051/api/agents` | GET | Список агентов |
| `http://127.0.0.1:50051/api/agents` | POST | Создание агента |
| `http://127.0.0.1:50051/api/agents/{id}` | DELETE | Удаление агента |
| `http://127.0.0.1:50051/v1/chat/completions` | POST | OpenAI-совместимый чат |
| `http://127.0.0.1:50051/` | GET | Интерфейс дашборда |

## Файлы, измененные в PR

- `crates/openfang-runtime/src/drivers/vertex.rs` (НОВЫЙ - ~790 строк)
- `crates/openfang-runtime/src/drivers/mod.rs` (+62 строки)
