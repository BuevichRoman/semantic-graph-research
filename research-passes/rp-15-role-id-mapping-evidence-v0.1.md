# Backend Semantic Graph — Research Pass 15
# Role ID Mapping Evidence v0.1

**Статус:** PARTIALLY ANSWERED / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-14 Role ID Semantic Mapping v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Исследовательский вопрос

> Где production source/config/data явно связывает числовой `id_role` с бизнес-смыслом роли?

Искомые источники:

```text
users_roles / role tables
seed INSERTs
role constants / enum
site_constant descriptions
explicit UI labels
role configuration
```

## 2. Правило доказательства

В этом pass действует более строгий критерий:

```text
explicit role-id ↔ role-name mapping
        ↓
CONFIRMED
```

Простое использование:

```text
id_role == 2
```

остаётся только Evidence использования ID.

## 3. Найденные релевантные contexts

Всего найдено релевантных contexts: **2**.

### `archive_17012026_1259/taxi/config/config.php:77`
```text
74: 
75: 	define('CONFIG_URL', ROOT_URL . CONFIG_URL_ROUTE . '/' . CONFIG . '/');
76: 	define('CONFIG_CACHE', ROOT_URL . 'cache/data' . (CONFIG ? '_' . CONFIG : ''));
77: 	class taxi {public static $data = array(); public static $version = ''; public static $check_auth_user_count = 0;public static $id_role = NULL; public static $data_private = array(); public static $data_stt = array(); public static $version_stt = ''; public static $data_sc = array(); public static $version_sc = '';}
78: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.php');
79: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.(stt).php');
80: 	@include_once($_SERVER['DOCUMENT_ROOT'] . CONFIG_CACHE . '.(sc).php');
```

### `archive_17012026_1259/taxi/config/system_bot.php:569`
```text
566: 			LEFT JOIN `car_users` USING (`id_user`)
567: 			LEFT JOIN `car` ON `car`.`id_car` = `car_users`.`id_car`
568: 			WHERE
569: 				`id_role` = '2' AND `id_verification_status` = '2' AND `deleted` = '0' AND `active` = '1' AND
570: 				`out_order` = '0' AND 
571: 				" . $sql_where_loc . "
572: 				`users_location`.`lat` IS NOT NULL AND `users_location`.`lng` IS NOT NULL AND 
```

## 4. Предварительная оценка

Контексты выше разделяются на:

### A. Direct mapping candidate

Источник прямо или почти прямо связывает:

```text
Role ID ↔ Role Name
```

Только такие fragments могут закрыть G-14.

### B. Role usage

Источник показывает:

```text
id_role
```

в authorization/API logic, но не сообщает business name.

Это не закрывает mapping.

### C. Domain mention

Слово `driver`, `admin`, `client` и т. п. встречается рядом с role-related code, но без явного numeric mapping.

Это тоже не закрывает mapping.

## 5. Текущий mapping

До подтверждения прямого mapping сохраняем:

| Role ID | Business Role | Status |
|---:|---|---|
| 2 | UNKNOWN | OPEN |
| 4 | UNKNOWN | OPEN |
| 5 | UNKNOWN | OPEN |
| 6 | UNKNOWN | OPEN |

## 6. Почему мы не используем косвенное доказательство

Например:

```text
setDriver()
```

и проверка:

```text
id_role == 2
```

может сильно подсказывать, что `2` — водитель.

Но это всё ещё не тот же Claim, что:

```text
2 IS Driver
```

Для Semantic Graph это два разных Claims:

```text
C1:
Role 2 is permitted for setDriver()

C2:
Role 2 represents Driver
```

C1 может быть CONFIRMED при отсутствии C2.

Это различие сохраняется.

## 7. Следующий шаг

Если среди найденных contexts есть прямой mapping, следующий pass должен открыть именно его источник и проверить:

```text
role ID
→ role name
→ source authority
→ scope/version
```

После первого подтверждения проверить остальные IDs **в том же источнике**, а не собирать их из разных косвенных мест.

## 8. Gap Report

```text
G-14-01  Role ID 2 → business role   OPEN
G-14-02  Role ID 4 → business role   OPEN
G-14-03  Role ID 5 → business role   OPEN
G-14-04  Role ID 6 → business role   OPEN
G-14-05  Complete Role × Operation matrix   OPEN
```

## 9. MCR

`MCR = NO CHANGE`.

Role ID mapping остаётся обычным provenance-bearing SemanticClaim.

## 10. Следующий шаг

Не расширять поиск по всему backend.

Выбрать самый сильный **direct mapping candidate** из найденных source contexts и проверить его целиком. Если прямого mapping действительно нет, закрыть этот факт как `UNKNOWN / SOURCE_GAP`, а не пытаться восстановить названия ролей статистически.
