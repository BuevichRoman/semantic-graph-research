# Backend Semantic Graph — Research Pass 01
## Code-first reconstruction of the existing backend

**Version:** RP-01 / v2.2 methodology  
**Status:** `PROVISIONAL — provenance completion required`  
**Research mode:** code-first  
**Scope:** whole existing backend at L0–L1, with limited DB cross-validation  
**Purpose:** reconstruct the existing backend semantic territory before investigating Platform Auth, Platform Cart or Platform Payment.

---

# 1. Research Scope

## 1.1 Research question

What functional areas and application-level capabilities actually exist in the backend, and what dependencies between them can be established from the code?

The pass must answer this without starting from:

```text
Cart
Auth
Payment
```

as predefined architectural boundaries.

The intended result is:

```text
Existing Backend
      ↓
Code Evidence
      ↓
Semantic Model
      ↓
Functional Areas
      ↓
Capabilities
      ↓
Dependencies
      ↓
Platform Candidates
```

No Platform Candidate is approved by this pass.

---

## 1.2 Source boundaries

The intended source inventory is:

```text
CODE
DB_SCHEMA
DB_DATA
API_CONTRACT
TEST
CONFIGURATION
DOCUMENTATION
```

For RP-01, the primary source is:

```text
CODE
```

The following sources are used only for cross-validation where already available:

```text
DB_SCHEMA
DB_DATA / site_constant
```

### Concrete source snapshot now available

The backend PHP source is available as:

```text
archive: archive_17012026_1259_clear (1).zip
sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
```

The database dump is available as:

```text
archive: aristotel_taxi.sql (1).zip
sha256: 501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
```

The PHP archive has no Git commit identifier in the available material. Therefore the archive SHA-256 is used as the immutable source revision.

Concrete provenance can now be attached to SourceFacts using:

```text
archive sha256
→ internal file
→ symbol
→ line range
```

Accordingly, the previous `provenance_status = INCOMPLETE` limitation is removed for findings that have been checked against the extracted source.

---

# 2. Research Pass Record

```text
pass_id: RP-01
scope_id: BACKEND-L0-L1
research_mode: CODE_FIRST
status: PROVISIONAL
```

The pass is not considered finally publishable until every material finding has a concrete SourceFact.

---

# 3. Source Inventory

## 3.1 Code

The source inventory was checked directly against the extracted PHP archive.

Primary files currently used:

```text
taxi/models/api.php
taxi/models/m_functions.php
taxi/controllers/c_api.php
taxi/controllers/c_index.php
taxi/config/system_bot.php
```

The source revision is:

```text
archive sha256:
e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
```

Concrete SourceFacts are recorded in §4.1.

The current pass therefore has reproducible CODE provenance for the findings explicitly listed there.

---

## 3.2 Database schema

Previously investigated database structures provide independent structural validation.

### Provenance status

```text
DB_SCHEMA SourceFacts:
PARTIALLY AVAILABLE
```

The database source should ultimately be recorded as concrete:

```text
database
schema
object
column
constraint
DDL snapshot
```

---

## 3.3 site_constant

`site_constant` was separately investigated as a configuration table.

The important methodological conclusion is:

```text
DB_DATA proves that a configuration value exists.
CODE proves where and how that value is used.
```

Therefore:

```text
site_constant row
```

must not automatically become:

```text
CONFIGURES Capability
```

without code Evidence showing its use.

---

# 4. Extraction Rule

RP-01 uses the following evidence discipline.

For every material finding:

```text
Physical Source
      ↓
SourceFact
      ↓
Evidence
      ↓
SemanticClaim
      ↓
Frame / Relation
```

No direct:

```text
CODE → Frame
```

is permitted in the final graph.

---

# 4.1. Concrete SourceFacts and Evidence

The following are the first machine-oriented provenance records extracted directly from the backend source snapshot.

## SF-CART-001 — Cart read capability

```text
id: SF-CART-001
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbol: selectCart
  lines: 17441-17557
observation:
  selectCart reads the cart table and joins trip, ticket and order.
```

Important code facts:

```text
cart.product → trip.id_trip
cart.product + cart.property → ticket
ticket.id_order → order
```

Evidence:

```text
E-CART-001
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-CART-001
source_fact_ids: [SF-CART-001]
```

Claim:

```text
C-CART-001
subject: Cart
predicate: EXPOSES
object: CartReadCapability
confidence: CONFIRMED
evidence_ids: [E-CART-001]
```

---

## SF-CART-002 — Cart update and persistence behavior

```text
id: SF-CART-002
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbol: updateCart
  lines: 17559-17660+
observation:
  updateCart deletes/inserts/updates rows in cart using id_user, product and property.
  In the stadium profile property is resolved through ticket.id_trip/id_seat.
  booking_limit is calculated using ticket_booking_duration.
```

Evidence:

```text
E-CART-002
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-CART-002
source_fact_ids: [SF-CART-002]
```

Claims:

```text
C-CART-002
subject: Cart
predicate: PERSISTS
object: cart
confidence: CONFIRMED
evidence_ids: [E-CART-002]

C-CART-003
subject: Cart
predicate: HAS_SLOT
value: product
confidence: CONFIRMED
evidence_ids: [E-CART-002]

C-CART-004
subject: Cart
predicate: HAS_SLOT
value: property
confidence: CONFIRMED
evidence_ids: [E-CART-002]
```

Important semantic restriction:

`product` and `property` are confirmed as application fields, but their general business meaning is **not** yet confirmed.

---

## SF-CART-003 — Cart clear capability

```text
id: SF-CART-003
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbol: clearCart
  lines: 19552-19603
observation:
  clearCart deletes cart rows for the authenticated user and can select
  product/property pairs through ticket-derived seat identifiers.
```

Evidence:

```text
E-CART-003
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-CART-003
source_fact_ids: [SF-CART-003]
```

Claim:

```text
C-CART-005
subject: Cart
predicate: EXPOSES
object: CartClearCapability
confidence: CONFIRMED
evidence_ids: [E-CART-003]
```

---

## SF-CART-004 — Cart ownership transfer

```text
id: SF-CART-004
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbol: moveCart
  lines: 23274-23325+
observation:
  moveCart updates cart.id_user from the current authenticated user to another user.
```

Evidence:

```text
E-CART-004
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-CART-004
source_fact_ids: [SF-CART-004]
```

Claim:

```text
C-CART-006
subject: Cart
predicate: SUPPORTS
object: CartOwnershipTransfer
confidence: CONFIRMED
evidence_ids: [E-CART-004]
```

---

## SF-CART-005 — Cart API exposure

```text
id: SF-CART-005
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/controllers/c_api.php
  symbols:
    selectCart: line 431
    updateCart: line 435
    clearCart: line 442
    moveCart: line 446
observation:
  c_api dispatches external API requests to Cart methods.
```

Evidence:

```text
E-CART-005
type: API
strength_category: CONTRACTUAL
independent_group: E-CART-005
source_fact_ids: [SF-CART-005]
```

Claim:

```text
C-CART-007
subject: Cart
predicate: EXPOSES
object: BackendAPI
confidence: CONFIRMED
evidence_ids: [E-CART-005]
```

---

## SF-AUTH-001 — Authentication code lifetime

```text
id: SF-AUTH-001
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbols:
    auth code creation: lines 480-529
    token/auth hash interval use: line 8353
observation:
  auth code expiration is calculated from constant auth_code_interval;
  auth hash/token retrieval is constrained by token_interval_for_auth_hash.
```

Evidence:

```text
E-AUTH-001
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-AUTH-001
source_fact_ids: [SF-AUTH-001]
```

Claim:

```text
C-AUTH-001
subject: Authentication
predicate: CONFIGURED_BY
object: auth_code_interval
confidence: CONFIRMED
evidence_ids: [E-AUTH-001]
```

---

## SF-CONFIG-001 — Driver selection configuration usage

```text
id: SF-CONFIG-001
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/config/system_bot.php
  symbols/lines:
    constant definitions: 30-140
    offered-driver count/duration processing: 264-355+
observation:
  d_s_sorting_city, d_s_sorting_intercity,
  d_s_offered_drivers_count and d_s_offered_drivers_duration
  are loaded from site_constants and used to calculate offered-driver behavior.
```

Evidence:

```text
E-CONFIG-001
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-CONFIG-001
source_fact_ids: [SF-CONFIG-001]
```

Claims:

```text
C-CONFIG-001
subject: Driver Selection
predicate: CONFIGURED_BY
object: d_s_sorting_city
confidence: CONFIRMED
evidence_ids: [E-CONFIG-001]

C-CONFIG-002
subject: Driver Selection
predicate: CONFIGURED_BY
object: d_s_offered_drivers_count
confidence: CONFIRMED
evidence_ids: [E-CONFIG-001]

C-CONFIG-003
subject: Driver Selection
predicate: CONFIGURED_BY
object: d_s_offered_drivers_duration
confidence: CONFIRMED
evidence_ids: [E-CONFIG-001]
```

---

## SF-ORDER-001 — Order lifecycle operations

```text
id: SF-ORDER-001
source_type: CODE
source_location:
  archive_sha256: e3ab7f347e2b4cb3f66caa6de64666e42b1915e15afcde209d1df3574017d9a5
  file: archive_17012026_1259/taxi/models/api.php
  symbols:
    confirmOrder: line 8181
    startOrder: line 8248
    completeOrder: line 6896
    cancelOrder: line 7050
    setCarIsArrived: line 6799
observation:
  backend exposes distinct order lifecycle operations for confirmation,
  arrival, start, completion and cancellation.
```

Evidence:

```text
E-ORDER-001
type: CODE_USAGE
strength_category: BEHAVIORAL
independent_group: E-ORDER-001
source_fact_ids: [SF-ORDER-001]
```

Claim:

```text
C-ORDER-001
subject: Order
predicate: HAS_LIFECYCLE
value: confirm → arrive → start → complete / cancel
confidence: CONFIRMED
evidence_ids: [E-ORDER-001]
```

This does not establish a standalone FSM component.

---

## SF-DB-001 — Cart schema

```text
id: SF-DB-001
source_type: DB_SCHEMA
source_location:
  archive_sha256: 501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
  file: aristotel_taxi.sql
  object: cart
  lines: 561-568
observation:
  cart contains id_user, product, property, count, booking_limit,
  session_token and complex_update.
```

Evidence:

```text
E-DB-001
type: TABLE_SCHEMA
strength_category: STRUCTURAL
independent_group: E-DB-001
source_fact_ids: [SF-DB-001]
```

Claim:

```text
C-DB-CART-001
subject: Cart
predicate: PERSISTS
object: cart
confidence: CONFIRMED
evidence_ids: [E-DB-001]
```

---

## SF-DB-002 — Cart → User FK

```text
id: SF-DB-002
source_type: DB_SCHEMA
source_location:
  archive_sha256: 501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
  file: aristotel_taxi.sql
  constraint: cart_fk_1
  line: 24508
observation:
  cart.id_user references users.id_user.
```

Evidence:

```text
E-DB-002
type: FOREIGN_KEY
strength_category: STRUCTURAL
independent_group: E-DB-002
source_fact_ids: [SF-DB-002]
```

Claim:

```text
C-CART-USER-001
subject: Cart
predicate: REFERENCES
object: User
confidence: CONFIRMED
evidence_ids: [E-DB-002]
```

---

## SF-DB-003 — site_constant schema and data

```text
id: SF-DB-003
source_type: DB_DATA
source_location:
  archive_sha256: 501d01a80491ad48ec9e5ff0b1b403086d01240b54570d9cd4c172d27e2fd1b6
  file: aristotel_taxi.sql
  table_definition: lines 14976-14992
  data:
    token_interval_for_auth_hash: line 15004
    d_s_sorting_city: line 15008
    d_s_sorting_intercity: line 15009
    d_s_offered_drivers_count: line 15012
    d_s_offered_drivers_duration: line 15062
    auth_code_interval: line 15087
    ticket_booking_duration: line 15099
    session_token_duration: line 15127
observation:
  site_constant stores runtime configuration values used by backend behavior.
```

Evidence:

```text
E-DB-CONFIG-001
type: CONFIGURATION
strength_category: CONFIGURATION
independent_group: E-DB-CONFIG-001
source_fact_ids: [SF-DB-003]
```

This Evidence confirms configuration data exists. The `CONFIGURES` relations above are confirmed separately by CODE Evidence.

---

## 4.2 Cross-source Claims now confirmed

The following claims have independent Code + DB support:

```text
Cart PERSISTS cart
Cart REFERENCES User
Cart exposes read/update/clear/move behavior
Authentication uses configurable auth-code/token intervals
Driver Selection is configured by site_constant values
Order has application-level lifecycle operations
```

The important distinction is:

```text
DB:
Cart → User
```

is structural Evidence, while:

```text
CODE:
Cart → Ticket / Trip / Order / Payment
```

requires application-level tracing.

At this stage only the structurally and behaviorally supported relations are promoted to `CONFIRMED`.

---

# 5. Initial Code-derived Semantic Map

The previous investigation supports the following **provisional** L1 areas:

```text
BACKEND
├── Identity & Access
├── Order / Booking
├── Trip / Fleet
├── Cart
├── Ticketing / Event
├── Financial
├── Pricing
├── Communication
├── Geo / Location
├── Task
└── Configuration / Business Rules
```

### Status

These are **provisional Semantic Frames**.

They are not yet `CONFIRMED` because the current RP lacks complete physical CODE SourceFacts.

---

# 6. Identity & Access

## 6.1 User

```text
Frame: User
identity: INFERRED
provenance: INCOMPLETE
```

Earlier code investigation indicates a common user identity model participating in authentication and other backend operations.

### Required SourceFacts

```text
CODE:
user model / entity
user lookup
authentication consumer
token consumer
```

```text
DB_SCHEMA:
users table
primary key
referencing tables
```

---

## 6.2 Authentication

```text
Frame: Authentication
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified:

```text
token handling
auth-code handling
authenticated API calls
SMS verification
authentication timing/configuration
```

These observations are sufficient to form a research hypothesis:

```text
Capability:
Authentication
```

but not yet sufficient for a final `CONFIRMED` Claim under v2.2.

---

## 6.3 Authorization

```text
Frame: Authorization
identity: INFERRED
provenance: INCOMPLETE
```

Role-related behavior was observed in the previous investigation.

Open question:

```text
Is Authorization a coherent backend capability,
or only distributed role checks?
```

This requires code tracing.

---

# 7. Order / Booking

## 7.1 Order

```text
Frame: Order
identity: INFERRED → candidate CONFIRMED
provenance: INCOMPLETE
```

Earlier code investigation identified lifecycle operations corresponding to:

```text
confirm
assign
arrive
start
complete
cancel
```

This suggests an application-level order lifecycle.

It must not yet be described as a standalone FSM unless the code demonstrates an explicit FSM abstraction.

### Required investigation

Trace:

```text
order creation
→ state transition
→ persistence
→ driver assignment
→ trip transition
→ completion/cancellation
```

---

## 7.2 Driver Selection

```text
Frame: Driver Selection
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified behavior involving:

```text
driver sorting
offers
driver attempts
selection
assignment
```

The database independently contains structures previously identified as:

```text
order_driver
order_driver_attempt
order_driver_select
order_offer
```

This is a strong cross-source hypothesis:

```text
Order
   ↓
Driver Selection
   ↓
Driver
```

But the application-level ownership must be established from code.

---

# 8. Trip / Fleet

## 8.1 Trip

```text
Frame: Trip
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation associated trip behavior with:

```text
driver
schedule
timing
pricing
aggregator-related data
```

Required SourceFacts must identify the actual application symbols implementing these relationships.

---

## 8.2 Driver / Car

```text
Frame: Driver
Frame: Car
identity: INFERRED
provenance: INCOMPLETE
```

Database structures provide independent structural evidence.

Required next step:

```text
CODE → Driver
CODE → Car
CODE → Trip
```

and then cross-validation against DB_SCHEMA.

---

# 9. Cart

## 9.1 Cart

```text
Frame: Cart
identity: CONFIRMED
provenance: COMPLETE for the claims listed in §4.1
```

Direct code Evidence now confirms:

```text
selectCart
updateCart
clearCart
moveCart
```

with exact file/symbol/line provenance in §4.1.

The database independently confirms the persisted `cart` structure and `cart.id_user → users.id_user`.

---

## 9.2 Cart persistence

Earlier database investigation indicates a Cart persistence model and a relation to User.

Provisional relation:

```text
Cart → User
confidence: INFERRED / awaiting provenance completion
```

---

## 9.3 Cart semantic dependencies

Earlier investigation suggested:

```text
Cart → Ticket
Cart → Trip
Cart → Order
Cart → Payment
```

These are deliberately recorded as:

```text
INFERRED
```

until exact code call/data-flow Evidence is attached.

The most important unresolved semantic question remains:

```text
What exactly is the semantic target of Cart.product?
What exactly is the semantic target of Cart.property?
```

This question must be resolved before defining Cart ownership.

---

# 10. Ticketing / Event

## 10.1 Ticket

```text
Frame: Ticket
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation associated Ticket with:

```text
Trip
Schedule
Order
```

The DB provides some structural validation, but application-level direction and ownership require code Evidence.

---

## 10.2 Event / Schedule

```text
Frame: Event
Frame: Schedule
identity: INFERRED
provenance: INCOMPLETE
```

Further code tracing is required to determine whether these form one domain or several capabilities.

---

# 11. Financial

The previous investigation identified the following separate concepts:

```text
Payment
Transaction
Currency Account
Deal
Subscription
Payment Method
Payment Service
Payment Service Method
```

They must remain separate Frames until the code establishes their semantic relations.

---

## 11.1 Payment

```text
Frame: Payment
identity: INFERRED
provenance: INCOMPLETE
```

The database provides structural evidence around:

```text
payment
payment_log
payment_status
payment_method
payment_service
payment_service_method
```

The code-first pass must now establish:

```text
who initiates payment
what entity payment belongs to
what completes payment
how provider results affect domain state
```

---

## 11.2 Transaction / Account / Deal / Subscription

```text
Frame: Transaction
Frame: Currency Account
Frame: Deal
Frame: Subscription
identity: INFERRED
provenance: INCOMPLETE
```

Important research question:

> Is Payment a reusable capability, or only one part of a broader financial capability already implemented by Account/Transaction/Deal?

This is precisely the kind of question the Semantic Graph is intended to answer.

---

# 12. External Payment Providers

Earlier investigation identified:

```text
Stripe
YooKassa
```

as payment integrations.

Provisional relations:

```text
Payment → Stripe
Payment → YooKassa
```

Status:

```text
INFERRED
```

until exact integration symbols and configuration references are captured.

Provider configuration must remain connected to the concrete source:

```text
CONFIGURATION
   ↓
SourceFact
   ↓
Evidence
   ↓
Payment / Provider Claim
```

---

# 13. Pricing

```text
Frame: Pricing
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified behavior around:

```text
tariff
price-time functions
commission
promocode
```

Provisional consumers:

```text
Order
Trip
Payment / Financial
```

Ownership remains UNKNOWN.

This is an important candidate for a shared capability.

---

# 14. Communication

```text
Frame: Communication
identity: INFERRED
provenance: INCOMPLETE
```

Observed mechanisms:

```text
SMS
Email
WhatsApp
Telegram
Contacts
Messages
```

Important semantic observation:

```text
SMS
├── Communication
└── Authentication / Verification
```

Therefore the same external mechanism may participate in more than one capability.

This is Evidence against identifying domains purely from integration names.

---

# 15. Geo / Location

```text
Frame: Geo / Location
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified location behavior around:

```text
user location
address / geocoding
distance
time
map operations
```

Provisional dependencies:

```text
User
Order
Driver Selection
```

Exact relations require code tracing.

---

# 16. Task

```text
Frame: Task
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified:

```text
task list
task actions
task logs
task status
```

Open question:

```text
Is Task a domain capability,
an infrastructure capability,
or a workflow mechanism shared by domains?
```

---

# 17. Configuration

## 17.1 Configuration as a semantic object

Configuration is treated as a cross-cutting capability.

The existence of:

```text
site_constant.X
```

creates:

```text
SourceFact(DB_DATA)
```

It does not by itself create:

```text
CONFIGURES relation
```

The relation requires code Evidence showing:

```text
site_constant.X
        ↓
read / lookup
        ↓
application behavior
```

---

## 17.2 Authentication configuration

Previously identified examples include:

```text
session_token_duration
token_interval_for_auth_hash
auth_code_interval
```

The following are now confirmed as application configuration relations because both
the DB_DATA values and their CODE reads are available:

```text
Driver Selection → d_s_sorting_city
Driver Selection → d_s_offered_drivers_count
Driver Selection → d_s_offered_drivers_duration
Authentication → auth_code_interval
```

`session_token_duration` is present in the backend configuration model and DB_DATA,
but its active runtime use requires further tracing before the CONFIGURES claim is promoted.


---

# 18. Provisional Cross-domain Graph

The current code-first hypothesis is:

```text
                         User
                    /      |       \
                   /       |        \
          Authentication  Cart     Order
                 |          |       / | \
                 |          |      /  |  \
        Communication       |     /   |   Pricing
                            |    /    |
                            |   /    Driver Selection
                            |  /          |
                            | /          Driver
                          Ticket         |
                            |            |
                          Trip ----------+
                            |
                         Schedule

Order / Cart / Trip
        ↓
      Payment
        ↓
 Transaction
        ↓
 Currency Account
        ↓
 Deal / Subscription
```

This diagram is **not yet a confirmed graph**.

It is the current research hypothesis generated from the available code investigation and partial DB validation.

---

# 19. Platform Candidates

The following are explicitly **Candidates**, not Platforms:

```text
Platform Candidate: Auth
status: RESEARCHING

Platform Candidate: Cart
status: RESEARCHING

Platform Candidate: Payment
status: RESEARCHING
```

No Candidate currently satisfies the v2.2 Decision Gate.

In particular:

```text
preconditions = UNKNOWN
human approval = absent
```

---

# 20. Main Semantic Findings

## Finding F-01

The backend contains multiple functional areas that cross the future platform boundaries.

```text
Pricing
Geo
Communication
Configuration
```

are not naturally owned by one of:

```text
Auth
Cart
Payment
```

**Status:** `INFERRED`  
**Provenance:** incomplete

---

## Finding F-02

Cart is a real application capability rather than merely a persistence structure.

Evidence basis:

```text
explicit Cart application operations
+
Cart persistence structures
```

**Status:** `INFERRED pending provenance completion`

---

## Finding F-03

Payment exists within a broader financial context.

```text
Payment
Transaction
Account
Deal
Subscription
```

should not automatically be collapsed into one platform boundary.

**Status:** `INFERRED`

---

## Finding F-04

Authentication is intertwined with User, Verification and Communication.

```text
Authentication
      ↓
User
      ↓
SMS / Verification
```

This makes the Auth boundary a semantic question, not a class/package extraction.

**Status:** `INFERRED`

---

## Finding F-05

Configuration is cross-cutting.

`site_constant` must be interpreted together with its application usage.

**Status:** `INFERRED`

---

# 21. Critical UNKNOWN

The following questions remain open.

### Cart

```text
What are Cart.product and Cart.property?
What owns Cart lifecycle?
Which Cart relations are direct versus indirect?
```

### Auth

```text
Where exactly is authentication implemented?
Where is authorization implemented?
Where does User Management begin/end?
Which SMS operations belong to verification?
```

### Payment

```text
What is the payment aggregate?
How does payment affect Order/Ticket/Trip?
Where is Transaction owned?
What is the relation to Account/Deal/Subscription?
```

### Shared capabilities

```text
What is the exact reusable boundary of Pricing?
What is the exact reusable boundary of Geo?
What is the exact reusable boundary of Communication?
```

### Configuration

```text
Which site_constant values are actually consumed?
By which capabilities?
Are any values legacy or dead?
```

---

# 22. Provenance Completion Report

RP-01 cannot yet pass the v2.2 Provenance Completeness Gate.

## Missing

For material CODE findings:

```text
repository
commit/revision
file
symbol
line_start
line_end
```

For DB findings:

```text
database
schema
object
column/constraint
snapshot
```

For site_constant findings:

```text
database
table
row identity
column
snapshot
```

## Required action

The next execution of RP-01 must attach concrete SourceFacts to every material finding.

The existing semantic map should then be re-evaluated.

Claims may:

```text
remain INFERRED
→ become CONFIRMED
→ become UNKNOWN
→ become CONFLICT
```

depending on actual Evidence.

---

# 23. Gap Report

## confirmed_findings

The following material findings now have concrete SourceFact provenance:

```text
Cart capability exists
Cart persists to cart
Cart references User
Cart exposes select/update/clear/move behavior
Order lifecycle operations exist
Authentication uses auth_code_interval
Driver Selection uses site_constant driver-selection parameters
site_constant contains the investigated configuration values
```

## inferred_findings

```text
Identity & Access exists
Order / Booking exists
Trip / Fleet exists
Cart exists
Ticketing / Event exists
Financial capability cluster exists
Pricing exists
Communication exists
Geo / Location exists
Task exists
Configuration is cross-cutting
```

## unknowns

See §21.

## open_conflicts

No explicit source contradiction is currently recorded.

Absence of contradiction must not be confused with confirmation.

## next_actions

```text
1. Continue SourceFact extraction for the remaining L1 areas.
2. Trace application-level Cart → Ticket / Trip / Order / Payment relations.
3. Trace Payment → Transaction → Account → Deal.
4. Trace Authentication → User → Role → Token → Verification.
5. Complete site_constant usage mapping.
6. Cross-validate each new application-level relation against DB_SCHEMA/API where available.
7. Publish the complete Backend Semantic Graph v0.1 snapshot only after the material L1 graph passes the Provenance Completeness Gate.
```

---

# 24. Status

```text
Research Pass: RP-01
Mode: CODE_FIRST
Semantic map: PROVISIONAL
Provenance: PARTIALLY COMPLETE — concrete SourceFacts exist for the claims in §4.1
Decision Gate: NOT REACHED
Platform decisions: NONE
```

The principal result of this pass is not a list of proposed platforms.

It is the initial hypothesis of the existing backend territory:

```text
Backend
   ↓
Functional Areas
   ↓
Capabilities
   ↓
Cross-domain dependencies
   ↓
Questions requiring deeper research
```

The next pass must improve **evidence quality and provenance**, not prematurely narrow the investigation to Cart, Auth or Payment.
