# Backend Semantic Graph — Research Pass 13
# `query_roles` Authorization Boundary v0.1

**Статус:** CONFIRMED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-12 `query_roles` SQL / Result Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

---

## 1. Исследовательский вопрос

> Является ли `query_roles` только ограничителем query scope или он непосредственно участвует в Authorization decision?

Ответ теперь получен непосредственно из production code:

```text
query_roles
    ↓
role validation
    ↓
ALLOW / REJECT
```

Следовательно:

> **`query_roles` является частью Authorization mechanism для метода `query`.**

---

# 2. Endpoint `/query`

В `c_api.php` перед вызовом `queryString()` выполняется:

```text
check_auth_user()
$API->id_role = taxi::$id_role
```

После чего значение:

```text
taxi::$data['site_constants']['query_roles']['value']
```

передаётся в:

```text
$API->queryString(...)
```

Это связывает:

```text
authenticated API context
    ↓
runtime API role
    ↓
query_roles
    ↓
queryString()
```

---

# 3. `query_roles` явно описан как список разрешённых ролей

В `cache/data.php`:

```text
'query_roles'
```

имеет описание:

```text
Список ролей, для которых доступен метод query.
```

и:

```text
Пустая строка означает, что доступно всем.
Значение должно иметь вид идентификаторов ролей через запятую.
Для водителя (роль 2) требуется check_state=2.
```

Текущее значение snapshot:

```text
'4'
```

Таким образом, configuration semantics прямо соответствует Authorization, а не только query filtering.

---

# 4. Непосредственная проверка в `queryString()`

`api.php`:

```text
public function queryString(
    $sql = "",
    $statement = "select",
    $var = NULL,
    $query_roles = '',
    $hash = ''
)
```

Сначала проверяется authentication:

```text
if (empty($_SESSION[UID]))
    return unauthorized access
```

Затем, если `query_roles` непуст:

```text
if ($this->id_role == 2 &&
    $_SESSION['id_verification_status'] != 2)
{
    return wrong user check state;
}
```

После этого:

```text
$query_roles = explode(',', $query_roles);
$query_roles = array_flip($query_roles);

if (!isset($query_roles[$this->id_role]))
{
    return forbidden role;
}
```

Это уже прямое Authorization Decision.

---

# 5. Доказанная цепочка

Полная цепочка:

```text
Request /query
      ↓
check_auth_user()
      ↓
Authenticated User
      ↓
taxi::$id_role
      ↓
API->id_role
      ↓
query_roles
      ↓
allowed-role set
      ↓
role membership check
      ↓
ALLOW
   OR
REJECT: forbidden role
```

Confidence:

```text
CONFIRMED
```

---

# 6. Это не QUERY_SCOPE

Ключевой момент:

`query_roles` **не участвует в построении SQL WHERE/JOIN или ограничении набора строк**.

Он проверяется до выполнения SQL:

```text
query_roles
    ↓
authorization check
    ↓
query execution
```

Поэтому его семантика:

```text
AUTHORIZATION
```

а не:

```text
QUERY_SCOPE
```

---

# 7. Authorization и Query Scope теперь разделены

В результате предыдущая гипотеза уточняется.

### Authorization

```text
query_roles
    ↓
allowed roles
    ↓
forbidden role / allow
```

### Query execution

После успешной проверки:

```text
SQL
    ↓
query(...)
    ↓
result set
```

Эти уровни разделены.

Сам `query_roles` не ограничивает result set.

---

# 8. Дополнительный authorization gate: verification state

Для:

```text
id_role = 2
```

есть дополнительное условие:

```text
$_SESSION['id_verification_status'] == 2
```

Если оно не выполнено:

```text
wrong user check state
```

Следовательно, Authorization для `query` зависит не только от Role.

Более точная модель:

```text
Runtime API Role
       +
Verification State
       ↓
Query Access Decision
```

---

# 9. Дополнительный gate для statement type

После role authorization `queryString()` проверяет тип statement.

Разрешены:

```text
select
```

а:

```text
update
insert
delete
replace
```

разрешаются только если:

```text
query_extended_statements !== false
```

Для:

```text
custom
```

есть отдельный gate:

```text
$this->id_role != 4
    → not enough rights
```

и дополнительная проверка:

```text
hash == md5('checking' . md5(API_KEY))
```

Следовательно, `/query` имеет несколько независимых authorization/security gates.

---

# 10. Итоговая модель `/query`

```text
                    /query
                       │
                       ↓
                Authentication
                       │
                       ↓
                 Runtime Role
                       │
             ┌─────────┴──────────┐
             ↓                    ↓
        query_roles         Verification State
             │                    │
             └─────────┬──────────┘
                       ↓
                Query Authorization
                       │
                 ALLOW / REJECT
                       │
                       ↓
                Statement Gate
                       │
          ┌────────────┼─────────────┐
          ↓            ↓             ↓
        SELECT      DML*          CUSTOM
          │            │             │
          │       extended flag    role=4
          │                          +
          │                        hash
          └────────────┬─────────────┘
                       ↓
                    SQL
                       ↓
                   Result
```

`DML*` = `update/insert/delete/replace`.

---

# 11. Что теперь можно добавить в Semantic Graph

### Claim

```text
Query API
    REQUIRES_ROLE_SET
    → query_roles
```

```text
confidence: CONFIRMED
```

### Claim

```text
Query API
    REJECTS
    → Runtime Role outside query_roles
```

```text
confidence: CONFIRMED
```

### Claim

```text
Driver Query Access
    REQUIRES
    Verification State = 2
```

```text
confidence: CONFIRMED
```

### Claim

```text
Custom Query Execution
    REQUIRES
    Runtime Role = 4
```

```text
confidence: CONFIRMED
```

---

# 12. Важное следствие для Role Permission Matrix

Теперь для конкретной операции можно записать:

```text
Operation:
    /query

Role Gate:
    query_roles

Additional condition:
    role 2 → verification_state = 2
```

Для текущего snapshot:

```text
query_roles = 4
```

поэтому непосредственно конфигурация текущей версии говорит:

```text
role 4
    → query access
```

При этом нельзя превращать это в глобальное утверждение:

```text
role 4 = administrator
```

Пока mapping Role ID → business role отдельно не подтверждён.

---

# 13. Что закрыто

```text
G-10-01
query_roles exact semantic effect
CLOSED

G-10-02
query_roles explicit permission rejection
CLOSED

G-12-01
exact query_roles authorization boundary
CLOSED
```

---

# 14. Что остаётся

```text
G-06-02
Role ID → business role mapping
OPEN

G-06-01
Complete Role × Operation matrix
OPEN

G-06-04
role{N}.php semantics
OPEN
```

Но теперь `query_roles` больше не является неизвестным.

---

# 15. Методологический результат

Этот эксперимент дал важное подтверждение Research Loop v2.3.

Исходная гипотеза:

```text
query_roles
    ?
    QUERY_SCOPE
```

прошла цепочку:

```text
UNKNOWN
   ↓
Research Question
   ↓
consumer
   ↓
queryString()
   ↓
role check
   ↓
forbidden role
   ↓
CONFIRMED AUTHORIZATION
```

То есть граф действительно помог не только сохранить неизвестность, но и **дойти до места, где неизвестность была снята реальным Evidence**.

---

# 16. Новый важный вывод о Capability

Теперь можно выделить:

```text
Query Access Control
```

как capability-level semantic construct.

Но не следует делать его отдельным фундаментальным Frame только потому, что найден один endpoint.

Корректнее:

```text
Query API
    HAS_CAPABILITY
    → Role-Gated Query Execution
```

а дальнейшие методы должны показать, является ли это частью более общей:

```text
Authorization Capability
```

---

# 17. Следующий шаг

Теперь не нужно больше исследовать `query_roles`.

Следующий рациональный проход:

# RP-14 — Role ID Semantic Mapping

Задача:

```text
role ID
   ↓
source/config/data
   ↓
business role meaning
```

Например:

```text
2 → ?
4 → ?
5 → ?
6 → ?
```

И только после этого возвращаемся к накопленной Role Permission Matrix:

```text
Business Role
      ×
Protected Operation
      ↓
ALLOW / REJECT
```

Это даст уже человечески читаемую authorization model, но каждое соответствие будет иметь собственное Evidence.
