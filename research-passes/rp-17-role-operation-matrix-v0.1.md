# Backend Semantic Graph — Research Pass 17
# Role × Operation Semantic Matrix v0.1

**Статус:** PROVISIONAL / EVIDENCE-GROUNDED  
**Методология:** Semantic Graph Research Methodology v2.3  
**Предшествующий проход:** RP-16 Role ID Mapping — Confirmed v0.1  
**Источник:** полный `archive_17012026_1259_clear.zip`  

## 1. Цель
Перевести operation-local role checks в семантическую матрицу `Business Role × Protected Operation`, не превращая отсутствие проверки в ALLOW.

## 2. Confirmed Role Mapping
| ID | Business Role |
|---:|---|
| 1 | Client |
| 2 | Driver |
| 4 | Administrator |
| 5 | Agent |
| 6 | Usher |
| 10 | Usher with extended powers |

Источник: `taxi/cache/data.php`. Все соответствия CONFIRMED.

## 3. Найденные operation-local role checks
Найдено direct `$this->id_role` comparison contexts: **161**.

| Function | Source | Role IDs | Evidence of rejection | Confidence |
|---|---|---|---|---|
| `registerUser` | `archive_17012026_1259/taxi/models/api.php:30` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `registerUser` | `archive_17012026_1259/taxi/models/api.php:37` | 4 (Administrator) | YES | CONFIRMED |
| `selectUser` | `archive_17012026_1259/taxi/models/api.php:661` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectUser` | `archive_17012026_1259/taxi/models/api.php:685` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `editUser` | `archive_17012026_1259/taxi/models/api.php:1056` | 4 (Administrator) | YES | CONFIRMED |
| `editUser` | `archive_17012026_1259/taxi/models/api.php:1379` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCar` | `archive_17012026_1259/taxi/models/api.php:1990` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCar` | `archive_17012026_1259/taxi/models/api.php:1997` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `controlCar` | `archive_17012026_1259/taxi/models/api.php:2145` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `controlCar` | `archive_17012026_1259/taxi/models/api.php:2169` | 4 (Administrator) | YES | CONFIRMED |
| `controlCar` | `archive_17012026_1259/taxi/models/api.php:2188` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `controlCar` | `archive_17012026_1259/taxi/models/api.php:2200` | 4 (Administrator) | YES | CONFIRMED |
| `controlCar` | `archive_17012026_1259/taxi/models/api.php:2225` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `createOrder` | `archive_17012026_1259/taxi/models/api.php:2849` | 1 (Client) | YES | CONFIRMED |
| `createOrder` | `archive_17012026_1259/taxi/models/api.php:2849` | 4 (Administrator) | YES | CONFIRMED |
| `createOrder` | `archive_17012026_1259/taxi/models/api.php:2849` | 5 (Agent) | YES | CONFIRMED |
| `createOrder` | `archive_17012026_1259/taxi/models/api.php:3471` | 4 (Administrator) | YES | CONFIRMED |
| `createOrder` | `archive_17012026_1259/taxi/models/api.php:3990` | 4 (Administrator) | YES | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4481` | 1 (Client) | YES | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4481` | 2 (Driver) | YES | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4481` | 5 (Agent) | YES | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4545` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4545` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4775` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4775` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4818` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4818` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4852` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectActiveOrder` | `archive_17012026_1259/taxi/models/api.php:4852` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:4948` | 2 (Driver) | YES | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:4948` | 4 (Administrator) | YES | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5003` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5020` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5067` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5356` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5512` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5524` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectProcessingOrder` | `archive_17012026_1259/taxi/models/api.php:5555` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5608` | 1 (Client) | YES | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5608` | 2 (Driver) | YES | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5608` | 5 (Agent) | YES | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5672` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5672` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5911` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:5911` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6011` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6011` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6053` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6053` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6084` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6084` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6094` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectArchiveOrder` | `archive_17012026_1259/taxi/models/api.php:6094` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `setDriver` | `archive_17012026_1259/taxi/models/api.php:6186` | 1 (Client) | YES | CONFIRMED |
| `setDriver` | `archive_17012026_1259/taxi/models/api.php:6186` | 2 (Driver) | YES | CONFIRMED |
| `setDriver` | `archive_17012026_1259/taxi/models/api.php:6186` | 5 (Agent) | YES | CONFIRMED |
| `setDriver` | `archive_17012026_1259/taxi/models/api.php:6191` | 1 (Client) | YES | CONFIRMED |
| `setDriver` | `archive_17012026_1259/taxi/models/api.php:6191` | 5 (Agent) | YES | CONFIRMED |
| `setCarIsArrived` | `archive_17012026_1259/taxi/models/api.php:6804` | 2 (Driver) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6901` | 1 (Client) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6901` | 2 (Driver) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6901` | 5 (Agent) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6906` | 1 (Client) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6906` | 5 (Agent) | YES | CONFIRMED |
| `completeOrder` | `archive_17012026_1259/taxi/models/api.php:6949` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7055` | 1 (Client) | YES | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7055` | 2 (Driver) | YES | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7055` | 4 (Administrator) | YES | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7055` | 5 (Agent) | YES | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7082` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7082` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7082` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `cancelOrder` | `archive_17012026_1259/taxi/models/api.php:7106` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `rateOrder` | `archive_17012026_1259/taxi/models/api.php:7401` | 1 (Client) | YES | CONFIRMED |
| `rateOrder` | `archive_17012026_1259/taxi/models/api.php:7401` | 2 (Driver) | YES | CONFIRMED |
| `rateOrder` | `archive_17012026_1259/taxi/models/api.php:7401` | 5 (Agent) | YES | CONFIRMED |
| `rateOrder` | `archive_17012026_1259/taxi/models/api.php:7406` | 1 (Client) | YES | CONFIRMED |
| `rateOrder` | `archive_17012026_1259/taxi/models/api.php:7406` | 5 (Agent) | YES | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7531` | 1 (Client) | YES | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7531` | 2 (Driver) | YES | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7531` | 4 (Administrator) | YES | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7531` | 5 (Agent) | YES | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7594` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7594` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7594` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7618` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7639` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7639` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7649` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7678` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7678` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7875` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7875` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7928` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7974` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7978` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:7978` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8020` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8020` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8020` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8055` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8069` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8069` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectOrder` | `archive_17012026_1259/taxi/models/api.php:8069` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `confirmOrder` | `archive_17012026_1259/taxi/models/api.php:8186` | 1 (Client) | YES | CONFIRMED |
| `confirmOrder` | `archive_17012026_1259/taxi/models/api.php:8186` | 5 (Agent) | YES | CONFIRMED |
| `startOrder` | `archive_17012026_1259/taxi/models/api.php:8253` | 2 (Driver) | YES | CONFIRMED |
| `editWaitingTime` | `archive_17012026_1259/taxi/models/api.php:8744` | 1 (Client) | YES | CONFIRMED |
| `editWaitingTime` | `archive_17012026_1259/taxi/models/api.php:8744` | 5 (Agent) | YES | CONFIRMED |
| `setCarUsed` | `archive_17012026_1259/taxi/models/api.php:8972` | 2 (Driver) | YES | CONFIRMED |
| `editOrder` | `archive_17012026_1259/taxi/models/api.php:9256` | 1 (Client) | YES | CONFIRMED |
| `editOrder` | `archive_17012026_1259/taxi/models/api.php:9256` | 2 (Driver) | YES | CONFIRMED |
| `editOrder` | `archive_17012026_1259/taxi/models/api.php:9256` | 5 (Agent) | YES | CONFIRMED |
| `editOrder` | `archive_17012026_1259/taxi/models/api.php:9312` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `editOrder` | `archive_17012026_1259/taxi/models/api.php:9312` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectFavoriteUser` | `archive_17012026_1259/taxi/models/api.php:10662` | 4 (Administrator) | YES | CONFIRMED |
| `selectFavoriteUser` | `archive_17012026_1259/taxi/models/api.php:10666` | 4 (Administrator) | YES | CONFIRMED |
| `addFavoriteUser` | `archive_17012026_1259/taxi/models/api.php:10804` | 4 (Administrator) | YES | CONFIRMED |
| `removeFavoriteUser` | `archive_17012026_1259/taxi/models/api.php:10891` | 4 (Administrator) | YES | CONFIRMED |
| `selectReferralUrl` | `archive_17012026_1259/taxi/models/api.php:10935` | 4 (Administrator) | YES | CONFIRMED |
| `selectReferralUser` | `archive_17012026_1259/taxi/models/api.php:11086` | 4 (Administrator) | YES | CONFIRMED |
| `selectReferralUser` | `archive_17012026_1259/taxi/models/api.php:11090` | 4 (Administrator) | YES | CONFIRMED |
| `setOrderTips` | `archive_17012026_1259/taxi/models/api.php:11206` | 1 (Client) | YES | CONFIRMED |
| `setOrderTips` | `archive_17012026_1259/taxi/models/api.php:11206` | 2 (Driver) | YES | CONFIRMED |
| `setOrderTips` | `archive_17012026_1259/taxi/models/api.php:11206` | 5 (Agent) | YES | CONFIRMED |
| `setOrderTips` | `archive_17012026_1259/taxi/models/api.php:11211` | 1 (Client) | YES | CONFIRMED |
| `setOrderTips` | `archive_17012026_1259/taxi/models/api.php:11211` | 5 (Agent) | YES | CONFIRMED |
| `createTrip` | `archive_17012026_1259/taxi/models/api.php:11373` | 2 (Driver) | YES | CONFIRMED |
| `createTrip` | `archive_17012026_1259/taxi/models/api.php:11375` | 4 (Administrator) | YES | CONFIRMED |
| `createTrip` | `archive_17012026_1259/taxi/models/api.php:11392` | 4 (Administrator) | YES | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12137` | 1 (Client) | YES | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12137` | 2 (Driver) | YES | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12137` | 4 (Administrator) | YES | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12137` | 5 (Agent) | YES | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12229` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12229` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12242` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12244` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12244` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12250` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12250` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12281` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12281` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12302` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12302` | 5 (Agent) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12592` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectTrip` | `archive_17012026_1259/taxi/models/api.php:12592` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `editTrip` | `archive_17012026_1259/taxi/models/api.php:12886` | 2 (Driver) | YES | CONFIRMED |
| `editTrip` | `archive_17012026_1259/taxi/models/api.php:12886` | 4 (Administrator) | YES | CONFIRMED |
| `editTrip` | `archive_17012026_1259/taxi/models/api.php:12982` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `offerOrder` | `archive_17012026_1259/taxi/models/api.php:13572` | 1 (Client) | YES | CONFIRMED |
| `offerOrder` | `archive_17012026_1259/taxi/models/api.php:13572` | 5 (Agent) | YES | CONFIRMED |
| `createDropboxFile` | `archive_17012026_1259/taxi/models/api.php:13730` | 4 (Administrator) | YES | CONFIRMED |
| `createDropboxFile` | `archive_17012026_1259/taxi/models/api.php:13751` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectDropboxFile` | `archive_17012026_1259/taxi/models/api.php:14010` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `editData` | `archive_17012026_1259/taxi/models/api.php:14346` | 2 (Driver) | YES | CONFIRMED |
| `editData` | `archive_17012026_1259/taxi/models/api.php:16850` | 4 (Administrator) | YES | CONFIRMED |
| `selectDataPrivate` | `archive_17012026_1259/taxi/models/api.php:17403` | 4 (Administrator) | YES | CONFIRMED |
| `selectCart` | `archive_17012026_1259/taxi/models/api.php:17449` | 4 (Administrator) | YES | CONFIRMED |
| `selectCart` | `archive_17012026_1259/taxi/models/api.php:17454` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectCart` | `archive_17012026_1259/taxi/models/api.php:17454` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `queryString` | `archive_17012026_1259/taxi/models/api.php:17892` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `queryString` | `archive_17012026_1259/taxi/models/api.php:17911` | 4 (Administrator) | YES | CONFIRMED |
| `writeTicket` | `archive_17012026_1259/taxi/models/api.php:18136` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `readTicket` | `archive_17012026_1259/taxi/models/api.php:18260` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `readTicket` | `archive_17012026_1259/taxi/models/api.php:18368` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `readTicket` | `archive_17012026_1259/taxi/models/api.php:18377` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18624` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18624` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18627` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18641` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18641` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectCartBlock` | `archive_17012026_1259/taxi/models/api.php:18644` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectTicket` | `archive_17012026_1259/taxi/models/api.php:18958` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `sendToTicketBuyerEmail` | `archive_17012026_1259/taxi/models/api.php:19126` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `sendToTicketBuyerEmail` | `archive_17012026_1259/taxi/models/api.php:19138` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `editTicket` | `archive_17012026_1259/taxi/models/api.php:19814` | 6 (Usher) | NOT ESTABLISHED | CONFIRMED |
| `editTicket` | `archive_17012026_1259/taxi/models/api.php:19816` | 6 (Usher) | NOT ESTABLISHED | CONFIRMED |
| `editTicket` | `archive_17012026_1259/taxi/models/api.php:19862` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `queryTemplate` | `archive_17012026_1259/taxi/models/api.php:20153` | 4 (Administrator) | YES | CONFIRMED |
| `includeTemplate` | `archive_17012026_1259/taxi/models/api.php:20284` | 4 (Administrator) | YES | CONFIRMED |
| `translate` | `archive_17012026_1259/taxi/models/api.php:20306` | 4 (Administrator) | YES | CONFIRMED |
| `statusCartBlock` | `archive_17012026_1259/taxi/models/api.php:20347` | 4 (Administrator) | YES | CONFIRMED |
| `statusCartBlock` | `archive_17012026_1259/taxi/models/api.php:20349` | 2 (Driver) | YES | CONFIRMED |
| `statusCartBlock` | `archive_17012026_1259/taxi/models/api.php:20413` | 4 (Administrator) | YES | CONFIRMED |
| `statusCartBlock` | `archive_17012026_1259/taxi/models/api.php:20463` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectInnerUser` | `archive_17012026_1259/taxi/models/api.php:20523` | 4 (Administrator) | YES | CONFIRMED |
| `selectInnerUser` | `archive_17012026_1259/taxi/models/api.php:20527` | 4 (Administrator) | YES | CONFIRMED |
| `requestTemplate` | `archive_17012026_1259/taxi/models/api.php:20674` | 4 (Administrator) | YES | CONFIRMED |
| `selectContact` | `archive_17012026_1259/taxi/models/api.php:20866` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `selectContact` | `archive_17012026_1259/taxi/models/api.php:20880` | 4 (Administrator) | YES | CONFIRMED |
| `selectContact` | `archive_17012026_1259/taxi/models/api.php:20884` | 4 (Administrator) | YES | CONFIRMED |
| `createContact` | `archive_17012026_1259/taxi/models/api.php:21264` | 4 (Administrator) | YES | CONFIRMED |
| `createContact` | `archive_17012026_1259/taxi/models/api.php:21268` | 4 (Administrator) | YES | CONFIRMED |
| `editContact` | `archive_17012026_1259/taxi/models/api.php:21446` | 4 (Administrator) | YES | CONFIRMED |
| `editContact` | `archive_17012026_1259/taxi/models/api.php:21448` | 4 (Administrator) | YES | CONFIRMED |
| `sendMessage` | `archive_17012026_1259/taxi/models/api.php:21838` | 4 (Administrator) | YES | CONFIRMED |
| `createTask` | `archive_17012026_1259/taxi/models/api.php:22616` | 4 (Administrator) | YES | CONFIRMED |
| `controlTask` | `archive_17012026_1259/taxi/models/api.php:22901` | 4 (Administrator) | YES | CONFIRMED |
| `selectTask` | `archive_17012026_1259/taxi/models/api.php:22932` | 4 (Administrator) | YES | CONFIRMED |
| `addTaskLog` | `archive_17012026_1259/taxi/models/api.php:23051` | 4 (Administrator) | YES | CONFIRMED |
| `getTaskLog` | `archive_17012026_1259/taxi/models/api.php:23184` | 4 (Administrator) | YES | CONFIRMED |
| `setTaskStatus` | `archive_17012026_1259/taxi/models/api.php:23239` | 4 (Administrator) | YES | CONFIRMED |
| `deleteDropboxFile` | `archive_17012026_1259/taxi/models/api.php:23390` | 4 (Administrator) | YES | CONFIRMED |
| `getDropboxFileData` | `archive_17012026_1259/taxi/models/api.php:23473` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23617` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23617` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23617` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23619` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23629` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23639` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23650` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23660` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:23670` | 2 (Driver) | NOT ESTABLISHED | CONFIRMED |
| `getTicketData` | `archive_17012026_1259/taxi/models/api.php:24140` | 1 (Client) | NOT ESTABLISHED | CONFIRMED |
| `checkTicket` | `archive_17012026_1259/taxi/models/api.php:24234` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `checkTicket` | `archive_17012026_1259/taxi/models/api.php:24234` | 6 (Usher) | NOT ESTABLISHED | CONFIRMED |
| `checkTicket` | `archive_17012026_1259/taxi/models/api.php:24245` | 6 (Usher) | NOT ESTABLISHED | CONFIRMED |
| `selectCheckTicketLog` | `archive_17012026_1259/taxi/models/api.php:24456` | 4 (Administrator) | YES | CONFIRMED |
| `createPayment` | `archive_17012026_1259/taxi/models/api.php:24772` | 4 (Administrator) | YES | CONFIRMED |
| `selectPayment` | `archive_17012026_1259/taxi/models/api.php:24910` | 4 (Administrator) | YES | CONFIRMED |
| `createSubscription` | `archive_17012026_1259/taxi/models/api.php:25064` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `depositCurrencyAccount` | `archive_17012026_1259/taxi/models/api.php:25381` | 4 (Administrator) | YES | CONFIRMED |
| `withdrawCurrencyAccount` | `archive_17012026_1259/taxi/models/api.php:25625` | 4 (Administrator) | YES | CONFIRMED |
| `transferCurrencyAccount` | `archive_17012026_1259/taxi/models/api.php:25845` | 4 (Administrator) | YES | CONFIRMED |
| `transferCurrencyAccount` | `archive_17012026_1259/taxi/models/api.php:25888` | 4 (Administrator) | YES | CONFIRMED |
| `completeDeal` | `archive_17012026_1259/taxi/models/api.php:26437` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `cancelDeal` | `archive_17012026_1259/taxi/models/api.php:26613` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `confirmDeal` | `archive_17012026_1259/taxi/models/api.php:26784` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |
| `confirmDeal` | `archive_17012026_1259/taxi/models/api.php:26788` | 4 (Administrator) | NOT ESTABLISHED | CONFIRMED |

## 4. Ограничение интерпретации
Один `id_role` comparison ещё не является полной permission relation. Для сложных условий решение восстанавливается по всей control-flow ветви.

Отсутствие role check = `UNKNOWN`, а не `ALLOW`.

## 5. Полностью подтверждённый `/query`
Для текущего snapshot уже доказана цепочка:

```text
Role ID 4
   ↓
Administrator
   ↓
query_roles = 4
   ↓
role membership check
   ↓
ALLOW
```

`Administrator → CAN_EXECUTE → /query = CONFIRMED`.

Текущая конфигурация `query_roles='4'` также даёт version-scoped rejection для остальных role IDs, если они проходят тот же membership check.

## 6. Semantic matrix — текущий подтверждённый слой
| Business Role | Operation | Decision | Scope |
|---|---|---|---|
| Administrator | `/query` | ALLOW | current backend snapshot |
| Client | `/query` | REJECT | current `query_roles=4` |
| Driver | `/query` | REJECT | current `query_roles=4` |
| Agent | `/query` | REJECT | current `query_roles=4` |
| Usher | `/query` | REJECT | current `query_roles=4` |
| Usher with extended powers | `/query` | REJECT | current `query_roles=4` |

## 7. Что ещё не утверждаем
Не утверждаем глобальную permission matrix всех backend methods. Не переносим права `/query` на другие operations. Не смешиваем authorization с query-scope.

## 8. Gap Report
```text
G-17-01  full control-flow normalization of all role checks   OPEN
G-17-02  complete Role × Operation matrix                    OPEN
G-17-03  frontend-specific role capability consumption        OPEN
G-17-04  authorization vs query-scope separation for all ops  OPEN
```

## 9. Следующий шаг
Выбрать наиболее насыщенные protected operations и нормализовать каждую как:

```text
Business Role × Operation × Decision × Preconditions × Evidence
```

После этого полученную backend authorization model можно сопоставлять с конкретной версией конкретного Frontend snapshot.