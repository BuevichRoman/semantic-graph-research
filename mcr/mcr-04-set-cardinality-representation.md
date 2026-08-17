# Semantic Graph MCR-04 — Set / Cardinality Representation

**Статус:** завершённый эксперимент  
**Основание:** source-level scan `Core Backend` из `archive_17012026_1259_clear.zip`  
**Проверяемый материал:** Q-01, Q-03, Q-04, Q-05  
**Цель:** найти минимальное общее представление для реально обнаруженных set/cardinality semantics, не вводя заранее `Quantifier`, `LogicNode`, `Rule`, `CollectionNode` или другую новую фундаментальную сущность.

---

## 1. Исходные реальные случаи

Source-level scan обнаружил несколько независимых конструкций.

### Q-01 — ALL DriverCandidacies

Ранее исследованное правило:

```text
Booking completes
IF ALL related DriverCandidacy
have state 6.
```

Это universal predicate над множеством.

---

### Q-03 — ALL requested users exist and have the required role

В:

```text
models/api.php::addFavoriteUser()
```

входной список `id_favorite` превращается в массив, затем SQL получает количество пользователей, чьи IDs входят в переданный список:

```text
COUNT(users.id_user)
WHERE users.id_user IN (requested IDs)
```

и backend сравнивает:

```text
found_count == requested_count
```

При несовпадении запрос отклоняется.

Кроме того, в другом участке `models/api.php` есть аналогичная конструкция:

```text
COUNT(users.id_user)
WHERE id_user IN (requested ids)
  AND id_role = 2
```

после чего:

```text
u_count == count(requested ids)
```

Иначе:

```text
user with wrong role
```

Это уже не просто существование объектов.

Семантика:

```text
ALL requested IDs
    resolve to users
AND
ALL resolved users
    satisfy role = 2
```

Это реальный source-level case.

---

### Q-04 — driver selection cardinality

В:

```text
config/system_bot.php
```

выбор водителей прекращается при:

```text
count(selected_drivers) == current_driver_count
```

Это:

```text
COUNT(S) == N
```

---

### Q-05 — trip seat cardinality

В:

```text
models/api.php
```

есть:

```text
looking_for_clients - count(trips_seats[id_trip]) == 0
```

что приводит к:

```text
looking_for_clients = 0
```

Это также:

```text
COUNT(S) == N
```

---

# 2. Вопрос MCR-04

Проверяем:

> Можно ли представить Q-01, Q-03, Q-04 и Q-05 одним минимальным расширением существующего языка `Frame / Relation / Fact`, используя уже известный паттерн reification?

Не требуется найти идеальную будущую модель.

Требуется установить, существует ли **одно простое представление**, которое уже сейчас покрывает реальные случаи.

---

# 3. Попытка №1 — Set как Frame

Представим множество самостоятельным Frame:

```text
Frame:
  RequestedUsers

Frame:
  DriverCandidacies

Frame:
  SelectedDrivers
```

и связываем элементы:

```text
RequestedUsers
    HAS_MEMBER
UserA

RequestedUsers
    HAS_MEMBER
UserB
```

или:

```text
SelectedDrivers
    HAS_MEMBER
DriverA

SelectedDrivers
    HAS_MEMBER
DriverB
```

Все отдельные membership facts выражаются существующим языком:

```text
Frame
    Relation
Frame
```

### Но

Q-01 требует:

```text
∀ x ∈ DriverCandidacies:
    x.state = 6
```

Q-03 требует:

```text
∀ x ∈ RequestedUsers:
    exists(User(x))
```

и:

```text
∀ x ∈ RequestedUsers:
    role(User(x)) = 2
```

Сам факт:

```text
Set
    HAS_MEMBER
Member
```

не содержит семантики:

```text ALL members satisfy P
```

### Результат

```text
PARTIAL
```

Set-as-Frame помогает представить множество, но не выражает universal predicate.

---

# 4. Попытка №2 — Set Frame + Relation to Predicate Frame

Попробуем:

```text
DriverCandidacies
    ALL_SATISFY
State6
```

или:

```text
RequestedUsers
    ALL_HAVE_ROLE
Role2
```

Проблема:

`ALL_SATISFY` уже не является обычной Relation в смысле текущего языка.

Она кодирует:

```text
quantifier
+
predicate
+
scope
```

То есть мы просто спрятали новый логический примитив в имя Relation.

Это нарушает правило минимальности.

### Результат

```text
REJECT
```

---

# 5. Попытка №3 — Reification

Создадим:

```text
Frame:
AllCandidatesFinished
```

и:

```text
Transition
    REQUIRES
AllCandidatesFinished
```

Аналогично:

```text
Frame:
AllRequestedUsersValid
```

Но содержимое этих Frames всё равно требует:

```text ALL members satisfy predicate
```

Если внутреннее содержание нельзя выразить текущим языком, reification ничего не добавляет.

Получается:

```text
opaque semantic label
```

а не структурированное знание.

### Результат

```text
REJECT
```

---

# 6. Попытка №4 — Cardinality как Set Frame

Для Q-04:

```text
SelectedDrivers
    HAS_MEMBER
DriverA
SelectedDrivers
    HAS_MEMBER
DriverB
SelectedDrivers
    HAS_MEMBER
DriverC
```

Нужно выразить:

```text COUNT(SelectedDrivers) == 3
```

Можно ввести:

```text
SelectedDrivers
    HAS_COUNT
3
```

Но `3` не является Frame.

В текущем языке:

```text
Fact = Frame → Relation → Frame
```

Поэтому scalar literal нельзя использовать как object без расширения модели.

### Результат

```text
FAIL
```

---

# 7. Попытка №5 — Cardinality value как Frame

Можно искусственно сделать:

```text
Frame:
Count3
```

и:

```text
SelectedDrivers
    HAS_COUNT
Count3
```

Но тогда:

```text
Count3
```

становится псевдо-сущностью.

Это создаёт отдельный Frame только ради значения:

```text
3
```

и не решает общий вопрос:

```text
3
4
5
N
```

### Результат

```text
REJECT
```

Это нарушение правила:

> атрибут не превращается в самостоятельный Frame без собственной семантической роли.

---

# 8. Сопоставление четырёх случаев

| Case | Семантика | Set нужен | Universal predicate | Cardinality | Scalar value |
|---|---|---:|---:|---:|---:|
| Q-01 | ALL candidates state=6 | Да | Да | Нет | Нет |
| Q-03 | ALL requested IDs resolve + role=2 | Да | Да | Косвенно | Нет |
| Q-04 | selected drivers count=N | Да | Нет | Да | Да |
| Q-05 | seats count=N | Да | Нет | Да | Да |

Это важный результат.

Q-01/Q-03 и Q-04/Q-05 действительно образуют две устойчивые подгруппы:

```text
A. Universal / set predicate
B. Cardinality / scalar comparison
```

Они не являются одной и той же операцией.

---

# 9. Попытка найти одно общее расширение

Можно предложить абстрактную конструкцию:

```text
SetRule
    scope = Set
    predicate = Predicate
    operator = ALL / COUNT / ...
    value = ...
```

Но это уже фактически новый мини-язык правил.

Например:

```text
SetRule(
    set = DriverCandidacies,
    operator = ALL,
    predicate = state == 6
)
```

или:

```text
SetRule(
    set = SelectedDrivers,
    operator = COUNT_EQ,
    value = 3
)
```

Такое решение **не следует из MCR-04**.

Оно лишь показывает, что мы можем описать проблему, если заранее введём новый Rule language.

Следовательно, его нельзя принимать как результат эксперимента.

---

# 10. Результат MCR-04

```text
MCR-04

Result:
FAIL — no single minimal representation found

Current language:
Frame / Relation / Fact

Reification:
useful, but insufficient

New fundamental type:
NOT INTRODUCED
```

То есть:

```text
Set/cardinality semantics
```

действительно является повторяющимся классом backend-поведения.

Но он распадается минимум на:

```text
Universal predicate
+
Cardinality/value comparison
```

и для обоих текущего языка недостаточно.

---

# 11. Что именно доказано

### Доказано

1. Множества реально присутствуют в Core Backend.
2. Membership может быть представлен бинарными Facts.
3. Universal predicate над множеством не выражается текущим языком.
4. Cardinality comparison не выражается текущим языком.
5. Scalar value не следует превращать в Frame.
6. Один универсальный `Quantifier` пока не является доказанным минимальным решением.
7. `LogicNode` также не является доказанным решением.

---

# 12. Что НЕ доказано

Не доказано:

```text
нужен Quantifier;
```

не доказано:

```text
нужен Rule;
```

не доказано:

```text
нужен CollectionNode;
```

не доказано:

```text
нужна отдельная Value Node;
```

Также не доказано, что universal predicate и cardinality должны иметь общий механизм.

---

# 13. Важный результат для структуры Semantic Graph

Теперь появилась первая настоящая граница между:

```text
Entity / Relation graph
```

и:

```text
Rule / Logic semantics
```

Текущий минимальный граф хорошо представляет:

```text
A
  └── relation → B
```

и через reification:

```text
A
  └── relation → R
R
  ├── relation → B
  ├── relation → C
  └── relation → D
```

Но не умеет выражать:

```text
FOR ALL x IN S:
    predicate(x)
```

или:

```text
COUNT(S) == N
```

без дополнительной семантики.

---

# 14. Следующий шаг — разделить исследования

Не делать MCR-05 «Quantifier».

Нужно провести два маленьких независимых эксперимента.

### MCR-05 — Universal Predicate

Только:

```text
Q-01
Q-03
```

Вопрос:

> Как минимально представить `ALL members satisfy predicate`, если вообще нужно расширение?

### MCR-06 — Cardinality

Только:

```text
Q-04
Q-05
```

Вопрос:

> Как минимально представить `COUNT(S) == N`, не превращая literals в Frames?

Это позволит не смешать две разные проблемы в одну теорию.

---

# 15. Решение по текущему Graph Language

Пока сохраняем:

```text
Frame
Relation
Fact
```

и:

```text
Frame reification
```

как рабочую основу.

Не добавляем:

```text
Quantifier
Rule
LogicNode
CollectionNode
ValueNode
```

---

# 16. Почему это не возврат к теоретизированию

MCR-04 не проектирует язык.

Он сделал ровно то, что должен был сделать:

```text
реальный код
    ↓
4 реальных правила
    ↓
попытка общего представления
    ↓
обнаружена общая граница
    ↓
граница разделена на две конкретные проблемы
```

То есть следующий шаг снова может быть привязан к реальному коду:

```text
MCR-05 → реальные ALL rules
MCR-06 → реальные COUNT rules
```

а не к абстрактной модели графа.

---

# 17. Итог

```text
Source Scan
    ↓
Q-01 Q-03 Q-04 Q-05
    ↓
MCR-04
    ↓
единое решение не найдено
    ↓
разделяем:
    ├── Universal Predicate
    └── Cardinality / Value
```

**Фундаментальный язык пока не меняется.**

Это текущая наиболее сильная эмпирическая граница, полученная непосредственно из исходного Core Backend.
