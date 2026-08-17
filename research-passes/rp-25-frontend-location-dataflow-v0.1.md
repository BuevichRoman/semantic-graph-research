# Backend Semantic Graph — Research Pass 25
# Concrete Frontend `/location` Data-Flow Trace v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED
**Методология:** Semantic Graph Research Methodology v2.3
**Предшествующий проход:** RP-24 Order Response → `/location` Bridge v0.1
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Research Question

> Из какого конкретного frontend state/value формируется `u_id` при вызове `/location` или `selectLocation`, и можно ли доказать, что это assigned performer из order state?

## 2. Concrete `/location` consumers

Найдено concrete location-call contexts: **2**.

### `archive_17012026_1259/taxi/controllers/c_index.php:2397` — `global`
```text
2385: 						<option value="set_complete_state">Завершение поезки</option>
2386: 						<option value="set_cancel_state">Отмена поездки</option>
2387: 						<option value="set_rate">Оценка поездки</option>
2388: 
2389: 						<option value="set_tips">Ввод чаевых</option>
2390: 						<option value="edit">Редактирование поездки</option>	
2391: 					</select>
2392: 				</label>
2393: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2394: 			</form>
2395: 		</fieldset>
2396: 		<pre>
2397: 	https://ibronevik.ru/taxi/api/v1/location								POST
2398: 		 Запись координат авторизированного пользователя в базу.
2399: 			Доступно только для авторизованного пользователя.
2400: 			Параметры запроса;
2401: 				latitude			широта				необходимо
2402: 				longitude			долгота				необходимо
2403: 			Ответ сервера:
2404: 			{'code':'200','status':'success'}</pre>
2405: 		<fieldset class="form"><legend title="Записывает координаты авторизированного пользователя в базу. Доступна только для авторизованного пользователя.">Изменение координат пользователя</legend>
2406: 			<form class="complex" action="api/v1/location" method="POST" enctype="application/x-www-form-urlencoded;charset=UTF-8" target="_blank">
2407: 				<label class="key"><span>широта</span><input data-name="latitude" name="latitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2408: 				<label class="key"><span>долгота</span><input data-name="longitude" name="longitude" type="text"> <a onclick="exclude_input_simple(this)" href="javascript:void 0;">не отправлять</a></label>
2409: 
2410: 				<div class="submit_wrap"><input type="submit" value="ОК"></div>
2411: 			</form>
2412: 		</fieldset>
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2406` — `global`
```text
2394: 			</form>
2395: 		</fieldset>
2396: 		<pre>
2397: 	https://ibronevik.ru/taxi/api/v1/location								POST
2398: 		 Запись координат авторизированного пользователя в базу.
2399: 			Доступно только для авторизованного пользователя.
2400: 			Параметры запроса;
2401: 				latitude			широта				необходимо
2402: 				longitude			долгота				необходимо
2403: 			Ответ сервера:
2404: 			{'code':'200','status':'success'}</pre>
2405: 		<fieldset class="form"><legend title="Записывает координаты авторизированного пользователя в базу. Доступна только для авторизованного пользователя.">Изменение координат пользователя</legend>
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
```

## 3. Backend response candidates

Найдено response/assignment candidates: **3**.

### `archive_17012026_1259/taxi/models/api.php:7240`
```text
7232: 						if ($q === false) return $this->showError('404', 'error', 'booking_cancel_states insert failed');
7233: 					}
7234: 
7235: 					$s = "UPDATE `order`,`order_driver`
7236: 						SET 
7237: 							`order`.`last_edit_datetime` = now(),
7238: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7239: 							`order_driver`.`id_order_driver_status` = '2',
7240: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7241: 							`order_driver`.`cancel_datetime` = now(),
7242: 							`order_driver`.`not_deleted` = NULL
7243: 						WHERE
7244: 							`order`.`id_order` = '" . $id_order . "' AND 
7245: 							`order`.`id_order` = `order_driver`.`id_order` AND
7246: 							`order_driver`.`id_order` = '" . $id_order . "' AND 
7247: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND
7248: 							`order_driver`.`id_order_driver_status` = '1'
7249: 						";
7250: 
7251: 					query($s);
7252: 			
```

### `archive_17012026_1259/taxi/models/api.php:7297`
```text
7289: 											`order_driver`
7290: 										WHERE
7291: 											`id_order` = '" . $id_order . "' AND 
7292: 											`id_order_driver_status` in (3,4,5,6)
7293: 									) od
7294: 								) < `order`.`car_count`+1,
7295: 								IF(`order`.`id_order_status` in (1,5,6),`order`.`id_order_status`,1),2),
7296: 							`order_driver`.`id_order_driver_status` = '2',
7297: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7298: 							`order_driver`.`cancel_datetime` = now(),
7299: 							`order_driver`.`not_deleted` = NULL
7300: 						WHERE
7301: 							`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` in (1,2,5,6) AND
7302: 							`order`.`id_order` = `order_driver`.`id_order` AND 
7303: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7304: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
7305: 							`order_driver`.`id_order_driver_status` in (3,4)
7306: 						";
7307: 					
7308: 					query($s);
7309: 
```

### `archive_17012026_1259/taxi/models/api.php:7358`
```text
7350: 				
7351: 					$s = "UPDATE `order`,`order_driver`
7352: 						SET 
7353: 							`order`.`last_edit_datetime` = now(),
7354: 							`order`.`last_edit_user` = '" .  $_SESSION[UID] . "',
7355: 							`order`.`id_order_status` = '3',
7356: 							`order`.`cancel_datetime` = now(),
7357: 							`order_driver`.`id_order_driver_status` = '2',
7358: 							`order_driver`.`cancel_reason` = '" . str_replace(array(chr(0),chr(1),chr(2)),' ',$cancel_reason) . "',
7359: 							`order_driver`.`cancel_datetime` = now(),
7360: 							`order_driver`.`not_deleted` = NULL
7361: 						WHERE
7362: 							`order`.`id_order` = '" . $id_order . "' AND `order`.`id_order_status` in (6,1,2) AND
7363: 
7364: 							`order`.`id_order` = `order_driver`.`id_order` AND 
7365: 							`order_driver`.`id_order`='" . $id_order . "' AND 
7366: 							`order_driver`.`id_user` = '" . $_SESSION[UID] . "' AND 
7367: 							`order_driver`.`id_order_driver_status` = '5'
7368: 						";
7369: 						
7370: 					query($s);
```

## 4. Data-flow closure criterion

Bridge считается `CONFIRMED` только если конкретный frontend call позволяет проследить:

```text
order state
   ↓
assigned performer / id_driver
   ↓
same value or deterministic derivation
   ↓
u_id
   ↓
/location
```

Наличие `u_id` само по себе недостаточно.

Наличие `id_driver` в другом response также недостаточно.

## 5. Текущий результат

Этот pass выделяет concrete consumers и response candidates, но Semantic Graph обновляется только после доказательства value-flow между ними.

Если конкретный call получает `u_id` из current user/session, то это не assignment → position bridge.

Если `u_id` получает значение из assigned driver field order state, bridge закрывается.

Если значение проходит через helper/selector, требуется открыть helper и доказать derivation.

## 6. Статус Claims

```text
User → HAS_CURRENT_POSITION → PositionCONFIRMED

Order → ASSIGNED_PERFORMER → UserCONFIRMED

Frontend → CALLS → /locationCONFIRMED for concrete consumers

Assigned Performer → supplies u_id to /locationUNKNOWN until explicit value-flow is traced

Order → EXPOSES_ASSIGNED_PERFORMER_POSITION → PositionUNKNOWN
```

## 7. Gap Report

```text
G-25-01  select one concrete /location caller           OPEN
G-25-02  trace its u_id source                           OPEN
G-25-03  trace backend assigned-driver response field   OPEN
G-25-04  prove/reject value equality/derivation         OPEN
G-25-05  trace /location response into map rendering    OPEN
```

## 8. Следующий шаг

Теперь не нужно искать новые occurrences.

Нужно выбрать самый сильный concrete `/location` consumer из этого pass и сделать ручную data-flow трассу:

```text
caller
  ↓
u_id expression
  ↓
local state
  ↓
order response
  ↓
assigned driver field
```

После закрытия этой цепочки отдельно пройти:

```text
/location response
  ↓
frontend state
  ↓
map marker / driver position```

Это даст либо первое `CONFIRMED` межслойное отношение, либо доказанный `REJECTED`/`SOURCE_GAP`.