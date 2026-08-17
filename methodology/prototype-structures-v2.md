# Semantic Graph Prototype Structures v2

**Статус:** нормативная спецификация протоструктур и методики исследования.

## 1. Назначение

Протоструктуры предназначены для доказательного восстановления семантики существующей системы до проектирования новой архитектуры.

Основная цепочка:

```text
SOURCE
  ↓
SOURCE FACT
  ↓
EVIDENCE
  ↓
SEMANTIC CLAIM
  ↓
FRAME / RELATION
  ↓
CAPABILITY
  ↓
PLATFORM CANDIDATE
  ↓
HUMAN ARCHITECTURAL DECISION
```

Протоструктуры не являются финальным графом и не являются архитектурным решением. Их задача — сделать исследование воспроизводимым, проверяемым и пригодным для последующего машинного анализа.

---

## 2. Основные принципы

1. Исследуется существующая система, а не желаемая архитектура.
2. Факт и его семантическая интерпретация хранятся раздельно.
3. Каждое существенное утверждение должно иметь provenance.
4. `CONFIRMED`, `INFERRED` и `UNKNOWN` не смешиваются.
5. `UNKNOWN` является допустимым состоянием.
6. Противоречивые Evidence не удаляются.
7. LLM может извлекать и предлагать, но не получает самостоятельного права подтверждать архитектурные факты.
8. `Platform Candidate` является выводом исследования, а `Platform` — внешним архитектурным решением.
9. Исходные Evidence являются неизменяемыми после публикации Research Pass.

---

## 3. Объектная модель

Протоструктурный граф состоит из следующих объектов:

- `SourceFact` — наблюдение непосредственно из источника.
- `Evidence` — доказательство конкретного семантического утверждения.
- `SemanticClaim` — отдельное утверждение, которому присваивается confidence.
- `Frame` — семантический объект, состоящий из identity и slots.
- `Slot` — именованное свойство или роль Frame.
- `Relation` — типизированное утверждение о связи двух Frames.
- `Conflict` — зафиксированное противоречие между Evidence или claims.
- `Capability` — агрегированный семантический функциональный контур.
- `PlatformCandidate` — кандидат на последующее выделение платформы.
- `GraphSnapshot` — опубликованное состояние исследовательского графа.

`SemanticClaim` введён специально: confidence не должен относиться только к Frame целиком. Отдельные claims внутри Frame и Relations должны иметь собственную доказательную историю.

---

## 4. SourceFact

`SourceFact` является максимально близким к источнику наблюдением. Он не должен содержать выводов о назначении объекта.

| Поле | Назначение |
|---|---|
| `id` | Уникальный стабильный идентификатор |
| `source_type` | Тип исходного материала |
| `source_location` | Файл, таблица, колонка, метод, endpoint и т. п. |
| `observation` | Краткое воспроизводимое наблюдение |
| `raw_reference` | Точная ссылка на источник, если доступна |
| `status` | `ACTIVE` или `SUPERSEDED` |

Допустимые `source_type`:

```text
DB_SCHEMA
SQL
CODE
API
TEST
CONFIGURATION
RUNTIME
DOCUMENTATION
```

Пример:

```text
cart.id_user → users.id_user
```

— это `SourceFact`.

Формулировка:

```text
Cart OWNS User
```

— уже `SemanticClaim`.

---

## 5. Evidence

`Evidence` связывает один или несколько `SourceFact` с конкретным `SemanticClaim`.

### Evidence types

| Evidence type | Strength category | Назначение |
|---|---|---|
| `FOREIGN_KEY` | `STRUCTURAL` | Явная relational связь |
| `TABLE_SCHEMA` | `STRUCTURAL` | Структура, тип или ограничение объекта БД |
| `SQL_USAGE` | `BEHAVIORAL` | Фактическое чтение/изменение данных SQL |
| `CODE_USAGE` | `BEHAVIORAL` | Поведение, вызовы и использование объектов кодом |
| `RUNTIME` | `BEHAVIORAL` | Наблюдение реального выполнения |
| `API_CONTRACT` | `CONTRACTUAL` | Публичный контракт |
| `TEST` | `TEST` | Проверенное тестом поведение |
| `CONFIGURATION` | `CONFIGURATION` | Значение или правило конфигурации |
| `DOCUMENTATION` | `DOCUMENTARY` | Заявленное назначение или контекст |

`Evidence strength` не является универсальным рейтингом истинности. Он отвечает на вопрос, какой аспект утверждения данный источник способен подтверждать.

Например:

- FK силён для структурной связи;
- код — для application behavior;
- API contract — для публичного контракта;
- тест — для проверенного поведения;
- документация — для заявленного назначения.

---

## 6. Независимость Evidence

Для повышения confidence учитывается независимость Evidence.

Два наблюдения из одного метода или одной копии документа не считаются двумя независимыми доказательствами.

Практические правила:

- DB FK + отдельное подтверждение в коде — независимая пара;
- два вызова одной функции — обычно одно поведенческое Evidence;
- документация, автоматически сгенерированная из того же кода, не является независимым источником;
- тест, повторяющий конкретный вызов кода, повышает доказательность поведения, но не создаёт независимого архитектурного источника.

Каждому Evidence может присваиваться `independent_group`. Evidence из одной группы не должно учитываться как независимое друг от друга.

---

## 7. SemanticClaim

`SemanticClaim` — атомарное утверждение, которое может быть подтверждено или опровергнуто.

Примеры:

```text
Cart exists as persisted entity.
Cart.owner → User.
Cart USES Ticket.
ticket.id_trip → Trip.
site_constant.ticket_booking_duration CONFIGURES TicketBooking.
```

Каждый claim имеет:

- `id`;
- `subject`;
- `predicate`;
- `object` или `value`;
- `evidence_ids`;
- `confidence`;
- `status`.

Именно `SemanticClaim` является основной единицей эпистемологического контроля.

---

## 8. Semantic Frame

`Frame` представляет устойчивый семантический объект.

Допустимые типы:

```text
DOMAIN
CAPABILITY
ENTITY
API
COMPONENT
CONFIGURATION
EXTERNAL_SERVICE
EVENT
FSM
PLATFORM_CANDIDATE
```

Frame имеет:

- identity;
- slots;
- implementation;
- persistence;
- configuration;
- evidence;
- confidence;
- status.

`Frame` может быть `CONFIRMED` как существующий объект, даже если часть его slots имеет `UNKNOWN`.

Например:

```text
Frame: Cart
confidence: CONFIRMED

owner → User
confidence: CONFIRMED

product → UNKNOWN

property → UNKNOWN
```

---

## 9. Slot

`Slot` — роль или свойство Frame.

Каждый Slot связан с отдельным SemanticClaim либо непосредственно с его доказательной историей.

### Правило UNKNOWN

Если свойство обнаружено, но target/value не установлен:

```text
slot.confidence = UNKNOWN
```

`UNKNOWN` slot не понижает confidence самого Frame.

### Пустой slot и UNKNOWN — разные состояния

**Пустой slot** означает:

> свойство пока не включено в модель.

**UNKNOWN** означает:

> свойство уже обнаружено как значимое, но его содержание не установлено.

---

## 10. Relation

`Relation` — специализированный SemanticClaim, где subject и object являются Frames.

### Категории отношений

#### STRUCTURAL

```text
FK
REFERENCES
PERSISTS
```

#### BEHAVIORAL

```text
CALLS
USES
READS
WRITES
TRIGGERS
PRODUCES
CONSUMES
```

#### SEMANTIC

```text
OWNS
BELONGS_TO
DEPENDS_ON
CONFIGURES
```

#### ARCHITECTURAL

```text
IMPLEMENTS
EXPOSES
PROVIDES
```

Нельзя использовать `FK` для application-level связи.

Например, отсутствие FK:

```text
cart → ticket
```

не опровергает:

```text
Cart --USES--> Ticket
```

если последняя связь доказана кодом.

---

## 11. Confidence model

Допустимы только:

```text
CONFIRMED
INFERRED
UNKNOWN
```

### CONFIRMED

Утверждение непосредственно подтверждено достаточным Evidence для данного типа утверждения либо подтверждено несколькими независимыми, непротиворечивыми Evidence.

### INFERRED

Утверждение является обоснованной семантической интерпретацией, но прямого подтверждения недостаточно.

### UNKNOWN

Данных недостаточно даже для обоснованной интерпретации.

Confidence не вычисляется как:

```text
average
min
max
```

по набору slots или Evidence.

---

## 12. Confidence Frame

Confidence Frame относится прежде всего к **identity** Frame.

Правила:

1. `Frame = CONFIRMED`, если identity подтверждена напрямую.
2. `Frame = INFERRED`, если identity выведена из Evidence, но прямого подтверждения нет.
3. `Frame = UNKNOWN`, если identity не установлена.
4. `UNKNOWN` slots не понижают Frame confidence.
5. Claims и Relations внутри Frame имеют собственный confidence.
6. Frame не может быть `CONFIRMED` только потому, что его slots кажутся подтверждёнными.

Пример:

```text
Cart = CONFIRMED
Cart.owner → User = CONFIRMED
Cart.product → UNKNOWN
Cart USES Ticket = INFERRED
```

---

## 13. Правила повышения confidence

LLM может **предложить** повышение confidence, но финальное `CONFIRMED` должно удовлетворять одному из условий:

1. существует прямое Evidence, достаточное для данного типа утверждения;
2. существует минимум два независимых непротиворечивых Evidence;
3. выполнен явный human review с указанием основания.

Если Evidence противоречивы, `CONFIRMED` автоматически не присваивается.

Claim остаётся `INFERRED` или `UNKNOWN` до разрешения конфликта.

### Запрет

Нельзя повышать confidence на основании:

- авторитетности LLM;
- повторения одной и той же информации в нескольких местах;
- названия класса/таблицы;
- предположения о «типичной архитектуре»;
- отсутствия найденного контрпримера.

---

## 14. Разрешение конфликтов

Нет глобального правила:

```text
DB > code > documentation
```

Сначала определяется тип утверждения, затем оценивается, какой источник способен его подтвердить.

### Для структурных утверждений

Первичны:

```text
DB_SCHEMA
FOREIGN_KEY
TABLE_SCHEMA
```

### Для фактического поведения

Первичны:

```text
CODE_USAGE
SQL_USAGE
RUNTIME
TEST
```

### Для публичного контракта

Первичны:

```text
API_CONTRACT
TEST
RUNTIME
```

### Для заявленного назначения

`DOCUMENTATION` является Evidence, но не отменяет фактическую реализацию.

### Для Configuration

Само значение конфигурации подтверждает наличие настройки.

Использование этой настройки кодом подтверждает её поведенческую роль.

Если конфликт не разрешён:

```text
Conflict.status = OPEN
```

а исходный claim остаётся `INFERRED` или `UNKNOWN`.

---

## 15. Capability

`Capability` — устойчивый функциональный контур, восстановленный из связанных действий, данных, правил и интеграций.

Capability должна описывать:

```text
implemented_by
exposes
persists
uses
configured_by
dependencies
evidence
confidence
```

Capability не выводится только из:

- имени класса;
- имени таблицы;
- endpoint;
- директории.

Например:

```text
Capability: Driver Selection

implemented_by:
  setDriver()
  offerOrder()
  driver-selection logic

persists:
  order_driver
  order_driver_attempt
  order_driver_select

configured_by:
  d_s_sorting_*
  d_s_offered_drivers_count
  d_s_offered_drivers_duration
```

---

## 16. Configuration как семантический объект

Если настройка влияет на:

- бизнес-правило;
- безопасность;
- lifecycle;
- интеграцию;
- форму взаимодействия;

она становится `Configuration Frame`.

Пример:

```text
ConfigurationVariable
  name = ticket_booking_duration
  value = 691200
```

затем:

```text
ConfigurationVariable
        |
    CONFIGURES
        ↓
TicketBooking
```

Сам факт существования строки в `site_constant` подтверждает ConfigurationVariable.

Но семантическая роль `CONFIGURES → X` требует Evidence её использования или однозначного контекста.

---

## 17. Platform Candidate

`Platform Candidate` является исследовательским выводом.

Он не означает, что граница платформы уже установлена.

Обязательные элементы:

```text
based_on
included_capabilities
dependencies
external_dependencies
boundary_questions
unresolved_questions
critical_unknowns
evidence
status
```

### boundary_questions

Относятся к границе уже понятного объекта:

> Что должно входить в ответственность Candidate?

### unresolved_questions

Относятся к отсутствующим знаниям:

> Что мы пока вообще не знаем?

Candidate не может автоматически стать Platform.

---

## 18. Platform Decision Gate

Gate состоит из двух частей.

### 18.1 Graph-checkable preconditions

Граф должен позволять проверить:

- ключевая capability исследована;
- граница capability описана;
- ключевые зависимости определены;
- данные и external integrations определены;
- critical UNKNOWN отсутствуют либо явно оформлены как принятый риск;
- ownership/boundary questions имеют решение;
- публичный контракт может быть сформулирован;
- миграционные последствия исследованы.

### 18.2 Human sign-off

Внешним актом должны быть:

- архитектурный review;
- явное решение команды о создании Platform;
- фиксация решения вне автоматического inference механизма.

Граф может определить:

```text
PRECONDITIONS = PASSED
```

но не может самостоятельно создать:

```text
Platform = APPROVED
```

---

## 19. Research Pass

`Research Pass` — ограниченный воспроизводимый цикл исследования с заданным Scope.

### Этапы

1. **Scope** — определить участок системы, вопросы и допустимые источники.
2. **Extraction** — извлечь SourceFacts.
3. **Evidence linking** — привязать Evidence к claims.
4. **Semantic proposal** — предложить Frames, Slots и Relations.
5. **Validation** — проверить исходные места.
6. **Review** — проверить спорные и CONFIRMED claims.
7. **Graph update** — создать новый snapshot.
8. **Gap report** — перечислить UNKNOWN, Conflicts и следующие вопросы.

### Критерий завершения Pass

Pass завершён, когда:

- поставленный вопрос получил подтверждённый ответ; **или**
- вопрос явно зафиксирован как `UNKNOWN`/`Conflict`;
- для UNKNOWN/Conflict определено следующее исследовательское действие.

«Посмотрели достаточно файлов» не является критерием завершения.

---

## 20. Роли человека и LLM

### LLM / автоматический extractor может

- находить кандидатов SourceFact;
- предлагать Frames;
- предлагать Relations;
- сопоставлять Evidence;
- обнаруживать конфликты;
- предлагать изменение confidence;
- формировать Gap Report.

### LLM не может без правила подтверждения

- объявлять claim `CONFIRMED`;
- создавать Platform;
- удалять противоречивое Evidence;
- считать отсутствие найденного Evidence доказательством отсутствия объекта.

### Human review обязателен

Для:

- архитектурно значимых `CONFIRMED` claims;
- разрешения существенных конфликтов;
- перехода `Platform Candidate → Platform`.

---

## 21. Versioning

Используется модель:

```text
Immutable Evidence
        +
Graph Snapshots
```

SourceFact и Evidence после публикации Pass не переписываются задним числом.

Исправление создаёт новое Evidence/SourceFact с указанием причины изменения.

Граф публикуется снимками:

```text
v0
v0.1
v0.2
...
```

Каждый snapshot должен позволять восстановить состояние графа и происхождение существенных утверждений.

История предыдущего вывода не уничтожается.

---

## 22. Conflict

`Conflict` фиксирует противоречие между Evidence или SemanticClaims.

Поля:

```text
id
claim_id
evidence_ids
status
resolution
```

Допустимые состояния:

```text
OPEN
RESOLVED
ACCEPTED_AS_UNCERTAINTY
```

Наличие Conflict не является ошибкой модели.

Скрытый Conflict является ошибкой процесса.

---

## 23. Quality Gates

Перед публикацией Research Pass проверяется:

- каждый существенный claim имеет Evidence или явно отмечен `UNKNOWN`;
- `CONFIRMED` claims имеют допустимое основание;
- application-level relations не помечены как FK без DB evidence;
- противоречия зарегистрированы;
- UNKNOWN не скрыты;
- Platform Candidate не оформлен как Platform;
- source references воспроизводимы;
- новый snapshot не уничтожает историю предыдущего;
- LLM-generated proposals отмечены как предложения до review.

---

## 24. Минимальная JSON Schema

Полная машинная схема является частью этой спецификации.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Semantic Graph Prototype Structures v2",
  "type": "object",
  "required": [
    "schema_version",
    "snapshot_id",
    "source_facts",
    "evidence",
    "claims",
    "frames",
    "relations",
    "capabilities",
    "candidates",
    "conflicts"
  ],
  "properties": {
    "schema_version": { "type": "string" },
    "snapshot_id": { "type": "string" },
    "source_facts": {
      "type": "array",
      "items": { "$ref": "#/$defs/sourceFact" }
    },
    "evidence": {
      "type": "array",
      "items": { "$ref": "#/$defs/evidence" }
    },
    "claims": {
      "type": "array",
      "items": { "$ref": "#/$defs/claim" }
    },
    "frames": {
      "type": "array",
      "items": { "$ref": "#/$defs/frame" }
    },
    "relations": {
      "type": "array",
      "items": { "$ref": "#/$defs/relation" }
    },
    "capabilities": {
      "type": "array",
      "items": { "$ref": "#/$defs/capability" }
    },
    "candidates": {
      "type": "array",
      "items": { "$ref": "#/$defs/candidate" }
    },
    "conflicts": {
      "type": "array",
      "items": { "$ref": "#/$defs/conflict" }
    }
  }
}
```

### Обязательные правила Schema

`SourceFact`:

```text
id
source_type
source_location
observation
status
```

`Evidence`:

```text
id
type
source_fact_ids
strength_category
```

`SemanticClaim`:

```text
id
subject
predicate
confidence
evidence_ids
status
```

`Frame`:

```text
id
type
name
identity_claim_id
confidence
status
```

`Relation`:

```text
id
type
source_frame
target_frame
claim_id
confidence
```

`PlatformCandidate`:

```text
id
name
based_on
status
```

---

## 25. Методика перехода от факта к семантике

Каждый переход должен быть объясним:

```text
DB:
cart.id_user → users.id_user
        ↓
SourceFact
        ↓
Evidence
        ↓
SemanticClaim:
Cart OWNS User
        ↓
Frame:
Cart
        ↓
Capability:
Cart Management
        ↓
Platform Candidate:
Cart
```

Неправильно:

```text
table cart
    ↓
Platform Cart
```

Нельзя перескакивать через Evidence и SemanticClaim.

---

## 26. Работа с UNKNOWN

`UNKNOWN` — не ошибка и не недостаток качества.

Он является указателем следующего исследования.

Например:

```text
Cart.product → UNKNOWN
```

означает:

- колонка существует;
- она семантически используется;
- target/value ещё не установлен.

Следующий Research Pass должен искать:

```text
где формируется product
где изменяется product
где читается product
какие значения туда попадают
какая capability их интерпретирует
```

---

## 27. Критерий зрелости протоструктуры

Для конкретного Scope протоструктура достаточно зрелая, если:

1. существенные SourceFacts извлечены;
2. существенные claims имеют Evidence;
3. identity ключевых Frames определена;
4. Relations типизированы;
5. confidence каждой существенной сущности объясним;
6. UNKNOWN и Conflicts явно перечислены;
7. следующий Research Pass определяется конкретными вопросами;
8. snapshot воспроизводим по источникам.

Завершённость **не означает отсутствие UNKNOWN**.

Она означает, что неизвестное стало явным и управляемым объектом дальнейшего исследования.

---

## 28. Переход к полноценному Semantic Graph

После стабилизации схемы и нескольких Research Pass протоструктуры могут быть импортированы в полноценный Semantic Graph.

При импорте не должна теряться provenance:

```text
Frame
  ↓
SemanticClaim
  ↓
Evidence
  ↓
SourceFact
  ↓
Original Source
```

Любая существенная семантическая связь должна оставаться трассируемой до исходного материала.

---

## 29. Ключевой принцип

Мы не строим платформы вокруг названий классов, таблиц или endpoint'ов.

Мы восстанавливаем:

```text
Existing System
      ↓
Evidence
      ↓
Semantic Model
      ↓
Capabilities
      ↓
Dependencies
      ↓
Platform Candidates
      ↓
Human Architectural Decision
      ↓
Platform Contract
```

Главный критерий качества:

> Граф должен быть способен объяснить не только «что известно», но и «почему это известно», «насколько это подтверждено» и «что ещё неизвестно».


---

## Appendix A. Machine-readable JSON Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Semantic Graph Prototype Structures v2",
  "type": "object",
  "required": [
    "schema_version",
    "snapshot_id",
    "source_facts",
    "evidence",
    "claims",
    "frames",
    "relations",
    "capabilities",
    "candidates",
    "conflicts"
  ],
  "properties": {
    "schema_version": {
      "type": "string"
    },
    "snapshot_id": {
      "type": "string"
    },
    "source_facts": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/sourceFact"
      }
    },
    "evidence": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/evidence"
      }
    },
    "claims": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/claim"
      }
    },
    "frames": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/frame"
      }
    },
    "relations": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/relation"
      }
    },
    "capabilities": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/capability"
      }
    },
    "candidates": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/candidate"
      }
    },
    "conflicts": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/conflict"
      }
    }
  },
  "$defs": {
    "confidence": {
      "type": "string",
      "enum": [
        "CONFIRMED",
        "INFERRED",
        "UNKNOWN"
      ]
    },
    "sourceFact": {
      "type": "object",
      "required": [
        "id",
        "source_type",
        "source_location",
        "observation",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "source_type": {
          "type": "string",
          "enum": [
            "DB_SCHEMA",
            "SQL",
            "CODE",
            "API",
            "TEST",
            "CONFIGURATION",
            "RUNTIME",
            "DOCUMENTATION"
          ]
        },
        "source_location": {
          "type": "string"
        },
        "observation": {
          "type": "string"
        },
        "raw_reference": {
          "type": "string"
        },
        "status": {
          "type": "string",
          "enum": [
            "ACTIVE",
            "SUPERSEDED"
          ]
        }
      }
    },
    "evidence": {
      "type": "object",
      "required": [
        "id",
        "type",
        "source_fact_ids",
        "strength_category"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "type": {
          "type": "string",
          "enum": [
            "FOREIGN_KEY",
            "TABLE_SCHEMA",
            "SQL_USAGE",
            "CODE_USAGE",
            "RUNTIME",
            "API_CONTRACT",
            "TEST",
            "CONFIGURATION",
            "DOCUMENTATION"
          ]
        },
        "source_fact_ids": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "minItems": 1
        },
        "strength_category": {
          "type": "string",
          "enum": [
            "STRUCTURAL",
            "BEHAVIORAL",
            "CONTRACTUAL",
            "TEST",
            "CONFIGURATION",
            "DOCUMENTARY"
          ]
        },
        "independent_group": {
          "type": "string"
        },
        "note": {
          "type": "string"
        }
      }
    },
    "claim": {
      "type": "object",
      "required": [
        "id",
        "subject",
        "predicate",
        "confidence",
        "evidence_ids",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "subject": {
          "type": "string"
        },
        "predicate": {
          "type": "string"
        },
        "object": {
          "type": "string"
        },
        "value": {
          "type": [
            "string",
            "number",
            "boolean",
            "null"
          ]
        },
        "confidence": {
          "$ref": "#/$defs/confidence"
        },
        "evidence_ids": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "status": {
          "type": "string",
          "enum": [
            "ACTIVE",
            "SUPERSEDED"
          ]
        }
      }
    },
    "frame": {
      "type": "object",
      "required": [
        "id",
        "type",
        "name",
        "confidence",
        "identity_claim_id",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "type": {
          "type": "string",
          "enum": [
            "DOMAIN",
            "CAPABILITY",
            "ENTITY",
            "API",
            "COMPONENT",
            "CONFIGURATION",
            "EXTERNAL_SERVICE",
            "EVENT",
            "FSM",
            "PLATFORM_CANDIDATE"
          ]
        },
        "name": {
          "type": "string"
        },
        "identity_claim_id": {
          "type": "string"
        },
        "confidence": {
          "$ref": "#/$defs/confidence"
        },
        "status": {
          "type": "string",
          "enum": [
            "RESEARCHING",
            "STABLE",
            "SUPERSEDED"
          ]
        }
      }
    },
    "relation": {
      "type": "object",
      "required": [
        "id",
        "type",
        "source_frame",
        "target_frame",
        "claim_id",
        "confidence"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "type": {
          "type": "string"
        },
        "source_frame": {
          "type": "string"
        },
        "target_frame": {
          "type": "string"
        },
        "claim_id": {
          "type": "string"
        },
        "confidence": {
          "$ref": "#/$defs/confidence"
        }
      }
    },
    "capability": {
      "type": "object",
      "required": [
        "id",
        "frame_id",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "frame_id": {
          "type": "string"
        },
        "implemented_by": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "dependencies": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "configured_by": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "status": {
          "type": "string",
          "enum": [
            "RESEARCHING",
            "STABLE",
            "SUPERSEDED"
          ]
        }
      }
    },
    "candidate": {
      "type": "object",
      "required": [
        "id",
        "name",
        "based_on",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
        "based_on": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "included_capabilities": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "dependencies": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "external_dependencies": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "boundary_questions": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "unresolved_questions": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "critical_unknowns": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "status": {
          "type": "string",
          "enum": [
            "RESEARCHING",
            "MATURE_FOR_REVIEW",
            "REJECTED",
            "APPROVED_AS_PLATFORM"
          ]
        }
      }
    },
    "conflict": {
      "type": "object",
      "required": [
        "id",
        "claim_id",
        "evidence_ids",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "claim_id": {
          "type": "string"
        },
        "evidence_ids": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "status": {
          "type": "string",
          "enum": [
            "OPEN",
            "RESOLVED",
            "ACCEPTED_AS_UNCERTAINTY"
          ]
        },
        "resolution": {
          "type": "string"
        }
      }
    }
  }
}
```
