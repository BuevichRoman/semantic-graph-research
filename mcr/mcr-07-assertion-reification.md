# Semantic Graph MCR-07 — Assertion Reification

**Статус:** завершённый эксперимент  
**Источник:** Core Backend snapshot `archive_17012026_1259_clear.zip`  
**Основание:** MCR-05, MCR-06  
**Проверяемые случаи:** Q-01, Q-03, Q-04, Q-05  
**Цель:** проверить, можно ли представить два уже подтверждённых expressive gaps — Universal Predicate и Cardinality — одним минимальным механизмом `Assertion`, не превращая его в полноценный логический язык и не добавляя новые фундаментальные типы графа.

---

# 1. Исходная точка

MCR-05 установил:

```text
Universal Predicate

∀ x ∈ S:
    P(x)
```

не выражается обычными:

```text
Frame
Relation
Fact
```

MCR-06 установил аналогично:

```text
Cardinality

COUNT(S) == N
```

не выражается текущим языком.

При этом оба класса реально встречаются в Core Backend.

Теперь проверяется не абстрактный `Quantifier`, а более узкая гипотеза:

> Может ли одно reified Assertion представить оба класса, если его внутренние компоненты сами являются обычными Frames / Slots / Values?

---

# 2. Ограничение эксперимента

MCR-07 не пытается построить:

```text
Rule Engine
Logic Engine
Inference Engine
```

Проверяется только:

```text
representation
```

То есть:

> Можно ли сохранить семантическую структуру утверждения так, чтобы она не превратилась в непрозрачный label?

Это принципиально.

---

# 3. Базовая гипотеза

Используем уже существующий `Frame` как носитель reified assertion.

Концептуально:

```text
Assertion
    ├── SCOPE
    ├── OPERATOR
    ├── PREDICATE / AGGREGATION
    └── VALUE
```

Здесь `Assertion` не является новым фундаментальным типом языка.

Это:

```text
обычный Frame
```

с определённой семантической ролью.

---

# 4. Assertion для Q-01

Исходное правило:

```text
ALL DriverCandidacy
    have state 6
```

Представляем:

```text
Frame:
    A_AllCandidatesState6
```

Slots:

```text
scope
    → DriverCandidacySet

operator
    → ALL

predicate
    → CandidateStateIs6
```

Граф:

```text
A_AllCandidatesState6
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

Predicate:

```text
Candidate
    HAS_STATE
State6
```

Теперь Assertion содержит отдельно:

```text
scope
operator
predicate
```

а не скрывает их внутри Relation name.

---

# 5. Проверка Q-03

Исходная семантика:

```text
ALL requested users
    EXIST
```

Представляем:

```text
A_AllRequestedUsersExist
```

Slots:

```text
scope
    → RequestedUserSet

operator
    → ALL

predicate
    → UserExists
```

Получается тот же шаблон:

```text
Assertion
    SCOPE
    OPERATOR = ALL
    PREDICATE
```

Различается только:

```text
scope
predicate
```

Это именно то, что требовалось проверить.

---

# 6. Проверка Q-03 с role predicate

Более сложный вариант:

```text
ALL requested users
    HAVE_ROLE
    2
```

Можно представить:

```text
A_AllRequestedUsersRole2
```

Slots:

```text
scope
    → RequestedUserSet

operator
    → ALL

predicate
    → UserHasRole2
```

А Predicate Frame:

```text
UserHasRole2
    ├── subject role
    └── value 2
```

или, если роль является отдельным семантическим Frame:

```text
UserHasRole2
    HAS_ROLE
Role2
```

Это снова не требует нового фундаментального типа.

---

# 7. Assertion для Q-04

Исходное правило:

```text
COUNT(selected_drivers) == current_driver_count
```

Представляем:

```text
A_SelectedDriversReachedTarget
```

Slots:

```text
scope
    → SelectedDrivers

operator
    → COUNT_EQ

value
    → current_driver_count
```

Граф:

```text
A_SelectedDriversReachedTarget
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

Здесь нет Predicate.

Вместо него используется:

```text
aggregation = COUNT
comparison = EQ
```

Но это можно представить оператором:

```text
COUNT_EQ
```

как семантическим оператором Assertion.

---

# 8. Assertion для Q-05

Исходное правило:

```text
COUNT(trips_seats[id_trip])
    ==
looking_for_clients
```

Представляем:

```text
A_TripSeatsReachedRequiredQuantity
```

Slots:

```text
scope
    → TripSeats

operator
    → COUNT_EQ

value
    → looking_for_clients
```

Структура идентична Q-04.

Различается только:

```text
scope
value
```

---

# 9. Сводка четырёх случаев

| Case | Scope | Operator | Predicate | Value |
|---|---|---|---|---|
| Q-01 | DriverCandidacies | ALL | State=6 | — |
| Q-03 | RequestedUsers | ALL | Exists | — |
| Q-03 role | RequestedUsers | ALL | Role=2 | — |
| Q-04 | SelectedDrivers | COUNT_EQ | — | target |
| Q-05 | TripSeats | COUNT_EQ | — | required |

Получается один структурный шаблон:

```text
Assertion
    SCOPE
    OPERATOR
    [PREDICATE]
    [VALUE]
```

---

# 10. Проверка минимальности

Теперь важно проверить, не спрятали ли мы новую систему в терминах.

## Scope

```text
Frame
```

уже существует.

## Predicate

```text
Frame
```

уже существует.

## Value

`Slot.value` уже поддерживает scalar value.

## Operator

Здесь появляется потенциально новый вопрос.

Можно сделать:

```text
ALL
COUNT_EQ
```

обычными семантическими Frames:

```text
Frame:
    ALL

Frame:
    COUNT_EQ
```

Тогда сам Assertion остаётся обычным Frame.

То есть даже Operator не требует нового фундаментального node type.

---

# 11. Полученная структура

В минимальном варианте:

```text
Assertion Frame
    │
    ├── SCOPE ───────→ Set Frame
    │
    ├── OPERATOR ────→ Operator Frame
    │
    ├── PREDICATE ───→ Predicate Frame
    │
    └── VALUE ───────→ scalar value
```

Для Universal Predicate:

```text
Assertion
 ├── Scope
 ├── ALL
 └── Predicate
```

Для Cardinality:

```text
Assertion
 ├── Scope
 ├── COUNT_EQ
 └── Value
```

---

# 12. Главный результат

Это важный результат:

```text
Universal Predicate
```

и:

```text
Cardinality
```

могут быть представлены одним **структурным шаблоном Assertion**.

При этом не потребовалось вводить:

```text
QuantifierNode
CardinalityNode
LogicNode
ValueNode
RuleNode
```

как новые фундаментальные типы.

---

# 13. Но есть принципиальное ограничение

Это ещё не означает, что:

```text
Assertion
```

умеет **вычислять** себя.

Например:

```text
COUNT_EQ
```

нужно интерпретировать как:

```text
count(scope) == value
```

а:

```text
ALL
```

как:

```text
for every member of scope:
    predicate(member)
```

Эта семантика должна существовать на уровне интерпретатора/consumer Semantic Graph.

Сам граф хранит:

```text
структуру утверждения
```

а не обязательно исполняет её.

---

# 14. Representation ≠ Execution

Это различие нужно закрепить.

```text
Semantic Graph
    ↓
Assertion structure
```

не означает:

```text
Semantic Graph
    ↓
Rule Engine
```

Следовательно, MCR-07 не создаёт скрытый executable rule engine.

Он только показывает, что representation может быть структурированной.

---

# 15. Проверка на Q-01

Можно восстановить исходную семантику:

```text
A_AllCandidatesState6
```

из:

```text
scope = DriverCandidacySet
operator = ALL
predicate = CandidateStateIs6
```

То есть не теряются:

```text
какое множество
какой оператор
какое условие
```

---

# 16. Проверка на Q-04

Аналогично:

```text
A_SelectedDriversReachedTarget
```

сохраняет:

```text
scope = SelectedDrivers
operator = COUNT_EQ
value = current_driver_count
```

Не теряется:

```text
что считаем
что сравниваем
с чем сравниваем
```

---

# 17. Проверка на существующую модель SemanticClaim

Здесь обнаруживается важное архитектурное совпадение.

В текущей спецификации уже есть:

```text
SemanticClaim
```

как атомарное утверждение:

```text
subject
predicate
object / value
```

и:

```text
Frame
```

как reifiable semantic object.

Поэтому `Assertion Frame` не обязательно должен становиться новым объектом протоструктуры.

Можно рассматривать:

```text
SemanticClaim
```

как **утверждение**, а:

```text
Assertion Frame
```

как его reified структурное представление, когда бинарной формы claim недостаточно.

Это существенно уменьшает количество новых сущностей.

---

# 18. Возникает новая гипотеза

Вместо:

```text
новый тип = Assertion
```

можно проверить:

```text
SemanticClaim
        ↓
простое утверждение

SemanticClaim
        ↓ reification
Assertion Frame
        ↓
SCOPE / OPERATOR / PREDICATE / VALUE
```

То есть мы не расширяем эпистемологическую модель.

Мы расширяем **форму представления claim**, когда его структура выходит за пределы:

```text
subject + predicate + object/value
```

---

# 19. Это лучше согласуется с текущей моделью

Сейчас:

```text
SemanticClaim
```

уже является основной единицей:

```text
confidence
evidence
status
provenance
```

Если ввести отдельный:

```text
Rule
```

или:

```text
Assertion
```

как параллельную сущность, возникает новая проблема:

```text
к чему относятся Evidence?
к Claim?
к Rule?
```

Reification позволяет избежать этого:

```text
Claim
  │
  └── reified_as → Assertion Frame
```

Evidence и confidence остаются на Claim.

---

# 20. MCR-07 Decision

```text
MCR-07

Question:
Can Universal Predicate and Cardinality
share one minimal representation?

Result:
PASS — structural representation

Common pattern:
Assertion structure

New fundamental node types:
NONE

Recommended interpretation:
SemanticClaim + optional reified Assertion Frame
```

---

# 21. Что MCR-07 НЕ утверждает

Не утверждается:

```text
Semantic Graph now has Rule Engine.
```

Не утверждается:

```text
ALL и COUNT_EQ являются окончательными operator vocabulary.
```

Не утверждается:

```text
Assertion Frame уже должен войти
в нормативную JSON Schema.
```

Не утверждается:

```text
все будущие logical semantics
будут представлены этим шаблоном.
```

MCR-07 доказал только структурную выразимость четырёх уже известных реальных случаев.

---

# 22. Следствие для V1 структуры

Теперь впервые появляется обоснованный кандидат на первую версию структуры:

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
└── Reified Claim
        │
        └── Assertion Structure
              ├── Scope
              ├── Operator
              ├── Predicate
              └── Value
```

При этом:

```text
Assertion
```

не обязан быть новым fundamental type.

Это может быть роль/структура Frame, связанная с конкретным SemanticClaim.

---

# 23. Почему это важно

До MCR-07 мы имели:

```text
Frame / Relation / Fact
```

и две подтверждённые дырки:

```text
Universal Predicate
Cardinality
```

Теперь:

```text
Universal Predicate
        ┐
        ├── Assertion Structure
Cardinality
        ┘
```

могут быть представлены одним минимальным механизмом.

Это первый эксперимент, который действительно даёт основание говорить о **первой версии структуры Semantic Graph**, а не только о протоструктурах отдельных backend facts.

---

# 24. Что делать дальше

Не надо сразу переписывать нормативную спецификацию v2.3.

Следующий шаг:

```text
Semantic Graph Structure V1 — First Crystallization
```

отдельный экспериментальный документ.

В него следует перенести только то, что подтверждено MCR:

```text
1. Frame
2. Relation
3. SemanticClaim
4. Evidence
5. Frame reification
6. Assertion structure
```

И отдельно оставить:

```text
OPEN
```

для:

```text
execution semantics
AND / OR
NOT
ANY
NONE
nested assertions
arithmetic comparisons
temporal conditions
causal rules
```

То есть MCR-07 даёт нам **первую кристаллизацию**, но не повод превращать граф в универсальный язык логики.
