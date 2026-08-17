# Backend Semantic Graph — Research Pass 37
# Authentication Reconciliation v0.1

**Статус:** EVIDENCE-GROUNDED / RECONCILIATION
**Методология:** Semantic Graph Research Methodology v2.3
**Предыдущая Auth-ветка:** RP-31 → RP-36
**RP-37 Order Lifecycle:** отменён как направление исследования; метка RP-37 повторно использована для этого reconciliation pass.
**Источники:** `taxi-master.zip` + `archive_17012026_1259_clear.zip` + ранее созданные Auth RP

## 1. Цель

Не продолжать новый domain concept, а восстановить накопленное Authentication Evidence и проверить, почему RP-36 не получил exact endpoint match.

Целевая цепочка:

```text
Taxi Frontend credential
        ↓
API request construction
        ↓
Core Backend auth gate
        ↓
authenticated User/context
```

## 2. Предыдущие Auth conclusions

### `Backend_Semantic_Graph_Research_Pass_02_Authentication.md`

Найдено релевантных строк: **216**.

- ## Authentication vertical slice: Frontend → Core Backend → DB → Configuration
- **Scope:** Authentication capability
- The first backend pass showed an Authentication area, but backend-only analysis cannot fully establish the semantics of a user-facing authentication capability.
- This pass therefore follows the authentication flow across the actual client and the shared Core Backend:
- Frontend authentication state / UI
- Authentication implementation
- DB / session / token / users_code
- The purpose is to establish what Authentication actually is in the existing system.
- The frontend does **not** contain its own authentication implementation.
- It contains an authentication client:
- The backend contains the authentication implementation.
- Authentication
- └── Core Backend Authentication Capability
- The frontend is a consumer of the shared Core Backend rather than the owner of the authentication backend logic.
- - login form accepts login and authentication type;
- - email and WhatsApp are presented as authentication modes;
- - successful/failed authentication is represented through `status`;
- - authenticated user causes logout UI to be rendered;
- subject: Authentication UI
- # 5. Frontend authentication modes
- # 6. Frontend API authentication adapter
- ## 6.1 Generic API authentication wrapper
- read userSelectors.tokens()
- require token + u_hash
- append token
- append u_hash
- value: token + u_hash
- # 7. Frontend token model
- ## 7.1 ITokens
- ITokens
- interface ITokens
- token
- u_hash
- C-AUTH-TOKEN-001
- subject: Frontend Authentication Session
- value: token + u_hash
- `token` and `u_hash` are not equivalent objects in the code. The frontend treats them as a pair.
- POST ${Config.API_URL}/token
- The returned token payload becomes:
- token
- u_hash
- subject: Frontend Authentication Client
- subject: Frontend Authentication Client
- object: CoreBackend /token
- # 9. Frontend authentication state machine
- FAIL  WHATSAPP       save tokens
- subject: Frontend Authentication
- This is an application interaction state machine, not necessarily the backend authentication FSM.
- # 10. Persistent frontend authentication
- 'state.user.tokens',
- JSON.stringify(result.tokens)
- subject: Frontend Authentication
- value: localStorage[state.user.tokens]
- # 11. Authentication restoration on application startup
- read localStorage tokens
- require token + u_hash
- SET_TOKENS
- remove stored tokens
- subject: Frontend Authentication
- value: token + u_hash
- subject: Frontend Authentication
- /token
- symbol: token route
- c_api.php dispatch branch for par[1] == token
- token:    312-318
- # 13. Core Backend Authentication implementation
- - supports configured authentication types;
- - validates password for password-based authentication;
- - checks authentication bans;
- - supports code-based authentication;
- - generates authentication codes;
- - creates authenticated session state;
- object: Authentication
- # 14. Password authentication
- Authentication
- The latter is already touching Authorization / Account State and therefore must not automatically be absorbed into a future Auth Platform boundary.
- # 15. Code-based authentication
- The backend supports code authentication in `authUser`.
- Authentication
- # 16. Authentication code persistence

### `Backend_Semantic_Graph_Research_Pass_03_Authentication_Deep_Trace_v0.1.md`

Найдено релевантных строк: **176**.

- # Authentication Deep Trace v0.1
- **Область:** Authentication Deep Trace
- **Предшествующий проход:** RP-02 Authentication
- ### RQ-03-A — Token/session chain
- selectToken
- token
- u_hash
- authenticated API request
- token
- u_hash
- как разные артефакты одного authentication/session contract.
- ### RQ-03-B — Authentication / Authorization boundary
- role
- используются Authentication, а какие должны моделироваться как отдельные User Account / Authorization / Verification semantics.
- token
- selectToken
- token_exists
- token
- session_token
- ITokens
- # 3. RQ-03-A — Token/session chain
- Backend `API::authUser` является подтверждённой точкой входа Core Backend authentication.
- create authenticated server state
- Authentication
- Authentication
- Authentication
- Его подтверждённая роль — authentication/session artifact.
- ## 3.3 selectToken
- API::selectToken
- token_interval_for_auth_hash
- для проверки/восстановления authenticated state и получения token.
- selectToken
- Authentication
- token
- # 4. Token persistence
- token
- token
- Token
- token
- Token
- token table
- Token semantic Frame
- # 5. u_hash
- Frontend после token flow получает:
- token + u_hash
- Frontend Authentication
- token + u_hash
- Frontend Authentication
- token + u_hash
- для subsequent authenticated API calls.
- u_hash
- u_hash
- status = INFERRED / authentication artifact
- # 6. Полученная token/session модель
- Authentication
- └── ISSUES → Token
- └── PERSISTS_IN → token
- Token + u_hash
- authenticated API calls
- token
- u_hash
- session_token
- # 7. session_token
- session_token
- session_token_duration
- Но исследованный backend содержит `createSessionToken()` / `checkSessionToken()` в состоянии, которое не даёт достаточного подтверждения активного production authentication path.
- session_token
- в отношении активного authentication flow.
- session_token_duration
- как активный параметр production authentication.
- # 8. Authenticated API request
- token + u_hash
- POST /token
- token + u_hash
- authenticated API wrapper
- and other authenticated endpoints
- Authentication
- Authenticated API Contract
- token = complete authentication state
- selectToken

### `Backend_Semantic_Graph_Research_Pass_04_Authorization_Gate_Deep_Trace_v0.1.md`

Найдено релевантных строк: **168**.

- # Authorization Gate Deep Trace v0.1
- **Предшествующий проход:** RP-03 Authentication Deep Trace
- **Цель:** установить семантическую роль `check_auth_user()` и `taxi::$id_role` в защищённых API-операциях и определить границу Authentication / Authorization без предположений.
- > Что именно делает `check_auth_user()`, и где в Core Backend начинается собственно role-based authorization?
- authenticated request
- check_auth_user()
- taxi::$id_role
- API->id_role
- 1. является ли `check_auth_user()` только authentication gate;
- 2. как вычисляется `taxi::$id_role`;
- 3. является ли `API->id_role` только контекстом текущего пользователя или частью authorization;
- 4. где выполняются дополнительные role/permission checks;
- 5. существует ли отдельная Authorization Capability.
- Backend RP-02 Authentication
- Backend RP-03 Authentication Deep Trace
- Semantic_Graph_V1_Authentication_End_to_End_Crystallization
- > В доступном текущем source corpus тело определения `check_auth_user()` не было найдено. Поэтому сам факт вызова функции исследован, а её внутренняя реализация пока не может быть подтверждена непосредственно.
- Это является `SOURCE_GAP`, а не доказательством отсутствия authorization logic.
- # 3. Первое наблюдение: check_auth_user является систематическим gate
- check_auth_user();
- $API->id_role = taxi::$id_role;
- /token
- check_auth_user();
- $API->id_role = taxi::$id_role;
- check_auth_user();
- $API->id_role = taxi::$id_role;
- Authentication Gate
- check_auth_user()
- check_auth_user()
- = Authorization
- # 5. Второе наблюдение: после gate устанавливается API->id_role
- check_auth_user()
- taxi::$id_role
- $API->id_role
- /token
- Authenticated API Context
- HAS_CURRENT_ROLE
- Role
- Current User Role Context
- # 6. Но id_role ещё не означает authorization
- check_auth_user()
- $API->id_role = taxi::$id_role
- knows current role
- checks permission for this role
- Role
- Это согласуется с ранее проведённым MCR-002: попытки выразить правило вида «доступно только X» через простые Role/API Relations не дали достаточной семантики. fileciteturn17file18
- # 7. Third observation: auth_user содержит два role-поля
- 'u_role'   => $_SESSION['id_role']
- 'u_a_role' => $API->id_role
- session user role
- API-level / actual role context
- Это важнее, чем просто наличие `id_role`.
- u_role
- $_SESSION['id_role']
- u_a_role
- $API->id_role
- u_role = base role
- u_a_role = active role
- u_role = user role
- u_a_role = authorization role
- без тела `check_auth_user()` и связанных role-resolution functions.
- C-ROLE-001
- Session Role
- C-ROLE-002
- API Role
- API Role
- IS_AUTHORIZATION_ROLE
- registerUser(..., taxi::$data['user_roles'])
- check_auth_user();
- $API->id_role = taxi::$id_role;
- Это подтверждает, что `user_roles` является configuration/data input для user registration. fileciteturn17file0turn17file11
- user_roles
- Authorization Policy
- # 10. `query_roles` — отдельный сильный сигнал
- taxi::$data['site_constants']['query_roles']['value']
- query_roles
- query_roles
- = complete authorization model
- и использование `query_roles`.
- # 11. `session_token` и роль

### `Backend_Semantic_Graph_Research_Pass_05_Role_Resolution_Trace_v0.2.md`

Найдено релевантных строк: **205**.

- # Role Resolution Trace v0.2
- **Предшествующий проход:** RP-05 Role Resolution Trace v0.1
- taxi::$id_role writer
- check_auth_user() implementation
- > `check_auth_user()` найден, и writer для `taxi::$id_role` найден.
- function check_auth_user()
- 6. получение $_SESSION['id_role']
- 7. установка taxi::$id_role
- 8. обработка request u_a_role
- 9. role validation через taxi::$data['user_roles']
- 10. подключение role-specific config
- Это уже достаточное Evidence для существенной части Role Resolution.
- # 3. Authentication Gate теперь полностью наблюдаем
- `check_auth_user()` начинается с:
- id_role,
- Следовательно, `check_auth_user()` не просто проверяет наличие session cookie.
- Следовательно, `check_auth_user()` действительно проверяет не только authentication identity, но и актуальную допустимость authenticated session.
- Authenticated Session
- Authenticated Session
- Authenticated Session
- Authentication-relevant ban state
- # 5. Writer для $_SESSION['id_role']
- Внутри `check_auth_user()`:
- $_SESSION['id_role'] = $d['id_role'];
- $d['id_role']
- users.id_role
- users.id_role
- $_SESSION['id_role']
- # 6. Writer для taxi::$id_role
- taxi::$id_role = $_SESSION['id_role'];
- Это базовое значение API role context.
- users.id_role
- $_SESSION['id_role']
- taxi::$id_role
- if (isset($_REQUEST['u_a_role']))
- $u_a_role = trim($_REQUEST['u_a_role']);
- Затем `taxi::$id_role` может быть изменён.
- $_SESSION['id_role']
- taxi::$id_role
- является runtime role context, который может быть изменён относительно исходной роли.
- # 8. Role switching для session role = 4
- $_SESSION['id_role'] == 4
- if (!empty(taxi::$data['user_roles'][$u_a_role]))
- taxi::$id_role = $u_a_role;
- u_a_role
- taxi::$id_role
- taxi::$data['user_roles']
- # 9. Role switching для session role = 6
- $_SESSION['id_role'] == 6
- $u_a_role != 4
- !empty(taxi::$data['user_roles'][$u_a_role])
- taxi::$id_role = $u_a_role;
- # 10. Role switching для остальных ролей
- $u_a_role != 4
- $u_a_role != 6
- !empty(taxi::$data['user_roles'][$u_a_role])
- taxi::$id_role = $u_a_role;
- users.id_role
- $_SESSION['id_role']
- base/current User Role
- taxi::$id_role
- optional u_a_role
- └── validated against user_roles
- u_role
- u_a_role
- u_role
- = $_SESSION['id_role']
- u_a_role
- = $API->id_role
- а `$API->id_role` получает значение из runtime:
- taxi::$id_role
- # 12. Новый вывод: API Role — не просто копия User Role
- u_role
- u_a_role
- u_role
- users.id_role
- → $_SESSION['id_role']
- u_a_role
- представляет runtime API role:
- taxi::$id_role

### `Backend_Semantic_Graph_Research_Pass_18_Authorization_Control_Flow_v0.1.md`

Найдено релевантных строк: **318**.

- # Authorization Control-Flow Normalization v0.1
- **Предшествующий проход:** RP-17 Role × Operation Semantic Matrix v0.1
- Нормализовать целые control-flow ветви вокруг role checks, чтобы отличить ALLOW, REJECT, PRECONDITION и UNKNOWN.
- ## 2. Подтверждённый role mapping
- | ID | Business Role |
- role condition
- Отдельная строка `id_role != N` без восстановления branch semantics не становится ALLOW.
- Всего direct role-check contexts: **161**; уникальных function contexts: **72**.
- - role IDs: `4=Administrator`
- 10804: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 23051: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 26613: 			if ($this->id_role != 4 && ($d['customer'] != $_SESSION[UID] || $d['performer'] != $_SESSION[UID])) return $this->showError('404', 'error', 'user not customer or performer');
- - role IDs: `1=Client, 2=Driver, 4=Administrator, 5=Agent`
- 7055: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5 && $this->id_role != 4)
- 7057: 				return $this->showError('404', 'error', 'wrong user role');
- 7066: 					if (!empty($site_cancel_states[$c_s]['user_roles']) && in_array($this->id_role,$site_cancel_states[$c_s]['user_roles']))
- 7082: 			if ($this->id_role == 1 || $this->id_role == 5 || $this->id_role == 4)
- 7106: 				if ($this->id_role != 4 && $d['client'] != $_SESSION[UID])
- - role IDs: `4=Administrator, 6=Usher`
- 24234: 			if ($this->id_role != 4 && $this->id_role != 6)
- 24236: 				return $this->showError('404', 'error', 'wrong role');
- 24245: 			if ($this->id_role == 6)
- - role IDs: `4=Administrator`
- 26437: 			if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
- - role IDs: `1=Client, 2=Driver, 5=Agent`
- 6901: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
- 6903: 				return $this->showError('404', 'error', 'wrong user role');
- 6906: 			if ($this->id_role == 1 || $this->id_role == 5)
- 6949: 			elseif ($this->id_role == 2)
- - role IDs: `4=Administrator`
- 26784: 				if ($this->id_role != 4 && $d['performer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not performer');
- 26788: 				if ($this->id_role != 4 && $d['customer'] != $_SESSION[UID]) return $this->showError('404', 'error', 'user not customer');
- - role IDs: `1=Client, 5=Agent`
- 8186: 			if ($this->id_role != 1 && $this->id_role != 5)
- 8188: 				return $this->showError('404', 'error', 'wrong user role');
- - role IDs: `2=Driver, 4=Administrator`
- 2145: 					if ($this->id_role == 2)
- 2164: 						return $this->showError('404', 'error', 'wrong user role');
- 2169: 					if ($this->id_role != 4)
- 2175: 							`id_role`,
- 2188: 					if ($this->id_role != 2) return $this->showError('404', 'error', 'wrong role of user');
- 2200: 				if (!empty($id_user) && $this->id_role != 4)
- 2225: 					if ($this->id_role == 4 || (in_array($_SESSION[UID], explode(',',$d['u_id']))
- 2255: 						return $this->showError('404', 'error', 'not enough rights or wrong user role');
- - role IDs: `4=Administrator`
- 22901: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 21163: 				'token'	=>		array(
- 21164: 										'name'	=>	'token',
- 21264: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- 21268: 					if ($filtered_data['owner'] != $_SESSION[UID] && $this->id_role != 4) return $this->showError('404', 'error', 'not enough rights for action');
- - role IDs: `4=Administrator`
- 13730: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- 13751: 				if ($this->id_role == 4)
- - role IDs: `1=Client, 4=Administrator, 5=Agent`
- 2849: 			if ($this->id_role != 1 && $this->id_role != 5 && $this->id_role != 4)
- 2851: 				return $this->showError('404', 'error', 'wrong user role');
- - role IDs: `4=Administrator`
- 24772: 				if ($this->id_role != 4)
- - role IDs: `4=Administrator`
- 25064: 			if ($this->id_role != 4)
- - role IDs: `4=Administrator`
- 22616: 			if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `2=Driver, 4=Administrator`
- 11373: 				if ($this->id_role != 2)
- 11375: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'wrong user role');
- 11392: 				if ($this->id_role != 4) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 23390: 			if ($this->id_role != 4 && $d['user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 25381: 				if ($this->id_role != 4 && $d['id_user'] != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
- - role IDs: `4=Administrator`
- 21367: 				'token'	=>		array(
- 21368: 										'name'	=>	'token',
- 21446: 			if ($this->id_role != 4 && ($d['id_owner_type'] != 1 || $d['owner'] != $_SESSION[UID])) return $this->showError('404', 'error', 'not enough rights');
- 21448: 			if ($this->id_role != 4)
- - role IDs: `2=Driver, 4=Administrator`
- 14346: 			if ($this->id_role == 2 && $_SESSION['id_verification_status'] != 2)

### `Backend_Semantic_Graph_Research_Pass_20_Authorization_Helper_Trace_v0.1.md`

Найдено релевантных строк: **145**.

- # Authorization Helper Trace v0.1
- > Являются ли разобранные role-sensitive checks реализациями одного общего authorization helper или независимыми локальными gates?
- Контекстов, содержащих явное определение известных authorization helper names: **1**.
- 14: 		check_auth_user();
- 19: 		elseif ($_SESSION['id_role'] != 4)
- 247: 	function check_auth_user()
- 249: 		taxi::$check_auth_user_count++;
- 252: 				`id_role`,
- 4745: 		check_auth_user();
- 2: check_auth_user();
- 22: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 23: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false);
- 36: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 41: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 36: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 41: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 46: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 41: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 46: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 53: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
- 104: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 128: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 211: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 300: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 304: 			elseif ($_GET['par'][1] == 'token')
- 306: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 304: 			elseif ($_GET['par'][1] == 'token')
- 306: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
- 343: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 348: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 343: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 348: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 383: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 389: 							check_auth_user(); $API->id_role = taxi::$id_role;
- 389: 							check_auth_user(); $API->id_role = taxi::$id_role;
- 404: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 426: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 452: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 467: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 475: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 475: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 493: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 520: 					if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
- 528: 											'u_role' => $_SESSION['id_role'],
- 549: 							if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
- 557: 						if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
- 581: 						if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
- 589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
- 26: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 27: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false,isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
- 40: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 45: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 40: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 45: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 50: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 45: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 50: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 55: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array(),isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
- 112: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 136: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 219: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 308: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 312: 			elseif ($_GET['par'][1] == 'token')
- 314: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 312: 			elseif ($_GET['par'][1] == 'token')
- 314: 				check_auth_user(); $API->id_role = taxi::$id_role;
- 317: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
- 351: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 356: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 351: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 356: 					check_auth_user(); $API->id_role = taxi::$id_role;
- 391: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 399: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 399: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 404: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 399: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 404: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 409: 								check_auth_user(); $API->id_role = taxi::$id_role;
- 404: 								check_auth_user(); $API->id_role = taxi::$id_role;

## 3. Frontend credential/injection evidence

Найдено frontend contexts: **231**.

### `src/serviceWorker.js:104`
```text
98:     })
99: }
100: 
101: function checkValidServiceWorker(swUrl, config) {
102:   // Check if the service worker can be found. If it can't reload the page.
103:   fetch(swUrl, {
104:     headers: { 'Service-Worker': 'script' },
105:   })
106:     .then(response => {
107:       // Ensure service worker exists, and that we really are getting a JS file.
108:       const contentType = response.headers.get('content-type')
109:       if (
110:         response.status === 404 ||
111:         (contentType != null && contentType.indexOf('javascript') === -1)
112:       ) {
113:         // No service worker found. Probably a different app. Reload the page.
114:         navigator.serviceWorker.ready.then(registration => {
```

### `src/serviceWorker.js:108`
```text
102:   // Check if the service worker can be found. If it can't reload the page.
103:   fetch(swUrl, {
104:     headers: { 'Service-Worker': 'script' },
105:   })
106:     .then(response => {
107:       // Ensure service worker exists, and that we really are getting a JS file.
108:       const contentType = response.headers.get('content-type')
109:       if (
110:         response.status === 404 ||
111:         (contentType != null && contentType.indexOf('javascript') === -1)
112:       ) {
113:         // No service worker found. Probably a different app. Reload the page.
114:         navigator.serviceWorker.ready.then(registration => {
115:           registration.unregister().then(() => {
116:             window.location.reload()
117:           })
118:         })
```

### `src/config.ts:63`
```text
57:         this.setDefaultName()
58:       }
59:     }
60:   }
61: 
62:   setConfig(name: string) {
63:     localStorage.setItem('config', name)
64:     _configName = name
65:     applyConfigName(this.API_URL, name)
66:   }
67: 
68:   clearConfig() {
69:     localStorage.removeItem('config')
70:     _configName = ''
71:     applyConfigName(this.API_URL)
72:   }
73: 
```

### `src/config.ts:69`
```text
63:     localStorage.setItem('config', name)
64:     _configName = name
65:     applyConfigName(this.API_URL, name)
66:   }
67: 
68:   clearConfig() {
69:     localStorage.removeItem('config')
70:     _configName = ''
71:     applyConfigName(this.API_URL)
72:   }
73: 
74:   setDefaultName() {
75:     applyConfigName(this.API_URL)
76:   }
77: 
78:   get API_URL() {
79:     return `${this.SERVER_URL}/api/v1`
```

### `src/config.ts:87`
```text
81: 
82:   get SERVER_URL() {
83:     return `https://ibronevik.ru/taxi/c/${_configName || DEFAULT_CONFIG_NAME}`
84:   }
85: 
86:   get SavedConfig() {
87:     return localStorage.getItem('config')
88:   }
89: }
90: 
91: const config = new Config()
92: 
93: export default config
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:321`
```text
315: }
316: 
317: // We need to provide a custom onError function for httpProxyMiddleware.
318: // It allows us to log custom error messages on the console.
319: function onProxyError(proxy) {
320:   return (err, req, res) => {
321:     const host = req.headers && req.headers.host;
322:     console.log(
323:       chalk.red('Proxy error:') +
324:         ' Could not proxy request ' +
325:         chalk.cyan(req.url) +
326:         ' from ' +
327:         chalk.cyan(host) +
328:         ' to ' +
329:         chalk.cyan(proxy) +
330:         '.'
331:     );
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:341`
```text
335:         ').'
336:     );
337:     console.log();
338: 
339:     // And immediately send the proper error response to the client.
340:     // Otherwise, the request will eventually timeout with ERR_EMPTY_RESPONSE on the client side.
341:     if (res.writeHead && !res.headersSent) {
342:       res.writeHead(500);
343:     }
344:     res.end(
345:       'Proxy error: Could not proxy request ' +
346:         req.url +
347:         ' from ' +
348:         host +
349:         ' to ' +
350:         proxy +
351:         ' (' +
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:426`
```text
420:       // However API calls like `fetch()` won’t generally accept text/html.
421:       // If this heuristic doesn’t work well for you, use `src/setupProxy.js`.
422:       context: function(pathname, req) {
423:         return (
424:           req.method !== 'GET' ||
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:427`
```text
421:       // If this heuristic doesn’t work well for you, use `src/setupProxy.js`.
422:       context: function(pathname, req) {
423:         return (
424:           req.method !== 'GET' ||
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
437:       },
```

### `lib/react-dev-utils/WebpackDevServerUtils.js:431`
```text
425:           (mayProxy(pathname) &&
426:             req.headers.accept &&
427:             req.headers.accept.indexOf('text/html') === -1)
428:         );
429:       },
430:       onProxyReq: proxyReq => {
431:         // Browsers may send Origin headers even with same-origin
432:         // requests. To prevent CORS issues, we have to change
433:         // the Origin to match the target URL.
434:         if (proxyReq.getHeader('origin')) {
435:           proxyReq.setHeader('origin', target);
436:         }
437:       },
438:       onError: onProxyError(target),
439:       secure: false,
440:       changeOrigin: true,
441:       ws: true,
```

### `lib/react-dev-utils/formatWebpackMessages.js:33`
```text
27:       if ('message' in message) {
28:         lines = message['message'].split('\n');
29:       }
30:     });
31:   }
32: 
33:   // Strip webpack-added headers off errors/warnings
34:   // https://github.com/webpack/webpack/blob/master/lib/ModuleError.js
35:   lines = lines.filter(line => !/Module [A-z ]+\(from/.test(line));
36: 
37:   // Transform parsing error into syntax error
38:   // TODO: move this to our ESLint formatter?
39:   lines = lines.map(line => {
40:     const parsingError = /Line (\d+):(?:(\d+):)?\s*Parsing error: (.+)$/.exec(
41:       line
42:     );
43:     if (!parsingError) {
```

### `src/utils/cookies.ts:1`
```text
1: export const setCookie = (name: string, value: string, days: number = 365) => {
2:   const date = new Date();
3:   date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
4:   const expires = `expires=${date.toUTCString()}`;
5:   document.cookie = `${name}=${value};${expires};path=/`;
6: };
7: 
8: export const getCookie = (name: string): string | null => {
9:   const nameEQ = `${name}=`;
10:   const ca = document.cookie.split(';');
11:   for (let i = 0; i < ca.length; i++) {
```

### `src/utils/cookies.ts:5`
```text
1: export const setCookie = (name: string, value: string, days: number = 365) => {
2:   const date = new Date();
3:   date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
4:   const expires = `expires=${date.toUTCString()}`;
5:   document.cookie = `${name}=${value};${expires};path=/`;
6: };
7: 
8: export const getCookie = (name: string): string | null => {
9:   const nameEQ = `${name}=`;
10:   const ca = document.cookie.split(';');
11:   for (let i = 0; i < ca.length; i++) {
12:     let c = ca[i];
13:     while (c.charAt(0) === ' ') c = c.substring(1, c.length);
14:     if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
15:   }
```

### `src/utils/cookies.ts:8`
```text
2:   const date = new Date();
3:   date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
4:   const expires = `expires=${date.toUTCString()}`;
5:   document.cookie = `${name}=${value};${expires};path=/`;
6: };
7: 
8: export const getCookie = (name: string): string | null => {
9:   const nameEQ = `${name}=`;
10:   const ca = document.cookie.split(';');
11:   for (let i = 0; i < ca.length; i++) {
12:     let c = ca[i];
13:     while (c.charAt(0) === ' ') c = c.substring(1, c.length);
14:     if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
15:   }
16:   return null;
17: };
18: 
```

### `src/utils/cookies.ts:10`
```text
4:   const expires = `expires=${date.toUTCString()}`;
5:   document.cookie = `${name}=${value};${expires};path=/`;
6: };
7: 
8: export const getCookie = (name: string): string | null => {
9:   const nameEQ = `${name}=`;
10:   const ca = document.cookie.split(';');
11:   for (let i = 0; i < ca.length; i++) {
12:     let c = ca[i];
13:     while (c.charAt(0) === ' ') c = c.substring(1, c.length);
14:     if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
15:   }
16:   return null;
17: };
18: 
19: export const deleteCookie = (name: string) => {
20:   document.cookie = `${name}=;expires=Thu, 01 Jan 1970 00:00:00 GMT;path=/`;
```

### `src/utils/cookies.ts:19`
```text
13:     while (c.charAt(0) === ' ') c = c.substring(1, c.length);
14:     if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
15:   }
16:   return null;
17: };
18: 
19: export const deleteCookie = (name: string) => {
20:   document.cookie = `${name}=;expires=Thu, 01 Jan 1970 00:00:00 GMT;path=/`;
21: }; 
```

### `src/utils/cookies.ts:20`
```text
14:     if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
15:   }
16:   return null;
17: };
18: 
19: export const deleteCookie = (name: string) => {
20:   document.cookie = `${name}=;expires=Thu, 01 Jan 1970 00:00:00 GMT;path=/`;
21: }; 
```

### `src/types/types.ts:621`
```text
615:   //
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
```

### `src/types/types.ts:622`
```text
616:   u_registration: Moment
617:   u_replies?: IReply[]
618:   u_choosen?: number
619:   ref_code?: string
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
```

### `src/types/types.ts:626`
```text
620:   role: EUserRoles
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
```

### `src/types/types.ts:627`
```text
621:   token?: string
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
```

### `src/types/types.ts:628`
```text
622:   u_hash?: string
623:   uploads?: any[]
624: }
625: 
626: export interface ITokens {
627:   token: string,
628:   u_hash: string
629: }
630: 
631: export interface IStaticMarker {
632:   latitude: number,
633:   longitude: number,
634:   popup?: string,
635:   tooltip?: string
636: }
637: 
638: export interface IAddressPoint {
```

### `src/types/types.ts:753`
```text
747: 
748: export type TBlockObject = { [key: string]: any }
749: 
750: export interface IRegisterResponse {
751:   u_id: number,
752:   string: string,
753:   token?: string,
754:   u_hash?: string
755: }
756: 
757: export type IFileUpload = {
758:   base64: string,
759:   name?: string,
760:   u_id?: string,
761:   private?: 0 | 1
762: }
763: 
```

### `src/types/types.ts:754`
```text
748: export type TBlockObject = { [key: string]: any }
749: 
750: export interface IRegisterResponse {
751:   u_id: number,
752:   string: string,
753:   token?: string,
754:   u_hash?: string
755: }
756: 
757: export type IFileUpload = {
758:   base64: string,
759:   name?: string,
760:   u_id?: string,
761:   private?: 0 | 1
762: }
763: 
764: 
```

### `src/API/order.ts:17`
```text
11:   IUser,
12: } from '../types/types'
13: import { IResponse } from '../types/api'
14: import { cloneFormData } from '../tools/utils'
15: import { convertOrder, reverseConvertOrder } from '../tools/convert'
16: import {
17:   addToFormData, apiMethod, IApiMethodArguments, IResponseFields,
18: } from '../tools/api'
19: import Config from '../config'
20: import { t, TRANSLATION } from '../localization'
21: import store from '../state'
22: import { userSelectors } from '../state/user'
23: import { EBookingActions } from './constants'
24: import { getUserCar } from './car'
25: 
26: async function _postDrive(
27:   { formData }: IApiMethodArguments,
```

### `src/API/order.ts:27`
```text
21: import store from '../state'
22: import { userSelectors } from '../state/user'
23: import { EBookingActions } from './constants'
24: import { getUserCar } from './car'
25: 
26: async function _postDrive(
27:   { formData }: IApiMethodArguments,
28:   data: IOrder,
29: ): Promise<IResponseFields & {
30:   b_id: IOrder['b_id'],
31:   b_driver_code: IOrder['b_driver_code']
32: }> {
33:   const defaults: Partial<IOrder> = {
34:     b_payment_way: EPaymentWays.Cash,
35:   }
36: 
37:   const converted = reverseConvertOrder({
```

### `src/API/order.ts:93`
```text
87:     })
88:     await axios.post(`${Config.API_URL}/drive/get/${result.b_id}`, params)
89:   }
90: 
91:   return result
92: }
93: export const postDrive = apiMethod<typeof _postDrive>(_postDrive)
94: 
95: const _cancelDrive = (
96:   { formData }: IApiMethodArguments,
97:   id: IOrder['b_id'],
98:   reason?: IOrder['b_cancel_reason'],
99: ) => {
100:   addToFormData(formData, {
101:     action: EBookingActions.SetCancelState,
102:     reason,
103:   })
```

### `src/API/order.ts:96`
```text
90: 
91:   return result
92: }
93: export const postDrive = apiMethod<typeof _postDrive>(_postDrive)
94: 
95: const _cancelDrive = (
96:   { formData }: IApiMethodArguments,
97:   id: IOrder['b_id'],
98:   reason?: IOrder['b_cancel_reason'],
99: ) => {
100:   addToFormData(formData, {
101:     action: EBookingActions.SetCancelState,
102:     reason,
103:   })
104: 
105:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
106:     .then(res => res.data)
```

### `src/API/order.ts:108`
```text
102:     reason,
103:   })
104: 
105:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
106:     .then(res => res.data)
107: }
108: export const cancelDrive = apiMethod<typeof _cancelDrive>(_cancelDrive)
109: 
110: export const getOrders: <TType extends EOrderTypes>(
111:   type?: TType,
112:   filter?: TType extends EOrderTypes.Ready ? {
113:     carClasses?: boolean
114:     locationClasses?: boolean
115:   } : undefined
116: ) => Promise<IResponse<'200', {
117:   booking: IOrder[]
118: }> | IResponse<'404', {
```

### `src/API/order.ts:120`
```text
114:     locationClasses?: boolean
115:   } : undefined
116: ) => Promise<IResponse<'200', {
117:   booking: IOrder[]
118: }> | IResponse<'404', {
119:   detail?: 'used_car_not_found'
120: }>> = apiMethod(async(
121:   { formData }: IApiMethodArguments,
122:   type: EOrderTypes = EOrderTypes.Active,
123:   filter: {
124:     carClasses?: boolean
125:     locationClasses?: boolean
126:   } = {},
127: ) => {
128:   const userID = userSelectors.user(store.getState())?.u_id
129: 
130:   addToFormData(formData, {
```

### `src/API/order.ts:121`
```text
115:   } : undefined
116: ) => Promise<IResponse<'200', {
117:   booking: IOrder[]
118: }> | IResponse<'404', {
119:   detail?: 'used_car_not_found'
120: }>> = apiMethod(async(
121:   { formData }: IApiMethodArguments,
122:   type: EOrderTypes = EOrderTypes.Active,
123:   filter: {
124:     carClasses?: boolean
125:     locationClasses?: boolean
126:   } = {},
127: ) => {
128:   const userID = userSelectors.user(store.getState())?.u_id
129: 
130:   addToFormData(formData, {
131:     array_type: 'list',
```

### `src/API/order.ts:134`
```text
128:   const userID = userSelectors.user(store.getState())?.u_id
129: 
130:   addToFormData(formData, {
131:     array_type: 'list',
132:   })
133: 
134:   const hiddenOrders = JSON.parse(localStorage.getItem('hiddenOrders') || '{}')
135:   const userHiddenOrders = hiddenOrders && userID && hiddenOrders[userID]
136: 
137:   const queryParams: string[] = []
138:   let URLAdditionalPath = ''
139:   switch (type) {
140:     case EOrderTypes.Active:
141:       queryParams.push('fields=00000000u1')
142:       break
143:     case EOrderTypes.Ready:
144:       URLAdditionalPath = '/now'
```

### `src/API/order.ts:192`
```text
186:         ),
187:     },
188:   }
189: })
190: 
191: const _getOrder = (
192:   { formData }: IApiMethodArguments,
193:   id: IOrder['b_id'],
194: ): Promise<IOrder | null> => {
195:   return axios.post(`${Config.API_URL}/drive/get/${id}?fields=00000000u1`, formData)
196:     .then(res => res.data.data)
197:     .then(res => (res.booking && res.booking[id] && convertOrder(res.booking[id])) || null)
198: }
199: export const getOrder = apiMethod<typeof _getOrder>(_getOrder)
200: 
201: const _editOrder = (
202:   { formData }: IApiMethodArguments,
```

### `src/API/order.ts:199`
```text
193:   id: IOrder['b_id'],
194: ): Promise<IOrder | null> => {
195:   return axios.post(`${Config.API_URL}/drive/get/${id}?fields=00000000u1`, formData)
196:     .then(res => res.data.data)
197:     .then(res => (res.booking && res.booking[id] && convertOrder(res.booking[id])) || null)
198: }
199: export const getOrder = apiMethod<typeof _getOrder>(_getOrder)
200: 
201: const _editOrder = (
202:   { formData }: IApiMethodArguments,
203:   id: IOrder['b_id'],
204:   data: IBookingAddresses & Stringify<IBookingCoordinates>,
205: ): Promise<IResponseFields> => {
206:   addToFormData(formData, {
207:     action: EBookingActions.Edit,
208:     data: JSON.stringify(data),
209:   })
```

### `src/API/order.ts:202`
```text
196:     .then(res => res.data.data)
197:     .then(res => (res.booking && res.booking[id] && convertOrder(res.booking[id])) || null)
198: }
199: export const getOrder = apiMethod<typeof _getOrder>(_getOrder)
200: 
201: const _editOrder = (
202:   { formData }: IApiMethodArguments,
203:   id: IOrder['b_id'],
204:   data: IBookingAddresses & Stringify<IBookingCoordinates>,
205: ): Promise<IResponseFields> => {
206:   addToFormData(formData, {
207:     action: EBookingActions.Edit,
208:     data: JSON.stringify(data),
209:   })
210: 
211:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
212:     .then(res => res.data)
```

### `src/API/order.ts:214`
```text
208:     data: JSON.stringify(data),
209:   })
210: 
211:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
212:     .then(res => res.data)
213: }
214: export const editOrder = apiMethod<typeof _editOrder>(_editOrder)
215: 
216: const _takeOrder = (
217:   { formData }: IApiMethodArguments,
218:   id: IOrder['b_id'],
219:   options: {
220:     votingNumber: number
221:     performers_price: number
222:   },
223:   candidate?: boolean,
224: ): Promise<{
```

### `src/API/order.ts:217`
```text
211:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
212:     .then(res => res.data)
213: }
214: export const editOrder = apiMethod<typeof _editOrder>(_editOrder)
215: 
216: const _takeOrder = (
217:   { formData }: IApiMethodArguments,
218:   id: IOrder['b_id'],
219:   options: {
220:     votingNumber: number
221:     performers_price: number
222:   },
223:   candidate?: boolean,
224: ): Promise<{
225:   /** Индекс водителя */
226:   c_index: string,
227:   /** Текущее число машин поездки с booking_driver_states=3,4,5,6 */
```

### `src/API/order.ts:259`
```text
253: 
254:       return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
255:         .then(res => res.data)
256:         .then(res => res.status === 'error' ? Promise.reject(res.message) : res)
257:     })
258: }
259: export const takeOrder = apiMethod<typeof _takeOrder>(_takeOrder)
260: 
261: const _chooseCandidate = (
262:   { formData }: IApiMethodArguments,
263:   id: IOrder['b_id'],
264:   user?: IUser['u_id'],
265: ): Promise<any> => {
266:   const userID = userSelectors.user(store.getState())?.u_id
267:   if (!userID) Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
268: 
269:   addToFormData(formData, {
```

### `src/API/order.ts:262`
```text
256:         .then(res => res.status === 'error' ? Promise.reject(res.message) : res)
257:     })
258: }
259: export const takeOrder = apiMethod<typeof _takeOrder>(_takeOrder)
260: 
261: const _chooseCandidate = (
262:   { formData }: IApiMethodArguments,
263:   id: IOrder['b_id'],
264:   user?: IUser['u_id'],
265: ): Promise<any> => {
266:   const userID = userSelectors.user(store.getState())?.u_id
267:   if (!userID) Promise.reject(t(TRANSLATION.WRONG_USER_ROLE))
268: 
269:   addToFormData(formData, {
270:     action: EBookingActions.SetPerformer,
271:     performer: '1',
272:     u_id: user,
```

### `src/API/order.ts:279`
```text
273:   })
274: 
275:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
276:     .then(res => res.data)
277:     .then(res => res.status === 'error' ? Promise.reject() : res)
278: }
279: export const chooseCandidate = apiMethod<typeof _chooseCandidate>(_chooseCandidate)
280: 
281: const _setOrderState = (
282:   { formData }: IApiMethodArguments,
283:   id: IOrder['b_id'],
284:   state: EBookingDriverState,
285: ) => {
286:   let action
287:   switch (state) {
288:     case EBookingDriverState.Arrived:
289:       action = EBookingActions.SetArriveState
```

### `src/API/order.ts:282`
```text
276:     .then(res => res.data)
277:     .then(res => res.status === 'error' ? Promise.reject() : res)
278: }
279: export const chooseCandidate = apiMethod<typeof _chooseCandidate>(_chooseCandidate)
280: 
281: const _setOrderState = (
282:   { formData }: IApiMethodArguments,
283:   id: IOrder['b_id'],
284:   state: EBookingDriverState,
285: ) => {
286:   let action
287:   switch (state) {
288:     case EBookingDriverState.Arrived:
289:       action = EBookingActions.SetArriveState
290:       break
291:     case EBookingDriverState.Started:
292:       action = EBookingActions.SetStartState
```

### `src/API/order.ts:309`
```text
303:   })
304: 
305:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
306:     .then(res => res.data)
307:     .then(res => res.status === 'error' ? Promise.reject() : res)
308: }
309: export const setOrderState = apiMethod<typeof _setOrderState>(_setOrderState)
310: 
311: const _setOrderRating = (
312:   { formData }: IApiMethodArguments,
313:   id: IOrder['b_id'],
314:   value: number,
315: ) => {
316:   addToFormData(formData, {
317:     action: EBookingActions.SetRate,
318:     value,
319:   })
```

### `src/API/order.ts:312`
```text
306:     .then(res => res.data)
307:     .then(res => res.status === 'error' ? Promise.reject() : res)
308: }
309: export const setOrderState = apiMethod<typeof _setOrderState>(_setOrderState)
310: 
311: const _setOrderRating = (
312:   { formData }: IApiMethodArguments,
313:   id: IOrder['b_id'],
314:   value: number,
315: ) => {
316:   addToFormData(formData, {
317:     action: EBookingActions.SetRate,
318:     value,
319:   })
320: 
321:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
322:     .then(res => res.data)
```

### `src/API/order.ts:327`
```text
321:   return axios.post(`${Config.API_URL}/drive/get/${id}`, formData)
322:     .then(res => res.data)
323: }
324: /**
325:  * @param value rating from 1 till 5
326:  */
327: export const setOrderRating = apiMethod<typeof _setOrderRating>(_setOrderRating)
328: 
329: const _setWaitingTime = (
330:   { formData }: IApiMethodArguments,
331:   id: IOrder['b_id'],
332:   previous: number,
333:   additional: number = 180,
334: ) => {
335:   addToFormData(formData, {
336:     action: EBookingActions.SetWaitingTime,
337:     previous,
```

### `src/API/order.ts:330`
```text
324: /**
325:  * @param value rating from 1 till 5
326:  */
327: export const setOrderRating = apiMethod<typeof _setOrderRating>(_setOrderRating)
328: 
329: const _setWaitingTime = (
330:   { formData }: IApiMethodArguments,
331:   id: IOrder['b_id'],
332:   previous: number,
333:   additional: number = 180,
334: ) => {
335:   addToFormData(formData, {
336:     action: EBookingActions.SetWaitingTime,
337:     previous,
338:     additional,
339:   })
340: 
```

### `src/API/order.ts:348`
```text
342:     .then(res => res.data)
343: }
344: /**
345:  * Adds time to wait
346:  * @param previous actual waiting time
347:  */
348: export const setWaitingTime = apiMethod<typeof _setWaitingTime>(_setWaitingTime)
```

### `src/API/car.ts:5`
```text
1: import axios from 'axios'
2: import { EBookingLocationKinds, ICar, IUser } from '../types/types'
3: import { IResponse } from '../types/api'
4: import { convertCar } from '../tools/convert'
5: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
6: import Config from '../config'
7: import SITE_CONSTANTS from '../siteConstants'
8: 
9: type TCreateCar = Pick<ICar,
10:   'cm_id' |
11:   'seats' |
12:   'registration_plate' |
13:   'color' |
14:   'cc_id'
15: > & Partial<Pick<ICar,
```

### `src/API/car.ts:26`
```text
20:   created_car: {
21:     c_id: ICar['c_id'],
22:     u_id: IUser['u_id']
23:   }
24: }
25: 
26: export const createCar = apiMethod(async(
27:   { formData }: IApiMethodArguments,
28:   fields: {
29:     u_id: IUser['u_id']
30:   } & TCreateCar,
31: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
32:   const { u_id, ...formFields } = fields
33:   addToFormData(formData, { data: JSON.stringify(formFields) })
34:   const { data } =
35:     await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)
36:   return data
```

### `src/API/car.ts:27`
```text
21:     c_id: ICar['c_id'],
22:     u_id: IUser['u_id']
23:   }
24: }
25: 
26: export const createCar = apiMethod(async(
27:   { formData }: IApiMethodArguments,
28:   fields: {
29:     u_id: IUser['u_id']
30:   } & TCreateCar,
31: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
32:   const { u_id, ...formFields } = fields
33:   addToFormData(formData, { data: JSON.stringify(formFields) })
34:   const { data } =
35:     await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)
36:   return data
37: })
```

### `src/API/car.ts:40`
```text
34:   const { data } =
35:     await axios.post(`${Config.API_URL}/user/${u_id}/car`, formData)
36:   return data
37: })
38: 
39: const _getCar = (
40:   { formData }: IApiMethodArguments,
41:   id: ICar['c_id'],
42: ): Promise<ICar | null> => {
43:   return axios.post(`${Config.API_URL}/car/${id}`, formData)
44:     .then(res => res.data.data)
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
```

### `src/API/car.ts:47`
```text
41:   id: ICar['c_id'],
42: ): Promise<ICar | null> => {
43:   return axios.post(`${Config.API_URL}/car/${id}`, formData)
44:     .then(res => res.data.data)
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
```

### `src/API/car.ts:50`
```text
44:     .then(res => res.data.data)
45:     .then(res => (res.car && res.car[id] && convertCar(res.car[id])) || null)
46: }
47: export const getCar = apiMethod<typeof _getCar>(_getCar)
48: 
49: const _getCars = (
50:   { formData }: IApiMethodArguments,
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
```

### `src/API/car.ts:57`
```text
51:   ids: IUser['u_id'][],
52: ): Promise<ICar[]> => {
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
```

### `src/API/car.ts:59`
```text
53:   return axios.post(`${Config.API_URL}/car/${ids.join(',')}`, formData)
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
```

### `src/API/car.ts:60`
```text
54:     .then(res => res.data.data)
55:     .then(res => Object.values(res.car).map(i => convertCar(i)))
56: }
57: export const getCars = apiMethod<typeof _getCars>(_getCars)
58: 
59: export const createUserCar = apiMethod(async(
60:   { formData }: IApiMethodArguments,
61:   fields: TCreateCar & {
62:     country?: keyof typeof SITE_CONSTANTS.COUNTRIES
63:   },
64: ): Promise<IResponse<'200', ICreateCarResponse> | IResponse<'404', {}>> => {
65:   const { country, ...formFields } = fields
66:   addToFormData(formData, { data: JSON.stringify(formFields) })
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
```

### `src/API/car.ts:73`
```text
67:   const { data: response } = await axios.post(`${Config.API_URL}/car`, formData)
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
71: })
72: 
73: const setDefaultCarLicenses = apiMethod(async(
74:   { formData }: IApiMethodArguments,
75:   id: ICar['c_id'],
76:   country: keyof typeof SITE_CONSTANTS.COUNTRIES,
77: ): Promise<void> => {
78:   const locationClass = SITE_CONSTANTS.BOOKING_LOCATION_CLASSES
79:     .find(lc => lc.kind === EBookingLocationKinds.Intercity)!
80:   addToFormData(formData, { data: JSON.stringify({
81:     licenses: [{
82:       en: 'license',
83:       b_l_c: [{
```

### `src/API/car.ts:74`
```text
68:   if (country)
69:     await setDefaultCarLicenses(response.data.created_car.c_id, country)
70:   return response
71: })
72: 
73: const setDefaultCarLicenses = apiMethod(async(
74:   { formData }: IApiMethodArguments,
75:   id: ICar['c_id'],
76:   country: keyof typeof SITE_CONSTANTS.COUNTRIES,
77: ): Promise<void> => {
78:   const locationClass = SITE_CONSTANTS.BOOKING_LOCATION_CLASSES
79:     .find(lc => lc.kind === EBookingLocationKinds.Intercity)!
80:   addToFormData(formData, { data: JSON.stringify({
81:     licenses: [{
82:       en: 'license',
83:       b_l_c: [{
84:         location: locationClass.id,
```

### `src/API/car.ts:93`
```text
87:     }],
88:   }) })
89:   await axios.post(`${Config.API_URL}/car/${id}`, formData)
90: })
91: 
92: const _getUserCars = (
93:   { formData }: IApiMethodArguments,
94: ): Promise<any> => {
95:   return axios.post(`${Config.API_URL}/user/authorized/car`, formData)
96:     .then(res => Object.values(res?.data?.data?.car || {}))
97: }
98: export const getUserCars = apiMethod<typeof _getUserCars>(_getUserCars)
99: 
100: const _getUserCar = (
101:   { formData }: IApiMethodArguments,
102:   id: IUser['u_id'],
103: ): Promise<ICar | null> => {
```

### `src/API/car.ts:98`
```text
92: const _getUserCars = (
93:   { formData }: IApiMethodArguments,
94: ): Promise<any> => {
95:   return axios.post(`${Config.API_URL}/user/authorized/car`, formData)
96:     .then(res => Object.values(res?.data?.data?.car || {}))
97: }
98: export const getUserCars = apiMethod<typeof _getUserCars>(_getUserCars)
99: 
100: const _getUserCar = (
101:   { formData }: IApiMethodArguments,
102:   id: IUser['u_id'],
103: ): Promise<ICar | null> => {
104:   addToFormData(formData, {
105:     array_type: 'list',
106:   })
107: 
108:   return axios.post(`${Config.API_URL}/user/${id}/car`, formData)
```

### `src/API/car.ts:101`
```text
95:   return axios.post(`${Config.API_URL}/user/authorized/car`, formData)
96:     .then(res => Object.values(res?.data?.data?.car || {}))
97: }
98: export const getUserCars = apiMethod<typeof _getUserCars>(_getUserCars)
99: 
100: const _getUserCar = (
101:   { formData }: IApiMethodArguments,
102:   id: IUser['u_id'],
103: ): Promise<ICar | null> => {
104:   addToFormData(formData, {
105:     array_type: 'list',
106:   })
107: 
108:   return axios.post(`${Config.API_URL}/user/${id}/car`, formData)
109:     .then(res => res.data.data)
110:     .then(res => (res.car && res.car[0]) || null)
111: }
```

### `src/API/car.ts:112`
```text
106:   })
107: 
108:   return axios.post(`${Config.API_URL}/user/${id}/car`, formData)
109:     .then(res => res.data.data)
110:     .then(res => (res.car && res.car[0]) || null)
111: }
112: export const getUserCar = apiMethod<typeof _getUserCar>(_getUserCar)
113: 
114: export const editCar = apiMethod(async(
115:   { formData }: IApiMethodArguments,
116:   id: ICar['c_id'],
117:   fields: Partial<Pick<ICar,
118:     'cm_id' |
119:     'seats' |
120:     'registration_plate' |
121:     'color' |
122:     'photo' |
```

### `src/API/car.ts:114`
```text
108:   return axios.post(`${Config.API_URL}/user/${id}/car`, formData)
109:     .then(res => res.data.data)
110:     .then(res => (res.car && res.car[0]) || null)
111: }
112: export const getUserCar = apiMethod<typeof _getUserCar>(_getUserCar)
113: 
114: export const editCar = apiMethod(async(
115:   { formData }: IApiMethodArguments,
116:   id: ICar['c_id'],
117:   fields: Partial<Pick<ICar,
118:     'cm_id' |
119:     'seats' |
120:     'registration_plate' |
121:     'color' |
122:     'photo' |
123:     'details' |
124:     'cc_id'
```

### `src/API/car.ts:115`
```text
109:     .then(res => res.data.data)
110:     .then(res => (res.car && res.car[0]) || null)
111: }
112: export const getUserCar = apiMethod<typeof _getUserCar>(_getUserCar)
113: 
114: export const editCar = apiMethod(async(
115:   { formData }: IApiMethodArguments,
116:   id: ICar['c_id'],
117:   fields: Partial<Pick<ICar,
118:     'cm_id' |
119:     'seats' |
120:     'registration_plate' |
121:     'color' |
122:     'photo' |
123:     'details' |
124:     'cc_id'
125:   >>,
```

### `src/API/car.ts:132`
```text
126: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
127:   addToFormData(formData, { data: JSON.stringify(fields) })
128:   const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)
129:   return data
130: })
131: 
132: export const driveCar = apiMethod(async(
133:   { formData }: IApiMethodArguments,
134:   car: ICar,
135: ): Promise<IResponse<'200', {}> | IResponse<'404', {
136:   detail?: 'not_modified'
137: }>> => {
138:   let { data: response } = await axios.post(
139:     `${Config.API_URL}/car/${car.c_id}/drive`,
140:     formData,
141:   )
142:   if (
```

### `src/API/car.ts:133`
```text
127:   addToFormData(formData, { data: JSON.stringify(fields) })
128:   const { data } = await axios.post(`${Config.API_URL}/car/${id}`, formData)
129:   return data
130: })
131: 
132: export const driveCar = apiMethod(async(
133:   { formData }: IApiMethodArguments,
134:   car: ICar,
135: ): Promise<IResponse<'200', {}> | IResponse<'404', {
136:   detail?: 'not_modified'
137: }>> => {
138:   let { data: response } = await axios.post(
139:     `${Config.API_URL}/car/${car.c_id}/drive`,
140:     formData,
141:   )
142:   if (
143:     response.code === '404' &&
```

### `src/API/car.ts:160`
```text
154:       syncUserResponse.data.detail === 'not_modified'
155:     ))) response = syncUserResponse
156:   }
157:   return response
158: })
159: 
160: const syncUserWithCar = apiMethod(async(
161:   { formData }: IApiMethodArguments,
162:   car: ICar,
163: ): Promise<IResponse<'200', {}> | IResponse<'404', {
164:   detail?: 'not_modified'
165: }>> => {
166:   const carClass = SITE_CONSTANTS.CAR_CLASSES[car.cc_id]
167:   addToFormData(formData, { data: JSON.stringify({
168:     'b_location_classes=': carClass?.booking_location_classes ??
169:       SITE_CONSTANTS.BOOKING_LOCATION_CLASSES.map(({ id }) => id),
170:   }) })
```

### `src/API/car.ts:161`
```text
155:     ))) response = syncUserResponse
156:   }
157:   return response
158: })
159: 
160: const syncUserWithCar = apiMethod(async(
161:   { formData }: IApiMethodArguments,
162:   car: ICar,
163: ): Promise<IResponse<'200', {}> | IResponse<'404', {
164:   detail?: 'not_modified'
165: }>> => {
166:   const carClass = SITE_CONSTANTS.CAR_CLASSES[car.cc_id]
167:   addToFormData(formData, { data: JSON.stringify({
168:     'b_location_classes=': carClass?.booking_location_classes ??
169:       SITE_CONSTANTS.BOOKING_LOCATION_CLASSES.map(({ id }) => id),
170:   }) })
171:   const { data: response } =
```

### `src/API/car.ts:181`
```text
175:     response.message === 'user or modified data not found'
176:   ) response.data = { detail: 'not_modified' }
177:   return response
178: })
179: 
180: async function _getUserDrivenCar(
181:   { formData }: IApiMethodArguments,
182: ): Promise<ICar> {
183:   const { data } = await axios.post(
184:     `${Config.API_URL}/user/authorized/car/driven`,
185:     formData,
186:   )
187:   return Object.values(data.data.car)[0] as ICar
188: }
189: export const getUserDrivenCar =
190:   apiMethod<typeof _getUserDrivenCar>(_getUserDrivenCar)
```

### `src/API/car.ts:190`
```text
184:     `${Config.API_URL}/user/authorized/car/driven`,
185:     formData,
186:   )
187:   return Object.values(data.data.car)[0] as ICar
188: }
189: export const getUserDrivenCar =
190:   apiMethod<typeof _getUserDrivenCar>(_getUserDrivenCar)
```

### `src/API/user.ts:4`
```text
1: import axios from 'axios'
2: import { IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: 
7: const _getUser = (
8:   { formData }: IApiMethodArguments,
9:   id: IUser['u_id'],
10: ): Promise<IUser | null> => {
11:   return axios.post(`${Config.API_URL}/user/${id}`, formData)
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
```

### `src/API/user.ts:8`
```text
2: import { IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: 
7: const _getUser = (
8:   { formData }: IApiMethodArguments,
9:   id: IUser['u_id'],
10: ): Promise<IUser | null> => {
11:   return axios.post(`${Config.API_URL}/user/${id}`, formData)
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
```

### `src/API/user.ts:15`
```text
9:   id: IUser['u_id'],
10: ): Promise<IUser | null> => {
11:   return axios.post(`${Config.API_URL}/user/${id}`, formData)
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
```

### `src/API/user.ts:18`
```text
12:     .then(res => res.data.data)
13:     .then(res => convertUser(res.user[id]) || null)
14: }
15: export const getUser = apiMethod<typeof _getUser>(_getUser)
16: 
17: const _getUsers = (
18:   { formData }: IApiMethodArguments,
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
```

### `src/API/user.ts:25`
```text
19:   ids: IUser['u_id'][],
20: ): Promise<IUser[]> => {
21:   return axios.post(`${Config.API_URL}/user/${ids.join(',')}`, formData)
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
```

### `src/API/user.ts:28`
```text
22:     .then(res => res.data.data)
23:     .then(res => Object.values(res.user).map(i => convertUser(i)))
24: }
25: export const getUsers = apiMethod<typeof _getUsers>(_getUsers)
26: 
27: const _getAuthorizedUser = (
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
```

### `src/API/user.ts:34`
```text
28:   { formData }: IApiMethodArguments,
29: ): Promise<IUser | null> => {
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
```

### `src/API/user.ts:36`
```text
30:   return axios.post(`${Config.API_URL}/user/authorized`, formData)
31:     .then(res => res.data.data)
32:     .then(res => convertUser(Object.values(res.user)[0] as IUser) || null)
33: }
34: export const getAuthorizedUser = apiMethod<typeof _getAuthorizedUser>(_getAuthorizedUser)
35: 
36: type TEditUser = Partial<Pick<IUser, 'u_id' | 'token' | 'u_hash'>>
37: export type TEditClient = TEditUser & Partial<Pick<IUser,
38:   'u_role' |
39:   'u_name' |
40:   'u_family' |
41:   'u_middle' |
42:   'u_phone' |
43:   'u_email' |
44:   'u_photo' |
45:   'u_lang' |
46:   'u_currency' |
```

### `src/API/user.ts:86`
```text
80:   'out_luggage' |
81:   'ref_code' |
82:   'u_details'
83: >>
84: 
85: const _editUser = (
86:   { formData }: IApiMethodArguments,
87:   data:
88:     TEditClient |
89:     TEditDriverCheckRequired |
90:     TEditDriverCheckActive,
91: ) => {
92:   // @TODO вернуть u_city когда наладим автозаполнение
93:   const { token, u_hash, u_id, u_city, ...userData } = data as (
94:     TEditClient &
95:     TEditDriverCheckRequired &
96:     TEditDriverCheckActive
```

### `src/API/user.ts:93`
```text
87:   data:
88:     TEditClient |
89:     TEditDriverCheckRequired |
90:     TEditDriverCheckActive,
91: ) => {
92:   // @TODO вернуть u_city когда наладим автозаполнение
93:   const { token, u_hash, u_id, u_city, ...userData } = data as (
94:     TEditClient &
95:     TEditDriverCheckRequired &
96:     TEditDriverCheckActive
97:   )
98:   if (token && u_hash && u_id) addToFormData(formData, { token, u_hash, u_id })
99:   addToFormData(formData, {
100:     data: JSON.stringify({
101:       u_city: u_city || undefined,
102:       ...reverseConvertUser(userData),
103:     }),
```

### `src/API/user.ts:98`
```text
92:   // @TODO вернуть u_city когда наладим автозаполнение
93:   const { token, u_hash, u_id, u_city, ...userData } = data as (
94:     TEditClient &
95:     TEditDriverCheckRequired &
96:     TEditDriverCheckActive
97:   )
98:   if (token && u_hash && u_id) addToFormData(formData, { token, u_hash, u_id })
99:   addToFormData(formData, {
100:     data: JSON.stringify({
101:       u_city: u_city || undefined,
102:       ...reverseConvertUser(userData),
103:     }),
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
```

### `src/API/user.ts:109`
```text
103:     }),
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
109: export const editUser = apiMethod<typeof _editUser>(_editUser)
110: export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })
```

### `src/API/user.ts:110`
```text
104:   })
105: 
106:   return axios.post(`${Config.API_URL}/user`, formData)
107:     .then(res => res.data)
108: }
109: export const editUser = apiMethod<typeof _editUser>(_editUser)
110: export const editUserAfterRegister = apiMethod<typeof _editUser>(_editUser, { authRequired: false })
```

### `src/API/auth.ts:2`
```text
1: import axios from 'axios'
2: import { EUserRoles, ITokens, IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: import { ERegistrationType } from '../state/user/constants'
7: 
8: const _register = (
9:   { formData }: IApiMethodArguments,
10:   data: Partial<IUser>,
11: ): Promise<{
12:   u_id: IUser['u_id'],
```

### `src/API/auth.ts:4`
```text
1: import axios from 'axios'
2: import { EUserRoles, ITokens, IUser } from '../types/types'
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: import { ERegistrationType } from '../state/user/constants'
7: 
8: const _register = (
9:   { formData }: IApiMethodArguments,
10:   data: Partial<IUser>,
11: ): Promise<{
12:   u_id: IUser['u_id'],
13:   email_status: boolean,
14:   string: string,
```

### `src/API/auth.ts:9`
```text
3: import { convertUser, reverseConvertUser } from '../tools/convert'
4: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
5: import Config from '../config'
6: import { ERegistrationType } from '../state/user/constants'
7: 
8: const _register = (
9:   { formData }: IApiMethodArguments,
10:   data: Partial<IUser>,
11: ): Promise<{
12:   u_id: IUser['u_id'],
13:   email_status: boolean,
14:   string: string,
15:   error?: string
16: } | null> => {
17:   addToFormData(formData, reverseConvertUser(data))
18:   if (data.u_role === EUserRoles.Driver) addToFormData(formData, { 'st': 1 })
19:   return axios.post(`${Config.API_URL}/register`, formData)
```

### `src/API/auth.ts:37`
```text
31:     })
32: }
33: /**
34:  * @returns email_status - if email is specified
35:  * @returns string - password if email is not specified
36:  */
37: export const register = apiMethod<typeof _register>(_register, { authRequired: false })
38: 
39: const _remindPassword = (
40:   { formData }: IApiMethodArguments,
41:   email: IUser['u_email'],
42: ) => {
43:   addToFormData(formData, {
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
```

### `src/API/auth.ts:40`
```text
34:  * @returns email_status - if email is specified
35:  * @returns string - password if email is not specified
36:  */
37: export const register = apiMethod<typeof _register>(_register, { authRequired: false })
38: 
39: const _remindPassword = (
40:   { formData }: IApiMethodArguments,
41:   email: IUser['u_email'],
42: ) => {
43:   addToFormData(formData, {
44:     u_email: email,
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
```

### `src/API/auth.ts:51`
```text
45:   })
46: 
47:   return axios.post(`${Config.API_URL}/remind`, formData)
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
```

### `src/API/auth.ts:54`
```text
48:     .then(res => res.data)
49:     .then(res => res.status === 'error' ? Promise.reject() : res)
50: }
51: export const remindPassword = apiMethod<typeof _remindPassword>(_remindPassword, { authRequired: false })
52: 
53: const _login = (
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
```

### `src/API/auth.ts:60`
```text
54:   { formData }: IApiMethodArguments,
55:   data: {
56:     login: IUser['u_email'] | IUser['u_phone'],
57:     password?: string | undefined,
58:     type: ERegistrationType
59:   },
60: ): Promise<{ user: IUser | null, tokens: ITokens | null, data: string | null } | null> => {
61:   addToFormData(formData, {
62:     ...data,
63:     au: 'f',
64:   })
65: 
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
```

### `src/API/auth.ts:72`
```text
66:   return axios.post(`${Config.API_URL}/auth`, formData)
67:     .then(res => res.data)
68:     .then(res => {
69:       if (res.message === 'wrong login') {
70:         return {
71:           user: null,
72:           tokens: null,
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
```

### `src/API/auth.ts:79`
```text
73:           data: res.message,
74:         }
75:       }
76:       if (res.data === 'code sent') {
77:         return {
78:           user: null,
79:           tokens: null,
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
```

### `src/API/auth.ts:86`
```text
80:           data: res.data,
81:         }
82:       }
83:       if (res.message === 'wrong phone'){
84:         return {
85:           user: null,
86:           tokens: null,
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
```

### `src/API/auth.ts:93`
```text
87:           data: res.message,
88:         }
89:       }
90:       if (res.message === 'wrong password') {
91:         return {
92:           user: null,
93:           tokens: null,
94:           data: res.message,
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
```

### `src/API/auth.ts:101`
```text
95:         }
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
```

### `src/API/auth.ts:102`
```text
96:       }
97: 
98:       if (!res?.auth_hash) {
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
```

### `src/API/auth.ts:105`
```text
99:         return Promise.reject()
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
```

### `src/API/auth.ts:106`
```text
100:       }
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
```

### `src/API/auth.ts:107`
```text
101:       const tokenFormData = new FormData()
102:       addToFormData(tokenFormData, {
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
```

### `src/API/auth.ts:109`
```text
103:         auth_hash: res?.auth_hash,
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
```

### `src/API/auth.ts:110`
```text
104:       })
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
```

### `src/API/auth.ts:111`
```text
105:       return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
106:         .then(tokenRes => tokenRes)
107:         .then(tokenRes => ({
108:           user: convertUser(res.auth_user),
109:           tokens: {
110:             token: tokenRes.data.data.token,
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
```

### `src/API/auth.ts:117`
```text
111:             u_hash: tokenRes.data.data.u_hash,
112:           },
113:           data: null,
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
```

### `src/API/auth.ts:120`
```text
114:         }))
115:     })
116: }
117: export const login = apiMethod<typeof _login>(_login, { authRequired: false })
118: 
119: const _whatsappSignUp = (
120:   _: IApiMethodArguments,
121:   data: {
122:     login: IUser['u_phone'],
123:     type: ERegistrationType
124:     ref_code?: string | undefined,
125:   },
126: ): Promise< {u_id: IUser['u_id'], string:string} | null> => {
127:   const waData = new FormData()
128:   addToFormData(waData, {
129:     u_phone: data.login,
130:     type: ERegistrationType.Whatsapp,
```

### `src/API/auth.ts:140`
```text
134:     .then(res => res.data)
135:     .then(res => {
136:       if (res.status === 'error') return Promise.reject(res)
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
```

### `src/API/auth.ts:143`
```text
137:       return res.data
138:     })
139: }
140: export const whatsappSignUp = apiMethod<typeof _whatsappSignUp>(_whatsappSignUp, { authRequired: false })
141: 
142: const _googleLogin = (
143:   { formData }: IApiMethodArguments,
144:   auth: {
145:     data: {
146:       u_name: string,
147:       u_phone: string,
148:       u_email: IUser['u_email'],
149:       type: ERegistrationType,
150:       u_role: EUserRoles,
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
```

### `src/API/auth.ts:157`
```text
151:       ref_code: string,
152:       u_details: any,
153:       st: string | undefined,
154:     } | null
155:     auth_hash: string | null
156:   },
157: ): Promise<{ user: IUser, tokens: ITokens } | null> => {
158:   console.log(auth)
159:   if(auth.auth_hash === null) {
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
```

### `src/API/auth.ts:166`
```text
160:     addToFormData(formData, {
161:       ...auth.data,
162:     })
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
```

### `src/API/auth.ts:169`
```text
163:     return axios.post(`${Config.API_URL}/register`, formData)
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
```

### `src/API/auth.ts:170`
```text
164:       .then(res => res.data)
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
```

### `src/API/auth.ts:171`
```text
165:       .then(res => {
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
```

### `src/API/auth.ts:172`
```text
166:         if (!res?.data?.token || !res?.data?.u_hash) {
167:           return Promise.reject()
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
```

### `src/API/auth.ts:174`
```text
168:         }
169:         const tokenFormData = new FormData()
170:         addToFormData(tokenFormData, {
171:           token: res?.data?.token,
172:           u_hash: res?.data?.u_hash,
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
```

### `src/API/auth.ts:179`
```text
173:         })
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
```

### `src/API/auth.ts:180`
```text
174:         return axios.post(`${Config.API_URL}/token/authorized`, tokenFormData,{ headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
```

### `src/API/auth.ts:181`
```text
175:           .then(userRes => userRes.data)
176:           .then(userRes => {
177:             return {
178:               user: convertUser(userRes.auth_user),
179:               tokens: {
180:                 token: userRes.data.token,
181:                 u_hash: userRes.data.u_hash,
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
```

### `src/API/auth.ts:188`
```text
182:               },
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
```

### `src/API/auth.ts:189`
```text
183:             }
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
```

### `src/API/auth.ts:190`
```text
184:           })
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
```

### `src/API/auth.ts:191`
```text
185:       })
186:   }
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
```

### `src/API/auth.ts:193`
```text
187:   else {
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
```

### `src/API/auth.ts:194`
```text
188:     const tokenFormData = 'auth_hash='+encodeURIComponent(auth.auth_hash)
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
```

### `src/API/auth.ts:195`
```text
189:     return axios.post(`${Config.API_URL}/token`, tokenFormData, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } })
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
```

### `src/API/auth.ts:196`
```text
190:       .then(tokenRes => tokenRes.data)
191:       .then(tokenRes => {
192:         return {
193:           user: convertUser(tokenRes.auth_user),
194:           tokens: {
195:             token: tokenRes.data.token,
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
```

### `src/API/auth.ts:202`
```text
196:             u_hash: tokenRes.data.u_hash,
197:           },
198:         }
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:205`
```text
199:       })
200:   }
201: }
202: export const googleLogin = apiMethod<typeof _googleLogin>(_googleLogin, { authRequired: false })
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/auth.ts:209`
```text
203: 
204: const _logout = (
205:   { formData }: IApiMethodArguments,
206: ): Promise<any> => {
207:   return axios.post(`${Config.API_URL}/logout/?`)
208: }
209: export const logout = apiMethod<typeof _logout>(_logout, { authRequired: false })
```

### `src/API/location.ts:3`
```text
1: import axios from 'axios'
2: import { IResponse } from '../types/api'
3: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
4: import Config from '../config'
5: 
6: export const sendPosition = apiMethod(async(
7:   { formData }: IApiMethodArguments,
8:   { latitude, longitude }: {
9:     latitude: number
10:     longitude: number
11:   },
12: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
13:   addToFormData(formData, { latitude, longitude })
```

### `src/API/location.ts:6`
```text
1: import axios from 'axios'
2: import { IResponse } from '../types/api'
3: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
4: import Config from '../config'
5: 
6: export const sendPosition = apiMethod(async(
7:   { formData }: IApiMethodArguments,
8:   { latitude, longitude }: {
9:     latitude: number
10:     longitude: number
11:   },
12: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
13:   addToFormData(formData, { latitude, longitude })
14:   const { data } = await axios.post(`${Config.API_URL}/location`, formData)
15:   return data
16: })
```

### `src/API/location.ts:7`
```text
1: import axios from 'axios'
2: import { IResponse } from '../types/api'
3: import { addToFormData, apiMethod, IApiMethodArguments } from '../tools/api'
4: import Config from '../config'
5: 
6: export const sendPosition = apiMethod(async(
7:   { formData }: IApiMethodArguments,
8:   { latitude, longitude }: {
9:     latitude: number
10:     longitude: number
11:   },
12: ): Promise<IResponse<'200', {}> | IResponse<'404', {}>> => {
13:   addToFormData(formData, { latitude, longitude })
14:   const { data } = await axios.post(`${Config.API_URL}/location`, formData)
15:   return data
16: })
```

### `src/API/index.ts:18`
```text
12:   ISuggestion,
13:   ITrip,
14: } from '../types/types'
15: import { getBase64, getHints } from '../tools/utils'
16: import { convertTrip, reverseConvertTrip } from '../tools/convert'
17: import {
18:   addToFormData, apiMethod, IApiMethodArguments, IResponseFields,
19: } from '../tools/api'
20: import getCountryISO3 from '../tools/countryISO2To3'
21: import SITE_CONSTANTS from '../siteConstants'
22: import Config from '../config'
23: import store from '../state'
24: import { configSelectors } from '../state/config'
25: import { userSelectors } from '../state/user'
26: import { EBookingActions } from './constants'
27: import { getCacheVersion } from './cacheVersion'
28: 
```

### `src/API/index.ts:77`
```text
71: } from './way'
72: export {
73:   getPolygonsIdsOnPoint,
74: } from './polygon'
75: 
76: const _uploadFile = (
77:   { formData }: IApiMethodArguments,
78:   data: any,
79: ): Promise<{
80:   dl_id: string
81: } | null> => {
82:   return getBase64(data.file)
83:     .then(base64 => {
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
```

### `src/API/index.ts:85`
```text
79: ): Promise<{
80:   dl_id: string
81: } | null> => {
82:   return getBase64(data.file)
83:     .then(base64 => {
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
88:         private: 0,
89:       })
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
```

### `src/API/index.ts:86`
```text
80:   dl_id: string
81: } | null> => {
82:   return getBase64(data.file)
83:     .then(base64 => {
84:       addToFormData(formData, {
85:         token: data.token,
86:         u_hash: data.u_hash,
87:         file: JSON.stringify({ base64, u_id: data.u_id }),
88:         private: 0,
89:       })
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
```

### `src/API/index.ts:96`
```text
90:       return formData
91:     })
92:     .then(form => axios.post(`${Config.API_URL}/dropbox/file`, form))
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
```

### `src/API/index.ts:99`
```text
93:     .then(res => ({ ...res, dl_id: res?.data?.data?.dl_id }))
94: }
95: 
96: export const uploadFile = apiMethod<typeof _uploadFile>(_uploadFile, { authRequired: false })
97: 
98: const _checkRefCode = (
99:   { formData }: IApiMethodArguments,
100:   ref_code: string,
101: ): Promise<boolean> => {
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
```

### `src/API/index.ts:108`
```text
102:   return axios.get(`${Config.API_URL}/referral/code/${ref_code}/check`)
103:     .then(res => {
104:       return res.data?.data?.ref_code_free || false
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
```

### `src/API/index.ts:111`
```text
105:     })
106: }
107: 
108: export const checkRefCode = apiMethod<typeof _checkRefCode>(_checkRefCode, { authRequired: false })
109: 
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
```

### `src/API/index.ts:116`
```text
110: const _checkConfig = (
111:   { formData }: IApiMethodArguments,
112:   config: string,
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
122:     t_id: ITrip['t_id'],
123: }> => {
124:   addToFormData(formData, {
125:     data: JSON.stringify(reverseConvertTrip(data)),
126:   })
```

### `src/API/index.ts:119`
```text
113: ): Promise<any> => {
114:   return axios.get(`${Config.API_URL}`, { params: { config } })
115: }
116: export const checkConfig = apiMethod<typeof _checkConfig>(_checkConfig, { authRequired: false })
117: 
118: const _postTrip = (
119:   { formData }: IApiMethodArguments,
120:   data: ITrip,
121: ): Promise<IResponseFields & {
122:     t_id: ITrip['t_id'],
123: }> => {
124:   addToFormData(formData, {
125:     data: JSON.stringify(reverseConvertTrip(data)),
126:   })
127: 
128:   return axios.post(`${Config.API_URL}/trip`, formData)
129:     .then(res => res.data)
```

### `src/API/index.ts:131`
```text
125:     data: JSON.stringify(reverseConvertTrip(data)),
126:   })
127: 
128:   return axios.post(`${Config.API_URL}/trip`, formData)
129:     .then(res => res.data)
130: }
131: export const postTrip = apiMethod<typeof _postTrip>(_postTrip)
132: 
133: const _getTrips = (
134:   { formData }: IApiMethodArguments,
135:   type: EOrderTypes = EOrderTypes.Active,
136: ): Promise<IOrder[]> => {
137:   addToFormData(formData, {
138:     array_type: 'list',
139:   })
140: 
141:   return axios.post(`${Config.API_URL}/trip`, formData)
```

### `src/API/index.ts:134`
```text
128:   return axios.post(`${Config.API_URL}/trip`, formData)
129:     .then(res => res.data)
130: }
131: export const postTrip = apiMethod<typeof _postTrip>(_postTrip)
132: 
133: const _getTrips = (
134:   { formData }: IApiMethodArguments,
135:   type: EOrderTypes = EOrderTypes.Active,
136: ): Promise<IOrder[]> => {
137:   addToFormData(formData, {
138:     array_type: 'list',
139:   })
140: 
141:   return axios.post(`${Config.API_URL}/trip`, formData)
142:     .then(res => res.data)
143:     .then(res =>
144:       res.data.trip || [],
```

### `src/API/index.ts:153`
```text
147:     .then(res =>
148:       res.sort(
149:         (a: IOrder, b: IOrder) => a.b_start_datetime < b.b_start_datetime ? 1 : -1,
150:       ),
151:     )
152: }
153: export const getTrips = apiMethod<typeof _getTrips>(_getTrips)
154: 
155: const _getWashTrips = (
156:   { formData }: IApiMethodArguments,
157:   type: EOrderTypes = EOrderTypes.Active,
158: ): Promise<IOrder[]> => {
159:   addToFormData(formData, {
160:     array_type: 'list',
161:   })
162: 
163:   return axios.post(`${Config.API_URL}/trip/get`, formData)
```

### `src/API/index.ts:156`
```text
150:       ),
151:     )
152: }
153: export const getTrips = apiMethod<typeof _getTrips>(_getTrips)
154: 
155: const _getWashTrips = (
156:   { formData }: IApiMethodArguments,
157:   type: EOrderTypes = EOrderTypes.Active,
158: ): Promise<IOrder[]> => {
159:   addToFormData(formData, {
160:     array_type: 'list',
161:   })
162: 
163:   return axios.post(`${Config.API_URL}/trip/get`, formData)
164:     .then(res => res.data)
165:     .then(res =>
166:       res.data.trip || [],
```

### `src/API/index.ts:175`
```text
169:     .then(res =>
170:       res.sort(
171:         (a: IOrder, b: IOrder) => a.b_start_datetime < b.b_start_datetime ? 1 : -1,
172:       ),
173:     )
174: }
175: export const getWashTrips = apiMethod<typeof _getWashTrips>(_getWashTrips, { authRequired: false })
176: 
177: const _getImageBlob = (
178:   { formData }: IApiMethodArguments,
179:   id: number,
180: ) => {
181:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
182:     responseType: 'blob',
183:   }).then(res => {
184:     return [id, URL.createObjectURL(res.data)]
185:   })
```

### `src/API/index.ts:178`
```text
172:       ),
173:     )
174: }
175: export const getWashTrips = apiMethod<typeof _getWashTrips>(_getWashTrips, { authRequired: false })
176: 
177: const _getImageBlob = (
178:   { formData }: IApiMethodArguments,
179:   id: number,
180: ) => {
181:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
182:     responseType: 'blob',
183:   }).then(res => {
184:     return [id, URL.createObjectURL(res.data)]
185:   })
186: }
187: export const getImageBlob = apiMethod<typeof _getImageBlob>(_getImageBlob)
188: 
```

### `src/API/index.ts:187`
```text
181:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
182:     responseType: 'blob',
183:   }).then(res => {
184:     return [id, URL.createObjectURL(res.data)]
185:   })
186: }
187: export const getImageBlob = apiMethod<typeof _getImageBlob>(_getImageBlob)
188: 
189: const _getImageFile = (
190:   { formData }: IApiMethodArguments,
191:   id: number,
192: ): Promise<[number, File]> => {
193:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
194:     responseType: 'blob',
195:   }).then(res => {
196:     return [id, new File([res.data], String(id))]
197:   })
```

### `src/API/index.ts:190`
```text
184:     return [id, URL.createObjectURL(res.data)]
185:   })
186: }
187: export const getImageBlob = apiMethod<typeof _getImageBlob>(_getImageBlob)
188: 
189: const _getImageFile = (
190:   { formData }: IApiMethodArguments,
191:   id: number,
192: ): Promise<[number, File]> => {
193:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
194:     responseType: 'blob',
195:   }).then(res => {
196:     return [id, new File([res.data], String(id))]
197:   })
198: }
199: export const getImageFile = apiMethod<typeof _getImageFile>(_getImageFile)
200: 
```

### `src/API/index.ts:199`
```text
193:   return axios.post(`${Config.API_URL}/dropbox/file/${id}`, formData, {
194:     responseType: 'blob',
195:   }).then(res => {
196:     return [id, new File([res.data], String(id))]
197:   })
198: }
199: export const getImageFile = apiMethod<typeof _getImageFile>(_getImageFile)
200: 
201: const _setOutDrive = (
202:   { formData }: IApiMethodArguments,
203:   isFinished: boolean,
204:   addresses?: {
205:     fromAddress?: string,
206:     fromLatitude?: string,
207:     fromLongitude?: string,
208:     toAddress?: string,
209:     toLatitude?: string,
```

### `src/API/index.ts:202`
```text
196:     return [id, new File([res.data], String(id))]
197:   })
198: }
199: export const getImageFile = apiMethod<typeof _getImageFile>(_getImageFile)
200: 
201: const _setOutDrive = (
202:   { formData }: IApiMethodArguments,
203:   isFinished: boolean,
204:   addresses?: {
205:     fromAddress?: string,
206:     fromLatitude?: string,
207:     fromLongitude?: string,
208:     toAddress?: string,
209:     toLatitude?: string,
210:     toLongitude?: string,
211:   },
212:   passengers?: IOrder['b_passengers_count'],
```

### `src/API/index.ts:236`
```text
230:     ),
231:   })
232: 
233:   return axios.post(`${Config.API_URL}/user`, formData)
234:     .then(res => res.data)
235: }
236: export const setOutDrive = apiMethod<typeof _setOutDrive>(_setOutDrive)
237: 
238: export const reverseGeocode = (
239:   lat: ValueOf<Stringify<IBookingCoordinatesLatitude>>,
240:   lng: ValueOf<Stringify<IBookingCoordinatesLongitude>>,
241:   { details = true }: {
242:     details?: boolean
243:   } = {},
244: ): Promise<IPlaceResponse> => {
245:   const language = configSelectors.language(store.getState())
246: 
```

### `src/API/index.ts:288`
```text
282:     .then(res =>
283:       res.data[0] &&
284:             ({ ...res.data[0], lat: parseFloat(res.data[0].lat), lon: parseFloat(res.data[0].lon) }),
285:     )
286: }
287: 
288: const orsToken = '<REDACTED>'
289: 
290: export const makeRoutePoints = (from: IAddressPoint, to: IAddressPoint): Promise<IRouteInfo> => {
291:   const convertPoint = (point: IAddressPoint) => `${point.longitude},${point.latitude}`
292: 
293:   return axios.get(
294:     'https://api.openrouteservice.org/v2/directions/driving-car',
295:     {
296:       params: {
297:         api_key: orsToken,
298:         start: convertPoint(from),
```

> `[REDACTED DURING PUBLIC REPOSITORY SANITIZATION]`
>
> В строке 288 исходного файла `src/API/index.ts` находился реальный
> OpenRouteService API key. Значение заменено на `<REDACTED>`; имя файла,
> номера строк и остальной verbatim-контекст цитаты сохранены без изменений.
> Credential принадлежит владельцу исходной системы; ротация/отзыв — на его стороне.

### `src/API/index.ts:297`
```text
291:   const convertPoint = (point: IAddressPoint) => `${point.longitude},${point.latitude}`
292: 
293:   return axios.get(
294:     'https://api.openrouteservice.org/v2/directions/driving-car',
295:     {
296:       params: {
297:         api_key: orsToken,
298:         start: convertPoint(from),
299:         end: convertPoint(to),
300:       },
301:     },
302:   )
303:     .then(res => res.data)
304:     .then(res => {
305:       const hours = Math.floor(res.features[0].properties.summary.duration / (60 * 60))
306:       const minutes = Math.round((res.features[0].properties.summary.duration - (hours * 60 * 60)) / 60)
307:       return {
```

### `src/localization/common.ts:450`
```text
444:   TIE_CARD_HEADER: 'tie_card_header',
445:   TIMER: 'timer',
446:   TIME_FROM: 'time_from',
447:   TIME_TILL: 'time_till',
448:   TO: 'to',
449:   TODAY: 'today',
450:   TOKEN: 'token',
451:   TOMORROW: 'tomorrow',
452:   TOOK_PASSENGER: 'took_passenger',
453:   TOOL_BOX: 'tool_box',
454:   TOP_P: 'top_p',
455:   TOY_STORAGE: 'toy_storage',
456:   TO_ORDER: 'to_order',
457:   TRAMPOLINE: 'trampoline',
458:   TRANSFER_DATE: 'transfer_date',
459:   TRANSPORTATION_DATE: 'transportation_date',
460:   TRANSPORT_P: 'transport_p',
```

### `src/localization/common.ts:482`
```text
476:   UMBRELLA: 'umbrella',
477:   UP_TO_100: 'up_to_100',
478:   URBAN: 'urban',
479:   URBAN_P: 'urban_p',
480:   URGENTLY: 'urgently',
481:   USER_IS_NOT_PERFORMER_ERROR: 'user_is_not_performer_error',
482:   U_HASH: 'u_hash',
483:   UNAUTHORIZED_ACCESS: 'unauthorized_access',
484:   VERY_EXPENSIVE: 'very_expensive',
485:   VIEW_P: 'view_p',
486:   VOTE: 'vote',
487:   VOTER: 'voter',
488:   VOTING: 'voting',
489:   WAGON: 'wagon',
490:   WAITING: 'waiting',
491:   WAITING_FOR_LONG: 'waiting_for_long',
492:   WARDROBE: 'wardrobe',
```

### `src/tools/api.ts:5`
```text
1: import { userSelectors } from '../state/user'
2: import { ParametersExceptFirst } from '../types'
3: import state from '../state'
4: 
5: export interface IApiMethodArguments {
6:   token: string,
7:   uHash: string,
8:   formData: FormData
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
```

### `src/tools/api.ts:6`
```text
1: import { userSelectors } from '../state/user'
2: import { ParametersExceptFirst } from '../types'
3: import state from '../state'
4: 
5: export interface IApiMethodArguments {
6:   token: string,
7:   uHash: string,
8:   formData: FormData
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
```

### `src/tools/api.ts:11`
```text
5: export interface IApiMethodArguments {
6:   token: string,
7:   uHash: string,
8:   formData: FormData
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
```

### `src/tools/api.ts:15`
```text
9: }
10: 
11: interface apiMethodOptions {
12:   authRequired?: boolean
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
```

### `src/tools/api.ts:19`
```text
13: }
14: 
15: export const apiMethod = <T extends (...args: any[]) => any>(
16:   method: T,
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
```

### `src/tools/api.ts:23`
```text
17:   {
18:     authRequired = true,
19:   }: apiMethodOptions = {},
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
```

### `src/tools/api.ts:26`
```text
20: ) => {
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
```

### `src/tools/api.ts:27`
```text
21:   return (...args: ParametersExceptFirst<T>): ReturnType<T> => {
22:     const formData = new FormData()
23:     let tokens
24: 
25:     if (authRequired) {
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
```

### `src/tools/api.ts:32`
```text
26:       tokens = userSelectors.tokens(state.getState())
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
```

### `src/tools/api.ts:33`
```text
27:       if (!tokens) {
28:         console.error('Auth failed for API call')
29:         return Promise.reject(new Error('Unauthorized user')) as ReturnType<T>
30:       }
31: 
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
```

### `src/tools/api.ts:38`
```text
32:       formData.append('token', tokens.token)
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
```

### `src/tools/api.ts:39`
```text
33:       formData.append('u_hash', tokens.u_hash)
34:     }
35: 
36:     const parameters = [
37:       {
38:         token: tokens?.token,
39:         u_hash: tokens?.u_hash,
40:         formData,
41:       }, ...args,
42:     ] as Parameters<T>
43:     return method(...parameters)
44:   }
45: }
46: 
47: export interface IResponseFields {
48:   /** Список валидных ключей */
49:   affected_fields: string[],
```

### `src/tools/hooks.ts:13`
```text
7: import {
8:   useWatch,
9:   DeepPartialSkipArrayKey, Control, FieldValues,
10: } from 'react-hook-form'
11: import _ from 'lodash'
12: import { IRootState, IDispatch } from '../state'
13: import { getItem, setItem } from './localStorage'
14: 
15: interface IAdditionalDataFlags {
16:   dirty?: boolean,
17:   previousValue?: boolean
18: }
19: interface IAdditionalData {
20:   dirty?: boolean,
21:   previousValue?: any
22: }
23: /**
```

### `src/tools/hooks.ts:24`
```text
18: }
19: interface IAdditionalData {
20:   dirty?: boolean,
21:   previousValue?: any
22: }
23: /**
24:  * Works like useState, but also caches value at localStorage
25:  * @param key localStorage access key. It should follow the format ${objectKey}.${valueKey} or just ${valueKey}
26:  * @param defaultValue default value will be used if cached value is not found or some error occured
27:  * @param allowableValues list of allowable values for cached value.
28:  * @param additionalData object containing data about additional field data passed to return. Do not change at runtime!
29:  *  If allowableValues does not includes cached value, defaultValue is used
30:  */
31: export const useCachedState = <T>(
32:   key: string,
33:   defaultValue?: T,
34:   allowableValues?: T[],
```

### `src/tools/hooks.ts:25`
```text
19: interface IAdditionalData {
20:   dirty?: boolean,
21:   previousValue?: any
22: }
23: /**
24:  * Works like useState, but also caches value at localStorage
25:  * @param key localStorage access key. It should follow the format ${objectKey}.${valueKey} or just ${valueKey}
26:  * @param defaultValue default value will be used if cached value is not found or some error occured
27:  * @param allowableValues list of allowable values for cached value.
28:  * @param additionalData object containing data about additional field data passed to return. Do not change at runtime!
29:  *  If allowableValues does not includes cached value, defaultValue is used
30:  */
31: export const useCachedState = <T>(
32:   key: string,
33:   defaultValue?: T,
34:   allowableValues?: T[],
35:   additionalData: IAdditionalDataFlags = {},
```

### `src/tools/localStorage.ts:13`
```text
7:     defaultValue === undefined ?
8:       allowableValues && allowableValues[0] :
9:       defaultValue
10:   ) as T
11:   let value
12:   try {
13:     value = localStorage.getItem(key)
14:     value = value !== null ? JSON.parse(value) : defaultValue
15:   } catch (error) {
16:     console.error(`Error occured at getItem(${key})`, error)
17:     value = defaultValue
18:   }
19:   return allowableValues ?
20:     allowableValues.includes(value) ?
21:       value :
22:       defaultValue :
23:     value ?? defaultValue
```

### `src/tools/localStorage.ts:28`
```text
22:       defaultValue :
23:     value ?? defaultValue
24: }
25: 
26: export function setItem<T>(key: string, value: T) {
27:   try {
28:     localStorage.setItem(key, JSON.stringify(value))
29:   } catch (error) {
30:     console.error(`Error occured at setItem(${key}, ${value})`, error)
31:   }
32: }
33: 
34: export function removeItem(key: string) {
35:   try {
36:     localStorage.removeItem(key)
37:   } catch (error) {
38:     console.error(`Error occured at removeItem(${key})`, error)
```

### `src/tools/localStorage.ts:36`
```text
30:     console.error(`Error occured at setItem(${key}, ${value})`, error)
31:   }
32: }
33: 
34: export function removeItem(key: string) {
35:   try {
36:     localStorage.removeItem(key)
37:   } catch (error) {
38:     console.error(`Error occured at removeItem(${key})`, error)
39:   }
40: }
```

### `src/tools/utils.ts:611`
```text
605:   // }
606:   return 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>'
607: }
608: 
609: export function addHiddenOrder(orderID?: IOrder['b_id'], userID?: IUser['u_id']) {
610:   if (!orderID || !userID) return
611:   let hiddenOrders = JSON.parse(localStorage.getItem('hiddenOrders') || '{}')
612:   if (hiddenOrders[userID]) {
613:     hiddenOrders[userID] = [...hiddenOrders[userID], orderID]
614:   } else {
615:     hiddenOrders[userID] = [orderID]
616:   }
617:   localStorage.setItem('hiddenOrders', JSON.stringify(hiddenOrders))
618: }
619: 
620: export const getCurrentPosition = (
621:   options?: Parameters<Geolocation['getCurrentPosition']>[2],
```

### `src/tools/utils.ts:617`
```text
611:   let hiddenOrders = JSON.parse(localStorage.getItem('hiddenOrders') || '{}')
612:   if (hiddenOrders[userID]) {
613:     hiddenOrders[userID] = [...hiddenOrders[userID], orderID]
614:   } else {
615:     hiddenOrders[userID] = [orderID]
616:   }
617:   localStorage.setItem('hiddenOrders', JSON.stringify(hiddenOrders))
618: }
619: 
620: export const getCurrentPosition = (
621:   options?: Parameters<Geolocation['getCurrentPosition']>[2],
622: ) =>
623:   new Promise<GeolocationPosition>((res, rej) => {
624:     navigator.geolocation.getCurrentPosition(res, rej, options)
625:   })
626: 
627: export const getOrderIcon = (order: IOrder) => {
```

### `src/pages/Sandbox/index.tsx:29`
```text
23: 
24:   useEffect(() => {
25:     const onChange = _.debounce(() => {
26:       try {
27:         if (editor.current) {
28:           setSettings(editor.current.get())
29:           localStorage.setItem('settings', JSON.stringify(editor.current.get()))
30:         }
31:         updateFrameKey()
32:       } catch (error) {
33:         console.error(error)
34:       }
35:     }, 700)
36:     editor.current = new JSONEditor(
37:       document.getElementById('settings-editor') as HTMLElement,
38:       {
39:         mode: 'code',
```

### `src/pages/Sandbox/index.tsx:44`
```text
38:       {
39:         mode: 'code',
40:         onChange,
41:       },
42:       settings,
43:     )
44:     const defaultSettings = JSON.parse(localStorage.getItem('settings') || JSON.stringify(settings))
45:     editor.current.set(defaultSettings)
46:     setSettings(defaultSettings)
47:   }, [])
48: 
49:   const openFile = (e: React.ChangeEvent<HTMLInputElement>) => {
50:     const file = e.target.files?.length && e.target.files[0]
51:     if (file) {
52:       const reader = new window.FileReader()
53:       reader.readAsText(file, 'UTF-8')
54:       reader.onload = (event) => {
```

### `src/components/Header/index.tsx:11`
```text
5: import moment from 'moment'
6: import { EBookingDriverState, EUserRoles, ILanguage } from '../../types/types'
7: import config from '../../config'
8: import images from '../../constants/images'
9: import SITE_CONSTANTS from '../../siteConstants'
10: import { useInterval } from '../../tools/hooks'
11: import { setCookie } from '../../utils/cookies'
12: import { IRootState } from '../../state'
13: import { configSelectors, configActionCreators } from '../../state/config'
14: import { modalsActionCreators } from '../../state/modals'
15: import { clientOrderSelectors } from '../../state/clientOrder'
16: import { ordersSelectors } from '../../state/orders'
17: import { userSelectors } from '../../state/user'
18: import { t, TRANSLATION } from '../../localization'
19: import { Burger } from '../Burger/Burger'
20: import './styles.scss'
21: 
```

### `src/components/Header/index.tsx:126`
```text
120:   }
121: 
122:   const languages = SITE_CONSTANTS.LANGUAGES
123:     .filter(x => x.iso !== (config.SavedConfig !== 'children' ? ' ' : 'ru'))
124: 
125:   const handleLanguageChange = (lang: ILanguage) => {
126:     setCookie('user_lang', lang.iso)
127:     setLanguage(lang)
128:     setLanguagesOpened(false)
129:   }
130: 
131:   return (
132:     <header className={cn('header', className)}>
133:       <div className="burger-wrapper">
134:         <div className="column">
135:           {
136:             detailedOrderID ?
```

### `src/components/version-info/index.tsx:8`
```text
2: import version from '../../version.json'
3: import './version-info.scss'
4: import Config from '../../config'
5: import { connect } from 'react-redux'
6: import { configActionCreators } from '../../state/config'
7: import SITE_CONSTANTS, {getApiConstants} from '../../siteConstants'
8: import { setCookie } from '../../utils/cookies'
9: import { ILanguage } from '../../types/types'
10: 
11: interface Language {
12:   native: string;
13:   ru: string;
14:   en: string;
15:   es: string | null;
16:   iso: string;
17:   logo: string;
18:   tr_code: string;
```

## 4. Core Backend authentication evidence

Найдено backend credential/auth-gate contexts: **252**.

### `archive_17012026_1259/taxi/index.php:6`
```php
1: <?php
2: 	header("Content-Type: text/html; charset=utf-8");
3: 	require_once('config/config.php');
4: 	if (ROOT_URL != '/') ini_set('session.cookie_path', ROOT_URL);
5: 	if (UID_SUFFIX != '') session_name(session_name() . UID_SUFFIX);
6: 	require_once('models/token.php');
7: 	if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
8: 	{
9: 		if (!isset($_COOKIE['vfoc']) || $_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))))){session_unset();}
10: 	}
11: 	if (isset($_POST['u_a_id']) || isset($_POST['u_a_email']) || isset($_POST['u_a_phone']) || isset($_POST['u_a_tg']) || isset($_POST['u_a_wa']))
12: 	{
13: 		require_once('models/m_functions.php');
14: 		check_auth_user();
15: 		if (empty($_SESSION[UID]))
16: 		{
17: 			show_error('unauthorized access');
18: 		}
19: 		elseif ($_SESSION['id_role'] != 4)
20: 		{
21: 			show_error('not enough rights');
22: 		}
```

### `archive_17012026_1259/taxi/index.php:7`
```php
1: <?php
2: 	header("Content-Type: text/html; charset=utf-8");
3: 	require_once('config/config.php');
4: 	if (ROOT_URL != '/') ini_set('session.cookie_path', ROOT_URL);
5: 	if (UID_SUFFIX != '') session_name(session_name() . UID_SUFFIX);
6: 	require_once('models/token.php');
7: 	if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
8: 	{
9: 		if (!isset($_COOKIE['vfoc']) || $_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))))){session_unset();}
10: 	}
11: 	if (isset($_POST['u_a_id']) || isset($_POST['u_a_email']) || isset($_POST['u_a_phone']) || isset($_POST['u_a_tg']) || isset($_POST['u_a_wa']))
12: 	{
13: 		require_once('models/m_functions.php');
14: 		check_auth_user();
15: 		if (empty($_SESSION[UID]))
16: 		{
17: 			show_error('unauthorized access');
18: 		}
19: 		elseif ($_SESSION['id_role'] != 4)
20: 		{
21: 			show_error('not enough rights');
22: 		}
23: 		else
```

### `archive_17012026_1259/taxi/index.php:14`
```php
6: 	require_once('models/token.php');
7: 	if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
8: 	{
9: 		if (!isset($_COOKIE['vfoc']) || $_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))))){session_unset();}
10: 	}
11: 	if (isset($_POST['u_a_id']) || isset($_POST['u_a_email']) || isset($_POST['u_a_phone']) || isset($_POST['u_a_tg']) || isset($_POST['u_a_wa']))
12: 	{
13: 		require_once('models/m_functions.php');
14: 		check_auth_user();
15: 		if (empty($_SESSION[UID]))
16: 		{
17: 			show_error('unauthorized access');
18: 		}
19: 		elseif ($_SESSION['id_role'] != 4)
20: 		{
21: 			show_error('not enough rights');
22: 		}
23: 		else
24: 		{
25: 			$AUID = get_id_user(isset($_POST['u_a_id'])?trim($_POST['u_a_id']):"",isset($_POST['u_a_email'])?trim($_POST['u_a_email']):"",isset($_POST['u_a_phone'])?trim($_POST['u_a_phone']):"",isset($_POST['u_a_tg'])?trim($_POST['u_a_tg']):"",isset($_POST['u_a_wa'])?trim($_POST['u_a_wa']):"");
26: 			if (is_array($AUID))
27: 			{
28: 				show_error($AUID);
29: 			}
30: 			else
```

### `archive_17012026_1259/taxi/models/api.php:15`
```php
7: 				'code' 		=> $code,
8: 				'status' 	=> $status,
9: 				'message' 	=> $message,
10: 				'data' 		=> array()
11: 			);
12: 		}
13: 
14: 
15: 		public function registerUser($role = 0, $phone = "", $email = "", $tg = "", $wa = "", $name = "", $referral_code = "", $reco = "", $ip = "", $data = "", $show_token = false, $roles = array())
16: 		{
17: 			$sql_user = "NULL";
18: 			$referrer = "";
19: 			if (empty($tg) && empty($wa))
20: 			{
21: 				if (empty($_SESSION[UID])) 
22: 				{
23: 					if ($role != 1 && $role != 2 && $role != 5)
24: 					{
25: 						return $this->showError('404', 'error', 'wrong user role');
26: 					}
27: 				}
28: 				else
29: 				{
30: 					if ($this->id_role != 4) return $this->showError('404', 'error', 'user is already authorized');
31: 					if (empty($roles[$role]))	return $this->showError('404', 'error', 'role not found');
```

### `archive_17012026_1259/taxi/models/api.php:316`
```php
308: 				FROM `ip_referral`
309: 				WHERE 
310: 					`ip` = INET_ATON('" . $ip . "')
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
```

### `archive_17012026_1259/taxi/models/api.php:318`
```php
310: 					`ip` = INET_ATON('" . $ip . "')
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
```

### `archive_17012026_1259/taxi/models/api.php:319`
```php
311: 				";
312: 			query($s);
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
```

### `archive_17012026_1259/taxi/models/api.php:321`
```php
313: 			$q = query($s);
314: 			if ($q === false) $out['warning'][] = 'ip database delete failed';
315: 
316: 			if ($show_token === true)
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
```

### `archive_17012026_1259/taxi/models/api.php:325`
```php
317: 			{
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
```

### `archive_17012026_1259/taxi/models/api.php:326`
```php
318: 				$token[0] = get_token($out['u_id']);
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
```

### `archive_17012026_1259/taxi/models/api.php:327`
```php
319: 				if (gettype($token[0]) !== 'string')
320: 				{
321: 					return $this->showError('404', 'error', 'token error: ' . $token[0]);
322: 				}
323: 				else
324: 				{
325: 					$token[1] = get_u_hash($token[0],$out['u_id']);
326: 					$out['token'] = $token[0];
327: 					$out['u_hash'] = $token[1];
328: 				};
329: 			}
330: 
331: 			return array(
332: 				'code' 		=>	'200',
333: 				'status' 	=>	'success',
334: 				'data' 		=>	$out
335: 			);
336: 		}
337: 
338: 		public function authUser($login = '', $password = '', $type = 'e-mail')
339: 		{
340: 			if (!empty($_SESSION[UID])) {
341: 				return $this->showError('404', 'error', 'user is already authorized');
342: 			}
343: 
```

### `archive_17012026_1259/taxi/models/api.php:8341`
```php
8333: 			return array(
8334: 				'code' 		=>	'200',
8335: 				'status' 	=>	'success'
8336: 			);
8337: 		}
8338: 
8339: 		public $associativeArray = true;
8340: 
8341: 		public function selectToken($id_user = "", $token = array())
8342: 		{
8343: 			if (empty($_SESSION[UID])) {
8344: 				if (!empty($_POST['auth_hash']))
8345: 				{
8346: 					list($_COOKIE[session_name()],$_COOKIE['vfoc']) = array_merge(explode('|', openssl____decrypt($_POST['auth_hash'])),array(''));
8347: 					if (!empty($_COOKIE[session_name()])) session_start();
8348: 					if (!empty($_SESSION[UID]))
8349: 					{
8350: 						if (empty($_COOKIE['vfoc']) 
8351: 							||$_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID])))) 
8352: 							|| empty($_SESSION['auth_time']) 
8353: 							|| time() > $_SESSION['auth_time'] + $this->constant['token_interval_for_auth_hash'])
8354: 						{
8355: 
8356: 							session_unset();
8357: 
```

### `archive_17012026_1259/taxi/models/api.php:8353`
```php
8345: 				{
8346: 					list($_COOKIE[session_name()],$_COOKIE['vfoc']) = array_merge(explode('|', openssl____decrypt($_POST['auth_hash'])),array(''));
8347: 					if (!empty($_COOKIE[session_name()])) session_start();
8348: 					if (!empty($_SESSION[UID]))
8349: 					{
8350: 						if (empty($_COOKIE['vfoc']) 
8351: 							||$_COOKIE['vfoc'] != md5(session_id() . strtolower(dechex(crc32($_SESSION[UID])))) 
8352: 							|| empty($_SESSION['auth_time']) 
8353: 							|| time() > $_SESSION['auth_time'] + $this->constant['token_interval_for_auth_hash'])
8354: 						{
8355: 
8356: 							session_unset();
8357: 
8358: 						}
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
```

### `archive_17012026_1259/taxi/models/api.php:8367`
```php
8359: 					}
8360: 				}
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
```

### `archive_17012026_1259/taxi/models/api.php:8369`
```php
8361: 
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
```

### `archive_17012026_1259/taxi/models/api.php:8370`
```php
8362: 				if (empty($_SESSION[UID])) return $this->showError('404', 'error', 'unauthorized access');
8363: 			}
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
```

### `archive_17012026_1259/taxi/models/api.php:8372`
```php
8364: 
8365: 			if (empty($id_user) || $id_user == 'authorized')
8366: 			{
8367: 				if (empty($token))
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
```

### `archive_17012026_1259/taxi/models/api.php:8376`
```php
8368: 				{
8369: 					$token[0] = get_token($_SESSION[UID]);
8370: 					if (gettype($token[0]) !== 'string')
8371: 					{
8372: 						return $this->showError('404', 'error', 'token error: ' . $token[0]);
8373: 					}
8374: 					else
8375: 					{
8376: 						$token[1] = get_u_hash($token[0],$_SESSION[UID]);
8377: 					}
8378: 				}
8379: 			}
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
```

### `archive_17012026_1259/taxi/models/api.php:8388`
```php
8380: 			else
8381: 			{
8382: 				return $this->showError('404', 'error', 'not enough rights');
8383: 			}
8384: 			
8385: 			return array(
8386: 				'code' 		=>	'200',
8387: 				'status' 	=>	'success',		
8388: 				'data' 		=>	array('token' => $token[0], 'u_hash' => $token[1]),
8389: 				'auth_user' =>	array(
8390: 									'u_id' => $_SESSION[UID],
8391: 									'u_name' => $_SESSION['name'],
8392: 									'u_family' => $_SESSION['family'],
8393: 									'u_middle' => $_SESSION['middle'],
8394: 									'u_email' => $_SESSION['email'],
8395: 									'u_phone' => $_SESSION['phone'],
8396: 									'u_role' => $_SESSION['id_role'],
8397: 									'u_a_role' => $this->id_role,
8398: 									'u_check_state' => $_SESSION['id_verification_status'],
8399: 									'u_ban' => $_SESSION['user_ban_status'],
8400: 									'u_active' => $_SESSION['active'],
8401: 									'u_photo' => $_SESSION['photo_link'],
8402: 									'u_birthday' => $_SESSION['birthday_date'],
8403: 									'u_lang' => $_SESSION['id_lang'],
8404: 									'u_currency' => $_SESSION['currency'],
```

### `archive_17012026_1259/taxi/models/api.php:8425`
```php
8417: 														'default'	=>	180,
8418: 														'type'		=>	'unsigned integer'
8419: 													),
8420: 			'waiting_interval_add'					=>	array(
8421: 														'default'	=>	180,
8422: 														'type'		=>	'non-zero unsigned integer'
8423: 													),
8424: 
8425: 			'token_interval_for_auth_hash'			=>	array(
8426: 														'default'	=>	10,
8427: 														'type'		=>	'unsigned integer'
8428: 													),
8429: 
8430: 			'limit_row_count'						=>	array(
8431: 														'default'	=>	5,
8432: 														'type'		=>	'non-zero unsigned integer'
8433: 													),
8434: 			'limit_row_count_max'					=>	array(
8435: 														'default'	=>	30,
8436: 														'type'		=>	'non-zero unsigned integer'
8437: 													),
8438: 
8439: 			'average_speed'							=>	array(
8440: 														'default'	=>	60,
8441: 														'type'		=>	'non-zero unsigned integer'
```

### `archive_17012026_1259/taxi/models/api.php:8670`
```php
8662: 			'email_date_format'			=>	array(
8663: 														'default'	=>	'Y-m-d H:i:s',
8664: 														'type'		=>	'date format'
8665: 													),
8666: 			'order_payment'				=>	array(
8667: 														'default'	=>	0,
8668: 														'type'		=>	'unsigned integer'
8669: 													),								
8670: 			'session_token_duration'	=>	array(
8671: 														'default'	=>	86400,
8672: 														'type'		=>	'unsigned integer'
8673: 													),
8674: 			'placing_order_duration'	=>	array(
8675: 														'default'	=>	300,
8676: 														'type'		=>	'unsigned integer'
8677: 													),
8678: 			'free_client_waiting_interval'		=>	array(
8679: 														'default'	=>	300,
8680: 														'type'		=>	'unsigned integer'
8681: 													),
8682: 			'select_trip_with_import'			=>	array(
8683: 														'default'	=>	false,
8684: 														'type'		=>	'boolean'
8685: 													),
8686: 			'sms_code_ip_attempts_count'		=>	array(
```

### `archive_17012026_1259/taxi/models/api.php:13693`
```php
13685: 			return array(
13686: 				'code' 		=>	'200',
13687: 				'status' 	=>	'success'
13688: 			);
13689: 		}
13690: 
13691: 		public function addPush($id_user = "", $id_push = "")
13692: 		{
13693: 			if (empty($_POST['token']) || empty($_POST['u_hash'])) return $this->showError('404', 'error', 'unauthorized access');
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
```

### `archive_17012026_1259/taxi/models/api.php:13694`
```php
13686: 				'code' 		=>	'200',
13687: 				'status' 	=>	'success'
13688: 			);
13689: 		}
13690: 
13691: 		public function addPush($id_user = "", $id_push = "")
13692: 		{
13693: 			if (empty($_POST['token']) || empty($_POST['u_hash'])) return $this->showError('404', 'error', 'unauthorized access');
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
13710: 		}
```

### `archive_17012026_1259/taxi/models/api.php:13702`
```php
13694: 			if ($_POST['token'] == 'test token' && $_POST['u_hash'] == 'test u_hash'){
13695: 				if (empty($id_user) || empty($id_push))
13696: 				{
13697: 					return $this->showError('404', 'error', 'empty u_id or p_id');
13698: 				}
13699: 			}
13700: 			else
13701: 			{
13702: 				return $this->showError('404', 'error', 'wrong token or u_hash');
13703: 			}
13704: 			return array(
13705: 				'code' 		=>	'200',
13706: 				'status' 	=>	'success',
13707: 				'u_id'		=>	$id_user,
13708: 				'p_id'		=>	$id_push
13709: 			);
13710: 		}
13711: 
13712: 		public function createDropboxFile($file = '')
13713: 		{
13714: 			if (empty($_SESSION[UID])) {
13715: 				return $this->showError('404', 'error', 'unauthorized access');
13716: 			}
13717: 
13718: 			@$file = json_decode($file,true);
```

### `archive_17012026_1259/taxi/models/api.php:20857`
```php
20849: 			}
20850: 			if (!empty($id_contact_item))
20851: 			{
20852: 				$sql_where[] = "`contact_items`.`id_contact_item` in ($id_contact_item)";
20853: 			}
20854: 			$sql_private_o_type_all = $sql_private_o_type_1 = $sql_private_o_type_other = ",`contact_items`.`contact_number` as 'number',
20855: 						`contact_items`.`key1`,
20856: 						`contact_items`.`key2`,
20857: 						`contact_items`.`token`,
20858: 						`contact_items`.`hash`,
20859: 						`contact_items`.`secret`,
20860: 						`contact_items`.`host`,
20861: 						`contact_items`.`port`,
20862: 						`contact_items`.`login`,
20863: 						`contact_items`.`password`,
20864: 						`contact_items`.`smtpsecure`,
20865: 						`contact_items`.`fromname`";
20866: 			if ($this->id_role != 4) $sql_private_o_type_other = ",NULL as 'number',
20867: 						NULL as 'key1',
20868: 						NULL as 'key2',
20869: 						NULL as 'token',
20870: 						NULL as 'hash',
20871: 						NULL as 'secret',
20872: 						NULL as 'host',
20873: 						NULL as 'port',
```

### `archive_17012026_1259/taxi/models/api.php:20869`
```php
20861: 						`contact_items`.`port`,
20862: 						`contact_items`.`login`,
20863: 						`contact_items`.`password`,
20864: 						`contact_items`.`smtpsecure`,
20865: 						`contact_items`.`fromname`";
20866: 			if ($this->id_role != 4) $sql_private_o_type_other = ",NULL as 'number',
20867: 						NULL as 'key1',
20868: 						NULL as 'key2',
20869: 						NULL as 'token',
20870: 						NULL as 'hash',
20871: 						NULL as 'secret',
20872: 						NULL as 'host',
20873: 						NULL as 'port',
20874: 						NULL as 'login',
20875: 						NULL as 'password',
20876: 						NULL as 'smtpsecure',
20877: 						NULL as 'fromname'";
20878: 			if ($id_user === NULL)
20879: 			{
20880: 				if ($this->id_role != 4) $id_user = $_SESSION[UID];
20881: 			}
20882: 			else
20883: 			{
20884: 				if ($this->id_role != 4 && $id_user != $_SESSION[UID]) return $this->showError('404', 'error', 'not enough rights');
20885: 			}
```

### `archive_17012026_1259/taxi/models/api.php:21163`
```php
21155: 				'key1'	=>		array(
21156: 										'name'	=>	'key1',
21157: 										'NULL'	=>	true
21158: 									),
21159: 				'key2'	=>		array(
21160: 										'name'	=>	'key2',
21161: 										'NULL'	=>	true
21162: 									),
21163: 				'token'	=>		array(
21164: 										'name'	=>	'token',
21165: 										'NULL'	=>	true
21166: 									),
21167: 				'hash'	=>		array(
21168: 										'name'	=>	'hash',
21169: 										'NULL'	=>	true
21170: 									),
21171: 				'secret'	=>		array(
21172: 										'name'	=>	'secret',
21173: 										'NULL'	=>	true
21174: 									),
21175: 				'host'	=>		array(
21176: 										'name'	=>	'host',
21177: 										'NULL'	=>	true
21178: 									),	
21179: 				'port'	=>		array(
```

### `archive_17012026_1259/taxi/models/api.php:21164`
```php
21156: 										'name'	=>	'key1',
21157: 										'NULL'	=>	true
21158: 									),
21159: 				'key2'	=>		array(
21160: 										'name'	=>	'key2',
21161: 										'NULL'	=>	true
21162: 									),
21163: 				'token'	=>		array(
21164: 										'name'	=>	'token',
21165: 										'NULL'	=>	true
21166: 									),
21167: 				'hash'	=>		array(
21168: 										'name'	=>	'hash',
21169: 										'NULL'	=>	true
21170: 									),
21171: 				'secret'	=>		array(
21172: 										'name'	=>	'secret',
21173: 										'NULL'	=>	true
21174: 									),
21175: 				'host'	=>		array(
21176: 										'name'	=>	'host',
21177: 										'NULL'	=>	true
21178: 									),	
21179: 				'port'	=>		array(
21180: 										'name'	=>	'port',
```

### `archive_17012026_1259/taxi/models/api.php:21367`
```php
21359: 				'key1'	=>		array(
21360: 										'name'	=>	'key1',
21361: 										'NULL'	=>	true
21362: 									),
21363: 				'key2'	=>		array(
21364: 										'name'	=>	'key2',
21365: 										'NULL'	=>	true
21366: 									),
21367: 				'token'	=>		array(
21368: 										'name'	=>	'token',
21369: 										'NULL'	=>	true
21370: 									),
21371: 				'hash'	=>		array(
21372: 										'name'	=>	'hash',
21373: 										'NULL'	=>	true
21374: 									),
21375: 				'secret'	=>		array(
21376: 										'name'	=>	'secret',
21377: 										'NULL'	=>	true
21378: 									),
21379: 				'host'	=>		array(
21380: 										'name'	=>	'host',
21381: 										'NULL'	=>	true
21382: 									),	
21383: 				'port'	=>		array(
```

### `archive_17012026_1259/taxi/models/api.php:21368`
```php
21360: 										'name'	=>	'key1',
21361: 										'NULL'	=>	true
21362: 									),
21363: 				'key2'	=>		array(
21364: 										'name'	=>	'key2',
21365: 										'NULL'	=>	true
21366: 									),
21367: 				'token'	=>		array(
21368: 										'name'	=>	'token',
21369: 										'NULL'	=>	true
21370: 									),
21371: 				'hash'	=>		array(
21372: 										'name'	=>	'hash',
21373: 										'NULL'	=>	true
21374: 									),
21375: 				'secret'	=>		array(
21376: 										'name'	=>	'secret',
21377: 										'NULL'	=>	true
21378: 									),
21379: 				'host'	=>		array(
21380: 										'name'	=>	'host',
21381: 										'NULL'	=>	true
21382: 									),	
21383: 				'port'	=>		array(
21384: 										'name'	=>	'port',
```

### `archive_17012026_1259/taxi/models/api.php:24296`
```php
24288: 
24289: 			return array(
24290: 				'code' 		=>	'200',
24291: 				'status' 	=>	'success',		
24292: 				'data' 		=>	$d
24293: 			);
24294: 		}
24295: 
24296: 		public $session_token = NULL;
24297: 
24298: 		private function createSessionToken()
24299: 		{
24300: 		
24301: 		
24302: 		}
24303: 
24304: 		private function checkSessionToken($session_token = '')
24305: 		{
24306: /*
24307: 			$this->constant['session_token_duration']*/
24308: 		
24309: 		}
24310: 
24311: 		private function checkPromocode($value = '',$actual = false, $id_schedule = array())
24312: 		{
```

### `archive_17012026_1259/taxi/models/api.php:24298`
```php
24290: 				'code' 		=>	'200',
24291: 				'status' 	=>	'success',		
24292: 				'data' 		=>	$d
24293: 			);
24294: 		}
24295: 
24296: 		public $session_token = NULL;
24297: 
24298: 		private function createSessionToken()
24299: 		{
24300: 		
24301: 		
24302: 		}
24303: 
24304: 		private function checkSessionToken($session_token = '')
24305: 		{
24306: /*
24307: 			$this->constant['session_token_duration']*/
24308: 		
24309: 		}
24310: 
24311: 		private function checkPromocode($value = '',$actual = false, $id_schedule = array())
24312: 		{
24313: 			$sql_if = '1';
24314: 			if ($actual == true)
```

### `archive_17012026_1259/taxi/models/api.php:24304`
```php
24296: 		public $session_token = NULL;
24297: 
24298: 		private function createSessionToken()
24299: 		{
24300: 		
24301: 		
24302: 		}
24303: 
24304: 		private function checkSessionToken($session_token = '')
24305: 		{
24306: /*
24307: 			$this->constant['session_token_duration']*/
24308: 		
24309: 		}
24310: 
24311: 		private function checkPromocode($value = '',$actual = false, $id_schedule = array())
24312: 		{
24313: 			$sql_if = '1';
24314: 			if ($actual == true)
24315: 			{
24316: 				$sql_if = "`active` = '1' AND now() < `limit`";
24317: 			}
24318: 			$sql_promocode = '`id_promocode`';
24319: 			$sql_schedule = '`id_schedule`';
24320: 			$sql_if_add = '';
```

### `archive_17012026_1259/taxi/models/api.php:24307`
```php
24299: 		{
24300: 		
24301: 		
24302: 		}
24303: 
24304: 		private function checkSessionToken($session_token = '')
24305: 		{
24306: /*
24307: 			$this->constant['session_token_duration']*/
24308: 		
24309: 		}
24310: 
24311: 		private function checkPromocode($value = '',$actual = false, $id_schedule = array())
24312: 		{
24313: 			$sql_if = '1';
24314: 			if ($actual == true)
24315: 			{
24316: 				$sql_if = "`active` = '1' AND now() < `limit`";
24317: 			}
24318: 			$sql_promocode = '`id_promocode`';
24319: 			$sql_schedule = '`id_schedule`';
24320: 			$sql_if_add = '';
24321: 			if (!empty($id_schedule))
24322: 			{ 
24323: 				$id_schedule = implode(',',$id_schedule);
```

### `archive_17012026_1259/taxi/models/m_functions.php:247`
```php
239: 		@$d = fetch_assoc($q);
240: 		return array(
241: 			  'auth'		=> $d !== NULL ? $d['a'] : true,
242: 			  'order'		=> isset($d['o']) ? $d['o'] :  NULL,
243: 			  'blog_topic'	=> isset($d['bt']) ? $d['bt'] :  NULL,
244: 			  'blog_post'	=> isset($d['bp']) ? $d['bp'] :  NULL
245: 		);
246: 	}
247: 	function check_auth_user()
248: 	{
249: 		taxi::$check_auth_user_count++;
250: 		if (empty($_SESSION[UID])) return false;
251: 		$s = "SELECT 
252: 				`id_role`,
253: 				`id_user`,
254: 				`name`,
255: 				`family`,
256: 				`middle`,
257: 				`phone`,
258: 				`email`,
259: 				`photo_link`,
260: 				`id_lang`,
261: 				`currency`,
262: 				`id_navigation`,
263: 				`id_verification_status`,
```

### `archive_17012026_1259/taxi/models/m_functions.php:249`
```php
241: 			  'auth'		=> $d !== NULL ? $d['a'] : true,
242: 			  'order'		=> isset($d['o']) ? $d['o'] :  NULL,
243: 			  'blog_topic'	=> isset($d['bt']) ? $d['bt'] :  NULL,
244: 			  'blog_post'	=> isset($d['bp']) ? $d['bp'] :  NULL
245: 		);
246: 	}
247: 	function check_auth_user()
248: 	{
249: 		taxi::$check_auth_user_count++;
250: 		if (empty($_SESSION[UID])) return false;
251: 		$s = "SELECT 
252: 				`id_role`,
253: 				`id_user`,
254: 				`name`,
255: 				`family`,
256: 				`middle`,
257: 				`phone`,
258: 				`email`,
259: 				`photo_link`,
260: 				`id_lang`,
261: 				`currency`,
262: 				`id_navigation`,
263: 				`id_verification_status`,
264: 				`deleted`,
265: 				`active`,
```

### `archive_17012026_1259/taxi/models/m_functions.php:336`
```php
328: 						if ($u_a_role != 4 && $u_a_role != 6 && !empty(taxi::$data['user_roles'][$u_a_role]))
329: 						{
330: 							taxi::$id_role = $u_a_role;
331: 						}
332: 					}
333: 				}
334: 				
335: 				$role_db_user = $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/list/' . CONFIG . '/role' . taxi::$id_role . '.php';
336: 				if (taxi::$check_auth_user_count === 1 && is_file($role_db_user))
337: 				{
338: 					include_once($role_db_user);
339: 					global $link;
340: 					mysqli_close($link);
341: 					define('USERNAME_F_R', $config_arr['USERNAME']);
342: 					define('PASSWORD_F_R', $config_arr['PASSWORD']);
343: 					if (!@$link = mysqli_connect(HOST, USERNAME_F_R, PASSWORD_F_R, DBNAME)) json_exit('400', 'error', 'Error with connection to database: ' . mysqli_connect_error(), NULL);
344: 					if (!query("SET NAMES 'utf8';")) json_exit('400', 'error', 'MySQL Error: ' . error_db(), NULL);
345: 					query("SET group_concat_max_len = 1024*1024;");
346: 				}			
347: 			}
348: 		}
349: 	}
350: 	
351: 	function mimeToext($type = '')
352: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:3402`
```php
3394: 		$iv = substr($c, 0, $ivlen);
3395: 		$hmac = substr($c, $ivlen, $sha2len=32);
3396: 		$ciphertext_raw = substr($c, $ivlen+$sha2len);
3397: 		$original_plaintext = openssl_decrypt($ciphertext_raw, $cipher, $key, $options=OPENSSL_RAW_DATA, $iv);
3398: 		$calcmac = hash_hmac('sha256', $ciphertext_raw, $key, $as_binary=true);
3399: 		if (@hash_equals($hmac, $calcmac)) return $original_plaintext; else return false;
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
```

### `archive_17012026_1259/taxi/models/m_functions.php:3404`
```php
3396: 		$ciphertext_raw = substr($c, $ivlen+$sha2len);
3397: 		$original_plaintext = openssl_decrypt($ciphertext_raw, $cipher, $key, $options=OPENSSL_RAW_DATA, $iv);
3398: 		$calcmac = hash_hmac('sha256', $ciphertext_raw, $key, $as_binary=true);
3399: 		if (@hash_equals($hmac, $calcmac)) return $original_plaintext; else return false;
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3407`
```php
3399: 		if (@hash_equals($hmac, $calcmac)) return $original_plaintext; else return false;
3400: 	}
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3409`
```php
3401: 
3402: 	function create_token()
3403: 	{
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
```

### `archive_17012026_1259/taxi/models/m_functions.php:3412`
```php
3404: 		return md5(microtime() . md5(CREATE_TOKEN_STRING . time()));
3405: 	}
3406: 
3407: 	function get_u_hash($token = '', $id_user = '')
3408: 	{
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
```

### `archive_17012026_1259/taxi/models/m_functions.php:3417`
```php
3409: 		return openssl____encrypt($id_user . '_' . md5(U_HASH_SECRET . md5($token)));
3410: 	}
3411: 
3412: 	function parse_u_hash($str = '', $token = '')
3413: 	{
3414: 		$str = openssl____decrypt($str);
3415: 		if ($str === false) return false;
3416: 		$arr = explode('_', $str);
3417: 		if (isset($arr[1]) && $arr[1] === md5(U_HASH_SECRET . md5($token)))
3418: 		{
3419: 			return $arr[0];
3420: 		}
3421: 		return false;
3422: 	}
3423: 
3424: 	if(!function_exists('hash_equals')) {
3425: 		function hash_equals($str1, $str2) {
3426: 			if(strlen($str1) != strlen($str2)) 
3427: 			{
3428: 				return false;
3429: 			} 
3430: 			else 
3431: 			{
3432: 				$res = $str1 ^ $str2;
3433: 				$ret = 0;
```

### `archive_17012026_1259/taxi/models/m_functions.php:3440`
```php
3432: 				$res = $str1 ^ $str2;
3433: 				$ret = 0;
3434: 				for($i = strlen($res) - 1; $i >= 0; $i--) $ret |= ord($res[$i]);
3435: 				return !$ret;
3436: 			}
3437: 		}
3438: 	}
3439: 
3440: 	function get_token($id_user = '')
3441: 	{
3442: 		$q = query("BEGIN");
3443: 		if ($q === false) return 0;
3444: 		
3445: 		$s = "SELECT 
3446: 				`token`
3447: 			FROM `token`
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
```

### `archive_17012026_1259/taxi/models/m_functions.php:3446`
```php
3438: 	}
3439: 
3440: 	function get_token($id_user = '')
3441: 	{
3442: 		$q = query("BEGIN");
3443: 		if ($q === false) return 0;
3444: 		
3445: 		$s = "SELECT 
3446: 				`token`
3447: 			FROM `token`
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
```

### `archive_17012026_1259/taxi/models/m_functions.php:3447`
```php
3439: 
3440: 	function get_token($id_user = '')
3441: 	{
3442: 		$q = query("BEGIN");
3443: 		if ($q === false) return 0;
3444: 		
3445: 		$s = "SELECT 
3446: 				`token`
3447: 			FROM `token`
3448: 			WHERE
3449: 				`id_user` = '" . $id_user . "'
3450: 			FOR UPDATE
3451: 			";
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3460`
```php
3452: 
3453: 		$q = query($s);
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
```

### `archive_17012026_1259/taxi/models/m_functions.php:3462`
```php
3454: 		if ($q === false) 
3455: 		{
3456: 			$q = query("ROLLBACK");
3457: 			if ($q === false) {return -2;} else {return -1;}
3458: 		}
3459: 		$d = fetch_assoc($q);
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3468`
```php
3460: 		if (isset($d['token']))
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3469`
```php
3461: 		{
3462: 			return $d['token'];
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:3471`
```php
3463: 		}
3464: 		else
3465: 		{
3466: 			global $link;
3467: 			do {
3468: 				$token = create_token();
3469: 				$s = "INSERT IGNORE INTO `token`
3470: 					SET 
3471: 						`token` = '" . $token . "',
3472: 						`id_user` = '" . $id_user . "'
3473: 					";
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:3482`
```php
3474: 
3475: 				$q = query($s);
3476: 			} while (mysqli_affected_rows($link) == 0);
3477: 			if (mysqli_affected_rows($link) == -1) return -3;
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
```

### `archive_17012026_1259/taxi/models/m_functions.php:3486`
```php
3478: 
3479: 			$q = query("COMMIT");
3480: 			if ($q === false) return -4;
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
```

### `archive_17012026_1259/taxi/models/m_functions.php:3489`
```php
3481: 
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3490`
```php
3482: 			return $token;
3483: 		}
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:3492`
```php
3484: 	}
3485: 
3486: 	function token_exists($token = '', $id_user = '')
3487: 	{
3488: 		$s = "SELECT 
3489: 				`token`
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
```

### `archive_17012026_1259/taxi/models/m_functions.php:3498`
```php
3490: 			FROM `token`
3491: 			WHERE
3492: 				`id_user` = '" . $id_user . "' AND `token` = '" . $token . "'
3493: 			";
3494: 
3495: 		$q = query($s);
3496: 		if ($q === false) return 0;
3497: 		$d = fetch_assoc($q);
3498: 		if (isset($d['token']))
3499: 		{
3500: 			return true;
3501: 		}
3502: 		else
3503: 		{
3504: 			return false;
3505: 		}
3506: 	}
3507: 
3508: 	function add_time_zone(&$date_str)
3509: 	{
3510: 		$date_str = empty($date_str) || $date_str === '0000-00-00 00:00:00' ? NULL : $date_str . MYSQL_TIME_ZONE;
3511: 	}
3512: 
3513: 	function replaceIfBlocks($key = "", $value = "", $str = "")
3514: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4745`
```php
4737: 		$contents = curl_exec($c);
4738: 		curl_close($c);
4739: 		if ($contents) return $contents; else return false;
4740: 	}
4741: 
4742: 	 function auth_user($id_user)
4743: 	 {
4744: 		$_SESSION[UID] = $id_user;
4745: 		check_auth_user();
4746: 		if (empty($_SESSION[UID])) return false;
4747: 		$_SESSION['auth_time'] = time();
4748: 		$vfoc = md5(session_id() . strtolower(dechex(crc32($_SESSION[UID]))));
4749: 		setcookie("vfoc", $vfoc, 0, ROOT_URL);
4750: 		return openssl____encrypt(session_id() . "|" . $vfoc);
4751: 	 }
4752: 
4753: 	function upload_to_dropbox($content,$filename_upload,$id_dropbox_link,$mode = 'add')
4754: 	{
4755: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4756: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4757: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4758: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4759: 		switch ($mode)
4760: 		{
4761: 			case 'add':
```

### `archive_17012026_1259/taxi/models/m_functions.php:4757`
```php
4749: 		setcookie("vfoc", $vfoc, 0, ROOT_URL);
4750: 		return openssl____encrypt(session_id() . "|" . $vfoc);
4751: 	 }
4752: 
4753: 	function upload_to_dropbox($content,$filename_upload,$id_dropbox_link,$mode = 'add')
4754: 	{
4755: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4756: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4757: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4758: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4759: 		switch ($mode)
4760: 		{
4761: 			case 'add':
4762: 				break;
4763: 			case 'overwrite':
4764: 				break;
4765: 			default:
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
```

### `archive_17012026_1259/taxi/models/m_functions.php:4758`
```php
4750: 		return openssl____encrypt(session_id() . "|" . $vfoc);
4751: 	 }
4752: 
4753: 	function upload_to_dropbox($content,$filename_upload,$id_dropbox_link,$mode = 'add')
4754: 	{
4755: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4756: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4757: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4758: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4759: 		switch ($mode)
4760: 		{
4761: 			case 'add':
4762: 				break;
4763: 			case 'overwrite':
4764: 				break;
4765: 			default:
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4769`
```php
4761: 			case 'add':
4762: 				break;
4763: 			case 'overwrite':
4764: 				break;
4765: 			default:
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4771`
```php
4763: 			case 'overwrite':
4764: 				break;
4765: 			default:
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
```

### `archive_17012026_1259/taxi/models/m_functions.php:4773`
```php
4765: 			default:
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
```

### `archive_17012026_1259/taxi/models/m_functions.php:4774`
```php
4766: 				$mode = 'add';
4767: 		}
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4776`
```php
4768: 
4769: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4770: 		$update = false;
4771: 		if (empty($dropbox_access_token))
4772: 		{
4773: 			$dropbox_access_token = update_dropbox_access_token();
4774: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4775: 			$update = true;
4776: 			$dropbox_access_token = $dropbox_access_token['value'];
4777: 		}
4778: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
4791: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4792: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
```

### `archive_17012026_1259/taxi/models/m_functions.php:4787`
```php
4779: 		while (true)
4780: 		{
4781: 			$c = curl_init();
4782: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4783: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/upload');
4784: 			curl_setopt($c,CURLOPT_POST, 1);
4785: 			curl_setopt($c,CURLOPT_POSTFIELDS,$content);
4786: 			$headers = array(
4787: 				"Authorization: Bearer $dropbox_access_token",
4788: 				"Dropbox-API-Arg: {\"path\":\"$path\",\"mode\":\"$mode\"}",
4789: 				"Content-Type: application/octet-stream"
4790: 			);
4791: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4792: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4793: 			$response = curl_exec($c);
4794: 			curl_close($c);
4795: 			if (empty($response)) return array('error'=>'upload failed');
4796: 			@$response = json_decode($response,true);
4797: 			if (empty($response) || !is_array($response)) 
4798: 			{
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
```

### `archive_17012026_1259/taxi/models/m_functions.php:4803`
```php
4795: 			if (empty($response)) return array('error'=>'upload failed');
4796: 			@$response = json_decode($response,true);
4797: 			if (empty($response) || !is_array($response)) 
4798: 			{
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
4814: 						$dropbox_access_token = $dropbox_access_token['value'];
4815: 						continue;
4816: 					}
4817: 				}
4818: 				else
4819: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4807`
```php
4799: 				 return array('error'=>'upload response error');
4800: 			}
4801: 			if (!empty($response['error']))
4802: 			{
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
4814: 						$dropbox_access_token = $dropbox_access_token['value'];
4815: 						continue;
4816: 					}
4817: 				}
4818: 				else
4819: 				{
4820: 					 return array('error'=>'upload error');
4821: 				}
4822: 			}
4823: 			return array('data'=>$response);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4811`
```php
4803: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
4814: 						$dropbox_access_token = $dropbox_access_token['value'];
4815: 						continue;
4816: 					}
4817: 				}
4818: 				else
4819: 				{
4820: 					 return array('error'=>'upload error');
4821: 				}
4822: 			}
4823: 			return array('data'=>$response);
4824: 		}	
4825: 	}
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
```

### `archive_17012026_1259/taxi/models/m_functions.php:4812`
```php
4804: 				{
4805: 					if ($update == true)
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
4814: 						$dropbox_access_token = $dropbox_access_token['value'];
4815: 						continue;
4816: 					}
4817: 				}
4818: 				else
4819: 				{
4820: 					 return array('error'=>'upload error');
4821: 				}
4822: 			}
4823: 			return array('data'=>$response);
4824: 		}	
4825: 	}
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
4828: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4814`
```php
4806: 					{
4807: 						return array('error'=>'access_token error');
4808: 					}
4809: 					else
4810: 					{
4811: 						$dropbox_access_token = update_dropbox_access_token();
4812: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4813: 						$update = true;
4814: 						$dropbox_access_token = $dropbox_access_token['value'];
4815: 						continue;
4816: 					}
4817: 				}
4818: 				else
4819: 				{
4820: 					 return array('error'=>'upload error');
4821: 				}
4822: 			}
4823: 			return array('data'=>$response);
4824: 		}	
4825: 	}
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
4828: 	{
4829: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4831`
```php
4823: 			return array('data'=>$response);
4824: 		}	
4825: 	}
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
4828: 	{
4829: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
```

### `archive_17012026_1259/taxi/models/m_functions.php:4832`
```php
4824: 		}	
4825: 	}
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
4828: 	{
4829: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
```

### `archive_17012026_1259/taxi/models/m_functions.php:4834`
```php
4826: 
4827: 	function download_from_dropbox($filename_upload = '', $id_dropbox_link = '', $type = '')
4828: 	{
4829: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
```

### `archive_17012026_1259/taxi/models/m_functions.php:4836`
```php
4828: 	{
4829: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4838`
```php
4830: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4839`
```php
4831: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4832: 		if (empty($filename_upload) || empty($id_dropbox_link) || empty($type) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4841`
```php
4833: 
4834: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
4835: 		$update = false;
4836: 		if (empty($dropbox_access_token))
4837: 		{
4838: 			$dropbox_access_token = update_dropbox_access_token();
4839: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
4840: 			$update = true;
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4849`
```php
4841: 			$dropbox_access_token = $dropbox_access_token['value'];
4842: 		}
4843: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link/$filename_upload";
4844: 		while (true)
4845: 		{
4846: 			$c = curl_init();
4847: 			curl_setopt($c,CURLOPT_URL, 'https://content.dropboxapi.com/2/files/download');
4848: 			$headers = array(
4849: 				"Authorization: Bearer $dropbox_access_token",
4850: 				"Dropbox-API-Arg: {\"path\":\"$path\"}"
4851: 			);
4852: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4853: 			curl_setopt($c, CURLOPT_HEADERFUNCTION, function($curl, $header) use (&$headers){
4854: 				$len = strlen($header);
4855: 				$header = explode(':', $header, 2);
4856: 				if (count($header) < 2) 
4857: 				{
4858: 					$header = trim($header[0]);				
4859: 					if (strlen($header) && mb_strpos($header,'200') === false) curl_setopt($curl,CURLOPT_RETURNTRANSFER, 1);
4860: 					return $len;
4861: 				}
4862: 				$header[0] = trim($header[0]);
4863: 				$header[1] = trim($header[1]);
4864: 				if (strtolower($header[0]) == 'content-length') header("Content-Length: {$header[1]}");
4865: 				$headers[$header[0]][] = $header[1];
```

### `archive_17012026_1259/taxi/models/m_functions.php:4886`
```php
4878: 			if (empty($response)) return array('error'=>'download failed');
4879: 			@$response = json_decode($response,true);
4880: 			if (empty($response) || !is_array($response)) 
4881: 			{
4882: 				 return array('error'=>'download response error');
4883: 			}
4884: 			if (!empty($response['error']))
4885: 			{
4886: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4887: 				{
4888: 					if ($update == true)
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
4892: 					else
4893: 					{
4894: 						$dropbox_access_token = update_dropbox_access_token();
4895: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4896: 						$update = true;
4897: 						$dropbox_access_token = $dropbox_access_token['value'];
4898: 						continue;
4899: 					}
4900: 				}
4901: 				else
4902: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4890`
```php
4882: 				 return array('error'=>'download response error');
4883: 			}
4884: 			if (!empty($response['error']))
4885: 			{
4886: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4887: 				{
4888: 					if ($update == true)
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
4892: 					else
4893: 					{
4894: 						$dropbox_access_token = update_dropbox_access_token();
4895: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4896: 						$update = true;
4897: 						$dropbox_access_token = $dropbox_access_token['value'];
4898: 						continue;
4899: 					}
4900: 				}
4901: 				else
4902: 				{
4903: 					 return array('error'=>'download error');
4904: 				}
4905: 			}
4906: 		}		
```

### `archive_17012026_1259/taxi/models/m_functions.php:4894`
```php
4886: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
4887: 				{
4888: 					if ($update == true)
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
4892: 					else
4893: 					{
4894: 						$dropbox_access_token = update_dropbox_access_token();
4895: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4896: 						$update = true;
4897: 						$dropbox_access_token = $dropbox_access_token['value'];
4898: 						continue;
4899: 					}
4900: 				}
4901: 				else
4902: 				{
4903: 					 return array('error'=>'download error');
4904: 				}
4905: 			}
4906: 		}		
4907: 	}
4908: 
4909: 	function update_dropbox($code = '')
4910: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4895`
```php
4887: 				{
4888: 					if ($update == true)
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
4892: 					else
4893: 					{
4894: 						$dropbox_access_token = update_dropbox_access_token();
4895: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4896: 						$update = true;
4897: 						$dropbox_access_token = $dropbox_access_token['value'];
4898: 						continue;
4899: 					}
4900: 				}
4901: 				else
4902: 				{
4903: 					 return array('error'=>'download error');
4904: 				}
4905: 			}
4906: 		}		
4907: 	}
4908: 
4909: 	function update_dropbox($code = '')
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4897`
```php
4889: 					{
4890: 						return array('error'=>'access_token error');
4891: 					}
4892: 					else
4893: 					{
4894: 						$dropbox_access_token = update_dropbox_access_token();
4895: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
4896: 						$update = true;
4897: 						$dropbox_access_token = $dropbox_access_token['value'];
4898: 						continue;
4899: 					}
4900: 				}
4901: 				else
4902: 				{
4903: 					 return array('error'=>'download error');
4904: 				}
4905: 			}
4906: 		}		
4907: 	}
4908: 
4909: 	function update_dropbox($code = '')
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4912: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
```

### `archive_17012026_1259/taxi/models/m_functions.php:4915`
```php
4907: 	}
4908: 
4909: 	function update_dropbox($code = '')
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4912: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
4914: 		
4915: 		$postvars = "code=" . urlencode($code) . "&grant_type=authorization_code&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4916: 		$c = curl_init();
4917: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4918: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4919: 		curl_setopt($c,CURLOPT_POST, 1);
4920: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4921: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4922: 		$headers = array(
4923: 			"Content-Type: application/x-www-form-urlencoded"
4924: 		);
4925: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4926: 		$response = curl_exec($c);
4927: 		curl_close($c);
4928: 		if (empty($response)) return array('error'=>'update failed');
4929: 		@$response = json_decode($response,true);
4930: 		if (empty($response) || !is_array($response)) 
4931: 		{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4918`
```php
4910: 	{
4911: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4912: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4913: 		if (strlen($code) == 0 || empty($client_id) || empty($client_secret)) return array('error'=>'empty parameters');
4914: 		
4915: 		$postvars = "code=" . urlencode($code) . "&grant_type=authorization_code&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4916: 		$c = curl_init();
4917: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4918: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4919: 		curl_setopt($c,CURLOPT_POST, 1);
4920: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4921: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4922: 		$headers = array(
4923: 			"Content-Type: application/x-www-form-urlencoded"
4924: 		);
4925: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4926: 		$response = curl_exec($c);
4927: 		curl_close($c);
4928: 		if (empty($response)) return array('error'=>'update failed');
4929: 		@$response = json_decode($response,true);
4930: 		if (empty($response) || !is_array($response)) 
4931: 		{
4932: 			 return array('error'=>'update response error');
4933: 		}
4934: 		if (!empty($response['error']))
```

### `archive_17012026_1259/taxi/models/m_functions.php:4945`
```php
4937: 			{
4938: 				return array('error'=>'code expired');
4939: 			}
4940: 			else
4941: 			{
4942: 				return array('error'=>'update error');
4943: 			}
4944: 		}
4945: 		if (empty($response['refresh_token'])) return array('error'=>'empty refresh_token');
4946: 		$refresh_token = $response['refresh_token'];
4947: 
4948: 		$s = "UPDATE `site_constant`
4949: 			SET
4950: 				`value` = '" . $refresh_token . "'
4951: 			WHERE
4952: 				`var` = 'dropbox_file_refresh_token'
4953: 			";
4954: 
4955: 		$q = query($s);
4956: 
4957: 		global $link;
4958: 		if (mysqli_affected_rows($link) === -1) 
4959: 		{
4960: 			return array('error'=>'database update failed');
4961: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:4946`
```php
4938: 				return array('error'=>'code expired');
4939: 			}
4940: 			else
4941: 			{
4942: 				return array('error'=>'update error');
4943: 			}
4944: 		}
4945: 		if (empty($response['refresh_token'])) return array('error'=>'empty refresh_token');
4946: 		$refresh_token = $response['refresh_token'];
4947: 
4948: 		$s = "UPDATE `site_constant`
4949: 			SET
4950: 				`value` = '" . $refresh_token . "'
4951: 			WHERE
4952: 				`var` = 'dropbox_file_refresh_token'
4953: 			";
4954: 
4955: 		$q = query($s);
4956: 
4957: 		global $link;
4958: 		if (mysqli_affected_rows($link) === -1) 
4959: 		{
4960: 			return array('error'=>'database update failed');
4961: 		}
4962: 		elseif (mysqli_affected_rows($link) === 0) 
```

### `archive_17012026_1259/taxi/models/m_functions.php:4950`
```php
4942: 				return array('error'=>'update error');
4943: 			}
4944: 		}
4945: 		if (empty($response['refresh_token'])) return array('error'=>'empty refresh_token');
4946: 		$refresh_token = $response['refresh_token'];
4947: 
4948: 		$s = "UPDATE `site_constant`
4949: 			SET
4950: 				`value` = '" . $refresh_token . "'
4951: 			WHERE
4952: 				`var` = 'dropbox_file_refresh_token'
4953: 			";
4954: 
4955: 		$q = query($s);
4956: 
4957: 		global $link;
4958: 		if (mysqli_affected_rows($link) === -1) 
4959: 		{
4960: 			return array('error'=>'database update failed');
4961: 		}
4962: 		elseif (mysqli_affected_rows($link) === 0) 
4963: 		{
4964: 			return array('error'=>'modified data not found');
4965: 		}
4966: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:4952`
```php
4944: 		}
4945: 		if (empty($response['refresh_token'])) return array('error'=>'empty refresh_token');
4946: 		$refresh_token = $response['refresh_token'];
4947: 
4948: 		$s = "UPDATE `site_constant`
4949: 			SET
4950: 				`value` = '" . $refresh_token . "'
4951: 			WHERE
4952: 				`var` = 'dropbox_file_refresh_token'
4953: 			";
4954: 
4955: 		$q = query($s);
4956: 
4957: 		global $link;
4958: 		if (mysqli_affected_rows($link) === -1) 
4959: 		{
4960: 			return array('error'=>'database update failed');
4961: 		}
4962: 		elseif (mysqli_affected_rows($link) === 0) 
4963: 		{
4964: 			return array('error'=>'modified data not found');
4965: 		}
4966: 
4967: 		$res = update_cache();
4968: 		if (!empty($res['error']))
```

### `archive_17012026_1259/taxi/models/m_functions.php:4973`
```php
4965: 		}
4966: 
4967: 		$res = update_cache();
4968: 		if (!empty($res['error']))
4969: 		{
4970: 			return array('error'=>array('update_cache'=>$res));
4971: 		}
4972: 
4973: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt','') === false) return array('error'=>'access_token clear error');
4974: 
4975: 		return array('refresh_token' => $refresh_token);
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4975`
```php
4967: 		$res = update_cache();
4968: 		if (!empty($res['error']))
4969: 		{
4970: 			return array('error'=>array('update_cache'=>$res));
4971: 		}
4972: 
4973: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt','') === false) return array('error'=>'access_token clear error');
4974: 
4975: 		return array('refresh_token' => $refresh_token);
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
```

### `archive_17012026_1259/taxi/models/m_functions.php:4978`
```php
4970: 			return array('error'=>array('update_cache'=>$res));
4971: 		}
4972: 
4973: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt','') === false) return array('error'=>'access_token clear error');
4974: 
4975: 		return array('refresh_token' => $refresh_token);
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4982`
```php
4974: 
4975: 		return array('refresh_token' => $refresh_token);
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
```

### `archive_17012026_1259/taxi/models/m_functions.php:4983`
```php
4975: 		return array('refresh_token' => $refresh_token);
4976: 	}
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
```

### `archive_17012026_1259/taxi/models/m_functions.php:4985`
```php
4977: 
4978: 	function update_dropbox_access_token()
4979: 	{
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
```

### `archive_17012026_1259/taxi/models/m_functions.php:4988`
```php
4980: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
4981: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
4982: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
4983: 		if (empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('value'=>'','error'=>'access_token: empty parameters');
4984: 
4985: 		$postvars = "grant_type=refresh_token&refresh_token=" . $refresh_token . "&client_id=" . urlencode($client_id) . "&client_secret=" . urlencode($client_secret);
4986: 		$c = curl_init();
4987: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
4988: 		curl_setopt($c,CURLOPT_URL, 'https://api.dropbox.com/oauth2/token');
4989: 		curl_setopt($c,CURLOPT_POST, 1);
4990: 		curl_setopt($c,CURLOPT_POSTFIELDS,$postvars);
4991: 		$headers = array(
4992: 			"Content-Type: application/x-www-form-urlencoded"
4993: 		);
4994: 		curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
4995: 		curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
4996: 		$response = curl_exec($c);
4997: 		curl_close($c);
4998: 		if (empty($response)) return array('value'=>'','error'=>'exec failed');
4999: 		@$response = json_decode($response,true);
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
```

### `archive_17012026_1259/taxi/models/m_functions.php:5008`
```php
5000: 		if (empty($response) || !is_array($response)) 
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
5014: 	}
5015: 
5016: 	function fields_for_langs($langs = array(), &$arr = array(), $out_prefix = '', $db_prefix = '', $null_on = false)
5017: 	{
5018: 		if (empty($langs)) return false;
5019: 		foreach ($langs as $lang)
5020: 		{
5021: 			$out_key = $out_prefix . $lang['iso'];
5022: 			$db_key = $db_prefix . $lang['iso'];
5023: 			$arr['allowed_fields'][$out_key] = array(
5024: 								'name'	=>	$db_key ,
```

### `archive_17012026_1259/taxi/models/m_functions.php:5009`
```php
5001: 		{
5002: 			 return array('value'=>'','error'=>'exec response error');
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
5014: 	}
5015: 
5016: 	function fields_for_langs($langs = array(), &$arr = array(), $out_prefix = '', $db_prefix = '', $null_on = false)
5017: 	{
5018: 		if (empty($langs)) return false;
5019: 		foreach ($langs as $lang)
5020: 		{
5021: 			$out_key = $out_prefix . $lang['iso'];
5022: 			$db_key = $db_prefix . $lang['iso'];
5023: 			$arr['allowed_fields'][$out_key] = array(
5024: 								'name'	=>	$db_key ,
5025: 								'NULL'	=>	$null_on
```

### `archive_17012026_1259/taxi/models/m_functions.php:5011`
```php
5003: 		}
5004: 		if (!empty($response['error_summary']))
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
5014: 	}
5015: 
5016: 	function fields_for_langs($langs = array(), &$arr = array(), $out_prefix = '', $db_prefix = '', $null_on = false)
5017: 	{
5018: 		if (empty($langs)) return false;
5019: 		foreach ($langs as $lang)
5020: 		{
5021: 			$out_key = $out_prefix . $lang['iso'];
5022: 			$db_key = $db_prefix . $lang['iso'];
5023: 			$arr['allowed_fields'][$out_key] = array(
5024: 								'name'	=>	$db_key ,
5025: 								'NULL'	=>	$null_on
5026: 			);
5027: 		}
```

### `archive_17012026_1259/taxi/models/m_functions.php:5013`
```php
5005: 		{
5006: 			return array('value'=>'','error'=>'exec error');
5007: 		}
5008: 		if (empty($response['access_token'])) return array('value'=>'','error'=>'empty access_token');
5009: 		$access_token = $response['access_token'];
5010: 		
5011: 		if (@file_put_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt',$access_token) === false) return array('error'=>'access_token write error');
5012: 
5013: 		return array('value'=>$access_token);
5014: 	}
5015: 
5016: 	function fields_for_langs($langs = array(), &$arr = array(), $out_prefix = '', $db_prefix = '', $null_on = false)
5017: 	{
5018: 		if (empty($langs)) return false;
5019: 		foreach ($langs as $lang)
5020: 		{
5021: 			$out_key = $out_prefix . $lang['iso'];
5022: 			$db_key = $db_prefix . $lang['iso'];
5023: 			$arr['allowed_fields'][$out_key] = array(
5024: 								'name'	=>	$db_key ,
5025: 								'NULL'	=>	$null_on
5026: 			);
5027: 		}
5028: 		return true;
5029: 	}
```

### `archive_17012026_1259/taxi/models/m_functions.php:6313`
```php
6305: 
6306:         return $data . $header;
6307:     }
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
```

### `archive_17012026_1259/taxi/models/m_functions.php:6314`
```php
6306:         return $data . $header;
6307:     }
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
6330: 			$msg = [];
```

### `archive_17012026_1259/taxi/models/m_functions.php:6316`
```php
6308: 
6309: 	function send_msg_to_tg($subject = "", $body = "", $sender_phone = "", $recipient = "", $code = "")
6310: 	{
6311: 		if ($subject === "" && $body === "" && $code === "") return array('error'=>'empty input data');
6312: 		if (!empty(taxi::$data_private['site_constants']['telegram_url']['value'])) $url = trim(taxi::$data_private['site_constants']['telegram_url']['value']);
6313: 		if (!empty(taxi::$data_private['site_constants']['telegram_token']['value'])) $token = trim(taxi::$data_private['site_constants']['telegram_token']['value']);
6314: 		if (empty($url) || empty($token)) return array('error'=>'empty parameters');
6315: 		$arr = array(
6316: 							'token'			=>		$token,
6317: 							'site'			=>		url('',0,1),
6318: 							'c'				=>		CONFIG
6319: 
6320: 		);
6321: 		if ($sender_phone === "")
6322: 		{
6323: 			return array('error'=>'empty sender phone');
6324: 		}
6325: 		$arr['sender']['phone'] = $sender_phone;
6326: 		if ($code === "")
6327: 		{
6328: 			if ($recipient === "") return array('error'=>'empty recipient');
6329: 			$arr['recipient'] = $recipient;
6330: 			$msg = [];
6331: 			if ($subject !== "") $msg[] = $subject;
6332: 			if ($body !== "") $msg[] = $body;
```

### `archive_17012026_1259/taxi/models/m_functions.php:6760`
```php
6752: 			header("Content-disposition: inline; filename=\"$filename\"; filename*=UTF-8''$filename_encoded");
6753: 		}
6754: 	}
6755: 
6756: 	function delete_from_dropbox($id_dropbox_link = '')
6757: 	{
6758: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
6759: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
```

### `archive_17012026_1259/taxi/models/m_functions.php:6761`
```php
6753: 		}
6754: 	}
6755: 
6756: 	function delete_from_dropbox($id_dropbox_link = '')
6757: 	{
6758: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
6759: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
```

### `archive_17012026_1259/taxi/models/m_functions.php:6763`
```php
6755: 
6756: 	function delete_from_dropbox($id_dropbox_link = '')
6757: 	{
6758: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
6759: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
```

### `archive_17012026_1259/taxi/models/m_functions.php:6765`
```php
6757: 	{
6758: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value'])) $client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
6759: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
```

### `archive_17012026_1259/taxi/models/m_functions.php:6767`
```php
6759: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value'])) $client_secret = trim(taxi::$data_private['site_constants']['dropbox_file_client_secret']['value']);
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
```

### `archive_17012026_1259/taxi/models/m_functions.php:6768`
```php
6760: 		if (!empty(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value'])) $refresh_token = trim(taxi::$data_private['site_constants']['dropbox_file_refresh_token']['value']);
6761: 		if (empty($id_dropbox_link) || empty($client_id) || empty($client_secret) || empty($refresh_token)) return array('error'=>'empty parameters');
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
```

### `archive_17012026_1259/taxi/models/m_functions.php:6770`
```php
6762: 
6763: 		@$dropbox_access_token = trim(file_get_contents($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/dropbox_access_token.txt'));
6764: 		$update = false;
6765: 		if (empty($dropbox_access_token))
6766: 		{
6767: 			$dropbox_access_token = update_dropbox_access_token();
6768: 			if (!empty($dropbox_access_token['error'])) return array('error'=>'new: ' . $dropbox_access_token['error']);
6769: 			$update = true;
6770: 			$dropbox_access_token = $dropbox_access_token['value'];
6771: 		}
6772: 		$constant = 'constant'; $path = "/c/{$constant('CONFIG')}/$id_dropbox_link";
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
6785: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
6786: 			$response = curl_exec($c);
```

### `archive_17012026_1259/taxi/models/m_functions.php:6781`
```php
6773: 		while (true)
6774: 		{
6775: 			$c = curl_init();
6776: 			curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
6777: 			curl_setopt($c,CURLOPT_URL, 'https://api.dropboxapi.com/2/files/delete_v2');
6778: 			curl_setopt($c,CURLOPT_POST, 1);
6779: 			curl_setopt($c,CURLOPT_POSTFIELDS,"{\"path\":\"$path\"}");
6780: 			$headers = array(
6781: 				"Authorization: Bearer $dropbox_access_token",
6782: 				"Content-Type: application/json"
6783: 			);
6784: 			curl_setopt($c, CURLOPT_HTTPHEADER, $headers);
6785: 			curl_setopt($c, CURLOPT_CAINFO, $_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
6786: 			$response = curl_exec($c);
6787: 			curl_close($c);
6788: 			if (empty($response)) return array('error'=>'upload failed');
6789: 			@$response = json_decode($response,true);
6790: 			if (empty($response) || !is_array($response)) 
6791: 			{
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:6796`
```php
6788: 			if (empty($response)) return array('error'=>'upload failed');
6789: 			@$response = json_decode($response,true);
6790: 			if (empty($response) || !is_array($response)) 
6791: 			{
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
6807: 						$dropbox_access_token = $dropbox_access_token['value'];
6808: 						continue;
6809: 					}
6810: 				}
6811: 				else
6812: 				{
```

### `archive_17012026_1259/taxi/models/m_functions.php:6800`
```php
6792: 				 return array('error'=>'upload response error');
6793: 			}
6794: 			if (!empty($response['error']))
6795: 			{
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
6807: 						$dropbox_access_token = $dropbox_access_token['value'];
6808: 						continue;
6809: 					}
6810: 				}
6811: 				else
6812: 				{
6813: 					 return array('error'=>'upload error');
6814: 				}
6815: 			}
6816: 			return array('data'=>$response);
```

### `archive_17012026_1259/taxi/models/m_functions.php:6804`
```php
6796: 				if (!empty($response['error_summary']) && substr($response['error_summary'],0,20) == 'expired_access_token')
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
6807: 						$dropbox_access_token = $dropbox_access_token['value'];
6808: 						continue;
6809: 					}
6810: 				}
6811: 				else
6812: 				{
6813: 					 return array('error'=>'upload error');
6814: 				}
6815: 			}
6816: 			return array('data'=>$response);
6817: 		}	
6818: 	}
6819: 
6820: 	function html_to_pdf($content)
```

### `archive_17012026_1259/taxi/models/m_functions.php:6805`
```php
6797: 				{
6798: 					if ($update == true)
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
6807: 						$dropbox_access_token = $dropbox_access_token['value'];
6808: 						continue;
6809: 					}
6810: 				}
6811: 				else
6812: 				{
6813: 					 return array('error'=>'upload error');
6814: 				}
6815: 			}
6816: 			return array('data'=>$response);
6817: 		}	
6818: 	}
6819: 
6820: 	function html_to_pdf($content)
6821: 	{
```

### `archive_17012026_1259/taxi/models/m_functions.php:6807`
```php
6799: 					{
6800: 						return array('error'=>'access_token error');
6801: 					}
6802: 					else
6803: 					{
6804: 						$dropbox_access_token = update_dropbox_access_token();
6805: 						if (!empty($dropbox_access_token['error'])) return array('error'=>'update: ' . $dropbox_access_token['error']);
6806: 						$update = true;
6807: 						$dropbox_access_token = $dropbox_access_token['value'];
6808: 						continue;
6809: 					}
6810: 				}
6811: 				else
6812: 				{
6813: 					 return array('error'=>'upload error');
6814: 				}
6815: 			}
6816: 			return array('data'=>$response);
6817: 		}	
6818: 	}
6819: 
6820: 	function html_to_pdf($content)
6821: 	{
6822: 		require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/html2pdf-5.2.8/src/Html2Pdf.php');
6823: 
```

### `archive_17012026_1259/taxi/models/m_functions.php:7244`
```php
7236: 													'currency'		=>	$currency
7237: 												),
7238: /*			'payout_destination_data'		=>		array(
7239: 													'type'			=>	'bank_card',
7240: 													'card'			=>	array(
7241: 																			'number'	=>	$card_number
7242: 													)												
7243: 												)*/
7244: 			'payout_token'				=>		$card_number
7245: 		);
7246: 		if (!empty($option))
7247: 		{
7248: 			foreach($option as $arr_key=>$arr_val)
7249: 			{
7250: 				$arr['metadata'][$arr_key] = $arr_val;
7251: 			}
7252: 		}
7253: 		$c = curl_init();
7254: 		curl_setopt($c,CURLOPT_RETURNTRANSFER, 1);
7255: 		curl_setopt($c,CURLOPT_URL, $url);
7256: 		curl_setopt($c,CURLOPT_FOLLOWLOCATION, true);
7257: 		curl_setopt($c,CURLOPT_USERPWD, "$agentid:$key");
7258: 		curl_setopt($c,CURLOPT_POST, 1);
7259: 		curl_setopt($c,CURLOPT_POSTFIELDS,json_encode($arr));
7260: 		curl_setopt($c,CURLOPT_CAINFO,$_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'config/cacert.pem');
```

### `archive_17012026_1259/taxi/models/token.php:2`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
```

### `archive_17012026_1259/taxi/models/token.php:6`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
```

### `archive_17012026_1259/taxi/models/token.php:7`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
23: 	{
```

### `archive_17012026_1259/taxi/models/token.php:9`
```php
1: <?php	
2: if (!empty($_POST['token']) && !empty($_POST['u_hash']))
3: {
4: 	require_once($_SERVER['DOCUMENT_ROOT'] . ROOT_URL . 'models/m_functions.php');
5: 
6: 	$_SESSION['token_auth'] = parse_u_hash($_POST['u_hash'],$_POST['token']);
7: 	if ($_SESSION['token_auth'] !== false) 
8: 	{
9: 		if (token_exists($_POST['token'],$_SESSION['token_auth'])) $_SESSION[UID] = $_SESSION['token_auth'];
10: 	}
11: 	if (empty($_SESSION[UID])) 
12: 	{
13: 		if ($_GET['route'] !== 'api')
14: 		{
15: 			$_GET['route'] = 'api';
16: 			unset($_GET['par']);
17: 		}
18: 	}
19: } 
20: else
21: {
22: 	if (!empty($_POST['auth_hash'])) 
23: 	{
24: 		if ($_GET['route'] !== 'api')
25: 		{
```

### `archive_17012026_1259/taxi/config/config.php:77`
```php
69: 
70: 	define('HOST', $config_arr['HOST']);
71: 	define('DBNAME', $config_arr['DBNAME']); 
72: 	define('USERNAME', $config_arr['USERNAME']);
73: 	define('PASSWORD', $config_arr['PASSWORD']);
74: 
75: 	define('CONFIG_URL', ROOT_URL . CONFIG_URL_ROUTE . '/' . CONFIG . '/');
76: 	define('CONFIG_CACHE', ROOT_URL . 'cache/data' . (CONFIG ? '_' . CONFIG : ''));
77: 	class taxi {public static $data = array(); public static $version = ''; public static $check_auth_user_count = 0;public static $id_role = NULL; public static $data_private = array(); public static $data_stt = array(); public static $version_stt = ''; public static $data_sc = array(); public static $version_sc = '';}
78: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.php');
79: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.(stt).php');
80: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.(sc).php');
81: 	define('FILE_PATH', ROOT_URL . trim('files/'));
82: 	define('CONFIG_FOLDER', ROOT_URL . CONFIG_URL_ROUTE . 'f/' . CONFIG . '/');
83: 	define('CONFIG_FILE_PATH', CONFIG_FOLDER . trim('files/'));
84: 	define('CONFIG_USER_FILE_PATH', CONFIG_FOLDER . trim('files_u/'));
85: 	define('STRIP_ROOT', str_replace('/','_',preg_replace('/\/$/','',ROOT_URL,1))); 
86: 	define('UID_SUFFIX',STRIP_ROOT.(CONFIG ? '-' . CONFIG : ''));
87: 	define('UID','UID'.':'.ROOT_URL.':'.CONFIG);
88: 
89: 	if (!empty(taxi::$data['site_constants']['the_language_of_the_service']['value'])
90: 		&& !empty(taxi::$data['langs'][abs((int)taxi::$data['site_constants']['the_language_of_the_service']['value'])])) 
91: 	{
92: 		define('DEFAULT_LANG', abs((int)taxi::$data['site_constants']['the_language_of_the_service']['value']));
93: 	}
```

### `archive_17012026_1259/taxi/config/load_constant.php:5`
```php
1: <?php
2: 	define('ROOT_URL', trim('/taxi/'));
3: 	define('API_KEY', 'Api_keY_f0r_upD@te_Cache');
4: 	define('OPENSSL____ENCRYPT_KEY', '|<:* openssl____word *:>|');
5: 	define('CREATE_TOKEN_STRING', '/stringF0rTokenCre@ting/');
6: 	define('U_HASH_SECRET', '@_string_F0r_Token_Checking');
7: 	define('PAGE_EDIT_LANGS','_elvf');
8: 	define('PAGE_DEBUG','dEbug*_');
9: 	define('PAGE_CONTROL_CONFIG','{contr0l_confiG:!}');
10: 	define('PAGE_CONTROL_CONFIG_ROLE','(contr0l_confiG:!)');
11: 	
12: 	define('SITE_EMAIL_HOST','smtp.beget.com');
13: 	define('SITE_EMAIL_PORT','2525');
14: 	define('SITE_EMAIL_USERNAME','info@ibronevik.ru');
15: 	define('SITE_EMAIL_PASSWORD','worD_t0-send<msg>');
16: 	define('SITE_EMAIL_SMTPSECURE','tsl');
17: 	define('SITE_EMAIL_FROMNAME','Ibronevik');
18: 	define('SITE_EMAIL_INTERVAL_SEC',60);
19: 	define('SITE_EMAIL_INTERVAL_IP_SEC',600);
```

### `archive_17012026_1259/taxi/config/load_constant.php:6`
```php
1: <?php
2: 	define('ROOT_URL', trim('/taxi/'));
3: 	define('API_KEY', 'Api_keY_f0r_upD@te_Cache');
4: 	define('OPENSSL____ENCRYPT_KEY', '|<:* openssl____word *:>|');
5: 	define('CREATE_TOKEN_STRING', '/stringF0rTokenCre@ting/');
6: 	define('U_HASH_SECRET', '@_string_F0r_Token_Checking');
7: 	define('PAGE_EDIT_LANGS','_elvf');
8: 	define('PAGE_DEBUG','dEbug*_');
9: 	define('PAGE_CONTROL_CONFIG','{contr0l_confiG:!}');
10: 	define('PAGE_CONTROL_CONFIG_ROLE','(contr0l_confiG:!)');
11: 	
12: 	define('SITE_EMAIL_HOST','smtp.beget.com');
13: 	define('SITE_EMAIL_PORT','2525');
14: 	define('SITE_EMAIL_USERNAME','info@ibronevik.ru');
15: 	define('SITE_EMAIL_PASSWORD','worD_t0-send<msg>');
16: 	define('SITE_EMAIL_SMTPSECURE','tsl');
17: 	define('SITE_EMAIL_FROMNAME','Ibronevik');
18: 	define('SITE_EMAIL_INTERVAL_SEC',60);
19: 	define('SITE_EMAIL_INTERVAL_IP_SEC',600);
```

### `archive_17012026_1259/taxi/cache/data.php:2816`
```php
2808:       1 => 'Куда',
2809:       2 => 'To',
2810:     ),
2811:     'today' => 
2812:     array (
2813:       1 => 'Сегодня',
2814:       2 => 'Today ',
2815:     ),
2816:     'token' => 
2817:     array (
2818:       1 => 'Токен',
2819:       2 => 'Token',
2820:     ),
2821:     'tomato_tc' => 
2822:     array (
2823:       1 => 'Помидоры',
2824:       2 => 'Tomato',
2825:     ),
2826:     'tomorrow' => 
2827:     array (
2828:       1 => 'Завтра',
2829:       2 => 'Tomorrow',
2830:     ),
2831:     'took_passenger' => 
2832:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:2819`
```php
2811:     'today' => 
2812:     array (
2813:       1 => 'Сегодня',
2814:       2 => 'Today ',
2815:     ),
2816:     'token' => 
2817:     array (
2818:       1 => 'Токен',
2819:       2 => 'Token',
2820:     ),
2821:     'tomato_tc' => 
2822:     array (
2823:       1 => 'Помидоры',
2824:       2 => 'Tomato',
2825:     ),
2826:     'tomorrow' => 
2827:     array (
2828:       1 => 'Завтра',
2829:       2 => 'Tomorrow',
2830:     ),
2831:     'took_passenger' => 
2832:     array (
2833:       1 => 'Взял пассажира',
2834:       2 => 'Took Order',
2835:     ),
```

### `archive_17012026_1259/taxi/cache/data.php:3021`
```php
3013:       1 => 'Изменение заказа доступно только после его взятия',
3014:       2 => 'User is not performer error',
3015:     ),
3016:     'use_p' => 
3017:     array (
3018:       1 => 'Использовать',
3019:       2 => 'Use',
3020:     ),
3021:     'u_hash' => 
3022:     array (
3023:       1 => 'Хэш',
3024:       2 => 'U_hash',
3025:     ),
3026:     'very_expensive' => 
3027:     array (
3028:       1 => 'Слишком дорого',
3029:       2 => 'Very expensive',
3030:     ),
3031:     'view_p' => 
3032:     array (
3033:       1 => 'Посмотреть',
3034:     ),
3035:     'vote' => 
3036:     array (
3037:       1 => 'Голосовать',
```

### `archive_17012026_1259/taxi/cache/data.php:3024`
```php
3016:     'use_p' => 
3017:     array (
3018:       1 => 'Использовать',
3019:       2 => 'Use',
3020:     ),
3021:     'u_hash' => 
3022:     array (
3023:       1 => 'Хэш',
3024:       2 => 'U_hash',
3025:     ),
3026:     'very_expensive' => 
3027:     array (
3028:       1 => 'Слишком дорого',
3029:       2 => 'Very expensive',
3030:     ),
3031:     'view_p' => 
3032:     array (
3033:       1 => 'Посмотреть',
3034:     ),
3035:     'vote' => 
3036:     array (
3037:       1 => 'Голосовать',
3038:       2 => 'Vote',
3039:     ),
3040:     'voter' => 
```

### `archive_17012026_1259/taxi/cache/data.php:11176`
```php
11168:       'about_es' => '',
11169:       'value_type' => '3',
11170:       'some' => '0',
11171:       'visibility' => '12',
11172:       'editable' => '0',
11173:     ),
11174:     3 => 
11175:     array (
11176:       'var' => 'session_token',
11177:       'ru' => 'Токен сессионного доступа',
11178:       'en' => 'Session access token',
11179:       'ar' => NULL,
11180:       'fr' => NULL,
11181:       'es' => NULL,
11182:       'about_ru' => '',
11183:       'about_en' => '',
11184:       'about_ar' => '',
11185:       'about_fr' => '',
11186:       'about_es' => '',
11187:       'value_type' => '1',
11188:       'some' => '0',
11189:       'visibility' => '12',
11190:       'editable' => '0',
11191:     ),
11192:     4 => 
```

### `archive_17012026_1259/taxi/cache/data.php:11178`
```php
11170:       'some' => '0',
11171:       'visibility' => '12',
11172:       'editable' => '0',
11173:     ),
11174:     3 => 
11175:     array (
11176:       'var' => 'session_token',
11177:       'ru' => 'Токен сессионного доступа',
11178:       'en' => 'Session access token',
11179:       'ar' => NULL,
11180:       'fr' => NULL,
11181:       'es' => NULL,
11182:       'about_ru' => '',
11183:       'about_en' => '',
11184:       'about_ar' => '',
11185:       'about_fr' => '',
11186:       'about_es' => '',
11187:       'value_type' => '1',
11188:       'some' => '0',
11189:       'visibility' => '12',
11190:       'editable' => '0',
11191:     ),
11192:     4 => 
11193:     array (
11194:       'var' => 'Phone',
```

### `archive_17012026_1259/taxi/cache/data.php:11775`
```php
11767:       'es' => NULL,
11768:       'about_ru' => 'Значение в секундах',
11769:       'about_en' => '',
11770:       'about_ar' => '',
11771:       'about_fr' => '',
11772:       'about_es' => '',
11773:       'value' => '180',
11774:     ),
11775:     'token_interval_for_auth_hash' => 
11776:     array (
11777:       'group' => '4',
11778:       'ru' => 'Время получения токена через auth_hash',
11779:       'en' => NULL,
11780:       'ar' => NULL,
11781:       'fr' => NULL,
11782:       'es' => NULL,
11783:       'about_ru' => 'Интервал в секундах после авторизации, в течении которого возможно получить с помощью auth_hash',
11784:       'about_en' => '',
11785:       'about_ar' => '',
11786:       'about_fr' => '',
11787:       'about_es' => '',
11788:       'value' => '10',
11789:     ),
11790:     'limit_row_count' => 
11791:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:13860`
```php
13852:       'es' => NULL,
13853:       'about_ru' => 'Пустая строка означает, что доступно всем. Значение должно иметь вид идентификаторов ролей через запятую. Для водителя (роль 2) требуется check_state=2.',
13854:       'about_en' => '',
13855:       'about_ar' => '',
13856:       'about_fr' => '',
13857:       'about_es' => '',
13858:       'value' => '4',
13859:     ),
13860:     'session_token_duration' => 
13861:     array (
13862:       'group' => '1',
13863:       'ru' => 'Время ожидания неактивного токена сессионного доступа.',
13864:       'en' => NULL,
13865:       'ar' => NULL,
13866:       'fr' => NULL,
13867:       'es' => NULL,
13868:       'about_ru' => 'Значение в секундах.',
13869:       'about_en' => '',
13870:       'about_ar' => '',
13871:       'about_fr' => '',
13872:       'about_es' => '',
13873:       'value' => '86400',
13874:     ),
13875:     'placing_order_duration' => 
13876:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:15222`
```php
15214:       'roles_view' => 
15215:       array (
15216:         0 => '2',
15217:         1 => '4',
15218:       ),
15219:     ),
15220:     6 => 
15221:     array (
15222:       'var' => 'hold_s_token',
15223:       'ru' => 'Токен сессионного доступа юзера, запросившего холдирование',
15224:       'en' => 'Session access token of the user who requested the hold',
15225:       'ar' => NULL,
15226:       'fr' => NULL,
15227:       'es' => NULL,
15228:       'about_ru' => '',
15229:       'about_en' => '',
15230:       'about_ar' => '',
15231:       'about_fr' => '',
15232:       'about_es' => '',
15233:       'value_type' => '1',
15234:       'some' => '0',
15235:       'roles_edit' => 
15236:       array (
15237:         0 => '4',
15238:       ),
```

### `archive_17012026_1259/taxi/cache/data.php:15224`
```php
15216:         0 => '2',
15217:         1 => '4',
15218:       ),
15219:     ),
15220:     6 => 
15221:     array (
15222:       'var' => 'hold_s_token',
15223:       'ru' => 'Токен сессионного доступа юзера, запросившего холдирование',
15224:       'en' => 'Session access token of the user who requested the hold',
15225:       'ar' => NULL,
15226:       'fr' => NULL,
15227:       'es' => NULL,
15228:       'about_ru' => '',
15229:       'about_en' => '',
15230:       'about_ar' => '',
15231:       'about_fr' => '',
15232:       'about_es' => '',
15233:       'value_type' => '1',
15234:       'some' => '0',
15235:       'roles_edit' => 
15236:       array (
15237:         0 => '4',
15238:       ),
15239:       'roles_view' => 
15240:       array (
```

### `archive_17012026_1259/taxi/cache/data.php:22649`
```php
22641:       'es' => NULL,
22642:       'about_ru' => '',
22643:       'about_en' => '',
22644:       'about_ar' => '',
22645:       'about_fr' => '',
22646:       'about_es' => '',
22647:       'value' => 'u6oji0fwyaoubro',
22648:     ),
22649:     'dropbox_file_refresh_token' => 
22650:     array (
22651:       'group' => '11',
22652:       'ru' => 'Значение refresh_token для приложения управления файлами дропбокс',
22653:       'en' => NULL,
22654:       'ar' => NULL,
22655:       'fr' => NULL,
22656:       'es' => NULL,
22657:       'about_ru' => 'Постоянный токен, с помощью которого получается временный access_token, необходимый для доступа к апи дропбокса',
22658:       'about_en' => '',
22659:       'about_ar' => '',
22660:       'about_fr' => '',
22661:       'about_es' => '',
22662:       'value' => '8Q8jCFUTDhAAAAAAAAAAARh5LYZFAVUmHChI-PDoXiSAaNiNLJxrPpF7xfP0zJRu',
22663:     ),
22664:     'stripe_request_url' => 
22665:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:22652`
```php
22644:       'about_ar' => '',
22645:       'about_fr' => '',
22646:       'about_es' => '',
22647:       'value' => 'u6oji0fwyaoubro',
22648:     ),
22649:     'dropbox_file_refresh_token' => 
22650:     array (
22651:       'group' => '11',
22652:       'ru' => 'Значение refresh_token для приложения управления файлами дропбокс',
22653:       'en' => NULL,
22654:       'ar' => NULL,
22655:       'fr' => NULL,
22656:       'es' => NULL,
22657:       'about_ru' => 'Постоянный токен, с помощью которого получается временный access_token, необходимый для доступа к апи дропбокса',
22658:       'about_en' => '',
22659:       'about_ar' => '',
22660:       'about_fr' => '',
22661:       'about_es' => '',
22662:       'value' => '8Q8jCFUTDhAAAAAAAAAAARh5LYZFAVUmHChI-PDoXiSAaNiNLJxrPpF7xfP0zJRu',
22663:     ),
22664:     'stripe_request_url' => 
22665:     array (
22666:       'group' => '14',
22667:       'ru' => 'Ссылка на сервер платежей страйп',
22668:       'en' => NULL,
```

### `archive_17012026_1259/taxi/cache/data.php:22657`
```php
22649:     'dropbox_file_refresh_token' => 
22650:     array (
22651:       'group' => '11',
22652:       'ru' => 'Значение refresh_token для приложения управления файлами дропбокс',
22653:       'en' => NULL,
22654:       'ar' => NULL,
22655:       'fr' => NULL,
22656:       'es' => NULL,
22657:       'about_ru' => 'Постоянный токен, с помощью которого получается временный access_token, необходимый для доступа к апи дропбокса',
22658:       'about_en' => '',
22659:       'about_ar' => '',
22660:       'about_fr' => '',
22661:       'about_es' => '',
22662:       'value' => '8Q8jCFUTDhAAAAAAAAAAARh5LYZFAVUmHChI-PDoXiSAaNiNLJxrPpF7xfP0zJRu',
22663:     ),
22664:     'stripe_request_url' => 
22665:     array (
22666:       'group' => '14',
22667:       'ru' => 'Ссылка на сервер платежей страйп',
22668:       'en' => NULL,
22669:       'ar' => NULL,
22670:       'fr' => NULL,
22671:       'es' => NULL,
22672:       'about_ru' => '',
22673:       'about_en' => '',
```

### `archive_17012026_1259/taxi/cache/data.php:22844`
```php
22836:       'es' => NULL,
22837:       'about_ru' => '',
22838:       'about_en' => '',
22839:       'about_ar' => '',
22840:       'about_fr' => '',
22841:       'about_es' => '',
22842:       'value' => 'https://eitfa.ru/cgi/tg_app.py',
22843:     ),
22844:     'telegram_token' => 
22845:     array (
22846:       'group' => '17',
22847:       'ru' => 'Токен для доступа к серверу сервиса управления телеграмом',
22848:       'en' => 'Token for access to the telegram management service server',
22849:       'ar' => NULL,
22850:       'fr' => NULL,
22851:       'es' => NULL,
22852:       'about_ru' => '',
22853:       'about_en' => '',
22854:       'about_ar' => '',
22855:       'about_fr' => '',
22856:       'about_es' => '',
22857:       'value' => '',
22858:     ),
22859:     'stripe_webhook_signing_secret' => 
22860:     array (
```

### `archive_17012026_1259/taxi/cache/data.php:22848`
```php
22840:       'about_fr' => '',
22841:       'about_es' => '',
22842:       'value' => 'https://eitfa.ru/cgi/tg_app.py',
22843:     ),
22844:     'telegram_token' => 
22845:     array (
22846:       'group' => '17',
22847:       'ru' => 'Токен для доступа к серверу сервиса управления телеграмом',
22848:       'en' => 'Token for access to the telegram management service server',
22849:       'ar' => NULL,
22850:       'fr' => NULL,
22851:       'es' => NULL,
22852:       'about_ru' => '',
22853:       'about_en' => '',
22854:       'about_ar' => '',
22855:       'about_fr' => '',
22856:       'about_es' => '',
22857:       'value' => '',
22858:     ),
22859:     'stripe_webhook_signing_secret' => 
22860:     array (
22861:       'group' => '14',
22862:       'ru' => 'Секретный ключ страйпа для вебхука',
22863:       'en' => NULL,
22864:       'ar' => NULL,
```

### `archive_17012026_1259/taxi/controllers/c_google.php:24`
```php
16: $client = new Google_Client();
17: $client->setClientId($clientID);
18: $client->setClientSecret($clientSecret);
19: $client->setRedirectUri($redirectUri);
20: $client->addScope("email");
21: $client->addScope("profile");
22: 
23: if (isset($_GET['code'])) {
24: 	$token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
25: 	$client->setAccessToken($token['access_token']);
26: 
27: 	$google_oauth = new Google_Service_Oauth2($client);
28: 	$google_account_info = $google_oauth->userinfo->get();
29: 	$email =  $google_account_info->email;
30: 	$name =  $google_account_info->name;
31:  
32: 	$id = get_id_user("",$email,"");
33: 	if (is_array($id))
34: 	{
35: 		if ($id['error'] == 'user not found')
36: 		{
37: 			redirect("$taxiAppUrl?u_email=".urlencode($email)."&u_name=".urlencode($name)); 
38: 		}
39: 		else
40: 		{
```

### `archive_17012026_1259/taxi/controllers/c_google.php:25`
```php
17: $client->setClientId($clientID);
18: $client->setClientSecret($clientSecret);
19: $client->setRedirectUri($redirectUri);
20: $client->addScope("email");
21: $client->addScope("profile");
22: 
23: if (isset($_GET['code'])) {
24: 	$token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
25: 	$client->setAccessToken($token['access_token']);
26: 
27: 	$google_oauth = new Google_Service_Oauth2($client);
28: 	$google_account_info = $google_oauth->userinfo->get();
29: 	$email =  $google_account_info->email;
30: 	$name =  $google_account_info->name;
31:  
32: 	$id = get_id_user("",$email,"");
33: 	if (is_array($id))
34: 	{
35: 		if ($id['error'] == 'user not found')
36: 		{
37: 			redirect("$taxiAppUrl?u_email=".urlencode($email)."&u_name=".urlencode($name)); 
38: 		}
39: 		else
40: 		{
41: 		    if (substr($id['error'],0,6) == 'trying')
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2`
```php
1: <?php
2: check_auth_user();
3: if (!CONFIG_FROM_PATH || isset($_GET['route']))
4: {
5: 	redirect(url(strlen($_SERVER['QUERY_STRING']) > 0 ? '?' . $_SERVER['QUERY_STRING'] : '',CONFIG_URL,0));	
6: }
7: ?>
8: <!DOCTYPE html>
9: <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7" lang="ru"> <![endif]-->  
10: <!--[if IE 7]><html class="no-js lt-ie9 lt-ie8" lang="ru"> <![endif]-->  
11: <!--[if IE 8]><html class="no-js lt-ie9 ie8" lang="ru"> <![endif]-->  
12: <!--[if gt IE 8]><!--><html class="no-js" lang="ru"> <!--<![endif]--> 
13: <head>
14:     <meta charset="utf-8">
15:     <title>Формы для теста апи<?php if (!empty($_SESSION[UID])) {
16: 	echo ' (user:' . $_SESSION[UID] . ',role:' . $_SESSION['id_role'] . ')';} ?></title>
17:     <meta http-equiv="x-rim-auto-match" content="none">
18:     <meta http-equiv="X-UA-Compatible" content="IE=edge">
```

### `archive_17012026_1259/taxi/controllers/c_index.php:187`
```php
179: 				busy user data: список из double phone, double email, double tg в зависимости от их проверки
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
```

### `archive_17012026_1259/taxi/controllers/c_index.php:188`
```php
180: 			Для пользователя автоматически создается реферальный код, равный "uid" плюс идентификатор пользователя.
181: 			
182: 			Ответ сервера:
183: 			{'code':'200','status':'success','data':{
184: 										'u_id':				"идентификатор пользователя",
185: 										'email status': 	true или false					если указан емейл и не указан password
186: 										'string':			"пароль"						если не указан емейл и не указан password
187: 										'token':			"токен",						если определен st
188: 										'u_hash':			"проверочный хеш"				если определен st
189: 									}
190: 			}
191: 
192: 	https://ibronevik.ru/taxi/api/v1/auth/							POST
193: 		Авторизация
194: 			Доступна только для неавторизованного пользователя.
195: 			Параметры запроса:
196: 				login			телефон или емейл в зависимости от значения type
197: 				password
198: 				type			"phone" или "e-mail" или 'whatsapp' или 'tg' или 'wa'
199: 
200: 			Если type='whatsapp' и password пустой, то
201: 				на ватцап телефона u_phone, указанному в login, высылается четырехзначный код.
202: 				Ответ сервера:{'code':'200','status':'success',{'data':'code sent'}}
203: 			Если type='whatsapp' и password указан, то	
204: 				код, указанный в password, сравнивается с кодом для пользователя телефона, указанного в login,
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2414`
```php
2406: 			<form class="complex" action="api/v1/location" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2415`
```php
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
2412: 		</fieldset>
2413: 		<pre>
2414: 	https://ibronevik.ru/taxi/api/v1/token									GET
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2423`
```php
2415: 	https://ibronevik.ru/taxi/api/v1/token/authorized						GET
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
2437: 										"auth":			"число активных банов на авторизацию",
2438: 										"order":		"число активных банов на создание или получения поездки",
2439: 										"blog_topic":	"число активных банов на создание темы в блоге",
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2424`
```php
2416: 		 Получение токена и проверочного хеша для авторизированного пользователя.
2417: 			Доступно только для авторизованного пользователя.
2418: 			Возможно подтверждение авторизации, отправляя POST параметр auth_hash, но только в течении 10 секунд после авторизации.
2419: 			Ответ сервера:
2420: 			Ответ сервера:
2421: 			{'code':'200','status':'success',
2422: 				"data":{
2423: 					"token":			"токен",
2424: 					"u_hash":			"проверочный хеш"
2425: 					}
2426: 				},
2427: 				"auth_user":{
2428: 					"u_id":				"идентификатор пользователя",
2429: 					"u_name":			"имя пользователя",
2430: 					"u_family":			"фамилия пользователя",
2431: 					"u_middle":			"отчество пользователя",
2432: 					"u_email":			"емейл пользователя",
2433: 					"u_phone":			"телефон пользователя",
2434: 					"u_role":			"идентификатор роли пользователя",					data.user_roles
2435: 					"u_check_state":	"идентификатор верификации пользователя",			data.user_check_states
2436: 					"u_ban":			{
2437: 										"auth":			"число активных банов на авторизацию",
2438: 										"order":		"число активных банов на создание или получения поездки",
2439: 										"blog_topic":	"число активных банов на создание темы в блоге",
2440: 										"blog_post":	"число активных банов на создание сообщения в чужой теме"
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2451`
```php
2443: 					"u_photo":			"ссылка на фото",
2444: 					"u_birthday":		"день рождения пользователя в виде год-месяц-день",
2445: 					"u_lang":			"идентификатор языка, выбранного пользователем",	data.langs
2446: 					"u_currency":		"iso4217 код валюты, выбранной пользователем",		data.currencies
2447: 					"u_gps_software":	"идентификатор навигации, выбранной пользователем	data.gps_softwares
2448: 				}
2449: 			}</pre>
2450: 		<fieldset class="form"><legend title="Значения токена и проверояного хеша авторизованного пользователя.">Получение токена и проверояного хеша</legend>
2451: 			<form action="api/v1/token" method="GET" enctype="application/x-www-form-urlencoded;charset=UTF-8" data-method="get" target="_blank">
2452: 				<label class="no_border exclude only_post_request"><span>хеш авторизации</span><input data-name="auth_hash" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>
2453: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2454: 			</form>
2455: 		</fieldset>
2456: 		<pre>
2457: 	https://ibronevik.ru/taxi/api/v1/remind/								POST
2458: 		Восстановление пароля.
2459: 			Параметры запроса:
2460: 				u_email				емейл
2461: 				u_phone				телефон
2462: 				u_tg				идентификатор телеграм
2463: 				u_wa				идентификатор ватсап
2464: 			Ответ сервера:
2465: 			{'code':'200','status':'success'}</pre>
2466: 		<fieldset class="form"><legend title="Восстанавливает пароль: ищет пользователя по указанному емейлу, генерирует новый пароль и отправляет его на почту пользателя.">Восстановление пароля</legend>
2467: 			<form action="api/v1/remind/" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
```

### `archive_17012026_1259/taxi/controllers/c_index.php:3213`
```php
3205: 				<label class="no_border exclude"><span>значение приватности через запятую (-1,0,1)</span><input data-name="private" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3206: 				<label class="no_border exclude"><span>значение удаления через запятую (0,1)</span><input data-name="deleted" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
3207: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3208: 			</form>
3209: 		</fieldset>
3210: 		
3211: 		<pre>	
3212: 	https://ibronevik.ru/taxi/api/v1/dropbox/update							POST
3213: 		Обновление refresh_token для дропбокса.
3214: 			Доступно для авторизованного и неавторизованного пользователя.
3215: 			Код получается на странице:
3216: 				<?php
3217: 				if (!empty($_SESSION[UID]) && taxi::$id_role == 4 && !empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']))
3218: 				{			
3219: 					$client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
3220: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id=$client_id&token_access_type=offline&response_type=code";
3221: 					$code_url = "<a href=\"$code_url\" target=\"_blank\">$code_url</a>";
3222: 				}
3223: 				else
3224: 				{
3225: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id={app key}&token_access_type=offline&response_type=code";
3226: 				}
3227: 				echo $code_url;
3228: 
3229: 				?>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:3220`
```php
3212: 	https://ibronevik.ru/taxi/api/v1/dropbox/update							POST
3213: 		Обновление refresh_token для дропбокса.
3214: 			Доступно для авторизованного и неавторизованного пользователя.
3215: 			Код получается на странице:
3216: 				<?php
3217: 				if (!empty($_SESSION[UID]) && taxi::$id_role == 4 && !empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']))
3218: 				{			
3219: 					$client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
3220: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id=$client_id&token_access_type=offline&response_type=code";
3221: 					$code_url = "<a href=\"$code_url\" target=\"_blank\">$code_url</a>";
3222: 				}
3223: 				else
3224: 				{
3225: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id={app key}&token_access_type=offline&response_type=code";
3226: 				}
3227: 				echo $code_url;
3228: 
3229: 				?>
3230: 			
3231: 			Ответ сервера:
3232: 			{'code':'200','status':'success','data:{'refresh_token':"постоянный токен"}}
3233: 		</pre>
3234: 		<fieldset class="form"><legend title="Доступна для авторизованного и неавторизованного пользователя. Необходимо указать hash.">Обновление refresh_token для дропбокса</legend>
3235: 			<form action="api/v1/dropbox/update" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3236: 				<label><span>код</span><input name="code" type="text"></label>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:3225`
```php
3217: 				if (!empty($_SESSION[UID]) && taxi::$id_role == 4 && !empty(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']))
3218: 				{			
3219: 					$client_id = trim(taxi::$data_private['site_constants']['dropbox_file_client_id']['value']);
3220: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id=$client_id&token_access_type=offline&response_type=code";
3221: 					$code_url = "<a href=\"$code_url\" target=\"_blank\">$code_url</a>";
3222: 				}
3223: 				else
3224: 				{
3225: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id={app key}&token_access_type=offline&response_type=code";
3226: 				}
3227: 				echo $code_url;
3228: 
3229: 				?>
3230: 			
3231: 			Ответ сервера:
3232: 			{'code':'200','status':'success','data:{'refresh_token':"постоянный токен"}}
3233: 		</pre>
3234: 		<fieldset class="form"><legend title="Доступна для авторизованного и неавторизованного пользователя. Необходимо указать hash.">Обновление refresh_token для дропбокса</legend>
3235: 			<form action="api/v1/dropbox/update" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3236: 				<label><span>код</span><input name="code" type="text"></label>
3237: 				<label><span>hash</span><input name="hash" type="text"></label>
3238: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3239: 			</form>
3240: 		</fieldset>
3241: 		<pre>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:3232`
```php
3224: 				{
3225: 					$code_url = "https://www.dropbox.com/oauth2/authorize?client_id={app key}&token_access_type=offline&response_type=code";
3226: 				}
3227: 				echo $code_url;
3228: 
3229: 				?>
3230: 			
3231: 			Ответ сервера:
3232: 			{'code':'200','status':'success','data:{'refresh_token':"постоянный токен"}}
3233: 		</pre>
3234: 		<fieldset class="form"><legend title="Доступна для авторизованного и неавторизованного пользователя. Необходимо указать hash.">Обновление refresh_token для дропбокса</legend>
3235: 			<form action="api/v1/dropbox/update" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3236: 				<label><span>код</span><input name="code" type="text"></label>
3237: 				<label><span>hash</span><input name="hash" type="text"></label>
3238: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3239: 			</form>
3240: 		</fieldset>
3241: 		<pre>
3242: 	https://ibronevik.ru/taxi/api/v1/cart									GET
3243: 		Корзина пользователя.	
3244: 			Доступно только для авторизованного пользователя.
3245: 			GET или POST параметр filter:
3246: 				all			выводит все, доступен админу
3247: 				trip		все для билетов продавца, доступен продавцу и админу
3248: 			Ответ сервера:
```

### `archive_17012026_1259/taxi/controllers/c_index.php:3234`
```php
3226: 				}
3227: 				echo $code_url;
3228: 
3229: 				?>
3230: 			
3231: 			Ответ сервера:
3232: 			{'code':'200','status':'success','data:{'refresh_token':"постоянный токен"}}
3233: 		</pre>
3234: 		<fieldset class="form"><legend title="Доступна для авторизованного и неавторизованного пользователя. Необходимо указать hash.">Обновление refresh_token для дропбокса</legend>
3235: 			<form action="api/v1/dropbox/update" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
3236: 				<label><span>код</span><input name="code" type="text"></label>
3237: 				<label><span>hash</span><input name="hash" type="text"></label>
3238: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
3239: 			</form>
3240: 		</fieldset>
3241: 		<pre>
3242: 	https://ibronevik.ru/taxi/api/v1/cart									GET
3243: 		Корзина пользователя.	
3244: 			Доступно только для авторизованного пользователя.
3245: 			GET или POST параметр filter:
3246: 				all			выводит все, доступен админу
3247: 				trip		все для билетов продавца, доступен продавцу и админу
3248: 			Ответ сервера:
3249: 			{'code':'200','status':'success',
3250: 			"data":{		
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4078`
```php
4070: 												{"api_header":["header_for_url","default_value_for_url"]}
4071: 											Значение ключа или нулевого элемента
4072: 												0		считается не указанным, загловки не меняются
4073: 												'0'		считается указанным, загловок меняется на '0'
4074: 												true	считается указанным, загловок меняется на '1'
4075: 												false	считается не указанным, загловки не меняются
4076: 					post_json				массив параметров с их значениями для запроса по url
4077: 											например:
4078: 												{"token":"long_string"}
4079: 					post_json_export		массив экспортируемых параметров, то есть:
4080: 												те параметры, которые будут браться из вызова апи
4081: 												и передаваться при запросе по ссылке url.
4082: 											Указанные параметры из апи передаются в url без изменений
4083: 											(при отсутствии не добавляются):
4084: 												{"api_request":null}
4085: 												{"api_request":""}
4086: 												{"api_request":[]}
4087: 												{"api_request":[null]}
4088: 												{"api_request":[""]}
4089: 												{"api_request":[[]]}
4090: 												{"api_request":{"3":"value"}}
4091: 												{"api_request":[null,null]}
4092: 												{"api_request":["",null]}
4093: 												{"api_request":[[],null]}
4094: 												{"api_request:{"2":null,"3":"value"}}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4183`
```php
4175: 							"co_class":		"идентификатор типа контактных данных", 				data.contact_classes
4176: 							"cid":			"идентификатор контакта в родной среде",
4177: 							"link":			"уникальное имя ссылка контакта в родной среде",
4178: 							"is_bot":		"0 или 1, бот ли это",
4179: 							"active":		"0 или 1",
4180: 							"number"		"телефон контакта в родной среде",			не админу доступен только для o_type=1
4181: 							"key1"			"ключ1 для доступа к контакту",				не админу доступен только для o_type=1
4182: 							"key2"			"ключ2 для доступа к контакту",				не админу доступен только для o_type=1
4183: 							"token"			"токен для доступа к контакту",				не админу доступен только для o_type=1
4184: 							"hash"			"хеш для доступа к контакту",				не админу доступен только для o_type=1
4185: 							"secret"		"секретное слово для доступа к контакту",	не админу доступен только для o_type=1
4186: 							"host"			"хост для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4187: 							"port"			"порт для доступа, как вариант для емейла",	не админу доступен только для o_type=1
4188: 							"login"			"логин для доступа",						не админу доступен только для o_type=1
4189: 							"password"		"пароль для доступа",						не админу доступен только для o_type=1
4190: 							"smtpsecure"	"протокол шифрования для доступа, 			не админу доступен только для o_type=1
4191: 											как вариант для емейла"
4192: 							"fromname"		"имя отправителя для доступа 				не админу доступен только для o_type=1
4193: 											как вариант для емейла"			
4194: 							<?php
4195: 								$sql_name = $sql_description = array();			
4196: 								foreach (taxi::$data['langs'] as $lang)
4197: 								{
4198: 									$sql_name[] = "\"{$lang['iso']}\"";
4199: 									$sql_description[] = "\"about_{$lang['iso']}\"";
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4257`
```php
4249: 										"o_type":		идентификатор типа владельца контакта 		data.owner_types
4250: 										"co_class":		идентификатор типа контактных данных 		data.contact_classes необходимо
4251: 										"number"		телефон контакта в родной среде
4252: 										"cid":			идентификатор контакта в родной среде
4253: 										"link":			уникальное имя ссылка контакта в родной среде
4254: 										"is_bot":		0 или 1, бот ли это
4255: 										"key1"			ключ1 для доступа к контакту
4256: 										"key2"			ключ2 для доступа к контакту
4257: 										"token"			токен для доступа к контакту
4258: 										"hash"			хеш для доступа к контакту
4259: 										"secret"		секретное слово для доступа к контакту
4260: 										"host"			хост для доступа, как вариант для емейла
4261: 										"port"			порт для доступа, как вариант для емейла
4262: 										"login"			логин для доступа
4263: 										"password"		пароль для доступа
4264: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4265: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4266: 										"active":		0 или 1
4267: 									})
4268: 				Админ может создавать любой контакт.
4269: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4270: 				Без указания o_type назначается 1.
4271: 				Если o_type указан и не равен 1, то необходимо указывать owner.
4272: 				Если же o_type не указан или равен 1, то без указания owner=авторизованный_u_id.
4273: 				Ответ сервера:
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4292`
```php
4284: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4285: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4286: 				<label class="json_key"><span>телефон контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="number" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4287: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4288: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4289: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4290: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4291: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4292: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4293: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4294: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4295: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4296: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4297: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4298: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4299: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4300: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4301: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4302: 				<?php
4303: 					$sql_name = $sql_description = array();			
4304: 					foreach (taxi::$data['langs'] as $lang)
4305: 					{
4306: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4307: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4308: 					}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4333`
```php
4325: 										"o_type":		идентификатор типа владельца контакта 		data.owner_types		только админ
4326: 										"co_class":		идентификатор типа контактных данных 		data.contact_classes
4327: 										"number"		телефон контакта в родной среде
4328: 										"cid":			идентификатор контакта в родной среде
4329: 										"link":			уникальное имя ссылка контакта в родной среде
4330: 										"is_bot":		0 или 1, бот ли это
4331: 										"key1"			ключ1 для доступа к контакту
4332: 										"key2"			ключ2 для доступа к контакту
4333: 										"token"			токен для доступа к контакту
4334: 										"hash"			хеш для доступа к контакту
4335: 										"secret"		секретное слово для доступа к контакту
4336: 										"host"			хост для доступа, как вариант для емейла
4337: 										"port"			порт для доступа, как вариант для емейла
4338: 										"login"			логин для доступа
4339: 										"password"		пароль для доступа
4340: 										"smtpsecure"	протокол шифрования для доступа, как вариант для емейла
4341: 										"fromname"		имя отправителя для доступа, как вариант для емейла	
4342: 										"active":		0 или 1
4343: 									})
4344: 				Админ может редактировать любой контакт.
4345: 				Остальные только с o_type=1 и owner=авторизованный_u_id.
4346: 				Ответ сервера:
4347: 				{'code':'200','status':'success'
4348: 					"data":{
4349: 						"affected_fields":		[..],			список валидных ключей
```

### `archive_17012026_1259/taxi/controllers/c_index.php:4365`
```php
4357: 				<label class="json_key"><span>идентификатор типа владельца контакта (data.owner_types)</span><input data-name="o_type" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4358: 				<label class="json_key"><span>идентификатор типа контактных данных (data.contact_classes)</span><input data-name="co_class" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4359: 				<label class="json_key"><span>телефон контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="number" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4360: 				<label class="json_key"><span>идентификатор контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="cid" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4361: 				<label class="json_key"><span>уникальное имя ссылка контакта в родной среде <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="link" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4362: 				<label class="json_key"><span>0 или 1, бот ли это</span><input data-name="is_bot" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4363: 				<label class="json_key"><span>ключ1 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key1" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4364: 				<label class="json_key"><span>ключ2 для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="key2" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4365: 				<label class="json_key"><span>токен для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="token" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4366: 				<label class="json_key"><span>хеш для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="hash" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4367: 				<label class="json_key"><span>секретное слово для доступа к контакту <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="secret" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4368: 				<label class="json_key"><span>хост для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="host" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4369: 				<label class="json_key"><span>порт для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="port" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4370: 				<label class="json_key"><span>логин для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="login" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4371: 				<label class="json_key"><span>пароль для доступа <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="password" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4372: 				<label class="json_key"><span>протокол шифрования для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="smtpsecure" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4373: 				<label class="json_key"><span>имя отправителя для доступа, как вариант для емейла <a onclick="set_null(this)" href="javascript:void 0;">null</a></span><input data-name="fromname" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4374: 				<label class="json_key"><span>0 или 1, активность</span><input data-name="active" type="text"> <a onclick="exclude_input(this)" href="javascript:void 0;">не отправлять</a></label>
4375: 				<?php
4376: 					$sql_name = $sql_description = array();			
4377: 					foreach (taxi::$data['langs'] as $lang)
4378: 					{
4379: 						$sql_name[] = "<label class=\"json_key\"><span>{$lang['iso']} <a onclick=\"set_null(this)\" href=\"javascript:void 0;\">null</a></span><input data-name=\"{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4380: 						$sql_description[] = "<label class=\"json_key\"><span>about_{$lang['iso']}</span><input data-name=\"about_{$lang['iso']}\" type=\"text\"> <a onclick=\"exclude_input(this)\" href=\"javascript:void 0;\">не отправлять</a></label>";
4381: 					}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5555`
```php
5547: 		</pre>
5548: 		<pre>
5549: 	Обновление (февраль 2020)
5550: 		https://ibronevik.ru/taxi/api/v1/user/authorized				GET
5551: 			Выводятся данные об авторизированном пользователе
5552: 
5553: 		Вместо кук авторизации теперь можно использовать токен для вызова методов апи.
5554: 		Для этого к каждому запросу присоединять передаваемые методом POST два параметра:<?php 
5555: 				$auth_token = $auth_u_hash = 'требуется авторизация для получения значения';
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5557`
```php
5549: 	Обновление (февраль 2020)
5550: 		https://ibronevik.ru/taxi/api/v1/user/authorized				GET
5551: 			Выводятся данные об авторизированном пользователе
5552: 
5553: 		Вместо кук авторизации теперь можно использовать токен для вызова методов апи.
5554: 		Для этого к каждому запросу присоединять передаваемые методом POST два параметра:<?php 
5555: 				$auth_token = $auth_u_hash = 'требуется авторизация для получения значения';
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5559`
```php
5551: 			Выводятся данные об авторизированном пользователе
5552: 
5553: 		Вместо кук авторизации теперь можно использовать токен для вызова методов апи.
5554: 		Для этого к каждому запросу присоединять передаваемые методом POST два параметра:<?php 
5555: 				$auth_token = $auth_u_hash = 'требуется авторизация для получения значения';
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5561`
```php
5553: 		Вместо кук авторизации теперь можно использовать токен для вызова методов апи.
5554: 		Для этого к каждому запросу присоединять передаваемые методом POST два параметра:<?php 
5555: 				$auth_token = $auth_u_hash = 'требуется авторизация для получения значения';
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5562`
```php
5554: 		Для этого к каждому запросу присоединять передаваемые методом POST два параметра:<?php 
5555: 				$auth_token = $auth_u_hash = 'требуется авторизация для получения значения';
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5564`
```php
5556: 
5557: 				if (!empty($_SESSION[UID]) && empty($_SESSION['token_auth']))
5558: 				{
5559: 					if (empty($_SESSION['token_auth']))
5560: 					{
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5569`
```php
5561: 						$auth_token = get_token($_SESSION[UID]);
5562: 						if (gettype($auth_token) !== 'string')
5563: 						{
5564: 							$auth_token = 'ошибка при получении: ' . $auth_token;
5565: 
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5574`
```php
5566: 						}
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5575`
```php
5567: 						else
5568: 						{
5569: 							$auth_u_hash = get_u_hash($auth_token,$_SESSION[UID]);
5570: 						}
5571: 					}
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5580`
```php
5572: 					else
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5581`
```php
5573: 					{
5574: 						$auth_token = $_POST['token'];
5575: 						$auth_u_hash = $_POST['u_hash'];
5576: 					}
5577: 				}
5578: 
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5587`
```php
5579: 			?> 
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
5603: 			cache version
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5588`
```php
5580: 			token	=	<input id="auth_token" type="text" style="width:294px;" value="<?php echo htmlentities($auth_token); ?>">
5581: 			u_hash	=	<input id="auth_u_hash" type="text" style="width:294px;" value="<?php echo htmlentities($auth_u_hash); ?>">
5582: 		После авторизации через форму на этой страницы значения этих параметров будут указаны.
5583: 
5584: 		Для вывода пользователей, машин и поездок списком надо добавлять к запросу GET или POST параметр:
5585: 			array_type	=	list
5586: 
5587: 		https://ibronevik.ru/taxi/api/v1/token							GET
5588: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5589: 			Добавлен метод получения токена.
5590: 			
5591: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5592: 			Параметр hash теперь требует метод POST для передачи на сервер.
5593: 			
5594: 		</pre>
5595: 		<pre>
5596: 	Обновление (май 2020)
5597: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
5598: 			Добавлена переменная version.
5599: 
5600: 		Для вывода версии файла data.js в запросе надо добавлять GET или POST параметр:
5601: 			cv
5602: 		Тогда в ответе сервера в виде массива будет содержаться ключ 
5603: 			cache version
5604: 		А значение, соответствующее этому ключу, будет версией.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5616`
```php
5608: 			Идентификатор класса машины	теперь не обязателен (в этом случае считается null).
5609: 				null означает, что класс машины любой.
5610: 			Для цели поезки теперь не обязательно указывать или адрес или координаты.
5611: 			Можно указывать дополнительные параметры b_options.
5612: 
5613: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}				POST
5614: 			Добавлено изменение максимального времени ожидания set_waiting_time.
5615: 			
5616: 		https://ibronevik.ru/taxi/api/v1/token							GET
5617: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5618: 			Добавлено подтверждение авторизации через auth_hash параметр.
5619: 
5620: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
5621: 			Добавлен вывод auth_hash.
5622: 
5623: 		Приоритет методов авторизации:
5624: 			auth_token и auth_u_hash
5625: 			auth_hash
5626: 			куки сессии
5627: 
5628: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5629: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5630: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5631: 			Список отображаемых поездок теперь зависит от b_start_datetime+b_max_waiting.
5632: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5617`
```php
5609: 				null означает, что класс машины любой.
5610: 			Для цели поезки теперь не обязательно указывать или адрес или координаты.
5611: 			Можно указывать дополнительные параметры b_options.
5612: 
5613: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id}				POST
5614: 			Добавлено изменение максимального времени ожидания set_waiting_time.
5615: 			
5616: 		https://ibronevik.ru/taxi/api/v1/token							GET
5617: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5618: 			Добавлено подтверждение авторизации через auth_hash параметр.
5619: 
5620: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
5621: 			Добавлен вывод auth_hash.
5622: 
5623: 		Приоритет методов авторизации:
5624: 			auth_token и auth_u_hash
5625: 			auth_hash
5626: 			куки сессии
5627: 
5628: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5629: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5630: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5631: 			Список отображаемых поездок теперь зависит от b_start_datetime+b_max_waiting.
5632: 
5633: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
```

### `archive_17012026_1259/taxi/controllers/c_index.php:5624`
```php
5616: 		https://ibronevik.ru/taxi/api/v1/token							GET
5617: 		https://ibronevik.ru/taxi/api/v1/token/authorized				GET
5618: 			Добавлено подтверждение авторизации через auth_hash параметр.
5619: 
5620: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
5621: 			Добавлен вывод auth_hash.
5622: 
5623: 		Приоритет методов авторизации:
5624: 			auth_token и auth_u_hash
5625: 			auth_hash
5626: 			куки сессии
5627: 
5628: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5629: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5630: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5631: 			Список отображаемых поездок теперь зависит от b_start_datetime+b_max_waiting.
5632: 
5633: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET
5634: 			В данные о поездках добавлено b_max_waiting_list.
5635: 			
5636: 		https://ibronevik.ru/taxi/api/v1/drive							GET
5637: 		https://ibronevik.ru/taxi/api/v1/drive/now						GET
5638: 		https://ibronevik.ru/taxi/api/v1/drive/archive					GET
5639: 		https://ibronevik.ru/taxi/api/v1/drive/get/{b_id1,b_id5}		GET	
5640: 			Добавлена обратная сортировка по времени изменения поездки.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6066`
```php
6058: 		Разсширенный auth и полный auth может иметь вид:
6059: 			'error': 'описание ошибки'
6060: 		Если au=e, то к любому ответу сервера будет добавлятся расширенный 'auth_user'.
6061: 		Если au=f, то к любому ответу сервера будет добавлятся полный 'auth_user' с полным 'auth_user'.
6062: 
6063: 		В стандартный массив данный	'auth_user' добавлено:		
6064: 			u_lang
6065: 
6066: 		Для неправильных значений token или u_hash к ответу сервера добавляется auth_error.
6067: 
6068: 		https://ibronevik.ru/taxi/api/v1/user/{u_id}
6069: 		https://ibronevik.ru/taxi/api/v1/user
6070: 			Водитель с u_check_state=2 может редактировать: 
6071: 				"u_active"
6072: 
6073: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
6074: 			В вывод файлов добавлены массивы:
6075: 				"cargo_case_models":{...}
6076: 				"cargo_case_types":{...}
6077: 				"cargo_value_types":{...}				
6078: 				"units_of_length":{...},
6079: 				"units_of_mass":{...},
6080: 				"units_of_volume":{...},	
6081: 				"unit_sets":{...},
6082: 		
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6654`
```php
6646: 				};
6647: 				button_for_get_fields_par.click();
6648: 
6649: 			</script>
6650: 		</pre>	
6651: 		<pre>
6652: 	Обновление (июнь 2022)
6653: 		https://ibronevik.ru/taxi/api/v1/push/{p_id}/add				POST
6654: 			c POST параметрами u_id, "token=test token", "u_hash=test u_hash"
6655: 			Заглушка для метода присвоения пользователю идентификатора сервиса сообщений
6656: 		
6657: 		https://ibronevik.ru/taxi/api/v1/register/						POST
6658: 			Добавлен параметр:
6659: 				data.u_details	
6660: 		</pre>
6661: 		<pre>
6662: 	Обновление (ноябрь 2022)
6663: 		https://ibronevik.ru/taxi/api/v1/register/						POST
6664: 			Добавлен параметр:
6665: 				st
6666: 				
6667: 		Администратор может заходить как любой пользователь с помощью POST параметра:
6668: 				u_a_id			идентификатор пользователя
6669: 				u_a_email		емейл пользователя
6670: 				u_a_phone		телефон пользователя
```

### `archive_17012026_1259/taxi/controllers/c_index.php:6681`
```php
6673: 				u_a_email
6674: 				u_a_phone
6675: 				
6676: 		Таким образом на сервере приложения для авторизации через гугл предлагается следующая схема:
6677: 			Создается пользователь админ, допустим, "веб приложение такси".
6678: 			Пользователь "веб приложение такси" получает токен, который сохраняется на сервере приложения.
6679: 			При авторизации через гугл получаем емейл пользователя.
6680: 			На сервере приложения "веб приложение такси" отправляет запрос:
6681: 				POST, https://ibronevik.ru/taxi/api/v1/token/authorized с данными: u_a_email="полученный емейл"
6682: 			Из ответа получаем токен.
6683: 			Если ответ содержит ошибку:
6684: 				{'code' => '404', 'status' => 'error', 'message' => 'user not found'}
6685: 				то приложению пользователя сообщается, что требуется регистрация.
6686: 				Далее приложение пользователя регистрирует его
6687: 					POST, https://ibronevik.ru/taxi/api/v1/register/ с данными: u_email="полученный емейл"&st
6688: 				токен из ответа используется приложением пользователя.
6689: 		</pre>
6690: 		<pre>
6691: 	Обновление (январь 2023)
6692: 		https://ibronevik.ru/taxi/api/v1/auth/							POST
6693: 			Добавлен type:
6694: 				whatsapp
6695: 			
6696: 		https://ibronevik.ru/taxi/google/
6697: 			Добавлена страница редиректа для аторизации через гугл.
```

### `archive_17012026_1259/taxi/controllers/c_index.php:7830`
```php
7822: 			Добавлена обработка вывода script_templates.id.file
7823: 
7824: 		https://ibronevik.ru/taxi/api/v1/script/template/{script_id}				POST	
7825: 		https://ibronevik.ru/taxi/api/v1/script/template/{script_var}?is_var=1		POST
7826: 			Добавлен методы выполнения скрипта	
7827: 		</pre>
7828: 		<pre>
7829: 	Обновление 3 (ноябрь 2023)
7830: 		Поправлена проверка u_hash, когда вводится некорректное значение
7831: 
7832: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
7833: 			В выводе блока sс изменены массивы:
7834: 				"schedule":{...}
7835: 					добавлено:
7836: 						options
7837: 			В основном блоке изменены массивы:
7838: 				"langs":{...}
7839: 					добавлено:
7840: 						tr_code
7841: 	
7842: 		Для редактирования языковых слов добавлены:
7843: 			regions
7844: 			script_templates
7845: 			site_images
7846: 			sql_templates
```

### `archive_17012026_1259/taxi/controllers/c_index.php:9343`
```php
9335: 		https://ibronevik.ru/taxi/api/v1/drive										POST			
9336: 			Добавлено:
9337: 				b_pc
9338: 
9339: 		https://ibronevik.ru/taxi/api/v1/promocode/check/							GET
9340: 			Добавлен метод проверки промокода
9341: 
9342: 		Добавлен GET или POST параметр:
9343: 			s_token	
9344: 		</pre>
9345: 		<pre>
9346: 	Обновление (декабрь 2024)
9347: 		https://ibronevik.ru/taxi/api/v1/ticket/check/								GET
9348: 			Добавлено:
9349: 				t_id
9350: 				status
9351: 				b_payment_datetime
9352: 				b_state
9353: 				
9354: 		https://ibronevik.ru/taxi/api/v1/data/										GET
9355: 			Добавлено для nocache:
9356: 				top
9357: 				options
9358: 				time_zone
9359: 				price_time_function
```

### `archive_17012026_1259/taxi/controllers/c_index.php:9367`
```php
9359: 				price_time_function
9360: 				currency
9361: 				currency_priority
9362: 				fee
9363: 				tariff
9364: 				tariff_priority
9365: 				code_ean_base64
9366: 
9367: 		Добавлена правильная обработка неправильных коротких u_hash и auth_hash
9368: 	
9369: 		https://ibronevik.ru/taxi/api/v1/trip/get/{t_id}/ticket/edit/	POST			
9370: 			Для параметра pass добавлено логирование действий билетера
9371: 
9372: 		https://ibronevik.ru/taxi/api/v1/ticket/check/log					GET
9373: 			Добавлен метод вывода логов.
9374: 								
9375: 		https://ibronevik.ru/taxi/api/v1/cache/update					POST
9376: 			В основной блок добавлены массивы:
9377: 				"map_place_polygons":{...}
9378: 				"favorite_addresses":{...}
9379: 			Ключи для каждого объекта map_place_polygons:
9380: 				upper
9381: 				var
9382: 				coordinates
9383: 				<?php
```

### `archive_17012026_1259/taxi/controllers/c_index.php:10965`
```php
10957: 				document.getElementsByTagName('form').forEach(function(el){
10958: 					el.setAttribute("data-method",el.method);
10959: 					var label = '<label class="no_border"><span>Показывать версию data.js</span><input class="checkbox" name="cv" type="checkbox" value=""></label>';
10960: 					label += '<label class="no_border"><span>Включить дебаг</span><input class="checkbox" name="debug" type="checkbox" value=""></label>';
10961: 					label += '<label class="no_border exclude"><span>Показывать данные авторизованного пользователя</span><select data-name="au"><option value="" selected>Стандартный auth_user</option><option value="e">Расширенный auth_user</option><option value="f">Полный auth_user</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10962: 					if (el.classList.contains("type_select")) {
10963: 						label += '<label class="no_border exclude"><span>формат массива</span><input data-name="array_type" type="text" value="list"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10964: 					}
10965: 					label += '<label class="no_border exclude only_post_request"><span>токен</span><input data-name="token" type="text" value="' + auth_token.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10966: 					label += '<label class="no_border exclude only_post_request"><span>проверочный хеш</span><input data-name="u_hash" type="text" value="' + auth_u_hash.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10967: 					if (el.classList.contains("out_control")) {
10968: 						label += '<label class="no_border exclude"><span>номер первой выводимой записи (начиная с нуля)</span><input data-name="lo" type="text" value="0"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10969: 						label += '<label class="no_border exclude"><span>количество выводимых записей</span><input data-name="lc" type="text" value="5"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10970: 					}
10971: 					label += '<label class="no_border exclude"><span>идентификатор языка</span><input data-name="lang" type="text" value="1"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10972: 					label += '<label class="no_border exclude"><span>роль пользователя</span><select data-name="u_a_role"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option><option value="6">Билетер</option><option value="7">Билетер с расширенными полномочиями</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10973: 					label += '<label class="no_border exclude only_post_request"><span>идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_id" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10974: 					label += '<label class="no_border exclude only_post_request"><span>емейл пользователя, авторизация которого имитируется</span><input data-name="u_a_email" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10975: 					label += '<label class="no_border exclude only_post_request"><span>телефон пользователя, авторизация которого имитируется</span><input data-name="u_a_phone" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10976: 					label += '<label class="no_border exclude only_post_request"><span>телеграм идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_tg" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10977: 					label += '<label class="no_border exclude only_post_request"><span>ватсап идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_wa" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10978: 					label += '<label class="no_border exclude"><span>адрес веб приложения</span><input data-name="appUrl" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10979: 					label += '<label class="no_border exclude"><span>токен сессионного доступа</span><input data-name="s_token" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10980: 					el.insertAdjacentHTML("AfterBegin",label);
10981: 				});
```

### `archive_17012026_1259/taxi/controllers/c_index.php:10966`
```php
10958: 					el.setAttribute("data-method",el.method);
10959: 					var label = '<label class="no_border"><span>Показывать версию data.js</span><input class="checkbox" name="cv" type="checkbox" value=""></label>';
10960: 					label += '<label class="no_border"><span>Включить дебаг</span><input class="checkbox" name="debug" type="checkbox" value=""></label>';
10961: 					label += '<label class="no_border exclude"><span>Показывать данные авторизованного пользователя</span><select data-name="au"><option value="" selected>Стандартный auth_user</option><option value="e">Расширенный auth_user</option><option value="f">Полный auth_user</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10962: 					if (el.classList.contains("type_select")) {
10963: 						label += '<label class="no_border exclude"><span>формат массива</span><input data-name="array_type" type="text" value="list"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10964: 					}
10965: 					label += '<label class="no_border exclude only_post_request"><span>токен</span><input data-name="token" type="text" value="' + auth_token.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10966: 					label += '<label class="no_border exclude only_post_request"><span>проверочный хеш</span><input data-name="u_hash" type="text" value="' + auth_u_hash.value.replace(/"/g,"&quot;") + '"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10967: 					if (el.classList.contains("out_control")) {
10968: 						label += '<label class="no_border exclude"><span>номер первой выводимой записи (начиная с нуля)</span><input data-name="lo" type="text" value="0"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10969: 						label += '<label class="no_border exclude"><span>количество выводимых записей</span><input data-name="lc" type="text" value="5"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10970: 					}
10971: 					label += '<label class="no_border exclude"><span>идентификатор языка</span><input data-name="lang" type="text" value="1"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10972: 					label += '<label class="no_border exclude"><span>роль пользователя</span><select data-name="u_a_role"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option><option value="6">Билетер</option><option value="7">Билетер с расширенными полномочиями</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10973: 					label += '<label class="no_border exclude only_post_request"><span>идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_id" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10974: 					label += '<label class="no_border exclude only_post_request"><span>емейл пользователя, авторизация которого имитируется</span><input data-name="u_a_email" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10975: 					label += '<label class="no_border exclude only_post_request"><span>телефон пользователя, авторизация которого имитируется</span><input data-name="u_a_phone" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10976: 					label += '<label class="no_border exclude only_post_request"><span>телеграм идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_tg" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10977: 					label += '<label class="no_border exclude only_post_request"><span>ватсап идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_wa" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10978: 					label += '<label class="no_border exclude"><span>адрес веб приложения</span><input data-name="appUrl" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10979: 					label += '<label class="no_border exclude"><span>токен сессионного доступа</span><input data-name="s_token" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10980: 					el.insertAdjacentHTML("AfterBegin",label);
10981: 				});
10982: 
```

### `archive_17012026_1259/taxi/controllers/c_index.php:10979`
```php
10971: 					label += '<label class="no_border exclude"><span>идентификатор языка</span><input data-name="lang" type="text" value="1"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10972: 					label += '<label class="no_border exclude"><span>роль пользователя</span><select data-name="u_a_role"><option value="1" selected>Клиент</option><option value="2">Водитель</option><option value="3">Диспетчер</option><option value="4">Администратор</option><option value="5">Агент</option><option value="6">Билетер</option><option value="7">Билетер с расширенными полномочиями</option></select><a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10973: 					label += '<label class="no_border exclude only_post_request"><span>идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_id" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10974: 					label += '<label class="no_border exclude only_post_request"><span>емейл пользователя, авторизация которого имитируется</span><input data-name="u_a_email" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10975: 					label += '<label class="no_border exclude only_post_request"><span>телефон пользователя, авторизация которого имитируется</span><input data-name="u_a_phone" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10976: 					label += '<label class="no_border exclude only_post_request"><span>телеграм идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_tg" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10977: 					label += '<label class="no_border exclude only_post_request"><span>ватсап идентификатор пользователя, авторизация которого имитируется</span><input data-name="u_a_wa" type="text" value=""> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10978: 					label += '<label class="no_border exclude"><span>адрес веб приложения</span><input data-name="appUrl" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>';
10979: 					label += '<label class="no_border exclude"><span>токен сессионного доступа</span><input data-name="s_token" type="text"> <a onclick="exclude_input_simple(this,true)" href="javascript:void 0;">не отправлять</a></label>';
10980: 					el.insertAdjacentHTML("AfterBegin",label);
10981: 				});
10982: 
10983: 				(function(){
10984: 					var input = get_version.querySelector('input[type="checkbox"]');
10985: 					input.checked = true;
10986: 					var label = input.parentNode;
10987: 					label.style.display = "none";
10988: 				})();
10989: 			</script>
10990: 	</div>
10991: </body>
10992: </html>
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:22`
```php
14: 			$API->associativeArray = false;
15: 		}
16: 		$API->tuneLimit(isset($_REQUEST['lo'])?$_REQUEST['lo']:NULL,isset($_REQUEST['lc'])?$_REQUEST['lc']:NULL);
17: 
18: 		if (isset($_GET['par'][1]))
19: 		{
20: 			if ($_GET['par'][1] == 'register')
21: 			{
22: 				check_auth_user(); $API->id_role = taxi::$id_role;
23: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false);
24: 			}
25: 			elseif ($_GET['par'][1] == 'auth')
26: 			{
27: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
28: 
29: 			}
30: 			elseif ($_GET['par'][1] == 'logout')
31: 			{
32: 				$out = $API->logout();
33: 			}
34: 			elseif ($_GET['par'][1] == 'remind')
35: 			{
36: 				check_auth_user(); $API->id_role = taxi::$id_role;
37: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'');
38: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:36`
```php
28: 
29: 			}
30: 			elseif ($_GET['par'][1] == 'logout')
31: 			{
32: 				$out = $API->logout();
33: 			}
34: 			elseif ($_GET['par'][1] == 'remind')
35: 			{
36: 				check_auth_user(); $API->id_role = taxi::$id_role;
37: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'');
38: 			}
39: 			elseif ($_GET['par'][1] == 'newpass')
40: 			{
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
48: 				{
49: 					if (isset($_POST['data']))
50: 					{
51: 						/*file_put_contents('req.log',$_POST['data']);
52: 	                    file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:41`
```php
33: 			}
34: 			elseif ($_GET['par'][1] == 'remind')
35: 			{
36: 				check_auth_user(); $API->id_role = taxi::$id_role;
37: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'');
38: 			}
39: 			elseif ($_GET['par'][1] == 'newpass')
40: 			{
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
48: 				{
49: 					if (isset($_POST['data']))
50: 					{
51: 						/*file_put_contents('req.log',$_POST['data']);
52: 	                    file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
53: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
54: 						/*file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
55: 					}
56: 					else
57: 					{
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:46`
```php
38: 			}
39: 			elseif ($_GET['par'][1] == 'newpass')
40: 			{
41: 				check_auth_user(); $API->id_role = taxi::$id_role;
42: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
43: 			}
44: 			elseif ($_GET['par'][1] == 'user')
45: 			{
46: 				check_auth_user(); $API->id_role = taxi::$id_role;
47: 				if (empty($_GET['par'][3]))
48: 				{
49: 					if (isset($_POST['data']))
50: 					{
51: 						/*file_put_contents('req.log',$_POST['data']);
52: 	                    file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
53: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
54: 						/*file_put_contents('req.log',"\n".print_r($API->selectUser($_SESSION[UID]),true),FILE_APPEND);*/
55: 					}
56: 					else
57: 					{
58: 						$out = $API->selectUser(isset($_GET['par'][2])? trim($_GET['par'][2]):'');
59: 					}
60: 				}
61: 				elseif ($_GET['par'][3] == 'car')
62: 				{
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:104`
```php
96: 						{
97: 							$out = $API->selectReferralUrl(trim($_GET['par'][2]));
98: 						}
99: 					}
100: 				}
101: 			}
102: 			elseif ($_GET['par'][1] == 'car')
103: 			{
104: 				check_auth_user(); $API->id_role = taxi::$id_role;
105: 				if (empty($_GET['par'][3]))
106: 				{
107: 					if (isset($_POST['data']))
108: 					{
109: 						$out = $API->controlCar($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array());
110: 					}
111: 					else
112: 					{
113: 						$out = $API->selectCar(isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array());
114: 					}
115: 				}
116: 				elseif ($_GET['par'][3] == 'drive')
117: 				{
118: 					$out = $API->setCarUsed(trim($_GET['par'][2]));
119: 				}
120: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:128`
```php
120: 			}
121: 			elseif ($_GET['par'][1] == 'cache' 
122: 				&& !empty($_GET['par'][2]) && $_GET['par'][2] == 'update' && !(empty($_POST['hash'])))
123: 			{
124: 				$out = $API->updateCache(trim($_POST['hash']));
125: 			}
126: 			elseif ($_GET['par'][1] == 'drive')
127: 			{
128: 				check_auth_user(); $API->id_role = taxi::$id_role;
129: 				if (empty($_GET['par'][2]))
130: 				{
131: 					if (isset($_POST['data']))
132: 					{
133: 						$out = $API->createOrder($_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['payment_card'])?taxi::$data['payment_card']:array(),isset(taxi::$data['booking_comments'])?taxi::$data['booking_comments']:array(),isset(taxi::$data['services'])?taxi::$data['services']:array(),isset(taxi::$data['contact_classes'])?taxi::$data['contact_classes']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['unit_sets'])?taxi::$data['unit_sets']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array(),isset(taxi::$data_sc['schedule'])?taxi::$data_sc['schedule']:array());
134: 					}
135: 					else
136: 					{
137: 						$out = $API->selectActiveOrder(isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
138: 					}
139: 				}
140: 				else
141: 				{
142: 					if ($_GET['par'][2] == 'get')
143: 					{
144: 						if (isset($_POST['action']))
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:211`
```php
203: 					elseif ($_GET['par'][2] == 'archive')
204: 					{
205: 						$out = $API->selectArchiveOrder(isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
206: 					}
207: 				}
208: 			}
209: 			elseif ($_GET['par'][1] == 'trip')
210: 			{
211: 				check_auth_user(); $API->id_role = taxi::$id_role;
212: 				if (empty($_GET['par'][2]))
213: 				{
214: 					if (isset($_POST['data']))
215: 					{
216: 						$out = $API->createTrip(empty($_POST['u_id'])?'':trim($_POST['u_id']),$_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),DEFAULT_PROFILE == 'stadium' ? array(taxi::$data_stt,taxi::$data_sc) : array());
217: 					}
218: 					else
219: 					{
220: 						$out = $API->selectTrip(NULL,'active',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
221: 					}
222: 				}
223: 				
224: 				else
225: 				{
226: 					if ($_GET['par'][2] == 'get')
227: 					{
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:300`
```php
292: 						$out = $API->selectTrip(NULL,'processing',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
293: 					}
294: 				}
295: 			}
296: 			elseif ($_GET['par'][1] == 'location')
297: 			{
298: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
299: 				{
300: 					check_auth_user(); $API->id_role = taxi::$id_role;
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:304`
```php
296: 			elseif ($_GET['par'][1] == 'location')
297: 			{
298: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
299: 				{
300: 					check_auth_user(); $API->id_role = taxi::$id_role;
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
317: 					if ($_GET['par'][3] == 'send')
318: 					{
319: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
320: 					}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:306`
```php
298: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
299: 				{
300: 					check_auth_user(); $API->id_role = taxi::$id_role;
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
317: 					if ($_GET['par'][3] == 'send')
318: 					{
319: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
320: 					}
321: 				}
322: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:309`
```php
301: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'token')
305: 			{
306: 				check_auth_user(); $API->id_role = taxi::$id_role;
307: 				if (empty($_GET['par'][3]))
308: 				{
309: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
310: 
311: 				}
312: 			}
313: 			elseif ($_GET['par'][1] == 'mail')
314: 			{
315: 				if (!empty($_GET['par'][3]))
316: 				{
317: 					if ($_GET['par'][3] == 'send')
318: 					{
319: 						$out = $API->sendMailToSite(isset(taxi::$data['site_emails'])?taxi::$data['site_emails']:array(),trim($_GET['par'][2]),isset($_POST['subject'])?trim($_POST['subject']):'',isset($_POST['body'])?trim($_POST['body']):'',isset($_POST['file'])?$_POST['file']:'');
320: 					}
321: 				}
322: 			}
323: 			elseif ($_GET['par'][1] == 'referral')
324: 			{
325: 				if (!empty($_GET['par'][2]))
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:343`
```php
335: 						}
336: 					}
337: 				}
338: 			}
339: 			elseif ($_GET['par'][1] == 'data')
340: 			{
341: 				if (isset($_POST['data']))
342: 				{
343: 					check_auth_user(); $API->id_role = taxi::$id_role;
344: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
345: 				}
346: 				elseif (isset($_REQUEST['private']))
347: 				{
348: 					check_auth_user(); $API->id_role = taxi::$id_role;
349: 					$out = $API->selectDataPrivate(taxi::$data_private);
350: 				}
351: 				elseif (empty($_REQUEST['fields']))
352: 				{
353: 					$out = $API->selectData(isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,taxi::$version,taxi::$data);
354: 				}
355: 				else
356: 				{
357: 					$out = $API->selectDataExt(trim($_REQUEST['fields']),isset($_REQUEST['easy'])?1:0,isset($_REQUEST['key'])?trim($_REQUEST['key']):false,empty($_REQUEST['nocache'])?false:true,isset($_REQUEST['team'])?trim($_REQUEST['team']):false,isset($_REQUEST['tournament'])?trim($_REQUEST['tournament']):false,isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,isset($_REQUEST['ucv2'])?trim($_REQUEST['ucv2']):NULL,isset($_REQUEST['ucv3'])?trim($_REQUEST['ucv3']):NULL,taxi::$version,taxi::$data,taxi::$version_stt,taxi::$data_stt,taxi::$version_sc,taxi::$data_sc);
358: 				}
359: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:348`
```php
340: 			{
341: 				if (isset($_POST['data']))
342: 				{
343: 					check_auth_user(); $API->id_role = taxi::$id_role;
344: 					$out = $API->editData($_POST['data'],taxi::$data,taxi::$data_private,taxi::$data_stt,taxi::$data_sc);
345: 				}
346: 				elseif (isset($_REQUEST['private']))
347: 				{
348: 					check_auth_user(); $API->id_role = taxi::$id_role;
349: 					$out = $API->selectDataPrivate(taxi::$data_private);
350: 				}
351: 				elseif (empty($_REQUEST['fields']))
352: 				{
353: 					$out = $API->selectData(isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,taxi::$version,taxi::$data);
354: 				}
355: 				else
356: 				{
357: 					$out = $API->selectDataExt(trim($_REQUEST['fields']),isset($_REQUEST['easy'])?1:0,isset($_REQUEST['key'])?trim($_REQUEST['key']):false,empty($_REQUEST['nocache'])?false:true,isset($_REQUEST['team'])?trim($_REQUEST['team']):false,isset($_REQUEST['tournament'])?trim($_REQUEST['tournament']):false,isset($_REQUEST['ucv'])?trim($_REQUEST['ucv']):NULL,isset($_REQUEST['ucv2'])?trim($_REQUEST['ucv2']):NULL,isset($_REQUEST['ucv3'])?trim($_REQUEST['ucv3']):NULL,taxi::$version,taxi::$data,taxi::$version_stt,taxi::$data_stt,taxi::$version_sc,taxi::$data_sc);
358: 				}
359: 			}
360: 			elseif ($_GET['par'][1] == 'push')
361: 			{
362: 				if (!empty($_GET['par'][2]))
363: 				{
364: 					if ($_GET['par'][3] == 'add')
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:383`
```php
375: 				if (!empty($_GET['par'][2]))
376: 				{
377: 					if ($_GET['par'][2] == 'file')
378: 					{
379: 						if (empty($_GET['par'][3]))
380: 						{
381: 							if (isset($_POST['file']))
382: 							{
383: 								check_auth_user(); $API->id_role = taxi::$id_role;
384: 								$out = $API->createDropboxFile($_POST['file']);
385: 							}
386: 						}
387: 						else
388: 						{
389: 							check_auth_user(); $API->id_role = taxi::$id_role;
390: 							$out = $API->selectDropboxFile(trim($_GET['par'][3]));
391: 						}
392: 					}
393: 					elseif ($_GET['par'][2] == 'update')
394: 					{
395: 						if (isset($_REQUEST['code']) && !(empty($_POST['hash'])))
396: 						{
397: 							$out = $API->updateDropbox(trim($_REQUEST['code']),trim($_POST['hash']));
398: 						}
399: 					}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:389`
```php
381: 							if (isset($_POST['file']))
382: 							{
383: 								check_auth_user(); $API->id_role = taxi::$id_role;
384: 								$out = $API->createDropboxFile($_POST['file']);
385: 							}
386: 						}
387: 						else
388: 						{
389: 							check_auth_user(); $API->id_role = taxi::$id_role;
390: 							$out = $API->selectDropboxFile(trim($_GET['par'][3]));
391: 						}
392: 					}
393: 					elseif ($_GET['par'][2] == 'update')
394: 					{
395: 						if (isset($_REQUEST['code']) && !(empty($_POST['hash'])))
396: 						{
397: 							$out = $API->updateDropbox(trim($_REQUEST['code']),trim($_POST['hash']));
398: 						}
399: 					}
400: 				}			
401: 			}
402: 			elseif ($_GET['par'][1] == 'cart')
403: 			{
404: 				check_auth_user(); $API->id_role = taxi::$id_role;
405: 				if (empty($_GET['par'][2]))
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:404`
```php
396: 						{
397: 							$out = $API->updateDropbox(trim($_REQUEST['code']),trim($_POST['hash']));
398: 						}
399: 					}
400: 				}			
401: 			}
402: 			elseif ($_GET['par'][1] == 'cart')
403: 			{
404: 				check_auth_user(); $API->id_role = taxi::$id_role;
405: 				if (empty($_GET['par'][2]))
406: 				{
407: 					if(empty($_REQUEST['prod']))
408: 					{
409: 						$out = $API->selectCart(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
410: 					}
411: 					else
412: 					{
413: 						$out = $API->updateCart(trim($_REQUEST['prod']),isset($_REQUEST['prop'])?trim($_REQUEST['prop']):NULL,isset($_REQUEST['count'])?(int)$_REQUEST['count']:1);
414: 					}
415: 				}
416: 				else
417: 				{
418: 					if ($_GET['par'][2] == 'clear')
419: 					{
420: 						$out = $API->clearCart(isset($_POST['item'])?$_POST['item']:'');
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:426`
```php
418: 					if ($_GET['par'][2] == 'clear')
419: 					{
420: 						$out = $API->clearCart(isset($_POST['item'])?$_POST['item']:'');
421: 					}
422: 				}
423: 			}
424: 			elseif ($_GET['par'][1] == 'cart_block')
425: 			{
426: 				check_auth_user(); $API->id_role = taxi::$id_role;
427: 				if (empty($_GET['par'][2]))
428: 				{
429: 					if(empty($_REQUEST['prod']))
430: 					{
431: 						$out = $API->selectCartBlock(isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL);
432: 					}
433: 					else
434: 					{
435: 						$out = $API->updateCartBlock(trim($_REQUEST['prod']),isset($_REQUEST['prop'])?trim($_REQUEST['prop']):NULL,isset($_REQUEST['count'])?(int)$_REQUEST['count']:1,isset($_REQUEST['notice'])?(empty($_REQUEST['notice'])?0:1):1,taxi::$data,taxi::$data_stt);
436: 					}
437: 				}
438: 				else
439: 				{
440: 					if ($_GET['par'][2] == 'clear')
441: 					{
442: 						$out = $API->clearCartBlock(isset($_POST['item'])?$_POST['item']:'');
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:452`
```php
444: 					elseif ($_GET['par'][2] == 'status')
445: 					{
446: 						$out = $API->statusCartBlock(isset($_POST['data'])?$_POST['data']:'');
447: 					}
448: 				}
449: 			}
450: 			elseif ($_GET['par'][1] == 'query')
451: 			{
452: 				check_auth_user(); $API->id_role = taxi::$id_role;
453: 				if (!empty($_GET['par'][2]))
454: 				{
455: 					if ($_GET['par'][2] == 'template')
456: 					{
457: 						$out = $API->queryTemplate(isset($_POST['data'])?$_POST['data']:'',empty($_GET['par'][3])?'':trim($_GET['par'][3]),isset(taxi::$data_private['sql_templates'])?taxi::$data_private['sql_templates']:array());
458: 					}
459: 					else
460: 					{
461: 						$out = $API->queryString(isset($_REQUEST['sql'])?trim($_REQUEST['sql']):'',trim($_GET['par'][2]));
462: 					}
463: 				}
464: 			}
465: 			elseif ($_GET['par'][1] == 'schedule')
466: 			{
467: 				check_auth_user(); $API->id_role = taxi::$id_role;
468: 				if (!empty($_GET['par'][2]) && $_GET['par'][2] == 'ticket')
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:467`
```php
459: 					else
460: 					{
461: 						$out = $API->queryString(isset($_REQUEST['sql'])?trim($_REQUEST['sql']):'',trim($_GET['par'][2]));
462: 					}
463: 				}
464: 			}
465: 			elseif ($_GET['par'][1] == 'schedule')
466: 			{
467: 				check_auth_user(); $API->id_role = taxi::$id_role;
468: 				if (!empty($_GET['par'][2]) && $_GET['par'][2] == 'ticket')
469: 				{
470: 					$out = $API->selectTicket(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'');
471: 				}
472: 			}
473: 			elseif ($_GET['par'][1] == 'email')
474: 			{
475: 				check_auth_user(); $API->id_role = taxi::$id_role;
476: 				if (!empty($_GET['par'][2]))
477: 				{
478: 					if ($_GET['par'][2] == 'verification')
479: 					{
480: 						if ($_GET['par'][3] == 'start')
481: 						{
482: 							$out = $API->startEmailVerification();
483: 						}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:475`
```php
467: 				check_auth_user(); $API->id_role = taxi::$id_role;
468: 				if (!empty($_GET['par'][2]) && $_GET['par'][2] == 'ticket')
469: 				{
470: 					$out = $API->selectTicket(isset($_REQUEST['sc_id'])?trim($_REQUEST['sc_id']):'');
471: 				}
472: 			}
473: 			elseif ($_GET['par'][1] == 'email')
474: 			{
475: 				check_auth_user(); $API->id_role = taxi::$id_role;
476: 				if (!empty($_GET['par'][2]))
477: 				{
478: 					if ($_GET['par'][2] == 'verification')
479: 					{
480: 						if ($_GET['par'][3] == 'start')
481: 						{
482: 							$out = $API->startEmailVerification();
483: 						}
484: 						elseif ($_GET['par'][3] == 'complete')
485: 						{
486: 							$out = $API->completeEmailVerification(isset($_GET['ev_hash'])?trim($_GET['ev_hash']):'');
487: 						}
488: 					}
489: 				}
490: 			}
491: 			elseif ($_GET['par'][1] == 'script')
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:493`
```php
485: 						{
486: 							$out = $API->completeEmailVerification(isset($_GET['ev_hash'])?trim($_GET['ev_hash']):'');
487: 						}
488: 					}
489: 				}
490: 			}
491: 			elseif ($_GET['par'][1] == 'script')
492: 			{
493: 				check_auth_user(); $API->id_role = taxi::$id_role;
494: 				if (!empty($_GET['par'][2]))
495: 				{
496: 					if ($_GET['par'][2] == 'template')
497: 					{
498: 						$out = $API->includeTemplate(empty($_GET['par'][3])?'':trim($_GET['par'][3]),empty($_REQUEST['is_var'])?false:true,isset(taxi::$data_private['script_templates'])?taxi::$data_private['script_templates']:array());
499: 					}
500: 				}
501: 			}
502: 			elseif ($_GET['par'][1] == 'translate')
503: 			{
504: 				$out = $API->translate(isset($_POST['data'])?$_POST['data']:'',isset($_POST['from'])?$_POST['from']:'',isset($_POST['to'])?$_POST['to']:'');
505: 			}
506: 		}
507: 		if (isset($_REQUEST['cv'])) 
508: 		{
509: 			$out['cache version'] = taxi::$version;
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:520`
```php
512: 		}
513: 		if (isset($_REQUEST['au']) && !empty($_SESSION[UID]))
514: 		{
515: 			$_REQUEST['au'] = trim($_REQUEST['au']);
516: 			if ($_REQUEST['au'] === '')
517: 			{
518: 				if (empty($out['auth_user']))
519: 				{
520: 					if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
521: 					$out['auth_user'] = array(
522: 											'u_id' => $_SESSION[UID],
523: 											'u_name' => $_SESSION['name'],
524: 											'u_family' => $_SESSION['family'],
525: 											'u_middle' => $_SESSION['middle'],
526: 											'u_email' => $_SESSION['email'],
527: 											'u_phone' => $_SESSION['phone'],
528: 											'u_role' => $_SESSION['id_role'],
529: 											'u_a_role' => $API->id_role,
530: 											'u_check_state' => $_SESSION['id_verification_status'],
531: 											'u_ban' => $_SESSION['user_ban_status'],
532: 											'u_active' => $_SESSION['active'],
533: 											'u_photo' => $_SESSION['photo_link'],
534: 											'u_birthday' => $_SESSION['birthday_date'],
535: 											'u_lang' => $_SESSION['id_lang'], 
536: 											'u_currency' => $_SESSION['currency'],
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:549`
```php
541: 			else
542: 			{
543: 				if ($_REQUEST['au'] === 'e')
544: 				{
545: 					if (empty($out['auth_user']) || !isset($out['auth_user']['u_description']))
546: 					{
547: 						if (empty($out['data']['user'][$_SESSION[UID]]))
548: 						{
549: 							if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
550: 							$out['auth_user'] = $API->selectUser($_SESSION[UID]);						
551: 							$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
552: 						}
553: 						else
554: 						{
555: 							$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
556: 						}
557: 						if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
558: 						unset($out['auth_user']['out_drive']);
559: 						unset($out['auth_user']['out_address']);
560: 						unset($out['auth_user']['out_latitude']);
561: 						unset($out['auth_user']['out_longitude']);
562: 						unset($out['auth_user']['out_est_datetime']);
563: 						unset($out['auth_user']['out_s_address']);
564: 						unset($out['auth_user']['out_s_latitude']);
565: 						unset($out['auth_user']['out_s_longitude']);
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:581`
```php
573: 						unset($out['auth_user']['b_location_classes']);
574: 						unset($out['auth_user']['u_email_checked']);
575: 					}
576: 				}
577: 				elseif ($_REQUEST['au'] === 'f')
578: 				{
579: 					if (empty($out['data']['user'][$_SESSION[UID]]))
580: 					{
581: 						if (empty(taxi::$check_auth_user_count)) {check_auth_user(); $API->id_role = taxi::$id_role;}
582: 						$out['auth_user'] = $API->selectUser($_SESSION[UID]);
583: 						$out['auth_user'] = isset($out['auth_user']['data']['user'][$_SESSION[UID]]) ? $out['auth_user']['data']['user'][$_SESSION[UID]] : array('error'=>$out['auth_user']['message']);
584: 					}
585: 					else
586: 					{
587: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
588: 					}
589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
590: 				}
591: 			}
592: 		}
593: 		if (isset($_SESSION['token_auth']) && empty($_SESSION[UID])) 
594: 		{
595: 			$out['auth_error'] = 'wrong token or u_hash';
596: 		}
597: 	}
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:593`
```php
585: 					else
586: 					{
587: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
588: 					}
589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
590: 				}
591: 			}
592: 		}
593: 		if (isset($_SESSION['token_auth']) && empty($_SESSION[UID])) 
594: 		{
595: 			$out['auth_error'] = 'wrong token or u_hash';
596: 		}
597: 	}
598: 
599: 	header('Content-Type: application/json;charset=UTF-8');
600: 	echo json_encode($out);
601: 	die();
602: ?>
```

### `archive_17012026_1259/taxi/controllers/c_api_test.php:595`
```php
587: 						$out['auth_user'] = $out['data']['user'][$_SESSION[UID]];
588: 					}
589: 					if (empty($out['auth_user']['error'])) $out['auth_user']['u_a_role'] = $API->id_role;
590: 				}
591: 			}
592: 		}
593: 		if (isset($_SESSION['token_auth']) && empty($_SESSION[UID])) 
594: 		{
595: 			$out['auth_error'] = 'wrong token or u_hash';
596: 		}
597: 	}
598: 
599: 	header('Content-Type: application/json;charset=UTF-8');
600: 	echo json_encode($out);
601: 	die();
602: ?>
```

### `archive_17012026_1259/taxi/controllers/c_api.php:26`
```php
18: 		{
19: 			$API->appUrl = $_REQUEST['appUrl'];
20: 		}
21: 
22: 		if (isset($_GET['par'][1]))
23: 		{
24: 			if ($_GET['par'][1] == 'register')
25: 			{
26: 				check_auth_user(); $API->id_role = taxi::$id_role;
27: 				$out = $API->registerUser(isset($_POST['u_role'])?trim($_POST['u_role']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'',isset($_POST['u_name'])?trim($_POST['u_name']):'',isset($_POST['ref_code'])?trim($_POST['ref_code']):'',isset($_COOKIE['reco'])?trim($_COOKIE['reco']):'',$_SERVER['REMOTE_ADDR'],isset($_POST['data'])?$_POST['data']:'',isset($_POST['st'])?true:false,isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array());
28: 			}
29: 			elseif ($_GET['par'][1] == 'auth')
30: 			{
31: 				$out = $API->authUser(isset($_POST['login'])?trim($_POST['login']):'',isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['type'])?trim($_POST['type']):'');
32: 
33: 			}
34: 			elseif ($_GET['par'][1] == 'logout')
35: 			{
36: 				$out = $API->logout();
37: 			}
38: 			elseif ($_GET['par'][1] == 'remind')
39: 			{
40: 				check_auth_user(); $API->id_role = taxi::$id_role;
41: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'');
42: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:40`
```php
32: 
33: 			}
34: 			elseif ($_GET['par'][1] == 'logout')
35: 			{
36: 				$out = $API->logout();
37: 			}
38: 			elseif ($_GET['par'][1] == 'remind')
39: 			{
40: 				check_auth_user(); $API->id_role = taxi::$id_role;
41: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'');
42: 			}
43: 			elseif ($_GET['par'][1] == 'newpass')
44: 			{
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
52: 				{
53: 					if (isset($_POST['data']))
54: 					{
55: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array(),isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
56: 					}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:45`
```php
37: 			}
38: 			elseif ($_GET['par'][1] == 'remind')
39: 			{
40: 				check_auth_user(); $API->id_role = taxi::$id_role;
41: 				$out = $API->remind(isset($_POST['u_email'])?trim($_POST['u_email']):'',isset($_POST['u_phone'])?trim($_POST['u_phone']):'',isset($_POST['u_tg'])?trim($_POST['u_tg']):'',isset($_POST['u_wa'])?trim($_POST['u_wa']):'');
42: 			}
43: 			elseif ($_GET['par'][1] == 'newpass')
44: 			{
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
52: 				{
53: 					if (isset($_POST['data']))
54: 					{
55: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array(),isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
56: 					}
57: 					else
58: 					{
59: 						$out = $API->selectUser(isset($_GET['par'][2])? trim($_GET['par'][2]):'',isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
60: 					}
61: 				}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:50`
```php
42: 			}
43: 			elseif ($_GET['par'][1] == 'newpass')
44: 			{
45: 				check_auth_user(); $API->id_role = taxi::$id_role;
46: 				$out = $API->newpass(isset($_POST['password'])?trim($_POST['password']):'',isset($_POST['new_password'])?trim($_POST['new_password']):'');
47: 			}
48: 			elseif ($_GET['par'][1] == 'user')
49: 			{
50: 				check_auth_user(); $API->id_role = taxi::$id_role;
51: 				if (empty($_GET['par'][3]))
52: 				{
53: 					if (isset($_POST['data']))
54: 					{
55: 						$out = $API->editUser($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['user_roles'])?taxi::$data['user_roles']:array(),isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
56: 					}
57: 					else
58: 					{
59: 						$out = $API->selectUser(isset($_GET['par'][2])? trim($_GET['par'][2]):'',isset(taxi::$data['users_props'])?taxi::$data['users_props']:array(),isset(taxi::$data['field_types'])?taxi::$data['field_types']:array());
60: 					}
61: 				}
62: 				elseif ($_GET['par'][3] == 'car')
63: 				{
64: 					if (isset($_POST['data']))
65: 					{
66: 						$out = $API->controlCar($_POST['data'],isset($_GET['par'][4])?trim($_GET['par'][4]):'',isset($_GET['par'][2])?trim($_GET['par'][2]):'',isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array());
```

### `archive_17012026_1259/taxi/controllers/c_api.php:112`
```php
104: 					if (empty($_GET['par'][4]))
105: 					{
106: 						$out = $API->selectInnerUser(trim($_GET['par'][2]));
107: 					}
108: 				}
109: 			}
110: 			elseif ($_GET['par'][1] == 'car')
111: 			{
112: 				check_auth_user(); $API->id_role = taxi::$id_role;
113: 				if (empty($_GET['par'][3]))
114: 				{
115: 					if (isset($_POST['data']))
116: 					{
117: 						$out = $API->controlCar($_POST['data'],isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array());
118: 					}
119: 					else
120: 					{
121: 						$out = $API->selectCar(isset($_GET['par'][2])?trim($_GET['par'][2]):'','',isset(taxi::$data['langs'])?taxi::$data['langs']:array());
122: 					}
123: 				}
124: 				elseif ($_GET['par'][3] == 'drive')
125: 				{
126: 					$out = $API->setCarUsed(trim($_GET['par'][2]));
127: 				}
128: 			}
```

### `archive_17012026_1259/taxi/controllers/c_api.php:136`
```php
128: 			}
129: 			elseif ($_GET['par'][1] == 'cache' 
130: 				&& !empty($_GET['par'][2]) && $_GET['par'][2] == 'update' && !(empty($_POST['hash'])))
131: 			{
132: 				$out = $API->updateCache(trim($_POST['hash']));
133: 			}
134: 			elseif ($_GET['par'][1] == 'drive')
135: 			{
136: 				check_auth_user(); $API->id_role = taxi::$id_role;
137: 				if (empty($_GET['par'][2]))
138: 				{
139: 					if (isset($_POST['data']))
140: 					{
141: 						$out = $API->createOrder($_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),isset(taxi::$data['payment_ways'])?taxi::$data['payment_ways']:array(),isset(taxi::$data['payment_card'])?taxi::$data['payment_card']:array(),isset(taxi::$data['booking_comments'])?taxi::$data['booking_comments']:array(),isset(taxi::$data['services'])?taxi::$data['services']:array(),isset(taxi::$data['contact_classes'])?taxi::$data['contact_classes']:array(),isset(taxi::$data['booking_location_classes'])?taxi::$data['booking_location_classes']:array(),isset(taxi::$data['currencies'])?taxi::$data['currencies']:array(),isset(taxi::$data['unit_sets'])?taxi::$data['unit_sets']:array(),isset(taxi::$data['countries'])?taxi::$data['countries']:array(),isset(taxi::$data['regions'])?taxi::$data['regions']:array(),isset(taxi::$data['cities'])?taxi::$data['cities']:array(),isset(taxi::$data_sc['schedule'])?taxi::$data_sc['schedule']:array(),isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array(),empty(taxi::$data['site_constants']['stripe_seat_title_template'])?'':taxi::$data['site_constants']['stripe_seat_title_template']['value'],empty(taxi::$data['site_constants']['stripe_request_duration'])?0:taxi::$data['site_constants']['stripe_request_duration']['value'],isset(taxi::$data_private['aggregators'])?taxi::$data_private['aggregators']:array());
142: 					}
143: 					else
144: 					{
145: 						$out = $API->selectActiveOrder(isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
146: 					}
147: 				}
148: 				else
149: 				{
150: 					if ($_GET['par'][2] == 'get')
151: 					{
152: 						if (isset($_POST['action']))
```

### `archive_17012026_1259/taxi/controllers/c_api.php:219`
```php
211: 					elseif ($_GET['par'][2] == 'archive')
212: 					{
213: 						$out = $API->selectArchiveOrder(isset($_REQUEST['fields'])?$_REQUEST['fields']:0);
214: 					}
215: 				}
216: 			}
217: 			elseif ($_GET['par'][1] == 'trip')
218: 			{
219: 				check_auth_user(); $API->id_role = taxi::$id_role;
220: 				if (empty($_GET['par'][2]))
221: 				{
222: 					if (isset($_POST['data']))
223: 					{
224: 						$out = $API->createTrip(empty($_POST['u_id'])?'':trim($_POST['u_id']),$_POST['data'],isset(taxi::$data['langs'])?taxi::$data['langs']:array(),DEFAULT_PROFILE == 'stadium' ? array(taxi::$data,taxi::$data_stt,taxi::$data_sc) : array());
225: 					}
226: 					else
227: 					{
228: 						$out = $API->selectTrip(NULL,'active',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0,isset($_REQUEST['raw_price'])?true:false,isset($_REQUEST['wi'])?true:false,isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array(),isset(taxi::$data_private['aggregators'])?taxi::$data_private['aggregators']:array());
229: 					}
230: 				}
231: 				
232: 				else
233: 				{
234: 					if ($_GET['par'][2] == 'get')
235: 					{
```

### `archive_17012026_1259/taxi/controllers/c_api.php:308`
```php
300: 						$out = $API->selectTrip(NULL,'processing',isset($_REQUEST['orders'])?trim($_REQUEST['orders']):"",isset($_REQUEST['filter'])?$_REQUEST['filter']:NULL,isset($_REQUEST['fields'])?$_REQUEST['fields']:0,isset($_REQUEST['raw_price'])?true:false,isset($_REQUEST['wi'])?true:false,isset(taxi::$data_private['price_time_functions'])?taxi::$data_private['price_time_functions']:array(),isset(taxi::$data_private['aggregators'])?taxi::$data_private['aggregators']:array());
301: 					}
302: 				}
303: 			}
304: 			elseif ($_GET['par'][1] == 'location')
305: 			{
306: 				if (isset($_POST['latitude']) && isset($_POST['longitude']))
307: 				{
308: 					check_auth_user(); $API->id_role = taxi::$id_role;
309: 					$out = $API->setLocation(trim($_POST['latitude']),trim($_POST['longitude']));
310: 				}
311: 			}
312: 			elseif ($_GET['par'][1] == 'token')
313: 			{
314: 				check_auth_user(); $API->id_role = taxi::$id_role;
315: 				if (empty($_GET['par'][3]))
316: 				{
317: 					$out = $API->selectToken(isset($_GET['par'][2])?trim($_GET['par'][2]):'',!empty($_SESSION['token_auth'])?array($_POST['token'],$_POST['u_hash']):array());
318: 
319: 				}
320: 			}
321: 			elseif ($_GET['par'][1] == 'mail')
322: 			{
323: 				if (!empty($_GET['par'][3]))
324: 				{
```

## 5. Почему RP-36 не следует считать окончательным доказательством отсутствия Auth bridge

RP-36 искал exact-string совпадение endpoint names. Это слишком узкий критерий для legacy API, где frontend может формировать URL через общий `API_URL/apiMethod`, а backend route может регистрироваться/разбираться другим способом.

Поэтому:

```text
0 exact endpoint strings    ≠absence of authentication relation```

Это корректирует именно метод поиска, а не семантический граф.

## 6. Что уже можно подтвердить

Если frontend evidence показывает общий request adapter, который добавляет credential fields ко всем API requests, а backend evidence показывает, что тот же credential участвует в `check_auth_user()`, relation может быть подтверждена даже без literal одинаковой строки endpoint.

Но это требует доказательства **одного и того же credential value-flow**, а не простого совпадения имён.

## 7. Current status

На текущем reconciliation pass:

```text
Frontend has authentication/request context       CONFIRMED
Core Backend has authentication gate              CONFIRMED
Frontend credential == backend auth credential     NOT YET CLOSED
Frontend AUTHENTICATES_WITH Core Backend           UNKNOWN```

То есть RP-36 снимается как слишком сильное основание для `SOURCE_GAP`. Правильнее сейчас использовать:

```text
UNKNOWN / VALUE_FLOW_UNRESOLVED```

## 8. Research Question for next step

> Какой конкретный credential создаётся/получается frontend и каким конкретным полем/операцией Core Backend `check_auth_user()` его проверяет?

Expected Evidence:

```text
frontend credential creation/storage        ↓
apiMethod/request construction        ↓
HTTP field/header/query        ↓
backend request extraction        ↓
check_auth_user() input        ↓
authenticated user```

Это теперь правильный следующий Auth pass. Не искать login endpoint вслепую и не переходить к Order.