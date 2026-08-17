# Semantic Graph — Quantified Rule Prevalence Check v0.1

**Статус:** исследовательский проход  
**Цель:** проверить, является ли проблема quantified rules устойчивым классом в исследуемом Core Backend, а не единичным случаем `R-2`.

---

## 1. Вопрос

После MCR-03 установлено:

```text
Frame / Relation / Fact
        +
Transition-as-Frame
```

не выражают правило:

```text
ALL DriverCandidacy
    satisfy
State 6
```

Но этого недостаточно для изменения языка.

Следующий вопрос:

> Встречается ли такая форма правила в существующем backend неоднократно и в разных функциональных областях?

Если нет, расширять язык сейчас преждевременно.

---

# 2. Ограничение источников

Для этого прохода доступны результаты уже выполненного исследования Core Backend и связанные исследовательские документы.

В них подтверждены:

- Order lifecycle;
- Driver Selection;
- Authentication / Authorization;
- Cart;
- Payment / Financial;
- Configuration;
- FSM;
- ранее исследованные открытые вопросы.

Однако полный исходный PHP-архив не представлен здесь как отдельный индексируемый корпус для автоматического поиска всех условных конструкций.

Поэтому этот документ **не утверждает**, что проведён полный статический scan всех PHP `if/foreach/count/array` условий.

Мы проверяем только тот корпус фактов и правил, который уже был извлечён и зафиксирован в Research Pass и MCR.

Это принципиальное ограничение результата.

---

# 3. Найденные кандидаты quantified / exclusive rules

## Q-01 — Order completion by all DriverCandidacies

Источник:

```text
R-2 / N-007
```

Правило:

```text
Booking completes
IF ALL related DriverCandidacy
have state 6.
```

Это подтверждённый экспериментальный пример quantified rule.

Статус:

```text
CONFIRMED AS RESEARCH CASE
```

Он уже дважды исследован:

```text
предыдущий MCR
+
MCR-03
```

и не выражается текущим языком. fileciteturn8file3turn8file7

---

## Q-02 — Exclusive access / "only X"

В более раннем исследовании был найден отдельный класс:

```text
доступно только X
```

с вариантами:

```text
Role → API
Role ↔ Permission
API → Role
```

Ни один вариант не выражал семантику исключительности без потери смысла.

Этот результат зарегистрирован как:

```text
MCR-002
```

и связан с:

```text
H-D
H-MCR002-A
H-MCR002-B
H-MCR002-C
```

fileciteturn8file7

Это **не тот же самый случай**, что Q-01.

Q-01:

```text
ALL members of a set satisfy predicate
```

Q-02:

```text
ONLY members satisfying condition are allowed
```

Они похожи по форме, но исследование специально не объединяло их без отдельного доказательства.

---

# 4. Что не является новым quantified rule

Следующие обнаруженные конструкции пока не следует считать дополнительными экземплярами Q-01/Q-02.

### FSM transition

```text
from_state + action → to_state
```

Это успешно выражается через:

```text
Transition Frame
```

и не требует квантора. fileciteturn6file3

### Order lifecycle

```text
confirm
arrive
start
complete
cancel
```

Это множество отдельных операций, а не правило `ALL/ONLY`.

### Driver Selection configuration

Например:

```text
d_s_offered_drivers_count
d_s_offered_drivers_duration
```

это configuration relations:

```text
Driver Selection
    CONFIGURED_BY
    Configuration
```

а не quantified condition. fileciteturn9file1

### site_constant

Наличие большого количества configuration variables само по себе не создаёт quantified rules.

---

# 5. Текущий результат по backend

В доступном исследовательском корпусе обнаружены:

```text
1 явный экземпляр ALL-over-set:
    Q-01

1 отдельный экземпляр exclusive/ONLY semantics:
    Q-02
```

При этом нет достаточного Evidence, чтобы утверждать:

```text
"quantified rules систематически распространены
по всему Core Backend"
```

Также нет достаточного Evidence, чтобы утверждать обратное:

```text
"это единственный quantified rule backend."
```

Следовательно:

```text
PREVALENCE = UNKNOWN
```

---

# 6. Почему этого достаточно, чтобы НЕ расширять язык

Сейчас мы имеем:

```text
Q-01
    ↓
не выражается
```

и:

```text
Q-02
    ↓
не выражается
```

Но:

```text
2 разных failure cases
```

ещё не означают:

```text
нужен Quantifier
```

Потому что они могут иметь разные семантические причины.

Q-01:

```text
quantification over collection
```

Q-02:

```text
exclusive authorization semantics
```

Общий признак:

```text
оба требуют более богатой логики, чем бинарная Relation
```

но этого недостаточно для объединения их в один новый язык.

---

# 7. Состояние R-2

После этого прохода:

```text
R-2
```

остаётся:

```text
OPEN
```

но его статус уточняется:

```text
Observed on:
  real order/candidate rule

Reproduced:
  YES

Independent second backend occurrence:
  NOT YET ESTABLISHED

Language extension:
  NOT JUSTIFIED
```

То есть R-2 нельзя пока объявить системным ограничением всего backend.

---

# 8. Что делать с Q-02

Q-02 тоже остаётся отдельным исследовательским вопросом.

Нельзя писать:

```text
Q-01 + Q-02 = Quantifier
```

Следует сохранить:

```text
R-2:
ALL over collection

H-D / MCR-002:
exclusive access semantics
```

как две независимые линии исследования.

Связь между ними может появиться только после отдельного MCR.

---

# 9. Решение по Graph Language

На текущем Evidence:

```text
Graph Language v0.1

Frame
Relation
Fact
```

остаётся без изменений.

Дополнительный рабочий паттерн:

```text
Frame reification
```

подтверждён MCR-02 для Transition.

Но новый фундаментальный тип:

```text
Quantifier
LogicNode
Condition
Rule
Collection
```

не вводится.

---

# 10. Что действительно нужно исследовать дальше

Вместо нового теоретического MCR следует выполнить **targeted source scan** Core Backend.

Искать не слово `quantifier`, а реальные формы поведения:

```text
if all ...
if every ...
if none ...
if only ...
count(...) == ...
count(...) >= ...
array must satisfy ...
all candidates ...
all drivers ...
all items ...
no remaining ...
only authorized ...
only when ...
```

Но результатом scan должны быть не автоматически созданные Frames.

Для каждого найденного случая:

```text
SourceFact
    ↓
Evidence
    ↓
Semantic interpretation
    ↓
candidate rule
```

и только затем:

```text
выражается Frame / Relation / Fact?
```

---

# 11. Критерий для следующего MCR

Новый MCR оправдан, если будет найдено хотя бы несколько **независимых случаев одного и того же класса**, например:

```text
Q-01a
Order completes if ALL candidates finished

Q-01b
Trip closes if ALL required steps completed

Q-01c
Batch succeeds if ALL items processed
```

и каждый из них независимо покажет:

```text
Frame / Relation / Fact
    FAIL
```

Тогда появляется основание исследовать:

```text
common expressive gap
```

а не один конкретный бизнес-rule.

---

# 12. Важный отрицательный результат

На текущем этапе мы **не обнаружили основания превращать quantified condition в элемент Graph Language**.

Это не недостаток исследования.

Это правильный результат экспериментальной методики:

```text
failure of expression
        ≠
language extension
```

Нужно сначала установить:

```text
frequency
+
semantic commonality
+
same expressive failure
```

---

# 13. Итог

```text
MCR-03
    ↓
Q-01 не выражается

Prevalence Check
    ↓
Q-01 подтверждён
Q-02 подтверждён как отдельный класс
общая распространённость quantified rules = UNKNOWN

Decision
    ↓
Graph Language v0.1 НЕ изменять
```

Следующий практический шаг — не MCR-04, а **полный targeted scan существующего Core Backend на условные/коллекционные правила**.

Если scan выявит устойчивый класс одинаковых failure cases, тогда появится основание для нового MCR.

Если нет — R-2 остаётся локальным открытым вопросом и не должен усложнять базовый язык графа.
