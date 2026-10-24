# OpenHands — автоматизация разработки ПО с помощью AI-агентов

OpenHands — open-source платформа AI-агентов для автоматизации инженерных задач. Агент обновляет зависимости, генерирует тесты, разрешает конфликты слияния и помогает исправлять ошибки в коде.

## Содержание

- [Требования](#требования)
- [Установка](#установка)
  - [Установка через Docker](#установка-через-docker)
  - [Установка через uv](#установка-через-uv)
  - [Установка через pip](#установка-через-pip)
  - [Офлайн-установка](#офлайн-установка)
- [Первый запуск](#первый-запуск)
- [Настройка](#настройка)
  - [Настройка LLM](#настройка-llm)
  - [Переменные окружения](#переменные-окружения)
  - [Проброс каталогов](#проброс-каталогов)
  - [Свой sandbox-образ](#свой-sandbox-образ)
- [Использование](#использование)
  - [Web-интерфейс](#web-интерфейс)
  - [Интерактивный CLI](#интерактивный-cli)
  - [Headless-режим](#headless-режим)
  - [Примеры задач](#примеры-задач)
  - [MCP-серверы](#mcp-серверы)
  - [REST API](#rest-api)
- [Флаги CLI](#флаги-cli)
- [Решение типовых проблем](#решение-типовых-проблем)

## Требования

| Компонент | Минимум | Рекомендуется |
|---|---|---|
| ОС | Linux, macOS, Windows + WSL2 | Linux или macOS |
| Docker | 26.0.0 | Docker Desktop 4.31.0 или новее |
| RAM | 4 ГБ | 8 ГБ и больше |
| CPU | 2 ядра | 4 ядра и больше |
| Диск | 5 ГБ | 20 ГБ и больше |
| Python | 3.12 для установки через uv/pip | 3.12 |
| LLM | API-ключ провайдера | Anthropic, OpenAI, Gemini, DeepSeek или локальный endpoint |

GPU не требуется. GPU нужен только для локальных моделей.

## Установка

В примерах используется тег `0.40`. Замените его на актуальную версию OpenHands.

### Установка через Docker

Docker-вариант ближе всего к production-запуску.

```bash
docker pull docker.all-hands.dev/all-hands-ai/runtime:0.40-nikolaik

docker run -it --rm --pull=always \
  -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.40-nikolaik \
  -e LOG_ALL_EVENTS=true \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/.openhands-state:/.openhands-state \
  -p 3000:3000 \
  --add-host host.docker.internal:host-gateway \
  --name openhands-app \
  docker.all-hands.dev/all-hands-ai/openhands:0.40
```

После запуска откройте `http://localhost:3000`.

Если порт `3000` занят, измените проброс порта:

```bash
-p 3001:3000
```

### Установка через uv

`uv` — рекомендуемый способ для локальной разработки.

```bash
# Установка uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Установка OpenHands
uv tool install openhands --python 3.12

# Запуск
openhands serve
```

Если команда `openhands` не найдена, добавьте каталог `~/.local/bin` в `PATH`.

### Установка через pip

```bash
pip install openhands
openhands serve
```

> ⚠️ Для MCP-серверов по умолчанию нужен `uv`. При установке через `pip` установите `uv` отдельно.

### Офлайн-установка

На машине с доступом к интернету скачайте sandbox-образ:

```bash
docker pull docker.all-hands.dev/all-hands-ai/runtime:0.40-nikolaik
docker save docker.all-hands.dev/all-hands-ai/runtime:0.40-nikolaik -o openhands-runtime.tar
```

Перенесите файл `openhands-runtime.tar` на изолированную машину и загрузите образ:

```bash
docker load -i openhands-runtime.tar
```

Для работы без внешнего LLM используйте локальный endpoint через `LLM_BASE_URL`, например Ollama или vLLM.

## Первый запуск

1. Откройте `http://localhost:3000` в браузере.
2. Выберите провайдера LLM.
3. Выберите модель.
4. Введите API-ключ в поле `API Key`.
5. Нажмите `Start Conversation`.

Проверка CLI:

```bash
openhands --help
```

## Настройка

### Настройка LLM

LLM можно настроить через Web-интерфейс или переменные окружения.

Пример для OpenAI-совместимого endpoint:

```bash
export LLM_MODEL="openai/gpt-4o-mini"
export LLM_API_KEY="YOUR_API_KEY"
export LLM_BASE_URL="http://localhost:11434/v1"

openhands serve
```

Пример для Anthropic:

```bash
export LLM_MODEL="anthropic/claude-sonnet-4-20250514"
export LLM_API_KEY="YOUR_API_KEY"

openhands serve
```

### Переменные окружения

| Переменная | Тип | По умолчанию | Описание |
|---|---|---|---|
| `LLM_API_KEY` | строка | — | API-ключ провайдера LLM |
| `LLM_MODEL` | строка | `claude-sonnet-4` | Имя модели в формате litellm |
| `LLM_BASE_URL` | строка | — | Свой endpoint для OpenAI-совместимого API |
| `OH_PERSISTENCE_DIR` | строка | `~/.openhands` | Каталог для хранения состояния |
| `SANDBOX_VOLUMES` | строка | — | Каталоги хоста для монтирования, через запятую |
| `RUNTIME` | строка | `docker` | Тип runtime: `docker`, `process`, `remote` |
| `AGENT_SERVER_IMAGE_REPOSITORY` | строка | образ по умолчанию | Репозиторий своего sandbox-образа |
| `AGENT_SERVER_IMAGE_TAG` | строка | `0.40` | Тег своего sandbox-образа |

### Проброс каталогов

Чтобы агент видел файлы проекта, смонтируйте каталог в sandbox.

Через Docker:

```bash
-e SANDBOX_VOLUMES="$(pwd):/workspace"
```

Через CLI:

```bash
openhands --mount-cwd
```

### Свой sandbox-образ

```bash
docker run -it --rm --pull=always \
  -e SANDBOX_RUNTIME_CONTAINER_IMAGE=my-registry/custom-runtime:latest \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 3000:3000 \
  --name openhands-app \
  docker.all-hands.dev/all-hands-ai/openhands:0.40
```

## Использование

### Web-интерфейс

Запустите сервер:

```bash
openhands serve
```

Откройте `http://localhost:3000`, создайте диалог и опишите задачу.

### Интерактивный CLI

```bash
openhands
```

В интерактивном режиме введите задачу и следите за действиями агента в терминале.

### Headless-режим

Headless-режим подходит для CI/CD и скриптов.

```bash
openhands --headless -t "Сгенерируй unit-тесты для src/utils.py"
```

Вывод в JSON:

```bash
openhands --headless --json -t "Создай минимальное Flask-приложение"
```

### Примеры задач

Обновление зависимостей:

```bash
openhands run "Обнови requests до последней версии и исправь тесты"
```

Генерация тестов:

```bash
openhands run "Сгенерируй pytest-тесты для src/utils.py с покрытием не ниже 90%"
```

Разрешение конфликта слияния:

```bash
openhands run "Разреши конфликт слияния в src/main.py, сохрани изменения из обеих веток"
```

Через Python SDK:

```python
from openhands import Agent, LLM

agent = Agent(llm=LLM(model="anthropic/claude-sonnet-4-20250514"))
result = agent.run("Обнови зависимости и исправь падающие тесты")
print(result.output)
```

### MCP-серверы

Подключение MCP-сервера `fetch`:

```python
from openhands import Agent

agent = Agent(llm=LLM(model="openai/gpt-5"))
agent.register_mcp_server("fetch", command="uvx mcp-server-fetch")
```

### REST API

| Метод | Endpoint | Описание |
|---|---|---|
| `POST` | `/api/v1/conversations` | Создать диалог |
| `GET` | `/api/v1/conversations/{id}` | Получить состояние диалога |
| `POST` | `/api/v1/conversations/{id}/messages` | Отправить сообщение |
| `GET` | `/api/v1/conversations/{id}/events` | Поток событий SSE |
| `DELETE` | `/api/v1/conversations/{id}` | Завершить диалог |
| `GET` | `/api/v1/models` | Список доступных моделей |
| `GET` | `/health` | Проверка состояния |

Пример запроса:

```bash
curl -X POST http://localhost:3000/api/v1/conversations \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Исправь падающий тест в test_utils.py"}'
```

Пример ответа:

```json
{
  "id": "conv_abc123",
  "status": "running",
  "created_at": "2026-10-24T10:00:00Z"
}
```

## Флаги CLI

| Флаг | Тип | По умолчанию | Описание |
|---|---|---|---|
| `--model` | строка | `claude-sonnet-4` | Идентификатор модели LLM |
| `--api-key` | строка | `$LLM_API_KEY` | API-ключ или переменная окружения |
| `--base-url` | строка | — | OpenAI-совместимый endpoint |
| `--max-iterations` | число | `100` | Максимум шагов агента |
| `--sandbox` | строка | `docker` | Тип sandbox: `docker`, `k8s`, `remote` |
| `--gpu` | флаг | `false` | Включить GPU через nvidia-docker |
| `--mount-cwd` | флаг | `false` | Смонтировать текущий каталог в sandbox |
| `--headless` | флаг | `false` | Режим без интерактивного ввода |
| `--json` | флаг | `false` | Вывод в JSON для headless-режима |

## Решение типовых проблем

| Ошибка | Решение |
|---|---|
| `Cannot connect to Docker daemon` | Запустите Docker Desktop или выполните `sudo systemctl start docker`. |
| `Port 3000 already in use` | Измените проброс порта: `-p 3001:3000`. |
| `LLM API key invalid` | Проверьте `LLM_API_KEY` и `LLM_MODEL` в настройках. |
| `docker: command not found` в Windows | Запустите команду внутри терминала WSL2. |
| `Sandbox image pull failed` | Проверьте доступ к `docker.all-hands.dev` и правильность тега образа. |
| `openhands: command not found` после uv | Добавьте `~/.local/bin` в `PATH` и перезапустите терминал. |
| Агент не видит файлы проекта | Смонтируйте каталог через `SANDBOX_VOLUMES` или флаг `--mount-cwd`. |
```
