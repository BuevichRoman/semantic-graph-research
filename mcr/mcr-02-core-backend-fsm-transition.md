# Semantic Graph MCR-02 — Core Backend FSM Transition

**Статус:** завершённый эксперимент  
**Цель:** проверить на реальном Core Backend FSM, может ли минимальный язык `Frame / Relation / Fact` выразить точный переход FSM, включая `from_state`, `action` и `to_state`, без введения нового фундаментального языка.

---

## 1. Исходные источники

Использованы существующие материалы по FSM Core и Taxi FSM:

- `fsm_states`, `fsm_actions`, `fsm_transitions`;
- `fsm_perform_action`;
- `fsm_action_logs`;
- `fsm_spec.py`;
- `taxi_order_fsm_seed.sql`;
- описание совместимости реального backend/FSM;
- результаты MCR-01.

Источник описывает существующий табличный FSM как:

```text
from_state + action -> to_state
```

с исполнением через:

```text
fsm_perform_action
```

и журналом:

```text
fsm_action_logs
```

Это подтверждено в описании FSM-ядра. fileciteturn6file7turn6file8

Taxi seed определяет конкретные состояния, actions и transitions для order FSM. Например:

```text
order_created
    + order_publish_vote
    -> order_vote_waiting_candidates
```

и:

```text
order_driver_arrived
    + order_start
    -> order_in_ride
```

а также:

```text
order_in_ride
    + order_finish
    -> order_completed
```

fileciteturn6file0turn6file11

---

# 2. Вопрос MCR

Проверяется конкретный вопрос:

> Можно ли выразить реальный FSM transition через существующие `Frame / Relation / Fact`, сохранив одновременно `from_state`, `action` и `to_state`, не вводя новый фундаментальный тип языка?

Это более точный вопрос, чем вопрос MCR-01.

MCR-01 проверял:

```text
Order
  state = X
```

MCR-02 проверяет:

```text
X
  --action-->
Y
```

---

# 3. Попытка №1 — бинарное Relation

Наивное представление:

```text
order_created
    TRANSITIONS_TO
order_vote_waiting_candidates
```

Получается:

```text
Frame
   ↓
Relation
   ↓
Frame
```

Но теряется:

```text
order_publish_vote
```

То есть два разных перехода:

```text
order_created
    --order_publish_vote-->
order_vote_waiting_candidates
```

и гипотетический:

```text
order_created
    --some_other_action-->
order_vote_waiting_candidates
```

становятся неразличимыми.

### Результат

```text
REJECT — direct binary transition
```

Причина:

> Binary Relation не имеет слота для action.

---

# 4. Попытка №2 — Action как Relation

Можно попробовать:

```text
order_created
    order_publish_vote
    order_vote_waiting_candidates
```

где `order_publish_vote` трактуется как тип Relation.

Но Relation имеет только:

```text
subject
object
```

Поэтому это фактически снова:

```text
order_created
    --order_publish_vote-->
order_vote_waiting_candidates
```

и с точки зрения выразительности проблема не меняется.

### Результат

```text
REJECT — action-as-relation
```

---

# 5. Попытка №3 — Action как Frame

В реальном FSM `order_publish_vote` является самостоятельным элементом словаря `fsm_actions`.

Это не придуманный нами объект.

Источник прямо содержит отдельные rows/actions:

```text
order_publish_vote
order_select_candidate
order_release_candidate
order_cancel_by_client
order_arrive
order_start
order_finish
...
```

fileciteturn6file11turn6file4

Поэтому Action может быть представлен как Frame:

```text
Frame:
    order_created

Frame:
    order_publish_vote

Frame:
    order_vote_waiting_candidates
```

Далее:

```text
order_created
    --PRECEDES-->
order_publish_vote

order_publish_vote
    --LEADS_TO-->
order_vote_waiting_candidates
```

Проблема:

такая схема уже позволяет различать action, но не гарантирует, что эти два факта являются **одним атомарным transition**.

Можно случайно получить:

```text
A PRECEDES action
```

и:

```text
action LEADS_TO B
```

без утверждения:

```text
A + action -> B
```

### Результат

```text
PARTIAL
```

Action как Frame полезен, но двух бинарных Facts недостаточно для сохранения атомарности transition.

---

# 6. Попытка №4 — Transition как Frame

Теперь используем реальную семантику таблицы:

```text
fsm_transitions
```

Одна строка transition имеет тройку:

```text
from_state
action
to_state
```

Это самостоятельный семантический объект перехода.

Создаём:

```text
Frame:
    T_order_publish_vote
```

и связываем:

```text
T_order_publish_vote
    --FROM_STATE-->
order_created

T_order_publish_vote
    --ACTION-->
order_publish_vote

T_order_publish_vote
    --TO_STATE-->
order_vote_waiting_candidates
```

Получаем:

```text
                ┌───────────────┐
                │ T_publish_vote│
                └───────┬───────┘
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
       FROM_STATE     ACTION     TO_STATE
             ↓          ↓          ↓
     order_created  publish_vote  waiting
```

Все элементы остаются обычными:

```text
Frame
Relation
Fact
```

Новый фундаментальный тип языка не появился.

---

# 7. Проверка на другом переходе

Берём:

```text
order_driver_arrived
    + order_start
    -> order_in_ride
```

Создаём:

```text
T_order_start
```

и:

```text
T_order_start
    --FROM_STATE-->
order_driver_arrived

T_order_start
    --ACTION-->
order_start

T_order_start
    --TO_STATE-->
order_in_ride
```

Семантика полностью различима.

То же представление работает для:

```text
order_in_ride
    + order_finish
    -> order_completed
```

fileciteturn6file0

---

# 8. Проверка ветвления

Особенно важен случай:

```text
order_created
```

имеет разные переходы:

```text
order_created
    + order_publish_vote
    -> order_vote_waiting_candidates

order_created
    + order_assign_direct
    -> order_driver_assigned

order_created
    + order_publish_offer
    -> order_offer_waiting
```

fileciteturn6file0turn6file11

При прямом binary Relation эти переходы начинают конфликтовать по форме.

При Transition-as-Frame:

```text
T1
FROM order_created
ACTION order_publish_vote
TO waiting_candidates

T2
FROM order_created
ACTION order_assign_direct
TO driver_assigned

T3
FROM order_created
ACTION order_publish_offer
TO offer_waiting
```

ветвление выражается естественно.

---

# 9. Проверка обратного перехода

FSM содержит:

```text
order_vote_driver_assigned
    + order_release_candidate
    -> order_vote_waiting_candidates
```

То есть:

```text
A → B
```

и:

```text
B → A
```

могут сосуществовать.

Transition Frame позволяет представить оба перехода как независимые Frames:

```text
T_select
T_release
```

со своими action.

Это сохраняет направление и различие операций.

---

# 10. Проверка нескольких actions из одного state

Для:

```text
order_vote_driver_assigned
```

есть как минимум:

```text
order_release_candidate
order_no_show
order_arrive
order_cancel_by_client
```

которые ведут в разные состояния.

Transition Frames:

```text
T_release
T_no_show
T_arrive
T_cancel
```

дают четыре независимых семантических объекта.

Таким образом, граф сохраняет branching FSM без специального `Transition` типа.

---

# 11. Проверка action identity

Важное свойство:

```text
order_arrive
```

используется более чем в одном переходе:

```text
order_vote_driver_assigned
    + order_arrive
    -> order_driver_arrived

order_driver_assigned
    + order_arrive
    -> order_driver_arrived
```

fileciteturn6file0

Следовательно:

```text
Action
```

должен быть отдельным Frame, если мы хотим моделировать его identity:

```text
order_arrive
```

а:

```text
Transition
```

должен быть отдельным Frame, если нужно различать конкретные применения этого Action.

Получается:

```text
Action Frame
        ↑
        │ ACTION
        │
Transition Frame
        │
        ├── FROM_STATE → State Frame
        └── TO_STATE   → State Frame
```

---

# 12. Это не ввод нового фундаментального типа

На первый взгляд появляется:

```text
Transition Frame
```

Но это не новый элемент языка.

Фундаментальный язык остаётся:

```text
Frame
Relation
Fact
```

`Transition` — это **семантическая роль конкретного Frame**, сформированного из реального объекта/строки `fsm_transitions`.

То же относится к:

```text
Action
State
Order
```

Они не требуют отдельных фундаментальных типов языка.

Это принципиально отличается от:

```text
добавить Type = TRANSITION
```

на уровне ядра языка.

---

# 13. Связь с MCR-01

MCR-01 установил:

```text
Order.booking_state
Order.driver_state
```

можно представить как атрибуты, если нужен только snapshot текущего состояния.

MCR-02 показывает:

если нужно представить **сам механизм FSM**, состояния должны стать Frames, потому что они участвуют в независимых отношениях:

```text
Transition
   FROM_STATE
      ↓
   State
```

Таким образом, получаем важное правило:

```text
State as attribute
```

достаточно для:

```text
"какое состояние сейчас?"
```

но:

```text
State as Frame
```

нужно для:

```text
"какие переходы существуют между состояниями?"
```

Это не противоречие MCR-01.

Это зависимость от уровня семантического вопроса.

---

# 14. Preconditions

Теперь проверяем более сложное:

```text
State A
    + Action
    + Condition
    -> State B
```

В текущем материале подтверждено, что существующий FSM engine имеет ограничения вокруг guards.

Документы по текущему engine описывают:

```text
from_state + action -> to_state
```

а отдельный guard layer в исходном SQL core не подтверждён; guard рассматривается как дополнительный runtime/application mechanism. fileciteturn6file7turn6file9

Следовательно, нельзя приписывать текущему `fsm_transitions` семантику:

```text
Transition
    HAS_GUARD
```

как уже существующий backend fact.

Это пока отдельный архитектурный слой.

---

# 15. Effects

Аналогично:

```text
transition
    effect
```

не следует считать частью существующего минимального FSM факта.

В материалах по расширению FSM предлагаются:

```text
guard_name
guard_params
effect_name
effect_params
```

но это уже направление развития FSM Platform, а не доказанный факт исходного FSM. fileciteturn6file6

Следовательно:

```text
Graph Fact:
Transition → EFFECT → X
```

не создаём в MCR-02 без отдельного SourceFact.

---

# 16. Timer

Аналогичная ситуация:

```text
expire
no-show
```

могут быть actions/transition semantics.

Но универсальный timer subsystem в текущем engine отдельно не подтверждён. Документация прямо отмечает его отсутствие в исходной модели. fileciteturn6file7

Поэтому:

```text
Timer
```

не вводится как Frame только из-за обсуждаемой архитектуры расширения.

---

# 17. Результат MCR-02

```text
MCR-02

Question:
Can Frame / Relation / Fact represent an actual FSM transition?

Result:
PASS — with reification
```

Минимальный язык **не требует расширения**.

Рабочий шаблон:

```text
Transition Frame
    │
    ├── FROM_STATE → State Frame
    ├── ACTION     → Action Frame
    └── TO_STATE   → State Frame
```

Все отношения остаются обычными бинарными Relations.

---

# 18. Что именно доказано

Доказано на реальном FSM-описании, что `Frame / Relation / Fact` может представить:

```text
1. states
2. actions
3. transitions
4. branching
5. reverse transitions
6. same action in multiple transitions
7. exact from/action/to triple
```

без введения:

```text
TransitionType
TernaryRelation
ActionRelation
LogicNode
```

---

# 19. Что НЕ доказано

Эксперимент не доказал, что тот же язык достаточен для:

```text
guard
precondition
quantifier
causal rule
effect
timer semantics
```

Это остаётся отдельными экспериментальными вопросами.

---

# 20. Новое важное наблюдение

MCR-02 показывает более общий принцип:

> Если объект исходной системы сам является носителем нескольких независимых отношений, он может быть представлен как Frame, даже если в исходной системе он не является «бизнес-сущностью».

`fsm_transitions` — именно такой случай.

Одна transition связывает:

```text
from state
action
to state
```

и поэтому её семантическая идентичность теряется при попытке представить её только одним бинарным Relation.

Reification восстанавливает эту идентичность:

```text
Transition Frame
```

при этом не расширяя фундаментальный язык.

---

# 21. Decision

```text
Language version:
Frame / Relation / Fact

MCR-02:
ACCEPTED

New fundamental types:
NONE

Required pattern:
Frame reification

FSM transition:
Transition-as-Frame

State:
Attribute for simple snapshot;
Frame when participating in transition relations.

Action:
Frame when its identity is semantically relevant.
```

---

# 22. Следующий эксперимент

Не нужно теперь сразу вводить `Transition` в официальный словарь типов.

Следует проверить этот же шаблон на **реальном более сложном правиле FSM**, где переход зависит не только от:

```text
from + action
```

но и от условия.

Первый кандидат:

```text
order_vote_driver_assigned
    + order_no_show
    -> order_vote_no_show
```

Затем отдельно проверить правило вида:

```text
Order completes
IF all DriverCandidacies have state FINISHED
```

Второй случай уже непосредственно связан с ранее открытым `R-2` / `OQ-001`.

Таким образом, следующий MCR должен проверять **условие**, а не снова переход.

---

# 23. Итоговая цепочка экспериментов

```text
MCR-01
Order lifecycle
    ↓
state as attribute
    ↓
PARTIAL PASS

MCR-02
Real FSM transition
    ↓
Transition-as-Frame
    ↓
PASS
    ↓
no language extension

MCR-03
Conditional / quantified transition
    ↓
?
```

На этом этапе мы по-прежнему не проектируем идеальный Semantic Graph.

Мы проверяем его выразительность на реальной системе шаг за шагом.
