> **Конвертировано из `source-docx/protostructures-methodology-ru-v1.docx`** (markitdown).
> Оригинал docx — источник истины, этот файл добавлен для читаемости в git/diff.
> Хронологически это предок `prototype-structures-v2.md`: другая целевая цепочка
> (`SOURCE FACTS → … → CONTRACTS` вместо `SOURCE → … → DECISION`).

Протоструктуры семантически-фреймового графа и методика работы с ними

# 1. Назначение документа

Документ определяет промежуточную модель хранения результатов исследования существующей системы до построения полноценного семантически-фреймового графа.

Протоструктуры предназначены для накопления фактов, семантических интерпретаций, связей, источников доказательств и уровня уверенности. Они должны позволять исследовать существующую систему без преждевременного проектирования новой архитектуры.

Основной принцип:
SOURCE FACTS → SEMANTIC FRAMES → RELATIONS → CAPABILITIES → PLATFORM CANDIDATES → CONTRACTS.

Протоструктуры являются рабочим исследовательским слоем. Они не являются окончательной архитектурой системы и не должны подменять собой исходный код, схему БД или действующие API-контракты.

# 2. Зачем нужны протоструктуры

При исследовании существующего backend информация поступает из разных источников: исходного кода, схемы БД, SQL, API, тестов, конфигурации и документации. Эти источники описывают систему с разных сторон и часто содержат противоречия.

Обычный текстовый отчёт плохо подходит для такой работы: трудно сохранять происхождение утверждений, различать факт и гипотезу, постепенно уточнять связи и автоматически сопоставлять результаты разных проходов.

Протоструктуры решают эту проблему. Каждое существенное знание сохраняется как отдельный объект с источником, типом доказательства, уровнем уверенности и связью с другими объектами.

# 3. Общая архитектура протоструктур

Рабочая модель состоит из пяти основных слоёв:

1. Source Fact — наблюдаемый факт непосредственно из источника.
2. Evidence — свидетельство, подтверждающее или опровергающее утверждение.
3. Semantic Frame — семантически интерпретированный объект с набором слотов.
4. Relation — типизированная связь между фреймами.
5. Candidate — производное утверждение о capability или потенциальной платформе.

Общий поток:

SOURCE
 ↓
FACT
 ↓
EVIDENCE
 ↓
SEMANTIC FRAME
 ↓
RELATION
 ↓
CAPABILITY
 ↓
PLATFORM CANDIDATE

Нельзя пропускать доказательный слой для существенных архитектурных выводов.

# 4. Source Fact

Source Fact — минимальная зафиксированная единица наблюдения.

Примеры:

DB:
table=cart, column=id\_user, FK→users.id\_user

DB:
site\_constant.var=ticket\_booking\_duration
site\_constant.value=691200

CODE:
models/api.php:updateCart() читает данные trip и ticket

API:
POST /api/...

Source Fact не должен содержать архитектурной интерпретации. Формулировка должна максимально близко следовать источнику.

Рекомендуемые поля:
- id;
- source\_type;
- source\_location;
- object;
- observation;
- extracted\_at;
- raw\_reference;
- status.

Source Fact считается фактом исследования только после проверки исходного материала.

# 5. Evidence

Evidence описывает, почему определённое семантическое утверждение считается подтверждённым.

Типы Evidence:

- FOREIGN\_KEY — явная связь в БД;
- TABLE\_SCHEMA — структура таблицы;
- SQL\_USAGE — использование таблицы или процедуры;
- CODE\_USAGE — использование объекта в коде;
- API\_CONTRACT — публичный API;
- TEST — подтверждение тестом;
- CONFIGURATION — значение или правило конфигурации;
- DOCUMENTATION — документация;
- RUNTIME — наблюдение реального выполнения.

Для каждого Evidence указываются:
- источник;
- конкретное место;
- наблюдение;
- связанное утверждение;
- уровень доверия.

Документация не должна автоматически иметь больший вес, чем код или БД. При противоречии источники сохраняются отдельно.

# 6. Semantic Frame

Semantic Frame представляет сущность, capability или другой объект системы через набор слотов.

Минимальная структура:

Frame
- id
- type
- name
- description
- slots
- implementation
- persistence
- configuration
- evidence
- confidence
- status

Типы фреймов на текущем этапе:

DOMAIN
CAPABILITY
ENTITY
API
COMPONENT
CONFIGURATION
EXTERNAL\_SERVICE
EVENT
FSM
PLATFORM\_CANDIDATE

Допускается расширение списка типов, если исследование обнаруживает новый устойчивый тип семантического объекта.

# 7. Slots

Slot — именованное свойство или роль фрейма.

Пример:

Frame: Cart

owner → User
product → Unknown
property → Unknown
booking\_limit → Booking constraint
session → Session

Каждый слот должен иметь:
- имя;
- значение или target frame;
- relation;
- confidence;
- evidence.

Slot может быть UNKNOWN. Не следует заполнять неизвестное значение догадкой.

Пустой слот и UNKNOWN имеют разный смысл: пустой слот означает, что свойство не установлено как часть модели; UNKNOWN означает, что свойство обнаружено, но его семантика ещё не установлена.

# 8. Relations

Связи должны быть типизированными.

Базовые типы:

- FK
- USES
- IMPLEMENTS
- EXPOSES
- PERSISTS
- READS
- WRITES
- CALLS
- CONFIGURES
- PROVIDES
- OWNS
- BELONGS\_TO
- DEPENDS\_ON
- TRIGGERS
- PRODUCES
- CONSUMES
- REFERENCES

Тип связи выбирается по смыслу источника.

Например:

cart --FK--> users

site\_constant.ticket\_booking\_duration --CONFIGURES--> TicketBooking

Cart --USES--> Ticket

Order --PERSISTS--> order

Controller --CALLS--> Service

Если связь существует только в application logic, нельзя маркировать её FK.

# 9. Confidence

Для всех существенных фреймов, слотов и связей используется уровень уверенности:

CONFIRMED
INFERRED
UNKNOWN

CONFIRMED означает, что утверждение непосредственно подтверждено источником.

INFERRED означает, что утверждение является обоснованной семантической интерпретацией нескольких фактов.

UNKNOWN означает, что необходимая информация отсутствует или пока не позволяет сделать вывод.

Пример:

Cart --OWNS--> User
CONFIRMED
source: cart.id\_user → users.id\_user

Cart --USES--> Ticket
INFERRED
source: updateCart(), selectCart(), ticket-related configuration

Нельзя повышать INFERRED до CONFIRMED только потому, что гипотеза кажется очевидной.

# 10. Frame для Capability

Capability — семантическая функция системы, которая может быть реализована несколькими компонентами.

Пример:

Capability: Driver Selection

implemented\_by:
- setDriver()
- offerOrder()
- driver selection logic

uses:
- Order
- Driver
- Location
- Rating

configured\_by:
- d\_s\_sorting\_\*
- d\_s\_offered\_drivers\_count
- d\_s\_offered\_drivers\_duration

persists:
- order\_driver
- order\_driver\_attempt
- order\_driver\_select

Capability должна выводиться из поведения системы, а не из названия класса.

# 11. Frame для Configuration

Конфигурация рассматривается как семантический объект, если она влияет на поведение системы.

Например:

ConfigurationVariable:
name = ticket\_booking\_duration
value = 691200
category = Stadium Ticket Booking

CONFIGURES → Ticket Booking

site\_constant не следует моделировать только как таблицу. В графе должны появляться сами семантически значимые configuration variables, если они влияют на бизнес-правила, безопасность, интеграции или lifecycle.

# 12. Platform Candidate

Platform Candidate — производный исследовательский объект.

Он означает только:

«В существующей системе обнаружена capability или группа capabilities, которые имеет смысл исследовать как основу будущей платформы».

Platform Candidate не означает принятого архитектурного решения.

Поля:
- name;
- based\_on;
- included\_capabilities;
- dependencies;
- external\_dependencies;
- ownership\_questions;
- unresolved\_questions;
- evidence;
- status.

Например:

Platform Candidate: Cart

based\_on:
Cart capability

dependencies:
User
Ticket
Trip
Order
Configuration

status:
RESEARCHING

До завершения исследования нельзя автоматически превращать Platform Candidate в Platform.

# 13. Методика исследования

Работа с протоструктурами выполняется итерационно.

Шаг 1. Inventory.
Составить перечень источников: код, БД, SQL, API, конфигурация, тесты, документация.

Шаг 2. Source extraction.
Извлечь минимальные Source Facts без интерпретации.

Шаг 3. Evidence registration.
Для существенных фактов указать конкретный источник.

Шаг 4. Frame construction.
Сгруппировать связанные факты в Semantic Frames.

Шаг 5. Relation extraction.
Построить типизированные связи между фреймами.

Шаг 6. Cross-source validation.
Сверить код, БД и API. При совпадении повышать уверенность; при противоречии сохранять оба наблюдения и фиксировать conflict.

Шаг 7. Capability synthesis.
Из повторяющихся и связанных действий вывести capability.

Шаг 8. Dependency analysis.
Определить зависимости capability от других capability, данных, конфигурации и внешних сервисов.

Шаг 9. Platform Candidate.
Только после этого выделять потенциальную платформу.

Шаг 10. Gap analysis.
Зафиксировать UNKNOWN и вопросы, которые требуют следующего прохода.

# 14. Правило движения от факта к семантике

Каждый переход должен быть объясним.

Неправильно:

table cart → Platform Cart

Правильно:

cart.id\_user → users.id\_user
 ↓
Source Fact
 ↓
Cart.owner → User
 ↓
Cart Frame
 ↓
Cart capability
 ↓
Platform Candidate: Cart

То же правило действует для Auth и Payment.

Название таблицы, endpoint или класса само по себе не является доказательством capability или platform boundary.

# 15. Работа с противоречиями

Если источники расходятся, исходные утверждения не удаляются.

Например:

Documentation:
Cart belongs to Order

DB:
нет FK cart → order

Code:
updateCart() участвует в формировании заказа

В графе сохраняются три Evidence.

Семантическое утверждение:

Cart --CONTRIBUTES\_TO--> Order

может иметь статус INFERRED.

Противоречие фиксируется отдельно:

CONFLICT:
documentation implies direct ownership;
DB does not enforce direct relation.

Это позволяет продолжать исследование, не превращая предположение в архитектурный факт.

# 16. Работа с UNKNOWN

UNKNOWN является нормальным состоянием протоструктуры.

Например:

Cart.product → UNKNOWN

не означает ошибку. Это означает:

- колонка product существует;
- она семантически используется;
- тип объекта, на который она ссылается, ещё не установлен.

Следующий проход должен искать места, где формируется и читается это значение.

Таким образом UNKNOWN становится рабочим указателем для дальнейшего исследования.

# 17. Версионирование

Протоструктуры должны быть версионируемыми.

Рекомендуемый цикл:

Backend Semantic Graph v0
→ v0.1
→ v0.2
→ ...

При изменении знания не следует переписывать историю молча.

Сохраняются:
- исходное утверждение;
- новое утверждение;
- причина изменения;
- новые Evidence.

Это особенно важно для confidence и архитектурных выводов.

# 18. Минимальная машинная структура

На начальном этапе достаточно следующей логической модели:

Graph
- frames[]
- relations[]
- evidence[]
- source\_facts[]
- candidates[]
- conflicts[]

Frame
- id
- type
- name
- slots[]
- evidence\_ids[]
- confidence
- status

Relation
- id
- type
- source\_frame
- target\_frame
- evidence\_ids[]
- confidence

Evidence
- id
- type
- source
- location
- observation

SourceFact
- id
- source\_type
- location
- observation

Эта модель не требует немедленного выбора графовой БД. Она может храниться в JSON/YAML/SQLite и позднее быть импортирована в полноценный Semantic Graph.

# 19. Применение к текущему Backend

Для текущего backend первыми протоструктурами уже являются:

Frames:
- User
- Authentication
- Authorization
- Order
- Driver Selection
- Driver
- Car
- Trip
- Cart
- Cart Block
- Ticket
- Schedule
- Payment
- Payment Service
- Payment Method
- Subscription
- Currency Account
- Transaction
- Deal
- Pricing
- Configuration & Business Rules

Evidence:
- DB schema
- site\_constant
- PHP API methods
- controllers
- SQL usage
- API forms/documentation

Особое внимание необходимо уделить:
- Cart → Product/Property/Trip/Ticket/Order;
- Payment → Transaction/Account/Deal;
- Authentication → User/Role/Token/SMS;
- site\_constant → business capabilities;
- Order → Trip → Ticket;
- Driver Selection → Order/Driver/Location.

# 20. Критерий зрелости протоструктуры

Протоструктура считается достаточно зрелой для перехода к следующему уровню, если:

1. существенные факты имеют источники;
2. семантические связи типизированы;
3. CONFIRMED и INFERRED не смешиваются;
4. UNKNOWN явно выделены;
5. противоречия не скрываются;
6. capability имеет реализационные доказательства;
7. зависимости capability видны;
8. Platform Candidate выводится из исследованного подграфа;
9. можно перейти от любого существенного вывода обратно к исходному коду, БД или API.

Главный критерий:

Semantic Graph должен быть способен объяснить не только «что он считает истинным», но и «почему он так считает».

# 21. Основной принцип методики

Протоструктуры не предназначены для красивого описания уже известной архитектуры.

Их назначение — фиксировать процесс восстановления семантики существующей системы.

Поэтому правильная последовательность:

наблюдение
→ факт
→ доказательство
→ фрейм
→ связь
→ capability
→ dependency
→ platform candidate.

Нельзя двигаться в обратную сторону:

название будущей платформы
→ поиск подходящих классов
→ объявление найденного подграфа платформой.

Таким образом Semantic Graph остаётся инструментом исследования, а не механизмом подтверждения заранее выбранной архитектуры.
