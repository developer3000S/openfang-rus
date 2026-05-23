````markdown
<p align="center">
  <img src="public/assets/openfang-logo.png" width="160" alt="OpenFang Logo" />
</p>

<h1 align="center">OpenFang</h1>
<h3 align="center">Операционная система для агентов</h3>

<p align="center">
  Открытая операционная система для агентов, написанная на Rust. 137K LOC. 14 crates. 1,767+ тестов. Нулевые предупреждения clippy.<br/>
  <strong>Один бинарник. Проверено в бою. Агенты, которые действительно работают за вас.</strong>
</p>

<p align="center">
  <a href="https://openfang.sh/docs">Документация</a> &bull;
  <a href="https://openfang.sh/docs/getting-started">Быстрый старт</a> &bull;
  <a href="https://x.com/openfangg">Twitter / X</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/language-Rust-orange?style=flat-square" alt="Rust" />
  <img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="MIT" />
  <img src="https://img.shields.io/badge/version-0.6.9-green?style=flat-square" alt="v0.6.9" />
  <img src="https://img.shields.io/badge/tests-2,696%2B%20passing-brightgreen?style=flat-square" alt="Tests" />
  <img src="https://img.shields.io/badge/clippy-0%20warnings-brightgreen?style=flat-square" alt="Clippy" />
  <a href="https://www.buymeacoffee.com/openfang" target="_blank"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-FFDD00?style=flat-square&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me A Coffee" /></a>
</p>

---

> **v0.5.10 (апрель 2026)**
>
> OpenFang функционально завершён, но всё ещё до v1.0. Ожидайте шероховатостей и возможных несовместимых изменений между минорными релизами. Мы быстро выпускаем и быстро фиксируем. Для production-релизов закрепляйте конкретный коммит до v1.0. [Сообщить об ошибке здесь.](https://github.com/RightNow-AI/openfang/issues)

---

## Что такое OpenFang?

OpenFang — это **открытая операционная система для агентов**. Не фреймворк для чат-ботов. Не оболочка на Python вокруг LLM. Не просто «оркестратор агентов». Полноценная операционная система для автономных агентов, написанная с нуля на Rust.

Традиционные фреймворки ждут ввода пользователя. OpenFang запускает **автономных агентов, которые работают за вас**: по расписаниям, круглосуточно, строят графы знаний, мониторят объекты, генерируют лиды, управляют соцсетями и отправляют отчёты на вашу панель управления.

Вся система компилируется в **один бинарник ~32MB**. Одна установка, одна команда — агенты в деле.

```bash
curl -fsSL https://openfang.sh/install | sh
openfang init
openfang start
# Панель доступна по адресу http://localhost:4200
```

<details>
<summary><strong>Windows</strong></summary>

```powershell
irm https://openfang.sh/install.ps1 | iex
openfang init
openfang start
```

</details>

---

## Hands: агенты, которые действительно что-то делают

<p align="center"><em>"Традиционные агенты ждут ввода. Hands работают <strong>за</strong> вас."</em></p>

**Hands** — ключевая инновация OpenFang. Готовые пакеты автономных возможностей, которые запускаются независимо, по расписанию, без необходимости ручных подсказок. Это не чат-бот. Это агент, который просыпается в 6 утра, изучает конкурентов, строит граф знаний, оценивает результаты и присылает отчёт в Telegram, прежде чем вы выпьете кофе.

Каждый Hand включает:
- **HAND.toml**: манифест с инструментами, настройками, требованиями и метриками для дашборда.
- **Системный prompt**: многофазный операционный план. Не одна строка — это 500+ слов экспертной процедуры.
- **SKILL.md**: справочник предметной области, включаемый в контекст во время выполнения.
- **Ограждения (Guardrails)**: ворота одобрения для чувствительных действий (например, Browser Hand требует подтверждения перед покупкой).

Всё собирается в бинарник. Никаких дополнительных загрузок, pip install или Docker pull.

### 7 встроенных Hands

| Hand | Что делает |
|------|------------|
| **Clip** | Загружает YouTube, находит лучшие фрагменты, режет вертикальные шоты с субтитрами и превью, опционально добавляет AI-озвучку и публикует в Telegram/WhatsApp. 8-фазный pipeline. FFmpeg + yt-dlp + 5 STT-бэкендов. |
| **Lead** | Ежедневно находит перспективы по ICP, дополняет данными, ставит скор 0-100, удаляет дубликаты из БД и поставляет квалифицированные лиды в CSV/JSON/Markdown. Со временем строит профили ICP. |
| **Collector** | OSINT-качество разведки. Вы даёте цель (компания, человек, тема). Мониторит изменения, отслеживает тональность, строит граф знаний и присылает критические оповещения при важных событиях. |
| **Predictor** | Суперпрогнозирование. Собирает сигналы, строит откалиброванные цепочки рассуждений, выдаёт прогнозы с интервалами доверия и отслеживает точность (Brier score). Есть контринтуитивный режим. |
| **Researcher** | Глубокий автономный исследователь. Сверяет источники, оценивает достоверность по CRAAP (Currency, Relevance, Authority, Accuracy, Purpose), генерирует отчёты с цитированием (APA), поддерживает несколько языков. |
| **Twitter** | Автономное управление аккаунтом Twitter/X. Генерирует контент в 7 форматах, планирует публикации, отвечает на упоминания, отслеживает метрики. Имеет очередь одобрения — ничего не публикуется без вашего OK. |
| **Browser** | Агент веб-автоматизации. Навигация, заполнение форм, клик по элементам, многошаговые рабочие процессы. Использует мост Playwright с сохранением сессий. **Обязательный gate на покупки**: никогда не потратит ваши деньги без явного подтверждения. |

```bash
# Активировать Researcher Hand. Он сразу начинает работу.
openfang hand activate researcher

# Проверить прогресс
openfang hand status researcher

# Активировать генерацию лидов по расписанию
openfang hand activate lead

# Приостановить без потери состояния
openfang hand pause lead

# Посмотреть список Hands
openfang hand list
```

**Создайте свой.** Опишите `HAND.toml` с инструментами, настройками и системным prompt. Публикуйте в FangHub.

---

## OpenFang и рынок

<p align="center">
  <img src="public/assets/openfang-vs-claws.png" width="600" alt="OpenFang vs OpenClaw vs ZeroClaw" />
</p>

### Бенчмарки: измерено, а не распиарено

Все данные — из официальной документации и публичных репозиториев, февраль 2026.

#### Время холодного старта (меньше — лучше)

```
ZeroClaw   ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10 ms
OpenFang   ██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  180 ms    ★
LangGraph  █████████████████░░░░░░░░░░░░░░░░░░░░░░░░  2.5 sec
CrewAI     ████████████████████░░░░░░░░░░░░░░░░░░░░░░  3.0 sec
AutoGen    ██████████████████████████░░░░░░░░░░░░░░░░  4.0 sec
OpenClaw   █████████████████████████████████████████░░  5.98 sec
```

#### Потребление памяти в простое (меньше — лучше)

```
ZeroClaw   █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    5 MB
OpenFang   ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   40 MB    ★
LangGraph  ██████████████████░░░░░░░░░░░░░░░░░░░░░░░░░  180 MB
CrewAI     ████████████████████░░░░░░░░░░░░░░░░░░░░░░░  200 MB
AutoGen    █████████████████████████░░░░░░░░░░░░░░░░░░  250 MB
OpenClaw   ████████████████████████████████████████░░░  394 MB
```

#### Размер установки (меньше — лучше)

```
ZeroClaw   █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  8.8 MB
OpenFang   ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   32 MB    ★
CrewAI     ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  100 MB
LangGraph  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  150 MB
AutoGen    ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░  200 MB
OpenClaw   ████████████████████████████████████████░░░  500 MB
```

#### Системы безопасности (больше — лучше)

```
OpenFang   ████████████████████████████████████████████   16      ★
ZeroClaw   ███████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░    6
OpenClaw   ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    3
AutoGen    █████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    2
LangGraph  █████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    2
CrewAI     ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    1
```

#### Канальные адаптеры (больше — лучше)

```
OpenFang   ████████████████████████████████████████████   40      ★
ZeroClaw   ███████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░   15
OpenClaw   █████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   13
CrewAI     ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
AutoGen    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
LangGraph  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
```

#### Поставщики LLM (больше — лучше)

```
ZeroClaw   ████████████████████████████████████████████   28
OpenFang   ██████████████████████████████████████████░░   27      ★
LangGraph  ██████████████████████░░░░░░░░░░░░░░░░░░░░░   15
CrewAI     ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10
OpenClaw   ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10
AutoGen    ███████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    8
```

### Сравнение по функциям

| Фича | OpenFang | OpenClaw | ZeroClaw | CrewAI | AutoGen | LangGraph |
|------|----------|----------|----------|--------|---------|-----------|
| **Язык** | **Rust** | TypeScript | **Rust** | Python | Python | Python |
| **Автономные Hands** | **7 встроенных** | Нет | Нет | Нет | Нет | Нет |
| **Слои безопасности** | **16** | 3 | 6 | 1 | Docker | AES enc. |
| **Песочница агентов** | **WASM с двойным учётом** | Нет | Allowlists | Нет | Docker | Нет |
| **Канальные адаптеры** | **40** | 13 | 15 | 0 | 0 | 0 |
| **Встроенные инструменты** | **53 + MCP + A2A** | 50+ | 12 | Плагины | MCP | LC tools |
| **Память** | **SQLite + vector** | File-based | SQLite FTS5 | 4-layer | External | Checkpoints |
| **Десктоп-приложение** | **Tauri 2.0** | Нет | Нет | Нет | Studio | Нет |
| **Аудит** | **Merkle hash-chain** | Logs | Logs | Tracing | Logs | Checkpoints |
| **Cold Start** | **<200ms** | ~6s | ~10ms | ~3s | ~4s | ~2.5s |
| **Размер установки** | **~32 MB** | ~500 MB | ~8.8 MB | ~100 MB | ~200 MB | ~150 MB |
| **Лицензия** | MIT | MIT | MIT | MIT | Apache 2.0 | MIT |

---

## 16 систем безопасности: глубинная защита

OpenFang не добавляет безопасность постфактум. Каждый слой автономно тестируем и не создаёт единоточечной уязвимости.

| # | Система | Что делает |
|---|--------|------------|
| 1 | **WASM Dual-Metered Sandbox** | Код инструментов запускается в WebAssembly с учётом топлива и прерыванием по эпохам. Watchdog убивает зависшие задачи. |
| 2 | **Merkle Hash-Chain Audit Trail** | Каждое действие криптографически связано с предыдущим. Подделка одной записи ломает цепочку. |
| 3 | **Information Flow Taint Tracking** | Метки распространяются по выполнению. Секреты отслеживаются от источника до стока. |
| 4 | **Ed25519 Signed Agent Manifests** | Идентичности и наборы возможностей агентов подписаны. |
| 5 | **SSRF Protection** | Блокирует приватные IP, cloud metadata и DNS rebind-атаки. |
| 6 | **Secret Zeroization** | `Zeroizing<String>` очищает API-ключи из памяти сразу после использования. |
| 7 | **OFP Mutual Authentication** | HMAC-SHA256 nonce-based, проверка в константное время для P2P. |
| 8 | **Capability Gates** | RBAC. Агенты декларируют требуемые инструменты, ядро это принуждает. |
| 9 | **Security Headers** | CSP, X-Frame-Options, HSTS, X-Content-Type-Options на всех ответах. |
| 10 | **Health Endpoint Redaction** | Публичный health возвращает минимальную информацию. Полные диагностические данные доступны только по аутентификации. |
| 11 | **Subprocess Sandbox** | `env_clear()` + селективная передача переменных. Изоляция дерева процессов и кросс-платформенное завершение. |
| 12 | **Prompt Injection Scanner** | Обнаруживает попытки переопределения, паттерны утечки данных и инъекции shell-ссылок в навыках. |
| 13 | **Loop Guard** | Обнаружение loop'ов вызовов инструментов по SHA256 с circuit breaker. Обрабатывает ping-pong сценарии. |
| 14 | **Session Repair** | 7-фазная валидация истории сообщений и автоматическое восстановление после повреждений. |
| 15 | **Path Traversal Prevention** | Каноникализация путей с блокировкой ухода через symlink. `../` не работает. |
| 16 | **GCRA Rate Limiter** | Cost-aware token bucket rate limiting с учётом IP и очисткой устаревших записей. |

---

## Архитектура

14 Rust crates. 137,728 строк кода. Модульная архитектура ядра.

```
openfang-kernel      Orchestration, workflows, metering, RBAC, scheduler, budget tracking
openfang-runtime     Agent loop, 3 LLM drivers, 53 tools, WASM sandbox, MCP, A2A
openfang-api         140+ REST/WS/SSE endpoints, OpenAI-compatible API, dashboard
openfang-channels    40 messaging adapters with rate limiting, DM/group policies
openfang-memory      SQLite persistence, vector embeddings, canonical sessions, compaction
openfang-types       Core types, taint tracking, Ed25519 manifest signing, model catalog
openfang-skills      60 bundled skills, SKILL.md parser, FangHub marketplace
openfang-hands       7 autonomous Hands, HAND.toml parser, lifecycle management
openfang-extensions  25 MCP templates, AES-256-GCM credential vault, OAuth2 PKCE
openfang-wire        OFP P2P protocol with HMAC-SHA256 mutual authentication
openfang-cli         CLI with daemon management, TUI dashboard, MCP server mode
openfang-desktop     Tauri 2.0 native app (system tray, notifications, global shortcuts)
openfang-migrate     OpenClaw, LangChain, AutoGPT migration engine
xtask                Build automation
```

---

## 40 канальных адаптеров

Подключите агентов ко всем платформам, где находятся ваши пользователи.

**Core:** Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Email (IMAP/SMTP)
**Enterprise:** Microsoft Teams, Mattermost, Google Chat, Webex, Feishu/Lark, Zulip
**Social:** LINE, Viber, Facebook Messenger, Mastodon, Bluesky, Reddit, LinkedIn, Twitch
**Community:** IRC, XMPP, Guilded, Revolt, Keybase, Discourse, Gitter
**Privacy:** Threema, Nostr, Mumble, Nextcloud Talk, Rocket.Chat, Ntfy, Gotify
**Workplace:** Pumble, Flock, Twist, DingTalk, Zalo, Webhooks

Каждый адаптер поддерживает переопределение моделей на канал, DM/group политики, rate limiting и форматирование вывода.

---

## WhatsApp Web Gateway (QR-код)

Подключите личный WhatsApp к OpenFang через QR-код, как в WhatsApp Web. Meta Business не требуется.

### Требования

- **Node.js >= 18** ([скачать](https://nodejs.org/))
- OpenFang установлен и инициализирован

### Настройка

**1. Установите зависимости шлюза:**

```bash
cd packages/whatsapp-gateway
npm install
```

**2. Настройте `config.toml`:**

```toml
[channels.whatsapp]
mode = "web"
default_agent = "assistant"
```

**3. Установите URL шлюза (один из вариантов):**

Добавьте в профиль оболочки для постоянства:

```bash
# macOS / Linux
echo 'export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"' >> ~/.zshrc
source ~/.zshrc
```

Или задайте в окружении при запуске:

```bash
export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"
```

**4. Запустите шлюз:**

```bash
node packages/whatsapp-gateway/index.js
```

Шлюз по умолчанию слушает порт `3009`. Переопределите `WHATSAPP_GATEWAY_PORT`.

**5. Запустите OpenFang:**

```bash
openfang start
# Панель: http://localhost:4200
```

**6. Отсканируйте QR-код:**

Откройте дашборд → **Channels** → **WhatsApp**. Появится QR-код. Отсканируйте в телефоне:

> **WhatsApp** → **Settings** → **Linked Devices** → **Link a Device**

После сканирования статус станет `connected`, входящие сообщения будут маршрутизироваться на настроенного агента.

### Переменные окружения шлюза

| Переменная | Описание | По умолчанию |
|-----------|----------|--------------|
| `WHATSAPP_WEB_GATEWAY_URL` | URL шлюза для подключения OpenFang | _(пусто = отключено)_ |
| `WHATSAPP_GATEWAY_PORT` | Порт, на котором слушает шлюз | `3009` |
| `OPENFANG_URL` | URL OpenFang API, куда шлюз репортит | `http://127.0.0.1:4200` |
| `OPENFANG_DEFAULT_AGENT` | Агент, который обрабатывает входящие сообщения | `assistant` |

### API-эндпойнты шлюза

| Метод | Путь | Описание |
|-------|------|----------|
| `POST` | `/login/start` | Генерация QR (возвращает base64 PNG) |
| `GET` | `/login/status` | Статус подключения (`disconnected`, `qr_ready`, `connected`) |
| `POST` | `/message/send` | Отправка сообщения (`{ "to": "5511999999999", "text": "Hello" }`) |
| `GET` | `/health` | Health check |

### Альтернатива: WhatsApp Cloud API

Для production используйте [WhatsApp Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api) с Meta Business аккаунтом. Смотрите конфигурацию в [доках по каналам](https://openfang.sh/docs/channels/whatsapp).


---

## 27 поставщиков LLM, 123+ моделей

3 нативных драйвера (Anthropic, Gemini, OpenAI-compatible) маршрутизируют на 27 провайдеров:

Anthropic, Gemini, OpenAI, Groq, DeepSeek, OpenRouter, Together, Mistral, Fireworks, Cohere, Perplexity, xAI, AI21, Cerebras, SambaNova, HuggingFace, Replicate, Ollama, vLLM, LM Studio, Qwen, MiniMax, Zhipu, Moonshot, Qianfan, Bedrock и другие.

Интеллектуальная маршрутизация по сложности задачи, автоматические fallback'ы, учёт затрат и per-model прайсинг.

---

## Миграция из OpenClaw

Уже используете OpenClaw? Одна команда:

```bash
# Мигрирует всё: агенты, память, навыки, конфиги.
openfang migrate --from openclaw

# Миграция из конкретного пути
openfang migrate --from openclaw --path ~/.openclaw

# Сначала dry-run
openfang migrate --from openclaw --dry-run
```

Миграционный движок импортирует агентов, историю разговоров, навыки и конфигурации. OpenFang нативно читает `SKILL.md` и совместим с ClawHub.

---

## OpenAI-совместимый API

Drop-in замена. Направьте существующие инструменты на OpenFang:

```bash
curl -X POST localhost:4200/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "researcher",
    "messages": [{"role": "user", "content": "Analyze Q4 market trends"}],
    "stream": true
  }'
```

140+ REST/WS/SSE эндпойнтов для агентов, памяти, workflow'ов, каналов, моделей, навыков, A2A, Hands и т.д.

---

## Быстрый старт

```bash
# 1. Установка (macOS/Linux)
curl -fsSL https://openfang.sh/install | sh

# 2. Инициализация. Проводит через настройку провайдеров.
openfang init

# 3. Запуск демона
openfang start

# 4. Дашборд: http://localhost:4200

# 5. Активировать Hand. Он начнёт работу.
openfang hand activate researcher

# 6. Поговорить с агентом
openfang chat researcher
> "What are the emerging trends in AI agent frameworks?"

# 7. Запустить преднастроенного агента
openfang agent spawn coder
```

<details>
<summary><strong>Windows (PowerShell)</strong></summary>

```powershell
irm https://openfang.sh/install.ps1 | iex
openfang init
openfang start
```

</details>

---

## Разработка

```bash
# Собрать workspace
cargo build --workspace --lib

# Запустить все тесты (1,767+)
cargo test --workspace

# Lint (должно быть 0 предупреждений)
cargo clippy --workspace --all-targets -- -D warnings

# Форматирование
cargo fmt --all -- --check
```

---

## Заметка о стабильности

OpenFang v0.5.10 — pre-1.0. Архитектура устойчива, тестовый набор обширен, модель безопасности глубока. Тем не менее:

- **Breaking changes** возможны между минорными версиями до v1.0.
- **Некоторые Hands** более зрелые, чем другие. Browser и Researcher наиболее проверены.
- **Возможны edge-case'ы.** Если найдете — [откройте issue](https://github.com/RightNow-AI/openfang/issues).
- **Для production** закрепляйте конкретный коммит до v1.0.

Мы выпускаем быстро и исправляем быстро. Цель — стабильный v1.0 к середине 2026.

---

## Безопасность

Для сообщения об уязвимости пишите на **jaber@rightnowai.co**. Мы рассматриваем сообщения в течение 48 часов.

---

## Лицензия

MIT. Используйте как хотите.

---

## Ссылки

- [Сайт и документация](https://openfang.sh)
- [Быстрый старт](https://openfang.sh/docs/getting-started)
- [GitHub](https://github.com/RightNow-AI/openfang)
- [Discord](https://discord.gg/sSJqgNnq6X)
- [Twitter / X](https://x.com/openfangg)

---

## Сделано RightNow

<p align="center">
  <a href="https://www.rightnowai.co/">
    <img src="public/assets/rightnow-logo.webp" width="60" alt="RightNow Logo" />
  </a>
</p>

<p align="center">
  OpenFang создан и поддерживается <a href="https://x.com/Akashi203"><strong>Jaber</strong></a>, основателем <a href="https://www.rightnowai.co/"><strong>RightNow</strong></a>.
</p>

<p align="center">
  <a href="https://www.rightnowai.co/">Website</a> &bull;
  <a href="https://x.com/Akashi203">Twitter / X</a> &bull;
  <a href="https://www.buymeacoffee.com/openfang" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
</p>

---

<p align="center">
  <strong>Сделано на Rust. Защищено 16 слоями. Агенты, которые действительно работают за вас.</strong>
</p>
