# Backend Semantic Graph — Research Pass 05
# Role Resolution Trace v0.2

**Статус:** PARTIALLY ANSWERED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-05 Role Resolution Trace v0.1  
**Новый источник:** полный `archive_17012026_1259_clear.zip`, загруженный для продолжения исследования

---

## 1. Причина обновления

В RP-05 v0.1 оставался `SOURCE_GAP`:

```text
taxi::$id_role writer
check_auth_user() implementation
```

После получения полного backend archive эти два пробела были непосредственно проверены по исходному коду.

Результат принципиально меняет RP-05:

> `check_auth_user()` найден, и writer для `taxi::$id_role` найден.

Поэтому прежний вывод `SOURCE_SNAPSHOT_INCOMPLETE` для этих двух пунктов снимается.

---

# 2. Найден физический источник

Файл:

```text
taxi/models/m_functions.php
```

содержит полное определение:

```text
function check_auth_user()
```

Внутри него последовательно выполняются:

```text
1. проверка $_SESSION[UID]
2. SELECT из users
3. проверка deleted
4. get_user_ban_status()
5. обновление $_SESSION
6. получение $_SESSION['id_role']
7. установка taxi::$id_role
8. обработка request u_a_role
9. role validation через taxi::$data['user_roles']
10. подключение role-specific config
```

Это уже достаточное Evidence для существенной части Role Resolution.

---

# 3. Authentication Gate теперь полностью наблюдаем

`check_auth_user()` начинается с:

```text
if (empty($_SESSION[UID])) return false;
```

Затем выполняется запрос:

```text
SELECT
    id_role,
    id_user,
    ...
    id_verification_status,
    deleted,
    active,
    ...
FROM users
WHERE id_user = $_SESSION[UID]
LIMIT 1
```

Следовательно, `check_auth_user()` не просто проверяет наличие session cookie.

Он повторно получает состояние текущего User из `users`.

---

# 4. Account validity checks

После получения User:

```text
if (empty($d['id_user']) || !empty($d['deleted']))
{
    unset($_SESSION);
    session_unset();
}
```

Затем:

```text
$user_ban_status = get_user_ban_status($d['id_user']);
```

и:

```text
if (!empty($user_ban_status['auth']))
{
    unset($_SESSION);
    session_unset();
}
```

Следовательно, `check_auth_user()` действительно проверяет не только authentication identity, но и актуальную допустимость authenticated session.

Подтверждено:

```text
Authenticated Session
    CHECKS
User existence

Authenticated Session
    CHECKS
User deleted state

Authenticated Session
    CHECKS
Authentication-relevant ban state
```

Confidence:

```text
CONFIRMED
```

---

# 5. Writer для $_SESSION['id_role']

Внутри `check_auth_user()`:

```text
$_SESSION['id_role'] = $d['id_role'];
```

где:

```text
$d['id_role']
```

получено непосредственно из:

```text
users.id_role
```

Следовательно, подтверждена цепочка:

```text
users.id_role
    ↓
$_SESSION['id_role']
```

Confidence:

```text
CONFIRMED
```

---

# 6. Writer для taxi::$id_role

Сразу после восстановления session state:

```text
taxi::$id_role = $_SESSION['id_role'];
```

Это базовое значение API role context.

Следовательно:

```text
users.id_role
    ↓
$_SESSION['id_role']
    ↓
taxi::$id_role
```

Теперь это не inference, а непосредственно наблюдаемая code chain.

Confidence:

```text
CONFIRMED
```

---

# 7. Но на этом цепочка не заканчивается

После базового присваивания backend проверяет:

```text
if (isset($_REQUEST['u_a_role']))
```

и:

```text
$u_a_role = trim($_REQUEST['u_a_role']);
```

Затем `taxi::$id_role` может быть изменён.

Это означает, что:

```text
$_SESSION['id_role']
```

является исходной ролью текущего User,

а:

```text
taxi::$id_role
```

является runtime role context, который может быть изменён относительно исходной роли.

Это уже подтверждённая семантическая разница.

---

# 8. Role switching для session role = 4

Если:

```text
$_SESSION['id_role'] == 4
```

то:

```text
if (!empty(taxi::$data['user_roles'][$u_a_role]))
{
    taxi::$id_role = $u_a_role;
}
```

То есть роль из request:

```text
u_a_role
```

может стать:

```text
taxi::$id_role
```

если она существует в:

```text
taxi::$data['user_roles']
```

---

# 9. Role switching для session role = 6

Если:

```text
$_SESSION['id_role'] == 6
```

разрешение более узкое:

```text
$u_a_role != 4
```

и:

```text
!empty(taxi::$data['user_roles'][$u_a_role])
```

Тогда:

```text
taxi::$id_role = $u_a_role;
```

---

# 10. Role switching для остальных ролей

Для остальных исходных ролей действует:

```text
$u_a_role != 4
$u_a_role != 6
```

и:

```text
!empty(taxi::$data['user_roles'][$u_a_role])
```

Только после этих проверок:

```text
taxi::$id_role = $u_a_role;
```

---

# 11. Новая семантическая модель

Теперь подтверждённая цепочка выглядит так:

```text
users.id_role
       ↓
$_SESSION['id_role']
       ↓
base/current User Role
       ↓
taxi::$id_role
       ↑
       │
   optional u_a_role
       │
       └── validated against user_roles
```

Следовательно:

```text
u_role
```

и:

```text
u_a_role
```

в `auth_user` действительно отражают разные уровни:

```text
u_role
    = $_SESSION['id_role']

u_a_role
    = $API->id_role
```

а `$API->id_role` получает значение из runtime:

```text
taxi::$id_role
```

---

# 12. Новый вывод: API Role — не просто копия User Role

До RP-05 v0.2 было неизвестно, почему:

```text
u_role
```

и:

```text
u_a_role
```

существуют одновременно.

Теперь причина доказана:

```text
u_role
```

представляет исходную роль из User/session:

```text
users.id_role
→ $_SESSION['id_role']
```

а:

```text
u_a_role
```

представляет runtime API role:

```text
taxi::$id_role
```

которая может быть изменена через:

```text
request.u_a_role
```

при выполнении role validation.

---

# 13. Это уже Authorization-relevant mechanism

Нельзя пока говорить:

```text
taxi::$id_role = complete Authorization system
```

Но теперь подтверждено значительно более сильное утверждение:

```text
Authenticated User
    ↓
Base Role
    ↓
Runtime API Role Context
    ↓
Role-specific backend behavior
```

и runtime role выбирается не произвольно:

```text
request.u_a_role
    ↓
user_roles validation
    ↓
special restrictions based on current role
    ↓
taxi::$id_role
```

Следовательно, это уже не просто отображение данных.

Это **role-resolution / authorization-context mechanism**.

---

# 14. Role-specific configuration

После разрешения роли строится:

```text
config/list/{CONFIG}/role{taxi::$id_role}.php
```

Если файл существует:

```text
include_once($role_db_user);
```

и после этого происходит отдельное подключение к DB с параметрами:

```text
$config_arr['USERNAME']
$config_arr['PASSWORD']
```

Следовательно, runtime role влияет не только на:

```text
API->id_role
```

но и на:

```text
role-specific backend configuration
```

Это существенно для будущего Platform Candidate.

---

# 15. Role-specific implementation

Можно подтвердить следующую цепочку:

```text
Runtime API Role
      ↓
role{N}.php
      ↓
role-specific configuration
      ↓
backend runtime context
```

Но пока нельзя утверждать:

```text
role{N}.php
    = permission definition
```

Потому что конкретное содержимое role files отсутствует в текущем archive в ожидаемом пути.

Это остаётся отдельным Evidence gap.

---

# 16. Role checks внутри API

В `api.php` найдены непосредственные проверки:

```text
if ($this->id_role != 2 && $this->id_role != 4)
{
    return wrong user role;
}
```

например, в:

```text
selectProcessingOrder()
```

Также:

```text
if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
{
    return wrong user role;
}
```

в:

```text
selectArchiveOrder()
setDriver()
```

Это уже прямое Evidence:

```text
API Role Context
    ↓
Role-specific permission / access check
    ↓
Operation allowed / rejected
```

Confidence:

```text
CONFIRMED
```

---

# 17. Authorization boundary теперь виден

Теперь можно разделить:

### Authentication

```text
check_auth_user()
```

проверяет:

```text
session identity
user existence
deleted state
ban/auth state
```

### Role Resolution

```text
users.id_role
    ↓
session role
    ↓
optional requested active role
    ↓
validated runtime API role
```

### Authorization / Access Control

```text
API->id_role
    ↓
role-specific checks
    ↓
wrong user role / operation rejection
```

Это уже подтверждённая трёхступенчатая модель.

---

# 18. Authentication ≠ Role Resolution ≠ Authorization

Это один из главных результатов RP-05.

```text
AUTHENTICATION
    Кто пользователь?
        ↓
ROLE RESOLUTION
    В каком role context выполняется request?
        ↓
AUTHORIZATION
    Разрешено ли этому role context
    выполнять конкретную operation?
```

Ранее эти уровни были только гипотезой.

Теперь они подтверждены разными участками production code.

---

# 19. Обновление Semantic Graph

Новые подтверждённые Frames:

```text
User
SessionRole
RuntimeRoleContext
AuthorizationCheck
```

Не обязательно вводить все четыре как фундаментальные Frame'ы.

Для текущей протоструктуры достаточно:

```text
User
Role
AuthenticatedSession
API Role Context
```

а `AuthorizationCheck` может оставаться reified assertion / capability-level construct.

---

# 20. Новые Relations

Подтверждены:

```text
User
    HAS_ROLE
    → Role
```

```text
AuthenticatedSession
    RESOLVES_TO
    → RuntimeRoleContext
```

```text
RuntimeRoleContext
    VALIDATED_BY
    → user_roles
```

```text
ProtectedOperation
    CHECKS_ROLE
    → RuntimeRoleContext
```

```text
RuntimeRoleContext
    CONFIGURES
    → Role-specific Backend Configuration
```

---

# 21. `u_a_role` — окончательная текущая трактовка

Теперь:

```text
u_a_role
```

не следует считать просто вторым названием `u_role`.

Наиболее точная текущая модель:

```text
u_role
    = persisted/session User Role

u_a_role
    = runtime API Role Context
```

Причём runtime context может быть:

```text
u_role
```

или:

```text
validated requested role
```

в зависимости от request.

Confidence:

```text
CONFIRMED
```

для различия источников и механизма.

Термин:

```text
active role
```

пока не вводится как нормативное название, если он не используется самим проектом.

---

# 22. Важное ограничение

Нельзя пока делать следующий вывод:

```text
User can freely switch roles.
```

Код этого не подтверждает.

Наоборот, он показывает ограничения:

```text
session role = 4
    → permitted role set A

session role = 6
    → permitted role set B

other roles
    → permitted role set C
```

Следовательно, `u_a_role` является **controlled role selection**, а не произвольным параметром.

---

# 23. Research Loop

Исходный UNKNOWN:

```text
Where is taxi::$id_role initialized?
```

получил:

```text
CONFIRMED
```

Цепочка:

```text
users.id_role
→ $_SESSION['id_role']
→ taxi::$id_role
```

Но затем появился второй вопрос:

```text
Can taxi::$id_role differ from $_SESSION['id_role']?
```

Результат:

```text
CONFIRMED
```

если request содержит допустимый:

```text
u_a_role
```

---

# 24. Следующий Research Question

Теперь наиболее интересный вопрос уже другой:

```text
RQ-05-4

What exactly is contained in role{N}.php,
and what backend behavior does role-specific
configuration change?
```

Expected Evidence:

```text
config/list/{CONFIG}/role*.php
```

и все consumers соответствующих configuration values.

Второй:

```text
RQ-05-5

What is the complete role permission matrix
for protected API operations?
```

Expected Evidence:

```text
$this->id_role comparisons
wrong user role branches
role-specific methods
role configuration
```

---

# 25. Gap Report

### G-05-01 — CLOSED

```text
taxi::$id_role writer
```

Result:

```text
CONFIRMED
```

---

### G-05-02 — CLOSED

```text
check_auth_user implementation
```

Result:

```text
CONFIRMED
```

---

### G-05-03 — CLOSED

```text
u_role vs u_a_role
```

Result:

```text
CONFIRMED
```

---

### G-05-04 — PARTIALLY CLOSED

```text
actual authorization decision
```

Result:

```text
CONFIRMED
```

для конкретных API methods с `$this->id_role` checks.

Но полная permission matrix:

```text
UNKNOWN / SOURCE_GAP
```

---

### G-05-05 — OPEN

```text
role{N}.php semantics
```

Result:

```text
UNKNOWN / SOURCE_GAP
```

---

# 26. MCR result

```text
MCR = NO CHANGE
```

Существующая структура Semantic Graph v0.1 достаточна.

Не требуется новый fundamental node:

```text
ActiveRole
Permission
AuthorizationNode
```

Нужные различия выражаются существующими:

```text
Frame
Relation
Claim
Evidence
```

---

# 27. Методологический результат

RP-05 v0.2 дал важное подтверждение v2.3.

Первоначально мы были готовы объявить:

```text
SOURCE_SNAPSHOT_INCOMPLETE
```

потому что исследовали только отдельные chunks.

После получения полного physical source выяснилось, что Evidence существует.

Это показывает, почему:

```text
UNKNOWN / SOURCE_GAP
```

должен означать:

> источник реально недоступен в текущем исследовании,

а не:

> конкретный поисковый результат не показал источник.

Поэтому для будущих Pass необходимо различать:

```text
SOURCE_NOT_PROVIDED
SOURCE_NOT_FOUND_IN_SNAPSHOT
SOURCE_SEARCH_INCONCLUSIVE
```

и не сводить их в одно состояние без проверки самого snapshot.

Это предложение относится к методологии v2.3 как точечное уточнение, но **не требует нового MCR**.

---

# 28. Текущая модель Identity / Role / Authorization

```text
                     users
                       │
                  id_user/id_role
                       │
                       ↓
              check_auth_user()
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
   Session Identity           Session Role
          │                         │
          │                         ↓
          │                 $_SESSION['id_role']
          │                         │
          │                         ↓
          │                  taxi::$id_role
          │                         ↑
          │                         │
          │                 request.u_a_role
          │                         │
          │                  user_roles validation
          │
          └──────────────┐
                         ↓
               Authenticated API
                     Context
                         │
                    API->id_role
                         │
          ┌──────────────┼───────────────┐
          ↓              ↓               ↓
      method A        method B        method C
          │              │               │
      role check      role check       role check
          │              │               │
          └──────────────┴───────────────┘
                         ↓
                  ALLOW / REJECT
```

Это уже **production-backed semantic model**, а не предположение.

---

# 29. Следующий шаг

Следующим не нужно снова исследовать Authentication.

Следует перейти к:

```text
RP-06 — Role Permission Matrix
```

Цель:

```text
role
  ×
protected operation
  ↓
ALLOW / REJECT / UNKNOWN
```

Начать с уже найденных прямых проверок:

```text
selectProcessingOrder
selectArchiveOrder
setDriver
```

а затем расширять только по фактическим `$this->id_role` checks.

Параллельно отдельно исследовать:

```text
role{N}.php
```

если файлы присутствуют в следующем backend snapshot.

Только после этого будет разумно решать, является ли:

```text
Authorization
```

самостоятельной Core Capability и какова её реальная граница.
