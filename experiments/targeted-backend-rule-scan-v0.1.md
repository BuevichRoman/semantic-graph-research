# Semantic Graph — Targeted Backend Rule Scan v0.1

**Статус:** исследовательский проход  
**Цель:** проверить доступный корпус Core Backend на повторяемость правил, которые потенциально требуют кванторов, исключительности или иной логической семантики поверх `Frame / Relation / Fact`.

---

## 1. Зачем этот проход

MCR-03 показал конкретный failure:

```text
ALL DriverCandidacy
    satisfy
State 6
```

не выражается текущим языком.

Но одного failure недостаточно для изменения языка.

Теперь проверяем:

```text
это единичное правило
        или
устойчивый класс backend-семантики?
```

---

# 2. Важное ограничение

В текущем доступном корпусе нет исходного PHP-архива как отдельного индексируемого набора файлов, по которому можно честно выполнить полный статический поиск каждого:

```text
if
foreach
count
array_filter
in_array
...
```

Поэтому этот проход **не называется полным source scan исходного PHP**.

Он выполнен по уже извлечённым из Core Backend SourceFacts, Claims и исследовательским материалам RP-01, а также по ранее исследованным FSM/Backend материалам.

Это принципиальное ограничение:

```text
Targeted research corpus scan
        ≠
полный статический анализ всего PHP
```

Поэтому отсутствие нового случая означает:

```text
не обнаружено в доступном корпусе
```

а не:

```text
такого правила нет в backend.
```

---

# 3. Поисковая матрица

Проверялись следующие семантические классы:

```text
ALL / EVERY
NONE
ONLY
EXCLUSIVE
COUNT-based conditions
set membership
multiple related objects
conditional transition
precondition
eligibility
permission restriction
```

В качестве технических индикаторов искались/проверялись материалы вокруг:

```text
all
every
none
only
count
candidate
driver
item
remaining
authorized
allowed
forbidden
condition
check
required
eligible
```

Но техническое совпадение слова само по себе не считается finding.

Finding появляется только тогда, когда из SourceFact можно восстановить семантическое правило.

---

# 4. Результат по ALL / EVERY

## Q-01 — Order completion by all DriverCandidacies

Ранее установленный случай:

```text
Booking completes
IF ALL related DriverCandidacy
have state 6.
```

Это реальный исследованный backend rule.

Он воспроизведён MCR-03 и не выражается:

```text
Frame / Relation / Fact
```

без дополнительной логической семантики.

Статус:

```text
CONFIRMED RESEARCH CASE
```

Но независимого второго backend finding того же класса в доступном корпусе не найдено.

Следовательно:

```text
Q-01
frequency:
UNKNOWN
```

---

# 5. Результат по ONLY / EXCLUSIVE

## Q-02 — Exclusive access

В предыдущем исследовании был найден отдельный класс:

```text
only X may access Y
```

с вариантами:

```text
Role → API
Role ↔ Permission
API → Role
```

Он не выражался без потери семантики.

Статус:

```text
CONFIRMED RESEARCH CASE
```

Но это **не считается вторым экземпляром Q-01**.

Причина:

```text
Q-01:
ALL members of a set satisfy predicate

Q-02:
ONLY members satisfying condition are allowed
```

Это разные логические конструкции.

Поэтому:

```text
Q-01 + Q-02
```

не объединяются в один новый тип.

---

# 6. Order lifecycle — НЕ quantified rule

В Core Backend подтверждены отдельные операции:

```text
confirmOrder
startOrder
completeOrder
cancelOrder
setCarIsArrived
```

SourceFact `SF-ORDER-001` фиксирует эти методы и соответствующие navigation anchors. Claim:

```text
Order
    HAS_LIFECYCLE
confirm → arrive → start → complete / cancel
```

имеет статус `CONFIRMED`. fileciteturn12file1

Это не является новым quantified rule.

Здесь:

```text
A → B
B → C
C → D
```

а не:

```text
ALL X satisfy P
```

Поэтому MCR-03 на этот материал не распространяется.

---

# 7. Driver Selection — найдено условное поведение, но не Q-01

В коде подтверждено использование:

```text
d_s_sorting_city
d_s_sorting_intercity
d_s_offered_drivers_count
d_s_offered_drivers_duration
```

для расчёта поведения Driver Selection.

SourceFact `SF-CONFIG-001` связывает эти значения с `system_bot.php`, а Claims `C-CONFIG-001..003` фиксируют `CONFIGURED_BY`. fileciteturn12file4

Это:

```text
configuration-driven behavior
```

а не quantified rule.

Особенно важно не перепутать:

```text
offered_drivers_count = N
```

с:

```text
ALL offered drivers satisfy P
```

Первое — configuration/value.

Второе — quantified predicate.

---

# 8. Cart — найдено условное и множественное поведение, но не доказан Q-01

В `selectCart` подтверждены связи:

```text
cart.product → trip.id_trip
cart.product + cart.property → ticket
ticket.id_order → order
```

В `updateCart` подтверждены операции над множеством строк `cart` и использование:

```text
id_user
product
property
booking_limit
```

fileciteturn12file6turn12file9

Однако из этих SourceFacts пока нельзя вывести правило вида:

```text
ALL cart items satisfy P
```

или:

```text
ONLY cart items satisfying P are allowed
```

Следовательно:

```text
Cart
    quantified rule:
    NOT CONFIRMED
```

---

# 9. Authentication — условия есть, но это не тот же класс

`API::authUser` содержит eligibility checks:

```text
unknown user
wrong password
deleted user
authentication ban
```

и code authentication с проверкой срока действия.

SourceFact `SF-BE-AUTH-006` подтверждает эту реализацию. fileciteturn12file10

Здесь присутствуют условия:

```text
IF credential valid
IF user not deleted
IF not banned
IF code not expired
```

Но это **обычные unary/predicate conditions**, а не:

```text
ALL members of collection satisfy predicate
```

Поэтому они не являются дополнительными экземплярами Q-01.

Это важное уточнение:

```text
conditional logic
        ≠
quantified logic
```

---

# 10. Authorization / Role

В доступном корпусе есть evidence участия:

```text
role
verification state
ban state
account state
```

в eligibility Authentication.

Но полноценного подтверждённого правила вида:

```text
ONLY users with role X
may access all Y
```

в Core Backend SourceFacts этого прохода не обнаружено.

Ранее исследованный Q-02 относится к отдельному authorization experiment и остаётся отдельным исследовательским случаем.

Статус:

```text
NEW Q-02 instances:
NOT ESTABLISHED
```

---

# 11. Payment / Financial

В доступных материалах подтверждены отдельные финансовые сущности:

```text
Payment
Transaction
Currency Account
Deal
Subscription
```

и payment provider integrations:

```text
Stripe
YooKassa
```

Но доступные SourceFacts не дают нового подтверждённого правила:

```text
ALL transactions satisfy ...
ALL payment items satisfy ...
ONLY X may perform ...
```

Следовательно:

```text
Q-01 instance:
NOT FOUND

Q-02 instance:
NOT FOUND
```

Это не означает отсутствия таких правил в исходном PHP.

---

# 12. Task

Task subsystem подтверждён как отдельная область:

```text
task_list
task actions
task logs
task status
```

Но доступные материалы не содержат конкретного quantified rule, например:

```text
Task completes if ALL actions complete
```

Поэтому:

```text
Q-01:
NOT FOUND in available evidence
```

---

# 13. FSM

Реальный FSM подтверждает:

```text
from_state + action -> to_state
```

и был успешно представлен через:

```text
Transition Frame
    ├── FROM_STATE
    ├── ACTION
    └── TO_STATE
```

MCR-02 прошёл без изменения фундаментального языка.

Guards в текущем FSM являются отдельным вопросом: материалы описывают их как application/Python-layer checks, а не как уже существующую семантику SQL Core. Поэтому проектируемые `guard_name`, `effect_name` и подобные поля нельзя использовать как доказательство текущего backend rule. fileciteturn8file5turn8file13

---

# 14. Сводка обнаруженных классов

| Класс | Пример | Найден | Повторён | Требует нового языка |
|---|---|---:|---:|---:|
| ALL over collection | all DriverCandidacies state=6 | Да | Нет | Пока не решаем |
| ONLY / exclusive | only X may access Y | Да | Нет | Пока не решаем |
| Unary condition | user not banned | Да | Да, множество | Нет |
| Configuration threshold/value | offered_drivers_count | Да | Да | Нет |
| FSM transition | from + action → to | Да | Да | Нет, reification работает |
| Generic collection membership | Cart items | Да | Да | Не доказано |
| Payment conditional rule | — | Не найдено | — | — |
| Task quantified rule | — | Не найдено | — | — |

---

# 15. Что реально установлено

Доступный исследовательский корпус показывает:

```text
условная логика
        ↓
распространена
```

Но:

```text
quantified logic
        ↓
распространённость UNKNOWN
```

И отдельно:

```text
exclusive semantics
        ↓
обнаружена
```

но:

```text
повторяемость UNKNOWN
```

---

# 16. Решение по Graph Language

Никаких изменений.

```text
Graph Language v0.1

Frame
Relation
Fact
```

и:

```text
Frame reification
```

остаются достаточной минимальной основой для текущего исследуемого уровня.

Не вводим:

```text
Quantifier
LogicNode
Condition
Rule
Collection
```

---

# 17. Статус R-2

После targeted scan:

```text
R-2
status: OPEN

Observed:
YES

Reproduced:
YES

Independent second instance:
NO

Prevalence:
UNKNOWN

Language extension:
NOT JUSTIFIED
```

Это более точная формулировка, чем:

```text
R-2 = limitation of Semantic Graph
```

Пока доказано только:

```text
R-2 = limitation for one confirmed class of real rule
```

---

# 18. Что этот проход НЕ доказал

Он не доказал:

```text
в backend существует только один quantified rule;
```

и не доказал:

```text
quantified rules являются редкими.
```

Причина — исходный PHP corpus не был доступен для полного статического scan в рамках этого прохода.

Это необходимо сохранить в provenance, иначе мы превратим отсутствие найденного Evidence в ложное Evidence отсутствия.

---

# 19. Нужен ли ещё один MCR сейчас?

Нет.

Следующий шаг не должен быть:

```text
MCR-04
```

и тем более:

```text
введение Quantifier
```

Сначала нужно получить полный source-level corpus Core Backend и выполнить именно технический scan:

```text
if
foreach
for
while
count
array_filter
array_reduce
in_array
array_intersect
array_diff
empty
isset
```

плюс семантический поиск:

```text
all
every
none
only
any
remaining
eligible
allowed
forbidden
required
must
```

После этого каждый кандидат должен пройти обычную цепочку:

```text
SourceFact
    ↓
Evidence
    ↓
SemanticClaim
    ↓
Frame / Relation / Fact
    ↓
MCR only if expression fails
```

---

# 20. Итог

Текущая эмпирическая картина:

```text
                 Backend rules
                       │
          ┌────────────┼─────────────┐
          ↓            ↓             ↓
       обычные      ALL/ONLY       FSM
       conditions   rules          transitions
          │            │             │
          ↓            ↓             ↓
       выражаются    есть gaps      reification
       текущим       но частота     работает
       языком        UNKNOWN
```

Поэтому **кристаллизацию Semantic Graph пока не расширяем**.

Следующая работа должна быть не теоретической, а технической: получить полный индекс исходного Core Backend и сделать настоящий source-level rule scan. Только его результат даст основание решить, является ли `ALL/ONLY` устойчивым классом семантики графа.
