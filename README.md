# MCP-сервер сети ветклиник БиоВет

[![BioVet MCP server on Glama](https://glama.ai/mcp/servers/Bio-Vet/biovet-mcp/badges/score.svg)](https://glama.ai/mcp/servers/Bio-Vet/biovet-mcp)
[![CI](https://github.com/Bio-Vet/biovet-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/Bio-Vet/biovet-mcp/actions/workflows/ci.yml)
[![smithery badge](https://smithery.ai/badge/pwnz13/biovet)](https://smithery.ai/servers/pwnz13/biovet)

**Официальный MCP-сервер сети [БиоВет](https://bio.vet/) — 20 ветеринарных клиник в Москве и Реутове, все работают круглосуточно, со своей лабораторией.**

ИИ-ассистент (Claude, агентные браузеры, любой MCP-клиент) через него может: найти ближайшую клинику, назвать живую цену из прайса, показать свободное время врача, **записать на приём**, оценить срочность по симптомам, проверить продукт или комнатное растение и ответить на вопрос о здоровье питомца из базы, вычитанной врачами сети.

| | |
|---|---|
| **Адрес** | `https://bio.vet/mcp` — Streamable HTTP, JSON-RPC 2.0, без состояния, авторизация не нужна |
| **Манифест** | [`bio.vet/.well-known/mcp.json`](https://bio.vet/.well-known/mcp.json) |
| **Язык** | инструменты принимают и отвечают по-русски: клиники в Москве, клиенты пишут по-русски |
| **Данные** | живые — прайс и расписание берутся из CRM сети в момент запроса |

## Подключение

**Claude Code** — одной строкой:

```bash
claude mcp add --transport http biovet https://bio.vet/mcp
```

**Claude Desktop** — *Настройки → Connectors → Add custom connector* → `https://bio.vet/mcp`.

**Любой MCP-клиент** — укажите тот же адрес как remote-сервер типа `streamable-http`.

## Инструменты

| Инструмент | Что делает |
|---|---|
| `find_clinic` | Ближайшая из 20 клиник по району, метро, улице или координатам — с живым рейтингом Яндекс.Карт и `clinic_id` для записи |
| `get_prices` | Живые цены из прайса сети: 900+ позиций, одинаковых во всех клиниках — ночью, в выходные и праздники цена та же |
| `check_slots` | Врачи клиники и их свободное время; если нужной специальности в клинике нет, подскажет, в каких клиниках сети она есть |
| `book_visit` | Настоящая запись на приём (имя, телефон, клиника, время). Ассистент **обязан** показать человеку сводку и получить подтверждение до вызова |
| `triage` | Срочность по описанию симптомов: *ехать немедленно / показать врачу сегодня / наблюдать*. Таблица признаков подтверждена врачами сети. Это маршрутизация, не диагноз |
| `food_check` | Можно ли питомцу продукт или комнатное растение — справочник на 71 позицию, вычитанный врачами |
| `ask_vet_kb` | Ответ на вопрос о здоровье и уходе из базы 3 727 пар «вопрос — ответ врача» со ссылкой на статью-источник |

## Как это выглядит

```bash
curl -s -X POST https://bio.vet/mcp -H 'Content-Type: application/json' -d '{
  "jsonrpc": "2.0", "id": 1, "method": "tools/call",
  "params": { "name": "find_clinic", "arguments": { "query": "Кутузовская" } }
}'
```

```
по запросу «Кутузовская» (сеть БиоВет, все работают круглосуточно, единый телефон +7 (495) 323-71-71):
• Кутузовская — Студенческая, 26 (м. Кутузовская). Рейтинг 4,8 (418 оценок). clinic_id=58
```

Ещё примеры ответов — коротко, как их видит ассистент:

```
triage    · «кот не может помочиться, кричит на лотке»
          → 🔴 Похоже на угрозу жизни. Ехать немедленно, все 20 клиник принимают круглосуточно

food_check · «виноград», собака
          → НЕЛЬЗЯ — опасно. Даже небольшое количество может вызвать острую почечную
            недостаточность. Точная токсичная доза неизвестна — опасна любая

get_prices · «узи брюшной полости»
          → УЗИ брюшной полости с заключением — 3 500 ₽ · A-FAST при травме — 2 150 ₽
```

## Правила, по которым сервер работает

- **Дозировок и назначений нет.** Ни один инструмент не выдаёт схем лечения и доз: триаж всегда заканчивается «покажите врачу», справочники ведут на приём.
- **Запись — только с подтверждения человека.** `book_visit` создаёт настоящую заявку в CRM клиники: ассистент сначала показывает сводку (клиника, врач, время, телефон) и получает явное «да».
- **Защита от лавины заявок:** не больше трёх записей с одного адреса в час; каждая заявка помечена источником `mcp` в журнале клиники, администратор перезванивает и подтверждает.
- **Чтение бесплатно и без слежки** — мы не собираем данные о клиентах MCP.

## Данные

Всё, что отдают инструменты, опубликовано как открытые датасеты сети (CC BY 4.0):

[ru-pet-health-qa](https://github.com/Bio-Vet/ru-pet-health-qa) — 3 727 пар «вопрос — ответ врача» (`ask_vet_kb`) ·
[pet-food-safety](https://github.com/Bio-Vet/pet-food-safety) — 71 продукт и растение (`food_check`) ·
[pet-symptom-triage](https://github.com/Bio-Vet/pet-symptom-triage) — 27 признаков срочности (`triage`) ·
[breed-health-reference](https://github.com/Bio-Vet/breed-health-reference) — 197 пород и их риски ·
[pet-parasite-prevention-schedules](https://github.com/Bio-Vet/pet-parasite-prevention-schedules) — 35 схем обработок ·
[pet-vaccination-schedules](https://github.com/Bio-Vet/pet-vaccination-schedules) — схемы прививок ·
[pet-vital-signs](https://github.com/Bio-Vet/pet-vital-signs) — физиологические нормы ·
[ru-vet-prices](https://github.com/Bio-Vet/ru-vet-prices) — срез цен по рынку

Витрина с описаниями — [bio.vet/open-data](https://bio.vet/open-data/).

## Поддержка

Сервер живёт на стороне клиники, кода в этом репозитории нет — здесь витрина, манифест и ежедневная проверка живости эндпоинта. Нашли ошибку в ответе инструмента или хотите инструмент, которого не хватает, — заведите [issue](https://github.com/Bio-Vet/biovet-mcp/issues). Вопрос о питомце — по телефону +7 (495) 323-71-71 круглосуточно.

---

## English

**Official MCP server of the BioVet network — 20 veterinary clinics in Moscow, Russia, all open 24/7.**

Lets an AI assistant find a clinic, quote live prices, show open appointment slots, **book a visit**, run a symptom urgency check, check whether a food or houseplant is safe, and answer pet-health questions from the network's vet-reviewed knowledge base.

- **Endpoint:** `https://bio.vet/mcp` — Streamable HTTP, stateless JSON-RPC 2.0, no auth
- **Manifest:** [`bio.vet/.well-known/mcp.json`](https://bio.vet/.well-known/mcp.json)
- **Language:** tools accept and answer **in Russian** — the clinics and their clients are Russian-speaking
- **Quick start:** `claude mcp add --transport http biovet https://bio.vet/mcp`

| Tool | What it does |
|---|---|
| `find_clinic` | Nearest of 20 clinics by district / metro / street or coordinates, with live Yandex Maps ratings and `clinic_id` |
| `get_prices` | Live prices from the network's price list (900+ services, identical in every clinic, no night surcharge) |
| `check_slots` | Doctors of a clinic with open appointment times; suggests other clinics when the specialty is missing |
| `book_visit` | Creates a real appointment. The assistant must show a summary and get explicit confirmation first |
| `triage` | Urgency check — *go now / see a vet today / observe* — from a sign table approved by the network's veterinarians. Routing, never a diagnosis |
| `food_check` | Is this food or houseplant safe for a pet — 71-item vet-reviewed reference |
| `ask_vet_kb` | Answers a pet-health question from 3 727 vet-reviewed Q&A pairs, with a link to the source article |

**Safety:** no drug dosages or treatment plans are ever returned; bookings are rate-limited (3 per IP per hour), tagged in the clinic journal and confirmed by a call-back; read tools are free and untracked.

**Data:** every reference behind the tools is published as an open dataset (CC BY 4.0) at [github.com/Bio-Vet](https://github.com/Bio-Vet) — see [bio.vet/open-data](https://bio.vet/open-data/).
