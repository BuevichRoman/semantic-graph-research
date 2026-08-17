# Backend Semantic Graph — Research Pass 03
# Authentication Deep Trace v0.1

**Статус:** COMPLETED / PROVISIONAL  
**Область:** Authentication Deep Trace  
**Предшествующий проход:** RP-02 Authentication  
**Методология:** Semantic Graph Research Methodology v2.3  
**Цель:** проверить две цепочки, оставшиеся открытыми в RP-02, не изменяя структуру Semantic Graph без MCR.

---

## 1. Исследовательские вопросы

### RQ-03-A — Token/session chain

Установить точную наблюдаемую цепочку:

```text
authUser
   ↓
auth_hash
   ↓
selectToken
   ↓
token
   ↓
u_hash
   ↓
authenticated API request
```

и определить, где заканчиваются:

```text
ServerSession
auth_hash
token
u_hash
```

как разные артефакты одного authentication/session contract.

### RQ-03-B — Authentication / Authorization boundary

Установить, какие данные:

```text
role
verification
ban
active/deleted state
```

используются Authentication, а какие должны моделироваться как отдельные User Account / Authorization / Verification semantics.

---

# 2. Source Scope

Используются уже зафиксированные физические snapshots RP-02:

```text
FRONTEND-TAXI-S0
BACKEND-S0
DB-S0
```

Frontend:

```text
Taxi Web v0.1.20
```

Core Backend:

```text
archive_17012026_1259_clear (1).zip
```

DB:

```text
aristotel_taxi.sql (1).zip
```

В RP-02 для backend уже зафиксированы SourceFacts по:

```text
c_api.php
    register
    auth
    logout
    remind
    token
    authorized-user handling

api.php
    authUser
    logout
    selectToken
    registerUser
    remind

m_functions.php
    auth_user
    token_exists
```

Для DB:

```text
users
users_code
token
session_token
site_constant
```

Для frontend:

```text
LoginForm
apiMethod
ITokens
_login
loginSaga
initUserSaga
registerSaga
logoutSaga
getAuthorizedUser
WACodeModal
RegisterForm
```

---

# 3. RQ-03-A — Token/session chain

## 3.1 First stage: authUser

Backend `API::authUser` является подтверждённой точкой входа Core Backend authentication.

Он:

```text
resolve user
    ↓
validate credentials / code
    ↓
check account eligibility
    ↓
create authenticated server state
    ↓
return auth_user + auth_hash
```

Статус:

```text
CONFIRMED
```

SourceFact:

```text
SF-BE-AUTH-006
```

Claim:

```text
C-BE-AUTH-001

Core Backend
    IMPLEMENTS
Authentication
```

---

## 3.2 auth_hash

После успешной аутентификации backend создаёт:

```text
auth_hash
```

одновременно с серверным session state.

Подтверждённые отношения:

```text
Authentication
    CREATES
ServerSession
```

```text
Authentication
    CREATES
auth_hash
```

Оба:

```text
CONFIRMED
```

Важно:

`auth_hash` не следует пока моделировать как User или business entity.

Его подтверждённая роль — authentication/session artifact.

---

## 3.3 selectToken

Следующий подтверждённый backend шаг:

```text
API::selectToken
```

Он использует:

```text
auth_hash
session cookie
vfoc
auth_time
token_interval_for_auth_hash
```

для проверки/восстановления authenticated state и получения token.

Следовательно:

```text
auth_hash
    ──USED_BY──>
selectToken
```

и:

```text
Authentication
    ──ISSUES / RESOLVES──>
token
```

могут быть представлены как semantic relations.

Точная внутренняя математическая формула `auth_hash` и точный формат проверки не являются отдельным semantic object.

---

# 4. Token persistence

DB содержит:

```text
token
```

с полями:

```text
token
ip
user_agent
id_user
datetime
```

Это подтверждает:

```text
Token
    PERSISTS_IN
token
```

и:

```text
Token
    REFERENCES
User
```

если DB structural evidence подтверждает соответствующий FK.

Важно не путать:

```text
token table
```

и:

```text
Token semantic Frame
```

Первое — physical implementation.

Второе — semantic object, если его самостоятельность необходима для графа.

---

# 5. u_hash

Frontend после token flow получает:

```text
token + u_hash
```

и сохраняет их локально.

Это подтверждает:

```text
Frontend Authentication
    STORES
token + u_hash
```

и:

```text
Frontend Authentication
    USES
token + u_hash
```

для subsequent authenticated API calls.

Но текущие Evidence не дают достаточного основания утверждать, что:

```text
u_hash
```

является самостоятельным backend session object.

Поэтому:

```text
u_hash
status = INFERRED / authentication artifact
```

а не самостоятельный фундаментальный Frame.

---

# 6. Полученная token/session модель

На текущем уровне доказательности:

```text
User
  ↑
  │
Authentication
  │
  ├── CREATES → ServerSession
  │
  ├── CREATES → auth_hash
  │
  └── ISSUES → Token
                  │
                  └── PERSISTS_IN → token

Token + u_hash
        ↓
     Frontend
        ↓
  localStorage
        ↓
authenticated API calls
```

При этом:

```text
auth_hash
token
u_hash
session_token
```

не следует объединять в один объект.

Они имеют разные Evidence и разные места использования.

---

# 7. session_token

DB содержит:

```text
session_token
```

и configuration:

```text
session_token_duration
```

Но исследованный backend содержит `createSessionToken()` / `checkSessionToken()` в состоянии, которое не даёт достаточного подтверждения активного production authentication path.

Поэтому:

```text
session_token
```

остаётся:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

в отношении активного authentication flow.

А:

```text
session_token_duration
```

остаётся:

```text
CONFIRMED
```

как существующая configuration value,

но:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

как активный параметр production authentication.

Это важный пример:

```text
DB_DATA existence
    ≠
active behavior
```

---

# 8. Authenticated API request

Frontend использует:

```text
token + u_hash
```

после восстановления пользователя.

Наблюдаемая цепочка:

```text
POST /auth
    ↓
auth_hash
    ↓
POST /token
    ↓
token + u_hash
    ↓
localStorage
    ↓
authenticated API wrapper
    ↓
/user/authorized
and other authenticated endpoints
```

Это подтверждает:

```text
Authentication
    EXPOSES
Authenticated API Contract
```

Но не означает:

```text
token = complete authentication state
```

Серверная session state остаётся отдельным подтверждённым элементом.

---

# 9. RQ-03-A result

### Confirmed

```text
authUser
    → ServerSession

authUser
    → auth_hash

selectToken
    → authenticated session validation

Authentication
    → Token issuance

Token
    → DB persistence

Frontend
    → stores token + u_hash

Frontend
    → authenticated API calls

/user/authorized
    → authorized user restoration
```

### Inferred

```text
u_hash
    → authentication/session artifact
```

### Unknown / Behavior Unresolved

```text
session_token
    → active authentication mechanism?

session_token_duration
    → active runtime configuration?

exact internal semantics of u_hash
```

### Результат

```text
RQ-03-A = PARTIALLY ANSWERED
```

Основной production chain подтверждён.

`u_hash` и альтернативный `session_token` path требуют отдельного source/runtime evidence для полного раскрытия.

---

# 10. RQ-03-B — Authentication / Authorization

Frontend User model содержит:

```text
u_role
u_check_state
u_ban
u_phone_checked
```

Backend session state содержит:

```text
id_role
id_verification_status
user_ban_status
active
```

Backend authentication проверяет часть этих состояний при установлении authenticated session.

Следовательно:

```text
Authentication
    USES
User Account State
```

подтверждено.

---

# 11. Role

Role участвует в authentication eligibility и после authentication сохраняется в session state.

Но:

```text
Authentication
    OWNS
Role
```

не доказано.

Более точная модель:

```text
User
  └── HAS_ROLE → Role

Authentication
  └── USES → User Account State / Role
```

Статус:

```text
CONFIRMED
```

для использования роли,

```text
UNKNOWN
```

для ownership role со стороны Authentication.

---

# 12. Verification

Authentication поддерживает:

```text
password verification
code verification
```

и session state содержит:

```text
id_verification_status
```

Следовательно, существует semantic distinction:

```text
Credential Verification
```

и:

```text
Account Verification State
```

Они не должны автоматически объединяться.

Текущий граф:

```text
User
   │
   └── HAS_VERIFICATION_STATE
             ↓
        Verification State

Authentication
   └── USES
        Verification / Credential State
```

Статус:

```text
CONFIRMED
```

для существования и использования этих состояний.

Граница отдельной `Verification Capability`:

```text
RESEARCHING
```

---

# 13. Ban / account eligibility

Backend authentication отклоняет пользователя при соответствующем ban/account state.

Это подтверждает:

```text
Authentication
    USES
Account Eligibility
```

Но:

```text
Authentication
    OWNS
Ban
```

не следует.

Более точная модель:

```text
User
 └── HAS_ACCOUNT_STATE
        ├── active
        ├── deleted
        ├── ban
        └── verification state

Authentication
 └── CHECKS
      Account Eligibility
```

---

# 14. Authorization boundary

В текущем source set нет достаточного evidence для полной реконструкции отдельной:

```text
Authorization Capability
```

при этом есть достаточное evidence, что:

```text
role
verification
ban
account state
```

участвуют в решениях о допустимости authentication.

Поэтому:

```text
Authentication
    ≠
Authorization
```

является методологически корректным разделением,

но:

```text
Authorization
    IMPLEMENTED_BY
...
```

пока:

```text
UNKNOWN / SOURCE_GAP
```

если требуемый authorization call chain ещё не прослежен до конкретной реализации.

Это принципиально отличается от утверждения:

```text
Authorization отсутствует.
```

Такого Evidence нет.

---

# 15. RQ-03-B result

### Confirmed

```text
Authentication uses User Account State.

Role participates in authentication eligibility.

Verification state participates in authentication eligibility.

Ban / active / deleted state participates in authentication eligibility.

Authentication and Authorization should not be collapsed into one semantic Frame.
```

### Unknown

```text
Exact Authorization implementation
Exact User Management boundary
Exact ownership of Role
Exact ownership of Verification
Exact ownership of Ban / Account State
```

### Result

```text
RQ-03-B = PARTIALLY ANSWERED
```

---

# 16. Проверка нового Research Loop

Этот pass впервые применяет v2.3 непосредственно к старым UNKNOWN.

## U-01: u_hash

```text
UNKNOWN
    ↓
Research Question
    ↓
Search Plan
    ↓
Frontend + backend token chain
```

Результат:

```text
INFERRED / authentication artifact
```

но не `CONFIRMED` как самостоятельная сущность.

---

## U-04: token / session_token / auth_hash / u_hash

Результат:

```text
Production authentication path:
auth_hash → token → token+u_hash → authenticated API
```

подтверждён.

Альтернативный:

```text
session_token
```

не подтверждён как активный путь.

Это переводит часть вопроса из:

```text
UNKNOWN
```

в:

```text
BEHAVIOR_UNRESOLVED
```

с конкретным следующим действием:

```text
runtime / broader source search
```

---

## U-06: Authentication / Authorization / Verification / User Management

Результат:

```text
Authentication ↔ User Account State
```

подтверждён.

Но полная граница:

```text
Authorization
User Management
Verification
```

остаётся:

```text
OPEN
```

---

# 17. Semantic Synthesis

Deep Trace позволяет синтезировать более точное понятие:

```text
Authentication Session Contract
```

Его нельзя найти в коде одним классом.

Он синтезирован из Claims:

```text
C-AUTH-SESSION-001
C-AUTH-SESSION-002
C-AUTH-TOKEN-001
C-AUTH-RESTORE-001
C-AUTH-RESTORE-002
C-AUTH-USER-001
```

Semantic Synthesis:

```text
Authentication Session Contract
    =
server session
+
auth_hash
+
token issuance
+
token/u_hash client representation
+
authorized-user restoration
```

Статус:

```text
INFERRED
```

Это пример реального Semantic Synthesis, а не source-derived Frame.

---

# 18. Provenance синтезированного понятия

```text
Authentication Session Contract
        │
        ├── derived_from → C-AUTH-SESSION-001
        ├── derived_from → C-AUTH-SESSION-002
        ├── derived_from → C-AUTH-TOKEN-001
        ├── derived_from → C-AUTH-RESTORE-001
        ├── derived_from → C-AUTH-RESTORE-002
        └── derived_from → C-AUTH-USER-001
```

Важно:

```text
Synthesis
```

не создаёт новое Evidence.

Он создаёт более высокий semantic Frame на основе уже существующих Claims.

---

# 19. Проверка первой кристаллизации

Для Authentication V1 остаётся достаточной.

Не потребовались:

```text
AuthNode
SessionNode
TokenNode
AuthorizationNode
VerificationNode
```

как новые фундаментальные типы.

Полученная структура:

```text
Frame
Relation
SemanticClaim
Evidence
    +
Reification
```

выражает исследованный подграф.

Следовательно:

```text
MCR
= NO CHANGE
```

---

# 20. Обновлённый Authentication subgraph

```text
                           User
                             │
                   HAS_ACCOUNT_STATE
                             ↓
                  Account / User State
                    ├── Role
                    ├── Verification
                    ├── Ban
                    ├── Active
                    └── Deleted
                             ↑
                             │ USES / CHECKS
                             │
                      Authentication
                     /       |        \
                    /        |         \
          Credential      Code       External IdP
          Verification   Verification
              │              │             │
              │          users_code       Google
              │              │
              └──────────────┘
                     ↓
                ServerSession
                     │
                 auth_hash
                     ↓
                   Token
                     │
                PERSISTS_IN
                     ↓
                   token
                     │
                     ├───────────────┐
                     ↓               ↓
                  token           u_hash
                     \               /
                      \             /
                       └── Frontend
                              ↓
                         localStorage
                              ↓
                    authenticated API
                              ↓
                       /user/authorized
```

---

# 21. Updated UNKNOWN / Gap Report

### G-03-01

```text
u_hash exact semantics
```

Status:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Next action:

```text
trace backend generation/validation and all consumers
```

### G-03-02

```text
session_token active path
```

Status:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Next action:

```text
search all production consumers + runtime evidence
```

### G-03-03

```text
Authorization implementation boundary
```

Status:

```text
UNKNOWN / SOURCE_GAP
```

Next action:

```text
trace check_auth_user / role checks / permission helpers
and all authorization-specific API paths
```

### G-03-04

```text
User Management boundary
```

Status:

```text
UNKNOWN / BEHAVIOR_UNRESOLVED
```

Next action:

```text
trace user CRUD, role administration, verification administration,
ban administration and their callers
```

### G-03-05

```text
Verification capability boundary
```

Status:

```text
RESEARCHING
```

Next action:

```text
separate credential verification from account verification workflows
```

---

# 22. Итог RP-03

```text
RQ-03-A = PARTIALLY ANSWERED
RQ-03-B = PARTIALLY ANSWERED
```

Основной production authentication path теперь можно считать:

```text
CONFIRMED
```

на уровне:

```text
authUser
→ ServerSession
→ auth_hash
→ selectToken
→ token
→ token + u_hash
→ authenticated API
→ /user/authorized
```

При этом:

```text
u_hash
session_token
Authorization implementation
User Management boundary
Verification boundary
```

не следует искусственно переводить в `CONFIRMED`.

---

# 23. Методологический результат

RP-03 подтвердил практическую полезность добавлений v2.3.

### Research Loop работает

Старый UNKNOWN был превращён не в догадку, а в:

```text
Research Question
→ Search Plan
→ Result
```

### UNKNOWN разделился

```text
SOURCE_GAP
```

и:

```text
BEHAVIOR_UNRESOLVED
```

оказались практически разными состояниями.

### Semantic Synthesis работает

`Authentication Session Contract` не найден как единый объект в коде, но его можно построить из нескольких подтверждённых Claims с сохранением provenance.

### V1 структура не потребовала изменения

Это особенно важно:

```text
исследование усложнилось,
структура графа — нет.
```

---

# 24. Следующий шаг

RP-03 не является основанием для Platform Auth.

Следующий полезный проход:

```text
Authorization / User Management Deep Trace
```

с отдельными Research Questions:

```text
RQ:
Где реализуется authorization?

RQ:
Кто владеет role?

RQ:
Кто владеет verification state?

RQ:
Кто изменяет ban/account state?

RQ:
Какие API используют эти проверки после authentication?
```

После этого можно будет впервые построить:

```text
User
 ├── Authentication
 ├── Authorization
 ├── Verification
 └── User Management
```

не как теоретическую декомпозицию, а как доказанный semantic subgraph.

До этого `Platform Candidate: Auth` остаётся:

```text
RESEARCHING
```
