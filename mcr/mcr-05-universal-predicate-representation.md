# Semantic Graph MCR-05 — Universal Predicate Representation

**Статус:** завершённый эксперимент  
**Источник:** Core Backend snapshot `archive_17012026_1259_clear.zip`  
**Основание:** MCR-04  
**Проверяемые случаи:** Q-01, Q-03  
**Цель:** определить минимальное представление реальных universal/set-predicate rules, не вводя заранее `Quantifier`, `LogicNode` или полноценный Rule Language.

---

## 1. Исходная проблема

MCR-04 разделил обнаруженную set/cardinality semantics на две группы:

```text
A. Universal predicate

∀ x ∈ S:
    P(x)

B. Cardinality

COUNT(S) == N
```

MCR-05 исследует только группу A.

---

# 2. Реальные случаи

## Q-01 — все DriverCandidacy находятся в требуемом состоянии

Ранее исследованное правило:

```text
Booking completes
IF ALL related DriverCandidacy
have state 6.
```

Это universal predicate над связанным множеством.

---

## Q-03 — все переданные пользователи существуют

В `models/api.php::addFavoriteUser()`:

```text
favorite = explode(',', id_favorite)
favorite_count = count(favorite)
```

Затем SQL:

```text
COUNT(users.id_user)
WHERE users.id_user IN (requested IDs)
```

и:

```text
f_in_u_count != favorite_count
```

приводит к ошибке:

```text
only X users ... of Y found
```

Следовательно, фактическая семантика:

```text
ALL requested IDs
    resolve to existing users
```

Источник: `models/api.php`, функция `addFavoriteUser()`.

---

# 3. Дополнительное наблюдение Q-03

В другом участке `models/api.php`:

```text
COUNT(id_user)
WHERE id_user IN (...)
  AND id_role = 2
```

затем:

```text
u_count != count(explode(',', id_user))
```

и ошибка:

```text
user with wrong role
```

Здесь правило сильнее:

```text
ALL requested users
    EXIST
AND
ALL requested users
    HAVE_ROLE(2)
```

Это полезный вариант Q-03, потому что predicate уже не просто `exists`, а:

```text
role(user) = 2
```

---

# 4. Попытка №1 — разложение на отдельные Facts

Для Q-01:

```text
Booking
    HAS_CANDIDACY
CandidateA

CandidateA
    HAS_STATE
State6

Booking
    HAS_CANDIDACY
CandidateB

CandidateB
    HAS_STATE
State6
```

Для Q-03:

```text
RequestedSet
    HAS_MEMBER
UserA

UserA
    EXISTS_AS
User

UserA
    HAS_ROLE
Role2
```

Все отдельные Facts выражаются.

Но из графа не следует:

```text
ALL candidates have state 6
```

или:

```text
ALL requested users have role 2
```

### Результат

```text
PARTIAL
```

---

# 5. Попытка №2 — Relation `ALL_SATISFY`

Можно ввести:

```text
CandidateSet
    ALL_SATISFY
State6
```

и:

```text
RequestedUsers
    ALL_SATISFY
Role2
```

Но это не обычная бинарная Relation.

Она содержит:

```text
subject = Set
predicate = State6 / Role2
operator = ALL
```

То есть:

```text
ALL_SATISFY
```

является скрытым новым логическим оператором.

### Результат

```text
REJECT
```

---

# 6. Попытка №3 — Universal Predicate как Frame

Можно сделать:

```text
Frame:
AllCandidatesFinished
```

и:

```text
Booking
    REQUIRES
AllCandidatesFinished
```

Но содержание Frame остаётся невыраженным.

Если:

```text
AllCandidatesFinished
```

не содержит структурированного представления:

```text
ALL candidates
    state = 6
```

то это всего лишь label.

### Результат

```text
REJECT
```

---

# 7. Попытка №4 — Predicate как Frame

Можно сделать:

```text
Frame:
StateIs6
```

и:

```text
Candidate
    SATISFIES
StateIs6
```

Это уже работает для **одного** Candidate.

Например:

```text
CandidateA
    SATISFIES
StateIs6

CandidateB
    SATISFIES
StateIs6
```

Но всё ещё отсутствует:

```text
ALL Candidate
```

То есть Predicate Frame решает:

```text
P(x)
```

но не:

```text
∀x∈S P(x)
```

### Результат

```text
PARTIAL
```

---

# 8. Попытка №5 — reified assertion

Используем уже подтверждённый паттерн reification:

```text
Frame:
AllCandidatesState6
```

и создаём отношения:

```text
AllCandidatesState6
    APPLIES_TO
CandidateSet

AllCandidatesState6
    REQUIRES
State6
```

Но опять возникает вопрос:

```text
что означает APPLIES_TO CandidateSet?
```

Если это просто:

```text
CandidateSet APPLIES_TO State6
```

то universal semantics отсутствует.

Если `APPLIES_TO` подразумевает:

```text
every member
```

то квантор снова скрыт внутри Relation.

### Результат

```text
REJECT
```

---

# 9. Попытка №6 — явно представить множество

Вводим не новый фундаментальный тип, а обычный Frame:

```text
Frame:
CandidateSet
```

и:

```text
CandidateSet
    HAS_MEMBER
CandidateA

CandidateSet
    HAS_MEMBER
CandidateB
```

Predicate:

```text
CandidateA
    HAS_STATE
State6

CandidateB
    HAS_STATE
State6
```

Теперь можно визуально увидеть:

```text
CandidateSet
 ├── CandidateA → State6
 └── CandidateB → State6
```

Но утверждение:

```text
CandidateSet ALL_HAVE State6
```

по-прежнему не следует из бинарных Facts.

### Результат

```text
PARTIAL
```

---

# 10. Что именно отсутствует

После всех попыток становится ясно:

нам не хватает не объекта:

```text
Set
```

и не объекта:

```text
Predicate
```

Они оба могут быть представлены обычными Frames.

Не хватает **операции над множеством**:

```text
FOR ALL members
```

То есть проблема имеет форму:

```text
Set
  +
Predicate
  +
Scope operator
```

где:

```text
Scope operator = ALL
```

---

# 11. Минимальный возможный примитив

Если бы мы расширяли язык минимально, естественная конструкция выглядела бы концептуально:

```text
UniversalAssertion
    set = CandidateSet
    predicate = StateIs6
```

Но это пока **гипотеза**, а не принятое решение.

Она показывает, что необходимый новый элемент относится не к Entity и не к Relation, а к:

```text
assertion over a set
```

---

# 12. Можно ли выразить это существующим Relation?

Нет, если Relation остаётся:

```text
binary relation:
Frame → Frame
```

Потому что universal assertion имеет минимум:

```text
Set
Predicate
Quantifier
```

и должна сохранять их раздельно.

Запись:

```text
Set → ALL_SATISFY → Predicate
```

может быть удобной сериализацией, но семантически `ALL_SATISFY` уже становится специализированным relation/operator.

---

# 13. Можно ли решить проблему через Frame reification?

Нет.

Reification помогает, когда объект сам имеет несколько независимых отношений.

Например:

```text
Transition
 ├── FROM_STATE
 ├── ACTION
 └── TO_STATE
```

Но universal assertion имеет дополнительную семантику:

```text
scope
```

Reification может дать имя assertion, но не создать scope semantics.

---

# 14. Q-01 и Q-03 действительно имеют общий класс

Теперь это можно утверждать увереннее.

### Q-01

```text
Set = DriverCandidacies
Predicate = state == 6
Quantifier = ALL
```

### Q-03

```text
Set = RequestedUsers
Predicate = exists / role == 2
Quantifier = ALL
```

Структура одинакова:

```text
ALL
    members of S
    satisfy P
```

Различается только predicate.

Это уже достаточно сильное основание считать:

```text
Universal Predicate
```

отдельным устойчивым семантическим классом backend.

---

# 15. Но новый тип пока НЕ принимаем

MCR-05 отвечает на вопрос:

> Можно ли выразить universal predicate существующим языком?

Ответ:

```text
NO
```

Но он не отвечает окончательно:

> Как именно расширять язык?

Поэтому пока не вводим:

```text
UniversalAssertion
Quantifier
LogicNode
Rule
```

в нормативную структуру.

---

# 16. Decision

```text
MCR-05

Question:
Can universal predicate be represented by
Frame / Relation / Fact?

Result:
FAIL

Evidence:
Q-01 + Q-03

Common expressive gap:
YES

New fundamental type:
NOT YET ACCEPTED
```

---

# 17. Важное отличие от MCR-03

MCR-03 показал:

```text
один quantified rule не выражается.
```

MCR-05 теперь показывает:

```text
два независимых реальных backend rules
имеют один и тот же expressive gap.
```

Это существенное усиление Evidence.

Теперь уже можно говорить не:

```text
локальное исключение
```

а:

```text
устойчивый класс Universal Predicate.
```

---

# 18. Следующий эксперимент

Теперь имеет смысл перейти к MCR-06:

```text
Cardinality / Value Representation
```

на:

```text
Q-04
COUNT(selected drivers) == N

Q-05
COUNT(seats) == N
```

Его задача:

проверить, является ли cardinality отдельным устойчивым классом и какая минимальная семантика требуется для его представления.

После MCR-05 + MCR-06 можно будет сравнить:

```text
Universal Predicate
        vs
Cardinality Assertion
```

и только после этого решать, нужен ли им общий механизм.

---

# 19. Текущее состояние

```text
Frame
Relation
Fact
    │
    ├── Entity facts       → OK
    ├── Transition         → OK via reification
    ├── Universal Predicate→ GAP confirmed
    └── Cardinality        → GAP pending MCR-06
```

Пока структура базового графа не изменяется.

MCR-05 впервые дал достаточно сильное Evidence, чтобы считать **Universal Predicate реальным классом семантики Core Backend**, но ещё не дал оснований превращать его в конкретный новый тип графа.
