# Семантический граф Backend — Research Pass 02
## Полный разбор Taxi Web и сопоставление с единым Core Backend

**Pass:** RP-02  
**Клиент:** `taxi-web` / Taxi Web  
**Версия клиента:** `0.1.20`  
**Build timestamp:** `Sun Mar 28 2025 21:37:36 GMT+0300 (GMT+03:00)`  
**Статус:** `IN PROGRESS`  
**Core Backend:** `BACKEND-S0`  
**Database:** `DB-S0`

---

## 1. Назначение прохода

После RP-01 был сформирован верхнеуровневый Semantic Backend Graph по общему Core Backend.

Authentication был первым подробно исследованным вертикальным срезом, но это не является границей текущего исследования.

Задача RP-02 — заново разобрать **весь конкретный Taxi Web frontend**, восстановить его семантические области и связать их с уже существующими областями Core Backend.

Важно:

```text
Taxi Web v0.1.20
        ↓
это один конкретный Client Application
        ↓
он использует
        ↓
Core Backend S0
```

Полученные из frontend утверждения относятся только к этой версии этого клиента.

Они не распространяются автоматически на другие frontend-приложения, даже если те используют тот же Core Backend.

---

# 2. SourceSnapshot

## 2.1. Frontend

```text
source_snapshot_id: FRONTEND-TAXI-WEB-S0

artifact:
taxi-master.zip

sha256:
960959041a5777ac6bf6fe072b6c68825dfa79f54e3db1509365804bae97b680

package:
taxi-app

version:
0.1.20

buildTimestamp:
Sun Mar 28 2025 21:37:36 GMT+0300 (GMT+03:00)
```

Источник содержит около 400 TypeScript/TSX и связанных исходных файлов в `src`.

Основные архитектурные зоны:

```text
src/API
src/state
src/pages
src/components
src/types
src/tools
src/siteConstants.ts
src/config.ts
src/Routes.tsx
src/App.tsx
```

---

## 2.2. Core Backend

```text
source_snapshot_id: BACKEND-S0

artifact:
archive_17012026_1259_clear (1).zip

sha256:
e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
```

Это общий Core Backend, к которому обращается исследуемый frontend.

---

## 2.3. Database

```text
source_snapshot_id: DB-S0

artifact:
aristotel_taxi.sql (1).zip

sha256:
501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
```

---

# 3. Архитектурная позиция клиента

Frontend построен вокруг:

```text
React
Redux
Redux-Saga
Axios
React Router
```

Корневая композиция состояния содержит:

```text
geolocation
user
cars
orders
ordersDetails
order
clientOrder
config
modals
areas
```

Root Saga объединяет соответствующие saga.

### Claim

```text
C-FE-ARCH-001

subject: Taxi Web v0.1.20
predicate: USES
object: Core Backend API
confidence: CONFIRMED
```

Evidence:

```text
CODE:
src/API/*
src/state/*/sagas.ts
```

---

# 4. Главный результат полного разбора

Frontend не является одной capability.

В нём обнаружены как минимум следующие семантические области:

```text
1. Authentication / Registration
2. User Profile / Account
3. Configuration / Site Configuration
4. Passenger Order Construction
5. Order Lifecycle / Tracking
6. Driver Order Execution
7. Candidate / Driver Selection
8. Vehicle Management
9. Geolocation
10. Address / Geocoding
11. Service Area / Polygon Resolution
12. Routing
13. Trip Management
14. Payment Representation
15. File / Media Management
16. Communication / Chat
17. Referral Code
18. Rating / Waiting / Tips
19. Delivery / Move / Big Truck options
20. Localization / Language / Currency
21. Client-side persistence
22. UI / Modal interaction layer
```

Не все эти области являются самостоятельными backend capabilities.

Некоторые являются:

```text
CLIENT INTERACTION
CLIENT STATE
CLIENT ADAPTER
SHARED DOMAIN MODEL
BACKEND CAPABILITY CONSUMPTION
EXTERNAL INTEGRATION
```

Именно это различие является целью semantic extraction.

---

# 5. Frontend architecture frame

## 5.1. API layer

Основной API façade:

```text
src/API/index.ts
```

Он экспортирует:

```text
Authentication
User
Car
Order
Way / Area
Polygon
File
Referral
Configuration
Trip
Location
```

и содержит дополнительные внешние API:

```text
Nominatim
OpenRouteService
HERE
chat.itest24.com
jecat.ru
```

### Semantic Claim

```text
C-FE-ARCH-002

subject: src/API
predicate: IMPLEMENTS
object: Client API Adapter Layer
confidence: CONFIRMED
```

---

# 6. Общий механизм авторизации запросов

`src/tools/api.ts` содержит `apiMethod`.

Для `authRequired = true`:

```text
Redux user.tokens
       ↓
token
u_hash
       ↓
FormData
       ↓
API method
```

### Claim

```text
C-FE-AUTH-CLIENT-001

subject: Taxi Web API Adapter
predicate: AUTHENTICATES_REQUESTS_WITH
value: token + u_hash
confidence: CONFIRMED
```

Это связывается с Core Backend authentication/token subgraph.

---

# 7. Authentication

Подробный материал первоначального Auth Pass сохраняется, но теперь рассматривается как **один subgraph полного RP-02**, а не как весь frontend.

Основные frontend SourceFacts:

```text
src/components/modals/LoginModal/Login.tsx
src/components/modals/LoginModal/Register.tsx
src/components/modals/LoginModal/WACodeModal.tsx
src/components/modals/LoginModal/RefCodeModal.tsx

src/API/auth.ts

src/state/user/*
```

Frontend использует:

```text
/register
/auth
/token
/token/authorized
/logout
```

и:

```text
/user/authorized
```

для восстановления пользователя.

### Cross-system relation

```text
Taxi Web Authentication
        CALLS
Core Backend /auth
        ↓
API::authUser
```

### Status

```text
CONFIRMED
```

для наблюдаемого Taxi Web v0.1.20.

---

# 8. User / Account

Frontend имеет отдельный User model:

```text
IUser
```

с полями:

```text
u_id
u_name
u_family
u_email
u_phone
u_role
u_check_state
u_ban
u_active
u_photo
u_birthday
u_phone_checked
u_lang
u_currency
u_city
u_tips
u_lang_skills
u_description
u_gps_software
out_drive
u_details
ref_code
token
u_hash
```

API:

```text
/user/{id}
/user/{ids}
/user/authorized
/user
```

### Claim

```text
C-FE-USER-001

subject: Taxi Web
predicate: USES
object: Core Backend User Management
confidence: CONFIRMED
```

### Important semantic distinction

Frontend одновременно использует User для:

```text
Authentication
Authorization
Profile
Driver data
Out-of-service drive
Referral
Localization
```

Следовательно:

```text
User
```

не следует автоматически относить к Authentication.

---

# 9. Configuration

Это одна из наиболее важных областей frontend.

`src/config.ts` выбирает конфигурационный профиль:

```text
config query parameter
        ↓
localStorage config
        ↓
Core Backend URL
        ↓
data.js / data_{name}.js
        ↓
window.data
```

Конфигурация загружается с:

```text
https://ibronevik.ru/taxi/cache/data*.js
```

через механизм cache version.

`src/siteConstants.ts` преобразует полученные данные в runtime configuration.

### Основные configuration groups

```text
map_mode
countries
Country_service
cities
palette
geo_default
radius_geo_name
passenger_order_config
mode_money
type_of_transport
type_of_cargo
c_options_valid_keys
car_classes
mode_response
booking_comments
booking_location_classes
calculation_benefits
langs
currency_of_the_service
```

### Claim

```text
C-FE-CONFIG-001

subject: Taxi Web
predicate: CONFIGURED_BY
object: Core Backend / runtime configuration
confidence: CONFIRMED
```

Это не просто UI configuration.

Часть параметров напрямую меняет domain behavior.

---

# 10. Passenger Order Construction

Главный клиентский state:

```text
state/clientOrder
```

содержит:

```text
from
to
fromPolygons
toPolygons
carClass
seats
comments
time
locationClass
phone
customerPrice
selectedOrder
status
message
```

Это полноценная client-side модель подготовки заказа.

### Claim

```text
C-FE-ORDER-001

subject: Passenger Order Construction
predicate: REPRESENTS
object: Order creation intent
confidence: CONFIRMED
```

---

# 11. Order creation

`VotingForm.tsx` собирает данные:

```text
from
to
carClass
seats
customerPrice
comments
phone
time
payment_way
services
```

и создаёт:

```text
ordersActionCreators.create
        ↓
API.postDrive
        ↓
POST /drive
```

### Cross-system relation

```text
Taxi Web Order Form
        ↓
CREATES
        ↓
Core Backend Booking
        ↓
API::createOrder
```

Это прямое подтверждение связи frontend order capability с backend Order/Booking.

---

# 12. Order domain model

Frontend `IOrder` является одной из наиболее семантически насыщенных структур системы.

Он содержит:

```text
booking identity
user
time
locations
passengers
luggage
car class
booking state
confirmation
waiting
drivers
comments
services
payment
rating
cancellation
voting
offer
selection
pricing
```

Важный вывод:

```text
IOrder
```

является не просто DTO одного экрана.

Он является клиентским представлением существенной части backend Booking/Order domain.

---

# 13. Order lifecycle

Frontend использует:

```text
EBookingStates
```

```text
Processing
Approved
Canceled
Completed
PendingActivation
OfferedToDrivers
```

и отдельно:

```text
EBookingDriverState
```

```text
Considering
Canceled
Performer
Arrived
Started
Finished
```

Это принципиально важное различие.

```text
Booking lifecycle
        ≠
Driver execution lifecycle
```

### Backend mapping

Frontend отправляет:

```text
/drive/get/{id}
```

с actions:

```text
set driver
cancel
set state
set rating
set waiting time
```

что соответствует backend order state manipulation.

---

# 14. Order polling

Frontend не использует WebSocket для основного order state.

`state/orders/sagas.ts` содержит polling:

```text
active orders:
5000 ms
driver active:
2000 ms

ready orders:
3000 ms

history:
10000 ms

single order:
2000 ms
```

### Claim

```text
C-FE-ORDER-RT-001

subject: Taxi Web Order State
predicate: SYNCHRONIZES_BY
value: polling
confidence: CONFIRMED
```

Это важная часть фактического runtime-контракта между клиентом и Core Backend.

---

# 15. Passenger execution flow

Frontend Passenger:

```text
Passenger page
    ↓
Map
    ↓
VotingForm
    ↓
create order
    ↓
active orders polling
    ↓
selected order
    ↓
driver state
    ↓
Candidate / Vote / Driver / OnTheWay UI
    ↓
Rating
```

Используются:

```text
candidateMode(order)
b_voting
drivers[].c_state
```

### Semantic Claim

```text
C-FE-PASSENGER-001

subject: Taxi Web Passenger UI
predicate: DERIVES_UI_STATE_FROM
object: Order + Driver state
confidence: CONFIRMED
```

Это означает, что часть frontend FSM является **derived UI state**, а не независимым backend FSM.

---

# 16. Candidate Selection

Frontend имеет отдельный flow:

```text
candidateMode(order)
        ↓
CandidatesModal
        ↓
getOrder
getUsers
getCars
        ↓
chooseCandidate
```

Backend endpoint:

```text
POST /drive/get/{id}
```

### Claim

```text
C-FE-CANDIDATE-001

subject: Candidate Selection UI
predicate: USES
object: Core Backend Driver Candidate capability
confidence: CONFIRMED
```

---

# 17. Driver execution

Driver frontend имеет:

```text
Driver Map
Driver Orders
Order page
CardModal
```

и использует:

```text
takeOrder
setOrderState
cancelDrive
```

Состояния:

```text
Considering
Performer
Arrived
Started
Finished
```

### Cross-system chain

```text
Driver UI
   ↓
takeOrder
   ↓
/drive/get/{id}
   ↓
Core Backend setDriver
```

и:

```text
Driver UI
   ↓
setOrderState
   ↓
/drive/get/{id}
   ↓
Core Backend order execution state
```

---

# 18. Driver location

Driver map использует:

```text
navigator geolocation
```

и:

```text
API.notifyPosition
```

которая обращается не к Core Backend:

```text
http://jecat.ru/car_api/api/notifypos.php
```

### Important finding

Это отдельная внешняя integration:

```text
Taxi Web Driver
       ↓
Position notification
       ↓
jecat.ru
```

Она не должна автоматически считаться частью Core Backend Location capability.

---

# 19. Geolocation

Frontend имеет отдельный state:

```text
state/geolocation
```

Он отвечает за:

```text
WATCH
UNWATCH
GET_SUCCESS
GET_FAIL
ACTIVATE_SENDING
DEACTIVATE_SENDING
```

Используется для:

```text
current location
passenger order origin
driver location
```

### Claim

```text
C-FE-GEO-001

subject: Taxi Web Geolocation
predicate: USES
object: Browser Geolocation API
confidence: CONFIRMED
```

---

# 20. Address / Geocoding

Frontend использует:

```text
Nominatim
```

для:

```text
reverseGeocode
geocode
```

и:

```text
HERE autosuggest
```

для suggestions.

Также локальные hints используются до обращения к внешнему geocoder.

### Claim

```text
C-FE-MAP-001

subject: Taxi Web Address Resolution
predicate: USES
object: External Geocoding Services
confidence: CONFIRMED
```

Это client-side integration, а не Core Backend capability.

---

# 21. Routing

Frontend напрямую вызывает:

```text
OpenRouteService
```

через:

```text
makeRoutePoints
```

### Claim

```text
C-FE-MAP-002

subject: Taxi Web Routing
predicate: USES
object: OpenRouteService
confidence: CONFIRMED
```

Это особенно важно для будущего Graph:

```text
Order
  USES
Routing

Routing
  IMPLEMENTED_BY
External Service
```

не следует автоматически связывать Routing с Core Backend только потому, что заказ существует в backend.

---

# 22. Service Areas / Polygons

Frontend использует:

```text
getPolygonsIdsOnPoint
```

для:

```text
from
to
```

и:

```text
getAreasIdsBetweenPoints
```

для driver/ready orders.

State:

```text
areas
clientOrder.fromPolygons
clientOrder.toPolygons
```

### Claim

```text
C-FE-GEO-002

subject: Order Location
predicate: RESOLVES_TO
object: Service Areas / Polygons
confidence: CONFIRMED
```

---

# 23. Vehicle Management

Frontend содержит отдельный state:

```text
cars
```

и API:

```text
createCar
getCar
getCars
createUserCar
getUserCars
getUserCar
editCar
driveCar
getUserDrivenCar
```

Backend endpoints:

```text
/user/{id}/car
/car
/car/{id}
/user/authorized/car
/user/authorized/car/driven
/car/{id}/drive
```

### Semantic model

```text
User
   ↓
owns / uses
   ↓
Car
   ↓
Car Class
```

Driver additionally selects/drives a car.

### Claim

```text
C-FE-CAR-001

subject: Taxi Web Vehicle Management
predicate: USES
object: Core Backend Vehicle Management
confidence: CONFIRMED
```

---

# 24. Car Class

Frontend receives/configures:

```text
ICarClass
```

with:

```text
id
seats
courier_call_rate
courier_fare_per_1_km
booking_location_classes
```

`siteConstants.CAR_CLASSES` is populated from runtime data.

Car class participates in:

```text
order creation
vehicle selection
pricing
delivery/courier modes
```

Therefore:

```text
Car Class
```

is a shared semantic dependency, not merely UI configuration.

---

# 25. Trip Management

Frontend exposes:

```text
postTrip
getTrips
getWashTrips
```

against:

```text
/trip
/trip/get
```

Model:

```text
ITrip
```

contains:

```text
driver
start
destination
schedule
real start
real completion
editor
creator
looking for clients
cancelled
options
orders
```

### Claim

```text
C-FE-TRIP-001

subject: Taxi Web Trip Management
predicate: USES
object: Core Backend Trip capability
confidence: CONFIRMED
```

Important distinction:

```text
Trip
  contains / aggregates
Order
```

is strongly represented by `ITrip.orders`.

Whether this is exactly the same persistence relation in DB requires backend/DB confirmation.

---

# 26. Payment representation

Frontend contains:

```text
EPaymentWays
    Cash
    Credit
    Paypal
```

`IOrder` contains:

```text
b_payment_way
b_payment_card
b_payment_sum
b_payment_datetime
b_tips
```

`IDriver` contains:

```text
c_payment_way
c_payment_card
c_payment_sum
c_payment_datetime
c_tips
```

The order creation form currently sends:

```text
b_payment_way: EPaymentWays.Cash
```

### Important finding

Frontend has a **Payment semantic model**, but no corresponding general Payment API adapter was found in `src/API`.

There is UI for attaching card details:

```text
CardDetailsModal
```

but its submit handler only validates/logs the form and closes the modal.

Therefore:

```text
Payment UI/model exists
        ≠
Payment capability is implemented by this frontend
```

### Cross-reference to Core Backend

The Core Backend exposes payment-related operations such as:

```text
createPayment
createSubscription
depositCurrencyAccount
withdrawCurrencyAccount
transferCurrencyAccount
createDeal
```

but these are not currently demonstrated as consumed by Taxi Web v0.1.20.

Status:

```text
Frontend → Payment Backend
UNKNOWN / NOT CONFIRMED
```

This is a significant finding for future Platform Payment research.

---

# 27. Files / Media

Frontend contains:

```text
uploadFile
getImageBlob
getImageFile
```

Backend endpoints:

```text
/dropbox/file
/dropbox/file/{id}
```

Used by:

```text
profile
user documents
order-related UI
```

The User model contains:

```text
u_photo
uploads
passport_photo
driver_license_photo
license_photo
```

### Claim

```text
C-FE-FILE-001

subject: Taxi Web File Management
predicate: USES
object: Core Backend File Storage capability
confidence: CONFIRMED
```

---

# 28. Communication / Chat

Frontend Chat component connects to:

```text
chat.itest24.com
```

and initializes the chat server periodically:

```text
API.activateChatServer()
```

every 30 seconds.

Chat identity is built from:

```text
user_id
order_id
```

Example:

```text
from = user_id + order_id
to   = other_user_id + order_id
```

For externally created orders, `b_options.createdBy` may select:

```text
sms
whatsapp
```

and frontend may open:

```text
tel:
https://wa.me/
```

### Claim

```text
C-FE-COMM-001

subject: Taxi Web Order Communication
predicate: USES
object: External Chat Service
confidence: CONFIRMED
```

### Important distinction

Communication is a capability consumed by the order interaction, but the current evidence does not establish that it belongs to Core Backend.

---

# 29. Referral

Frontend uses:

```text
checkRefCode
```

against:

```text
/referral/code/{ref_code}/check
```

Referral code participates in:

```text
registration
profile editing
```

### Claim

```text
C-FE-REF-001

subject: Taxi Web Referral
predicate: USES
object: Core Backend Referral capability
confidence: CONFIRMED
```

---

# 30. Rating / Waiting / Tips

Frontend exposes:

```text
setOrderRating
setWaitingTime
```

and UI for:

```text
rating
tips
comments
```

Backend request:

```text
/drive/get/{id}
```

uses actions:

```text
set_rate
set_waiting_time
set_tips
```

### Claim

```text
C-FE-ORDER-ACTION-001

subject: Taxi Web Order Feedback
predicate: USES
object: Core Backend Order Feedback capability
confidence: CONFIRMED
```

---

# 31. Delivery / Move / Big Truck

The frontend contains substantial domain structures for:

```text
courier transport
move types
cargo
cargo weight
size
cars count
truck types
truck services
rooms
furniture
elevator
stairs
loading
```

These are represented primarily inside:

```text
IOptions
```

and UI components:

```text
components/passenger-order/delivery/*
components/passenger-order/move/*
components/rooms
components/furniture
components/BigTruckServices
```

### Current semantic conclusion

This is a substantial **Order Option / Service Configuration subgraph**.

It is not yet justified to split it into independent backend domains.

The backend receives these values as order data through:

```text
/drive
```

Therefore currently:

```text
Delivery / Move / Big Truck
        USES
Order Creation
```

with potential future subdomains.

Status:

```text
CONFIRMED as client behavior
INFERRED as independent backend capability
```

---

# 32. Localization

Frontend has:

```text
localization/*
```

and runtime language configuration:

```text
LANGUAGES
THE_LANGUAGE_OF_THE_SERVICE
u_lang
```

It also sends language-sensitive requests to external geocoding.

Localization is therefore partly:

```text
Client infrastructure
```

and partly:

```text
User preference
```

It should not be treated as a business platform candidate at this stage.

---

# 33. Client-side persistence

The frontend persists:

```text
authentication tokens
order form selections
comments
time
phone
car class
seats
location class
```

in localStorage.

Examples:

```text
state.user.tokens

state.clientOrder.carClass
state.clientOrder.seats
state.clientOrder.locationClass
state.clientOrder.comments
state.clientOrder.time
state.clientOrder.phone
```

This is important because:

```text
Backend persistence
        ≠
Client persistence
```

Both may describe the same business concept at different layers.

---

# 34. Runtime UI state

`state/modals` contains a large collection of UI interaction states:

```text
cancel
pick time
comments
driver
rating
tie card
card details
vote
seats
login
alarm
WA code
ref code
take passenger
driver cancel
on the way
map
message
profile
candidates
delete files
order card
chat
```

These are not automatically Semantic Domain Frames.

They are primarily:

```text
CLIENT_INTERACTION_STATE
```

They become semantic evidence when they expose a real domain behavior or backend contract.

---

# 35. Frontend semantic areas — consolidated map

```text
Taxi Web v0.1.20
│
├── Authentication
│   ├── Login
│   ├── Registration
│   ├── WhatsApp verification
│   ├── Google login
│   ├── Token
│   ├── Session restoration
│   └── Logout
│
├── User / Account
│   ├── Profile
│   ├── Role
│   ├── Verification state
│   ├── Ban state
│   ├── Language
│   ├── Currency
│   └── Referral
│
├── Order / Booking
│   ├── Creation
│   ├── Editing
│   ├── Cancellation
│   ├── Active
│   ├── Ready
│   ├── History
│   ├── Polling
│   ├── Candidate selection
│   ├── Driver execution
│   ├── Rating
│   ├── Waiting
│   └── Tips
│
├── Trip
│
├── Driver
│   ├── Orders
│   ├── Location
│   ├── Vehicle
│   └── Out-of-service drive
│
├── Vehicle
│   ├── Car
│   ├── Car Class
│   └── Driven Car
│
├── Location
│   ├── Geolocation
│   ├── Geocoding
│   ├── Suggestions
│   ├── Routing
│   └── Service Areas
│
├── Order Services / Options
│   ├── Passenger count
│   ├── Luggage
│   ├── Courier
│   ├── Move
│   ├── Big Truck
│   ├── Furniture
│   └── Additional services
│
├── Payment Representation
│   ├── Cash
│   ├── Credit
│   ├── PayPal
│   └── Card UI
│
├── Files / Media
│
├── Communication
│   ├── Chat
│   ├── SMS
│   └── WhatsApp
│
└── Client Infrastructure
    ├── Configuration
    ├── Localization
    ├── localStorage
    └── UI state
```

---

# 36. Frontend → Core Backend mapping

| Frontend area | Core Backend relation | Status |
|---|---|---|
| Authentication | `/auth`, `/token`, `/register`, `/logout` | CONFIRMED |
| User | `/user/*` | CONFIRMED |
| Referral | `/referral/*` | CONFIRMED |
| Order | `/drive/*` | CONFIRMED |
| Trip | `/trip/*` | CONFIRMED |
| Vehicle | `/car/*`, `/user/*/car` | CONFIRMED |
| Files | `/dropbox/file/*` | CONFIRMED |
| Location sending | `/location` | CONFIRMED |
| Runtime config | `/api/v1?config=...` + data.js | CONFIRMED |
| Payment | backend payment endpoints exist, Taxi Web usage not found | UNKNOWN |
| Chat | external service | CONFIRMED external |
| Geocoding | external services | CONFIRMED external |
| Routing | OpenRouteService | CONFIRMED external |
| Driver position | jecat.ru | CONFIRMED external |
| Areas / polygons | backend/data endpoint involvement exists; complete mapping needs further tracing | INFERRED |

---

# 37. Important cross-domain dependencies

The frontend reveals several dependencies that are important for future platform boundaries.

## Authentication

```text
Authentication
 ├── User
 ├── Verification
 ├── Account State
 ├── Communication
 └── Configuration
```

## Order

```text
Order
 ├── User
 ├── Vehicle / Car Class
 ├── Location
 ├── Pricing
 ├── Payment Representation
 ├── Driver
 ├── Candidate Selection
 ├── Communication
 └── Services / Options
```

## Driver

```text
Driver
 ├── User
 ├── Car
 ├── Order
 ├── Geolocation
 └── Communication
```

## Vehicle

```text
Vehicle
 ├── User
 ├── Car Class
 └── Order / Driver
```

This confirms that future platform boundaries cannot be derived from isolated frontend modules.

---

# 38. Important negative findings

Several things exist in the frontend without proving corresponding backend ownership.

### Payment

```text
Payment model exists
Card UI exists
```

but:

```text
Taxi Web → Core Backend Payment API
```

is not confirmed.

### Session Token Duration

Configuration may exist in backend DB, but active client behavior is not enough to establish it.

### Delivery / Move

Large UI/domain model exists, but independent backend capability boundaries are not yet established.

### Routing

The client performs routing directly through OpenRouteService.

Therefore routing should not automatically be classified as Core Backend capability.

---

# 39. Client-specific versus global claims

All of the following are claims about:

```text
taxi-web v0.1.20
```

unless supported independently elsewhere:

```text
Taxi Web calls /auth
Taxi Web polls orders every 2 seconds
Taxi Web uses OpenRouteService
Taxi Web stores clientOrder in localStorage
Taxi Web uses chat.itest24.com
Taxi Web represents payment ways Cash/Credit/PayPal
```

These MUST NOT be rewritten as:

```text
All Taxi clients do this
```

without additional independent Evidence.

---

# 40. First cross-client research hypothesis

The next clients can be represented independently:

```text
Core Backend S0
    │
    ├── Taxi Web v0.1.20
    │
    ├── Taxi Driver App v...
    │
    ├── Taxi Admin v...
    │
    └── other clients
```

After separate passes, comparison may reveal:

```text
shared backend capability
        +
multiple independent clients
        +
stable shared contract
        ↓
strong Platform Candidate evidence
```

This is more reliable than identifying a platform candidate from a backend class name.

---

# 41. Current Platform Candidate signals

The full frontend pass strengthens the following candidates:

## Authentication

```text
Platform Candidate: Auth
status: RESEARCHING
```

Strong evidence:

```text
Core Backend implementation
+
Taxi Web client consumption
```

Still required:

```text
other clients
ownership
dependencies
contract boundary
authorization separation
```

## Cart

No distinct Cart semantic subgraph was established in this frontend.

The current client primarily represents:

```text
Order
```

and order construction.

Therefore this frontend does **not** provide evidence that a standalone Cart capability exists.

This is important because it prevents us from projecting the desired Platform Cart architecture onto the client.

## Payment

Payment semantics are present in the order model, but active Payment service consumption is not established.

Therefore:

```text
Platform Candidate: Payment
status: RESEARCHING
```

but frontend evidence is currently insufficient to define its boundary.

---

# 42. Research quality status

## Confirmed

```text
Taxi Web v0.1.20
    ↓
Core Backend
```

with confirmed usage of:

```text
Authentication
User
Order
Trip
Vehicle
Files
Referral
Location sending
Configuration
```

and confirmed external usage of:

```text
Nominatim
OpenRouteService
HERE
chat.itest24.com
jecat.ru
```

## Inferred

```text
Areas / polygon backend semantic ownership
Delivery / Move independent capability
Payment backend usage
Google authentication complete backend path
```

## UNKNOWN

```text
Whether all other clients share the same frontend/backend contracts
Exact ownership of Payment UI semantics
Exact backend ownership of area/polygon resolution
Complete semantics of token / u_hash across all clients
```

---

# 43. Главный результат RP-02

Первый backend-only граф показал:

```text
Core Backend
    ↓
Capabilities
```

Полный разбор Taxi Web добавил:

```text
Core Backend
    ↑
Client-specific usage
    ↑
Taxi Web v0.1.20
```

Получилась новая семантическая структура:

```text
                    Core Backend
                         │
        ┌────────────────┼────────────────┐
        │                │                │
 Authentication        Order           Vehicle
        ↑                ↑                ↑
        │                │                │
 Taxi Web v0.1.20 ──────┴────────────────┘
        │
        ├── Client State
        ├── UI Interaction
        ├── External Integrations
        └── Client-specific behavior
```

Это не означает, что frontend является частью backend capability.

Это означает, что Semantic Graph теперь способен различать:

```text
что backend предоставляет
```

и:

```text
как конкретный клиент конкретной версии это использует.
```

---

# 44. Следующий шаг

Не следует сразу проектировать Platform Auth.

Следующий рациональный шаг — использовать этот полный Taxi Web pass как основу для **детального cross-source исследования Authentication**, а затем проверить ещё один участок, где frontend существенно влияет на понимание backend.

Одновременно необходимо сохранить этот документ как отдельный client-specific Research Pass.

Следующий frontend другого приложения должен получить:

```text
новый SourceSnapshot
+
новый Research Pass
```

а не изменять RP-02.

И только после нескольких таких passes можно строить:

```text
Client Usage Graph
        ↓
Shared Capability Analysis
        ↓
Platform Candidate
        ↓
Decision Gate
```
