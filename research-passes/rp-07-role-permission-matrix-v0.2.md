# Backend Semantic Graph — Research Pass 07
# Role Permission Matrix v0.2

**Статус:** PROVISIONAL / PARTIALLY ANSWERED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-06 Role Permission Matrix v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Цель

Расширить RP-06 и собрать operation-local authorization evidence по всему доступному PHP source corpus, не превращая отсутствие проверки в `ALLOW` и не выводя бизнес-названия ролей из числовых ID.

## 2. Нормативное правило

В матрицу попадает только то, что подтверждается прямым production code:

```text
role check
   ↓
Evidence
   ↓
Claim
   ↓
Operation-local ALLOW/REJECT
```

Если операция не содержит найденного role check, статус остаётся `UNKNOWN`, а не `ALLOW`.

## 3. Найденные прямые role checks

### `registerUser` — `archive_17012026_1259/taxi/models/api.php:30`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectUser` — `archive_17012026_1259/taxi/models/api.php:661`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editUser` — `archive_17012026_1259/taxi/models/api.php:1056`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectCar` — `archive_17012026_1259/taxi/models/api.php:1990`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `controlCar` — `archive_17012026_1259/taxi/models/api.php:2145`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `controlCar` — `archive_17012026_1259/taxi/models/api.php:2169`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `createOrder` — `archive_17012026_1259/taxi/models/api.php:2849`

- role IDs, участвующие в локальной проверке: `[1, 4, 5]`
- явная ветка `wrong user role`: `YES`

### `createOrder` — `archive_17012026_1259/taxi/models/api.php:3471`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectActiveOrder` — `archive_17012026_1259/taxi/models/api.php:4481`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `selectActiveOrder` — `archive_17012026_1259/taxi/models/api.php:4545`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectProcessingOrder` — `archive_17012026_1259/taxi/models/api.php:4948`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `YES`

### `selectProcessingOrder` — `archive_17012026_1259/taxi/models/api.php:5003`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectProcessingOrder` — `archive_17012026_1259/taxi/models/api.php:5020`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectArchiveOrder` — `archive_17012026_1259/taxi/models/api.php:5608`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `selectArchiveOrder` — `archive_17012026_1259/taxi/models/api.php:5672`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `setDriver` — `archive_17012026_1259/taxi/models/api.php:6186`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `setDriver` — `archive_17012026_1259/taxi/models/api.php:6191`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `setCarIsArrived` — `archive_17012026_1259/taxi/models/api.php:6804`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `YES`

### `completeOrder` — `archive_17012026_1259/taxi/models/api.php:6901`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `completeOrder` — `archive_17012026_1259/taxi/models/api.php:6906`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `completeOrder` — `archive_17012026_1259/taxi/models/api.php:6949`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `cancelOrder` — `archive_17012026_1259/taxi/models/api.php:7055`

- role IDs, участвующие в локальной проверке: `[1, 2, 4, 5]`
- явная ветка `wrong user role`: `YES`

### `cancelOrder` — `archive_17012026_1259/taxi/models/api.php:7082`

- role IDs, участвующие в локальной проверке: `[1, 4, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `cancelOrder` — `archive_17012026_1259/taxi/models/api.php:7106`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `rateOrder` — `archive_17012026_1259/taxi/models/api.php:7401`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `rateOrder` — `archive_17012026_1259/taxi/models/api.php:7406`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7531`

- role IDs, участвующие в локальной проверке: `[1, 2, 4, 5]`
- явная ветка `wrong user role`: `YES`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7594`

- role IDs, участвующие в локальной проверке: `[1, 4, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7618`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7639`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7678`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectOrder` — `archive_17012026_1259/taxi/models/api.php:7928`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `confirmOrder` — `archive_17012026_1259/taxi/models/api.php:8186`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `startOrder` — `archive_17012026_1259/taxi/models/api.php:8253`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `YES`

### `editWaitingTime` — `archive_17012026_1259/taxi/models/api.php:8744`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `setCarUsed` — `archive_17012026_1259/taxi/models/api.php:8972`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `YES`

### `editOrder` — `archive_17012026_1259/taxi/models/api.php:9256`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `editOrder` — `archive_17012026_1259/taxi/models/api.php:9312`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectFavoriteUser` — `archive_17012026_1259/taxi/models/api.php:10662`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `addFavoriteUser` — `archive_17012026_1259/taxi/models/api.php:10804`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `removeFavoriteUser` — `archive_17012026_1259/taxi/models/api.php:10891`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectReferralUrl` — `archive_17012026_1259/taxi/models/api.php:10935`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectReferralUser` — `archive_17012026_1259/taxi/models/api.php:11086`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `setOrderTips` — `archive_17012026_1259/taxi/models/api.php:11206`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `YES`

### `setOrderTips` — `archive_17012026_1259/taxi/models/api.php:11211`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `createTrip` — `archive_17012026_1259/taxi/models/api.php:11373`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `YES`

### `createTrip` — `archive_17012026_1259/taxi/models/api.php:11392`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectTrip` — `archive_17012026_1259/taxi/models/api.php:12137`

- role IDs, участвующие в локальной проверке: `[1, 2, 4, 5]`
- явная ветка `wrong user role`: `YES`

### `selectTrip` — `archive_17012026_1259/taxi/models/api.php:12229`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectTrip` — `archive_17012026_1259/taxi/models/api.php:12242`

- role IDs, участвующие в локальной проверке: `[1, 2, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectTrip` — `archive_17012026_1259/taxi/models/api.php:12250`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editTrip` — `archive_17012026_1259/taxi/models/api.php:12886`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `YES`

### `editTrip` — `archive_17012026_1259/taxi/models/api.php:12982`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `offerOrder` — `archive_17012026_1259/taxi/models/api.php:13572`

- role IDs, участвующие в локальной проверке: `[1, 5]`
- явная ветка `wrong user role`: `YES`

### `createDropboxFile` — `archive_17012026_1259/taxi/models/api.php:13730`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectDropboxFile` — `archive_17012026_1259/taxi/models/api.php:14010`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editData` — `archive_17012026_1259/taxi/models/api.php:14346`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editData` — `archive_17012026_1259/taxi/models/api.php:16850`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectDataPrivate` — `archive_17012026_1259/taxi/models/api.php:17403`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectCart` — `archive_17012026_1259/taxi/models/api.php:17449`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `queryString` — `archive_17012026_1259/taxi/models/api.php:17892`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `queryString` — `archive_17012026_1259/taxi/models/api.php:17911`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `queryString` — `archive_17012026_1259/taxi/models/api.php:17936`

- role IDs, участвующие в локальной проверке: `[]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `writeTicket` — `archive_17012026_1259/taxi/models/api.php:18136`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `readTicket` — `archive_17012026_1259/taxi/models/api.php:18260`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectCartBlock` — `archive_17012026_1259/taxi/models/api.php:18624`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectTicket` — `archive_17012026_1259/taxi/models/api.php:18958`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `sendToTicketBuyerEmail` — `archive_17012026_1259/taxi/models/api.php:19126`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editTicket` — `archive_17012026_1259/taxi/models/api.php:19814`

- role IDs, участвующие в локальной проверке: `[4, 6]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editTicket` — `archive_17012026_1259/taxi/models/api.php:19862`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `queryTemplate` — `archive_17012026_1259/taxi/models/api.php:20153`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `queryTemplate` — `archive_17012026_1259/taxi/models/api.php:20196`

- role IDs, участвующие в локальной проверке: `[]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `includeTemplate` — `archive_17012026_1259/taxi/models/api.php:20284`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `translate` — `archive_17012026_1259/taxi/models/api.php:20306`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `statusCartBlock` — `archive_17012026_1259/taxi/models/api.php:20347`

- role IDs, участвующие в локальной проверке: `[2, 4]`
- явная ветка `wrong user role`: `YES`

### `statusCartBlock` — `archive_17012026_1259/taxi/models/api.php:20413`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `YES`

### `statusCartBlock` — `archive_17012026_1259/taxi/models/api.php:20463`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectInnerUser` — `archive_17012026_1259/taxi/models/api.php:20523`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `requestTemplate` — `archive_17012026_1259/taxi/models/api.php:20674`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectContact` — `archive_17012026_1259/taxi/models/api.php:20866`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `createContact` — `archive_17012026_1259/taxi/models/api.php:21264`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `editContact` — `archive_17012026_1259/taxi/models/api.php:21446`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `sendMessage` — `archive_17012026_1259/taxi/models/api.php:21838`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `createTask` — `archive_17012026_1259/taxi/models/api.php:22616`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `controlTask` — `archive_17012026_1259/taxi/models/api.php:22901`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectTask` — `archive_17012026_1259/taxi/models/api.php:22932`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `addTaskLog` — `archive_17012026_1259/taxi/models/api.php:23051`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getTaskLog` — `archive_17012026_1259/taxi/models/api.php:23184`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `setTaskStatus` — `archive_17012026_1259/taxi/models/api.php:23239`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `deleteDropboxFile` — `archive_17012026_1259/taxi/models/api.php:23390`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getDropboxFileData` — `archive_17012026_1259/taxi/models/api.php:23473`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getTicketData` — `archive_17012026_1259/taxi/models/api.php:23617`

- role IDs, участвующие в локальной проверке: `[1, 2, 4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getTicketData` — `archive_17012026_1259/taxi/models/api.php:23629`

- role IDs, участвующие в локальной проверке: `[1]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getTicketData` — `archive_17012026_1259/taxi/models/api.php:23639`

- role IDs, участвующие в локальной проверке: `[2]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `getTicketData` — `archive_17012026_1259/taxi/models/api.php:23650`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `checkTicket` — `archive_17012026_1259/taxi/models/api.php:24234`

- role IDs, участвующие в локальной проверке: `[4, 6]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `checkTicket` — `archive_17012026_1259/taxi/models/api.php:24245`

- role IDs, участвующие в локальной проверке: `[6]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectCheckTicketLog` — `archive_17012026_1259/taxi/models/api.php:24456`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `createPayment` — `archive_17012026_1259/taxi/models/api.php:24772`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `selectPayment` — `archive_17012026_1259/taxi/models/api.php:24910`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `createSubscription` — `archive_17012026_1259/taxi/models/api.php:25064`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `depositCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php:25381`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `withdrawCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php:25625`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `transferCurrencyAccount` — `archive_17012026_1259/taxi/models/api.php:25845`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `completeDeal` — `archive_17012026_1259/taxi/models/api.php:26437`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `cancelDeal` — `archive_17012026_1259/taxi/models/api.php:26613`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

### `confirmDeal` — `archive_17012026_1259/taxi/models/api.php:26784`

- role IDs, участвующие в локальной проверке: `[4]`
- явная ветка `wrong user role`: `NOT FOUND IN WINDOW`

## 4. Матрица

| Implementation | Function | Role IDs explicitly checked | Explicit rejection | Confidence |
|---|---|---:|---|---|

| `archive_17012026_1259/taxi/models/api.php:30` | `registerUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:661` | `selectUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:1056` | `editUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:1990` | `selectCar` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:2145` | `controlCar` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:2169` | `controlCar` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:2849` | `createOrder` | `[1, 4, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:3471` | `createOrder` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:4481` | `selectActiveOrder` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:4545` | `selectActiveOrder` | `[1, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:4948` | `selectProcessingOrder` | `[2, 4]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:5003` | `selectProcessingOrder` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:5020` | `selectProcessingOrder` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:5608` | `selectArchiveOrder` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:5672` | `selectArchiveOrder` | `[1, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6186` | `setDriver` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6191` | `setDriver` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6804` | `setCarIsArrived` | `[2]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6901` | `completeOrder` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6906` | `completeOrder` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:6949` | `completeOrder` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7055` | `cancelOrder` | `[1, 2, 4, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7082` | `cancelOrder` | `[1, 4, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7106` | `cancelOrder` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7401` | `rateOrder` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7406` | `rateOrder` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7531` | `selectOrder` | `[1, 2, 4, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7594` | `selectOrder` | `[1, 4, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7618` | `selectOrder` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7639` | `selectOrder` | `[2, 4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7678` | `selectOrder` | `[1, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:7928` | `selectOrder` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:8186` | `confirmOrder` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:8253` | `startOrder` | `[2]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:8744` | `editWaitingTime` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:8972` | `setCarUsed` | `[2]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:9256` | `editOrder` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:9312` | `editOrder` | `[1, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:10662` | `selectFavoriteUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:10804` | `addFavoriteUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:10891` | `removeFavoriteUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:10935` | `selectReferralUrl` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:11086` | `selectReferralUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:11206` | `setOrderTips` | `[1, 2, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:11211` | `setOrderTips` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:11373` | `createTrip` | `[2, 4]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:11392` | `createTrip` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12137` | `selectTrip` | `[1, 2, 4, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12229` | `selectTrip` | `[2, 4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12242` | `selectTrip` | `[1, 2, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12250` | `selectTrip` | `[1, 5]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12886` | `editTrip` | `[2, 4]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:12982` | `editTrip` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:13572` | `offerOrder` | `[1, 5]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:13730` | `createDropboxFile` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:14010` | `selectDropboxFile` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:14346` | `editData` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:16850` | `editData` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:17403` | `selectDataPrivate` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:17449` | `selectCart` | `[2, 4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:17892` | `queryString` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:17911` | `queryString` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:17936` | `queryString` | `[]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:18136` | `writeTicket` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:18260` | `readTicket` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:18624` | `selectCartBlock` | `[2, 4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:18958` | `selectTicket` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:19126` | `sendToTicketBuyerEmail` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:19814` | `editTicket` | `[4, 6]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:19862` | `editTicket` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20153` | `queryTemplate` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20196` | `queryTemplate` | `[]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20284` | `includeTemplate` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20306` | `translate` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20347` | `statusCartBlock` | `[2, 4]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20413` | `statusCartBlock` | `[4]` | `YES` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20463` | `statusCartBlock` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20523` | `selectInnerUser` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20674` | `requestTemplate` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:20866` | `selectContact` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:21264` | `createContact` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:21446` | `editContact` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:21838` | `sendMessage` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:22616` | `createTask` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:22901` | `controlTask` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:22932` | `selectTask` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23051` | `addTaskLog` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23184` | `getTaskLog` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23239` | `setTaskStatus` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23390` | `deleteDropboxFile` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23473` | `getDropboxFileData` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23617` | `getTicketData` | `[1, 2, 4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23629` | `getTicketData` | `[1]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23639` | `getTicketData` | `[2]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:23650` | `getTicketData` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:24234` | `checkTicket` | `[4, 6]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:24245` | `checkTicket` | `[6]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:24456` | `selectCheckTicketLog` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:24772` | `createPayment` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:24910` | `selectPayment` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:25064` | `createSubscription` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:25381` | `depositCurrencyAccount` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:25625` | `withdrawCurrencyAccount` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:25845` | `transferCurrencyAccount` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:26437` | `completeDeal` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:26613` | `cancelDeal` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |

| `archive_17012026_1259/taxi/models/api.php:26784` | `confirmDeal` | `[4]` | `NOT ESTABLISHED` | `CONFIRMED` |


## 5. Интерпретация


Например, проверка вида:

```php
$this->id_role != 2 && $this->id_role != 4
```

даёт прямое Evidence, что текущая операция отвергает role contexts, не удовлетворяющие условию, и что роли `2` и `4` являются допустимыми относительно этой конкретной проверки.

Это не означает:

```text
Role 2 = конкретная бизнес-роль
Role 4 = конкретная бизнес-роль
```

и не означает, что роли 2 и 4 разрешены для всех API.

Аналогично:

```text
нет role check
```

не превращается в:

```text
ALLOW_ALL
```

## 6. Role ID → business role

Полная таблица соответствия числового `id_role` бизнес-ролям этим проходом не устанавливается. Если mapping не найден непосредственно в configuration/source, он остаётся UNKNOWN.

## 7. `user_roles`

Подтверждено использование `taxi::$data['user_roles']` в role resolution и role-sensitive operations. Это Evidence существования role configuration, но не доказательство того, что вся эта структура является полной authorization policy.

## 8. `query_roles`

`query_roles` используется query-related code и передаётся в query processing. Его точная семантика и степень участия в authorization остаются отдельным Research Question.

## 9. Role-specific configuration

Наличие обращения к `role{N}.php` подтверждает зависимость runtime context от выбранной роли. Содержимое и семантика этих файлов должны быть исследованы отдельно, прежде чем считать их permission policy.

## 10. Что теперь можно считать подтверждённым


```text
Authentication
    ↓
Role Resolution
    ↓
Runtime API Role
    ↓
Operation-local role checks
    ↓
ALLOW / REJECT
```

Это подтверждённая архитектурная последовательность на уровне production code.

Но глобальной таблицы:

```text
Role × All Operations
```

пока нет.

## 11. Gap Report


```text
G-07-01
Complete Role × Operation matrix
STATUS: OPEN

G-07-02
Role ID → business role mapping
STATUS: OPEN

G-07-03
query_roles semantics
STATUS: OPEN

G-07-04
role{N}.php semantics
STATUS: OPEN
```

## 12. MCR result

`MCR = NO CHANGE`.

Текущая структура Semantic Graph достаточна. Нужна дополнительная Evidence, а не новый фундаментальный тип графа.

## 13. Следующий шаг

Следующий проход должен закрывать не всю матрицу сразу, а два наиболее ценных источника семантики: `role{N}.php` и `query_roles`. После этого operation-local checks можно сопоставить с реальными role capabilities и только затем строить клиентские subgraphs.
