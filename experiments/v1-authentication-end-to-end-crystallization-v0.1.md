# Semantic Graph V1 — Authentication End-to-End Crystallization v0.1

**Статус:** экспериментальный end-to-end pass  
**Основание:** `Semantic_Graph_First_Crystallization_Experiment_v0.1`, Backend RP-02 Authentication, Taxi Web/Core Backend Mapping  
**Объект:** Taxi Web v0.1.20 + общий Core Backend + связанные DB/configuration structures  
**Цель:** проверить первую кристаллизованную структуру Semantic Graph на реальном сквозном подграфе Authentication.

---

## 1. Результат эксперимента

Первая кристаллизация выдерживает Authentication без введения нового фундаментального типа.

Используемая структура:

```text
Frame
Relation
SemanticClaim
Evidence
    +
Frame Reification
    +
Reified Assertion
```

позволяет представить:

```text
Taxi Web
    ↓
Authentication
    ↓
User
    ↓
Credential / Verification
    ↓
Session / Token
    ↓
Core Backend
    ↓
DB / Configuration
```

При этом не потребовалось вводить:

```text
AuthNode
TokenNode
SessionNode
APIEndpointNode
FrontendNode
```

как специальные фундаментальные типы.

---

# 2. Важный результат: клиентская версия

Исследуемый frontend фиксируется как:

```text
Taxi Web v0.1.20
```

Это принципиально.

Граф не утверждает:

```text
Taxi Web = универсальный клиент Core Backend
```

Он утверждает:

```text
Taxi Web v0.1.20
    USES
Core Backend Authentication
```

Другой frontend должен получить отдельный client-specific Research Pass.

---

# 3. Минимальный Frame vocabulary

Для данного подграфа достаточно следующих Frames:

```text
Core Backend
Taxi Web v0.1.20

Authentication
User

Credential Verification
Verification Code
Authentication Session
Authentication Token

Users Code Storage
Token Storage
Configuration
Communication
```

Не все из них обязаны стать самостоятельными Frames в финальном графе.

Особенно:

```text
Credential
Verification Code
Authentication Session
Authentication Token
```

могут при дальнейшем анализе оказаться структурными объектами Authentication, а не самостоятельными domain Frames.

Поэтому их статус:

```text
RESEARCHING
```

до завершения token/session tracing.

---

# 4. Подтверждённые базовые Relations

## 4.1 Frontend → Authentication

```text
Taxi Web v0.1.20
    CALLS
Authentication
```

**Confidence:** CONFIRMED

Основание:

- frontend authentication components;
- `src/API/auth.ts`;
- auth sagas;
- observed `/auth`, `/token`, `/user/authorized`, `/logout` usage.

---

## 4.2 Backend → Authentication

```text
Core Backend
    IMPLEMENTS
Authentication
```

**Confidence:** CONFIRMED

Основание:

```text
API::authUser
c_api.php auth route
m_functions.php authentication helpers
```

---

## 4.3 Authentication → User

```text
Authentication
    USES
User
```

**Confidence:** CONFIRMED

Backend authentication resolves the user and validates account-related state.

Frontend restores the authorized user through:

```text
/user/authorized
```

---

## 4.4 Authentication → Configuration

```text
Authentication
    USES
Configuration
```

**Confidence:** CONFIRMED

Authentication behavior uses configuration including:

```text
auth_code_interval
```

The existence of a configuration value does not by itself prove every related setting is active.

---

## 4.5 Authentication → Verification Code

```text
Authentication
    USES
Verification Code
```

**Confidence:** CONFIRMED

Backend supports code-based authentication and persists authentication codes in:

```text
users_code
```

---

## 4.6 Verification Code → Users Code Storage

```text
Verification Code
    PERSISTS_IN
Users Code Storage
```

**Confidence:** CONFIRMED

Database structure:

```text
users_code
```

contains:

```text
id_user
code
expire_datetime
auth_type
json
...
```

---

# 5. Frontend token restoration

Frontend startup contains the following observed chain:

```text
localStorage
    ↓
token + u_hash
    ↓
SET_TOKENS
    ↓
getAuthorizedUser()
    ↓
/user/authorized
    ↓
authorized User
```

This can be represented as:

```text
Taxi Web Authentication
    RESTORES_SESSION_FROM
Authentication Token
```

and:

```text
Taxi Web Authentication
    VALIDATES_SESSION_THROUGH
Core Backend Authorized User API
```

Both are:

```text
CONFIRMED
```

for Taxi Web v0.1.20.

---

# 6. Почему token пока не делаем отдельным фундаментальным типом

Frontend явно хранит:

```text
token
u_hash
```

и использует их при authenticated API calls.

Но из текущего материала ещё не следует, что:

```text
token
u_hash
```

являются самостоятельными бизнес-сущностями.

Они могут быть:

```text
authentication artifacts
```

или:

```text
session credentials
```

Поэтому первая кристаллизация использует:

```text
Authentication
    HAS_ARTIFACT
token/u_hash
```

только как рабочую семантическую гипотезу, если это понадобится.

Новый Frame type:

```text
TOKEN
```

не вводится автоматически.

---

# 7. Server-side authentication

Backend `API::authUser`:

```text
resolve user
    ↓
validate credentials
    ↓
validate account eligibility
    ↓
optional code verification
    ↓
create authenticated session state
    ↓
return auth_user + auth_hash
```

Semantic representation:

```text
Authentication
    ESTABLISHES
Authentication Session
```

**Confidence:** CONFIRMED

Но точная семантика:

```text
auth_hash
session_token
token
u_hash
```

ещё не полностью восстановлена.

Поэтому:

```text
Authentication Session
```

пока является подтверждённым поведением, но не полностью раскрытым data object.

---

# 8. Authentication code as Assertion

Здесь первая кристаллизованная структура действительно становится полезной.

Backend имеет правило:

```text
submitted authentication code
    must match stored code
AND
stored code must not be expired
```

Это уже не простая Relation.

Концептуально:

```text
Assertion Frame
    SCOPE
        Authentication Code

    OPERATOR
        AND

    PREDICATE
        CodeMatchesStoredCode

    PREDICATE
        CodeNotExpired
```

Однако это **не добавляется пока в нормативную V1 структуру**, потому что `AND` ещё не прошёл отдельный MCR.

Это важная граница:

```text
структурная возможность
    ≠
подтверждённый vocabulary оператора
```

---

# 9. Что реально подтвердил Assertion mechanism

На текущем Authentication pass уже можно использовать reified Assertion для уже подтверждённых классов:

```text
Universal Predicate
Cardinality
```

Например, из backend:

```text
all requested users exist
```

может быть:

```text
Assertion
    SCOPE
        Requested Users

    OPERATOR
        ALL

    PREDICATE
        User Exists
```

Это уже подтверждённый ранее паттерн MCR-05/MCR-07.

Но сложные:

```text
AND
OR
NOT
```

не считаются подтверждёнными частями V1 только потому, что Authentication их использует.

---

# 10. Communication

Backend authentication использует:

```text
SMS
Email
WhatsApp
Telegram
```

для доставки verification code.

Поэтому:

```text
Authentication
    USES
Communication
```

**Confidence:** CONFIRMED

При этом:

```text
Communication
```

не включается автоматически в Authentication.

Это отдельная shared capability candidate.

---

# 11. User boundary

Authentication использует:

```text
User
```

и frontend User model содержит одновременно:

```text
authentication-related fields
authorization-related fields
profile fields
driver fields
localization fields
referral fields
```

Следовательно:

```text
Authentication
    USES
User
```

не означает:

```text
Authentication
    OWNS
User
```

Это важная проверка первой структуры.

Она позволяет сохранять зависимость без ложного вывода ownership.

---

# 12. Authorization boundary

В Authentication code присутствуют проверки:

```text
deleted user
banned user
role
verification state
```

Но из этого нельзя заключить:

```text
Authentication OWNS Authorization
```

Правильная текущая модель:

```text
Authentication
    DEPENDS_ON
Account / User State
```

и:

```text
Authorization
    UNKNOWN / RESEARCHING
```

до отдельного исследования.

Это предотвращает преждевременное превращение Auth Candidate в готовую Platform Auth.

---

# 13. Configuration boundary

Authentication использует:

```text
auth_code_interval
```

а общий backend имеет центральный:

```text
site_constant
```

Semantic relation:

```text
Authentication
    CONFIGURED_BY
Configuration
```

Подробные отдельные configuration values должны появляться в графе только при наличии:

```text
SourceFact:
site_constant row
+
CODE usage
```

То есть:

```text
DB setting exists
```

само по себе недостаточно для утверждения:

```text
Authentication uses setting
```

если usage не подтверждён.

---

# 14. Cross-source provenance

Authentication теперь имеет физические источники трёх классов:

```text
FRONTEND
    Taxi Web source

BACKEND
    Core Backend PHP

DATABASE
    users
    users_code
    token
    session_token

CONFIGURATION
    site_constant
```

Это именно тот случай, ради которого был введён cross-source Semantic Graph.

---

# 15. Полученный подграф

Минимальная версия:

```text
                         Core Backend
                              │
                         IMPLEMENTS
                              ↓
                       Authentication
                       /     |      \
                    USES    USES    USES
                     ↓       ↓       ↓
                   User   Config  Communication
                     │
                     │
              Verification Code
                     │
                 PERSISTS_IN
                     ↓
                users_code


Taxi Web v0.1.20
          │
        CALLS
          ↓
    Authentication
          │
    RESTORES_SESSION_FROM
          ↓
     token + u_hash
          │
   VALIDATES_THROUGH
          ↓
 /user/authorized
```

Это уже полноценный semantic subgraph, а не список файлов.

---

# 16. Что показала первая кристаллизация

### Она выдержала

```text
Frontend → Capability
Backend → Capability
Capability → Entity
Capability → Configuration
Capability → Shared Capability
Capability → Persistence
Frontend → Backend API behavior
```

без новых фундаментальных типов.

### Она также выдержала

```text
Universal Predicate
```

через:

```text
reified Assertion
```

для ранее подтверждённого класса.

---

# 17. Где структура пока остановилась

Остались три открытые области:

```text
1. Token/session semantics

token
u_hash
auth_hash
session_token
```

```text
2. Authorization boundary

Authentication
vs
Authorization
vs
Verification
vs
User Management
```

```text
3. Compound Assertions

AND
OR
NOT
```

Именно они сейчас являются следующими исследовательскими вопросами, а не основаниями для изменения структуры V1.

---

# 18. Главный результат

Для Authentication первая кристаллизованная модель:

```text
PASS
```

в смысле:

> существующая V1 структура способна представить реальный end-to-end Authentication subgraph без добавления нового фундаментального node type.

Это не означает:

```text
Platform Auth = готова
```

и не означает:

```text
Authentication boundary = окончательно определена.
```

Это означает только:

```text
Semantic Graph V1
    ↓
достаточно выразителен
    ↓
для текущего Authentication subgraph
```

---

# 19. Platform Candidate

После этого pass:

```text
Platform Candidate: Auth
status: RESEARCHING
```

остаётся без изменения.

Причины:

```text
Authorization boundary
Token/session semantics
User ownership
Communication dependency
cross-client reuse
```

ещё требуют исследования.

---

# 20. Следующий шаг

Следующий эксперимент должен быть не изменением V1.

Нужно провести **Authentication Deep Trace** по двум оставшимся наиболее критичным цепочкам:

```text
A.

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

и:

```text
B.

Authentication
    ↓
User
    ↓
Role / Verification / Ban
    ↓
Authorization
```

После этого можно проверить второй frontend.

Именно второй frontend даст ответ на важный вопрос:

```text
Authentication
    ↓
shared Core Backend capability
    ↓
different client implementation
```

а не только:

```text
Taxi Web
    ↓
Core Backend
```

Это будет следующим реальным тестом того, что Semantic Graph помогает отделять общую capability от client-specific implementation.
