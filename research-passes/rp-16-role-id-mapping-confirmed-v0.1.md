# Backend Semantic Graph — Research Pass 16
# Role ID Mapping — Confirmed v0.1

**Статус:** CONFIRMED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-15 Role ID Mapping Evidence v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`

---

## 1. Исследовательский вопрос

> Какой business meaning имеют числовые `id_role` в Core Backend?

RP-15 оставил этот вопрос открытым, поскольку использование `id_role` в authorization checks само по себе не доказывает название роли.

RP-16 нашёл прямой authoritative mapping в:

```text
taxi/cache/data.php
```

в конфигурационном объекте:

```text
user_roles
```

---

## 2. Прямое доказательство

Источник:

```text
archive_17012026_1259/taxi/cache/data.php
```

Фрагмент:

```text
20369:   'user_roles' => 
20370:   array (
20371:     1 => 
20372:     array (
20373:       'ru' => 'Клиент',
20374:       'en' => 'Client',
20375:       'ar' => NULL,
20376:       'fr' => NULL,
20377:       'es' => NULL,
20378:     ),
20379:     2 => 
20380:     array (
20381:       'ru' => 'Водитель',
20382:       'en' => 'Driver',
20383:       'ar' => NULL,
20384:       'fr' => NULL,
20385:       'es' => NULL,
20386:     ),
20387:     4 => 
20388:     array (
20389:       'ru' => 'Администратор',
20390:       'en' => 'Administrator',
20391:       'ar' => NULL,
20392:       'fr' => NULL,
20393:       'es' => NULL,
20394:     ),
20395:     5 => 
20396:     array (
20397:       'ru' => 'Агент',
20398:       'en' => 'Agent',
20399:       'ar' => NULL,
20400:       'fr' => NULL,
20401:       'es' => NULL,
20402:     ),
20403:     6 => 
20404:     array (
20405:       'ru' => 'Билетер',
20406:       'en' => 'Usher',
20407:       'ar' => NULL,
20408:       'fr' => NULL,
20409:       'es' => NULL,
20410:     ),
20411:     10 => 
20412:     array (
20413:       'ru' => 'Билетер с расширенными полномочиями',
20414:       'en' => 'Usher with extended powers',
20415:       'ar' => NULL,
20416:       'fr' => NULL,
20417:       'es' => NULL,
20418:     ),
20419:   ),
```

Здесь структура однозначна:

```text
user_roles
    ├── 1 → Client
    ├── 2 → Driver
    ├── 4 → Administrator
    ├── 5 → Agent
    ├── 6 → Usher
    └── 10 → Usher with extended powers
```

Это не inference по использованию роли.

Это непосредственная конфигурационная декларация:

```text
Role ID
    ↔
Role name
```

Confidence:

```text
CONFIRMED
```

---

## 3. Подтверждённая таблица Role ID

| Role ID | Русское имя | English name | Status |
|---:|---|---|---|
| 1 | Клиент | Client | CONFIRMED |
| 2 | Водитель | Driver | CONFIRMED |
| 4 | Администратор | Administrator | CONFIRMED |
| 5 | Агент | Agent | CONFIRMED |
| 6 | Билетер | Usher | CONFIRMED |
| 10 | Билетер с расширенными полномочиями | Usher with extended powers | CONFIRMED |

---

## 4. Важное ограничение scope

Это mapping для **данного backend snapshot**.

Он не утверждает, что:

```text
role 2 = Driver
```

будет неизменно во всех будущих версиях Core Backend.

Правильная provenance-модель:

```text
Role ID 2
    ──IS_NAMED_AS──>
Driver
    evidence:
    taxi/cache/data.php
    snapshot:
    archive_17012026_1259
```

Если в новой версии configuration изменится, появится новый Claim / Snapshot.

---

## 5. Связь с предыдущими RP

Теперь можно соединить два ранее независимых вида Evidence.

### RP-13

Код доказал:

```text
role 2
    → query access
```

### RP-16

Configuration доказал:

```text
role 2
    → Driver
```

Следовательно, теперь допустим более сильный semantic Claim:

```text
Driver
    → CAN_ACCESS
    → Query API
```

при условии, что relation сохраняет оба Evidence.

Аналогично:

```text
role 4
    → Administrator
    → allowed by query_roles='4'
    → Query API
```

---

## 6. Role Permission Matrix получает семантику

Теперь числовая матрица может быть переведена:

```text
Role ID 2
    ↓
Driver
    ↓
/query
    ↓
ALLOWED
```

и:

```text
Role ID 4
    ↓
Administrator
    ↓
/query
    ↓
ALLOWED
```

При этом:

```text
role 1 = Client
```

не означает автоматически:

```text
Client → /query = REJECT
```

если для конкретной версии `query_roles` это не подтверждено отдельным Evidence.

Это важное правило: **mapping роли и permission relation остаются двумя отдельными Claims.**

---

## 7. Дополнительное подтверждение через production usage

RP-13 уже установил:

```text
query_roles = '4'
```

в текущем snapshot.

RP-16 теперь даёт:

```text
4 = Administrator
```

Поэтому для текущей версии backend можно уверенно сформулировать:

```text
Administrator
    → allowed role for /query
```

Confidence:

```text
CONFIRMED
```

---

## 8. Role ID 2

Для `role = 2` ранее найдено production usage:

```text
id_role = '2'
```

совместно с:

```text
id_verification_status = '2'
```

в выборке водителей.

Теперь mapping:

```text
2 = Driver
```

подтверждён конфигурацией.

Поэтому можно связать:

```text
Driver
    → requires verification state 2
    → for relevant driver operation
```

Но это **не означает**, что verification state 2 является общей предпосылкой для всех Driver operations.

---

## 9. Graph update

Добавляются provenance-bearing claims:

```text
Claim:
Role(1) IS_NAMED_AS Client

Claim:
Role(2) IS_NAMED_AS Driver

Claim:
Role(4) IS_NAMED_AS Administrator

Claim:
Role(5) IS_NAMED_AS Agent

Claim:
Role(6) IS_NAMED_AS Usher

Claim:
Role(10) IS_NAMED_AS Usher with extended powers
```

Все:

```text
confidence = CONFIRMED
```

---

## 10. Gap Report

Закрыто:

```text
G-14-01  Role ID 2 → business role
CLOSED

G-14-02  Role ID 4 → business role
CLOSED

G-14-03  Role ID 5 → business role
CLOSED

G-14-04  Role ID 6 → business role
CLOSED
```

Остаётся:

```text
G-14-05  Complete Role × Operation matrix
OPEN
```

---

## 11. Методологический результат

Это хороший пример того, зачем Semantic Graph разделяет:

```text
Source Fact
    ↓
Claim
```

и не смешивает разные виды Evidence.

У нас были:

```text
Code:
id_role == 2
```

и отдельно:

```text
Configuration:
2 → Driver
```

Только после соединения двух Claims мы получили:

```text
Driver
    → protected operation
```

без догадки о роли.

---

## 12. Следующий шаг

Теперь Role ID mapping больше не является Research Question.

Следующий проход должен использовать подтверждённую таблицу:

```text
1 Client
2 Driver
4 Administrator
5 Agent
6 Usher
10 Usher with extended powers
```

и вернуться к накопленным operation-local role checks, чтобы построить первую **семантическую Role × Operation Matrix** с provenance каждого разрешения/запрета.
