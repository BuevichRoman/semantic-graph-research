# Semantic Graph — First Crystallization Experiment v0.1

**Статус:** экспериментальная первая кристаллизация  
**Основание:** MCR-01 … MCR-07  
**Источник эмпирических данных:** Core Backend snapshot `archive_17012026_1259_clear.zip` и ранее выполненные Research Pass  
**Цель:** зафиксировать минимальную структуру Semantic Graph, которая уже подтверждена экспериментами, не превращая результат в окончательную архитектуру или универсальный язык логики.

---

# 1. Назначение

До этого момента структура Semantic Graph исследовалась по частям.

Последовательность была:

```text
реальный backend
      ↓
Source Facts
      ↓
Semantic Claims
      ↓
MCR
      ↓
проверка выразительности
      ↓
кристаллизация
```

MCR-01 … MCR-07 показали, какие конструкции:

1. непосредственно выражаются текущим языком;
2. требуют reification;
3. выходят за пределы простой бинарной семантики;
4. могут быть представлены единым структурным механизмом.

Этот документ фиксирует **первую рабочую версию структуры**, а не завершённую спецификацию.

---

# 2. Что именно кристаллизуется

Кристаллизуется не весь Semantic Graph.

Фиксируются только элементы, для которых уже есть достаточное Evidence:

```text
Frame
Relation
SemanticClaim
Evidence
Frame Reification
Reified Assertion Structure
```

Не кристаллизуются:

```text
Quantifier
Rule
LogicNode
ValueNode
```

как самостоятельные фундаментальные типы.

---

# 3. Базовая единица — Frame

`Frame` представляет семантически идентифицируемый объект исследуемой системы.

Примеры, подтверждённые исследованиями:

```text
Order
User
Driver
DriverCandidacy
Trip
Cart
Payment
FSM State
FSM Action
FSM Transition
```

Важно:

`Frame` не означает обязательно business entity.

Например:

```text
FSM Transition
```

может быть Frame, потому что один переход является носителем нескольких независимых отношений:

```text
FROM_STATE
ACTION
TO_STATE
```

Это было подтверждено MCR-02.

---

# 4. Relation

`Relation` представляет связь между двумя семантическими объектами.

Базовая форма:

```text
Frame
    ── Relation ──>
Frame
```

Примеры:

```text
Order
    HAS_STATE
State

Order
    HAS_CANDIDACY
DriverCandidacy

Transition
    FROM_STATE
State

Transition
    ACTION
Action

Transition
    TO_STATE
State
```

Relation остаётся бинарной.

Не вводится специальная:

```text
TernaryRelation
```

MCR-02 показал, что более сложная структура transition может быть получена через reification.

---

# 5. SemanticClaim

`SemanticClaim` является утверждением о семантике системы.

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

Claim сохраняет:

```text
Evidence
Confidence
Status
Provenance
```

Именно Claim является основной эпистемологической единицей.

Это важно для сохранения различия:

```text
что утверждается
```

и:

```text
насколько это утверждение подтверждено
```

---

# 6. Evidence

`Evidence` связывает SemanticClaim с наблюдаемым источником.

Источником может быть:

```text
CODE
DB
API_CONTRACT
DOCUMENTATION
TEST
RUNTIME
```

или другой тип, предусмотренный нормативной методикой.

В первой кристаллизации Evidence не меняется.

Сохраняется основной принцип:

```text
Source Fact
    ↓
Evidence
    ↓
Semantic Claim
```

Semantic Graph не должен превращать inference в source fact.

---

# 7. Confidence

Confidence остаётся свойством Claim, а не отдельным типом графа.

Сохраняется эпистемологическое разделение:

```text
CONFIRMED
INFERRED
UNKNOWN
```

При этом:

```text
UNKNOWN
```

не означает отсутствие Slot.

Это означает:

> значение или связь исследованием не установлены.

Это правило протоструктур сохраняется без изменений.

---

# 8. Frame Reification

MCR-02 показал, что некоторые элементы системы сами являются носителями нескольких независимых отношений.

Пример:

```text
Transition
    ├── FROM_STATE → State
    ├── ACTION     → Action
    └── TO_STATE   → State
```

Поэтому:

```text
Transition
```

не требуется делать новым фундаментальным типом.

Используется:

```text
Frame Reification
```

То есть:

```text
семантически сложный объект
        ↓
обычный Frame
        ↓
несколько Relations
```

Это первый подтверждённый механизм расширения выразительности без расширения фундаментального языка.

---

# 9. Почему Transition остаётся Frame

В исходном backend:

```text
from_state
action
to_state
```

образуют одну transition record.

Если представить её только Relation:

```text
StateA
    TRANSITIONS_TO
StateB
```

теряется Action.

Если представить Action отдельной Relation:

```text
StateA
    order_start
StateB
```

теряется атомарность transition.

Reification сохраняет:

```text
Transition Frame
```

и три независимые Relations.

Следовательно:

```text
Transition
```

является семантической ролью Frame, а не новым фундаментальным типом.

---

# 10. Reified Assertion

MCR-05 и MCR-06 установили два устойчивых класса backend semantics:

```text
Universal Predicate

∀ x ∈ S:
    P(x)
```

и:

```text
Cardinality

COUNT(S) == N
```

Оба не выражаются обычной бинарной Relation.

MCR-07 показал, что их можно представить единым структурным шаблоном:

```text
Assertion Frame
    ├── SCOPE
    ├── OPERATOR
    ├── PREDICATE
    └── VALUE
```

При этом не требуется новый фундаментальный тип.

---

# 11. Assertion как reified Claim

Важное решение первой кристаллизации:

`Assertion` не вводится как параллельная сущность рядом с `SemanticClaim`.

Вместо этого:

```text
SemanticClaim
       │
       └── reified_as
              ↓
       Assertion Frame
```

Таким образом:

```text
Claim
```

остаётся эпистемологической единицей.

А:

```text
Assertion Frame
```

является структурным представлением сложного Claim.

Это позволяет сохранить:

```text
Evidence
Confidence
Status
Provenance
```

на Claim.

---

# 12. Universal Predicate

Реальный пример:

```text
ALL DriverCandidacy
    have state 6
```

может быть представлен:

```text
Assertion Frame
    ├── SCOPE
    │      ↓
    │  DriverCandidacySet
    │
    ├── OPERATOR
    │      ↓
    │     ALL
    │
    └── PREDICATE
           ↓
      CandidateStateIs6
```

`ALL`, `DriverCandidacySet` и `CandidateStateIs6` могут сами быть Frames.

Новый:

```text
QuantifierNode
```

не требуется.

---

# 13. Cardinality Assertion

Реальный пример:

```text
COUNT(selected_drivers)
    ==
current_driver_count
```

может быть представлен:

```text
Assertion Frame
    ├── SCOPE
    │      ↓
    │  SelectedDrivers
    │
    ├── OPERATOR
    │      ↓
    │   COUNT_EQ
    │
    └── VALUE
           ↓
    current_driver_count
```

Здесь `VALUE` может быть scalar value.

Это важно:

```text
scalar value
```

не превращается автоматически в Frame.

---

# 14. Observation и Assertion

Первая кристаллизация фиксирует различие.

### Observation

```text
SelectedDrivers.count = 3
```

означает:

> наблюдаемое текущее значение.

### Assertion

```text
COUNT(SelectedDrivers) == 3
```

означает:

> структурированное условие.

Они не должны сливаться.

Следовательно:

```text
Slot.value
```

может хранить observation.

А:

```text
Assertion Frame
```

может описывать comparison semantics.

---

# 15. Минимальная V1-картина

Получается:

```text
Semantic Graph V1
│
├── Frame
│
├── Relation
│
├── SemanticClaim
│
├── Evidence
│
└── Reification
      │
      ├── Reified Frame
      │     └── complex semantic object
      │
      └── Reified Assertion
            ├── Scope
            ├── Operator
            ├── Predicate
            └── Value
```

`Reification` здесь является механизмом, а не новым node type.

---

# 16. Что подтверждено экспериментами

| Element | Evidence | Status |
|---|---|---|
| Frame | MCR-01, MCR-02 | CONFIRMED |
| Binary Relation | MCR-01, MCR-02 | CONFIRMED |
| SemanticClaim | Prototype specification + Research Pass | CONFIRMED |
| Evidence / provenance | Prototype specification + Research Pass | CONFIRMED |
| Frame reification | MCR-02 | CONFIRMED |
| Assertion reification | MCR-07 | CONFIRMED |
| Universal Predicate structure | MCR-05 + MCR-07 | CONFIRMED |
| Cardinality structure | MCR-06 + MCR-07 | CONFIRMED |
| Quantifier as fundamental node | — | NOT ESTABLISHED |
| Rule as fundamental node | — | NOT ESTABLISHED |
| LogicNode | — | NOT ESTABLISHED |

---

# 17. Что пока остаётся открытым

Не кристаллизуются:

```text
AND
OR
NOT
ANY
NONE
```

как отдельные фундаментальные механизмы.

Также открыты:

```text
nested assertions
arithmetic comparison
inequality
ranges
temporal conditions
causal conditions
effects
guards
execution semantics
```

Причина не в том, что они невозможны.

Причина:

> для них ещё нет достаточного экспериментального Evidence.

---

# 18. Assertion не является Rule Engine

Это критическое ограничение.

Мы получили:

```text
Assertion
```

как структурированное semantic representation.

Но не получили:

```text
execution engine
```

Следовательно:

```text
Semantic Graph
    ≠
FSM Engine
    ≠
Rule Engine
```

Граф может хранить:

```text
COUNT(S) == N
```

не будучи обязанным самостоятельно вычислять `COUNT`.

---

# 19. Связь с backend FSM

FSM теперь можно представить:

```text
State Frame
    ↑
    │ FROM_STATE
Transition Frame
    │
    ├── ACTION
    ↓
Action Frame
    │
    └── TO_STATE
          ↓
       State Frame
```

Если transition имеет условие:

```text
ALL candidates finished
```

можно дополнительно:

```text
Transition Frame
    REQUIRES
Assertion Frame
```

где Assertion:

```text
SCOPE = CandidateSet
OPERATOR = ALL
PREDICATE = StateFinished
```

Это связывает результаты MCR-02 и MCR-07.

При этом условие не становится частью самого фундаментального Transition type.

---

# 20. Связь с Platform Research

Это особенно важно для дальнейшего выделения:

```text
Platform Auth
Platform Cart
Platform Payment
```

Semantic Graph теперь потенциально способен хранить не только:

```text
Entity
    → dependency
```

но и:

```text
Capability
    → implementation
    → data
    → transition
    → assertion
```

Например:

```text
Cart
   └── Assertion
          SCOPE = CartItems
          OPERATOR = ALL
          PREDICATE = ...
```

Но конкретные platform boundaries из этого документа не выводятся.

---

# 21. Что НЕ следует делать после этой кристаллизации

Нельзя автоматически:

```text
1. переписывать всю Prototype Structures v2.2;
2. объявлять V1 окончательной моделью;
3. добавлять все логические операторы;
4. строить Rule Engine;
5. менять существующий backend;
6. выводить Platform Auth / Cart / Payment.
```

Этот документ фиксирует результат эксперимента.

Нормативная спецификация должна быть изменена только после отдельного review.

---

# 22. Граница между протоструктурой и структурой

Получилась полезная граница.

### Prototype layer

Содержит:

```text
SourceFact
Evidence
Claim
Frame
Relation
Slot
Candidate
```

и допускает:

```text
UNKNOWN
INFERRED
CONFLICT
```

### Crystallized graph layer

Добавляет подтверждённый механизм:

```text
Reification
```

и конкретный reified pattern:

```text
Assertion
```

Но это не отменяет provenance/prototype layer.

Цепочка остаётся:

```text
Source
   ↓
Evidence
   ↓
SemanticClaim
   ↓
Frame / Relation
   ↓
Reification when required
```

---

# 23. Предварительная структура данных

Концептуально:

```text
Frame {
    id
    type
    name
    slots
    ...
}
```

```text
Relation {
    id
    source_frame
    type
    target_frame
    claim_id
}
```

```text
SemanticClaim {
    id
    subject
    predicate
    object | value
    evidence_ids
    confidence
    status
}
```

```text
AssertionFrame {
    id
    claim_id
    scope
    operator
    predicate
    value
}
```

Здесь `AssertionFrame` — концептуальное имя reified Frame, а не предложение добавить новый фундаментальный `type`.

---

# 24. Provenance

Каждый кристаллизованный элемент должен сохранять происхождение.

Например:

```text
AssertionFrame:
    A_SelectedDriversReachedTarget
```

должен быть связан с Claim, который в свою очередь связан с:

```text
Evidence
    ↓
SourceFact
    ↓
actual backend source
```

Нельзя создавать:

```text
Assertion
```

только потому, что она выглядит логично.

Основание должно быть:

```text
real source behavior
+
Semantic Claim
+
Evidence
```

---

# 25. Confidence

Кристаллизация структуры не меняет confidence model.

Если:

```text
Claim = CONFIRMED
```

то reified Assertion наследует provenance этого Claim.

Но:

```text
Assertion Frame
```

не должен самостоятельно превращать:

```text
INFERRED
```

в:

```text
CONFIRMED
```

Структурная кристаллизация не является повышением Evidence.

---

# 26. Первый практический вывод

Теперь можно перейти от вопроса:

```text
"какой язык графа нам придумать?"
```

к более практическому:

```text
"какие элементы этой структуры действительно нужны
для хранения Semantic Backend Graph?"
```

Это существенный переход.

Мы получили минимальную структуру не из теории, а из:

```text
реальный код
    ↓
Research Pass
    ↓
MCR-01 … MCR-07
    ↓
экспериментальная кристаллизация
```

---

# 27. Итог

Первая кристаллизация даёт:

```text
Frame
Relation
SemanticClaim
Evidence
```

как базовые элементы.

И:

```text
Frame Reification
```

как подтверждённый механизм представления более сложной семантики.

Для reified assertions подтверждён шаблон:

```text
Assertion Frame
    ├── Scope
    ├── Operator
    ├── Predicate
    └── Value
```

Он покрывает:

```text
Universal Predicate
Cardinality
```

на реальных случаях Core Backend.

При этом:

```text
Quantifier
Rule
LogicNode
```

не стали фундаментальными типами.

---

# 28. Следующий шаг

Не расширять структуру теоретически.

Следующий эксперимент должен быть практическим:

```text
Semantic Backend Graph v0.1
        ↓
применить первую кристаллизованную структуру
        ↓
к реальному Core Backend
        ↓
не ко всему коду сразу,
а к одному ограниченному подграфу
```

Оптимальный следующий подграф — **Authentication**, поскольку он уже исследован и дополнительно имеет связанный frontend-код Taxi Web.

Цель:

```text
проверить,
достаточна ли V1-структура
для реального end-to-end semantic subgraph:
Frontend
    ↕
Core Backend
    ↕
DB / configuration
```

Если структура выдерживает Authentication без новых фундаментальных типов, можно считать первую кристаллизацию практически подтверждённой и переходить к следующему домену.
