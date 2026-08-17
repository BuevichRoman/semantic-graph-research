# Semantic Graph MCR-03 — Quantified Order Completion Rule

**Статус:** завершённый эксперимент  
**Цель:** проверить, может ли минимальный язык `Frame / Relation / Fact` выразить реальное правило над множеством объектов, не вводя заранее `Quantifier`, `LogicNode` или новый фундаментальный тип.

---

## 1. Исходный факт

В ранее проведённом исследовании зафиксировано реальное правило:

```text
Booking завершается,
если ВСЕ связанные DriverCandidacy
имеют состояние 6.
```

Это правило было отдельно проверено как `R-2 / N-007`.

Предыдущий эксперимент показал:

> разложение правила на отдельные факты не устраняет проблему квантора по переменному множеству.

Именно поэтому вопрос остаётся открытым как `R-2`. fileciteturn6file0turn6file2

MCR-03 не предполагает заранее, что этот вывод окончателен. Мы повторно проверяем его на минимальном языке, уже после MCR-02.

---

# 2. Гипотеза

Проверяем:

> Можно ли выразить правило `ALL DriverCandidacies satisfy state 6 → Booking completes` через существующие `Frame / Relation / Fact`, возможно используя уже принятый шаблон reification, без введения нового фундаментального типа `Quantifier` или `LogicNode`?

Это одна гипотеза.

Не проверяем одновременно:

```text
values
causality
permissions
guards
events
```

---

# 3. Семантическая структура правила

У правила есть как минимум следующие элементы:

```text
Booking
DriverCandidacy
State 6
Completion
```

и условие:

```text
ALL DriverCandidacy
    satisfy
State 6
```

после чего:

```text
Booking
    becomes
Completed
```

В отличие от MCR-02 здесь проблема не в представлении одного transition.

Проблема в том, что условие относится **ко всему множеству связанных объектов**.

---

# 4. Попытка №1 — разложить на обычные Facts

Представим:

```text
Booking
    HAS
DriverCandidacy A

DriverCandidacy A
    HAS_STATE
6

Booking
    HAS
DriverCandidacy B

DriverCandidacy B
    HAS_STATE
6
```

и:

```text
Booking
    HAS_STATE
Completed
```

Все отдельные факты выражаются.

Но из них не следует автоматически:

```text
ALL DriverCandidacy have state 6
```

Граф содержит:

```text
A → state 6
B → state 6
```

но не содержит оператора:

```text
∀ x ∈ DriverCandidacies
```

### Результат

```text
PARTIAL
```

Отдельные наблюдаемые факты выражаются.

Само правило — нет.

---

# 5. Попытка №2 — Relation на Booking и DriverCandidacy

Можно записать:

```text
Booking
    HAS_CANDIDACY
DriverCandidacy
```

и:

```text
DriverCandidacy
    HAS_STATE
State6
```

Но:

```text
Booking
    COMPLETES_IF
DriverCandidacy
```

не означает:

```text
Booking completes if ALL candidacies satisfy condition
```

Бинарная Relation выражает связь с конкретным объектом, но не область действия над множеством.

### Результат

```text
REJECT — insufficient semantics
```

---

# 6. Попытка №3 — Transition Frame

После MCR-02 мы можем использовать:

```text
Transition Frame
```

Например:

```text
T_complete_booking
```

и:

```text
T_complete_booking
    FROM_STATE
BookingWaiting

T_complete_booking
    ACTION
complete

T_complete_booking
    TO_STATE
Completed
```

Это корректно выражает сам transition.

Но остаётся условие:

```text
ALL DriverCandidacies
    HAVE_STATE
6
```

Transition Frame не решает эту часть.

### Результат

```text
PARTIAL
```

MCR-02 решает transition identity, но не quantified condition.

---

# 7. Попытка №4 — Condition как Frame

Можно попытаться создать:

```text
Frame:
AllCandidatesFinished
```

и:

```text
T_complete_booking
    REQUIRES
AllCandidatesFinished
```

Но тогда необходимо выразить содержание:

```text
AllCandidatesFinished
```

Внутри него снова находится:

```text
ALL
DriverCandidacy
state = 6
```

То есть мы просто спрятали нерешённую проблему в новый Frame.

### Результат

```text
REJECT
```

Причина:

> Новый Frame не решает проблему выразительности, если его семантическое содержание нельзя представить существующим языком.

Это соответствует принципу:

```text
имя не появляется до доказательства необходимости.
```

---

# 8. Попытка №5 — reification

Из предыдущего исследования известно, что reification иногда позволяет представить сложные факты.

Попробуем:

```text
Frame:
Condition_AllCandidatesFinished
```

с отношениями:

```text
Condition
    APPLIES_TO
DriverCandidacy

Condition
    REQUIRES
State6
```

Но это снова описывает:

```text
Condition applies to DriverCandidacy
```

а не:

```text
Condition applies to EVERY DriverCandidacy
```

Без отдельной семантики области действия reification не добавляет квантор.

### Результат

```text
REJECT
```

---

# 9. Попытка №6 — Relation как квантор

Можно было бы попытаться определить специальную Relation:

```text
Booking
    ALL_CANDIDACIES_FINISHED
```

Но это уже не обычная бинарная Relation между двумя Frame.

Она содержит скрытую семантику:

```text
collection
quantifier
predicate
```

То есть фактически становится специализированным логическим оператором.

Это противоречит текущей модели:

```text
Relation = typed relation between two Frames
```

### Результат

```text
REJECT
```

---

# 10. Почему MCR-02 не решает MCR-03

MCR-02 установил:

```text
Transition
    FROM_STATE → State
    ACTION     → Action
    TO_STATE   → State
```

Это решает проблему **структуры одного перехода**.

MCR-03 требует представить:

```text
Transition
    REQUIRES
    ∀ x ∈ CandidateSet:
        x.state = 6
```

Это уже не проблема количества полей Transition.

Это проблема:

```text
quantification over a set
```

Поэтому нельзя считать MCR-03 просто расширением MCR-02.

---

# 11. Результат эксперимента

Получаем:

```text
Frame / Relation / Fact
        +
Transition-as-Frame
```

позволяют выразить:

```text
Booking
    HAS
DriverCandidacy

DriverCandidacy
    HAS_STATE
State6

Transition
    FROM_STATE
...

Transition
    ACTION
complete

Transition
    TO_STATE
Completed
```

Но они не позволяют выразить единым семантическим объектом:

```text
ALL DriverCandidacy satisfy condition
```

без введения дополнительной логической семантики.

---

# 12. Decision

```text
MCR-03

Result:
FAIL — quantified rule not expressible

Fundamental language change:
NOT YET

New types introduced:
NONE
```

Важно:

`FAIL` относится к **выразимости конкретного класса правила**, а не к качеству языка в целом.

---

# 13. Что доказано

На данном эксперименте подтверждено:

```text
Frame / Relation / Fact
```

достаточны для:

```text
individual entities
individual states
individual relations
individual transitions
```

А:

```text
Transition-as-Frame
```

достаточен для:

```text
from_state
action
to_state
```

Но не для:

```text
ALL / ANY / NONE
```

над переменным множеством.

---

# 14. Связь с R-2

MCR-03 воспроизводит ранее обнаруженный:

```text
R-2
```

на том же классе реального правила.

Предыдущее исследование уже зафиксировало, что попытка свести правило множества к простому правилу между двумя Frames не удалась. fileciteturn6file0

Теперь после отдельной проверки transition representation результат остаётся тем же:

```text
Transition reification
        ≠
quantifier semantics
```

Следовательно, R-2 следует сохранить как открытый исследовательский вопрос.

---

# 15. Что НЕ следует делать

MCR-03 не даёт основания немедленно вводить:

```text
Quantifier
LogicNode
Condition
Predicate
Collection
Rule
```

Причина:

мы доказали только:

```text
текущий язык не выражает данный класс правила.
```

Но ещё не доказали:

```text
какая минимальная конструкция должна его выражать.
```

Это разные утверждения.

---

# 16. Следующая гипотеза

Теперь можно сформулировать отдельную, более узкую гипотезу:

> Возможно ли представить quantified condition через существующие Frames и Relations, если множество кандидатов и predicate условия представлены явно, без введения общего LogicNode?

Но это уже **новый MCR**, если мы решим его проверять.

Не следует объединять его с MCR-03.

---

# 17. Текущее состояние языка

После трёх экспериментов:

```text
Graph Language v0.1

Frame
Relation
Fact
```

дополнительно подтверждён рабочий паттерн:

```text
Frame reification
```

для объектов, которые сами являются носителями нескольких отношений.

### Подтверждено:

```text
State
Action
Transition
```

могут быть представлены как Frames, когда этого требует семантический вопрос.

### Не решено:

```text
Quantified conditions
Fact → Relation → Fact
Literal / arbitrary values
```

---

# 18. Итоговая цепочка

```text
MCR-01
Order lifecycle
    ↓
PARTIAL PASS

MCR-02
FSM transition
    ↓
PASS
Transition-as-Frame
    ↓
no language extension

MCR-03
ALL DriverCandidacy satisfy condition
    ↓
FAIL
    ↓
R-2 reproduced
    ↓
no language extension yet
```

Это именно тот результат, который нам нужен на текущем этапе:

```text
не придумали новый тип,
а получили эмпирическую границу
выразительности текущего языка.
```

Следующий шаг теперь не обязан быть MCR-04. Можно вернуться к реальному backend-графу и проверить, насколько часто такие quantified rules вообще встречаются за пределами уже известного примера. Это позволит не расширять язык из-за одного случая, если он не является устойчивым классом.
