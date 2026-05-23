# Провайдеры

Список и заметки по настройке поддерживаемых провайдеров LLM.
Поддерживаемые провайдеры: Anthropic, Gemini, OpenAI, Groq, DeepSeek, OpenRouter, Together, Mistral, Fireworks, Cohere, Perplexity, xAI, AI21, Cerebras, SambaNova, HuggingFace, Replicate, Ollama, vLLM, LM Studio, Qwen, MiniMax, Zhipu, Moonshot, Qianfan, Bedrock и другие.

Указывайте провайдера по умолчанию в `config.toml` в секции `[default_model]`. Каждый агент может переопределить провайдера в своём манифесте.

### Общая настройка

1. Установите API-ключ в переменной окружения, например `export GROQ_API_KEY=...`.
2. Укажите `api_key_env` в `config.toml` или в манифесте агента.

### Локальные (self-hosted) провайдеры

Для локальных раннеров (Ollama, vLLM, LM Studio) укажите `base_url` и оставьте `api_key_env` пустым.

```toml
[default_model]
provider = "ollama"
model = "llama-3.2"
base_url = "http://localhost:11434"
api_key_env = ""
```

# Руководство по провайдерам LLM

OpenFang поставляется с обширным каталогом моделей, включающим **3 нативных драйвера LLM**, **20 провайдеров**, **51 встроенную модель** и **23 алиаса (псевдонима)**. Каждый провайдер использует один из трех проверенных временем драйверов: нативный драйвер **Anthropic**, нативный драйвер **Gemini** или универсальный драйвер, **совместимый с OpenAI**. Это руководство является основным источником информации по настройке, выбору и управлению провайдерами LLM в OpenFang.

---

## Содержание

1. [Быстрая настройка](#quick-setup)
2. [Справочник провайдеров](#provider-reference)
3. [Каталог моделей](#model-catalog)
4. [Алиасы моделей](#model-aliases)
5. [Переопределение модели для конкретного агента](#per-agent-model-override)
6. [Маршрутизация моделей](#model-routing)
7. [Отслеживание затрат](#cost-tracking)
8. [Резервные провайдеры (Fallback)](#fallback-providers)
9. [Эндпоинты API](#api-endpoints)
10. [Команды каналов](#channel-commands)

---

<a name="quick-setup"></a>
## Быстрая настройка

Самый быстрый путь от установки до запуска:

```bash
# Выберите ОДНОГО провайдера — установите его переменную окружения — готово.
export GEMINI_API_KEY="your-key"        # Доступен бесплатный уровень
# ИЛИ
export GROQ_API_KEY="your-key"          # Доступен бесплатный уровень
# ИЛИ
export ANTHROPIC_API_KEY="your-key"
# ИЛИ
export OPENAI_API_KEY="your-key"
```

OpenFang при запуске автоматически определяет, для каких провайдеров настроены API-ключи. Любая модель, провайдер которой прошел аутентификацию, становится доступной немедленно. Локальные провайдеры (Ollama, vLLM, LM Studio) вообще не требуют ключа.

Для Gemini подходят переменные `GEMINI_API_KEY` или `GOOGLE_API_KEY`.

---

<a name="provider-reference"></a>
## Справочник провайдеров

### 1. Anthropic

| | |
|---|---|
| **Отображаемое имя** | Anthropic |
| **Драйвер** | Native Anthropic (Messages API) |
| **Переменная окружения** | `ANTHROPIC_API_KEY` |
| **Base URL** | `https://api.anthropic.com` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `x-api-key` |
| **Модели** | 3 |

**Доступные модели:**
- `claude-opus-4-20250514` (Frontier)
- `claude-sonnet-4-20250514` (Smart)
- `claude-haiku-4-5-20251001` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [console.anthropic.com](https://console.anthropic.com)
2. Создайте API-ключ в разделе Settings > API Keys
3. `export ANTHROPIC_API_KEY="sk-ant-..."`

---

### 2. OpenAI

| | |
|---|---|
| **Отображаемое имя** | OpenAI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `OPENAI_API_KEY` |
| **Base URL** | `https://api.openai.com/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 6 |

**Доступные модели:**
- `gpt-4.1` (Frontier)
- `gpt-4o` (Smart)
- `o3-mini` (Smart)
- `gpt-4.1-mini` (Balanced)
- `gpt-4o-mini` (Fast)
- `gpt-4.1-nano` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [platform.openai.com](https://platform.openai.com)
2. Создайте API-ключ в разделе API Keys
3. `export OPENAI_API_KEY="sk-..."`

---

### 3. Google Gemini

| | |
|---|---|
| **Отображаемое имя** | Google Gemini |
| **Драйвер** | Native Gemini (generateContent API) |
| **Переменная окружения** | `GEMINI_API_KEY` (или `GOOGLE_API_KEY`) |
| **Base URL** | `https://generativelanguage.googleapis.com` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (щедрый бесплатный уровень) |
| **Аутентификация** | заголовок `x-goog-api-key` |
| **Модели** | 3 |

**Доступные модели:**
- `gemini-2.5-pro` (Frontier)
- `gemini-2.5-flash` (Smart)
- `gemini-2.0-flash` (Fast)

**Настройка:**
1. Перейдите на [aistudio.google.com](https://aistudio.google.com)
2. Получите API-ключ (бесплатный уровень включен)
3. `export GEMINI_API_KEY="AIza..."` или `export GOOGLE_API_KEY="AIza..."`

**Примечание:** Драйвер Gemini является полностью нативной реализацией. Он не совместим с OpenAI. Модель указывается в пути URL, системный промпт — через `systemInstruction`, инструменты — через `functionDeclarations`, потоковая передача — через `streamGenerateContent?alt=sse`.

---

### 4. DeepSeek

| | |
|---|---|
| **Отображаемое имя** | DeepSeek |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `DEEPSEEK_API_KEY` |
| **Base URL** | `https://api.deepseek.com/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `deepseek-chat` (Smart) — DeepSeek V3
- `deepseek-reasoner` (Smart) — DeepSeek R1, без поддержки инструментов

**Настройка:**
1. Зарегистрируйтесь на [platform.deepseek.com](https://platform.deepseek.com)
2. Создайте API-ключ
3. `export DEEPSEEK_API_KEY="sk-..."`

---

### 5. Groq

| | |
|---|---|
| **Отображаемое имя** | Groq |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `GROQ_API_KEY` |
| **Base URL** | `https://api.groq.com/openai/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (с ограничениями скорости) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 4 |

**Доступные модели:**
- `llama-3.3-70b-versatile` (Balanced)
- `mixtral-8x7b-32768` (Balanced)
- `llama-3.1-8b-instant` (Fast)
- `gemma2-9b-it` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [console.groq.com](https://console.groq.com)
2. Создайте API-ключ
3. `export GROQ_API_KEY="gsk_..."`

**Примечание:** Groq запускает модели с открытым исходным кодом на специализированном оборудовании LPU. Чрезвычайно быстрый вывод. Бесплатный уровень имеет ограничения по скорости, но вполне пригоден для использования.

---

### 6. OpenRouter

| | |
|---|---|
| **Отображаемое имя** | OpenRouter |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `OPENROUTER_API_KEY` |
| **Base URL** | `https://openrouter.ai/api/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные кредиты для некоторых моделей) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 10 |

**Доступные модели:**
- `openrouter/google/gemini-2.5-flash` (Smart) — дешевая, быстрая, контекст 1M (по умолчанию)
- `openrouter/anthropic/claude-sonnet-4` (Smart) — сильные рассуждения + инструменты
- `openrouter/openai/gpt-4o` (Smart) — GPT-4o через OpenRouter
- `openrouter/deepseek/deepseek-chat` (Smart) — DeepSeek V3
- `openrouter/meta-llama/llama-3.3-70b-instruct` (Balanced) — Llama 3.3 70B
- `openrouter/qwen/qwen-2.5-72b-instruct` (Balanced) — Qwen 2.5 72B
- `openrouter/google/gemini-2.5-pro` (Frontier) — Gemini 2.5 Pro
- `openrouter/mistralai/mistral-large-latest` (Smart) — Mistral Large
- `openrouter/google/gemma-2-9b-it` (Fast) — Gemma 2 9B, бесплатно
- `openrouter/deepseek/deepseek-r1` (Frontier) — DeepSeek R1 (рассуждения)

**Настройка:**
1. Зарегистрируйтесь на [openrouter.ai](https://openrouter.ai)
2. Создайте API-ключ в разделе Keys
3. `export OPENROUTER_API_KEY="sk-or-..."`

**Примечание:** OpenRouter — это единый шлюз к более чем 200 моделям от множества провайдеров. Идентификаторы моделей используют формат вышестоящего провайдера (например, `google/gemini-2.5-flash`). Вы можете использовать любую модель из каталога OpenRouter, указав полный путь к модели с префиксом `openrouter/`.

---

### 7. Mistral AI

| | |
|---|---|
| **Отображаемое имя** | Mistral AI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `MISTRAL_API_KEY` |
| **Base URL** | `https://api.mistral.ai/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 3 |

**Доступные модели:**
- `mistral-large-latest` (Smart)
- `codestral-latest` (Smart)
- `mistral-small-latest` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [console.mistral.ai](https://console.mistral.ai)
2. Создайте API-ключ
3. `export MISTRAL_API_KEY="..."`

---

### 8. Together AI

| | |
|---|---|
| **Отображаемое имя** | Together AI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `TOGETHER_API_KEY` |
| **Base URL** | `https://api.together.xyz/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные кредиты при регистрации) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 3 |

**Доступные модели:**
- `meta-llama/Meta-Llama-3.1-405B-Instruct-Turbo` (Frontier)
- `Qwen/Qwen2.5-72B-Instruct-Turbo` (Smart)
- `mistralai/Mixtral-8x22B-Instruct-v0.1` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [api.together.ai](https://api.together.ai)
2. Создайте API-ключ
3. `export TOGETHER_API_KEY="..."`

---

### 9. Fireworks AI

| | |
|---|---|
| **Отображаемое имя** | Fireworks AI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `FIREWORKS_API_KEY` |
| **Base URL** | `https://api.fireworks.ai/inference/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные кредиты при регистрации) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `accounts/fireworks/models/llama-v3p1-405b-instruct` (Frontier)
- `accounts/fireworks/models/mixtral-8x22b-instruct` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [fireworks.ai](https://fireworks.ai)
2. Создайте API-ключ
3. `export FIREWORKS_API_KEY="..."`

---

### 10. Ollama

| | |
|---|---|
| **Отображаемое имя** | Ollama |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `OLLAMA_API_KEY` (не требуется) |
| **Base URL** | `http://localhost:11434/v1` |
| **Требуется ключ** | **Нет** |
| **Бесплатный уровень** | Бесплатно (локально) |
| **Аутентификация** | Нет (локально) |
| **Модели** | 3 встроенных + автоопределение |

**Доступные модели (встроенные):**
- `llama3.2` (Local)
- `mistral:latest` (Local)
- `phi3` (Local)

**Настройка:**
1. Установите Ollama с [ollama.com](https://ollama.com)
2. Скачайте модель: `ollama pull llama3.2`
3. Запустите сервер: `ollama serve`
4. Переменная окружения не нужна — Ollama всегда доступна

**Примечание:** OpenFang автоматически обнаруживает модели из запущенного экземпляра Ollama и объединяет их в каталог с уровнем `Local` и нулевой стоимостью. Любая скачанная вами модель сразу становится доступной для использования.

---

### 11. vLLM

| | |
|---|---|
| **Отображаемое имя** | vLLM |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `VLLM_API_KEY` (не требуется) |
| **Base URL** | `http://localhost:8000/v1` |
| **Требуется ключ** | **Нет** |
| **Бесплатный уровень** | Бесплатно (self-hosted) |
| **Аутентификация** | Нет (локально) |
| **Модели** | 1 встроенная + автоопределение |

**Доступные модели (встроенные):**
- `vllm-local` (Local)

**Настройка:**
1. Установите vLLM: `pip install vllm`
2. Запустите сервер: `python -m vllm.entrypoints.openai.api_server --model <model-name>`
3. Переменная окружения не нужна

---

### 12. LM Studio

| | |
|---|---|
| **Отображаемое имя** | LM Studio |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `LMSTUDIO_API_KEY` (не требуется) |
| **Base URL** | `http://localhost:1234/v1` |
| **Требуется ключ** | **Нет** |
| **Бесплатный уровень** | Бесплатно (локально) |
| **Аутентификация** | Нет (локально) |
| **Модели** | 1 встроенная + автоопределение |

**Доступные модели (встроенные):**
- `lmstudio-local` (Local)

**Настройка:**
1. Скачайте LM Studio с [lmstudio.ai](https://lmstudio.ai)
2. Скачайте модель через встроенный браузер моделей
3. Запустите локальный сервер на вкладке "Local Server"
4. Переменная окружения не нужна

---

### 13. Perplexity AI

| | |
|---|---|
| **Отображаемое имя** | Perplexity AI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `PERPLEXITY_API_KEY` |
| **Base URL** | `https://api.perplexity.ai` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `sonar-pro` (Smart) — онлайн-поиск
- `sonar` (Balanced) — онлайн-поиск

**Настройка:**
1. Зарегистрируйтесь на [perplexity.ai](https://www.perplexity.ai)
2. Перейдите в настройки API и сгенерируйте ключ
3. `export PERPLEXITY_API_KEY="pplx-..."`

**Примечание:** Модели Perplexity имеют встроенный веб-поиск. Они не поддерживают использование инструментов (tools).

---

### 14. Cohere

| | |
|---|---|
| **Отображаемое имя** | Cohere |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `COHERE_API_KEY` |
| **Base URL** | `https://api.cohere.com/v2` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (пробный период с ограничениями) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `command-r-plus` (Smart)
- `command-r` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [dashboard.cohere.com](https://dashboard.cohere.com)
2. Создайте API-ключ
3. `export COHERE_API_KEY="..."`

---

### 15. AI21 Labs

| | |
|---|---|
| **Отображаемое имя** | AI21 Labs |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `AI21_API_KEY` |
| **Base URL** | `https://api.ai21.com/studio/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные кредиты) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 1 |

**Доступные модели:**
- `jamba-1.5-large` (Smart)

**Настройка:**
1. Зарегистрируйтесь на [studio.ai21.com](https://studio.ai21.com)
2. Создайте API-ключ
3. `export AI21_API_KEY="..."`

---

### 16. Cerebras

| | |
|---|---|
| **Отображаемое имя** | Cerebras |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `CEREBRAS_API_KEY` |
| **Base URL** | `https://api.cerebras.ai/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (щедрый бесплатный уровень) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `cerebras/llama3.3-70b` (Balanced)
- `cerebras/llama3.1-8b` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [cloud.cerebras.ai](https://cloud.cerebras.ai)
2. Создайте API-ключ
3. `export CEREBRAS_API_KEY="..."`

**Примечание:** Cerebras запускает инференс на чипах размером с целую пластину. Ультрабыстро и ультрадешево ($0.06 за миллион токенов как на вход, так и на выход для модели 70B).

---

### 17. SambaNova

| | |
|---|---|
| **Отображаемое имя** | SambaNova |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `SAMBANOVA_API_KEY` |
| **Base URL** | `https://api.sambanova.ai/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные кредиты) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 1 |

**Доступные модели:**
- `sambanova/llama-3.3-70b` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [cloud.sambanova.ai](https://cloud.sambanova.ai)
2. Создайте API-ключ
3. `export SAMBANOVA_API_KEY="..."`

---

### 18. Hugging Face

| | |
|---|---|
| **Отображаемое имя** | Hugging Face |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `HF_API_KEY` |
| **Base URL** | `https://api-inference.huggingface.co/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (с ограничениями скорости) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 1 |

**Доступные модели:**
- `hf/meta-llama/Llama-3.3-70B-Instruct` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [huggingface.co](https://huggingface.co)
2. Создайте токен в разделе Settings > Access Tokens
3. `export HF_API_KEY="hf_..."`

---

### 19. xAI

| | |
|---|---|
| **Отображаемое имя** | xAI |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `XAI_API_KEY` |
| **Base URL** | `https://api.x.ai/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Да (ограниченные бесплатные кредиты) |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 2 |

**Доступные модели:**
- `grok-2` (Smart) — поддерживает зрение (vision)
- `grok-2-mini` (Fast)

**Настройка:**
1. Зарегистрируйтесь на [console.x.ai](https://console.x.ai)
2. Создайте API-ключ
3. `export XAI_API_KEY="xai-..."`

---

### 20. Replicate

| | |
|---|---|
| **Отображаемое имя** | Replicate |
| **Драйвер** | OpenAI-compatible |
| **Переменная окружения** | `REPLICATE_API_TOKEN` |
| **Base URL** | `https://api.replicate.com/v1` |
| **Требуется ключ** | Да |
| **Бесплатный уровень** | Нет |
| **Аутентификация** | заголовок `Authorization: Bearer` |
| **Модели** | 1 |

**Доступные модели:**
- `replicate/meta-llama-3.3-70b-instruct` (Balanced)

**Настройка:**
1. Зарегистрируйтесь на [replicate.com](https://replicate.com)
2. Перейдите в раздел Account > API Tokens
3. `export REPLICATE_API_TOKEN="r8_..."`

---

<a name="model-catalog"></a>
## Каталог моделей

Полный каталог всех 51 встроенной модели, отсортированный по провайдерам. Цены указаны за миллион токенов.

| # | Model ID | Отображаемое имя | Провайдер | Уровень (Tier) | Окно контекста | Макс. выход | Вход $/M | Выход $/M | Tools | Vision |
|---|----------|-------------|----------|------|---------------|------------|-----------|------------|-------|--------|
| 1 | `claude-opus-4-20250514` | Claude Opus 4 | anthropic | Frontier | 200,000 | 32,000 | $15.00 | $75.00 | Да | Да |
| 2 | `claude-sonnet-4-20250514` | Claude Sonnet 4 | anthropic | Smart | 200,000 | 64,000 | $3.00 | $15.00 | Да | Да |
| 3 | `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | anthropic | Fast | 200,000 | 8,192 | $0.25 | $1.25 | Да | Да |
| 4 | `gpt-4.1` | GPT-4.1 | openai | Frontier | 1,047,576 | 32,768 | $2.00 | $8.00 | Да | Да |
| 5 | `gpt-4o` | GPT-4o | openai | Smart | 128,000 | 16,384 | $2.50 | $10.00 | Да | Да |
| 6 | `o3-mini` | o3-mini | openai | Smart | 200,000 | 100,000 | $1.10 | $4.40 | Да | Нет |
| 7 | `gpt-4.1-mini` | GPT-4.1 Mini | openai | Balanced | 1,047,576 | 32,768 | $0.40 | $1.60 | Да | Да |
| 8 | `gpt-4o-mini` | GPT-4o Mini | openai | Fast | 128,000 | 16,384 | $0.15 | $0.60 | Да | Да |
| 9 | `gpt-4.1-nano` | GPT-4.1 Nano | openai | Fast | 1,047,576 | 32,768 | $0.10 | $0.40 | Да | Нет |
| 10 | `gemini-2.5-pro` | Gemini 2.5 Pro | gemini | Frontier | 1,048,576 | 65,536 | $1.25 | $10.00 | Да | Да |
| 11 | `gemini-2.5-flash` | Gemini 2.5 Flash | gemini | Smart | 1,048,576 | 65,536 | $0.15 | $0.60 | Да | Да |
| 12 | `gemini-2.0-flash` | Gemini 2.0 Flash | gemini | Fast | 1,048,576 | 8,192 | $0.10 | $0.40 | Да | Да |
| 13 | `deepseek-chat` | DeepSeek V3 | deepseek | Smart | 64,000 | 8,192 | $0.27 | $1.10 | Да | Нет |
| 14 | `deepseek-reasoner` | DeepSeek R1 | deepseek | Smart | 64,000 | 8,192 | $0.55 | $2.19 | Нет | Нет |
| 15 | `llama-3.3-70b-versatile` | Llama 3.3 70B | groq | Balanced | 128,000 | 32,768 | $0.059 | $0.079 | Да | Нет |
| 16 | `mixtral-8x7b-32768` | Mixtral 8x7B | groq | Balanced | 32,768 | 4,096 | $0.024 | $0.024 | Да | Нет |
| 17 | `llama-3.1-8b-instant` | Llama 3.1 8B | groq | Fast | 128,000 | 8,192 | $0.05 | $0.08 | Да | Нет |
| 18 | `gemma2-9b-it` | Gemma 2 9B | groq | Fast | 8,192 | 4,096 | $0.02 | $0.02 | Нет | Нет |
| 19 | `openrouter/google/gemini-2.5-flash` | Gemini 2.5 Flash (OpenRouter) | openrouter | Smart | 1,048,576 | 65,536 | $0.15 | $0.60 | Да | Да |
| 20 | `openrouter/anthropic/claude-sonnet-4` | Claude Sonnet 4 (OpenRouter) | openrouter | Smart | 200,000 | 64,000 | $3.00 | $15.00 | Да | Да |
| 21 | `openrouter/openai/gpt-4o` | GPT-4o (OpenRouter) | openrouter | Smart | 128,000 | 16,384 | $2.50 | $10.00 | Да | Да |
| 22 | `openrouter/deepseek/deepseek-chat` | DeepSeek V3 (OpenRouter) | openrouter | Smart | 128,000 | 32,768 | $0.14 | $0.28 | Да | Нет |
| 23 | `openrouter/meta-llama/llama-3.3-70b-instruct` | Llama 3.3 70B (OpenRouter) | openrouter | Balanced | 128,000 | 32,768 | $0.39 | $0.39 | Да | Нет |
| 24 | `openrouter/qwen/qwen-2.5-72b-instruct` | Qwen 2.5 72B (OpenRouter) | openrouter | Balanced | 128,000 | 32,768 | $0.36 | $0.36 | Да | Нет |
| 25 | `openrouter/google/gemini-2.5-pro` | Gemini 2.5 Pro (OpenRouter) | openrouter | Frontier | 1,048,576 | 65,536 | $1.25 | $10.00 | Да | Да |
| 26 | `openrouter/mistralai/mistral-large-latest` | Mistral Large (OpenRouter) | openrouter | Smart | 128,000 | 8,192 | $2.00 | $6.00 | Да | Нет |
| 27 | `openrouter/google/gemma-2-9b-it` | Gemma 2 9B (OpenRouter) | openrouter | Fast | 8,192 | 4,096 | $0.00 | $0.00 | Нет | Нет |
| 28 | `openrouter/deepseek/deepseek-r1` | DeepSeek R1 (OpenRouter) | openrouter | Frontier | 128,000 | 32,768 | $0.55 | $2.19 | Нет | Нет |
| 29 | `mistral-large-latest` | Mistral Large | mistral | Smart | 128,000 | 8,192 | $2.00 | $6.00 | Да | Нет |
| 30 | `codestral-latest` | Codestral | mistral | Smart | 32,000 | 8,192 | $0.30 | $0.90 | Да | Нет |
| 31 | `mistral-small-latest` | Mistral Small | mistral | Fast | 128,000 | 8,192 | $0.10 | $0.30 | Да | Нет |
| 32 | `meta-llama/Meta-Llama-3.1-405B-Instruct-Turbo` | Llama 3.1 405B (Together) | together | Frontier | 130,000 | 4,096 | $3.50 | $3.50 | Да | Нет |
| 33 | `Qwen/Qwen2.5-72B-Instruct-Turbo` | Qwen 2.5 72B (Together) | together | Smart | 32,768 | 4,096 | $0.20 | $0.60 | Да | Нет |
| 34 | `mistralai/Mixtral-8x22B-Instruct-v0.1` | Mixtral 8x22B (Together) | together | Balanced | 65,536 | 4,096 | $0.60 | $0.60 | Да | Нет |
| 35 | `accounts/fireworks/models/llama-v3p1-405b-instruct` | Llama 3.1 405B (Fireworks) | fireworks | Frontier | 131,072 | 16,384 | $3.00 | $3.00 | Да | Нет |
| 36 | `accounts/fireworks/models/mixtral-8x22b-instruct` | Mixtral 8x22B (Fireworks) | fireworks | Balanced | 65,536 | 4,096 | $0.90 | $0.90 | Да | Нет |
| 37 | `llama3.2` | Llama 3.2 (Ollama) | ollama | Local | 128,000 | 4,096 | $0.00 | $0.00 | Да | Нет |
| 38 | `mistral:latest` | Mistral (Ollama) | ollama | Local | 32,768 | 4,096 | $0.00 | $0.00 | Да | Нет |
| 39 | `phi3` | Phi-3 (Ollama) | ollama | Local | 128,000 | 4,096 | $0.00 | $0.00 | Нет | Нет |
| 40 | `vllm-local` | vLLM Local Model | vllm | Local | 32,768 | 4,096 | $0.00 | $0.00 | Да | Нет |
| 41 | `lmstudio-local` | LM Studio Local Model | lmstudio | Local | 32,768 | 4,096 | $0.00 | $0.00 | Да | Нет |
| 42 | `sonar-pro` | Sonar Pro | perplexity | Smart | 200,000 | 8,192 | $3.00 | $15.00 | Нет | Нет |
| 43 | `sonar` | Sonar | perplexity | Balanced | 128,000 | 8,192 | $1.00 | $5.00 | Нет | Нет |
| 44 | `command-r-plus` | Command R+ | cohere | Smart | 128,000 | 4,096 | $2.50 | $10.00 | Да | Нет |
| 45 | `command-r` | Command R | cohere | Balanced | 128,000 | 4,096 | $0.15 | $0.60 | Да | Нет |
| 46 | `jamba-1.5-large` | Jamba 1.5 Large | ai21 | Smart | 256,000 | 4,096 | $2.00 | $8.00 | Да | Нет |
| 47 | `cerebras/llama3.3-70b` | Llama 3.3 70B (Cerebras) | cerebras | Balanced | 128,000 | 8,192 | $0.06 | $0.06 | Да | Нет |
| 48 | `cerebras/llama3.1-8b` | Llama 3.1 8B (Cerebras) | cerebras | Fast | 128,000 | 8,192 | $0.01 | $0.01 | Да | Нет |
| 49 | `sambanova/llama-3.3-70b` | Llama 3.3 70B (SambaNova) | sambanova | Balanced | 128,000 | 8,192 | $0.06 | $0.06 | Да | Нет |
| 50 | `grok-2` | Grok 2 | xai | Smart | 131,072 | 32,768 | $2.00 | $10.00 | Да | Да |
| 51 | `grok-2-mini` | Grok 2 Mini | xai | Fast | 131,072 | 32,768 | $0.30 | $0.50 | Да | Нет |
| 52 | `hf/meta-llama/Llama-3.3-70B-Instruct` | Llama 3.3 70B (HF) | huggingface | Balanced | 128,000 | 4,096 | $0.30 | $0.30 | Нет | Нет |
| 53 | `replicate/meta-llama-3.3-70b-instruct` | Llama 3.3 70B (Replicate) | replicate | Balanced | 128,000 | 4,096 | $0.40 | $0.40 | Нет | Нет |

**Уровни моделей (Tiers):**

| Уровень | Описание | Типичное использование |
|------|------------|------------|
| **Frontier** | Самые мощные, самая высокая стоимость | Оркестрация, архитектура, аудит безопасности |
| **Smart** | Сильные рассуждения, умеренная стоимость | Кодинг, код-ревью, исследования, анализ |
| **Balanced** | Хороший баланс цены и качества | Планирование, тексты, DevOps, повседневные задачи |
| **Fast** | Самый дешевый облачный инференс | Операции, перевод, простые вопросы, проверки здоровья |
| **Local** | Self-hosted, нулевая стоимость | Приватность, офлайн, разработка |

**Примечания:**
- Локальные провайдеры (Ollama, vLLM, LM Studio) автоматически обнаруживают модели во время работы. Любая скачанная и запущенная вами модель будет добавлена в каталог с уровнем `Local` и нулевой стоимостью.
- В таблице выше приведены 53 встроенные модели. Общее число в 51, упомянутое ранее, относится к фиксированному набору, а фактическое число может меняться в зависимости от локально обнаруженных моделей.

---

<a name="model-aliases"></a>
## Алиасы моделей

Все 23 алиаса (псевдонима) разрешаются в канонические ID моделей. Алиасы нечувствительны к регистру.

| Алиас | Разрешается в |
|-------|------------|
| `sonnet` | `claude-sonnet-4-20250514` |
| `claude-sonnet` | `claude-sonnet-4-20250514` |
| `haiku` | `claude-haiku-4-5-20251001` |
| `claude-haiku` | `claude-haiku-4-5-20251001` |
| `opus` | `claude-opus-4-20250514` |
| `claude-opus` | `claude-opus-4-20250514` |
| `gpt4` | `gpt-4o` |
| `gpt4o` | `gpt-4o` |
| `gpt4-mini` | `gpt-4o-mini` |
| `flash` | `gemini-2.5-flash` |
| `gemini-flash` | `gemini-2.5-flash` |
| `gemini-pro` | `gemini-2.5-pro` |
| `deepseek` | `deepseek-chat` |
| `llama` | `llama-3.3-70b-versatile` |
| `llama-70b` | `llama-3.3-70b-versatile` |
| `mixtral` | `mixtral-8x7b-32768` |
| `mistral` | `mistral-large-latest` |
| `codestral` | `codestral-latest` |
| `grok` | `grok-2` |
| `grok-mini` | `grok-2-mini` |
| `sonar` | `sonar-pro` |
| `jamba` | `jamba-1.5-large` |
| `command-r` | `command-r-plus` |

Вы можете использовать алиасы везде, где принимается ID модели: в файлах конфигурации, вызовах REST API, командах чата и конфигурации маршрутизации моделей.

---

<a name="per-agent-model-override"></a>
## Переопределение модели для конкретного агента

Каждый агент в вашем `config.toml` может указывать собственную модель, переопределяя глобальное значение по умолчанию:

```toml
# Глобальная модель по умолчанию
[agents.defaults]
model = "claude-sonnet-4-20250514"

# Переопределение для конкретного агента: используйте алиас или полный ID модели
[[agents]]
name = "orchestrator"
model = "opus"                      # алиас для claude-opus-4-20250514

[[agents]]
name = "ops"
model = "llama-3.3-70b-versatile"   # дешевая модель Groq для простых операций

[[agents]]
name = "coder"
model = "gemini-2.5-flash"          # быстро + дешево + контекст 1M

[[agents]]
name = "researcher"
model = "sonar-pro"                 # Perplexity со встроенным веб-поиском

# Вы также можете жестко закрепить модель в манифесте агента TOML
[[agents]]
name = "production-bot"
pinned_model = "claude-sonnet-4-20250514"  # никогда не перенаправляется автоматически
```

Если в манифесте агента установлено `pinned_model`, этот агент всегда будет использовать указанную модель независимо от настроек маршрутизации. Это используется в **режиме стабилизации** (`KernelMode::Stable`), где модель заморожена для надежности в продакшене.

---

<a name="model-routing"></a>
## Маршрутизация моделей

OpenFang может автоматически выбирать самую дешевую модель, способную обработать каждый запрос. Это настраивается для каждого агента через `ModelRoutingConfig`.

### Как это работает

1. **ModelRouter** оценивает каждый входящий `CompletionRequest` на основе эвристик.
2. Оценка сопоставляется с уровнем сложности задачи (**TaskComplexity**): `Simple` (простая), `Medium` (средняя) или `Complex` (сложная).
3. Для каждого уровня назначена предварительно настроенная модель.

### Эвристики оценки

| Сигнал | Вес | Логика |
|--------|--------|-------|
| Общая длина сообщения | 1 балл за каждые ~4 символа | Примерная оценка токенов |
| Наличие инструментов | +20 за каждый определенный инструмент | Инструменты подразумевают многоэтапную работу |
| Маркеры кода | +30 за каждый найденный маркер | Обратные кавычки, `fn`, `def`, `class`, `import`, `function`, `async`, `await`, `struct`, `impl`, `return` |
| Глубина диалога | +15 за каждое сообщение после 10-го | Глубокий контекст = более сложные рассуждения |
| Длина системного промпта | +1 за каждые 10 символов сверх 500 | Длинные системные промпты подразумевают сложные задачи |

### Пороги сложности

| Сложность | Диапазон баллов | Модель по умолчанию |
|-----------|-------------|---------------|
| Simple | баллы < 100 | `claude-haiku-4-5-20251001` |
| Medium | 100 <= баллы < 500 | `claude-sonnet-4-20250514` |
| Complex | баллы >= 500 | `claude-sonnet-4-20250514` |

### Конфигурация

```toml
# В манифесте агента или в config.toml
[routing]
simple_model = "claude-haiku-4-5-20251001"
medium_model = "gemini-2.5-flash"
complex_model = "claude-sonnet-4-20250514"
simple_threshold = 100
complex_threshold = 500
```

Маршрутизатор также интегрирован с каталогом моделей:
- **`validate_models()`** проверяет существование всех настроенных ID моделей в каталоге.
- **`resolve_aliases()`** разворачивает алиасы в канонические ID (например, `"sonnet"` становится `"claude-sonnet-4-20250514"`).

---

<a name="cost-tracking"></a>
## Отслеживание затрат

OpenFang отслеживает стоимость каждого вызова LLM и может применять квоты расходов для каждого агента.

### Оценка стоимости ответа

После каждого вызова LLM стоимость рассчитывается следующим образом:

```
cost = (входные_токены / 1,000,000) * входная_ставка + (выходные_токены / 1,000,000) * выходная_ставка
```

`MeteringEngine` сначала ищет точную цену в **каталоге моделей**. Если модель не найдена, используется эвристика сопоставления по шаблону.

### Ставки стоимости (за миллион токенов)

| Шаблон модели | Вход $/M | Выход $/M |
|--------------|-----------|------------|
| `*haiku*` | $0.25 | $1.25 |
| `*sonnet*` | $3.00 | $15.00 |
| `*opus*` | $15.00 | $75.00 |
| `gpt-4o-mini` | $0.15 | $0.60 |
| `gpt-4o` | $2.50 | $10.00 |
| `gpt-4.1-nano` | $0.10 | $0.40 |
| `gpt-4.1-mini` | $0.40 | $1.60 |
| `gpt-4.1` | $2.00 | $8.00 |
| `o3-mini` | $1.10 | $4.40 |
| `gemini-2.5-pro` | $1.25 | $10.00 |
| `gemini-2.5-flash` | $0.15 | $0.60 |
| `gemini-2.0-flash` | $0.10 | $0.40 |
| `deepseek-reasoner` / `deepseek-r1` | $0.55 | $2.19 |
| `*deepseek*` | $0.27 | $1.10 |
| `*cerebras*` | $0.06 | $0.06 |
| `*sambanova*` | $0.06 | $0.06 |
| `*replicate*` | $0.40 | $0.40 |
| `*llama*` / `*mixtral*` | $0.05 | $0.10 |
| `*qwen*` | $0.20 | $0.60 |
| `mistral-large*` | $2.00 | $6.00 |
| `*mistral*` (прочие) | $0.10 | $0.30 |
| `command-r-plus` | $2.50 | $10.00 |
| `command-r` | $0.15 | $0.60 |
| `sonar-pro` | $3.00 | $15.00 |
| `*sonar*` (прочие) | $1.00 | $5.00 |
| `grok-2-mini` / `grok-mini` | $0.30 | $0.50 |
| `*grok*` (прочие) | $2.00 | $10.00 |
| `*jamba*` | $2.00 | $8.00 |
| По умолчанию (неизвестно) | $1.00 | $3.00 |

### Контроль квот

Квоты проверяются при каждом вызове LLM. Если агент превышает свой почасовой лимит, вызов отклоняется с ошибкой `QuotaExceeded`.

```toml
# Квота для агента в config.toml
[[agents]]
name = "chatbot"
[agents.resources]
max_cost_per_hour_usd = 5.00   # ограничение в $5 в час
```

Нижний колонтитул использования (если включен) добавляет информацию о стоимости к каждому ответу:

```
> Cost: $0.0042 | Tokens: 1,200 in / 340 out | Model: claude-sonnet-4-20250514
```

---

<a name="fallback-providers"></a>
## Резервные провайдеры (Fallback)

`FallbackDriver` объединяет несколько драйверов LLM в цепочку. Если основной драйвер дает сбой, автоматически пробуется следующий драйвер в цепочке.

### Поведение

- При успехе: возвращает результат немедленно.
- При ошибках **ограничения скорости / перегрузки** (`429`, `529`): передает ошибку выше для логики повторных попыток (переключение на резерв НЕ происходит, так как основной драйвер должен быть опрошен повторно после паузы).
- При **всех остальных ошибках**: записывает предупреждение в лог и пробует следующий драйвер в цепочке.
- Если все драйверы дали сбой: возвращает последнюю ошибку.

### Конфигурация

Резервные цепочки настраиваются в манифесте агента или в `config.toml`. `FallbackDriver` используется автоматически, когда агент находится в **режиме стабилизации** (`KernelMode::Stable`) или когда для надежности настроено несколько провайдеров.

```toml
# Пример: основной Anthropic, резервный Gemini, затем Groq
[[agents]]
name = "production-bot"
model = "claude-sonnet-4-20250514"
fallback_models = ["gemini-2.5-flash", "llama-3.3-70b-versatile"]
```

Драйвер резервного копирования создает цепочку: `AnthropicDriver -> GeminiDriver -> OpenAIDriver(Groq)`.

---

<a name="api-endpoints"></a>
## Эндпоинты API

### Список всех моделей

```
GET /api/models
```

Возвращает полный каталог моделей с метаданными, ценами и флагами функций.

**Ответ:**
```json
[
  {
    "id": "claude-sonnet-4-20250514",
    "display_name": "Claude Sonnet 4",
    "provider": "anthropic",
    "tier": "Smart",
    "context_window": 200000,
    "max_output_tokens": 64000,
    "input_cost_per_m": 3.0,
    "output_cost_per_m": 15.0,
    "supports_tools": true,
    "supports_vision": true,
    "supports_streaming": true,
    "aliases": ["sonnet", "claude-sonnet"]
  }
]
```

### Получение конкретной модели

```
GET /api/models/{id}
```

Возвращает данные по одной модели. Поддерживает как канонические ID, так и алиасы.

```
GET /api/models/sonnet
GET /api/models/claude-sonnet-4-20250514
```

### Список алиасов

```
GET /api/models/aliases
```

Возвращает карту соответствия алиасов каноническим ID.

**Ответ:**
```json
{
  "sonnet": "claude-sonnet-4-20250514",
  "haiku": "claude-haiku-4-5-20251001",
  "flash": "gemini-2.5-flash",
  "grok": "grok-2"
}
```

### Список провайдеров

```
GET /api/providers
```

Возвращает список всех 20 провайдеров со статусом аутентификации и количеством моделей.

**Ответ:**
```json
[
  {
    "id": "anthropic",
    "display_name": "Anthropic",
    "api_key_env": "ANTHROPIC_API_KEY",
    "base_url": "https://api.anthropic.com",
    "key_required": true,
    "auth_status": "Configured",
    "model_count": 3
  },
  {
    "id": "ollama",
    "display_name": "Ollama",
    "api_key_env": "OLLAMA_API_KEY",
    "base_url": "http://localhost:11434/v1",
    "key_required": false,
    "auth_status": "NotRequired",
    "model_count": 5
  }
]
```

Значения статуса аутентификации: `Configured` (Настроено), `Missing` (Отсутствует), `NotRequired` (Не требуется).

### Установка API-ключа провайдера

```
POST /api/providers/{name}/key
Content-Type: application/json

{ "api_key": "sk-..." }
```

Настраивает API-ключ для провайдера во время работы (сохраняется как `Zeroizing<String>`, стирается из памяти при удалении объекта).

### Удаление API-ключа провайдера

```
DELETE /api/providers/{name}/key
```

Удаляет настроенный API-ключ провайдера.

### Тестирование соединения с провайдером

```
POST /api/providers/{name}/test
```

Отправляет минимальный тестовый запрос для проверки доступности провайдера и валидности API-ключа.

---

<a name="channel-commands"></a>
## Команды каналов

В любом канале доступны две команды чата для просмотра моделей и провайдеров:

### `/models`

Выводит список всех доступных моделей с их уровнем, провайдером и окном контекста. Показывает только модели тех провайдеров, для которых настроена аутентификация (или она не требуется).

```
/models
```

Пример вывода:
```
Доступные модели (12):

Frontier:
  claude-opus-4-20250514 (Anthropic) — 200K ctx
  gemini-2.5-pro (Google Gemini) — 1M ctx

Smart:
  claude-sonnet-4-20250514 (Anthropic) — 200K ctx
  gemini-2.5-flash (Google Gemini) — 1M ctx
  deepseek-chat (DeepSeek) — 64K ctx

Balanced:
  llama-3.3-70b-versatile (Groq) — 128K ctx

Fast:
  claude-haiku-4-5-20251001 (Anthropic) — 200K ctx
  gemini-2.0-flash (Google Gemini) — 1M ctx

Local:
  llama3.2 (Ollama) — 128K ctx
```

### `/providers`

Выводит список всех 20 провайдеров со статусом их аутентификации.

```
/providers
```

Пример вывода:
```
Провайдеры LLM (20):

  Anthropic          ANTHROPIC_API_KEY       Configured    3 модели
  OpenAI             OPENAI_API_KEY          Missing       6 моделей
  Google Gemini      GEMINI_API_KEY          Configured    3 модели
  DeepSeek           DEEPSEEK_API_KEY        Missing       2 модели
  Groq               GROQ_API_KEY            Configured    4 модели
  Ollama             (ключ не нужен)         Ready         3 модели
  vLLM               (ключ не нужен)         Ready         1 модель
  LM Studio          (ключ не нужен)         Ready         1 модель
  ...
```

---

## Сводка переменных окружения

Краткий справочник по переменным окружения всех провайдеров:

| Провайдер | Переменная окружения | Требуется |
|----------|---------|----------|
| Anthropic | `ANTHROPIC_API_KEY` | Да |
| OpenAI | `OPENAI_API_KEY` | Да |
| Google Gemini | `GEMINI_API_KEY` или `GOOGLE_API_KEY` | Да |
| DeepSeek | `DEEPSEEK_API_KEY` | Да |
| Groq | `GROQ_API_KEY` | Да |
| OpenRouter | `OPENROUTER_API_KEY` | Да |
| Mistral AI | `MISTRAL_API_KEY` | Да |
| Together AI | `TOGETHER_API_KEY` | Да |
| Fireworks AI | `FIREWORKS_API_KEY` | Да |
| Ollama | `OLLAMA_API_KEY` | Нет |
| vLLM | `VLLM_API_KEY` | Нет |
| LM Studio | `LMSTUDIO_API_KEY` | Нет |
| Perplexity AI | `PERPLEXITY_API_KEY` | Да |
| Cohere | `COHERE_API_KEY` | Да |
| AI21 Labs | `AI21_API_KEY` | Да |
| Cerebras | `CEREBRAS_API_KEY` | Да |
| SambaNova | `SAMBANOVA_API_KEY` | Да |
| Hugging Face | `HF_API_KEY` | Да |
| xAI | `XAI_API_KEY` | Да |
| Replicate | `REPLICATE_API_TOKEN` | Да |

---

## Заметки по безопасности

- Все API-ключи хранятся как `Zeroizing<String>` — данные ключа автоматически перезаписываются нулями при удалении объекта из памяти.
- Определение аутентификации (`detect_auth()`) только проверяет наличие переменной через `std::env::var()` — оно никогда не считывает и не записывает фактическое значение секрета в логи.
- API-ключи провайдеров, установленные через REST API (`POST /api/providers/{name}/key`), следуют той же политике обнуления.
- Эндпоинт состояния (`/api/health`) никогда не раскрывает статус аутентификации провайдера или API-ключи. Подробная информация доступна по адресу `/api/health/detail`, который требует аутентификации.
- Все структуры `DriverConfig` и `KernelConfig` реализуют `Debug` с сокрытием секретов — API-ключи выводятся в логах как `"***"`.
