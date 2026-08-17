# Backend Semantic Graph — Research Pass 37
# Authentication Credential Value-Flow v0.4

**Статус:** CONFIRMED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предыдущая версия:** RP-37 Authentication Credential Value-Flow v0.3  
**Источники:** `taxi-master.zip` + `archive_17012026_1259_clear.zip`

## 1. Research Question

> Какой credential передаёт Taxi Frontend в Core Backend API и каким механизмом Core Backend превращает этот credential в authenticated User?

## 2. Frontend: credential creation

Конкретный login flow в `src/API/auth.ts`:

```text
POST /auth
    ↓
response.auth_hash
    ↓
POST /token
    ↓
tokenRes.data.data.token
tokenRes.data.data.u_hash
```

Frontend формирует результат:

```text
tokens = {
    token,
    u_hash
}
```

Этот результат возвращается из `API.login()`.

## 3. Frontend: credential persistence

`src/state/user/sagas.ts` сохраняет полученные credentials:

```text
result.tokens
    ↓
localStorage
    ↓
state.user.tokens
```

При инициализации приложения:

```text
localStorage.state.user.tokens
    ↓
SET_TOKENS
    ↓
Redux user state
```

Затем frontend вызывает:

```text
API.getAuthorizedUser()
```

То есть authentication credentials не являются одноразовым login response: они становятся persistent client API context.

## 4. Frontend: common API injection

`src/tools/api.ts` содержит общий `apiMethod()`.

При `authRequired = true`:

```text
userSelectors.tokens(state.getState())
        ↓
tokens.token
tokens.u_hash
        ↓
FormData
        ↓
POST request
```

Конкретно:

```text
formData.append('token', tokens.token)
formData.append('u_hash', tokens.u_hash)
```

Это закрывает frontend side value-flow:

```text
Login
  → token/u_hash
  → localStorage
  → Redux
  → apiMethod()
  → POST token + u_hash
```

## 5. Core Backend: token → user identity

Core Backend `models/token.php` обрабатывает каждый request, содержащий:

```text
POST token
POST u_hash
```

Затем:

```text
parse_u_hash(u_hash, token)
        ↓
$_SESSION['token_auth']
        ↓
token_exists(token, token_auth)
        ↓
$_SESSION[UID] = token_auth
```

Это принципиально важное Evidence:

```text
Frontend token/u_hash
        ↓
Backend parse_u_hash()
        ↓
User ID
        ↓
$_SESSION[UID]
```

## 6. Core Backend: cryptographic relation

`get_u_hash()` формирует `u_hash` из:

```text
id_user
+
token
+
U_HASH_SECRET
```

а `parse_u_hash()`:

```text
decrypt(u_hash)
        ↓
id_user + hash
        ↓
recalculate hash(token)
        ↓
compare
        ↓
id_user
```

Следовательно, `u_hash` не является независимым идентификатором пользователя.

Он криптографически связан одновременно с:

```text
User ID
+
token
```

## 7. Core Backend: authenticated context

После успешного token validation:

```text
$_SESSION[UID]
```

становится идентификатором authenticated User.

`check_auth_user()` затем проверяет:

```text
$_SESSION[UID]
        ↓
users.id_user
        ↓
authenticated user record
```

Это уже отдельная authorization/authentication gate.

## 8. Полный Evidence chain

Теперь существует полностью подтверждённая цепочка:

```text
Taxi Frontend login
        ↓
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
Redux user.tokens
        ↓
apiMethod()
        ↓
POST token + u_hash
        ↓
Core Backend token.php
        ↓
parse_u_hash(u_hash, token)
        ↓
token_exists()
        ↓
$_SESSION[UID]
        ↓
check_auth_user()
        ↓
authenticated User
```

## 9. Final Claim

```text
Taxi Frontend
    ──AUTHENTICATES_WITH──>
Core Backend
```

**CONFIRMED**

Provenance:

```text
Frontend Snapshot:
taxi-master.zip

Backend Snapshot:
archive_17012026_1259_clear.zip

Frontend Evidence:
src/API/auth.ts
src/tools/api.ts
src/state/user/sagas.ts

Backend Evidence:
models/token.php
models/m_functions.php
models/api.php
```

## 10. Важное уточнение

`/auth` и `/token` — это две разные стадии.

Нельзя записывать:

```text
login → token
```

как один backend operation.

Фактически:

```text
/auth
    = authentication/session establishment
      + auth_hash

/token
    = получение API token/u_hash
      из установленной authenticated session
```

Для обычных последующих API calls используется уже:

```text
token + u_hash
```

## 11. Authentication ≠ Authorization

Этот pass закрывает:

```text
WHO IS THE USER?
```

через:

```text
token/u_hash → $_SESSION[UID]
```

Но он не утверждает, что конкретный frontend role имеет право выполнять каждую операцию.

Authorization остаётся отдельным graph layer:

```text
authenticated User
        ↓
role / permission
        ↓
operation allowed?
```

## 12. Методологический результат

RP-37 стал хорошим примером полного Research Loop:

```text
UNKNOWN
   ↓
Research Question
   ↓
Expected Evidence
   ↓
Frontend credential tracing
   ↓
Backend credential tracing
   ↓
value-flow join
   ↓
CONFIRMED
```

Также подтверждена важная поправка к RP-36:

```text
exact endpoint string match
```

не является обязательным условием для cross-layer Claim.

Главное Evidence — непрерывный **semantic/value flow**.

## 13. Следующий шаг

Authentication как вертикаль для этого Frontend snapshot **закрыта**.

Не создавать дальнейшие Auth RP без новой исследовательской причины.

Теперь можно переходить к следующему domain concept.
