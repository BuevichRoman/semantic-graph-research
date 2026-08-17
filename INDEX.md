# Реестр документов

Полный список содержимого репозитория: 63 markdown-документа (62 из исходной выгрузки +
1 конвертированный из `.docx`) и 2 оригинала `.docx`.

Колонка **«Исходное имя»** — имя файла в выгрузке генератора от 17.08.2026. Нужна для
сверки: если генератор выдаст файл повторно, по ней видно, куда он ложится.

Пометка **`← актуальная`** отмечает свежую версию внутри цепочки ревизий. Определялась по
подзаголовку и статусу внутри документа, **не по имени файла** — в исходной выгрузке имена
вводили в заблуждение.

---

## Итог

| Файл | Статус | Что | Исходное имя |
|---|---|---|---|
| [`current-research-summary.md`](current-research-summary.md) | ← актуальная | Текущее кумулятивное состояние: нормативные документы v2.3, состав корпуса, подтверждённые результаты | *(добавлен при консолидации)* |
| [`research-summary.md`](research-summary.md) | **`HISTORICAL`** | Резюме ранней стадии: язык, дисциплина процесса, завершённые MCR, открытые вопросы, архитектура Semantic Graph Builder. Не является нормативным описанием v2.3 | `Semantic_Graph_Research_Summary.md` |

---

## `methodology/` — нормативный слой

| Файл | Версия | Что | Исходное имя |
|---|---|---|---|
| [`research-methodology-v2.3.md`](methodology/research-methodology-v2.3.md) | v2.3 ← актуальная | Нормативная методология исследования существующих систем | `Semantic_Graph_Research_Methodology_v2.3.md` |
| [`prototype-structures-v2.3.md`](methodology/prototype-structures-v2.3.md) | v2.3 ← актуальная | Спецификация протоструктур (3422 строки) | `Semantic_Graph_Prototype_Structures_v2.3.md` |
| [`prototype-structures-v2.2.md`](methodology/prototype-structures-v2.2.md) | v2.2 | Предыдущая ревизия | `Semantic_Graph_Prototype_Structures_v2.2.md` |
| [`prototype-structures-v2-rev2.md`](methodology/prototype-structures-v2-rev2.md) | v2 (2-я ревизия) | Расширенная v2; в заголовке по-прежнему «v2» | `Semantic_Graph_Prototype_Structures_v2-1.md` |
| [`prototype-structures-v2.md`](methodology/prototype-structures-v2.md) | v2 | Первая редакция v2 | `Semantic_Graph_Prototype_Structures_v2.md` |
| [`prototype-structures-v1-ru.md`](methodology/prototype-structures-v1-ru.md) | v1 | Предок всей линии: цепочка `SOURCE FACTS → … → CONTRACTS`. Конвертирован из `.docx` | *(добавлен при сборке репо)* |

Линия версий: `v1-ru` → `v2` → `v2-rev2` → `v2.2` → **`v2.3`**.

---

## `mcr/` — Model Change Records

Эксперименты над самим языком графа. Читаются по порядку — связная цепочка.
**MCR-06 в выгрузке отсутствует.**

| Файл | Статус | Вопрос | Исходное имя |
|---|---|---|---|
| [`mcr-01-order-lifecycle.md`](mcr/mcr-01-order-lifecycle.md) | эксперимент | Хватает ли `Frame / Relation / Fact` на реальный lifecycle заказа | `Semantic_Graph_MCR_01_Order_Lifecycle.md` |
| [`mcr-02-core-backend-fsm-transition.md`](mcr/mcr-02-core-backend-fsm-transition.md) | завершён | Переход FSM Core Backend | `Semantic_Graph_MCR_02_Core_Backend_FSM_Transition.md` |
| [`mcr-03-quantified-order-completion.md`](mcr/mcr-03-quantified-order-completion.md) | завершён | Правило завершения заказа с квантором | `Semantic_Graph_MCR_03_Quantified_Order_Completion.md` |
| [`mcr-04-set-cardinality-representation.md`](mcr/mcr-04-set-cardinality-representation.md) | завершён | Представление множества и кардинальности | `Semantic_Graph_MCR_04_Set_Cardinality_Representation.md` |
| [`mcr-05-universal-predicate-representation.md`](mcr/mcr-05-universal-predicate-representation.md) | завершён | Универсальный предикат | `Semantic_Graph_MCR_05_Universal_Predicate_Representation.md` |
| [`mcr-07-assertion-reification.md`](mcr/mcr-07-assertion-reification.md) | завершён | Реификация утверждения (`Fact` как объект `Relation`) | `Semantic_Graph_MCR_07_Assertion_Reification.md` |

---

## `research-passes/` — проходы по коду

Источники: `archive_17012026_1259_clear.zip` (Core Backend), `taxi-master.zip` (Taxi Frontend).

### RP-01…RP-07 — реконструкция и ролевая модель

| Файл | Статус | Тема | Исходное имя |
|---|---|---|---|
| [`rp-01-code-reconstruction-provenance-rev2.md`](research-passes/rp-01-code-reconstruction-provenance-rev2.md) | ← актуальная | Code-first реконструкция бэкенда, полный provenance (1799 строк) | `Backend_Semantic_Graph_Research_Pass_01_Code_v2_2_provenance_1.md` |
| [`rp-01-code-reconstruction-provenance.md`](research-passes/rp-01-code-reconstruction-provenance.md) | ревизия | То же, предыдущая ревизия (1701 строка) | `Backend_Semantic_Graph_Research_Pass_01_Code_v2.2_provenance.md` |
| [`rp-01-code-reconstruction.md`](research-passes/rp-01-code-reconstruction.md) | ревизия | Первая редакция, без provenance (1209 строк) | `Backend_Semantic_Graph_Research_Pass_01_Code_v2.2.md` |
| [`rp-02-authentication.md`](research-passes/rp-02-authentication.md) | `IN PROGRESS / PROVISIONAL` | Аутентификация, ранний проход (RU) | `Backend_Semantic_Graph_Research_Pass_02_Authentication_RU.md` |
| [`rp-02-taxiweb-core-backend-mapping.md`](research-passes/rp-02-taxiweb-core-backend-mapping.md) | `IN PROGRESS` | Сопоставление Taxi Web ↔ Core Backend. **Другая тема под тем же номером** | `Backend_Semantic_Graph_Research_Pass_02_TaxiWeb_CoreBackend_Mapping.md` |
| [`rp-03-authentication-deep-trace-v0.1.md`](research-passes/rp-03-authentication-deep-trace-v0.1.md) | `COMPLETED / PROVISIONAL` | Глубокая трассировка аутентификации | `Backend_Semantic_Graph_Research_Pass_03_Authentication_Deep_Trace.md` |
| [`rp-04-authorization-gate-deep-trace-v0.1.md`](research-passes/rp-04-authorization-gate-deep-trace-v0.1.md) | `PARTIALLY ANSWERED / PROVISIONAL` | Authorization gate | `Backend_Semantic_Graph_Research_Pass_04_Authorization_Gate_Deep.md` |
| [`rp-05-role-resolution-trace-v0.2.md`](research-passes/rp-05-role-resolution-trace-v0.2.md) | `PARTIALLY ANSWERED` ← актуальная | Разрешение роли. **В выгрузке лежал как `(2)`** | `Backend_Semantic_Graph_Research_Pass_05_Role_Resolution_Trace_v0 (2).md` |
| [`rp-05-role-resolution-trace-v0.1.md`](research-passes/rp-05-role-resolution-trace-v0.1.md) | `PARTIALLY ANSWERED / SOURCE-GAP` | Предыдущая версия | `Backend_Semantic_Graph_Research_Pass_05_Role_Resolution_Trace_v0.md` |
| [`rp-06-role-permission-matrix-v0.1.md`](research-passes/rp-06-role-permission-matrix-v0.1.md) | `PARTIALLY ANSWERED / PROVISIONAL` | Матрица роль × право, v0.1 | `Backend_Semantic_Graph_Research_Pass_06_Role_Permission_Matrix_v0.md` |
| [`rp-07-role-permission-matrix-v0.2.md`](research-passes/rp-07-role-permission-matrix-v0.2.md) | `PROVISIONAL / PARTIALLY ANSWERED` | Та же матрица, v0.2 — **отдельный проход, а не ревизия RP-06** | `Backend_Semantic_Graph_Research_Pass_07_Role_Permission_Matrix_v0.md` |

### RP-08…RP-13 — `query_roles`

Цепочка закрывается на RP-13: `query_roles` участвует в authorization decision, а не только ограничивает scope.

| Файл | Статус | Тема | Исходное имя |
|---|---|---|---|
| [`rp-08-role-config-and-query-roles-v0.1.md`](research-passes/rp-08-role-config-and-query-roles-v0.1.md) | `PROVISIONAL` | Конфигурация ролей и `query_roles`. **~1.2 МБ — сырые дампы `taxi/cache/data.json`** | `Backend_Semantic_Graph_Research_Pass_08_Role_Config_and_Query_Roles.md` |
| [`rp-09-query-roles-deep-trace-v0.1.md`](research-passes/rp-09-query-roles-deep-trace-v0.1.md) | `PARTIALLY ANSWERED` | Глубокая трассировка. **~1.2 МБ, те же дампы** | `Backend_Semantic_Graph_Research_Pass_09_Query_Roles_Deep_Trace_v0.md` |
| [`rp-10-query-roles-consumer-trace-v0.1.md`](research-passes/rp-10-query-roles-consumer-trace-v0.1.md) | `PARTIALLY ANSWERED` | Кто потребляет `query_roles` | `Backend_Semantic_Graph_Research_Pass_10_Query_Roles_Consumer_Trace.md` |
| [`rp-11-query-roles-result-boundary-v0.1.md`](research-passes/rp-11-query-roles-result-boundary-v0.1.md) | `PROVISIONAL` | Граница результата | `Backend_Semantic_Graph_Research_Pass_11_Query_Roles_Result_Boundary.md` |
| [`rp-12-query-roles-sql-result-trace-v0.1.md`](research-passes/rp-12-query-roles-sql-result-trace-v0.1.md) | `PROVISIONAL` | SQL → результат | `Backend_Semantic_Graph_Research_Pass_12_Query_Roles_SQL_Result_Trace.md` |
| [`rp-13-query-roles-authorization-boundary-v0.1.md`](research-passes/rp-13-query-roles-authorization-boundary-v0.1.md) | **`CONFIRMED`** | Граница авторизации — итог цепочки | `Backend_Semantic_Graph_Research_Pass_13_Query_Roles_Authorization.md` |

### RP-14…RP-21 — роли, авторизация, `setDriver`

| Файл | Статус | Тема | Исходное имя |
|---|---|---|---|
| [`rp-14-role-id-semantic-mapping-v0.1.md`](research-passes/rp-14-role-id-semantic-mapping-v0.1.md) | `PARTIALLY ANSWERED / EVIDENCE-GROUNDED` | Семантика role ID | `Backend_Semantic_Graph_Research_Pass_14_Role_ID_Semantic_Mapping.md` |
| [`rp-15-role-id-mapping-evidence-v0.1.md`](research-passes/rp-15-role-id-mapping-evidence-v0.1.md) | `PARTIALLY ANSWERED / EVIDENCE-GROUNDED` | Доказательная база маппинга | `Backend_Semantic_Graph_Research_Pass_15_Role_ID_Mapping_Evidence.md` |
| [`rp-16-role-id-mapping-confirmed-v0.1.md`](research-passes/rp-16-role-id-mapping-confirmed-v0.1.md) | **`CONFIRMED`** | Маппинг role ID подтверждён | `Backend_Semantic_Graph_Research_Pass_16_Role_ID_Mapping_Confirmed.md` |
| [`rp-17-role-operation-matrix-v0.1.md`](research-passes/rp-17-role-operation-matrix-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Матрица роль × операция | `Backend_Semantic_Graph_Research_Pass_17_Role_Operation_Matrix_v0.md` |
| [`rp-18-authorization-control-flow-normalization-v0.1.md`](research-passes/rp-18-authorization-control-flow-normalization-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Нормализация control-flow авторизации (11670 строк) | `Backend_Semantic_Graph_Research_Pass_18_Authorization_Control_Flow.md` |
| [`rp-19-reject-branch-normalization-v0.1.md`](research-passes/rp-19-reject-branch-normalization-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Нормализация reject-ветки | `Backend_Semantic_Graph_Research_Pass_19_Reject_Branch_Normalization.md` |
| [`rp-20-authorization-helper-trace-v0.1.md`](research-passes/rp-20-authorization-helper-trace-v0.1.md) | `CONFIRMED / PARTIALLY RESOLVED` | Трассировка authorization helper | `Backend_Semantic_Graph_Research_Pass_20_Authorization_Helper_Trace.md` |
| [`rp-21-setdriver-full-trace-v0.1.md`](research-passes/rp-21-setdriver-full-trace-v0.1.md) | **`CONFIRMED`** | `setDriver`: авторизация и предусловия целиком | `Backend_Semantic_Graph_Research_Pass_21_SetDriver_Full_Trace_v0.md` |

### RP-22…RP-30 — позиция водителя и `/location`

Цепочка закрывается результатом `CONFIRMED NEGATIVE`: связи, которую искали, в системе нет.

| Файл | Статус | Тема | Исходное имя |
|---|---|---|---|
| [`rp-22-assignment-to-position-trace-v0.1.md`](research-passes/rp-22-assignment-to-position-trace-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Assignment → User → Position | `Backend_Semantic_Graph_Research_Pass_22_Assignment_to_Position_Trace.md` |
| [`rp-23-assignment-position-exposure-v0.1.md`](research-passes/rp-23-assignment-position-exposure-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Где позиция выходит наружу | `Backend_Semantic_Graph_Research_Pass_23_Assignment_Position_Exposure.md` |
| [`rp-24-order-response-to-location-bridge-v0.1.md`](research-passes/rp-24-order-response-to-location-bridge-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Мост order response → `/location` | `Backend_Semantic_Graph_Research_Pass_24_Order_Response_to_Location.md` |
| [`rp-25-frontend-location-dataflow-v0.1.md`](research-passes/rp-25-frontend-location-dataflow-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Data-flow `/location` на фронте | `Backend_Semantic_Graph_Research_Pass_25_Frontend_Location_Dataflow.md` |
| [`rp-26-concrete-location-value-flow-v0.1.md`](research-passes/rp-26-concrete-location-value-flow-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Конкретный value-flow | `Backend_Semantic_Graph_Research_Pass_26_Concrete_Location_Value.md` |
| [`rp-27-taxi-frontend-source-audit-v0.1.md`](research-passes/rp-27-taxi-frontend-source-audit-v0.1.md) | `SOURCE AUDIT` | Аудит исходников фронта | `Backend_Semantic_Graph_Research_Pass_27_Taxi_Frontend_Source_Audit.md` |
| [`rp-28-frontend-source-gap-closure-v0.1.md`](research-passes/rp-28-frontend-source-gap-closure-v0.1.md) | `SOURCE_GAP CONFIRMED` | Пробел в исходниках подтверждён как факт | `Backend_Semantic_Graph_Research_Pass_28_Frontend_Source_Gap_Closure.md` |
| [`rp-29-taxi-frontend-location-trace-v0.1.md`](research-passes/rp-29-taxi-frontend-location-trace-v0.1.md) | `PARTIALLY CONFIRMED` | Location / назначенный водитель на фронте | `Backend_Semantic_Graph_Research_Pass_29_Taxi_Frontend_Location_Trace.md` |
| [`rp-30-driver-position-passenger-map-v0.1.md`](research-passes/rp-30-driver-position-passenger-map-v0.1.md) | **`CONFIRMED NEGATIVE / AS-IS`** | Позиция водителя → карта пассажира: потребителя нет | `Backend_Semantic_Graph_Research_Pass_30_Driver_Position_Passenger.md` |

### RP-31…RP-37 — аутентификация фронт ↔ бэкенд

| Файл | Статус | Тема | Исходное имя |
|---|---|---|---|
| [`rp-31-taxi-frontend-authentication-trace-v0.1.md`](research-passes/rp-31-taxi-frontend-authentication-trace-v0.1.md) | `PROVISIONAL / EVIDENCE-GROUNDED` | Аутентификация на фронте | `Backend_Semantic_Graph_Research_Pass_31_Taxi_Frontend_Authentication.md` |
| [`rp-32-taxi-frontend-auth-api-state-v0.1.md`](research-passes/rp-32-taxi-frontend-auth-api-state-v0.1.md) | `EVIDENCE-GROUNDED / PROVISIONAL` | API → состояние | `Backend_Semantic_Graph_Research_Pass_32_Taxi_Frontend_Auth_API_State.md` |
| [`rp-33-taxi-frontend-auth-concrete-flow-v0.1.md`](research-passes/rp-33-taxi-frontend-auth-concrete-flow-v0.1.md) | `EVIDENCE-GROUNDED / PROVISIONAL` | Конкретный flow. **API-ключ из фронтенда санитизирован** (`<REDACTED>`) | `Backend_Semantic_Graph_Research_Pass_33_Taxi_Frontend_Auth_Concrete.md` |
| [`rp-34-taxi-frontend-login-endpoint-v0.1.md`](research-passes/rp-34-taxi-frontend-login-endpoint-v0.1.md) | `EVIDENCE-GROUNDED / PROVISIONAL` | Login endpoint | `Backend_Semantic_Graph_Research_Pass_34_Taxi_Frontend_Login_Endpoint.md` |
| [`rp-35-taxi-auth-frontend-backend-trace-v0.2.md`](research-passes/rp-35-taxi-auth-frontend-backend-trace-v0.2.md) | `EVIDENCE-GROUNDED / PROVISIONAL` | Фронт ↔ Core Backend, v0.2. **API-ключ санитизирован** (`<REDACTED>`) | `Backend_Semantic_Graph_Research_Pass_35_Taxi_Auth_Frontend_Backend.md` |
| [`rp-36-taxi-auth-exact-endpoint-match-v0.1.md`](research-passes/rp-36-taxi-auth-exact-endpoint-match-v0.1.md) | `EVIDENCE-GROUNDED / PROVISIONAL` | Точное сопоставление route | `Backend_Semantic_Graph_Research_Pass_36_Taxi_Auth_Exact_Endpoint.md` |
| [`rp-37-authentication-credential-value-flow-v0.4.md`](research-passes/rp-37-authentication-credential-value-flow-v0.4.md) | **`CONFIRMED`** ← актуальная | Value-flow credentials, v0.4. **В выгрузке лежал как `(2)` — самый свежий документ корпуса с самым невнятным именем** | `Backend_Semantic_Graph_Research_Pass_37_Authentication_Credential (2).md` |
| [`rp-37-authentication-credential-value-flow-v0.3.md`](research-passes/rp-37-authentication-credential-value-flow-v0.3.md) | `EVIDENCE-GROUNDED / RECONCILIATION` | Предыдущая версия. **API-ключ санитизирован** (`<REDACTED>`) | `Backend_Semantic_Graph_Research_Pass_37_Authentication_Credential.md` |
| [`rp-37-authentication-reconciliation-v0.1.md`](research-passes/rp-37-authentication-reconciliation-v0.1.md) | `EVIDENCE-GROUNDED / RECONCILIATION` | Сведение расхождений (10581 строка). **API-ключ санитизирован** (`<REDACTED>`) | `Backend_Semantic_Graph_Research_Pass_37_Authentication_Reconciliation.md` |

---

## `experiments/` — разовые проверки

| Файл | Статус | Что | Исходное имя |
|---|---|---|---|
| [`first-crystallization-experiment-v0.1.md`](experiments/first-crystallization-experiment-v0.1.md) | экспериментальная кристаллизация | Первая кристаллизация языка по итогам MCR-01…07 | `Semantic_Graph_First_Crystallization_Experiment_v0.1.md` |
| [`first-crystallization-experiment-v0.1-variant.md`](experiments/first-crystallization-experiment-v0.1-variant.md) | экспериментальная первая кристаллизация | Другой вариант той же кристаллизации: иная структура изложения, начинается с «Назначение» | `Semantic_Graph_First_Crystallization_Experiment_v0.1-2.md` |
| [`v1-authentication-end-to-end-crystallization-v0.1.md`](experiments/v1-authentication-end-to-end-crystallization-v0.1.md) | экспериментальный end-to-end pass | Кристаллизация аутентификации от начала до конца | `Semantic_Graph_V1_Authentication_End_to_End_Crystallization_v0_1.md` |
| [`targeted-backend-rule-scan-v0.1.md`](experiments/targeted-backend-rule-scan-v0.1.md) | исследовательский проход | Точечный поиск правил в бэкенде | `Semantic_Graph_Targeted_Backend_Rule_Scan_v0.1.md` |
| [`source-level-rule-scan-v0.2.md`](experiments/source-level-rule-scan-v0.2.md) | фактический source-level scan | Скан правил по исходникам Core Backend | `Semantic_Graph_Source_Level_Rule_Scan_v0.2.md` |
| [`quantified-rule-prevalence-check-v0.1.md`](experiments/quantified-rule-prevalence-check-v0.1.md) | исследовательский проход | Насколько часто встречаются правила с квантором | `Semantic_Graph_Quantified_Rule_Prevalence_Check_v0.1.md` |
| [`structure-historical-reconstruction.md`](experiments/structure-historical-reconstruction.md) | историческая реконструкция | Как менялась структура графа / Project IR | `Semantic_Graph_Structure_Historical_Reconstruction.md` |

---

## `source-docx/` — оригиналы

| Файл | Что | Исходное имя |
|---|---|---|
| [`prototype-structures-v2-complete.docx`](source-docx/prototype-structures-v2-complete.docx) | `.docx`-форма `methodology/prototype-structures-v2.md`. Содержимое совпадает | `Semantic_Graph_Prototype_Structures_v2_complete.docx` |
| [`protostructures-methodology-ru-v1.docx`](source-docx/protostructures-methodology-ru-v1.docx) | Самый ранний документ линии. Markdown-версия — `methodology/prototype-structures-v1-ru.md` | `Протоструктуры_семантически_фреймового_графа_и_методика.docx` |

---

## Что не перенесено

| Исходный файл | Почему |
|---|---|
| `Semantic_Graph_First_Crystallization_Experiment_v0.1-1.md` | Побайтово идентичен `Semantic_Graph_First_Crystallization_Experiment_v0.1.md` (совпадает md5). Не версия и не вариант — точная копия, ничего не теряется. |

Всё остальное из выгрузки перенесено без изменения содержимого.
