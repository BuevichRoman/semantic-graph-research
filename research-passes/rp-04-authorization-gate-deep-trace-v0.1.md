# Backend Semantic Graph — Research Pass 04
# Authorization Gate Deep Trace v0.1

**Статус:** PARTIALLY ANSWERED / PROVISIONAL  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-03 Authentication Deep Trace  
**Цель:** установить семантическую роль `check_auth_user()` и `taxi::$id_role` в защищённых API-операциях и определить границу Authentication / Authorization без предположений.

---

## 1. Исследовательский вопрос

### RQ-04

> Что именно делает `check_auth_user()`, и где в Core Backend начинается собственно role-based authorization?

Цепочка:

```text
authenticated request
        ↓
check_auth_user()
        ↓
taxi::$id_role
        ↓
API->id_role
        ↓
protected operation
```

Нужно установить:

1. является ли `check_auth_user()` только authentication gate;
2. как вычисляется `taxi::$id_role`;
3. является ли `API->id_role` только контекстом текущего пользователя или частью authorization;
4. где выполняются дополнительные role/permission checks;
5. существует ли отдельная Authorization Capability.

---

# 2. Source Scope

Использованы доступные source snapshots:

```text
BACKEND-S0
DB-S0
```

Основной physical source:

```text
c_api.php
```

Также использованы ранее зафиксированные результаты:

```text
api.php
m_functions.php
Backend RP-02 Authentication
Backend RP-03 Authentication Deep Trace
Semantic_Graph_V1_Authentication_End_to_End_Crystallization
Semantic_Graph_MCR-002
```

Важное ограничение:

> В доступном текущем source corpus тело определения `check_auth_user()` не было найдено. Поэтому сам факт вызова функции исследован, а её внутренняя реализация пока не может быть подтверждена непосредственно.

Это является `SOURCE_GAP`, а не доказательством отсутствия authorization logic.

---

# 3. Первое наблюдение: check_auth_user является систематическим gate

В `c_api.php` один и тот же паттерн повторяется перед большим количеством операций:

```text
check_auth_user();
$API->id_role = taxi::$id_role;
$out = $API->...
```

Он используется, в частности, перед:

```text
/register
/remind
/newpass
/user/...
/location
/token
/cart
/cart_block
/query
/schedule
/email
/script
/translate
/outer_script
/contact
/task
/ticket
/payment
/subscription
/account
/dropbox
/deal
```

Например, `/location`:

```text
check_auth_user();
$API->id_role = taxi::$id_role;
$out = $API->setLocation(...);
```

или:

```text
check_auth_user();
$API->id_role = taxi::$id_role;
$out = $API->selectLocation(...);
```

Это непосредственно подтверждается исходным `c_api.php`. fileciteturn17file3

Для `/payment`, `/subscription` и `/account` наблюдается тот же шаблон. fileciteturn17file1

---

# 4. Что это позволяет утверждать

Можно уверенно зафиксировать:

```text
Protected API Operation
    REQUIRES
Authentication Gate
```

где implementation point:

```text
check_auth_user()
```

Confidence:

```text
CONFIRMED
```

Но нельзя пока записать:

```text
check_auth_user()
    = Authorization
```

потому что тело функции и вызываемая ею внутренняя логика отсутствуют в доступном source evidence.

---

# 5. Второе наблюдение: после gate устанавливается API->id_role

Практически в том же шаблоне:

```text
check_auth_user()
    ↓
taxi::$id_role
    ↓
$API->id_role
```

Это не единичный случай.

Например:

```text
/location
/token
/payment
/subscription
/account
/user
/contact
/task
```

используют эту последовательность. fileciteturn17file1turn17file3

Поэтому можно подтвердить:

```text
Authenticated API Context
    HAS_CURRENT_ROLE
Role
```

или более осторожно:

```text
Protected API Operation
    RECEIVES
Current User Role Context
```

Confidence:

```text
CONFIRMED
```

---

# 6. Но id_role ещё не означает authorization

Это принципиальная граница.

Из:

```text
check_auth_user()
$API->id_role = taxi::$id_role
```

следует:

```text
API operation
    knows current role
```

Но не следует:

```text
API operation
    checks permission for this role
```

И тем более не следует:

```text
Role
    EXCLUSIVELY_ALLOWED_TO
API
```

Это согласуется с ранее проведённым MCR-002: попытки выразить правило вида «доступно только X» через простые Role/API Relations не дали достаточной семантики. fileciteturn17file18

Поэтому в текущем проходе не вводится новая Relation для quantified access rules.

---

# 7. Third observation: auth_user содержит два role-поля

В `c_api.php` при формировании `auth_user` одновременно присутствуют:

```text
'u_role'   => $_SESSION['id_role']
'u_a_role' => $API->id_role
```

То есть backend различает как минимум два значения:

```text
session user role
```

и:

```text
API-level / actual role context
```

Это непосредственно видно в коде формирования `auth_user`. fileciteturn17file5turn17file12

Это важнее, чем просто наличие `id_role`.

---

# 8. Семантика двух ролей пока не установлена

Можно подтвердить только:

```text
u_role
    ←
$_SESSION['id_role']

u_a_role
    ←
$API->id_role
```

Но нельзя пока уверенно назвать:

```text
u_role = base role
u_a_role = active role
```

или:

```text
u_role = user role
u_a_role = authorization role
```

без тела `check_auth_user()` и связанных role-resolution functions.

Поэтому:

```text
C-ROLE-001
User Session
    HAS
Session Role
CONFIRMED
```

и:

```text
C-ROLE-002
API Request Context
    HAS
API Role
CONFIRMED
```

но:

```text
API Role
    IS_AUTHORIZATION_ROLE
```

остаётся:

```text
UNKNOWN / SOURCE_GAP
```

---

# 9. User registration показывает ещё одну роль-связь

При `/register` backend вызывает:

```text
registerUser(..., taxi::$data['user_roles'])
```

после:

```text
check_auth_user();
$API->id_role = taxi::$id_role;
```

Это подтверждает, что `user_roles` является configuration/data input для user registration. fileciteturn17file0turn17file11

Но:

```text
user_roles
```

не следует автоматически считать:

```text
Authorization Policy
```

Нужно найти место, где эта configuration используется для принятия permission decision.

---

# 10. `query_roles` — отдельный сильный сигнал

В `query` API присутствует:

```text
taxi::$data['site_constants']['query_roles']['value']
```

которое передаётся в:

```text
queryString(...)
```

Это особенно интересно, потому что здесь configuration явно называется:

```text
query_roles
```

и передаётся непосредственно в выполнение query-related API. fileciteturn17file13

Но даже здесь пока нельзя утверждать:

```text
query_roles
    = complete authorization model
```

Нужно исследовать тело:

```text
queryString()
```

и использование `query_roles`.

---

# 11. `session_token` и роль

Для `/cart`:

```text
check_auth_user();
$API->id_role = taxi::$id_role;

if (isset($_REQUEST['s_token']))
    $API->session_token = trim($_REQUEST['s_token']);
```

Это показывает, что:

```text
authentication gate
```

и:

```text
session_token
```

могут участвовать в одной операции, но они находятся на разных уровнях цепочки. fileciteturn17file8

Пока нельзя сделать вывод:

```text
session_token = authorization credential
```

---

# 12. `/location` как контрольный пример

Для `/location` наблюдаем:

```text
Request
   ↓
check_auth_user()
   ↓
API->id_role
   ↓
setLocation / selectLocation
```

При `selectLocation()` frontend/request может передавать:

```text
u_id
service
point
history
```

Но из `c_api.php` нельзя установить, разрешено ли текущему пользователю получать location любого `u_id`, только собственного пользователя или пользователей определённого типа.

Следовательно:

```text
Current User
    READS
Requested User Position
```

пока:

```text
UNKNOWN
```

а не CONFIRMED.

Это напрямую продолжает результат RP-02/RP-03 по `/location`.

---

# 13. Authorization candidate model

На текущем Evidence можно построить только такую модель:

```text
User
   │
   └── HAS → Session State
                │
                ├── Session Role
                ├── Verification State
                ├── Ban State
                └── Active State
                         │
                         ↓
                 check_auth_user()
                         │
                         ↓
                Authenticated API Context
                         │
                         └── HAS → API Role
                                     │
                                     ↓
                              Protected Operation
```

Эта модель:

```text
CONFIRMED
```

на уровне существования этих контекстов и переходов.

Но следующий слой:

```text
API Role
    ↓
Permission Decision
```

пока:

```text
UNKNOWN / SOURCE_GAP
```

---

# 14. Authentication / Authorization boundary

Текущий результат позволяет уточнить границу.

### Authentication

Подтверждено:

```text
Authentication
    establishes authenticated session
    validates account eligibility
    provides current authenticated user
```

### Authorization

Подтверждено пока только наличие **authorization-relevant context**:

```text
role
API role
query_roles
user_roles
```

Но реализация permission decision ещё не найдена.

Следовательно:

```text
Authentication
    ≠
Authorization
```

подтверждается как корректная исследовательская декомпозиция.

Но:

```text
Authorization Capability
    IMPLEMENTED_BY
X
```

пока не подтверждено.

---

# 15. Research Loop result

Исходный UNKNOWN:

```text
Authorization implementation boundary
```

преобразован в более точные Research Questions:

```text
RQ-04-1:
Where is check_auth_user() implemented?

RQ-04-2:
How is taxi::$id_role calculated?

RQ-04-3:
What is the semantic difference between
$_SESSION['id_role'] and $API->id_role?

RQ-04-4:
Where is user_roles interpreted?

RQ-04-5:
Where is query_roles interpreted?

RQ-04-6:
Where does the system make an actual permission decision?
```

---

# 16. Expected Evidence

Для RQ-04-1:

```text
definition of check_auth_user()
```

Для RQ-04-2:

```text
definition / assignments of taxi::$id_role
```

Для RQ-04-3:

```text
all writers/readers of:
$_SESSION['id_role']
$API->id_role
```

Для RQ-04-4:

```text
user_roles configuration
+
registerUser / editUser / role validation code
```

Для RQ-04-5:

```text
query_roles configuration
+
queryString implementation
```

Для RQ-04-6:

```text
role comparison
permission lookup
access-denied branch
policy helper
```

---

# 17. Что уже можно добавить в протоструктуру

### Claim C-AUTHZ-001

```text
Protected API Operation
    REQUIRES
Authentication Gate
```

```text
confidence: CONFIRMED
```

Evidence:

```text
CODE_USAGE
c_api.php
```

---

### Claim C-AUTHZ-002

```text
Authenticated API Context
    HAS
API Role Context
```

```text
confidence: CONFIRMED
```

Evidence:

```text
CODE_USAGE
c_api.php
```

---

### Claim C-AUTHZ-003

```text
Authentication Response
    EXPOSES
User Role
```

```text
confidence: CONFIRMED
```

Поскольку `auth_user` содержит одновременно `u_role` и `u_a_role`. fileciteturn17file12

---

### Claim C-AUTHZ-004

```text
Role Configuration
    CONFIGURES
User Registration / Role-related API behavior
```

```text
confidence: CONFIRMED
```

для `user_roles` как входного configuration/data source. fileciteturn17file0

Но это ещё не Claim:

```text
Role Configuration
    IMPLEMENTS
Authorization Policy
```

---

# 18. Что НЕ добавляем

Пока запрещено фиксировать как CONFIRMED:

```text
Role
    AUTHORIZES
API

Role
    OWNS
Permission

check_auth_user
    IMPLEMENTS
Authorization

query_roles
    IS
Authorization Policy

u_a_role
    IS
Effective Authorization Role

Driver
    CAN
Location

User A
    CAN_READ
User B Position
```

Для этих утверждений пока нет достаточного Evidence.

---

# 19. Методологический результат

RP-04 подтвердил ещё одну полезную границу v2.3:

```text
Authentication Gate
```

может быть доказан по call sites даже при недоступном теле функции.

Но:

```text
Semantic meaning of the gate
```

не должен восстанавливаться из имени функции.

То есть:

```text
check_auth_user
```

даёт:

```text
SourceFact:
protected operation invokes check_auth_user
```

но не:

```text
SourceFact:
check_auth_user performs authorization
```

Второе требует тела реализации или независимого Evidence.

Это прямое практическое подтверждение принципа:

> Имя появляется последним.

---

# 20. MCR result

Новый MCR не требуется.

Текущий язык:

```text
Frame
Relation
SemanticClaim
Evidence
ResearchQuestion
```

достаточен для фиксации результата.

Проблема не в выразительности языка.

Проблема в отсутствии части source evidence.

Следовательно:

```text
MCR = NO CHANGE
```

---

# 21. Gap Report

## G-04-01

```text
check_auth_user implementation
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
locate function definition
```

---

## G-04-02

```text
taxi::$id_role resolution
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
find all writers and initialization paths
```

---

## G-04-03

```text
u_role vs u_a_role semantics
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
trace both values through session initialization,
role resolution and API consumers
```

---

## G-04-04

```text
user_roles policy semantics
```

Status:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Next action:

```text
trace registerUser/editUser and all role-related consumers
```

---

## G-04-05

```text
query_roles semantics
```

Status:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Next action:

```text
trace queryString()
```

---

## G-04-06

```text
actual permission decision
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
search role/permission checks and access-denied branches
```

---

# 22. Итог RP-04

На текущем source corpus доказано:

```text
Protected API
    ↓
Authentication Gate
    ↓
Current API Role Context
    ↓
Protected Operation
```

и:

```text
Session Role
    ≠ necessarily
API Role
```

поскольку код явно поддерживает оба значения:

```text
u_role
u_a_role
```

Но смысл различия пока не установлен.

Главный результат:

> **Мы пока не нашли Authorization. Мы нашли authentication gate и role context, который следует за ним.**

Это более точный результат, чем предыдущая гипотеза «`check_auth_user()` возможно является Authorization».

---

# 23. Следующий шаг

Следующий pass должен быть уже не широким:

```text
RP-05 Role Resolution Trace
```

с единственной задачей:

```text
taxi::$id_role
```

Нужно найти:

```text
writer
→ source value
→ transformation
→ consumers
```

и параллельно:

```text
$_SESSION['id_role']
```

После этого можно будет установить, является ли:

```text
u_a_role
```

действительно эффективной ролью для API authorization или это другое понятие.

До этого `Authorization Capability` остаётся:

```text
RESEARCHING
```
