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

````
<p align="center">
  <img src="public/assets/openfang-logo.png" width="160" alt="OpenFang Logo" />
</p>

<h1 align="center">OpenFang</h1>
<h3 align="center">The Agent Operating System</h3>

<p align="center">
  Open-source Agent OS built in Rust. 137K LOC. 14 crates. 1,767+ tests. Zero clippy warnings.<br/>
  <strong>One binary. Battle-tested. Agents that actually work for you.</strong>
</p>

<p align="center">
  <a href="https://openfang.sh/docs">Documentation</a> &bull;
  <a href="https://openfang.sh/docs/getting-started">Quick Start</a> &bull;
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

> **v0.5.10 (April 2026)**
>
> OpenFang is feature complete but still pre-1.0. Expect rough edges and breaking changes between minor versions. We ship fast and fix fast. Pin to a specific commit for production use until v1.0. [Report issues here.](https://github.com/RightNow-AI/openfang/issues)

---

## What is OpenFang?

OpenFang is an **open-source Agent Operating System**. Not a chatbot framework. Not a Python wrapper around an LLM. Not a "multi-agent orchestrator." A full operating system for autonomous agents, built from scratch in Rust.

Traditional agent frameworks wait for you to type something. OpenFang runs **autonomous agents that work for you**: on schedules, 24/7, building knowledge graphs, monitoring targets, generating leads, managing your social media, and reporting results to your dashboard.

The entire system compiles to a **single ~32MB binary**. One install, one command, your agents are live.

```bash
curl -fsSL https://openfang.sh/install | sh
openfang init
openfang start
# Dashboard live at http://localhost:4200
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

## Hands: Agents That Actually Do Things

<p align="center"><em>"Traditional agents wait for you to type. Hands work <strong>for</strong> you."</em></p>

**Hands** are OpenFang's core innovation. Pre-built autonomous capability packages that run independently, on schedules, without you having to prompt them. This is not a chatbot. This is an agent that wakes up at 6 AM, researches your competitors, builds a knowledge graph, scores the findings, and delivers a report to your Telegram before you've had coffee.

Each Hand bundles:
- **HAND.toml**: manifest declaring tools, settings, requirements, and dashboard metrics.
- **System Prompt**: multi-phase operational playbook. Not a one-liner. These are 500+ word expert procedures.
- **SKILL.md**: domain expertise reference injected into context at runtime.
- **Guardrails**: approval gates for sensitive actions (e.g. Browser Hand requires approval before any purchase).

All compiled into the binary. No downloading, no pip install, no Docker pull.

### The 7 Bundled Hands

| Hand | What It Actually Does |
|------|----------------------|
| **Clip** | Takes a YouTube URL, downloads it, identifies the best moments, cuts them into vertical shorts with captions and thumbnails, optionally adds AI voice-over, and publishes to Telegram and WhatsApp. 8-phase pipeline. FFmpeg + yt-dlp + 5 STT backends. |
| **Lead** | Runs daily. Discovers prospects matching your ICP, enriches them with web research, scores 0-100, deduplicates against your existing database, and delivers qualified leads in CSV/JSON/Markdown. Builds ICP profiles over time. |
| **Collector** | OSINT grade intelligence. You give it a target (company, person, topic). It monitors continuously: change detection, sentiment tracking, knowledge graph construction, and critical alerts when something important shifts. |
| **Predictor** | Superforecasting engine. Collects signals from multiple sources, builds calibrated reasoning chains, makes predictions with confidence intervals, and tracks its own accuracy using Brier scores. Has a contrarian mode that deliberately argues against consensus. |
| **Researcher** | Deep autonomous researcher. Cross-references multiple sources, evaluates credibility using CRAAP criteria (Currency, Relevance, Authority, Accuracy, Purpose), generates cited reports with APA formatting, supports multiple languages. |
| **Twitter** | Autonomous Twitter/X account manager. Creates content in 7 rotating formats, schedules posts for optimal engagement, responds to mentions, tracks performance metrics. Has an approval queue, so nothing posts without your OK. |
| **Browser** | Web automation agent. Navigates sites, fills forms, clicks buttons, handles multi-step workflows. Uses Playwright bridge with session persistence. **Mandatory purchase approval gate**: it will never spend your money without explicit confirmation. |

```bash
# Activate the Researcher Hand. It starts working immediately.
openfang hand activate researcher

# Check its progress anytime
openfang hand status researcher

# Activate lead generation on a daily schedule
openfang hand activate lead

# Pause without losing state
openfang hand pause lead

# See all available Hands
openfang hand list
```

**Build your own.** Define a `HAND.toml` with tools, settings, and a system prompt. Publish to FangHub.

---

## OpenFang vs The Landscape

<p align="center">
  <img src="public/assets/openfang-vs-claws.png" width="600" alt="OpenFang vs OpenClaw vs ZeroClaw" />
</p>

### Benchmarks: Measured, Not Marketed

All data from official documentation and public repositories, February 2026.

#### Cold Start Time (lower is better)

```
ZeroClaw   ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10 ms
OpenFang   ██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  180 ms    ★
LangGraph  █████████████████░░░░░░░░░░░░░░░░░░░░░░░░░  2.5 sec
CrewAI     ████████████████████░░░░░░░░░░░░░░░░░░░░░░  3.0 sec
AutoGen    ██████████████████████████░░░░░░░░░░░░░░░░░  4.0 sec
OpenClaw   █████████████████████████████████████████░░  5.98 sec
```

#### Idle Memory Usage (lower is better)

```
ZeroClaw   █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    5 MB
OpenFang   ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   40 MB    ★
LangGraph  ██████████████████░░░░░░░░░░░░░░░░░░░░░░░░░  180 MB
CrewAI     ████████████████████░░░░░░░░░░░░░░░░░░░░░░░  200 MB
AutoGen    █████████████████████████░░░░░░░░░░░░░░░░░░  250 MB
OpenClaw   ████████████████████████████████████████░░░░  394 MB
```

#### Install Size (lower is better)

```
ZeroClaw   █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  8.8 MB
OpenFang   ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   32 MB    ★
CrewAI     ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  100 MB
LangGraph  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  150 MB
AutoGen    ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░  200 MB
OpenClaw   ████████████████████████████████████████░░░░  500 MB
```

#### Security Systems (higher is better)

```
OpenFang   ████████████████████████████████████████████   16      ★
ZeroClaw   ███████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░    6
OpenClaw   ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    3
AutoGen    █████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    2
LangGraph  █████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    2
CrewAI     ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    1
```

#### Channel Adapters (higher is better)

```
OpenFang   ████████████████████████████████████████████   40      ★
ZeroClaw   ███████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░   15
OpenClaw   █████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   13
CrewAI     ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
AutoGen    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
LangGraph  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    0
```

#### LLM Providers (higher is better)

```
ZeroClaw   ████████████████████████████████████████████   28
OpenFang   ██████████████████████████████████████████░░   27      ★
LangGraph  ██████████████████████░░░░░░░░░░░░░░░░░░░░░   15
CrewAI     ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10
OpenClaw   ██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   10
AutoGen    ███████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░    8
```

### Feature-by-Feature Comparison

| Feature | OpenFang | OpenClaw | ZeroClaw | CrewAI | AutoGen | LangGraph |
|---------|----------|----------|----------|--------|---------|-----------|
| **Language** | **Rust** | TypeScript | **Rust** | Python | Python | Python |
| **Autonomous Hands** | **7 built-in** | None | None | None | None | None |
| **Security Layers** | **16 discrete** | 3 basic | 6 layers | 1 basic | Docker | AES enc. |
| **Agent Sandbox** | **WASM dual-metered** | None | Allowlists | None | Docker | None |
| **Channel Adapters** | **40** | 13 | 15 | 0 | 0 | 0 |
| **Built-in Tools** | **53 + MCP + A2A** | 50+ | 12 | Plugins | MCP | LC tools |
| **Memory** | **SQLite + vector** | File-based | SQLite FTS5 | 4-layer | External | Checkpoints |
| **Desktop App** | **Tauri 2.0** | None | None | None | Studio | None |
| **Audit Trail** | **Merkle hash-chain** | Logs | Logs | Tracing | Logs | Checkpoints |
| **Cold Start** | **<200ms** | ~6s | ~10ms | ~3s | ~4s | ~2.5s |
| **Install Size** | **~32 MB** | ~500 MB | ~8.8 MB | ~100 MB | ~200 MB | ~150 MB |
| **License** | MIT | MIT | MIT | MIT | Apache 2.0 | MIT |

---

## 16 Security Systems: Defense in Depth

OpenFang doesn't bolt security on after the fact. Every layer is independently testable and operates without a single point of failure.

| # | System | What It Does |
|---|--------|-------------|
| 1 | **WASM Dual-Metered Sandbox** | Tool code runs in WebAssembly with fuel metering + epoch interruption. A watchdog thread kills runaway code. |
| 2 | **Merkle Hash-Chain Audit Trail** | Every action is cryptographically linked to the previous one. Tamper with one entry and the entire chain breaks. |
| 3 | **Information Flow Taint Tracking** | Labels propagate through execution. Secrets are tracked from source to sink. |
| 4 | **Ed25519 Signed Agent Manifests** | Every agent identity and capability set is cryptographically signed. |
| 5 | **SSRF Protection** | Blocks private IPs, cloud metadata endpoints, and DNS rebinding attacks. |
| 6 | **Secret Zeroization** | `Zeroizing<String>` auto-wipes API keys from memory the instant they're no longer needed. |
| 7 | **OFP Mutual Authentication** | HMAC-SHA256 nonce-based, constant-time verification for P2P networking. |
| 8 | **Capability Gates** | Role based access control. Agents declare required tools, the kernel enforces it. |
| 9 | **Security Headers** | CSP, X-Frame-Options, HSTS, X-Content-Type-Options on every response. |
| 10 | **Health Endpoint Redaction** | Public health check returns minimal info. Full diagnostics require authentication. |
| 11 | **Subprocess Sandbox** | `env_clear()` + selective variable passthrough. Process tree isolation with cross-platform kill. |
| 12 | **Prompt Injection Scanner** | Detects override attempts, data exfiltration patterns, and shell reference injection in skills. |
| 13 | **Loop Guard** | SHA256-based tool call loop detection with circuit breaker. Handles ping-pong patterns. |
| 14 | **Session Repair** | 7-phase message history validation and automatic recovery from corruption. |
| 15 | **Path Traversal Prevention** | Canonicalization with symlink escape prevention. ``../`` doesn't work here. |
| 16 | **GCRA Rate Limiter** | Cost-aware token bucket rate limiting with per-IP tracking and stale cleanup. |

---

## Architecture

14 Rust crates. 137,728 lines of code. Modular kernel design.

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

## 40 Channel Adapters

Connect your agents to every platform your users are on.

**Core:** Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Email (IMAP/SMTP)
**Enterprise:** Microsoft Teams, Mattermost, Google Chat, Webex, Feishu/Lark, Zulip
**Social:** LINE, Viber, Facebook Messenger, Mastodon, Bluesky, Reddit, LinkedIn, Twitch
**Community:** IRC, XMPP, Guilded, Revolt, Keybase, Discourse, Gitter
**Privacy:** Threema, Nostr, Mumble, Nextcloud Talk, Rocket.Chat, Ntfy, Gotify
**Workplace:** Pumble, Flock, Twist, DingTalk, Zalo, Webhooks

Each adapter supports per-channel model overrides, DM/group policies, rate limiting, and output formatting.

---

## WhatsApp Web Gateway (QR Code)

Connect your personal WhatsApp account to OpenFang via QR code, just like WhatsApp Web. No Meta Business account required.

### Prerequisites

- **Node.js >= 18** installed ([download](https://nodejs.org/))
- OpenFang installed and initialized

### Setup

**1. Install the gateway dependencies:**

```bash
cd packages/whatsapp-gateway
npm install
```

**2. Configure `config.toml`:**

```toml
[channels.whatsapp]
mode = "web"
default_agent = "assistant"
```

**3. Set the gateway URL (choose one):**

Add to your shell profile for persistence:

```bash
# macOS / Linux
echo 'export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"' >> ~/.zshrc
source ~/.zshrc
```

Or set it inline when starting the gateway:

```bash
export WHATSAPP_WEB_GATEWAY_URL="http://127.0.0.1:3009"
```

**4. Start the gateway:**

```bash
node packages/whatsapp-gateway/index.js
```

The gateway listens on port `3009` by default. Override with `WHATSAPP_GATEWAY_PORT`.

**5. Start OpenFang:**

```bash
openfang start
# Dashboard at http://localhost:4200
```

**6. Scan the QR code:**

Open the dashboard → **Channels** → **WhatsApp**. A QR code will appear. Scan it with your phone:

> **WhatsApp** → **Settings** → **Linked Devices** → **Link a Device**

Once scanned, the status changes to `connected` and incoming messages are routed to your configured agent.

### Gateway Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WHATSAPP_WEB_GATEWAY_URL` | Gateway URL for OpenFang to connect to | _(empty = disabled)_ |
| `WHATSAPP_GATEWAY_PORT` | Port the gateway listens on | `3009` |
| `OPENFANG_URL` | OpenFang API URL the gateway reports to | `http://127.0.0.1:4200` |
| `OPENFANG_DEFAULT_AGENT` | Agent that handles incoming messages | `assistant` |

### Gateway API Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| `POST` | `/login/start` | Generate QR code (returns base64 PNG) |
| `GET` | `/login/status` | Connection status (`disconnected`, `qr_ready`, `connected`) |
| `POST` | `/message/send` | Send a message (`{ "to": "5511999999999", "text": "Hello" }`) |
| `GET` | `/health` | Health check |

### Alternative: WhatsApp Cloud API

For production workloads, use the [WhatsApp Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api) with a Meta Business account. See the [Cloud API configuration docs](https://openfang.sh/docs/channels/whatsapp).



---

## 27 LLM Providers, 123+ Models

3 native drivers (Anthropic, Gemini, OpenAI-compatible) route to 27 providers:

Anthropic, Gemini, OpenAI, Groq, DeepSeek, OpenRouter, Together, Mistral, Fireworks, Cohere, Perplexity, xAI, AI21, Cerebras, SambaNova, HuggingFace, Replicate, Ollama, vLLM, LM Studio, Qwen, MiniMax, Zhipu, Moonshot, Qianfan, Bedrock, and more.

Intelligent routing with task complexity scoring, automatic fallback, cost tracking, and per-model pricing.

---

## Migrate from OpenClaw

Already running OpenClaw? One command:

```bash
# Migrate everything: agents, memory, skills, configs.
openfang migrate --from openclaw

# Migrate from a specific path
openfang migrate --from openclaw --path ~/.openclaw

# Dry run first to see what would change
openfang migrate --from openclaw --dry-run
```

The migration engine imports your agents, conversation history, skills, and configuration. OpenFang reads SKILL.md natively and is compatible with the ClawHub marketplace.

---

## OpenAI-Compatible API

Drop-in replacement. Point your existing tools at OpenFang:

```bash
curl -X POST localhost:4200/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "researcher",
    "messages": [{"role": "user", "content": "Analyze Q4 market trends"}],
    "stream": true
  }'
```

140+ REST/WS/SSE endpoints covering agents, memory, workflows, channels, models, skills, A2A, Hands, and more.

---

## Quick Start

```bash
# 1. Install (macOS/Linux)
curl -fsSL https://openfang.sh/install | sh

# 2. Initialize. Walks you through provider setup.
openfang init

# 3. Start the daemon
openfang start

# 4. Dashboard is live at http://localhost:4200

# 5. Activate a Hand. It starts working for you.
openfang hand activate researcher

# 6. Chat with an agent
openfang chat researcher
> "What are the emerging trends in AI agent frameworks?"

# 7. Spawn a pre-built agent
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

## Development

```bash
# Build the workspace
cargo build --workspace --lib

# Run all tests (1,767+)
cargo test --workspace

# Lint (must be 0 warnings)
cargo clippy --workspace --all-targets -- -D warnings

# Format
cargo fmt --all -- --check
```

---

## Stability Notice

OpenFang v0.5.10 is pre-1.0. The architecture is solid, the test suite is comprehensive, and the security model is deep. That said:

- **Breaking changes** may occur between minor versions until v1.0.
- **Some Hands** are more mature than others. Browser and Researcher are the most battle tested.
- **Edge cases** exist. If you find one, [open an issue](https://github.com/RightNow-AI/openfang/issues).
- **Pin to a specific commit** for production deployments until v1.0.

We ship fast and fix fast. The goal is a rock solid v1.0 by mid 2026.

---

## Security

To report a security vulnerability, email **jaber@rightnowai.co**. We take all reports seriously and will respond within 48 hours.

---

## License

MIT. Use it however you want.

---

## Links

- [Website & Documentation](https://openfang.sh)
- [Quick Start Guide](https://openfang.sh/docs/getting-started)
- [GitHub](https://github.com/RightNow-AI/openfang)
- [Discord](https://discord.gg/sSJqgNnq6X)
- [Twitter / X](https://x.com/openfangg)

---

## Built by RightNow

<p align="center">
  <a href="https://www.rightnowai.co/">
    <img src="public/assets/rightnow-logo.webp" width="60" alt="RightNow Logo" />
  </a>
</p>

<p align="center">
  OpenFang is built and maintained by <a href="https://x.com/Akashi203"><strong>Jaber</strong></a>, Founder of <a href="https://www.rightnowai.co/"><strong>RightNow</strong></a>.
</p>

<p align="center">
  <a href="https://www.rightnowai.co/">Website</a> &bull;
  <a href="https://x.com/Akashi203">Twitter / X</a> &bull;
  <a href="https://www.buymeacoffee.com/openfang" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
</p>

---

<p align="center">
  <strong>Built with Rust. Secured with 16 layers. Agents that actually work for you.</strong>
</p>
