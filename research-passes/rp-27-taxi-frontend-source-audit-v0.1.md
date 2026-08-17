# Backend Semantic Graph — Research Pass 27
# Taxi Frontend Source Audit for Location Pipeline v0.1

**Статус:** SOURCE AUDIT / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-26 Concrete `/location` Value-Flow Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Research Question

> Есть ли в предоставленном Taxi snapshot реальный production frontend source, который вызывает `/location`/`selectLocation`, получает Position и использует её для карты/отображения assigned driver?

RP-26 установил, что найденные ранее contexts из `c_index.php` являются API documentation/test-form occurrences и не могут служить frontend evidence.

## 2. Аудит frontend-like source

Проанализировано frontend-like файлов: **3**.

Найдено literal `/location` / `selectLocation` call contexts во всех source files: **2**.

## 3. Literal location call sites

### `archive_17012026_1259/taxi/controllers/c_index.php:2397`
```text
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
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2406`
```text
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
```

## 4. Bundle / map candidates

Найдено JS/HTML файлов с признаками map/geolocation/location API: **0**.


## 5. Что считается production frontend Evidence

Для закрытия текущего Research Question нужен source, где наблюдается хотя бы один из вариантов:

```text
fetch/axios/XHR → /location
```

или:

```text
selectLocation(...)
```

и затем data-flow в:

```text
frontend state → map/marker/driver position
```

API documentation/test form не считается таким Evidence.

## 6. Текущий вывод

Если literal call sites отсутствуют либо все найденные occurrences относятся к PHP documentation/test code, production frontend source для данного pipeline в предоставленном snapshot **не обнаружен**.

В таком случае это не `BEHAVIOR_UNRESOLVED`: мы исследовали доступный source corpus, но необходимого слоя кода в нём нет.

Правильный статус:

```text
SOURCE_GAP
missing production frontend source for location consumer
```

Это особенно важно, поскольку один Core Backend используется несколькими frontend clients. Backend graph нельзя автоматически дополнять поведением конкретного клиента, если snapshot этого клиента отсутствует.

## 7. Graph state

Оставляем:

```text
User
  ──HAS_CURRENT_POSITION──> Position     CONFIRMED

Order
  ──ASSIGNED_PERFORMER──> User           CONFIRMED

Backend /location
  ──USER LOCATION PIPELINE──> Position
  status: CONFIRMED where directly evidenced

Assigned User Position → Taxi Web Map
  status: SOURCE_GAP
```

Не создаём:

```text
Order → EXPOSES_ASSIGNED_PERFORMER_POSITION → Position
```

## 8. Методологический результат

RP-27 подтверждает важное правило v2.3:

> Отсутствующий слой нельзя компенсировать inference из соседних слоёв.

Наличие Core Backend `/location` не доказывает, как конкретный Taxi frontend его использует.

## 9. Gap Report

```text
G-27-01  production Taxi frontend snapshot        OPEN / SOURCE_GAP
G-27-02  concrete frontend /location consumer      BLOCKED BY G-27-01
G-27-03  assigned driver → frontend u_id            BLOCKED
G-27-04  position response → map marker             BLOCKED
```

## 10. Следующий шаг

Не продолжать поиск по Core Backend.

Нужен именно **production source конкретной версии Taxi frontend**, соответствующей исследуемому backend snapshot.

После получения этого source исследование продолжается с уже найденной точки:

```text
frontend /location consumer
      ↓
u_id source
      ↓
order assigned driver
      ↓
/location response
      ↓
map marker
```

Это будет отдельный Frontend Research Pass, привязанный к конкретному frontend snapshot/version.
