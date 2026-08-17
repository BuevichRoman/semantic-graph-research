# Семантический граф Backend — Research Pass 02
## Вертикальный срез Authentication: Frontend → Core Backend → БД → конфигурация

**Проход:** RP-02  
**Область:** capability Authentication  
**Статус:** `IN PROGRESS / PROVISIONAL`  
**Принцип исследования:** один Core Backend, несколько приложений/клиентов

---

## 1. Исследовательский вопрос

Первый проход по backend выявил область Authentication, однако одного backend недостаточно для полного восстановления семантики пользовательской аутентификации.

Поэтому этот проход прослеживает authentication flow через реальный клиент и общий Core Backend:

```text
Taxi Frontend
      ↓
Frontend authentication state / UI
      ↓
Frontend API adapter
      ↓
Core Backend API
      ↓
Authentication implementation
      ↓
DB / session / token / users_code
      ↓
Configuration / site_constant
```

Цель не состоит в проектировании Platform Auth.

Цель — установить, чем Authentication фактически является в существующей системе.

---

# 2. Source Snapshots

## 2.1 Frontend

```text
source_snapshot_id: FRONTEND-TAXI-S0
artifact: taxi-master.zip
sha256:
960959041a5777ac6bf6fe072b6c68825dfa79f54e3db1509365804bae97b680
```

## 2.2 Core Backend

```text
source_snapshot_id: BACKEND-S0
artifact: archive_17012026_1259_clear (1).zip
sha256:
e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
```

## 2.3 База данных

```text
source_snapshot_id: DB-S0
artifact: aristotel_taxi.sql (1).zip
sha256:
501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
```

Три snapshot являются независимыми физическими источниками.

---

# 3. System-level finding

Frontend **не содержит собственной реализации аутентификации**.

Он содержит клиент аутентификации:

```text
UI
 ↓
Redux user state / sagas
 ↓
API/auth.ts
 ↓
Core Backend API
```

Backend содержит реализацию аутентификации.

Следовательно:

```text
Authentication
   ├── Frontend Client Capability
   └── Core Backend Authentication Capability
```

Это первое конкретное подтверждение того, что Semantic Graph должен уметь представлять capability, проходящую через границы приложений.

Frontend является потребителем общего Core Backend, а не владельцем backend-логики аутентификации.

---

# 4. Frontend semantic structures

## 4.1 Login UI

### SourceFact

```text
SF-FE-AUTH-001

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/components/modals/LoginModal/Login.tsx

symbol:
LoginForm

structural_anchor:
kind: React component
name: LoginForm

navigation_hint:
form / submit handling around lines 1-300
```

Наблюдаемое поведение:

- login form accepts login and authentication type;
- email and WhatsApp are presented as authentication modes;
- password is shown for the normal password mode;
- successful/failed authentication is represented through `status`;
- authenticated user causes logout UI to be rendered;
- driver and client context affects post-login navigation.

### Semantic Claim

```text
C-FE-AUTH-001

subject: Authentication UI
predicate: PROVIDES
object: SignInInteraction
confidence: CONFIRMED
```

Evidence:

```text
E-FE-AUTH-001
type: CODE_USAGE
strength_category: BEHAVIORAL
source_fact_ids:
  - SF-FE-AUTH-001
```

---

# 5. Frontend authentication modes

Frontend определяет:

```text
ERegistrationType
    Email
    Phone
    Whatsapp
```

и вкладки входа:

```text
sign-in
sign-up
```

Фактический интерфейс входа предоставляет:

```text
Email
Whatsapp
```

Phone входит в общую модель типов регистрации/входа, но в исследованном LoginForm не отображается как отдельный вариант входа.

Это различие существенно:

```text
Capability доменной модели
    ≠
вариант, фактически предоставляемый UI
```

---

# 6. Frontend API authentication adapter

## 6.1 Общий wrapper аутентифицированного API

### SourceFact

```text
SF-FE-AUTH-002

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/tools/api.ts

symbol:
apiMethod

structural_anchor:
kind: function
name: apiMethod
```

Наблюдаемое поведение:

```text
authRequired = true
        ↓
read userSelectors.tokens()
        ↓
require token + u_hash
        ↓
append token
append u_hash
        ↓
invoke API method
```

### Claim

```text
C-FE-AUTH-002

subject: Frontend API Client
predicate: AUTHENTICATES_REQUESTS_WITH
value: token + u_hash
confidence: CONFIRMED
```

Это сильный факт frontend-to-auth контракта.

---

# 7. Frontend token model

## 7.1 ITokens

### SourceFact

```text
SF-FE-AUTH-003

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/types/types.ts

symbol:
ITokens
structural_anchor:
interface ITokens
```

Наблюдаемые поля:

```text
token
u_hash
```

### Claim

```text
C-AUTH-TOKEN-001

subject: Frontend Authentication Session
predicate: REPRESENTED_BY
value: token + u_hash
confidence: CONFIRMED
```

Important:

`token` and `u_hash` are not equivalent objects in the code. Frontend рассматривает их как пару.

---

# 8. Frontend login request

## 8.1 API login

### SourceFact

```text
SF-FE-AUTH-004

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/API/auth.ts

symbol:
_login
structural_anchor:
function _login
```

Наблюдаемое поведение:

```text
POST ${Config.API_URL}/auth

parameters:
login
password
type
au = f
```

Frontend ожидает несколько вариантов ответа backend:

```text
wrong login
code sent
wrong phone
wrong password
auth_hash
```

Если возвращён `auth_hash`:

```text
POST ${Config.API_URL}/token
```

with:

```text
auth_hash
```

Полученный token payload становится:

```text
token
u_hash
```

### Claim

```text
C-AUTH-CONTRACT-001

subject: Frontend Authentication Client
predicate: CALLS
object: CoreBackend /auth
confidence: CONFIRMED
```

```text
C-AUTH-CONTRACT-002

subject: CoreBackend /auth
predicate: MAY_RETURN
value: auth_hash
confidence: CONFIRMED
```

```text
C-AUTH-CONTRACT-003

subject: Frontend Authentication Client
predicate: CALLS
object: CoreBackend /token
confidence: CONFIRMED
```

---

# 9. Frontend authentication state machine

Redux saga задаёт state flow уровня приложения.

## Login

```text
LOGIN_REQUEST
      ↓
LOGIN_START
      ↓
API.login
      ↓
 ┌────┼──────────────┐
 ↓    ↓              ↓
wrong code sent      success
 ↓    ↓              ↓
FAIL  WHATSAPP       save tokens
      ↓              ↓
      verification   LOGIN_SUCCESS
                     ↓
                  set user
```

Состояние реализовано в:

```text
src/state/user/constants.ts
src/state/user/sagas.ts
src/state/user/actionCreators.ts
```

### Claim

```text
C-FE-AUTH-FLOW-001

subject: Frontend Authentication
predicate: HAS_STATE_FLOW
value:
LOGIN_REQUEST → LOGIN_START → API.login →
FAIL / WHATSAPP / SUCCESS
confidence: CONFIRMED
```

Это машина состояний взаимодействия приложения, а не обязательно FSM аутентификации backend.

---

# 10. Persistent frontend authentication

## SourceFact

```text
SF-FE-AUTH-005

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/state/user/sagas.ts

symbol:
loginSaga
structural_anchor:
function* loginSaga
```

При успешном входе:

```text
localStorage.setItem(
  'state.user.tokens',
  JSON.stringify(result.tokens)
)
```

### Claim

```text
C-FE-AUTH-004

subject: Frontend Authentication
predicate: PERSISTS
value: localStorage[state.user.tokens]
confidence: CONFIRMED
```

---

# 11. Authentication restoration on application startup

## SourceFact

```text
SF-FE-AUTH-006

source_type: CODE
source_snapshot_id: FRONTEND-TAXI-S0

file:
src/state/user/sagas.ts

symbol:
initUserSaga
structural_anchor:
function* initUserSaga
```

Наблюдаемое поведение:

```text
read localStorage tokens
      ↓
require token + u_hash
      ↓
SET_TOKENS
      ↓
API.getAuthorizedUser()
      ↓
setUser(user)
```

Если авторизованного пользователя получить не удаётся:

```text
remove stored tokens
```

### Claim

```text
C-AUTH-RESTORE-001

subject: Frontend Authentication
predicate: RESTORES_SESSION_FROM
value: token + u_hash
confidence: CONFIRMED
```

```text
C-AUTH-RESTORE-002

subject: Frontend Authentication
predicate: VALIDATES_SESSION_THROUGH
object: CoreBackend /user/authorized
confidence: CONFIRMED
```

---

# 12. Core Backend API surface

Контроллер общего Core Backend предоставляет:

```text
/register
/auth
/logout
/remind
/token
```

Source:

```text
archive_17012026_1259/taxi/controllers/c_api.php
```

### SourceFacts

```text
SF-BE-AUTH-001
symbol: register route
structural anchor:
  c_api.php dispatch branch for par[1] == register

SF-BE-AUTH-002
symbol: auth route
structural anchor:
  c_api.php dispatch branch for par[1] == auth

SF-BE-AUTH-003
symbol: logout route
structural anchor:
  c_api.php dispatch branch for par[1] == logout

SF-BE-AUTH-004
symbol: remind route
structural anchor:
  c_api.php dispatch branch for par[1] == remind

SF-BE-AUTH-005
symbol: token route
structural anchor:
  c_api.php dispatch branch for par[1] == token
```

Навигационные подсказки from the current snapshot:

```text
register: 24-27
auth:     29-31
logout:   34-36
remind:   38-41
token:    312-318
```

Номера строк являются только навигационными метаданными.

---

# 13. Core Backend Authentication implementation

## 13.1 authUser

### SourceFact

```text
SF-BE-AUTH-006

source_type: CODE
source_snapshot_id: BACKEND-S0

file:
archive_17012026_1259/taxi/models/api.php

symbol:
API::authUser

structural_anchor:
kind: method
owner: API
name: authUser
```

Метод:

- rejects an already authorized session;
- validates login;
- supports configured authentication types;
- performs user lookup;
- validates password for password-based authentication;
- checks deleted users;
- checks authentication bans;
- supports code-based authentication;
- generates authentication codes;
- sends them through configured channels;
- persists `users_code`;
- validates submitted code and expiration;
- creates authenticated session state;
- returns `auth_user` and `auth_hash`.

### Claim

```text
C-BE-AUTH-001

subject: Core Backend
predicate: IMPLEMENTS
object: Authentication
confidence: CONFIRMED
```

This is now a direct code-confirmed claim.

---

# 14. Password authentication

The backend queries `users` using the selected login type and checks:

```text
md5(md5(password)) == users.pwd
```

Он отклоняет:

```text
unknown user
wrong password
deleted user
banned user
```

Важное семантическое различие:

```text
Authentication
   ├── credential verification
   └── account eligibility checks
```

Последнее уже затрагивает Authorization / Account State и поэтому не должно автоматически включаться в будущую границу Auth Platform.

---

# 15. Code-based authentication

Backend поддерживает кодовую аутентификацию в `authUser`.

Flow:

```text
login
 ↓
resolve user
 ↓
select delivery method
 ↓
generate 4-digit code
 ↓
send code
 ↓
persist users_code
 ↓
return "code sent"
```

В исследованном коде обнаружены следующие каналы доставки:

```text
WhatsApp
Email
SMS
Telegram
```

Это создаёт междоменную связь:

```text
Authentication
      ↓
Verification Code
      ↓
Communication
      ├── WhatsApp
      ├── Email
      ├── SMS
      └── Telegram
```

Следовательно, Communication — не просто внешняя интеграция.

---

# 16. Authentication code persistence

## DB SourceFact

```text
SF-DB-AUTH-001

source_type: DB_SCHEMA
source_snapshot_id: DB-S0

table:
users_code

structural_anchor:
table users_code
```

Поля:

```text
id_user
code
expire_datetime
auth_type
json
last_edit_int_timestamp
start_int_timestamp
```

### Claim

```text
C-AUTH-CODE-001

subject: Authentication Code
predicate: PERSISTS
object: users_code
confidence: CONFIRMED
```

---

# 17. Authentication code expiration

Core backend writes:

```text
expire_datetime =
time() + auth_code_interval
```

и проверяет:

```text
users_code.expire_datetime > current time
```

Source:

```text
API::authUser
```

Конфигурация:

```text
auth_code_interval
```

### Claim

```text
C-AUTH-CONFIG-001

subject: Authentication Code
predicate: CONFIGURED_BY
object: auth_code_interval
confidence: CONFIRMED
```

Это подтверждается:

```text
CODE Evidence
+
DB_DATA site_constant Evidence
```

---

# 18. Authentication session creation

После успешной проверки учётных данных/кода backend создаёт:

```text
$_SESSION[UID]
$_SESSION[name]
$_SESSION[family]
$_SESSION[email]
$_SESSION[phone]
$_SESSION[id_role]
$_SESSION[id_verification_status]
$_SESSION[user_ban_status]
$_SESSION[active]
$_SESSION[id_lang]
$_SESSION[currency]
$_SESSION[id_navigation]
```

Также создаются:

```text
vfoc cookie
auth_hash
```

### Claim

```text
C-AUTH-SESSION-001

subject: Core Backend Authentication
predicate: CREATES
object: ServerSession
confidence: CONFIRMED
```

```text
C-AUTH-SESSION-002

subject: Core Backend Authentication
predicate: CREATES
object: auth_hash
confidence: CONFIRMED
```

---

# 19. Token issuance

The backend exposes `/token`.

`API::selectToken` can reconstruct/validate the authenticated session from:

```text
auth_hash
```

Он проверяет:

```text
session cookie
vfoc
auth_time
token_interval_for_auth_hash
```

после чего получает token.

The backend also persists tokens in:

```text
token
```

### DB SourceFact

```text
SF-DB-AUTH-002

source_type: DB_SCHEMA
source_snapshot_id: DB-S0

table:
token

fields:
token
ip
user_agent
id_user
datetime
```

### Claim

```text
C-AUTH-TOKEN-001

subject: Core Backend Authentication
predicate: PERSISTS
object: token
confidence: CONFIRMED
```

---

# 20. Frontend ↔ backend token contract

Полная наблюдаемая цепочка:

```text
Frontend
  ↓
POST /auth
  ↓
Core Backend authUser()
  ↓
auth_hash
  ↓
Frontend POST /token
  ↓
Core Backend selectToken()
  ↓
token + u_hash
  ↓
Frontend localStorage
  ↓
subsequent authenticated API calls
```

Это наиболее важная вертикальная связь, обнаруженная в RP-02.

Пара `token + u_hash` во frontend не является независимым механизмом аутентификации.

Это клиентское представление authentication/session-контракта, реализованного Core Backend.

---

# 21. Authorized-user restoration

Frontend:

```text
API.getAuthorizedUser()
```

calls:

```text
POST /user/authorized
```

with the generic authenticated API wrapper.

The Core Backend then reconstructs the current authorized user from the server-side authenticated state and returns an `auth_user` representation.

В backend `c_api.php` есть явная обработка `au`, формирующая `auth_user` из аутентифицированной сессии и/или через `selectUser()`.

### Claim

```text
C-AUTH-USER-001

subject: Core Backend Authentication
predicate: EXPOSES
object: AuthorizedUser
confidence: CONFIRMED
```

---

# 22. Logout

Frontend:

```text
logoutSaga
    ↓
remove localStorage user/tokens
    ↓
API.logout()
```

Backend:

```text
API::logout()
    ↓
unset($_SESSION[UID])
session_unset()
```

### Semantic relation

```text
Frontend Logout
      ↓
Core Backend Logout
      ↓
Server Session termination
```

The frontend additionally removes its local token representation.

---

# 23. Registration

Authentication связана с регистрацией.

Frontend registration:

```text
RegisterForm
    ↓
userActionCreators.register
    ↓
registerSaga
    ↓
API.register
    ↓
POST /register
```

Backend:

```text
/register
    ↓
API::registerUser
```

Registration returns token information in the frontend's observed contract, after which the frontend stores:

```text
token
u_hash
```

and calls:

```text
initUserSaga
```

Следовательно:

```text
Registration
    ↓
Authentication bootstrap
```

is a confirmed application flow.

Но:

```text
Registration = Authentication
```

is **not** a valid semantic conclusion.

They are related capabilities.

---

# 24. Google authentication

Frontend поддерживает точку входа через Google OAuth:

```text
accounts.google.com
    ↓
Config.SERVER_URL /google/
```

The frontend's `googleLogin()` handles either:

```text
auth_hash
```

or a registration flow followed by:

```text
/token/authorized
```

Это указывает на второй вход в контур аутентификации:

```text
External Identity Provider
        ↓
Core Backend
        ↓
token / u_hash
        ↓
Frontend
```

Точная реализация backend `/google/` в этом проходе полностью не прослежена.

Следовательно:

```text
Google Authentication
status: INFERRED
```

не `CONFIRMED` как полный end-to-end backend flow.

---

# 25. User model discovered from frontend

Frontend `IUser` exposes:

```text
u_id
u_name
u_phone
u_role
u_check_state
u_ban
token
u_hash
u_lang
u_currency
...
```

Важные для семантики Authentication поля:

```text
u_role
u_check_state
u_ban.auth
token
u_hash
u_phone
u_phone_checked
```

Это показывает, что состояние аутентификации во frontend не ограничивается:

```text authenticated / unauthenticated
```

It also consumes account state relevant to authorization and verification.

Следовательно:

```text Authentication
       ↕
User
       ↕
Verification
       ↕
Authorization
```

должны оставаться отдельными семантическими Frame.

---

# 26. Authentication / Authorization boundary

Текущие Evidence позволяют различить:

### Authentication

Evidence includes:

```text login
password/code verification
session creation
auth_hash
token
u_hash
authorized-user restoration
logout
```

### Authorization / пригодность учётной записи

Evidence includes:

```text user role
id_role
u_a_role
auth ban
verification state
active/deleted status
```

Backend применяет часть этих проверок непосредственно во время аутентификации.

Следовательно, граница реализации не совпадает с семантической границей.

Это один из центральных выводов.

```text
Authentication
   USES
      User Account State
   USES
      Role / Verification / Ban information
```

It does not prove that Role or Verification is owned by Authentication.

---

# 27. Configuration subgraph

Обнаружена следующая релевантная конфигурация:

```text
auth_code_interval
token_interval_for_auth_hash
session_token_duration
```

### Confirmed

```text
auth_code_interval
token_interval_for_auth_hash
```

активно используются кодом аутентификации.

### Важный UNKNOWN

`session_token_duration` существует в конфигурации БД, однако текущая реализация backend содержит:

```text
createSessionToken()
checkSessionToken()
```

при фактически пустой/закомментированной реализации.

Следовательно:

```text
session_token_duration
```

не следует считать активным поведением Authentication только на основании существования настройки.

Это конкретный пример принципа:

```text
DB_DATA existence
      ≠
active capability
```

---

# 28. Authentication semantic graph

Текущий вертикальный срез можно представить так:

```text
                         User
                          │
              ┌───────────┼────────────┐
              │           │            │
       Authentication  Role      Verification
              │
       ┌──────┼─────────────┐
       │      │             │
    Password Code        External IdP
             │               │
      ┌──────┼──────┐        │
      │      │      │        │
     SMS   Email WhatsApp  Google
      │      │      │        │
      └──────┴──────┘        │
             │               │
        users_code           │
             │               │
             └──────┬────────┘
                    ↓
               ServerSession
                    ↓
                auth_hash
                    ↓
                  Token
                 /     \
             token     u_hash
                \       /
                 Frontend
                    ↓
              localStorage
                    ↓
          authenticated API calls
```

---

# 29. Cross-source provenance summary

## Frontend

Для следующих объектов уже существуют конкретные SourceFact:

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

## Core Backend

Для следующих объектов уже существуют конкретные SourceFact:

```text
c_api.php:
register
auth
logout
remind
token
authorized-user response handling

api.php:
authUser
logout
selectToken
registerUser
remind

m_functions.php:
auth_user
token_exists
```

## База данных

Для следующих объектов уже существуют конкретные SourceFact:

```text
users
users_code
token
session_token
site_constant
```

---

# 30. Current confirmed claims

Следующие утверждения уже достаточно обоснованы несколькими физическими источниками:

```text
1. The frontend is an authentication client of the Core Backend.

2. The Core Backend implements the authentication capability.

3. Authentication supports password-based login.

4. Authentication supports code-based login.

5. Authentication codes are persisted in users_code.

6. Authentication code lifetime is configured by auth_code_interval.

7. Successful authentication creates server-side session state.

8. Backend authentication produces auth_hash.

9. Backend token issuance produces token/u_hash consumed by frontend.

10. Frontend persists token/u_hash locally.

11. Frontend restores authentication through /user/authorized.

12. Logout terminates frontend and backend authentication state.

13. Authentication uses communication mechanisms including
    SMS / Email / WhatsApp / Telegram.

14. User role, verification and ban state participate in authentication
    eligibility, but are not thereby proven to be owned by Authentication.

15. session_token_duration exists as configuration but is not confirmed
    as active session-token behavior.
```

---

# 31. Inferred claims

```text
Google Authentication is an end-to-end Core Backend capability.

Authentication and Authorization are separate reusable capabilities.

Communication is a shared capability used by Authentication.

Registration and Authentication are separate capabilities with a
post-registration authentication bootstrap.

u_hash is a security/session-related artifact rather than an independent
business entity.
```

Для повышения их статуса требуется дополнительная трассировка backend.

---

# 32. UNKNOWN / open questions

### U-01

Exact semantics and lifecycle of:

```text
u_hash
```

### U-02

Exact `/token/authorized` backend implementation.

### U-03

Exact `/google/` backend implementation.

### U-04

Complete relationship:

```text
token
session_token
auth_hash
u_hash
```

There are at least three different authentication/session mechanisms in the codebase.

### U-05

Whether `session_token` is legacy/dead code or an active path elsewhere.

### U-06

Exact boundary between:

```text
Authentication
Authorization
Verification
User Management
```

### U-07

Whether the same Core Backend authentication contract is consumed by all
other frontend applications.

This is strongly suggested by the architecture statement that there is one
Core Backend, but requires additional client repositories to confirm.

---

# 33. Platform Candidate status

```text
Platform Candidate: Auth
status: RESEARCHING
```

RP-02 **не рекомендует** создавать Platform Auth.

Он формирует значительно лучше определённый подграф-кандидат:

```text
Authentication
├── Credential Verification
├── Code Verification
├── Session Establishment
├── Token Issuance
├── Authorized User Restoration
├── Logout
└── External Authentication ingress
```

с зависимостями:

```text
User
Role
Verification
Account State
Communication
External Identity Provider
Configuration
```

This dependency graph is more important than the existence of `authUser()` itself.

---

# 34. Key architectural finding

Capability Authentication **не находится в одном frontend-модуле или одном backend-классе**.

Её фактическая реализация представляет собой:

```text
Frontend Interaction
        +
Frontend State
        +
Frontend API Adapter
        +
Core Backend Authentication
        +
Session / Token infrastructure
        +
Users / users_code / token DB structures
        +
Configuration
        +
Communication
```

Следовательно, семантический объект:

```text
Authentication
```

должен представляться графом, охватывающим несколько физических компонентов.

Это подтверждает решение включать frontend-источники в семантическое исследование там, где backend-only представление не позволяет восстановить capability полностью.

---

# 35. Next Research Pass

Следующий проход не должен сразу переходить к проектированию Platform Auth.

Остаётся исследовать наиболее важные вопросы:

```text
Authentication
      ↓
Authorization
      ↓
User / Role / Verification
```

and separately:

```text
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

Вторую цепочку следует проследить до точной семантики БД и runtime.

Только после разрешения этих вопросов Auth subgraph можно считать достаточно зрелым для Platform Candidate Decision Gate.
