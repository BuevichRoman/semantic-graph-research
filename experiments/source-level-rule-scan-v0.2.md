# Semantic Graph — Core Backend Source-Level Rule Scan v0.2

**Статус:** фактический source-level scan  
**Источник:** `archive_17012026_1259_clear.zip`  
**Объект:** Core Backend snapshot из загруженного архива  
**Цель:** проверить, насколько часто в реальном PHP backend встречаются правила над множествами/коллекциями, и являются ли они одним и тем же классом выразительности.

---

## 1. Что изменилось по сравнению с v0.1

Теперь исходный PHP-корпус действительно доступен.

В архиве обнаружено:

```text
PHP files: 46
```

Базовый технический scan дал:

```text
if              4613
foreach         452
count()         108
in_array()      24
array_filter()  0
array_reduce()  0
array_intersect 0
array_diff      0
```

Это уже настоящий source-level scan, а не scan ранее извлечённых SourceFacts.

---

# 2. Важное различие

Само наличие:

```text
count()
foreach
in_array()
```

не означает quantified semantic rule.

Например:

```php
count($seat_s_b_r_s) < 4
```

может быть простой проверкой формата.

Поэтому технический индикатор является только кандидатом:

```text
PHP construct
    ↓
candidate
    ↓
semantic inspection
    ↓
только тогда SourceFact / Claim
```

---

# 3. Найденные семантически значимые случаи

## Q-01 — ALL DriverCandidacies

Это исходный известный случай:

```text
Booking completes
IF ALL related DriverCandidacy
have state 6
```

Он остаётся базовым reference case.

Важно:

этот scan не считает его новым discovery, поскольку он уже был исследован ранее.

---

## Q-03 — completeness of a requested user set

Файл:

```text
models/api.php
```

Метод:

```text
addFavoriteUser($id_user, $id_favorite)
```

В коде:

1. входной список `id_favorite` разбирается в массив;
2. вычисляется `favorite_count`;
3. SQL получает пользователей, чьи IDs входят в переданный список;
4. SQL вычисляет `COUNT(users.id_user)`;
5. backend сравнивает количество найденных пользователей с количеством запрошенных;
6. при несовпадении возвращается ошибка:

```text
only X users ... of Y found
```

Семантика:

```text
ALL requested user IDs
    must correspond to existing users
```

Это уже второй реальный случай, но он отличается от Q-01 по predicate:

```text
Q-01:
ALL related objects satisfy state predicate

Q-03:
ALL requested identifiers have corresponding objects
```

Общий механизм:

```text
set completeness
```

Но это ещё не означает, что обе семантики должны иметь один Graph primitive.

---

# 4. Q-04 — driver selection cardinality threshold

Файл:

```text
config/system_bot.php
```

В цикле выбора водителей:

```text
count($drivers[$id_order])
    ==
current_driver_count
```

При достижении количества:

```text
order_for_select_list
```

удаляется из дальнейшей обработки.

Семантика:

```text
Driver selection is complete
when accumulated driver count reaches configured target.
```

Это третий реальный класс множественной логики.

Но это НЕ:

```text
ALL drivers satisfy P
```

а:

```text
COUNT(set) == N
```

Поэтому его нельзя автоматически объединять с Q-01.

---

# 5. Q-05 — trip seat cardinality

Файл:

```text
models/api.php
```

Метод:

```text
createOrder(...)
```

В одном участке:

```text
looking_for_clients - count(trips_seats[id_trip]) == 0
```

После этого:

```text
looking_for_clients = 0
```

Семантика:

```text
required number of client/seat assignments
    has been reached.
```

Это ещё один реальный count-based rule.

Но снова:

```text
COUNT(set) == expected value
```

а не:

```text
ALL elements satisfy predicate.
```

---

# 6. Что показал source-level scan

Теперь у нас уже есть реальная группа случаев:

```text
Q-01
ALL objects satisfy predicate

Q-03
ALL requested identifiers resolve to objects

Q-04
COUNT(objects) reaches target

Q-05
COUNT(objects) reaches required quantity
```

То есть в backend действительно присутствует **устойчивый класс правил над множествами/кардинальностями**.

Но внутри него минимум три разные семантики:

```text
A. universal predicate
B. set completeness
C. cardinality threshold
```

Поэтому пока неправильно вводить единый:

```text
Quantifier
```

---

# 7. Более важный результат

Ранее мы спрашивали:

> Является ли Q-01 единичным исключением?

Теперь ответ:

```text
НЕТ.
```

Source-level scan показывает несколько независимых реальных случаев, где результат зависит от свойств множества.

Но вопрос:

> Нужен ли один общий Graph primitive?

пока остаётся:

```text
UNKNOWN
```

---

# 8. Почему нельзя пока вводить Quantifier

`ALL` в Q-01 и Q-03 выглядит одинаково только на поверхности.

### Q-01

```text
∀ candidate:
    candidate.state = 6
```

### Q-03

```text
∀ requested_id:
    exists(user(requested_id))
```

### Q-04

```text
count(selected_drivers) = target
```

### Q-05

```text
count(seats) = required
```

Q-04/Q-05 вообще не требуют universal predicate.

Следовательно:

```text
"есть операции над множествами"
        ≠
"нужен Quantifier Frame"
```

---

# 9. Что теперь действительно доказано

У нас есть достаточно Evidence для нового утверждения:

> **Set/cardinality semantics является реальным и повторяющимся классом поведения Core Backend.**

Но пока нет достаточного Evidence для утверждения:

> **Все эти случаи должны быть представлены одним новым примитивом Semantic Graph.**

Это принципиальная граница.

---

# 10. Воздействие на Graph Language

Текущая база остаётся:

```text
Frame
Relation
Fact
```

Но теперь появляется новый кандидат на **отдельный слой семантики**:

```text
Set / cardinality semantics
```

Его пока не нужно превращать в новый тип языка.

Лучше сначала проверить минимальный общий механизм.

---

# 11. Следующий MCR

Следующий эксперимент должен быть не абстрактным `Quantifier`.

Нужно взять два независимых реальных случая:

```text
Q-01
ALL candidates satisfy P

Q-03
ALL requested IDs resolve to objects
```

и попытаться выразить их одним минимальным расширением поверх:

```text
Frame / Relation / Fact
```

Вопрос:

> Можно ли представить оба случая через reification одного `Set/Collection` факта, не вводя полноценную логическую систему?

Параллельно проверить:

```text
Q-04
COUNT(set) == N
```

чтобы понять, является ли cardinality частью того же механизма или отдельной семантикой.

---

# 12. Что НЕ делать

Пока не вводить:

```text
Quantifier
LogicNode
Rule
Condition
Predicate
CollectionNode
```

как обязательные фундаментальные типы.

Сначала требуется проверить минимальное расширение.

---

# 13. Обновлённый статус исследования

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

MCR-03
ALL candidates
    ↓
FAIL

Source-Level Scan
    ↓
Q-01 confirmed
Q-03 confirmed
Q-04 confirmed
Q-05 confirmed

Conclusion:
Set/cardinality semantics is recurrent.
Common representation is still UNKNOWN.
```

---

# 14. Практический вывод

Теперь у нас появилась настоящая причина продолжать исследование структуры графа.

Но не причина проектировать её заранее.

Правильная последовательность:

```text
реальный код
    ↓
реальные set/cardinality rules
    ↓
минимальная попытка общего представления
    ↓
если работает
    → сохраняем Frame/Relation/Fact

если не работает
    ↓
MCR
    ↓
минимальное расширение
```

Это уже не теоретическое упражнение: четыре конкретных класса поведения извлечены непосредственно из загруженного Core Backend snapshot.

---

## 15. Следующий шаг

Провести **MCR-04 — Set/Cardinality Representation** на:

```text
Q-01
Q-03
Q-04
Q-05
```

Цель MCR-04:

```text
не придумать Quantifier,
а найти минимальное общее представление
для реально существующих set/cardinality semantics.
```

