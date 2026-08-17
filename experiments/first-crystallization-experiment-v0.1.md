# Semantic Graph — First Crystallization Experiment v0.1

**Статус:** экспериментальная кристаллизация  
**Назначение:** проверить минимальный язык `Frame / Relation / Fact` на уже исследованных материалах RP-01 и RP-02.  
**Это НЕ:** финальная спецификация Semantic Graph, новая теория языка или решение Platform Architecture.

---

## 1. Цель эксперимента

У нас уже есть:

```text
RP-01
Core Backend
    ↓
Backend semantic findings

RP-02
Taxi Web v0.1.20
    ↓
Client semantic findings
```

Теперь не проектируем структуру графа сверху.

Вместо этого берём реальные утверждения из этих двух проходов и пытаемся выразить их минимальным языком:

```text
Frame
Relation
Fact
```

Именно такой подход соответствует ранее проведённому исследованию языка: новое понятие вводится только после того, как реальный факт невозможно выразить существующим языком. fileciteturn3file0

---

# 2. Исходный минимальный язык

## Frame

Самостоятельная сущность, которая участвует хотя бы в одной независимой семантической связи.

Всё, что не требует самостоятельной связи, остаётся атрибутом.

## Relation

Типизированная связь между двумя Frame.

## Fact

Триплет:

```text
(subject Frame, Relation, object Frame)
```

Эта модель была эмпирически проверена на реальных артефактах и показала, что операционные локальные факты вроде:

```text
API — creates → BusinessEntity
API — updates → DatabaseField
```

в большинстве случаев выражаются без семантической натяжки. fileciteturn3file4

---

# 3. Правило эксперимента

Для каждого реального finding:

```text
Finding
   ↓
выбрать Frame subject
   ↓
выбрать Frame object
   ↓
выбрать существующий Relation
   ↓
получился Fact?
```

Если да:

```text
ACCEPTED
```

Если нет:

```text
REJECT-Frame
или
REJECT-Relation
```

Если появляется соблазн добавить новый тип:

```text
не добавлять его сразу
        ↓
зафиксировать конкретную невозможность
        ↓
проверить альтернативные выражения
        ↓
только затем MCR
```

Это соответствует замороженной дисциплине исследования языка. fileciteturn3file3

---

# 4. Первый набор Frames

Из RP-01 и RP-02 без введения новых понятий можно выделить:

```text
Core Backend
Taxi Web v0.1.20

Authentication
User
Order
Trip
Car
Car Class

Configuration
Referral
File Storage

Geolocation
Geocoding
Routing
Service Area

Communication
Payment
Candidate Selection

External Chat Service
OpenRouteService
Nominatim
HERE
jecat.ru
```

Не утверждается, что это окончательный словарь Frame.

Это только минимальный набор, необходимый для проверки уже обнаруженных связей.

---

# 5. Authentication — первый тест

Из RP-02:

```text
Taxi Web Authentication
    CALLS
Core Backend /auth
```

и:

```text
Core Backend
    IMPLEMENTS
Authentication
```

Преобразуем в Frame/Relation/Fact:

```text
F1:
Taxi Web v0.1.20
    CALLS
Authentication API

F2:
Core Backend
    IMPLEMENTS
Authentication
```

Если `/auth` считать отдельным Frame:

```text
F3:
Taxi Web v0.1.20
    CALLS
/auth

F4:
Core Backend
    IMPLEMENTS
/auth
```

Но здесь возникает вопрос уровня моделирования.

`/auth` может быть:

```text
самостоятельной семантической сущностью
```

или:

```text
атрибутом / интерфейсной деталью Authentication.
```

Для первого эксперимента не нужно решать это теоретически.

Берём более устойчивый вариант:

```text
Taxi Web
    CALLS
Authentication

Core Backend
    IMPLEMENTS
Authentication
```

---

# 6. Сквозная цепочка Authentication

Из RP-02 подтверждена цепочка:

```text
Taxi Web
    CALLS
Authentication

Core Backend
    IMPLEMENTS
Authentication

Authentication
    USES
User

Authentication
    USES
Configuration

Taxi Web
    USES
token + u_hash
```

Последнее утверждение пока не является хорошим Frame/Relation/Frame фактом:

```text
token + u_hash
```

не обязательно самостоятельная семантическая сущность.

По правилу Frame/attribute его пока разумно оставить атрибутом/деталью Authentication interaction.

**Результат:** минимальный язык проходит этот участок без расширения.

---

# 7. User

Из RP-02:

```text
Taxi Web
    USES
Core Backend User Management
```

В минимальном языке:

```text
Taxi Web
    USES
User
```

и:

```text
Authentication
    USES
User
```

Оба факта выражаются.

Не требуется вводить:

```text
UserProfile
AccountState
AuthenticationUser
```

только потому, что frontend использует разные поля User.

Это соответствует правилу:

> имя появляется последним; сущность не выделяется в отдельный Frame без независимой семантической связи. fileciteturn3file16

---

# 8. Order

RP-02 показывает:

```text
Taxi Web
    CREATES
Order

Taxi Web
    READS
Order

Taxi Web
    UPDATES
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
Car Class

Order
    USES
Geolocation

Order
    USES
Payment
```

Большинство этих утверждений непосредственно выражаются минимальным языком.

Особенно важно:

```text
Order
    USES
Payment
```

не означает:

```text
Order OWNS Payment
```

Это сохраняет текущую неопределённость границы Payment.

---

# 9. Driver / Candidate

Из RP-02:

```text
Candidate Selection UI
    USES
Core Backend Driver Candidate capability
```

В минимальном языке:

```text
Taxi Web
    USES
Candidate Selection

Order
    USES
Candidate Selection
```

Это выражается.

Но более сильное утверждение:

```text
Order
    HAS
CandidateSet
```

пока не требуется.

---

# 10. Vehicle

Подтверждено:

```text
Taxi Web
    USES
Vehicle Management

Vehicle Management
    USES
Car

Car
    USES
Car Class
```

В минимальном графе это можно представить:

```text
Taxi Web
    USES
Car

Car
    USES
Car Class
```

Если `Vehicle Management` не является самостоятельной бизнес-сущностью, его можно не делать Frame.

Это хороший пример того, как минимальный язык не заставляет нас превращать каждый capability name в узел.

---

# 11. Trip

RP-02 подтверждает:

```text
Taxi Web
    USES
Trip

Trip
    USES
Order
```

Минимальный язык выражает это непосредственно.

Пока нет необходимости вводить:

```text
TripOrderRelation
TripAggregate
TripLifecycle
```

---

# 12. Configuration

Есть:

```text
Taxi Web
    CONFIGURED_BY
Core Backend Configuration
```

и:

```text
Authentication
    USES
Configuration
```

Оба факта выражаются.

При этом отдельные значения `site_constant` или runtime configuration пока не становятся Frame автоматически.

Например:

```text
auth_code_interval
```

может оставаться значением/атрибутом Configuration.

Это особенно важно после исследования `site_constant`: наличие настройки само по себе не доказывает активное поведение.

---

# 13. External integrations

RP-02 подтверждает:

```text
Taxi Web
    USES
Nominatim

Taxi Web
    USES
HERE

Taxi Web
    USES
OpenRouteService

Taxi Web
    USES
External Chat Service

Taxi Web
    USES
jecat.ru
```

Минимальный язык справляется.

Никаких новых типов:

```text
ExternalService
Integration
Provider
```

пока не требуется.

Если различие между внешним сервисом и внутренним компонентом потребуется для конкретного вопроса, это станет отдельным экспериментом.

---

# 14. Files

Подтверждено:

```text
Taxi Web
    USES
Core Backend File Storage

User
    USES
File Storage
```

Минимальный язык выражает оба факта.

---

# 15. Referral

Подтверждено:

```text
Taxi Web
    USES
Referral
```

и:

```text
User
    USES
Referral
```

Минимальный язык проходит.

---

# 16. Payment — намеренно неполный результат

Frontend содержит:

```text
Cash
Credit
PayPal

Card UI

b_payment_way
b_payment_card
b_payment_sum
b_payment_datetime
b_tips
```

Но RP-02 не подтвердил активное использование Taxi Web соответствующих backend Payment API.

Поэтому нельзя строить:

```text
Taxi Web
    USES
Payment Service
```

как CONFIRMED fact.

Можно зафиксировать:

```text
Taxi Web
    USES
Payment Representation
```

но это уже требует решить, является ли `Payment Representation` самостоятельным Frame или клиентским атрибутным слоем.

Для первого эксперимента лучше не вводить новый Frame.

**Статус:** UNKNOWN / не кристаллизовать.

---

# 17. Где минимальный язык начинает испытываться

До этого момента большинство обнаруженных фактов имеют форму:

```text
A
  ↓
Relation
  ↓
B
```

Это именно тот класс фактов, на котором минимальный язык был проверен ранее. fileciteturn3file0

Но RP-01/RP-02 содержат более сложные конструкции.

---

# 18. Первый потенциальный разрыв: lifecycle

Frontend содержит:

```text
Processing
Approved
Canceled
Completed
PendingActivation
OfferedToDrivers
```

и:

```text
Considering
Performer
Arrived
Started
Finished
```

Можно представить:

```text
Order
    HAS_STATE
Processing
```

если State является Frame.

Но тогда возникают вопросы:

```text
Processing
    →?
Approved
    →?
Canceled
```

Нужно ли моделировать переход:

```text
Processing
    TRANSITIONS_TO
Approved
```

или достаточно хранить состояние как атрибут Order?

Пока это **не основание для изменения языка**.

Причина: текущий вопрос не требуется для ответа на основной вопрос RP-02.

Статус:

```text
OPEN MODELING QUESTION
```

не MCR.

---

# 19. Второй потенциальный разрыв: действия

Frontend имеет операции:

```text
createOrder
cancelOrder
takeOrder
setOrderState
setOrderRating
setWaitingTime
setTips
```

Их можно представить как:

```text
Taxi Web
    CALLS
Order API
```

Но иногда хочется выразить:

```text
cancelOrder
    CHANGES
Order state
```

Здесь субъектом становится действие/операция.

Возникает тот же класс проблемы, который уже был замечен в предыдущем исследовании:

```text
Fact ↔ Fact
```

или:

```text
Operation
    changes
State
```

Пока достаточно первого варианта:

```text
Taxi Web
    CALLS
Order API
```

Поэтому новый Frame типа `Operation` **не вводится**.

---

# 20. Третий потенциальный разрыв: причинность

RP-01/RP-02 содержат правила, где хочется выразить:

```text
изменение A
    вызывает
изменение B
```

Предыдущее исследование уже показало:

```text
Fact → Relation → Fact
```

как отдельную нерешённую проблему.

Это зафиксировано как:

```text
OQ-001
```

и было воспроизведено на независимом правиле. fileciteturn3file3

Поэтому RP-01/RP-02 не должны самостоятельно вводить `causes`, `LogicNode` или `FactNode`.

---

# 21. Четвёртый потенциальный разрыв: значения

Пример:

```text
polling interval = 2000 ms
```

или:

```text
auth_code_interval = N
```

Минимальный язык:

```text
Taxi Web
    CONFIGURED_BY
Configuration
```

может выразить наличие зависимости.

Но:

```text
Configuration
    HAS_VALUE
2000
```

требует решения о том, является ли literal самостоятельным объектом графа.

Это уже совпадает с ранее открытым:

```text
R-6
R-7
OQ-002
```

которые пока не решены. fileciteturn3file2

Поэтому новый `Value` Frame сейчас не вводится.

---

# 22. Пятый потенциальный разрыв: множество

Frontend содержит:

```text
Order
    ↓
multiple Drivers
    ↓
multiple Candidates
```

Можно выразить:

```text
Order
    USES
Driver
```

Но утверждение:

```text
Order completes when ALL DriverCandidacies
have state 6
```

не выражается простым бинарным Fact.

Это уже было обнаружено на независимом эксперименте `N-007`.

Статус:

```text
R-2 / OQ
```

остаётся открытым. fileciteturn3file3

---

# 23. Результат первого прогона

На реальных материалах RP-01 + RP-02 минимальный язык:

```text
Frame
Relation
Fact
```

покрывает значительную часть верхнеуровневых семантических связей:

```text
Client → uses → Backend Capability
Backend → implements → Capability
Capability → uses → Entity
Entity → uses → Entity
Client → uses → External Service
```

Это хороший результат.

Но это НЕ доказывает, что минимальный язык является достаточным для полного Semantic Graph.

---

# 24. Что пока НЕ добавляем

По результатам этого прогона нет основания вводить:

```text
Actor
Role
Operation
Event
LogicNode
ValueNode
Quantifier
Condition
ExternalService
Integration
State
FactNode
```

Некоторые из них могут оказаться нужными.

Но для каждого требуется отдельный реальный случай, который не выражается текущим языком.

Это соответствует правилу исследования:

```text
не нравится выражение
        ≠
нужен новый тип
```

и:

```text
похожи две проблемы
        ≠
это одна проблема
```

Именно такая дисциплина ранее предотвратила преждевременное введение `LogicNode` и смешение OQ-001/OQ-002. fileciteturn3file16

---

# 25. Предварительная структура Graph v0.1

Пока достаточно:

```text
Graph
│
├── Frames
│
│   ├── system
│   ├── client
│   ├── capability
│   ├── domain entity
│   └── external entity
│
└── Facts
    ├── subject
    ├── relation
    └── object
```

При этом:

```text
Evidence
SourceFact
ResearchPass
Confidence
```

не являются дополнительными элементами минимального **языка предметного графа**.

Они остаются provenance / research layer, который сопровождает Facts.

Это различие принципиально:

```text
Research Model
        ↓
дает доказательство
        ↓
Graph Model
        ↓
дает семантическое представление
```

---

# 26. Кристаллизация пока НЕ завершена

После первого прогона можно зафиксировать только:

```text
Graph Language v0.1

Frame
Relation
Fact
```

как минимальное рабочее ядро.

Нельзя пока фиксировать:

```text
final Frame taxonomy
final Relation taxonomy
final State model
final Value model
final Event model
final Logic model
```

поскольку реальные эксперименты этого ещё не доказали.

---

# 27. Следующий эксперимент

Следующий шаг должен быть не написанием большой спецификации.

Нужно взять один конкретный сложный фрагмент реального backend/frontend, где текущий язык показывает:

```text
REJECT-Frame
```

или:

```text
REJECT-Relation
```

и провести один MCR.

Предпочтительный кандидат:

```text
Order lifecycle
```

потому что он одновременно содержит:

```text
state
transition
action
condition
multiple drivers
```

Но эксперимент должен проверять **один вопрос**, а не всё сразу.

Например:

> Можно ли выразить переход состояния Order через существующие `Frame / Relation / Fact`, не вводя новый тип `State` или `Transition`?

Только если ответ будет отрицательным после исчерпания альтернатив, появляется основание для MCR.

---

# 28. Итог

Первый практический вывод:

```text
Мы не нуждаемся сейчас в новой большой теории Semantic Graph.
```

У нас уже есть минимальный язык:

```text
Frame
Relation
Fact
```

и реальный материал, на котором его можно продолжать проверять.

Поэтому следующий режим работы:

```text
реальный backend/frontend факт
        ↓
Frame / Relation / Fact
        ↓
получилось?
   ├── да → следующий факт
   │
   └── нет
        ↓
     альтернативы
        ↓
     MCR только при реальной необходимости
```

Это позволяет двигаться от существующей системы к структуре графа, не возвращаясь к теоретическому проектированию графа «на будущее».
