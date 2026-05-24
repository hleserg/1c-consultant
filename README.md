# 1C ERP Consultant Agent

> A local MCP server that turns Claude into a competent consultant for your **1C:Enterprise ERP** configuration — one that researches real documentation and answers from sources instead of hallucinating.

**Status:** early development · **License:** MIT · **Language:** Python ≥ 3.11

*Read this in [Русский](#-агент-консультант-по-1с-erp) below.*

---

## What is this?

Working in 1C ERP constantly raises "how do I do X here", "why won't this document post", "where do I click to get this report". The usual ways to find answers all fall short:

- **ITS support line** — slow and often generic, answering without understanding your situation.
- **ITS knowledge base search** — clunky; drowns relevant results in noise, doesn't understand plain-language questions.
- **A generic AI chat** — hallucinates: confidently describes buttons and paths that don't exist, and doesn't know *your* configuration or its customizations.

The knowledge already exists — in the ITS documentation, the platform technical docs, and the configuration itself (whose structure can be exported). The problem is reaching it quickly and honestly.

This project gives Claude **tool "hands"** to reach real 1C sources, plus **strict behavioral rules** that force it to use those hands and never make things up.

> **The core idea:** not a "better search", but a **competent specialist** — it gathers the context of your problem, digs through the sources itself, answers to the point, and does not invent.

## How it works

The system is a local **MCP server** (Model Context Protocol) that connects to the Claude desktop app and exposes tools: documentation search, configuration lookup, image display, onboarding, and more.

```
USER  ──►  Claude (desktop)  ──►  MCP server (tools + behavioral contract)
                                        │
                          ┌─────────────┼──────────────┐
                     Search layer   Model manager   Live procedures
                                        │
                          ┌─────────────┴──────────────┐
                  Config parser (XML)        Docs indexer + OCR
                                        │
                  ┌─────────────────────┼─────────────────────┐
            1C config dump (XML)     Offline ITS         Platform docs
```

All "diligence" lives in the **server**, not the model: it returns whole sections, expands nested topics, and emits a "completeness signal" (there are still *N* sections to read) — so the model can't stop at the first half-read snippet.

## Data sources (all legal)

| Source | What it provides | How it's obtained |
|---|---|---|
| **Configuration export (XML)** | Full structure: objects, attributes, relations, register movements, embedded help | Designer → "Dump configuration to files" |
| **Offline 1C:ITS** | Applied documentation for the configuration | its.1c.ru/itsoffline, with a valid subscription |
| **Platform tech docs** | Documentation for the 1C:Enterprise platform itself | Free official portal |

No scraping of the closed ITS — only what the user has legitimate access to. XML format specifications are referenced from the open [cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills) project (MIT).

## Tech stack

- **Python ≥ 3.11**, `uv` for dependency management
- **FastMCP** — the MCP server framework
- **Claude** — the conversational model (also provides native vision for user screenshots, and serves as quality judge)
- **BGE-M3** via FlagEmbedding — local hybrid (dense + sparse) embeddings
- **Cohere Rerank v4** — reranking (with a local BGE-reranker fallback when no key is present)
- **OCR** for screenshots in the documentation
- **Sentry** — observability (logging, tracing, metrics, AI monitoring)

Some decisions are deliberately deferred and will be made from real **trial data** (final search strategy, OCR vs. multimodal embedding for images, vector store choice) — see the project's ADR.

## Roadmap (high level)

The work is split into ~21 tasks across phases, tracked in Linear:

- **Phase 0** — bootstrap, approvals, data collection (repo/CI, stack ADR, project skeleton)
- **Phase 1** — parsers and model (configuration parser, docs indexer + OCR, model manager)
- **Phase 2** — search core and MCP (search layer, MCP server, onboarding, behavioral contract)
- **Phase 3** — wiring (CLI/indexing orchestration, Cohere adapters, packaging, live procedures)
- **Phase 4** — evaluation (experiment mode + telemetry, LLM judge, analysis, monthly doc updates)
- **Phase 5** — optional (release monitoring)

See [`TASKS.md`](./TASKS.md) for the full task map.

## Project goal, honestly stated

"Fully hallucination-free" is a *direction*, not a switch — no AI system can guarantee 100%. The goal is robust behavior: **"almost always goes to the sources and answers to the point."**

## License

[MIT](./LICENSE) © Sergey Khlebnikov

---
---

# 🇷🇺 Агент-консультант по 1С ERP

> Локальный MCP-сервер, который превращает Claude в компетентного консультанта по вашей конфигурации **1С:Предприятие ERP** — он исследует настоящую документацию и отвечает из источников, а не фантазирует.

**Статус:** ранняя разработка · **Лицензия:** MIT · **Язык:** Python ≥ 3.11

## Что это?

Работа в 1С ERP постоянно упирается в вопросы «как тут сделать вот это», «почему документ не проводится», «куда нажать, чтобы получился нужный отчёт». Привычные способы найти ответ подводят:

- **Линия консультаций ИТС** — медленно и часто формально, отвечают не вникая в ситуацию.
- **Поиск по базе знаний ИТС** — работает коряво, топит нужное в нерелевантном, не понимает вопрос «своими словами».
- **Обычный ИИ-чат** — фантазирует: уверенно описывает несуществующие кнопки и пути, не знает *вашу* конфигурацию и её доработки.

Знание уже существует — в документации ИТС, в техдокументации платформы и в самой конфигурации (её структуру можно выгрузить). Проблема в том, чтобы добраться до него быстро и честно.

Проект даёт Claude **инструменты-«руки»** для доступа к реальным источникам 1С и **строгие правила поведения**, которые заставляют этими руками пользоваться и не выдумывать.

> **Главная идея:** не «улучшенный поиск», а **компетентный специалист** — собирает контекст проблемы, сам копает в источниках, отвечает по делу и не выдумывает.

## Как это работает

Система — локальный **MCP-сервер** (Model Context Protocol), который подключается к настольному приложению Claude и предоставляет инструменты: поиск по документации, просмотр структуры конфигурации, показ картинок, онбординг и другие.

```
ПОЛЬЗОВАТЕЛЬ ──► Claude (десктоп) ──► MCP-сервер (инструменты + контракт)
                                            │
                          ┌─────────────────┼──────────────────┐
                    Поисковый слой    Менеджер модели    Живые процедуры
                                            │
                          ┌─────────────────┴──────────────────┐
                 Парсер конфигурации (XML)        Индексатор доки + OCR
                                            │
                  ┌─────────────────────────┼─────────────────────────┐
            Выгрузка 1С (XML)           Офлайн-ИТС            Доки платформы
```

Вся «дотошность» вынесена в **сервер**, а не в модель: он возвращает целые разделы, разворачивает вложенные темы и отдаёт «сигнал полноты» (осталось ещё *N* разделов) — чтобы модель не останавливалась на первом, прочитанном наполовину фрагменте.

## Источники данных (все легальные)

| Источник | Что даёт | Как получаем |
|---|---|---|
| **Выгрузка конфигурации (XML)** | Полное устройство: объекты, реквизиты, связи, движения регистров, встроенная справка | Конфигуратор → «Выгрузить конфигурацию в файлы» |
| **Офлайн-версия 1С:ИТС** | Прикладная документация по конфигурации | its.1c.ru/itsoffline, по действующей подписке |
| **Техдокументация платформы** | Документация по самой платформе 1С:Предприятие | Бесплатный официальный портал |

Никакого скрейпинга закрытого ИТС — только то, к чему у пользователя есть законный доступ. Спецификации XML-форматов берутся как референс из открытого проекта [cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills) (MIT).

## Стек

- **Python ≥ 3.11**, `uv` для управления зависимостями
- **FastMCP** — фреймворк MCP-сервера
- **Claude** — разговорная модель (нативно видит скриншоты пользователя, а также служит судьёй качества)
- **BGE-M3** через FlagEmbedding — локальные гибридные (dense + sparse) эмбеддинги
- **Cohere Rerank v4** — переранжирование (с фолбэком на локальный BGE-reranker без ключа)
- **OCR** для скриншотов в документации
- **Sentry** — наблюдаемость (логи, трейсинг, метрики, AI-мониторинг)

Часть решений сознательно отложена и будет принята по данным реальной **обкатки** (финальная стратегия поиска, OCR против мультимодальных эмбеддингов для картинок, выбор векторного хранилища) — см. ADR проекта.

## Дорожная карта (укрупнённо)

Работа разбита на ~21 задачу по фазам, ведётся в Linear:

- **Фаза 0** — фундамент, согласования, сбор данных (репозиторий/CI, ADR по стеку, каркас проекта)
- **Фаза 1** — парсеры и модель (парсер конфигурации, индексатор доки + OCR, менеджер модели)
- **Фаза 2** — ядро поиска и MCP (поисковый слой, MCP-сервер, онбординг, поведенческий контракт)
- **Фаза 3** — обвязка (CLI/оркестрация индексации, Cohere-адаптеры, упаковка, живые процедуры)
- **Фаза 4** — обкатка (экспериментальный режим + телеметрия, LLM-судья, анализ, ежемесячные обновления)
- **Фаза 5** — опционально (мониторинг релизов)

Полная карта задач — в [`TASKS.md`](./TASKS.md).

## Цель проекта, честно

«Полностью без фантазий» — это *направление*, а не выключатель: ни одна ИИ-система не даёт 100% гарантии. Цель — устойчивое поведение: **«почти всегда сам идёт в источники и отвечает по делу».**

## Лицензия

[MIT](./LICENSE) © Сергей Хлебников
