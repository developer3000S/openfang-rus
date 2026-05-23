# Документация OpenFang

Добро пожаловать в документацию OpenFang. OpenFang — открытая операционная система для агентов: 14 Rust-crates, 40 каналов, 60 навыков, ~20 провайдеров LLM, 76 API-эндпойнтов и 16 систем безопасности в одном бинарнике.

---

## Быстрый доступ

| Руководство | Описание |
|------------:|:---------|
| [Getting Started](getting-started.md) | Установка, первый агент и первая сессия чата |
| [Configuration](configuration.md) | Полный справочник `config.toml` |
| [CLI Reference](cli-reference.md) | Команды CLI и примеры |
| [Troubleshooting](troubleshooting.md) | Частые проблемы и диагностика |

## Основные концепции

| Руководство | Описание |
|------------:|:---------|
| [Architecture](architecture.md) | Структура crate'ов, загрузка ядра, жизненный цикл агента, подсистема памяти |
| [Agent Templates](agent-templates.md) | 30 предустановленных шаблонов агентов |
| [Workflows](workflows.md) | Мультиагентные конвейеры с ветвлениями и триггерами |
| [Security](security.md) | 16 слоёв защиты (defense-in-depth) |

## Интеграции

| Руководство | Описание |
|------------:|:---------|
| [Channel Adapters](channel-adapters.md) | 40 каналов — настройка и адаптеры |
| [LLM Providers](providers.md) | Провайдеры и маршрутизация моделей |
| [Skills](skill-development.md) | Встроенные навыки и разработка новых |
| [MCP & A2A](mcp-a2a.md) | Протоколы Model Context Protocol и Agent-to-Agent |

## Справочник

| Руководство | Описание |
|------------:|:---------|
| [API Reference](api-reference.md) | Все REST/WS/SSE эндпойнты с примерами |
| [Desktop App](desktop.md) | Nativное приложение Tauri — сборка и архитектура |

## Эксплуатация и релизы

| Руководство | Описание |
|------------:|:---------|
| [Production Checklist](production-checklist.md) | Контрольный список перед продакшен-релизом |

## Дополнительные ресурсы

| Ресурс | Описание |
|------:|:---------|
| [CONTRIBUTING.md](../CONTRIBUTING.md) | Настройка разработки и правила PR |
| [MIGRATION.md](../MIGRATION.md) | Миграция из OpenClaw / LangChain / AutoGPT |
| [SECURITY.md](../SECURITY.md) | Политика безопасности и отчёты об уязвимостях |
| [CHANGELOG.md](../CHANGELOG.md) | История релизов |

---

## Быстрый старт (30 секунд)

```bash
export GROQ_API_KEY="your-key"
openfang init && openfang start
# Откройте http://127.0.0.1:4200
```

## Ключевые числа

| Метрика | Количество |
|--------:|:----------|
| Crates | 14 |
| Шаблонов агентов | 30 |
| Каналов | 40 |
| Встроенных навыков | 60 |
| Встроенных инструментов | 38 |
| Провайдеров LLM | 20 |
| Моделей в каталоге | 51 |
| Псевдонимов моделей | 23 |
| API-эндпойнтов | 76 |
| Систем безопасности | 16 |
| Тестов | 967 |

## Важные пути

| Путь | Описание |
|-----:|:--------|
| `~/.openfang/config.toml` | Основной файл конфигурации |
| `~/.openfang/data/openfang.db` | SQLite база данных |
| `~/.openfang/skills/` | Установленные навыки |
| `~/.openfang/daemon.json` | Информация о демоне (PID, порт) |
| `agents/` | Шаблоны агентов |

## Переменные окружения (важные)

| Переменная | Провайдер |
|-----------:|:--------|
| `ANTHROPIC_API_KEY` | Anthropic (Claude) |
| `OPENAI_API_KEY` | OpenAI |
| `GEMINI_API_KEY` | Google Gemini |
| `GROQ_API_KEY` | Groq |
| `DEEPSEEK_API_KEY` | DeepSeek |
| `XAI_API_KEY` | xAI |

Достаточно одного ключа провайдера для запуска. Groq предоставляет бесплатный тариф.
