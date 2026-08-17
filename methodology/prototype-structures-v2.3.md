# Semantic Graph Prototype Structures v2.3

**Статус:** нормативная спецификация протоструктур и методики исследования.

**Версия:** v2.2 — согласование invariants, lifecycle, audit trail и Research Pass outputs.

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
- `ResearchScope` — формализованная граница конкретного исследования.
- `GraphSnapshot` — опубликованное состояние исследовательского графа, связанное с Research Pass и предыдущим snapshot.

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

Каждому Evidence присваивается `independent_group`.

Конвенция:
- если Evidence является единственным представителем своей группы, `independent_group` равен его собственному `id`;
- если несколько Evidence получены из одного независимого источникового основания, они получают один общий group id;
- `independent_group = UNGROUPED` не используется.

Таким образом, одиночное Evidence автоматически образует собственную независимую группу, но не получает дополнительного веса только из-за этого.

Evidence из одной группы не должны учитываться как независимые друг от друга.

### Cardinality

Допускается:

```text
1 Evidence → N Claims
1 Claim    → N Evidence
```

Одно Evidence может поддерживать несколько claims, если каждый claim действительно следует из одного и того же наблюдения.

Это не делает claims независимыми: независимость относится к Evidence groups, а не к количеству claims.

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

Каждый claim может также иметь audit metadata:

```text
proposed_by
proposed_at
proposal_reason
human_confirmed_confidence
reviewed_by
reviewed_at
review_reference
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
```

Frame имеет:

- `id`;
- `type`;
- `name`;
- `identity_claim_id`;
- `slots`;
- `implementation`;
- `persistence`;
- `configuration`;
- `evidence_ids`;
- `confidence`;
- `status`.

`implementation`, `persistence` и `configuration` являются ссылками на Frames, Claims или связанные объекты, а не свободным архитектурным описанием.

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

### Slot ↔ Claim invariant

Для каждого Slot:

```text
slot.claim_id → Claim
```

Если Slot содержит `target_frame`, то:

```text
claim.object === slot.target_frame
```

Если Slot содержит `value`, то:

```text
claim.value === slot.value
```

Slot не может содержать одновременно `target_frame` и `value`.

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

Для каждой Relation:

```text
relation.claim_id → Claim
```

и должны выполняться invariants:

```text
claim.subject  === relation.source_frame
claim.object   === relation.target_frame
claim.predicate === relation.type
```

Relation не должна существовать с Claim, описывающим другую пару Frames или другой predicate.

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

## 11.1 Identity Claim

Каждый Frame обязан иметь отдельный `identity_claim_id`.

Identity Claim — это обычный `SemanticClaim`, но с фиксированной конвенцией:

```text
subject = frame_id
predicate = EXISTS_AS
object = frame.type
```

Пример:

```text
subject: frame:Cart
predicate: EXISTS_AS
object: ENTITY
```

Для конкретной сущности допускается дополнительно:

```text
predicate: IS_INSTANCE_OF
object: DomainType
```

Но именно `EXISTS_AS` является обязательным claim для определения confidence identity Frame.

Правило:

- `EXISTS_AS = CONFIRMED` → Frame identity = `CONFIRMED`;
- `EXISTS_AS = INFERRED` → Frame identity = `INFERRED`;
- `EXISTS_AS = UNKNOWN` → Frame identity = `UNKNOWN`.

Другие claims и slots не изменяют identity confidence автоматически.

### Identity invariant

Для каждого Frame:

```text
frame.confidence === claims[frame.identity_claim_id].confidence
```

`frame.confidence` является денормализованным представлением confidence identity claim и не может независимо изменяться.

Если identity claim меняет confidence, Frame должен получить то же значение в следующем snapshot.

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

`Capability` и `Frame(type = CAPABILITY)` не являются двумя независимыми представлениями.

Если Capability оформлена в машинной модели, `capability.frame_id` обязан ссылаться на Frame с:

```text
frame.type = CAPABILITY
```

Таким образом:

```text
Frame(type=CAPABILITY)
        ↑
        |
Capability
```

`Frame` является семантической идентичностью Capability, а объект `Capability` — её структурированным capability-профилем (`implemented_by`, `exposes`, `persists`, `uses`, `dependencies` и т. д.).

Для одной Capability не допускается несколько независимых Capability-объектов, ссылающихся на разные Frames без отдельного claim/provenance, объясняющего такое разделение.

Capability должна описывать:

```text
id
frame_id
implemented_by
exposes
persists
uses
configured_by
dependencies
evidence_ids
confidence
status
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
preconditions
status
```

### boundary_questions

Относятся к границе уже понятного объекта:

> Что должно входить в ответственность Candidate?

### unresolved_questions

Относятся к отсутствующим знаниям:

> Что мы пока вообще не знаем?

### preconditions

`preconditions` содержит результаты восьми graph-checkable проверок из §18.1. Каждая проверка является отдельным машиночитаемым состоянием, а не текстовой пометкой.

Candidate не может автоматически стать Platform.

`PlatformCandidate` является отдельным объектом, а не разновидностью `Frame`. Он может ссылаться на один или несколько Frames и Capabilities через `based_on` и `included_capabilities`.

Таким образом:

```text
Frame / Capability
       ↓
PlatformCandidate
       ↓
Human Architectural Decision
```

Не допускается существование второго независимого `Frame` с типом `PLATFORM_CANDIDATE` для описания того же Candidate.

## 18. Platform Decision Gate

Gate состоит из двух частей.

### 18.1 Graph-checkable preconditions

Для каждого Candidate граф должен хранить отдельное состояние:

```text
capability_researched
boundary_defined
dependencies_defined
data_and_integrations_defined
critical_unknowns_resolved
ownership_boundary_resolved
public_contract_formulable
migration_impact_researched
```

Допустимые значения:

```text
PASSED
FAILED
UNKNOWN
```

`PRECONDITIONS = PASSED` допускается только если все восемь проверок имеют значение `PASSED`.

`FAILED` и `UNKNOWN` различаются:

- `FAILED` означает, что проверка выполнена и условие не выполнено;
- `UNKNOWN` означает, что данных для проверки недостаточно.

Для Decision Gate оба состояния блокируют `PRECONDITIONS = PASSED`, но требуют разных следующих действий: `FAILED` требует устранения/пересмотра условия, `UNKNOWN` — дополнительного исследования.

Это не означает, что Candidate стал Platform. Это означает только, что он достиг состояния, при котором может быть вынесен на архитектурное решение.

### 18.2 Human sign-off

Переход Candidate → Platform является внешним актом.

При `status = APPROVED_AS_PLATFORM` обязательны:

```text
approved_by
approval_reference
approved_at
```

`approval_reference` должен ссылаться на внешний архитектурный review, решение или иной зарегистрированный акт.

Таким образом, запись:

```text
status = APPROVED_AS_PLATFORM
```

сама по себе недостаточна.

Граф может определить:

```text
PRECONDITIONS = PASSED
```

но не может самостоятельно создать human sign-off.

## 19. Research Pass

`Research Pass` — ограниченный воспроизводимый цикл исследования с заданным Scope.

### Этапы

1. **Scope** — определить участок системы, вопросы и допустимые источники.
2. **Scope freeze** — зафиксировать ResearchScope.
3. **Extraction** — извлечь SourceFacts.
4. **Evidence linking** — привязать Evidence к claims.
5. **Semantic proposal** — предложить Frames, Slots и Relations.
6. **Validation** — проверить исходные места.
7. **Review** — проверить спорные и CONFIRMED claims.
8. **Graph update** — создать новый snapshot.
9. **Gap report** — опубликовать обязательный Gap Report.

### ### Обязательный выход: Gap Report

Каждый завершённый Research Pass обязан публиковать `Gap Report`, содержащий:

```text
confirmed_findings
inferred_findings
unknowns
open_conflicts
next_actions
```

`next_actions` обязателен даже если все поставленные вопросы получили ответы: он должен либо содержать следующий scope, либо явно указывать, что для данного scope дальнейшее исследование не требуется.

Критерий завершения Pass

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

### Audit trail для confidence

Если LLM или автоматический extractor предлагает изменение confidence, исходное значение не перезаписывается без audit trail.

Минимальная запись предложения:

```text
proposed_confidence
proposed_by
proposed_at
proposal_reason
```

После human review:

```text
human_confirmed_confidence
reviewed_by
reviewed_at
review_reference
```

Если human review не выполнен, предложение остаётся предложением и не является основанием для `CONFIRMED`.

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
Research Pass
        +
Graph Snapshot
```

`GraphSnapshot` является отдельным объектом со следующими обязательными полями:

```text
snapshot_id
schema_version
created_at
pass_id
parent_snapshot_id
```

`parent_snapshot_id` может быть `null` только для первого snapshot.

`pass_id` связывает snapshot с конкретным Research Pass.

SourceFact и Evidence после публикации Pass не переписываются задним числом.

Исправление создаёт новое Evidence/SourceFact с указанием причины изменения.

Граф публикуется снимками:

```text
v0
v0.1
v0.2
...
```

Каждый snapshot должен позволять восстановить состояние графа и цепочку:

```text
GraphSnapshot
    ↓
Research Pass
    ↓
Frames / Claims / Relations
    ↓
Evidence
    ↓
SourceFact
    ↓
Original Source
```

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

## 24. Lifecycle и superseding

Разные типы объектов имеют собственные operational statuses, но действуют общие правила lifecycle.

### ACTIVE / RESEARCHING / OPEN

Объект является текущим рабочим объектом исследования.

### STABLE / COMPLETED / RESOLVED / MATURE_FOR_REVIEW

Объект достиг состояния, при котором его текущее содержание считается завершённым для данного уровня процесса.

### SUPERSEDED

`SUPERSEDED` означает, что объект больше не является текущей версией утверждения, но его история сохраняется.

Новый объект или snapshot должен содержать provenance на заменяемый объект либо на причину изменения.

`SUPERSEDED` не означает:

- удаление;
- ошибочность;
- отсутствие исторической ценности.

### Lifecycle rule

Нельзя изменять опубликованный SourceFact или Evidence задним числом. Исправление создаёт новую запись.

Для Claims, Frames и Capabilities изменение, влияющее на семантику, должно либо создавать новую версию объекта, либо переводить прежний объект в `SUPERSEDED` с сохранением связи на замену.

Для Candidate изменение его архитектурного статуса фиксируется новым snapshot; исторический статус не стирается.


## 25. Methodology — Research Pass

Методика определяет, **как проводить исследование существующей системы с использованием протоструктур**.

Главный принцип:

```text
Physical Sources
      ↓
SourceFact
      ↓
Evidence
      ↓
SemanticClaim
      ↓
Frame / Relation
      ↓
Capability
      ↓
PlatformCandidate
      ↓
Human Architectural Decision
```

Нельзя перескакивать через уровни.

Особенно запрещается переход:

```text
Source
  ↓
Platform Candidate
```

без промежуточных фактов, Evidence и Semantic Claims.

---

## 25.1. Research Scope

Каждый Research Pass начинается с формального `ResearchScope`.

Scope должен определить:

```text
scope_id
description
source_boundaries
questions
```

`source_boundaries` должны указывать, какие физические источники входят в исследование:

```text
core_system_source_snapshots
client_source_snapshots
database_source_snapshots
API contract version
test suite
configuration snapshot
documentation revision
```

Если исследование пересекает границу Core Backend ↔ Client Application, Scope MUST явно различать:

```text
core_system
client_id
client_version
source_snapshot_id
```

Например:

```yaml
core_system:
  source_snapshot_id: BACKEND-S0

client:
  client_id: taxi-web
  client_version: 1.4.0
  source_snapshot_id: FRONTEND-TAXI-WEB-S0
```

Если версия клиента неизвестна, фиксируется:

```text
client_version: UNKNOWN
```

а не обобщённое значение `Frontend`.

Scope является частью воспроизводимости.

Если источник изменился после начала Pass, это не должно незаметно смешиваться с исходным исследованием.

---

## 25.2. Source Inventory

До извлечения семантики составляется inventory физических источников.

Минимальные категории:

```text
CODE
DB_SCHEMA
DB_DATA
API_CONTRACT
TEST
RUNTIME
CONFIGURATION
DOCUMENTATION
```

Для каждого источника фиксируется стабильный идентификатор и версия/состояние, если они существуют.

Пример:

```text
CODE
repository = backend
commit = abc123
file = models/api.php
symbol = selectCart
lines = 120-147
```

или:

```text
DB_DATA
database = backend
table = site_constant
key = session_token_duration
column = value
row_identity = key
```

---

## 25.3. SourceSnapshot

`SourceSnapshot` — неизменяемое состояние физического источника, на котором основано исследование.

Это нормативное понятие методики и не зависит от Git.

Минимально:

```text
source_snapshot_id
source_type
artifact_identity
created_at / captured_at
```

`artifact_identity` должен позволять однозначно отличить данный снимок от другой версии источника.

Примеры:

```text
CODE:
archive + SHA-256

CODE:
repository + Git commit

DB_SCHEMA:
SQL dump + SHA-256

DB_DATA:
database snapshot + SHA-256

DOCUMENTATION:
document revision + artifact hash
```

После создания SourceSnapshot его содержимое считается immutable.

Изменение физического источника создаёт новый SourceSnapshot, а не изменяет старый.

### 25.3.1. Роль источника в системе

`SourceSnapshot` идентифицирует физическое состояние источника, но сам по себе не определяет его роль в исследуемой системе.

Методика различает как минимум:

```text
CORE_SYSTEM
CLIENT_APPLICATION
DATABASE
EXTERNAL_SYSTEM
DOCUMENTATION
CONFIGURATION
TEST_SUITE
```

Для `CLIENT_APPLICATION`, если данные известны, обязательно фиксируются:

```text
client_id
client_version
```

Таким образом, нельзя использовать единый абстрактный источник `Frontend`, если в системе существует несколько независимых клиентов одного Core Backend.

Например:

```text
CORE BACKEND
     │
     ├── taxi-web v1.4
     ├── taxi-driver v2.7
     └── taxi-admin v5.1
```

Каждый из них является отдельным SourceSnapshot и отдельным объектом исследования.

### 25.3.2. Client-specific provenance

Любой Claim, выведенный из frontend-кода, относится к конкретному клиенту и конкретной версии клиента.

Например:

```text
Taxi Web v1.4
    CALLS
Core Backend /auth
```

не доказывает:

```text
All clients CALL /auth
```

Следовательно:

```text
Client-specific observation
        ≠
Global system fact
```

`client_id` и `client_version` наследуются через `source_snapshot_id` и не должны теряться при построении Evidence и Claim.

### 25.3.3. Core Backend и Client Semantic Graph

В графе допускается одновременно существование:

```text
Core Backend Semantic Graph
```

и:

```text
Client-specific Semantic Graph
```

Они не являются независимыми графами знаний. Они соединяются Relations, отражающими использование клиентом возможностей Core Backend.

Например:

```text
Taxi Web v1.4
      ↓
Authentication UI
      ↓
CALLS
      ↓
Core Backend /auth
      ↓
IMPLEMENTED_BY
      ↓
API::authUser
```

Такой граф позволяет различать:

1. что Core Backend предоставляет;
2. что конкретный клиент использует;
3. что является общим для нескольких клиентов;
4. что является особенностью конкретного клиента.

### 25.3.4. Один Core Backend — много клиентов

Один SourceSnapshot Core Backend может быть связан с несколькими независимыми client Research Pass:

```text
BACKEND-S0
   │
   ├── RP-02: taxi-web v1.4
   ├── RP-03: taxi-driver v2.7
   └── RP-04: taxi-admin v5.1
```

Каждый такой Pass исследует конкретный Client Application и его конкретную версию.

Если разные клиенты используют одну backend capability, это создаёт независимые Evidence:

```text
taxi-web v1.4
    USES → Authentication

taxi-driver v2.7
    USES → Authentication
```

Повторение такой связи между независимыми клиентами является сильным Evidence для анализа переиспользуемости capability, но само по себе не превращает её в Platform Candidate.

### 25.3.5. Версии одного клиента

Разные версии одного клиента также являются разными SourceSnapshots:

```text
taxi-web v1.4 → FRONTEND-TAXI-WEB-S0
taxi-web v1.5 → FRONTEND-TAXI-WEB-S1
```

Если они используют разные backend flows:

```text
v1.4 → /auth
v1.5 → /token/authorized
```

оба утверждения сохраняются с собственной provenance.

Новый SourceSnapshot не переписывает старый.

При необходимости отдельная Relation может фиксировать изменение:

```text
CHANGED_TO
REPLACED_BY
NO_LONGER_USES
```

но только при наличии соответствующего Evidence.

### 25.3.6. Запрет глобализации client-specific claims

Следующее рассуждение недопустимо:

```text
Taxi Web v1.4 → /auth
therefore
Core Backend Authentication → /auth for all clients
```

Для вывода о нескольких клиентах необходимы независимые Evidence:

```text
Taxi Web v1.4 → /auth
Taxi Driver v2.7 → /auth
Taxi Admin v5.1 → /auth
```

Только после этого можно сформировать более общий Claim о характере использования Core Backend.

Confidence такого общего Claim определяется обычными правилами Evidence independence и не повышается автоматически только из-за количества клиентов.

### 25.3.7. Client research не меняет identity backend Frame

Исследование клиента может обнаружить:

```text
новое использование
новую зависимость
новый контракт
новую семантическую интерпретацию
```

но не должно автоматически изменять identity Core Backend Frame.

Например:

```text
Frontend → Authentication
```

не означает:

```text
Frontend OWNS Authentication
```

Ownership является отдельным Claim и требует собственного Evidence.

### 25.3.8. Последствие для Platform Candidate

При исследовании Platform Candidate следует отдельно проверять:

```text
1. Backend capability существует.
2. Capability используется одним или несколькими клиентами.
3. Граница capability и её зависимости поняты.
4. Использование несколькими клиентами подтверждено независимыми Evidence.
```

Наличие нескольких клиентов является Evidence переиспользуемости, но не является самостоятельным Decision Gate.

---

## 25.4. SourceFact — физически воспроизводимое наблюдение

`SourceFact` — минимальное наблюдение непосредственно над физическим источником.

SourceFact **не является интерпретацией**.

Правильно:

```text
models/api.php contains method selectCart()
```

Неправильно:

```text
Cart is a reusable platform
```

Второе является Semantic Claim.

### Обязательный provenance

Каждый SourceFact обязан содержать ссылку на физический источник, достаточную для независимой повторной проверки.

`source_location` должен быть конкретным для типа источника.

### CODE

```text
repository
revision / commit
file
symbol
line_start
line_end
```

### DB_SCHEMA

```text
database
schema
object_type
object_name
column / constraint
DDL revision or extraction timestamp
```

### DB_DATA

```text
database
schema
table
row_identity / key
column
value reference
snapshot / extraction timestamp
```

Для чувствительных значений допускается хранить ссылку, hash или нормализованный representation вместо полного значения, если это не мешает проверке утверждения.

### API_CONTRACT

```text
contract
version
method
path
operation
section / definition
```

### TEST

```text
repository
revision
file
test_name
line_start
line_end
```

### DOCUMENTATION

```text
document
revision
section
page / anchor
```

### RUNTIME

```text
environment
execution_id
timestamp
observed_operation
correlation_id
```

### CONFIGURATION

```text
source
revision / snapshot
key
location
```

---

## 25.5. SourceFact quality gate

SourceFact принимается в граф только если другой исследователь или автоматический validator может ответить:

> Где именно в исходной системе было сделано это наблюдение?

Если ответом является только:

```text
backend code
database
documentation
```

без конкретного location, SourceFact считается недостаточно воспроизводимым.

Такой факт может оставаться рабочим observation, но не должен быть единственным Evidence для `CONFIRMED` claim.

---

## 25.6. Evidence

Evidence связывает один или несколько SourceFacts с Semantic Claim.

```text
SourceFact
    ↓
Evidence
    ↓
Claim
```

Evidence отвечает на вопрос:

> Почему это наблюдение является основанием для данного утверждения?

Evidence не должно добавлять скрытую семантику.

Например:

```text
SourceFact:
models/api.php::selectCart()

Evidence:
CODE_USAGE
```

может подтверждать:

```text
Claim:
Cart capability exists
```

но само по себе не подтверждает:

```text
Claim:
Cart is reusable across products
```

---

## 25.7. Evidence provenance

Каждое Evidence обязано сохранять:

```text
source_fact_ids
type
strength_category
independent_group
```

Допускается:

```text
1 Evidence → N Claims
1 Claim    → N Evidence
```

Одно физическое наблюдение может подтверждать несколько claims.

Однако claims не становятся независимыми только потому, что используют разные Evidence, если Evidence происходят из одной `independent_group`.

---

## 25.8. Independent Evidence

Независимость определяется не количеством записей, а независимостью основания наблюдения.

Примеры:

```text
CODE_USAGE
DB_SCHEMA
API_CONTRACT
```

могут быть независимыми Evidence.

Но:

```text
README
generated documentation
LLM summary of README
```

не являются тремя независимыми Evidence.

### Конвенция

Если Evidence единственное в своей группе:

```text
independent_group = evidence.id
```

Если несколько Evidence основаны на одном источниковом основании:

```text
independent_group = common_group_id
```

`UNGROUPED` не используется.

---

## 25.9. SemanticClaim

Только после фиксации SourceFact и Evidence создаётся Semantic Claim.

Claim должен отвечать на вопрос:

> Что именно мы утверждаем о системе?

Claim обязан иметь:

```text
subject
predicate
object OR value
confidence
evidence_ids
```

`object` и `value` взаимоисключающие.

Пример:

```text
subject = Cart
predicate = PERSISTS
object = cart
```

или:

```text
subject = session_token_duration
predicate = HAS_VALUE
value = 3600
```

---

## 25.10. Confidence

Confidence относится прежде всего к Claim.

Допустимые значения:

```text
CONFIRMED
INFERRED
UNKNOWN
```

### CONFIRMED

Claim является `CONFIRMED`, если:

1. существует воспроизводимый SourceFact;
2. Evidence действительно поддерживает Claim;
3. нет unresolved conflict, делающего Claim недостоверным;
4. выполнено одно из условий:
   - есть несколько независимых подтверждений;
   - либо есть одно прямое Evidence достаточной природы для данного типа утверждения;
   - либо человек явно подтвердил Claim после review.

### INFERRED

Claim является `INFERRED`, если он логически следует из доступных Evidence, но прямого подтверждения недостаточно.

### UNKNOWN

`UNKNOWN` означает, что вопрос исследован, но имеющихся источников недостаточно для подтверждения или опровержения.

`UNKNOWN` не означает:

```text
"мы ещё не посмотрели"
```

Для такого случая используется открытый research question.

---

## 25.11. LLM и confidence

LLM может:

- извлекать SourceFacts;
- предлагать Evidence;
- предлагать Claims;
- предлагать повышение confidence;
- обнаруживать конфликты;
- формировать следующий research question.

LLM не может самостоятельно сделать Claim `CONFIRMED`.

Если LLM предлагает изменение confidence, сохраняется audit trail:

```text
proposed_confidence
proposed_by
proposed_at
proposal_reason
```

После human review:

```text
human_confirmed_confidence
reviewed_by
reviewed_at
review_reference
```

Если review отсутствует, предложение LLM остаётся предложением.

---

## 25.12. Human Review

Human review обязателен для:

```text
CONFIRMED
```

в случаях, когда подтверждение не является однозначным прямым структурным фактом.

Human review также обязателен для:

```text
PlatformCandidate
architectural boundary
conflict resolution
critical UNKNOWN resolution
```

Человек не должен подтверждать Claim только на основании формулировки LLM.

Review должен проверять:

```text
SourceFact
Evidence
Claim
confidence
alternative interpretations
```

---

## 25.13. Semantic Frame construction

Frame создаётся после того, как существуют Claims, описывающие его identity и существенные свойства.

Минимальная последовательность:

```text
SourceFact
   ↓
Evidence
   ↓
Identity Claim
   ↓
Frame
```

Identity Claim:

```text
subject = frame.id
predicate = EXISTS_AS
object = frame.type
```

Frame confidence обязан совпадать с confidence identity claim:

```text
frame.confidence
    =
claims[frame.identity_claim_id].confidence
```

UNKNOWN slots не понижают identity confidence Frame.

---

## 25.14. Slot construction

Slot является структурированным представлением Claim, относящегося к Frame.

```text
Frame
 └── Slot
       └── Claim
             └── Evidence
                   └── SourceFact
```

Если Slot содержит:

```text
target_frame
```

то Claim должен иметь соответствующий `object`.

Если Slot содержит:

```text
value
```

то Claim должен иметь соответствующий `value`.

Нельзя заполнять Slot только потому, что такое поле кажется логичным для данного Frame.

---

## 25.15. Relation construction

Relation также является семантическим утверждением.

Обязательная цепочка:

```text
SourceFact
   ↓
Evidence
   ↓
Claim
   ↓
Relation
```

Для Relation должны выполняться:

```text
claim.subject  = relation.source_frame
claim.object   = relation.target_frame
claim.predicate = relation.type
```

Следовательно, Relation не является самостоятельным источником знания.

Она является машиночитаемым представлением подтверждённого Semantic Claim.

---

## 25.16. Capability extraction

Capability формируется после обнаружения устойчивого функционального поведения.

Нельзя выводить Capability только из:

```text
class name
service name
table name
endpoint name
```

Необходимо найти совокупность Evidence, показывающую, что система действительно выполняет определённую функцию.

Минимальная цепочка:

```text
Code / DB / API / Tests
        ↓
SourceFacts
        ↓
Evidence
        ↓
Claims
        ↓
Frames / Relations
        ↓
Capability
```

Capability должна позволять ответить:

```text
what it does
who/what implements it
what it exposes
what data it persists
what it uses
what it depends on
how it is configured
```

---

## 25.17. Cross-source validation

После первоначального извлечения выполняется cross-source validation.

Минимальный принцип:

```text
CODE
  ↕
DB_SCHEMA
  ↕
DB_DATA
  ↕
API
  ↕
TEST
```

Источники не имеют глобального приоритета.

При этом для конкретного типа утверждения может существовать естественный первичный источник:

| Утверждение | Первичный источник |
|---|---|
| existence of table/column/FK | DB_SCHEMA |
| actual code behavior | CODE |
| public endpoint contract | API_CONTRACT |
| observed runtime behavior | RUNTIME |
| test behavior | TEST |
| configuration value | DB_DATA / CONFIGURATION |
| documented intended behavior | DOCUMENTATION |

Это не означает, что первичный источник автоматически «побеждает» другие.

Если источники противоречат друг другу, создаётся Conflict.

---

## 25.18. Conflict handling

При конфликте нельзя молча выбирать одну версию.

Сохраняются:

```text
Evidence A
Evidence B
Conflict
```

После review возможны:

```text
RESOLVED
ACCEPTED_AS_UNCERTAINTY
```

Если конфликт не разрешён, затронутый Claim не должен повышаться до `CONFIRMED`.

---

## 25.19. Configuration and site_constant

Configuration исследуется как семантический объект, а не только как таблица.

Для `site_constant` применяется отдельная последовательность:

```text
DB_DATA
   ↓
SourceFact
   ↓
Evidence
   ↓
Configuration Claim
   ↓
CONFIGURES Relation
   ↓
Capability / Domain
```

Само наличие строки в `site_constant` подтверждает существование configuration fact.

Но утверждение:

```text
site_constant.X CONFIGURES Capability Y
```

требует Evidence из кода, документации или runtime, показывающего использование `X` в контексте `Y`.

Таким образом, исследование `site_constant` должно иметь две независимые части:

```text
1. What values exist?
2. Where and how are they used?
```

---

## 25.20. Research Pass operational procedure

Один Research Pass выполняется следующим образом.

### Step 1 — Freeze Scope

Зафиксировать:

```text
ResearchScope
source revisions
questions
```

### Step 2 — Inventory

Составить список физических источников.

### Step 3 — Extract SourceFacts

Извлечь только наблюдаемые факты.

Каждый факт сразу получает точный provenance.

### Step 4 — Build Evidence

Связать SourceFacts с потенциальными утверждениями.

### Step 5 — Create Claims

Сформировать минимальные Semantic Claims.

### Step 6 — Validate

Проверить Claims по независимым источникам.

### Step 7 — Resolve / record conflicts

Не скрывать противоречия.

### Step 8 — Build Frames / Relations

Создать семантические структуры только из подтверждённых или явно маркированных inferred claims.

### Step 9 — Extract Capabilities

Объединить устойчивые функциональные Claims в Capability.

### Step 10 — Identify Platform Candidates

Только после исследования Capability и зависимостей.

### Step 11 — Publish Snapshot

Создать новый GraphSnapshot.

### Step 12 — Publish Gap Report

Обязательно зафиксировать:

```text
confirmed_findings
inferred_findings
unknowns
open_conflicts
next_actions
```

---

## 25.21. Research Pass completion criterion

Research Pass считается завершённым, когда для каждого вопроса Scope существует один из результатов:

```text
CONFIRMED
INFERRED
UNKNOWN
CONFLICT
```

и для каждого результата существует provenance до физических источников.

Нельзя завершить Pass формулировкой:

```text
"исследовано"
```

без возможности пройти цепочку:

```text
Conclusion
   ↓
Claim
   ↓
Evidence
   ↓
SourceFact
   ↓
Physical Source
```

---

## 25.22. Provenance completeness gate

Перед публикацией Snapshot выполняется обязательная проверка:

### Для каждого CONFIRMED Claim

```text
Claim
 ↓
≥1 Evidence
 ↓
≥1 SourceFact
 ↓
concrete physical location
```

### Для каждого INFERRED Claim

```text
Claim
 ↓
Evidence
 ↓
SourceFact
 ↓
physical location
```

и дополнительно фиксируется причина inference.

### Для каждого UNKNOWN

должно существовать:

```text
question
searched_sources
reason_insufficient
next_action
```

### Для каждого Conflict

должно существовать:

```text
claim_ids
evidence_ids
resolution status
```

Snapshot, не проходящий Provenance Completeness Gate, не публикуется как исследовательский результат.

---

## 25.23. Auditability

Исследование должно быть воспроизводимым другим исследователем без участия автора первоначального Pass.

Для существенного вывода должно быть возможно:

```text
открыть Claim
→ перейти к Evidence
→ перейти к SourceFact
→ открыть исходный файл / DDL / строку данных / endpoint / test
→ повторить проверку
```

Если физический источник изменился, provenance должен указывать состояние источника, использованное в исходном Pass:

```text
commit
snapshot
revision
timestamp
```

---

## 25.24. Versioning

Используется модель:

```text
Immutable Evidence
        +
Research Pass
        +
Graph Snapshot
```

Новый Pass не переписывает историю предыдущего Pass.

Если новое исследование опровергает старое утверждение:

```text
old Claim → SUPERSEDED
new Claim → ACTIVE
```

при этом обе версии остаются доступными через provenance.

Snapshot связывается с:

```text
parent_snapshot_id
pass_id
```

и тем самым сохраняет историю исследования.

---

## 25.25. Platform Candidate Decision

Platform Candidate появляется только после исследования соответствующего Capability subgraph.

Candidate должен иметь:

```text
based_on
included_capabilities
dependencies
external_dependencies
boundary_questions
unresolved_questions
critical_unknowns
preconditions
status
```

Сначала проверяются восемь graph-checkable preconditions.

Только после:

```text
PRECONDITIONS = PASSED
```

Candidate может быть вынесен на architectural review.

`APPROVED_AS_PLATFORM` является только результатом внешнего human decision и требует:

```text
approved_by
approval_reference
approved_at
```

LLM не может установить этот статус самостоятельно.

---

## 25.26. Gap Report

Gap Report является обязательным артефактом каждого завершённого Pass.

Он должен показывать:

```text
What we know
What we infer
What we do not know
What conflicts
What must be investigated next
```

Минимальная структура:

```text
confirmed_findings
inferred_findings
unknowns
open_conflicts
next_actions
```

Gap Report не является «списком недоделок». Он является картой границ текущего знания.

---

## 25.27. Transition to full Semantic Graph

Протоструктуры считаются готовыми к импорту в полноценный Semantic Graph только если сохраняется вся provenance chain:

```text
Graph Node / Relation
       ↓
Semantic Claim
       ↓
Evidence
       ↓
SourceFact
       ↓
Physical Source
```

Импорт не должен терять:

- stable IDs;
- confidence;
- provenance;
- Research Pass;
- Snapshot;
- conflicts;
- UNKNOWN;
- superseding history.

Если после импорта невозможно восстановить происхождение существенного утверждения, импорт считается неполным.

---

## 25.28. Ключевой принцип методики

Мы не начинаем с вопроса:

> «Какую платформу построить?»

Мы начинаем с вопроса:

> «Что фактически существует в системе и какие семантические утверждения об этом можно доказать?»

Поэтому правильная последовательность:

```text
Existing System
      ↓
Physical Sources
      ↓
Reproducible SourceFacts
      ↓
Evidence
      ↓
Semantic Claims
      ↓
Semantic Frames / Relations
      ↓
Capabilities
      ↓
Dependencies
      ↓
Platform Candidates
      ↓
Human Architectural Decision
```

**Платформа является результатом исследования, а не исходной предпосылкой исследования.**


## Appendix A. Machine-readable JSON Schema

Ниже находится **единственная нормативная JSON Schema** документа. Разделы выше определяют семантику полей и правил; эта схема определяет их машинное представление.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "semantic-graph-prototype-structures-v2.2.schema.json",
  "title": "Semantic Graph Prototype Structures v2.2",
  "type": "object",
  "required": [
    "schema_version",
    "snapshot",
    "research_pass",
    "research_scope",
    "source_facts",
    "evidence",
    "claims",
    "frames",
    "relations",
    "capabilities",
    "candidates",
    "conflicts",
    "gap_report"
  ],
  "properties": {
    "schema_version": {
      "type": "string"
    },
    "snapshot": {
      "$ref": "#/$defs/graphSnapshot"
    },
    "research_pass": {
      "$ref": "#/$defs/researchPass"
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
    },
    "research_scope": {
      "$ref": "#/$defs/researchScope"
    },
    "gap_report": {
      "$ref": "#/$defs/gapReport"
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
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false
    },
    "evidence": {
      "type": "object",
      "required": [
        "id",
        "type",
        "source_fact_ids",
        "strength_category",
        "independent_group"
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
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false
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
            "null",
            "object",
            "array"
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
        },
        "proposed_by": {
          "type": "string"
        },
        "proposed_at": {
          "type": "string",
          "format": "date-time"
        },
        "proposal_reason": {
          "type": "string"
        },
        "human_confirmed_confidence": {
          "$ref": "#/$defs/confidence"
        },
        "reviewed_by": {
          "type": "string"
        },
        "reviewed_at": {
          "type": "string",
          "format": "date-time"
        },
        "review_reference": {
          "type": "string"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false,
      "oneOf": [
        {
          "required": [
            "object"
          ],
          "not": {
            "required": [
              "value"
            ]
          }
        },
        {
          "required": [
            "value"
          ],
          "not": {
            "required": [
              "object"
            ]
          }
        }
      ]
    },
    "slot": {
      "type": "object",
      "required": [
        "name",
        "claim_id",
        "confidence"
      ],
      "properties": {
        "name": {
          "type": "string"
        },
        "target_frame": {
          "type": "string"
        },
        "value": {
          "type": [
            "string",
            "number",
            "boolean",
            "null",
            "object",
            "array"
          ]
        },
        "relation": {
          "type": "string"
        },
        "claim_id": {
          "type": "string"
        },
        "confidence": {
          "$ref": "#/$defs/confidence"
        }
      },
      "additionalProperties": false,
      "oneOf": [
        {
          "required": [
            "target_frame"
          ],
          "not": {
            "required": [
              "value"
            ]
          }
        },
        {
          "required": [
            "value"
          ],
          "not": {
            "required": [
              "target_frame"
            ]
          }
        }
      ]
    },
    "frame": {
      "type": "object",
      "required": [
        "id",
        "type",
        "name",
        "identity_claim_id",
        "slots",
        "implementation",
        "persistence",
        "configuration",
        "evidence_ids",
        "confidence",
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
            "FSM"
          ]
        },
        "name": {
          "type": "string"
        },
        "identity_claim_id": {
          "type": "string"
        },
        "slots": {
          "type": "array",
          "items": {
            "$ref": "#/$defs/slot"
          }
        },
        "implementation": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "persistence": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "configuration": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "evidence_ids": {
          "type": "array",
          "items": {
            "type": "string"
          }
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
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false,
      "description": "frame.confidence is a denormalized copy of the confidence of claims[identity_claim_id]. The values must be equal."
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
          "type": "string",
          "enum": [
            "FK",
            "REFERENCES",
            "PERSISTS",
            "CALLS",
            "USES",
            "READS",
            "WRITES",
            "TRIGGERS",
            "PRODUCES",
            "CONSUMES",
            "OWNS",
            "BELONGS_TO",
            "DEPENDS_ON",
            "CONFIGURES",
            "IMPLEMENTS",
            "EXPOSES",
            "PROVIDES"
          ]
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
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false
    },
    "capability": {
      "type": "object",
      "required": [
        "id",
        "frame_id",
        "implemented_by",
        "exposes",
        "persists",
        "uses",
        "configured_by",
        "dependencies",
        "evidence_ids",
        "confidence",
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
        "exposes": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "persists": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "uses": {
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
        "dependencies": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "evidence_ids": {
          "type": "array",
          "items": {
            "type": "string"
          }
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
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false
    },
    "preconditions": {
      "type": "object",
      "required": [
        "capability_researched",
        "boundary_defined",
        "dependencies_defined",
        "data_and_integrations_defined",
        "critical_unknowns_resolved",
        "ownership_boundary_resolved",
        "public_contract_formulable",
        "migration_impact_researched"
      ],
      "properties": {
        "capability_researched": {
          "$ref": "#/$defs/preconditionState"
        },
        "boundary_defined": {
          "$ref": "#/$defs/preconditionState"
        },
        "dependencies_defined": {
          "$ref": "#/$defs/preconditionState"
        },
        "data_and_integrations_defined": {
          "$ref": "#/$defs/preconditionState"
        },
        "critical_unknowns_resolved": {
          "$ref": "#/$defs/preconditionState"
        },
        "ownership_boundary_resolved": {
          "$ref": "#/$defs/preconditionState"
        },
        "public_contract_formulable": {
          "$ref": "#/$defs/preconditionState"
        },
        "migration_impact_researched": {
          "$ref": "#/$defs/preconditionState"
        }
      },
      "additionalProperties": false
    },
    "preconditionState": {
      "type": "string",
      "enum": [
        "PASSED",
        "FAILED",
        "UNKNOWN"
      ]
    },
    "candidate": {
      "type": "object",
      "required": [
        "id",
        "name",
        "based_on",
        "included_capabilities",
        "dependencies",
        "external_dependencies",
        "boundary_questions",
        "unresolved_questions",
        "critical_unknowns",
        "preconditions",
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
        "preconditions": {
          "$ref": "#/$defs/preconditions"
        },
        "status": {
          "type": "string",
          "enum": [
            "RESEARCHING",
            "MATURE_FOR_REVIEW",
            "REJECTED",
            "APPROVED_AS_PLATFORM"
          ]
        },
        "approved_by": {
          "type": "string"
        },
        "approval_reference": {
          "type": "string"
        },
        "approved_at": {
          "type": "string",
          "format": "date-time"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "allOf": [
        {
          "if": {
            "properties": {
              "status": {
                "const": "APPROVED_AS_PLATFORM"
              }
            }
          },
          "then": {
            "required": [
              "approved_by",
              "approval_reference",
              "approved_at"
            ]
          }
        }
      ],
      "additionalProperties": false
    },
    "researchPass": {
      "type": "object",
      "required": [
        "pass_id",
        "scope_id",
        "started_at",
        "status"
      ],
      "properties": {
        "pass_id": {
          "type": "string"
        },
        "scope": {
          "type": "string"
        },
        "started_at": {
          "type": "string",
          "format": "date-time"
        },
        "completed_at": {
          "type": "string",
          "format": "date-time"
        },
        "status": {
          "type": "string",
          "enum": [
            "OPEN",
            "COMPLETED",
            "ABANDONED"
          ]
        },
        "scope_id": {
          "type": "string"
        },
        "gap_report_id": {
          "type": "string"
        }
      },
      "additionalProperties": false
    },
    "graphSnapshot": {
      "type": "object",
      "required": [
        "snapshot_id",
        "schema_version",
        "created_at",
        "pass_id",
        "parent_snapshot_id"
      ],
      "properties": {
        "snapshot_id": {
          "type": "string"
        },
        "schema_version": {
          "type": "string"
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "pass_id": {
          "type": "string"
        },
        "parent_snapshot_id": {
          "type": [
            "string",
            "null"
          ]
        }
      },
      "additionalProperties": false
    },
    "conflict": {
      "type": "object",
      "required": [
        "id",
        "claim_ids",
        "evidence_ids",
        "status"
      ],
      "properties": {
        "id": {
          "type": "string"
        },
        "evidence_ids": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "minItems": 2
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
        },
        "claim_ids": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "minItems": 1
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "additionalProperties": false
    },
    "researchScope": {
      "type": "object",
      "required": [
        "scope_id",
        "description",
        "source_boundaries",
        "questions"
      ],
      "properties": {
        "scope_id": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "source_boundaries": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "questions": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "additionalProperties": false
    },
    "gapReport": {
      "type": "object",
      "required": [
        "gap_report_id",
        "pass_id",
        "confirmed_findings",
        "inferred_findings",
        "unknowns",
        "open_conflicts",
        "next_actions"
      ],
      "properties": {
        "gap_report_id": {
          "type": "string"
        },
        "pass_id": {
          "type": "string"
        },
        "confirmed_findings": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "inferred_findings": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "unknowns": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "open_conflicts": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "next_actions": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "minItems": 1
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```
