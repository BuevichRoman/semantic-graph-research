# Semantic Graph Research Methodology v2.3

**Статус:** нормативная методология исследования существующих систем.  
**Основание:** v2.2 + результаты практических Research Pass и исследования `/location`.  
**Назначение:** воспроизводимое доказательное восстановление семантики существующей системы и управление дальнейшим исследованием через Semantic Graph.

---

## 1. Назначение

Методология применяется до проектирования новой архитектуры.

Основная последовательность:

```text
Existing System
      ↓
Physical Sources
      ↓
Source Facts
      ↓
Evidence
      ↓
Semantic Claims
      ↓
Semantic Frames / Relations
      ↓
Capabilities
      ↓
Dependencies / Boundaries
      ↓
Platform Candidates
      ↓
Human Architectural Decision
```

Главный принцип:

> Платформа является результатом исследования существующей системы, а не исходной границей исследования.

Методология не является методологией рефакторинга и не предполагает, что существующие классы, endpoint'ы или таблицы совпадают с будущими доменными границами.

---

## 2. Evidence First

Исследование начинается с физического источника.

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
```

Запрещён прямой переход:

```text
DB table → Domain
Class OrderService → Order Platform
```

Каждый существенный семантический вывод должен иметь воспроизводимую provenance chain.

---

## 3. Источники истины

Приоритет:

1. фактический код;
2. database schema / SQL / stored procedures;
3. API contracts;
4. configuration;
5. tests;
6. runtime evidence;
7. documentation;
8. названия, комментарии и conventions — только как вспомогательные признаки.

При противоречии источников конфликт фиксируется. Документация не переписывает код молча, код не уничтожает документацию молча.

---

## 4. Source Snapshot

Каждый Research Pass работает с конкретными версиями источников.

Например:

```text
Core Backend S0
DB S0
Taxi Web v0.1.20
```

Frontend, backend и DB могут иметь разные версии.

Для общего Core Backend это особенно важно:

```text
Core Backend
   ↑
   ├── Client A
   ├── Client B
   └── Client C
```

Client-specific поведение не становится автоматически свойством Core Backend.

---

## 5. Source Fact

`SourceFact` — минимальное наблюдение непосредственно из источника.

Пример:

```text
Source: models/api.php
Observation: function X executes SELECT ... FROM users_location
```

SourceFact не должен содержать вывод:

```text
Driver uses location
```

если этого прямо не наблюдается.

SourceFact отвечает:

> Что физически обнаружено?

---

## 6. Evidence

`Evidence` связывает Semantic Claim с одним или несколькими SourceFact.

Возможные типы:

```text
FOREIGN_KEY
TABLE_SCHEMA
SQL_USAGE
CODE_USAGE
API_CONTRACT
TEST
RUNTIME
DOCUMENTATION
CONFIGURATION
```

Evidence не является самим Claim.

Один Evidence может поддерживать несколько Claims, если это действительно один источник подтверждения.

---

## 7. Независимость Evidence

`independent_group` предотвращает двойной подсчёт нескольких наблюдений одного происхождения.

Для одиночного Evidence:

```text
independent_group = evidence_id
```

Наблюдения одного метода или одного автоматически сгенерированного документа не становятся независимыми только потому, что записаны раздельно.

---

## 8. Semantic Claim

`SemanticClaim` — утверждение исследователя о семантике системы.

Минимально:

```text
subject
predicate
object
```

или:

```text
subject
predicate
value
```

`object` и `value` взаимоисключающие.

Claim сохраняет:

```text
evidence_ids
confidence
status
provenance
```

Claim отвечает:

> Что мы утверждаем о системе?

---

## 9. Confidence

Допустимые уровни:

```text
CONFIRMED
INFERRED
UNKNOWN
```

**CONFIRMED** — подтверждено допустимым Evidence.

**INFERRED** — обоснованный вывод, для которого недостаточно прямого подтверждения.

**UNKNOWN** — утверждение не установлено.

UNKNOWN является допустимым состоянием.

---

## 9.1. Три различных набора статусов

Слово «статус» в корпусе относится к трём разным измерениям. Они не являются
синонимами и не должны смешиваться в одном поле.

### Claim Confidence

Относится к **отдельному Semantic Claim** и отвечает на вопрос, насколько
утверждение о системе установлено:

```text
CONFIRMED
INFERRED
UNKNOWN
```

Определения — раздел 9. Разновидности UNKNOWN — раздел 10.

### Research Result

Относится к **Research Pass** и отвечает на вопрос, чем закончился проход
по своему Research Question:

```text
CONFIRMED
REJECTED
SOURCE_GAP
```

**CONFIRMED** — Research Question закрыт положительно, Claim подтверждён Evidence.

**REJECTED** — Research Question закрыт отрицательно: проверяемое утверждение
опровергнуто (раздел 38).

**SOURCE_GAP** — проход завершён, но вопрос не закрыт, потому что необходимый
источник отсутствует в доступном snapshot.

`SOURCE_GAP` как Research Result означает исход прохода. Он не является
значением Claim Confidence: соответствующий Claim при этом остаётся
`UNKNOWN / SOURCE_GAP` (раздел 10).

### Document Status

Относится к **документу корпуса** и отвечает на вопрос, в каком состоянии
находится сам текст:

```text
PROVISIONAL
COMPLETED
RECONCILIATION
HISTORICAL
```

**PROVISIONAL** — документ отражает незавершённое состояние работы.

**COMPLETED** — работа по документу завершена.

**RECONCILIATION** — документ сводит между собой результаты других проходов.

**HISTORICAL** — документ описывает более раннюю стадию и не является
нормативным для текущей версии методологии.

### Правило

Три набора не пересекаются по смыслу и не переиспользуют значения друг друга.

Запись вида

```text
**Статус:** CONFIRMED
```

в шапке документа недопустима, если имеется в виду результат исследования или
confidence отдельного Claim: Document Status описывает документ, а не
утверждение и не проход.

---

## 10. Два вида UNKNOWN

Практическое исследование показало необходимость различать:

### UNKNOWN / SOURCE_GAP

Не хватает физического источника.

Например:

```text
selectLocation()
```

известен по call site, но его тело отсутствует в доступном source snapshot.

### UNKNOWN / BEHAVIOR_UNRESOLVED

Источник исследован, но он не устанавливает требуемое поведение.

Например:

```text
Driver Selection → User Position
```

если соответствующий код исследован, но связь не доказана.

Эти состояния нельзя смешивать.

---

## 11. UNKNOWN → Research Question

Каждый существенный UNKNOWN должен по возможности порождать конкретный Research Question.

Например:

```text
RQ:
Does GET /location?u_id=B allow User A
to obtain User B's current position?
```

Research Question описывает, **что необходимо выяснить**, а не является Claim о системе.

---

## 12. Research Question

Минимально:

```text
id
question
origin_claim_ids
expected_evidence
search_plan
status
result_claim_ids
```

Статусы:

```text
OPEN
IN_PROGRESS
ANSWERED
REJECTED
SOURCE_GAP
```

---

## 13. Research Loop

Основной цикл v2.3:

```text
UNKNOWN
   ↓
Research Question
   ↓
Expected Evidence
   ↓
Search Plan
   ↓
Source
   ↓
Source Fact
   ↓
Evidence
   ↓
Research Result
   ├── CONFIRMED
   ├── INFERRED
   ├── REJECTED
   └── SOURCE_GAP
```

Если предполагаемая связь опровергнута достаточным Evidence, результатом является `REJECTED`, а не UNKNOWN.

---

## 14. Search Plan

Search Plan описывает, где и что искать.

Пример:

```text
RQ:
Does Driver Selection use users_location?

Search Plan:
1. inspect Driver Selection call chain;
2. inspect SQL reads/writes of users_location;
3. inspect position helpers;
4. inspect FSM transition inputs;
5. inspect frontend/API path.
```

Это позволяет следующему исследователю продолжить работу без восстановления всего контекста.

---

## 15. Frame

`Frame` представляет семантически самостоятельный объект.

Критерий:

> Frame способен участвовать хотя бы в одной независимой семантической связи.

Если значение не имеет самостоятельной роли, оно остаётся Slot/value.

Пример:

```text
User
    HAS_CURRENT_POSITION
Position
```

---

## 16. Domain Role ≠ фундаментальный субъект

Например:

```text
User
```

может выступать как:

```text
Passenger
Driver
```

в разных контекстах.

Поэтому:

```text
Passenger = User in order context
Driver    = User in performer context
```

не означает автоматически два разных фундаментальных объекта.

---

## 17. Relation

Relation — типизированная семантическая связь между Frames.

Не использовать универсальный `USES`, если известна более точная семантика:

```text
HAS_CURRENT_POSITION
READS_POSITION
OBSERVES_POSITION
PERSISTS
PRODUCES
EXPOSES
CONFIGURED_BY
DEPENDS_ON
```

Связь:

```text
A USES B
```

не позволяет автоматически вывести:

```text
A READS B
```

или:

```text
A PERSISTS B
```

---

## 18. Relation provenance

Каждая существенная Relation должна иметь provenance через Claim.

Например:

```text
User
 ──HAS_CURRENT_POSITION──>
Position

confidence: CONFIRMED
evidence: DB_SCHEMA + CODE_USAGE
```

против:

```text
Driver Selection
 ──DEPENDS_ON──>
User Position

confidence: INFERRED
```

Это разные уровни утверждения.

---

## 19. Capability

Capability описывает функциональную способность системы.

Она не равна автоматически:

```text
class
endpoint
service
table
feature
```

Например:

```text
Location Exposure
```

может быть реализована через:

```text
API
Service
DB
Frontend
External integration
```

---

## 20. Capability Dependency ≠ Code Usage

Если:

```text
Capability A
    DEPENDS_ON
Concept B
```

это не означает:

```text
implementation A
    READS
table B
```

и не означает:

```text
code A
    CALLS
function B
```

Каждый уровень должен иметь собственное Evidence.

---

## 21. Semantic Synthesis

Не все важные понятия существуют буквально в коде.

Например:

```text
Direct Order
Vote Order
Order Execution Mode
```

могут быть восстановлены из совокупности:

```text
API
FSM
DB
configuration
frontend behavior
documentation
```

Поэтому существует отдельный процесс:

```text
Source Facts
    ↓
Claims
    ↓
Semantic Synthesis
    ↓
Synthesized Frame
```

---

## 22. Semantic Synthesis не создаёт Evidence

Semantic Synthesis является актом интерпретации.

Он может предложить новый Frame, но не создаёт:

```text
SourceFact
Evidence
```

Пример:

```text
Synthesis S-017

proposed_frame:
    Vote Order

input_claims:
    C1
    C2
    C3
    C4

confidence:
    INFERRED
```

Новый Frame обязан сохранять ссылки на Claims, из которых он синтезирован.

---

## 23. Source-derived / Reconstructed / Synthesized

Следует различать три происхождения:

### Source-derived

Непосредственно соответствует объекту/структуре источника:

```text
User
Order
users_code
```

### Reconstructed

Однозначно восстанавливается из нескольких источников, хотя не существует как отдельный объект:

```text
Authentication Session
```

### Synthesized

Более высокий уровень абстракции, созданный из нескольких Claims:

```text
Vote Order
Direct Order
Order Execution Mode
```

Все три могут быть Frames, но provenance различается.

---

## 23.1. Lifecycle Semantic Synthesis

Происхождение из раздела 23 задаёт нормативную последовательность:

```text
Source-derived
↓
Reconstructed
↓
Synthesized
↓
validation by additional evidence
↓
CONFIRMED / REJECTED
```

Правило:

> `Synthesized` не означает `CONFIRMED`. Синтезированное понятие становится
> подтверждённым только после появления дополнительного Evidence.

До появления такого Evidence синтезированный Frame остаётся на уровне
`INFERRED` (раздел 9) и не может использоваться как подтверждённое основание
для Platform Decision Gate (раздел 45).

Валидация выполняется отдельным Research Pass, результат которого выражается
в терминах Research Result (раздел 9.1): `CONFIRMED`, `REJECTED` или
`SOURCE_GAP`.

---

## 24. Frame Reification

Если объект имеет несколько независимых отношений, он может быть reified как Frame.

Например:

```text
Transition
    ├── FROM_STATE
    ├── ACTION
    └── TO_STATE
```

Reification — механизм, а не новый фундаментальный node type.

---

## 25. Reified Assertion

Для сложного утверждения, которое не выражается простой тройкой:

```text
subject
predicate
object/value
```

может использоваться Assertion Frame.

Подтверждённый шаблон:

```text
Assertion Frame
    ├── SCOPE
    ├── OPERATOR
    ├── PREDICATE
    └── VALUE
```

Он покрывает проверенные случаи:

```text
ALL members of S satisfy P
COUNT(S) == N
```

---

## 26. Assertion ≠ Rule Engine

Assertion хранит структуру утверждения.

Она не означает автоматически:

```text
execution
inference
evaluation
```

Поэтому:

```text
Semantic Graph
    ≠ FSM Engine
    ≠ Rule Engine
```

---

## 27. AS-IS и TO-BE

Это обязательное разделение.

### AS-IS

```text
что реально существует
```

### TO-BE

```text
что система должна делать
```

Например:

```text
AS-IS:
User → Current Position
```

против:

```text
TO-BE:
Active Order
 → Assigned User
 → Current Position
 → visible to order owner
```

TO-BE не должен попадать в AS-IS Semantic Graph как существующее поведение.

---

## 28. Requirement

Product requirement — не Claim о текущей системе.

Например:

```text
REQUIREMENT:
Client must see assigned User's current Position.
```

Requirement хранится в отдельном слое и может ссылаться на AS-IS:

```text
Requirement
    DEPENDS_ON
AS-IS Claim
```

или:

```text
Requirement
    GAP
AS-IS Capability
```

Requirement не изменяет confidence AS-IS.

---

## 29. Research Pass

Один Research Pass — ограниченный воспроизводимый цикл.

Минимальная последовательность:

```text
1. Scope
2. Source Snapshot
3. Extraction
4. Source Fact Validation
5. Evidence Linking
6. Semantic Interpretation
7. Semantic Synthesis
8. Frame / Relation construction
9. UNKNOWN analysis
10. Research Questions
11. Review
12. Graph update
13. Gap Report
```

Pass не обязан исследовать весь backend.

---

## 30. Scope

Scope явно определяет:

```text
что исследуем
какие источники
какие версии
какой frontend/client
какие вопросы
```

Например:

```text
Core Backend S0
+
Taxi Web v0.1.20
```

остаются различимыми источниками.

---

## 31. Code-first Research Pass

Для конкретного метода:

```text
method
 ↓
inputs
 ↓
branches
 ↓
called functions
 ↓
DB reads
 ↓
DB writes
 ↓
configuration
 ↓
FSM
 ↓
external calls
 ↓
return
```

После локального разбора scope расширяется только туда, куда ведёт Evidence.

Не требуется сразу строить полный call graph.

---

## 32. Frontend-specific Research Pass

Frontend является отдельным источником семантики.

Он может подтверждать:

```text
API usage
user-visible behavior
state interpretation
client-specific workflow
```

Но frontend конкретной версии не должен автоматически менять семантику общего backend.

Например:

```text
Taxi Web v0.1.20
    USES
Core Backend Location API
```

не означает:

```text
Core Backend Location API
    EXISTS ONLY FOR Taxi Web
```

---

## 33. Cross-system Research Pass

Для сценария:

```text
Frontend
    ↕
Backend
    ↕
DB
    ↕
External Service
```

каждый участок получает собственные SourceFacts.

Semantic Claim может связывать их, но provenance сохраняет всю цепочку.

---

## 34. Review и роль LLM

LLM может:

- извлекать SourceFacts;
- группировать наблюдения;
- предлагать Claims;
- предлагать Semantic Synthesis;
- предлагать повышение confidence;
- формировать Research Questions.

LLM не получает самостоятельного права:

- подтверждать архитектурные факты;
- создавать Evidence;
- скрывать Conflict;
- удалять UNKNOWN;
- превращать Candidate в Platform.

---

## 35. Повышение confidence

LLM может предложить:

```text
proposed_confidence = CONFIRMED
```

но окончательное:

```text
confidence = CONFIRMED
```

требует:

1. прямого допустимого Evidence;
2. либо нескольких независимых Evidence;
3. либо явного human review.

Proposal и final decision должны быть различимы в audit trail.

---

## 36. Semantic Synthesis Review

Особенно осторожно проверяются абстракции:

```text
Vote Order
Order Execution Mode
Shared Capability
Ownership
Boundary
```

Для них сохраняются:

```text
input_claim_ids
synthesis_reason
proposed_by
confidence
review status
```

«Логично выглядит» не является Evidence.

---

## 37. Conflict

Conflict фиксируется, если Evidence или Claims противоречат друг другу.

Состояния:

```text
OPEN
RESOLVED
ACCEPTED_AS_UNCERTAINTY
```

Конфликт не скрывается.

---

## 38. Rejected Claim

Если исследование достаточно подтверждает отсутствие предполагаемой связи:

```text
REJECTED
```

а не UNKNOWN.

Пример:

```text
Hypothesis:
Driver Selection reads users_location

Research:
call chain + SQL + helpers investigated

Result:
no such dependency
```

→ `REJECTED`.

Если физический источник отсутствует:

```text
UNKNOWN / SOURCE_GAP
```

---

## 39. Gap Report

Каждый завершённый Research Pass имеет Gap Report.

Для каждого gap:

```text
id
description
confidence
unknown_type
research_question_id
next_action
```

Gap Report является обязательным выходом Pass.

---

## 40. Lifecycle и superseding

SourceFacts и Evidence после публикации Pass не переписываются задним числом.

Если источник изменился:

```text
old SourceFact
    SUPERSEDED

new SourceFact
    ACTIVE
```

Graph Snapshots сохраняют историю:

```text
Snapshot v0
    ↓
Snapshot v0.1
    ↓
Snapshot v0.2
```

---

## 41. Provenance

Каждый существенный элемент должен позволять восстановить:

```text
Graph Frame / Relation
        ↓
Semantic Claim
        ↓
Evidence
        ↓
SourceFact
        ↓
Physical Source
        ↓
Source Snapshot
        ↓
Research Pass
```

Потеря provenance означает неполноту исследования.

---

## 42. Feature ≠ Capability

Feature/use case не превращается автоматически в Capability.

Например:

```text
Client sees driver's position on map
```

может включать:

```text
Assignment
User Position
Position Exposure
Map Rendering
```

Сначала восстанавливаются Claims/Capabilities, затем их связи.

---

## 43. Semantic Concept ≠ Implementation

Один semantic concept может иметь несколько implementation paths:

```text
Position
 ├── Core Backend Location API
 └── External Driver Position Pipeline
```

если Evidence подтверждает общий semantic concept.

Поэтому:

```text
semantic concept
    ≠
implementation
```

---

## 44. Platform Candidate

Platform Candidate является результатом исследования.

Нельзя создавать его только из названия:

```text
CartService
AuthService
PaymentService
```

До Candidate должны быть исследованы:

```text
capability
dependencies
ownership
data
external integrations
contracts
client usage
boundary questions
```

---

## 45. Platform Decision Gate

Graph-checkable preconditions:

```text
1. boundaries researched;
2. critical dependencies identified;
3. data and external integrations identified;
4. critical UNKNOWN resolved or explicitly accepted as risk;
5. ownership/boundary questions answered;
6. public contract can be stated without hidden dependencies;
7. impact on existing backend researched.
```

Отдельно существует:

```text
Human Sign-off
```

Архитектурное решение команды не является свойством графа.

`APPROVED_AS_PLATFORM` требует:

```text
approved_by
approval_reference
approved_at
```

---

## 46. Completion criteria

Research Pass завершён, если существенные вопросы:

```text
ответ подтверждён
```

или:

```text
UNKNOWN
+
Research Question
+
next action
```

или:

```text
Conflict зарегистрирован
```

и при этом:

```text
Gap Report создан
provenance сохранена
```

---

## 47. MCR

MCR (Model Change Record) используется для изменения языка/структуры Semantic Graph.

Модель изменения:

```text
Question
    ↓
Experiment
    ↓
Result
    ↓
Decision
    ↓
Model Change / No Change
```

Новое фундаментальное понятие нельзя вводить только потому, что оно удобно.

Правило:

> Новое понятие вводится только после того, как реальный граф не смог ответить на конкретный вопрос без него и альтернативные формулировки были исчерпаны.

---

## 48. Research Pass ≠ MCR

### Research Pass

Исследует существующую систему и производит:

```text
SourceFacts
Claims
Frames
Relations
Capabilities
Research Questions
```

### MCR

Проверяет достаточность самого языка Semantic Graph и может изменить:

```text
Graph Structure
Relation vocabulary
Representation mechanism
```

Не каждый UNKNOWN backend является MCR.

---

## 49. Кристаллизация

Протоструктуры являются промежуточным исследовательским слоем.

После MCR может быть получена первая кристаллизация:

```text
Frame
Relation
SemanticClaim
Evidence
```

с:

```text
Frame Reification
Reified Assertion
```

если они подтверждены экспериментами.

Кристаллизация не отменяет provenance.

---

## 50. Что не добавляется без MCR

Не вводить автоматически:

```text
Quantifier
Rule
LogicNode
ValueNode
Condition
ActorNode
```

как фундаментальные типы только потому, что они кажутся полезными.

Для нового фундаментального типа требуется конкретный expressive gap и MCR.

---

## 51. Практический алгоритм

```text
1. Fix Snapshot.
2. Define Scope.
3. Identify physical sources.
4. Extract SourceFacts.
5. Link Evidence.
6. Form Claims.
7. Separate CONFIRMED / INFERRED / UNKNOWN.
8. Classify UNKNOWN as SOURCE_GAP or BEHAVIOR_UNRESOLVED.
9. Create Research Questions.
10. Build Search Plans.
11. Continue until question is answered, rejected, or remains SOURCE_GAP.
12. Identify recurring semantic concepts.
13. Perform Semantic Synthesis.
14. Build Frames / Relations.
15. Identify Capabilities.
16. Identify dependencies and boundaries.
17. Produce Gap Report.
18. Review.
19. Publish immutable Research Pass.
```

---

## 52. Критерий качества

Качество исследования определяется не количеством CONFIRMED.

Хорошее исследование должно обеспечивать:

```text
CONFIRMED
    действительно доказано;

INFERRED
    действительно является выводом;

UNKNOWN
    точно локализован;

REJECTED
    имеет достаточное отрицательное Evidence;

SYNTHESIZED
    имеет input Claims;

TO-BE
    не смешан с AS-IS.
```

---

## 53. Итоговая архитектура исследовательского процесса

```text
                 Physical Sources
                        ↓
                   Source Facts
                        ↓
                     Evidence
                        ↓
                  Semantic Claims
                     ↙       ↘
                UNKNOWN      Synthesis
                   ↓             ↓
          Research Question    Frame
                   ↓             ↓
              Search Plan     Relation
                   ↓             ↓
              New Evidence   Capability
                   └──────┬──────┘
                          ↓
                  Semantic Graph
                          ↓
                  Platform Candidate
                          ↓
                  Human Decision
```

---

## 54. Ключевой принцип v2.3

Semantic Graph — не только хранилище результата reverse engineering.

Он является частью замкнутого исследовательского процесса:

```text
Source
   ↓
Fact
   ↓
Evidence
   ↓
Claim
   ↓
Graph
   ↓
UNKNOWN
   ↓
Research Question
   ↓
Search Plan
   ↓
New Evidence
   ↓
Updated Graph
```

А когда несколько подтверждённых Claims требуют более высокого понятия:

```text
Claims
   ↓
Semantic Synthesis
   ↓
Synthesized Frame
```

Методология одновременно обеспечивает:

```text
доказательность
воспроизводимость
provenance
управление неизвестностью
контролируемый semantic synthesis
```

и не позволяет исследованию незаметно превратиться в проектирование новой архитектуры.

---

## 55. Финальная последовательность проекта

```text
Existing System
      ↓
Source Snapshots
      ↓
Research Passes
      ↓
Source Facts
      ↓
Evidence
      ↓
Claims
      ↓
Research Questions / Loops
      ↓
Semantic Synthesis
      ↓
Frames / Relations
      ↓
Capabilities
      ↓
Dependencies / Boundaries
      ↓
Platform Candidates
      ↓
Human Architectural Decision
      ↓
Platform Contract
```

Платформа остаётся последним этапом.

Исследование начинается с существующей системы и заканчивается только там, где доказательная модель позволяет безопасно принять архитектурное решение.
