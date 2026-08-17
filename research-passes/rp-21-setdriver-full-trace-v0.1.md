# Backend Semantic Graph — Research Pass 21
# `setDriver` Full Authorization / Preconditions Trace v0.1

**Статус:** CONFIRMED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-20 Authorization Helper Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  
**Scope:** Core Backend snapshot `archive_17012026_1259`

---

## 1. Research Question

> Как устроено право выполнения `setDriver`: какие роли допускаются, какие дополнительные Preconditions действуют и какие проверки относятся уже не к Authorization, а к бизнес-состоянию заказа/исполнителя?

Выбран именно `setDriver`, потому что это не `/query`, и он содержит несколько уровней проверки:

```text
Authentication
    ↓
Role Gate
    ↓
Role-specific path
    ↓
Order / performer Preconditions
    ↓
Verification / ban / car Preconditions
    ↓
Mutation
```

---

## 2. API entry point

В `controllers/c_api.php` `setDriver` вызывается как API operation:

```php
160: 								if ($_POST['action'] == 'set_waiting_time')
161: 								{
162: 									$out = $API->editWaitingTime(trim($_GET['par'][3]),isset($_POST['previous'])?trim($_POST['previous']):NULL,isset($_POST['additional'])?trim($_POST['additional']):NULL);
163: 								}
164: 								if ($_POST['action'] == 'set_performer')
165: 								{
166: 									$out = $API->setDriver(isset($_POST['data'])?$_POST['data']:'',trim($_GET['par'][3]),!empty($_POST['u_id'])?trim($_POST['u_id']):'',empty($_POST['performer'])?0:1,isset($_POST['b_driver_code'])?trim($_POST['b_driver_code']):NULL,isset($_POST['t_id'])?trim($_POST['t_id']):'');
167: 								}
168: 								elseif ($_POST['action'] == 'set_arrive_state')
169: 								{
170: 									$out = $API->setCarIsArrived(trim($_GET['par'][3]));
```

Конкретный вызов:

```text
$API->setDriver(...)
```

Следовательно:

```text
API request
    ↓
setDriver()
```

является подтверждённым production path.

---

## 3. Authentication Gate

Первый gate внутри `setDriver()`:

```php
6180: 		public function setDriver($data = '', $id_order = "", $id_user = "", $appointed_performer = 0, $code = NULL, $trips = "")
6181: 		{
6182: 			if (empty($_SESSION[UID])) {
6183: 				return $this->showError('404', 'error', 'unauthorized access');
6184: 			}
6185: 
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
6189: 			}
```

Семантика:

```text
no authenticated session
    ↓
REJECT
    "unauthorized access"
```

Это Authentication, а не Role Authorization.

---

# 4. Role Authorization Gate

Непосредственно после authentication:

```php
6186: 			if ($this->id_role != 1 && $this->id_role != 2 && $this->id_role != 5)
6187: 			{
6188: 				return $this->showError('404', 'error', 'wrong user role');
6189: 			}
```

Разрешённые Role IDs:

```text
1 = Client
2 = Driver
5 = Agent
```

Все остальные:

```text
REJECT
    "wrong user role"
```

Role mapping из `taxi/cache/data.php` подтверждён RP-16:

```text
1 → Client
2 → Driver
5 → Agent
```

Поэтому основной Authorization Claim:

```text
setDriver
    ALLOWED_FOR
        Client
        Driver
        Agent
```

Confidence:

```text
CONFIRMED
```

---

# 5. Authorization ≠ одинаковое поведение ролей

После общего role gate происходит ветвление:

```php
6191: 			if ($this->id_role == 1 || $this->id_role == 5)
6192: 			{
```

То есть:

```text
Client / Agent
    ↓
one execution path

Driver
    ↓
another execution path
```

Это очень важное наблюдение.

`setDriver` не означает:

```text
Client = Driver = Agent
```

в смысле capability semantics.

Общее право войти в operation существует, но дальнейшая семантика различается.

---

# 6. Client / Agent path

Для:

```text
Client
Agent
```

код сначала получает данные заказа и выбранного исполнителя:

```php
6193: 				$s = "SELECT
6194: 						`order`.`id_order`,
6195: 						`order`.`client`,
6196: 						`order`.`id_order_status`,
6197: 						`order`.`is_confirmed`,
6198: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6199: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6200: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $id_user 
6201: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state
6202: 					FROM `order` 
6203: 					LEFT JOIN `order_driver` USING (`id_order`)				
6204: 					WHERE	
6205: 						`order`.`id_order` = '" . $id_order . "'
6206: 					LIMIT 1
6207: 					";
6208: 
6209: 				$q = query($s);
6210: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
6211: 				
6212: 				$d = fetch_assoc($q);
6213: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
```

После этого применяются Preconditions.

### Order state

```php
6214: 				if ($d['id_order_status'] != 1) return $this->showError('404', 'error', 'wrong booking state');
6215: 
```

Требуется:

```text
order.id_order_status = 1
```

Иначе:

```text
REJECT
    wrong booking state
```

---

# 7. Ownership Precondition

Проверяется:

```php
6216: 				if ($d['client'] != $_SESSION[UID]) 
6217: 				{
6218: 					return $this->showError('404', 'error', $_SESSION[UID] . ' is not author');
6219: 				}
```

Условие:

```text
order.client == current authenticated user
```

Иначе:

```text
REJECT
    user is not author
```

Это не Role Authorization.

Это:

```text
RESOURCE OWNERSHIP PRECONDITION
```

Semantic relation:

```text
Client
    → MAY_SET_DRIVER_FOR
    → own Order
```

а не:

```text
Client
    → MAY_SET_DRIVER_FOR
    → any Order
```

---

# 8. Performer Preconditions

Далее:

```php
6220: 				if (empty($d['u_id'])) 
6221: 				{
6222: 					return $this->showError('404', 'error', $id_user . ' is not performer');
6223: 				}
6224: 				if (empty($d['is_confirmed'])) return $this->showError('404', 'error', 'booking not confirmed');
6225: 				if ($d['c_state'] != 1) 
6226: 				{
6227: 					return $this->showError('404', 'error', 'wrong booking driver state');
6228: 				}
```

проверяются:

```text
selected user is performer
order is confirmed
driver booking state = 1
```

То есть операция зависит от состояния relation:

```text
Order
    └── has performer
```

и состояния этого performer assignment.

Это уже:

```text
DOMAIN PRECONDITIONS
```

а не Authorization.

---

# 9. Mutation

После прохождения всех этих условий выполняется транзакция:

```php
6230: 				$q = query("BEGIN");
6231: 				if ($q === false) return $this->showError('404', 'error', 'begin query failed');
6232: 	
6233: 				$s = "UPDATE `order`,`order_driver`,(
6234: 							SELECT
6235: 
6236: 								'" . $id_order . "' as `id_order`,
6237: 								@c_c_count := COUNT(`id_order`) as current_car_count
6238: 							FROM
6239: 								`order_driver`
6240: 							WHERE
6241: 								`id_order` = '" . $id_order . "' AND 
6242: 								`id_order_driver_status` in (3,4,5,6)
6243: 
6244: 						) od
6245: 					SET 
6246: 						`order`.`id_order_status` = (SELECT @id := 
6247: 							IF(od.`current_car_count` < `order`.`car_count`-1,1,2)),
6248: 						`order`.`approve_datetime` = 
6249: 							IF(od.`current_car_count` < `order`.`car_count`-1,`order`.`approve_datetime`,now()),	
6250: 						`order`.`car_count` = (SELECT @c_count := `order`.`car_count`),	
6251: 						`order_driver`.`id_order_driver_status` = 
6252: 							IF( od.`current_car_count` < `order`.`car_count`,3,1),
6253: 						`order_driver`.`appoint_datetime` = 
6254: 							IF( od.`current_car_count` < `order`.`car_count`,now(),
6255: 							`order_driver`.`appoint_datetime`)
6256: 					WHERE
6257: 						`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` = '1' AND
6258: 						`order`.`id_order` = `order_driver`.`id_order` AND 
6259: 						`order_driver`.`id_order`='" . $id_order . "' AND 
6260: 						`order_driver`.`id_user` = '" . $id_user . "' AND 
6261: 						`order_driver`.`id_order_driver_status` = '1' AND
6262: 						`order`.`id_order` = od.`id_order`
6263: 					";
6264: 				
6265: 				query($s);
```

и изменяются:

```text
order.id_order_status
order.approve_datetime
order.car_count
order_driver.id_order_driver_status
order_driver.appoint_datetime
```

Таким образом `setDriver` — не просто read capability.

Это:

```text
MUTATING ORDER / PERFORMER ASSIGNMENT
```

---

# 10. Driver path

Для Role 2 ветка начинается после:

```text
if ($this->id_role == 1 || $this->id_role == 5)
```

то есть:

```text
Driver
    ↓
else branch
```

Driver получает другую семантику.

Входные данные ограничиваются whitelist:

```php
6343: 				$allowed_fields = array(									
6344: 										'c_id'					=>		'id_car',
6345: 										'c_payment_way'			=>		'id_payment_method',
6346: 										'c_payment_card'		=>		'id_payment_card',
6347: 										'c_options'				=>		'options'
6348: 				);
6349: 				$forbidden_fields = array();
6350: 				$affected_fields = array();
6351: 				$affected_keys = array();
6352: 				$filtered_data = array();
6353: 				
6354: 				if (!empty($data))
6355: 				{
6356: 					$data = json_decode($data,true);
6357: 					
6358: 					if (!is_array($data)) 
6359: 					{
6360: 						return $this->showError('404', 'error', 'wrong data');
6361: 					}
6362: 
6363: 					foreach ($data as $key => $value)
6364: 					{
6365: 						if (!empty($allowed_fields[$key]))
6366: 						{
6367: 							if (is_string($value)) $data[$key] = trim($value);
6368: 							$affected_fields[] = $key;		
6369: 							$affected_keys[$key] = true;		
6370: 							$filtered_data[$allowed_fields[$key]] = $data[$key];
6371: 						}
6372: 						else
6373: 						{
6374: 							$forbidden_fields[] = $key;
6375: 						}
6376: 					}
6377: 				}
```

Разрешены только:

```text
c_id
c_payment_way
c_payment_card
c_options
```

Все остальные поля:

```text
forbidden_fields
```

Это отдельный:

```text
INPUT FIELD POLICY
```

а не Role Authorization.

---

# 11. Driver order context

Driver path получает заказ и собственное состояние участника:

```php
6383: 				$s = "SELECT
6384: 						`order`.`id_order`,
6385: 						`order`.`client`,
6386: 						`order`.`id_order_status`,
6387: 						`order`.`is_confirmed`,
6388: 						`order`.`car_count`,
6389: 						`order`.`code`,
6390: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6391: 							. "',`order_driver`.`id_user`,NULL)) as u_id,
6392: 						GROUP_CONCAT(IF(`order_driver`.`id_user` = '" . $_SESSION[UID] 
6393: 							. "',`order_driver`.`id_order_driver_status`,NULL)) as c_state,
6394: 						GROUP_CONCAT(IF(`order_driver`.`id_car` = '" 
6395: 							. (!empty($filtered_data['id_car']) ? $filtered_data['id_car'] : "")
6396: 							. "' AND `order_driver`.`not_deleted` IS NOT NULL,`order_driver`.`id_car`,NULL)) as c_id,
6397: 						COUNT(IF(`order_driver`.`id_order_driver_status` in (3,4,5,6),
6398: 							1,NULL)) as current_car_count,
6399: 						COUNT(IF(`order_driver`.`id_order_driver_status` = '3',
6400: 							1,NULL)) as appointed_performer_count,
6401: 						IFNULL(MAX(`order_driver`.`index`),0) as max_index,
6402: 						(SELECT
6403: 							COUNT(`order_driver_attempt`.`id_order`)
6404: 						 FROM `order_driver_attempt`
6405: 						 WHERE 
6406: 							`order_driver_attempt`.`id_order` = `order`.`id_order` AND 
6407: 							`order_driver_attempt`.`id_user`= '" . $_SESSION[UID] . "'
6408: 						) as attempt,
6409: 						IF(`order`.`id_order_status` = '6',(
6410: 							SELECT
6411: 								`order_driver_select`.`id_user`
6412: 							 FROM `order_driver_select`
6413: 							 WHERE 
6414: 								`order_driver_select`.`id_order` = `order`.`id_order` AND 
6415: 								`order_driver_select`.`id_user`= '" . $_SESSION[UID] . "'
6416: 						),NULL) as selected,
6417: 						`order`.`night_time`,
6418: 						`order`.`distance_estimate`,
6419: 						`order`.`id_car_class`,
6420: 						`order`.`options`
6421: 					FROM `order` 
6422: 					LEFT JOIN `order_driver` USING (`id_order`)				
6423: 					WHERE	
6424: 						`order`.`id_order` = '" . $id_order . "'
6425: 					LIMIT 1
6426: 					FOR UPDATE
6427: 					";
6428: 
6429: 				$q = query($s);
6430: 				if ($q === false) return $this->showError('404', 'error', 'database select failed');
```

В частности:

```text
current user's order_driver relation
current driver state
car relation
attempt count
selected state
```

Это показывает, что Driver operation определяется не только ролью.

---

# 12. Driver cannot act on own order

Проверяется:

```php
6438: 				if (empty($d['id_order'])) return $this->showError('404', 'error', 'booking not found');
6439: 				if ($d['client'] == $_SESSION[UID]) return $this->showError('404', 'error', 'driver is author');
6440: 				if ((int)$d['attempt'] >= $this->constant['driver_code_attempt_count_max'])
6441: 				{
6442: 					return $this->showError('404', 'error', 'maximum attempts exceeded');
6443: 				}
```

Если:

```text
order.client == current user
```

операция отклоняется:

```text
driver is author
```

Это:

```text
DOMAIN RELATION PRECONDITION
```

---

# 13. Driver code

При наличии `code`:

```php
6445: 				if (isset($code)) {
6446: 					if (strlen($d['code']) == 0) 
6447: 					{
6448: 						return $this->showError('404', 'error', 'booking with empty driver code');
6449: 					}
6450: 					if ($d['code'] == $this->prepareForDriverCode($code))
6451: 					{
6452: 						$appointed_performer = 1;
6453: 					}
6454: 					else
6455: 					{
6456: 						$s = "INSERT INTO `order_driver_attempt`
6457: 							SET 
6458: 								`id_order` = '" . $id_order . "',
6459: 								`id_user` = '" . $_SESSION[UID]  . "',
6460: 								`datetime` = now()
6461: 							";
6462: 
6463: 						$q = query($s);
6464: 						if ($q === false) return $this->showError('404', 'error', 'database attempt insert failed');
6465: 						$q = query("COMMIT");
6466: 						if ($q === false) return $this->showError('404', 'error', 'commit failed');
6467: 						return $this->showError('404', 'error', 'wrong driver code');
6468: 					}				
```

происходит:

```text
provided code
    ↓
prepareForDriverCode()
    ↓
compare with order.code
```

При ошибке:

```text
attempt is persisted
    ↓
REJECT
    wrong driver code
```

Следовательно:

```text
Driver Authorization
```

не означает автоматического права стать исполнителем.

Есть дополнительная:

```text
POSSESSION / SECRET CODE PRECONDITION
```

---

# 14. Verification State

Если исполнитель ещё не назначен:

```php
6477: 				if (empty($d['u_id'])) 
6478: 				{
6479: 					if ($_SESSION['id_verification_status'] != 2)
6480: 					{
6481: 						return $this->showError('404', 'error', 'wrong user check state');
6482: 					}
6483: 					if (!empty($_SESSION['user_ban_status']['order']))
6484: 					{
6485: 						return $this->showError('404', 'error', 'user banned');
6486: 					}
```

требуется:

```text
$_SESSION['id_verification_status'] == 2
```

Иначе:

```text
REJECT
    wrong user check state
```

Также проверяется:

```text
user_ban_status['order']
```

и при наличии запрета:

```text
REJECT
    user banned
```

Semantic model:

```text
Driver
    ├── requires Verification State = 2
    └── requires no Order Ban
```

Это Preconditions.

---

# 15. Car ownership

При переданном `id_car`:

```php
6487: 					if (!empty($d['c_id'])) 
6488: 					{
6489: 						return $this->showError('404', 'error', 'car is already used in this drive');
6490: 					}
6491: 					elseif (!empty($filtered_data['id_car']))
6492: 					{
6493: 						$s = "SELECT
6494: 								`car`.`id_car`,
6495: 								cu.`id_user`,
6496: 								`car`.`id_car_class`
6497: 							FROM `car`
6498: 							LEFT JOIN (
6499: 									SELECT
6500: 										`id_car`,
6501: 										`id_user`
6502: 									FROM
6503: 										`car_users`
6504: 									WHERE
6505: 										`id_user` = '" . $_SESSION[UID] . "'
6506: 								) cu USING (`id_car`)						
6507: 							WHERE
6508: 								`car`.`id_car` = '" . $filtered_data['id_car'] . "'
6509: 							";
6510: 
6511: 						$q = query($s);
6512: 						if ($q === false) return $this->showError('404', 'error', 'car database select failed');
6513: 						
6514: 						$d_c = fetch_assoc($q);
6515: 						if (empty($d_c['id_car'])) 
6516: 						{
6517: 							return $this->showError('404', 'error', 'car not found');
6518: 						}
6519: 						if (empty($d_c['id_user'])) 
6520: 						{
6521: 							return $this->showError('404', 'error', 'car is not this driver');
6522: 						}
```

backend проверяет:

```text
car exists
car belongs to current driver
```

При нарушении:

```text
REJECT
```

Таким образом:

```text
Driver
    → may use
    → own/assigned Car
```

но не произвольный автомобиль.

---

# 16. Итоговая semantic model `setDriver`

```text
                 setDriver
                     │
                     ↓
              Authentication
                     │
                     ↓
                 Role Gate
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      Client / Agent          Driver
          │                     │
          ↓                     ↓
   Order Ownership       Driver Preconditions
          │                     │
   Order State             Verification
          │                Ban State
   Performer State         Driver Code
          │                Car Ownership
          └──────────┬──────────┘
                     ↓
                Mutation
                     ↓
          Order / Performer State
```

---

# 17. Role × Operation × Preconditions

| Business Role | Operation | Decision | Preconditions | Status |
|---|---|---|---|---|
| Client | `setDriver` | ALLOW | authenticated + own order + order state + performer state | CONFIRMED |
| Agent | `setDriver` | ALLOW | authenticated + own-order check still applies in shared path | CONFIRMED for role gate; detailed business semantics require care |
| Driver | `setDriver` | ALLOW | authenticated + verification + no order ban + driver/order/car/code conditions | CONFIRMED |
| Administrator | `setDriver` | REJECT | role gate | CONFIRMED |
| Usher | `setDriver` | REJECT | role gate | CONFIRMED |
| Usher with extended powers | `setDriver` | REJECT | role gate | CONFIRMED |

Важно:

`ALLOW` здесь означает прохождение operation-specific gate, а не гарантию успешной мутации при любых входных данных. Business Preconditions могут привести к `REJECT`.

---

# 18. Особенно важный результат для Semantic Graph

`setDriver` показывает, что отношение:

```text
Role
    → CAN_EXECUTE
    → Operation
```

слишком грубое.

Более точная структура:

```text
Role
    → MAY_INVOKE
    → Operation

Operation
    → REQUIRES
    → Authentication

Operation
    → REQUIRES_ROLE
    → Role Set

Operation
    → REQUIRES_PRECONDITION
    → Order Ownership

Operation
    → REQUIRES_PRECONDITION
    → Order State

Operation
    → REQUIRES_PRECONDITION
    → Verification State

Operation
    → REQUIRES_PRECONDITION
    → Car Ownership

Operation
    → REQUIRES_PRECONDITION
    → Driver Code
```

Это уже значительно ближе к реальной semantic structure production backend.

---

# 19. Методологический результат

RP-20 предполагал:

```text
authentication helper
+
operation-local authorization
```

RP-21 показывает следующий уровень:

```text
Authentication
    ↓
Authorization
    ↓
Domain Preconditions
    ↓
Mutation
```

И это нельзя смешивать.

Например:

```text
verification_status = 2
```

не следует помещать в:

```text
Role Permission Matrix
```

как будто это ещё одна роль.

Это отдельный Claim:

```text
Driver Operation
    REQUIRES
Verification State = 2
```

---

# 20. MCR

`MCR = NO CHANGE`.

Не требуется новый фундаментальный тип Semantic Graph.

Но становится практически необходимым иметь typed relations минимум:

```text
REQUIRES_AUTHENTICATION
REQUIRES_ROLE
REQUIRES_PRECONDITION
MAY_INVOKE
REJECTS
MUTATES
```

Это не изменение методологии v2.3, а конкретное evidence из production pass, подтверждающее полезность уже заложенного relation typing.

---

# 21. Gap Report

Закрыто:

```text
G-21-01  полный role gate setDriver          CLOSED
G-21-02  Client/Agent execution path        CLOSED
G-21-03  Driver execution path               CLOSED
G-21-04  major domain preconditions          CLOSED
```

Открыто:

```text
G-21-05  точный API route/name для setDriver
G-21-06  frontend consumers setDriver
G-21-07  relation setDriver → order assignment
G-21-08  связь setDriver с map/assigned performer
```

Последние два особенно интересны для уже исследованной задачи карты.

---

# 22. Следующий шаг

Не исследовать ещё одну случайную authorization operation.

Теперь есть смысл перейти вертикально:

```text
setDriver
    ↓
Order → Performer Assignment
    ↓
assigned User
    ↓
current Position
    ↓
Position Exposure
    ↓
Taxi Frontend map
```

Это соединит две ранее независимые ветки исследования:

```text
Authorization
```

и:

```text
Location / Map
```

и позволит проверить, как Semantic Graph связывает **право изменить assignment** с последующим **frontend-visible domain state**.
