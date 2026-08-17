# Semantic Graph MCR-01 — Order Lifecycle

**Статус:** эксперимент  
**Цель:** проверить, достаточно ли минимального языка `Frame / Relation / Fact` для представления реального lifecycle заказа в Taxi Web + Core Backend.

---

## 1. Исходный материал

Эксперимент использует уже исследованные:

```text
RP-01 — Core Backend
RP-02 — Taxi Web v0.1.20
```

Из RP-02 подтверждены два набора состояний.

### Booking state

```text
Processing
Approved
Canceled
Completed
PendingActivation
OfferedToDrivers
```

### Driver state

```text
Considering
Canceled
Performer
Arrived
Started
Finished
```

Также подтверждены операции:

```text
createOrder
cancelOrder
takeOrder
setOrderState
setOrderRating
setWaitingTime
setTips
```

Это именно исходный материал исследования клиента; эксперимент не добавляет новых состояний или операций.

---

# 2. Вопрос MCR

Проверяем один вопрос:

> Можно ли выразить реальный lifecycle заказа через `Frame / Relation / Fact`, не вводя новые фундаментальные типы `State`, `Transition`, `Operation` или `Event`?

Важно:

**не требуется сейчас построить полный executable FSM.**

Проверяется именно способность Semantic Graph представить семантику существующей системы.

---

# 3. Первый вариант: состояния как Frame

Можно представить:

```text
Frame:
  Order

Frame:
  Processing

Frame:
  Approved

Frame:
  Canceled

Frame:
  Completed
```

и отношения:

```text
Order
  HAS_STATE
  Processing

Order
  HAS_STATE
  Approved
```

Но здесь возникает проблема.

`Processing` и `Approved` не являются самостоятельными сущностями системы в том же смысле, что:

```text
Order
User
Car
Trip
```

Они являются значениями состояния Order.

Следовательно, создание для каждого состояния самостоятельного Frame увеличивает граф без необходимости.

### Результат

```text
REJECT — State-as-Frame
```

Причина:

> для представления текущего состояния достаточно атрибута Frame `Order`, если задача состоит только в фиксации состояния.

Новый фундаментальный тип `State` не требуется.

---

# 4. Второй вариант: состояние как атрибут Frame

Представим:

```text
Frame:
  Order

attribute:
  booking_state = Processing
```

или:

```text
Order.booking_state = Approved
```

Это позволяет представить факт:

```text
Order
  state
  Approved
```

без создания дополнительного Frame.

Для текущего вопроса это достаточно.

### Результат

```text
ACCEPT
```

Но это пока не отвечает на вопрос о переходах.

---

# 5. Переход состояния

Реальная система содержит действия, которые меняют состояние:

```text
cancelOrder
takeOrder
setOrderState
```

Интуитивно хочется представить:

```text
Processing
    TRANSITIONS_TO
Canceled
```

или:

```text
Order
    TRANSITIONS_TO
Canceled
```

Но это уже смешивает:

```text
текущее состояние
```

и:

```text
правило / возможность перехода.
```

`Fact` вида:

```text
Order → state → Canceled
```

описывает состояние.

Он не описывает:

```text
при каком условии
каким действием
из какого состояния
в какое состояние
```

---

# 6. Попытка выразить transition без нового типа

Можно представить переход как Relation:

```text
Order
    CAN_TRANSITION_TO
Canceled
```

Но это теряет источник перехода:

```text
Processing
    →
cancelOrder
    →
Canceled
```

То есть бинарное отношение не хранит:

```text
precondition
action
result
```

Если попытаться представить действие отдельным Frame:

```text
cancelOrder
    CHANGES
Order
```

мы уже вводим `Operation` как самостоятельный Frame.

Это не обязательно неправильно, но это уже изменение языка.

---

# 7. Проверка необходимости Operation

Сначала рассматриваем, нужен ли нам вообще самостоятельный Frame:

```text
cancelOrder
```

Для текущего Graph v0.1 вопрос:

> Нужно ли графу знать, что конкретно `cancelOrder` является отдельной семантической сущностью?

Ответ пока:

**нет.**

Для верхнеуровневого Semantic Graph достаточно:

```text
Taxi Web
    CALLS
Order API
```

и:

```text
Order
    HAS_STATE
Canceled
```

Связать эти два факта причинной зависимостью сейчас нельзя без расширения языка.

Но отсутствие этой связи не мешает представить основные сущности системы.

### Результат

```text
Operation Frame
не вводится.
```

---

# 8. Driver lifecycle

Второй набор состояний:

```text
Considering
Performer
Arrived
Started
Finished
```

можно аналогично представить как атрибут:

```text
Order.driver_state
```

Например:

```text
Order.driver_state = Arrived
```

Это снова работает для представления состояния.

Но появляется дополнительная семантика:

```text
booking_state
+
driver_state
```

Они относятся к разным lifecycle одного заказа.

Это важный результат.

### Вывод

Нельзя сводить:

```text
Order state
```

и:

```text
Driver execution state
```

в один enum.

Но это не требует нового Frame type.

Можно сохранить:

```text
Order
  booking_state
  driver_state
```

---

# 9. Candidate lifecycle

В RP-02 обнаружено:

```text
Candidate Selection
```

и состояние кандидата:

```text
Considering
Canceled
Performer
Arrived
Started
Finished
```

Можно представить:

```text
Order
    USES
Candidate Selection
```

и отдельно:

```text
Order.driver_state = Considering
```

Но если понадобится представить:

```text
Order
    has
    multiple candidates
```

с индивидуальным состоянием каждого кандидата, простого scalar attribute уже недостаточно.

Например:

```text
Order
 ├── Driver A → Considering
 ├── Driver B → Canceled
 └── Driver C → Performer
```

Это уже не одно состояние Order.

---

# 10. Граница эксперимента: множество кандидатов

Это важный случай.

Чтобы представить:

```text
Order
    candidate A → state
    candidate B → state
    candidate C → state
```

нам потребуется либо:

```text
Candidate
```

как самостоятельный Frame,

либо специализированная структура отношений/квалификаторов.

Но это **не следует вводить сейчас**, потому что вопрос MCR-01 был про lifecycle Order, а не про полную модель candidate set.

Более того, предыдущий эксперимент уже показал отдельную проблему:

```text
ALL candidate states satisfy condition
```

которая не выражается простым бинарным Fact.

Это остаётся открытым вопросом, а не поводом немедленно вводить `Quantifier` или `LogicNode`.

---

# 11. Минимальное представление lifecycle

Для текущего Graph v0.1 достаточно:

```text
Frame:
  Order

Attributes:
  booking_state
  driver_state
```

и отношений:

```text
Taxi Web
    CREATES
Order

Taxi Web
    UPDATES
Order

Taxi Web
    READS
Order
```

а также:

```text
Order
    USES
User

Order
    USES
Car

Order
    USES
Candidate Selection
```

Это представляет существующую систему без расширения фундаментального языка.

---

# 12. Что потеряно

При таком минимальном представлении невозможно полноценно выразить:

```text
Processing
    --cancelOrder-->
Canceled
```

как единый семантический объект.

Также нельзя непосредственно представить:

```text
если все кандидаты достигли состояния X,
то Order переходит в Y
```

или:

```text
только из состояния A
разрешено действие B
```

То есть теряется:

```text
transition semantics
preconditions
actions
quantifiers
causal rules
```

---

# 13. Является ли это провалом минимального языка?

Нет.

Для текущего уровня графа вопрос:

> Как устроены основные семантические области и зависимости?

решается.

Для вопроса:

> Как точно воспроизвести executable FSM backend?

решение `Frame / Relation / Fact` в текущем виде недостаточно.

Это разные уровни детализации.

---

# 14. Важное различие

Получается:

```text
Semantic Graph
```

может представить:

```text
Order
  HAS_STATE
  Approved
```

или хранить:

```text
Order.booking_state = Approved
```

но это ещё не означает, что граф является:

```text
FSM execution model
```

Следовательно:

```text
Semantic Graph
    ≠
FSM
```

Граф может ссылаться на FSM как на отдельную семантическую структуру/источник.

---

# 15. Результат MCR-01

## Подтверждено

`Frame / Relation / Fact` достаточно для:

```text
Order
User
Car
Trip
Candidate Selection
Authentication
Payment representation
```

и их основных зависимостей.

## Подтверждено с ограничением

Состояние сущности можно хранить как атрибут:

```text
Order.booking_state
Order.driver_state
```

Для этого новый Frame type `State` не нужен.

## Не выражается минимальным языком

Полностью:

```text
State Transition
Precondition
Action → State change
Quantified Candidate condition
```

## Но

Это пока не является достаточным основанием для изменения Graph Language.

---

# 16. MCR Decision

```text
MCR-01

Result:
PARTIAL PASS

Graph Language:
Frame / Relation / Fact

Change required:
NO

New types introduced:
NONE
```

---

# 17. Зафиксированные Open Questions

### OQ-ORDER-001

Как представить transition semantics:

```text
State A
   ↓ action
State B
```

не превращая Semantic Graph в FSM?

### OQ-ORDER-002

Как представить action/precondition, если это потребуется для semantic reasoning?

### OQ-ORDER-003

Как представить множество связанных объектов:

```text
Order
  → Candidate A
  → Candidate B
  → Candidate C
```

с индивидуальными состояниями?

### OQ-ORDER-004

Как представить quantified rule:

```text
ALL candidates satisfy condition
```

без преждевременного введения `LogicNode` / `Quantifier`?

---

# 18. Следующий шаг

MCR-01 не должен автоматически порождать новый тип.

Следующий эксперимент имеет смысл провести на **реальном backend FSM**, а не продолжать теоретически обсуждать Order lifecycle.

То есть:

```text
Core Backend FSM
      ↓
реальные transitions
      ↓
конкретные SourceFacts
      ↓
попытка выразить их
Frame / Relation / Fact
      ↓
если не получается
      ↓
конкретный MCR
```

Это позволит проверить, является ли обнаруженный разрыв свойством реальной семантики backend FSM или лишь следствием слишком поверхностного представления frontend lifecycle.

---

# 19. Итог

MCR-01 дал полезный отрицательный результат:

```text
Frame / Relation / Fact
```

достаточны для **карты семантических сущностей и основных зависимостей**, но не доказаны достаточными для **описания правил переходов и логики FSM**.

Поэтому сейчас ничего не расширяем.

Следующий материал для эксперимента — **реальная FSM Core Backend**, где можно проверить конкретные transition rules.
