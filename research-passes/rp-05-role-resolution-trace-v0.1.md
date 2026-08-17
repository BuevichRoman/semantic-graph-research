# Backend Semantic Graph — Research Pass 05
# Role Resolution Trace v0.1

**Статус:** PARTIALLY ANSWERED / SOURCE-GAP  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-04 Authorization Gate Deep Trace  
**Цель:** проследить происхождение `taxi::$id_role` и сопоставить его с `$_SESSION['id_role']`, `u_role` и `u_a_role`.

---

## 1. Исследовательский вопрос

### RQ-05

Установить:

```text
taxi::$id_role
    ↓
writer
    ↓
source value
    ↓
transformation
    ↓
consumer
```

и отдельно:

```text
$_SESSION['id_role']
    ↓
writer
    ↓
consumer
```

Главный вопрос:

> Что именно означает `API->id_role`, и почему оно может отличаться от `$_SESSION['id_role']`?

---

# 2. Source Scope

Используются:

```text
BACKEND-S0
DB-S0
```

Backend snapshot:

```text
archive_17012026_1259_clear (1).zip

sha256:
e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
```

Ранее подтверждённые physical sources:

```text
taxi/controllers/c_api.php
taxi/models/api.php
taxi/models/m_functions.php
```

DB snapshot:

```text
aristotel_taxi.sql (1).zip

sha256:
501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
```

---

# 3. Уже подтверждённый участок цепочки

`c_api.php` многократно содержит:

```text
check_auth_user();
$API->id_role = taxi::$id_role;
$out = $API->...
```

Это подтверждено для большого числа защищённых операций, включая:

```text
/location
/token
/cart
/user
/car
/task
/ticket
/payment
/subscription
/account
/deal
/dropbox
```

Например:

```text
/location
    check_auth_user()
    $API->id_role = taxi::$id_role
    $API->setLocation(...)
```

и:

```text
/token
    check_auth_user()
    $API->id_role = taxi::$id_role
    $API->selectToken(...)
```

fileciteturn20file15

Для `cart` одновременно устанавливается:

```text
$API->id_role = taxi::$id_role
```

и при наличии request parameter:

```text
$API->session_token = $_REQUEST['s_token']
```

fileciteturn20file16

Следовательно, подтверждён:

```text
Authentication Gate
    ↓
API Role Context
    ↓
Protected Operation
```

Но происхождение `taxi::$id_role` пока не подтверждено.

---

# 4. Два role-представления действительно существуют

При формировании `auth_user` backend возвращает одновременно:

```text
'u_role'   => $_SESSION['id_role']
'u_a_role' => $API->id_role
```

Это не предположение: оба значения непосредственно присутствуют в `c_api.php`.

fileciteturn17file13

Тем самым подтверждается:

```text
Session Role
```

и:

```text
API Role
```

являются двумя физически различными значениями в runtime contract.

### Confirmed Claims

```text
C-ROLE-001

User Session
    HAS
Session Role

confidence:
CONFIRMED
```

```text
C-ROLE-002

Authenticated API Context
    HAS
API Role Context

confidence:
CONFIRMED
```

---

# 5. Что пытались установить в RP-05

Был выполнен поиск по доступному source corpus для:

```text
taxi::$id_role
taxi::$id_role assignments
function check_auth_user
$_SESSION['id_role']
user_roles
role resolution
```

В доступном индексированном материале удалось подтвердить call sites и consumer sites, но не получить физическое определение:

```text
check_auth_user()
```

и не получить подтверждённый writer для:

```text
taxi::$id_role
```

за пределами:

```text
$API->id_role = taxi::$id_role
```

Следовательно, методология требует остановиться здесь, а не достраивать происхождение по имени переменной.

---

# 6. Важное различие: writer и consumer

Найденное:

```text
$API->id_role = taxi::$id_role
```

является **consumer evidence** для `taxi::$id_role`.

Оно не является:

```text
writer evidence
```

для `taxi::$id_role`.

Поэтому нельзя ошибочно записать:

```text
taxi::$id_role
    DERIVED_FROM
$_SESSION['id_role']
```

только потому, что рядом существует:

```text
u_role = $_SESSION['id_role']
u_a_role = $API->id_role
```

Это было бы ровно тем самым semantic shortcut, против которого направлена v2.3.

---

# 7. `u_role` и `u_a_role`

Физически подтверждено:

```text
u_role
    ← $_SESSION['id_role']

u_a_role
    ← $API->id_role
```

Но пока не подтверждено:

```text
u_role = persistent/base role
```

или:

```text
u_a_role = effective/active role
```

или:

```text
u_a_role = authorization role
```

Любая из этих формулировок остаётся гипотезой до получения writer/transform evidence.

---

# 8. `user_roles`

В `c_api.php` `user_roles` передаётся в role-sensitive операции.

Например:

```text
/register
    → registerUser(..., taxi::$data['user_roles'])
```

и:

```text
/user
    → editUser(..., taxi::$data['user_roles'], ...)
```

fileciteturn18file0
fileciteturn19file5

Также `user_roles` передаётся в некоторые communication/user-related методы.

Например, `sendMessage` получает:

```text
taxi::$data['user_roles']
```

fileciteturn20file14

Это подтверждает:

```text
user_roles
    CONFIGURES / PARAMETERIZES
role-related backend behavior
```

Но не подтверждает:

```text
user_roles
    IS
authorization policy
```

---

# 9. `query_roles`

В query-related API используется:

```text
taxi::$data['site_constants']['query_roles']['value']
```

и это значение передаётся в:

```text
queryString(...)
```

fileciteturn17file18

Это сильное Evidence того, что существуют role-related query rules.

Но без тела `queryString()` нельзя установить:

```text
как именно query_roles интерпретируется;
какие роли разрешены;
является ли это authorization;
является ли это фильтром query construction.
```

Поэтому:

```text
query_roles semantics
    UNKNOWN / BEHAVIOR_UNRESOLVED
```

---

# 10. DB Evidence

Предыдущий DB pass подтверждает наличие:

```text
users
users_roles
```

и структурных связей вокруг User/Role.

Также database snapshot подтверждает существование configuration values в `site_constant`, включая authentication-related configuration.

Но DB schema сама по себе не устанавливает runtime transformation:

```text
users.id_role
    ↓
$_SESSION['id_role']
    ↓
taxi::$id_role
```

Поэтому этот участок нельзя достроить DB-only reasoning.

---

# 11. Что сейчас можно зафиксировать в графе

```text
User Session
    HAS
Session Role

Authenticated API Context
    HAS
API Role Context

Protected API Operation
    RECEIVES
API Role Context
```

Все три:

```text
CONFIRMED
```

Evidence:

```text
CODE_USAGE
c_api.php
```

---

# 12. Что пока нельзя фиксировать

Не фиксировать как CONFIRMED:

```text
API Role
    = Session Role
```

```text
API Role
    = Effective Role
```

```text
API Role
    = Authorization Role
```

```text
API Role
    DERIVED_FROM
$_SESSION['id_role']
```

```text
user_roles
    = Authorization Policy
```

```text
query_roles
    = Authorization Policy
```

Для этих Claims пока нет требуемой provenance chain.

---

# 13. Research Loop result

Исходный вопрос:

```text
What is taxi::$id_role?
```

не получил окончательного ответа.

Но UNKNOWN был локализован значительно точнее:

```text
UNKNOWN
    ↓
missing writer / initialization source
```

Это:

```text
UNKNOWN / SOURCE_GAP
```

а не:

```text
BEHAVIOR_UNRESOLVED
```

поскольку вопрос о происхождении значения ещё нельзя решить без недостающего source fragment.

---

# 14. Новые Research Questions

RP-05 порождает три более точных вопроса.

### RQ-05-1

```text
Where is taxi::$id_role initialized or assigned?
```

Expected Evidence:

```text
static property declaration
constructor
check_auth_user implementation
role-resolution helper
bootstrap/session initialization
```

---

### RQ-05-2

```text
What is the difference between
$_SESSION['id_role'] and taxi::$id_role?
```

Expected Evidence:

```text
writers of both values
readers of both values
transformations between them
```

---

### RQ-05-3

```text
Where is API role actually used to make
a permission or data-scope decision?
```

Expected Evidence:

```text
role comparisons
permission checks
query filters
access-denied branches
role-specific code paths
```

---

# 15. Gap Report

## G-05-01

```text
taxi::$id_role writer
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
locate declaration + all assignments
```

---

## G-05-02

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

## G-05-03

```text
u_role vs u_a_role semantics
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
trace writers and transformations
```

---

## G-05-04

```text
actual authorization decision
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
trace role-dependent branches and query filters
```

---

## G-05-05

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

# 16. MCR result

```text
MCR = NO CHANGE
```

RP-05 не выявил недостатка выразительности Semantic Graph.

Нам не нужен новый:

```text
RoleNode
PermissionNode
AuthorizationNode
```

Проблема исключительно в недостающем Evidence.

Это важно: **мы не должны менять структуру графа из-за отсутствующего кода.**

---

# 17. Методологический результат

RP-05 подтвердил важное правило v2.3:

> Consumer не является Evidence происхождения значения.

Наблюдение:

```text
$API->id_role = taxi::$id_role
```

доказывает использование `taxi::$id_role`.

Наблюдение:

```text
u_a_role = $API->id_role
```

доказывает выдачу API role во frontend-facing contract.

Но только writer может доказать происхождение `taxi::$id_role`.

Это именно тот случай, для которого нужен Research Loop:

```text
UNKNOWN
   ↓
Research Question
   ↓
Expected Evidence
   ↓
Search Plan
   ↓
SOURCE_GAP
```

а не:

```text
UNKNOWN
   ↓
LLM inference
   ↓
CONFIRMED
```

---

# 18. Текущее состояние Identity & Access

На данный момент наиболее корректная модель:

```text
                    User
                      │
             ┌────────┴────────┐
             ↓                 ↓
       Session State       Account State
             │                 │
       Session Role       Verification
             │             Ban / Active
             │
             ↓
      Authentication Gate
             │
             ↓
   Authenticated API Context
             │
             ↓
       API Role Context
             │
             ↓
    Protected API Operation
```

При этом:

```text
Authentication
    CONFIRMED

Role Context
    CONFIRMED

Authorization Capability
    RESEARCHING
```

---

# 19. Следующий шаг

RP-05 не следует продолжать расширением поиска по всем ролям.

Следующий рациональный шаг — получить именно отсутствующий source fragment:

```text
definition of check_auth_user()
```

и:

```text
writer(s) of taxi::$id_role
```

После получения этих двух фрагментов RP-05 можно закрыть одним коротким pass:

```text
writer
→ source
→ transformation
→ consumer
```

Если же эти фрагменты физически отсутствуют в `BACKEND-S0`, это нужно зафиксировать как:

```text
SOURCE_SNAPSHOT_INCOMPLETE
```

и получить новую версию backend snapshot.

До этого любые утверждения о том, что `u_a_role` является effective/authorization role, остаются INFERRED/UNKNOWN и не должны попадать в CONFIRMED semantic graph.
