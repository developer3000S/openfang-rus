# Чек-лист для выпуска в продакшен

Всё, что необходимо сделать перед установкой тега `v0.1.0` и отправкой продукта пользователям. Пункты упорядочены по зависимостям — выполняйте их сверху вниз.

---

## 1. Генерация пары ключей для подписи Tauri

**Статус:** БЛОКИРУЮЩИЙ — без этого автообновление не будет работать. Ни один пользователь никогда не получит обновление.

Обновлятору Tauri требуется пара ключей Ed25519. Приватный ключ подписывает каждый пакет релиза, а публичный ключ встраивается в бинарный файл приложения, чтобы оно могло проверять обновления.

```bash
# Установите Tauri CLI (если еще не установлен)
cargo install tauri-cli --locked

# Сгенерируйте пару ключей
cargo tauri signer generate -w ~/.tauri/openfang.key
```

Команда выведет:

```
Your public key was generated successfully:
dW50cnVzdGVkIGNvb...  <-- СКОПИРУЙТЕ ЭТО

Your private key was saved to: ~/.tauri/openfang.key
```

Сохраните оба значения. Они понадобятся для шагов 2 и 3.

---

## 2. Установка публичного ключа в `tauri.conf.json`

**Статус:** БЛОКИРУЮЩИЙ — плейсхолдер должен быть заменен перед сборкой.

Откройте `crates/openfang-desktop/tauri.conf.json` и замените:

```json
"pubkey": "PLACEHOLDER_REPLACE_WITH_GENERATED_PUBKEY"
```

на реальную строку публичного ключа из шага 1:

```json
"pubkey": "dW50cnVzdGVkIGNvb..."
```

---

## 3. Добавление секретов в репозиторий GitHub

**Статус:** БЛОКИРУЮЩИЙ — рабочий процесс релиза CI/CD завершится ошибкой без этих данных.

Перейдите в **GitHub repo → Settings → Secrets and variables → Actions → New repository secret** и добавьте:

| Имя секрета | Значение | Обязательно |
|---|---|---|
| `TAURI_SIGNING_PRIVATE_KEY` | Содержимое файла `~/.tauri/openfang.key` | Да |
| `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` | Пароль, установленный при генерации (или пустая строка) | Да |

### Опционально — подпись кода для macOS

Без этого пользователи macOS увидят предупреждение о "приложении от неопознанного разработчика". Требуется аккаунт Apple Developer ($99/год).

| Имя секрета | Значение |
|---|---|
| `APPLE_CERTIFICATE` | Файл сертификата `.p12` в кодировке Base64 |
| `APPLE_CERTIFICATE_PASSWORD` | Пароль для файла .p12 |
| `APPLE_SIGNING_IDENTITY` | напр. `Developer ID Application: Your Name (TEAMID)` |
| `APPLE_ID` | Ваш email Apple ID |
| `APPLE_PASSWORD` | Пароль приложения с appleid.apple.com |
| `APPLE_TEAM_ID` | Ваш 10-символьный Team ID |

Для генерации base64 сертификата:
```bash
base64 -i Certificates.p12 | pbcopy
```

---

## 4. Создание иконок

**Статус:** ПРОВЕРКА — убедитесь, что иконки не являются заглушками.

Следующие файлы иконок должны находиться в `crates/openfang-desktop/icons/`:

| Файл | Размер | Использование |
|---|---|---|
| `icon.png` | 1024x1024 | Исходная иконка, генерация .icns для macOS |
| `icon.ico` | разн. разм. | Панель задач Windows, установщик |
| `32x32.png` | 32x32 | Системный трей, малые контексты |
| `128x128.png` | 128x128 | Списки приложений |
| `128x128@2x.png` | 256x256 | Дисплеи HiDPI/Retina |

Убедитесь, что это реальные брендированные иконки. Генерация из одного исходного SVG:

```bash
# Использование ImageMagick
convert icon.svg -resize 1024x1024 icon.png
convert icon.svg -resize 32x32 32x32.png
convert icon.svg -resize 128x128 128x128.png
convert icon.svg -resize 256x256 128x128@2x.png
convert icon.svg -resize 256x256 -define icon:auto-resize=256,128,64,48,32,16 icon.ico
```

---

## 5. Настройка домена `openfang.sh`

**Статус:** БЛОКИРУЮЩИЙ для скриптов установки — пользователи запускают `curl -sSf https://openfang.sh | sh`.

Варианты:
- **GitHub Pages**: Направьте `openfang.sh` на сайт GitHub Pages, который перенаправляет `/` на `scripts/install.sh`, а `/install.ps1` на `scripts/install.ps1`.
- **Cloudflare Workers / Vercel**: Отдавайте скрипты установки с правильными заголовками `Content-Type: text/plain`.

Пока домен не настроен, пользователи могут устанавливать через:
```bash
curl -sSf https://raw.githubusercontent.com/RightNow-AI/openfang/main/scripts/install.sh | sh
```

---

## 6. Проверка сборки Dockerfile

**Статус:** ПРОВЕРКА — Dockerfile должен выдавать рабочий образ.

```bash
docker build -t openfang:local .
docker run --rm openfang:local --version
docker run --rm -p 4200:4200 -v openfang-data:/data openfang:local start
```

Подтвердите:
- Бинарный файл запускается и выводит версию.
- Команда `start` запускает ядро и API-сервер.
- Порт 4200 доступен.
- Том `/data` сохраняется между перезапусками контейнера.

---

## 7. Локальная проверка скриптов установки

**Статус:** ПРОВЕРКА перед релизом.

### Linux/macOS
```bash
# Тест против реального релиза GitHub (после первого тега)
bash scripts/install.sh

# Или только проверка синтаксиса
bash -n scripts/install.sh
shellcheck scripts/install.sh
```

### Windows (PowerShell)
```powershell
# Тест против реального релиза GitHub (после первого тега)
powershell -ExecutionPolicy Bypass -File scripts/install.ps1
```

---

## 8. Написание CHANGELOG.md для v0.1.0

**Статус:** ПРОВЕРКА — подтвердите, что он охватывает все выпущенные функции.

Убедитесь, что файл существует в корне репозитория и содержит:
- Описание всех 14 крейтов.
- Ключевые возможности: 40 каналов, 60 навыков, 20 провайдеров, 51 модель.
- Системы безопасности.
- Десктопное приложение с автообновлением.
- Путь миграции с OpenClaw.
- Опции установки через Docker и CLI.

---

## 9. Первый релиз — тег и пуш

Как только шаги 1-8 будут выполнены:

```bash
# Убедитесь, что версия совпадает везде
grep '"version"' crates/openfang-desktop/tauri.conf.json
grep '^version' Cargo.toml

# Закоммитьте финальные изменения
git add -A
git commit -m "chore: prepare v0.1.0 release"

# Создайте тег и запушьте
git tag v0.1.0
git push origin main --tags
```

Это запустит рабочий процесс релиза, который:
1. Соберет установщики для 4 платформ.
2. Сгенерирует подписанный `latest.json` для автообновления.
3. Соберет бинарные файлы CLI для 5 целей.
4. Соберет и запушит мультиархитектурный Docker-образ.
5. Создаст GitHub Release со всеми артефактами.

---

## 10. Пострелизная проверка

После завершения рабочего процесса релиза (~15-30 мин):

### Страница релиза GitHub
- [ ] Присутствуют `.msi` и `.exe` (Windows).
- [ ] Присутствует `.dmg` (macOS).
- [ ] Присутствуют `.AppImage` и `.deb` (Linux).
- [ ] Присутствует `latest.json`.
- [ ] Присутствуют архивы CLI `.tar.gz` (5 целей).
- [ ] Присутствуют контрольные суммы SHA256.

### Скрипты установки
```bash
# Linux/macOS
curl -sSf https://openfang.sh | sh
openfang --version  # Должно вывести v0.1.0

# Windows PowerShell
irm https://openfang.sh/install.ps1 | iex
openfang --version
```

---

## Быстрая справка — что что блокирует

```
Шаг 1 (генерация ключей) ──┬──> Шаг 2 (публичный ключ в конфиге)
                          └──> Шаг 3 (секреты в GitHub)
                                 │
Шаг 4 (иконки) ──────────┤
Шаг 5 (домен) ───────────┤
Шаг 6 (Dockerfile) ──────┤
Шаг 7 (скрипты уст.) ────┤
Шаг 8 (CHANGELOG) ───────┘
                         │
                         v
                  Шаг 9 (тег + пуш)
                         │
                         v
                  Шаг 10 (проверка)
```

Шаги 4-8 могут выполняться параллельно. Шаги 1-3 последовательны и должны быть выполнены первыми.
