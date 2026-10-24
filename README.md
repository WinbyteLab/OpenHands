# OpenHands — Code Less, Make More

[![Build Status](https://github.com/OpenHands/OpenHands/actions/workflows/build.yml/badge.svg)](https://github.com/OpenHands/OpenHands/actions)
[![Coverage](https://img.shields.io/badge/coverage-≥85%25-brightgreen)](https://github.com/OpenHands/OpenHands)
[![Version](https://img.shields.io/badge/version-1.25.0-blue)](https://github.com/OpenHands/OpenHands/releases)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Downloads](https://img.shields.io/pypi/dm/openhands)](https://pypi.org/project/openhands/)
[![Docker Pulls](https://img.shields.io/docker/pulls/all-hands-ai/openhands)](https://hub.docker.com/r/all-hands-ai/openhands)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/OpenHands/OpenHands/badge)](https://securityscorecards.dev/viewer/?uri=github.com/OpenHands/OpenHands)
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/OpenHands/badge)](https://bestpractices.coreinfrastructure.org/projects/OpenHands)
[![SWE-Bench](https://img.shields.io/badge/SWE--Bench-77.6%25-important)](https://www.swebench.com/)

**OpenHands** — open-source платформа AI-агентов для автоматизации разработки ПО. Проект решает реальные инженерные задачи: dependency upgrades, генерацию unit-тестов, разрешение merge conflicts, сканирование уязвимостей. Аудитория: developers, researchers, enterprises.

<!-- toc -->
- [Why / Problem Statement](#why--problem-statement)
- [Features](#features)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Usage](#usage)
- [API Reference](#api-reference)
- [AI/ML specifics](#aiml-specifics)
- [Performance & Benchmarks](#performance--benchmarks)
- [Security](#security)
- [Testing & Quality](#testing--quality)
- [Observability](#observability)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Community & Support](#community--support)
- [License & Citation](#license--citation)
- [Acknowledgments & Star History](#acknowledgments--star-history)
- [Footer](#footer)
<!-- tocstop -->

## Why / Problem Statement

Разработка ПО требует рутинных операций, которые отнимают время у инженеров.

- **Dependency upgrades** — ручное обновление пакетов и разрешение конфликтов версий.
- **Генерация тестов** — написание boilerplate-тестов для legacy-кода.
- **Merge conflicts** — ручное слияние веток в крупных репозиториях.
- **Сканирование уязвимостей** — поиск и исправление security issues.

### Сравнение с альтернативами

| Критерий | OpenHands | Devin | SWE-Agent | AutoCodeRover |
|---|---|---|---|---|
| Open-source | ✅ MIT | ❌ Proprietary | ✅ MIT | ✅ MIT |
| SWE-Bench Verified | **77.6%** | ~13.8% (2024) | ~53% | ~45% |
| Sandboxing | Docker / K8s / Remote | Proprietary | Local only | Limited |
| Multi-agent | ✅ 6 types | ❌ | ❌ | ❌ |
| Self-hosted | ✅ | ❌ | ✅ | ✅ |

### Use-cases

1. **Dependency upgrade** — агент обновляет `requirements.txt`, запускает тесты, исправляет breaking changes.
2. **Unit test generation** — агент анализирует модуль и генерирует pytest-тесты с покрытием ≥80%.
3. **Merge conflict resolution** — агент читает diff, разрешает конфликты и создаёт commit.

## Features

| Категория | Возможность | Статус |
|---|---|---|
| **Core** | CodeAct paradigm (агент пишет код в terminal) | ✅ |
| **Core** | Docker / Kubernetes sandboxing | ✅ |
| **Core** | Web UI + CLI | ✅ |
| **Core** | Multi-LLM support (OpenAI, Anthropic, Gemini, Bedrock, DeepSeek) | ✅ |
| **Advanced** | 6 agent types (CodeAct, Browsing, VisualBrowsing, Loc, Readonly, Dummy) | ✅ |
| **Advanced** | 8+ condensation strategies для context window | ✅ |
| **Advanced** | MCP integration (dynamic tool registration) | ✅ |
| **AI-Powered** | Prompt injection defense (SecurityAnalyzer) | ✅ |
| **AI-Powered** | Agent delegation (nested agents) | ✅ |
| **AI-Powered** | StuckDetector (loop detection) | ✅ |

### Ограничения и non-goals

- ❌ Не заменяет code review человеком — только автоматизирует рутину.
- ❌ Не гарантирует 100% корректность патчей — требуется валидация.
- ❌ V0 архитектура удалена в 2026-04-01 — используйте V1 SDK.

## Architecture

```mermaid
flowchart TB
    subgraph UI["User Interface"]
        Web["Web UI (React)"]
        CLI["CLI"]
    end

    subgraph App["Application Layer"]
        AppServer["App Server (FastAPI)"]
        Router["Conversation Router"]
    end

    subgraph SDK["OpenHands SDK (V1)"]
        Agent["Agent (CodeAct)"]
        LLM["LLM (litellm)"]
        Tools["Tools: bash, editor, browser"]
        Memory["Memory & Condenser"]
    end

    subgraph Runtime["Runtime Layer"]
        Docker["Docker Runtime"]
        K8s["Kubernetes Runtime"]
        Remote["Remote Runtime (API)"]
    end

    subgraph Sandbox["Sandbox (isolated)"]
        AgentServer["agent-server (FastAPI)"]
        FS["Filesystem"]
        Shell["Shell"]
    end

    Web --> AppServer
    CLI --> AppServer
    AppServer --> Router
    Router --> Agent
    Agent --> LLM
    Agent --> Tools
    Agent --> Memory
    Router --> Runtime
    Runtime --> Sandbox
    AgentServer --> FS
    AgentServer --> Shell
