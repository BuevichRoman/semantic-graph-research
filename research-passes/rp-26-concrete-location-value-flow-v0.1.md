# Backend Semantic Graph — Research Pass 26
# Concrete `/location` Value-Flow Trace v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-25 Concrete Frontend `/location` Data-Flow Trace v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Research Question

> Из какого конкретного значения формируется `u_id` в найденном frontend `/location` consumer и выводится ли оно из assigned performer/order state?

## 2. Найденные concrete location consumers

Количество concrete call sites: **2**.

### `archive_17012026_1259/taxi/controllers/c_index.php:2397` — `global`
Source range: `2317-2332`
```text
2317: 						Ключи полученного для "=" и "+" значения проверяются в зависимости от констант:	
2318: 							data.site_constants.b_options_valid_keys
2319: 							data.site_constants.b_options_edit_list_keys
2320: 							data.site_constants.b_options_edit_list_readonly
2321: 						или
2322: 							data.site_constants.c_options_valid_keys
2323: 							data.site_constants.c_options_edit_list_keys
2324: 							data.site_constants.c_options_edit_list_readonly
2325: 						Ответ сервера:
2326: 						{'code':'200','status':'success',
2327: 							"data":{
2328: 								"affected_fields":		[..],			список валидных ключей
2329: 								"forbidden_fields:		[..],			список невалидных ключей
2330: 								"wrong_data_fields":	[..]			список ключей с некорректными данными
2331: 							}
2332: 						}
```

### `archive_17012026_1259/taxi/controllers/c_index.php:2406` — `global`
Source range: `2326-2332`
```text
2326: 						{'code':'200','status':'success',
2327: 							"data":{
2328: 								"affected_fields":		[..],			список валидных ключей
2329: 								"forbidden_fields:		[..],			список невалидных ключей
2330: 								"wrong_data_fields":	[..]			список ключей с некорректными данными
2331: 							}
2332: 						}
```

## 3. Переменные, непосредственно участвующие в value-flow


## 4. Критерий доказательства

Нужна конкретная цепочка:

```text
assigned driver field
      ↓
frontend state / variable
      ↓
u_id
      ↓
/location
```

Если `u_id` задаётся константой, current-user ID или независимым источником, assignment → position bridge не подтверждается.

## 5. Фактический результат проверки call sites

Два найденных `/location` contexts находятся в `taxi/controllers/c_index.php` и являются **документацией/API test form**, а не production frontend data-flow consumer.

Фрагмент описывает:

```text
POST /api/v1/location
    ↓
Запись координат авторизированного пользователя в базу
```

и содержит форму с `latitude` / `longitude`.

В этом source context отсутствуют:

```text
order state
assigned driver ID
u_id → /location
frontend map state
```

Следовательно, найденные два contexts **не закрывают** G-26-01…G-26-04.

Это важная отрицательная находка: предыдущая формулировка «concrete `/location` consumers» была слишком сильной. Правильнее называть их `API documentation / test-form occurrences`.



Поэтому Semantic Graph сохраняет:

```text
Order → ASSIGNED_PERFORMER → User          CONFIRMED
User → HAS_CURRENT_POSITION → Position     CONFIRMED
Frontend → CALLS → /location               CONFIRMED
Assigned Performer → supplies u_id         UNKNOWN
Order → EXPOSES_ASSIGNED_PERFORMER_POSITION UNKNOWN
```

## 6. Backend side

Backend API documentation действительно показывает `u_id` как идентификатор авторизированного пользователя в token response, но это **не является доказательством**, что `u_id` для `/location` выбирается из assigned performer.

Более того, сама документация `/location` прямо описывает запись координат **авторизированного пользователя**, а не передачу произвольного `u_id`.

Поэтому нельзя строить:

```text
assigned Driver
    → u_id
    → POST /location
```

на основании этого документационного контекста.

## 7. Gap Report

```text
G-26-01  real production frontend /location consumer      OPEN
G-26-02  exact u_id assignment in frontend state           OPEN
G-26-03  assigned driver ID → u_id derivation              OPEN
G-26-04  /location response → map marker                    OPEN
G-26-05  determine whether location is user-self write     OPEN
```

Новый G-26-05 появился потому, что API documentation прямо указывает на запись координат **авторизированного пользователя**. Это может означать отдельный self-location pipeline, а не consumer path для просмотра позиции другого пользователя.

## 8. Следующий шаг

Не считать `c_index.php` frontend consumer.

Следующий pass должен искать **реальный клиентский код конкретного Taxi Web snapshot** (JS/TS/templates/assets), который фактически вызывает API для получения/отображения позиции.

Если такого кода в предоставленном snapshot нет, это должно быть зафиксировано как:

```text
SOURCE_GAP
    отсутствует production frontend source
```

и не компенсироваться документацией API.

Если frontend source найден, пройти backward data-flow:

```text
/location call
 ↓
u_id expression
 ↓
local state
 ↓
order response
 ↓
assigned driver
```

После этого отдельно пройти forward flow:

```text
/location response
 ↓
position state
 ↓
map marker
```

Цель:

```text
u_id
 ↓
source variable
 ↓
order response / assigned driver
```

После этого отдельно пройти forward flow:

```text
/location response
 ↓
position state
 ↓
map marker```