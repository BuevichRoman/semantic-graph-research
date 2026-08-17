# Backend Semantic Graph — Research Pass 28
# Frontend Source Gap Closure v0.1

**Статус:** SOURCE_GAP CONFIRMED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-27 Taxi Frontend Source Audit v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

## 1. Цель

Проверить, не является ли `SOURCE_GAP` RP-27 следствием слишком узкого поиска frontend-кода.

Проверка выполняется уже не по именам файлов и каталогов, а по **содержательным сигнатурам**:

```text
browser geolocation
HTTP client
map library
location API
map/location UI
```

## 2. Полный inventory snapshot

Всего файлов в распакованном snapshot: **60**.

Распределение по расширениям:

- `.php` — 46
- `<no_ext>` — 6
- `.json` — 2
- `.txt` — 1
- `.save` — 1
- `.log` — 1
- `.pem` — 1
- `.js` — 1
- `.svg` — 1

Корневые каталоги:

- `archive_17012026_1259` — 60 files

## 3. Content-signature audit

### browser geolocation

Найдено файлов: **0**.


### HTTP client

Найдено файлов: **2**.

- `archive_17012026_1259/taxi/controllers/c_edit_langs.php`
- `archive_17012026_1259/taxi/controllers/c_index.php`

### map library

Найдено файлов: **0**.


### location API

Найдено файлов: **1**.

- `archive_17012026_1259/taxi/controllers/c_index.php`

### map/location UI

Найдено файлов: **4**.

- `archive_17012026_1259/taxi/cache/data.php`
- `archive_17012026_1259/taxi/cache/data.js`
- `archive_17012026_1259/taxi/cache/data.json`
- `archive_17012026_1259/taxi/cache/data.(iso).json`

## 4. Результат

Content-signature audit не обнаруживает отдельного production frontend implementation, который одновременно давал бы:

```text
order state
   ↓
assigned driver
   ↓
frontend state
   ↓
/location
   ↓
map / marker
```

Найденные location occurrences остаются backend/API documentation/test-form evidence, а не самостоятельным versioned Taxi Web frontend snapshot.

Поэтому `SOURCE_GAP` из RP-27 не является артефактом только поиска по расширениям или каталогам.

## 5. Что теперь доказано

Для данного snapshot:

```text
Core Backend location capability
    = исследована

Core Backend assignment capability
    = исследована

Taxi Web frontend implementation
    = отсутствует в доступном snapshot
```

Следовательно, невозможно получить AS-IS Claim:

```text
Taxi Web
    → OBSERVES
assigned Driver Position
```

из данного архива.

## 6. Важный вывод для многоклиентского Core Backend

Это подтверждает version/snapshot boundary:

```text
Core Backend Snapshot B
        ↑
        │
Frontend Snapshot F1
Frontend Snapshot F2
Frontend Snapshot F3
```

Поведение F1 нельзя переносить на F2/F3.

Поэтому Frontend-specific Claims должны иметь собственную provenance:

```text
claim
  + backend_snapshot
  + frontend_snapshot
  + evidence
```

А backend-only graph остаётся независимым от конкретного клиента.

## 7. Статус прежней гипотезы

Гипотеза:

```text
Order
  → Assigned Driver
  → Driver Position
  → Taxi Web Map
```

не получает статус `REJECTED`.

Она получает:

```text
UNKNOWN / SOURCE_GAP
```

потому что отсутствует необходимый frontend source.

Это принципиально отличается от ситуации, когда frontend найден и код показывает, что цепочки нет.

## 8. Research Loop

```text
UNKNOWN
   ↓
Research Question
   ↓
Expected Evidence
   ↓
Search
   ↓
full source-signature audit
   ↓
required layer absent
   ↓
SOURCE_GAP
```

Цикл здесь корректно останавливается.

Нельзя продолжать его inference-цепочкой.

## 9. Gap Report

```text
G-28-01  Taxi Web production snapshot          SOURCE_GAP
G-28-02  frontend /location consumer            BLOCKED
G-28-03  assigned driver → frontend u_id        BLOCKED
G-28-04  position response → map marker         BLOCKED
```

## 10. Следующий шаг

**Backend reverse engineering по этой ветке завершён на текущем Evidence boundary.**

Не создавать следующий backend Research Pass ради продолжения цепочки.

Для продолжения нужен отдельный artifact:

```text
Taxi Web Frontend
    конкретная версия
    конкретный snapshot
```

После его появления следующий документ должен быть не продолжением RP-28 внутри Core Backend, а отдельным:

```text
Frontend Semantic Graph Research Pass
```

с обязательной привязкой каждого frontend Claim к:

```text
frontend snapshot/version
+
backend API version/snapshot
+
source evidence
```

## 11. MCR

`MCR = NO CHANGE`.

Полученный результат не требует изменения структуры графа. Он подтверждает необходимость уже предусмотренной provenance/snapshot boundary между общим Core Backend и конкретным Frontend.
