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

### Important provenance limitation

The material currently available for this RP contains the results of an earlier backend code investigation, but does not contain the original PHP repository snapshot with stable commit/file/line references.

Therefore this document **does not invent source locations**.

Where a finding originates from the earlier code investigation, it is recorded as:

```text
provenance_status = INCOMPLETE
```

Such a finding may guide further research, but cannot by itself satisfy the v2.2 `CONFIRMED` provenance gate.

This distinction is intentional.

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

The previous investigation identified backend application areas in PHP code, including controller/model/API functionality and application methods associated with:

```text
authentication
authorization
users
orders
driver selection
trips
drivers
cars
cart
tickets
payments
transactions
accounts
deals
subscriptions
pricing
communication
geo/location
tasks
configuration
```

### Provenance status

```text
CODE SourceFacts:
INCOMPLETE
```

Required completion:

```text
repository
revision / commit
file
symbol
line_start
line_end
```

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
identity: INFERRED
provenance: INCOMPLETE
```

Earlier investigation identified explicit application behavior corresponding to:

```text
selectCart
updateCart
clearCart
moveCart
```

This is strong Evidence that Cart is an actual application capability.

However, under v2.2 the final Claim must point to concrete:

```text
file
symbol
line range
revision
```

before it can become `CONFIRMED`.

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

These remain:

```text
Configuration Claims
confidence: INFERRED
provenance: INCOMPLETE
```

until the corresponding code reads are attached.

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

At the current artifact level:

```text
None of the material semantic findings are promoted to final
CONFIRMED status because concrete SourceFact provenance is incomplete.
```

This is intentional and follows v2.2.

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
1. Obtain/fix the exact backend repository snapshot.
2. Extract concrete CODE SourceFacts.
3. Attach each material finding to exact symbols and line ranges.
4. Re-run cross-source validation against DB_SCHEMA.
5. Extract DB_DATA SourceFacts for relevant site_constant rows.
6. Trace every relevant site_constant read in CODE.
7. Recalculate Claim confidence.
8. Publish Backend Semantic Graph v0.1 only after Provenance Completeness Gate passes.
```

---

# 24. Status

```text
Research Pass: RP-01
Mode: CODE_FIRST
Semantic map: PROVISIONAL
Provenance: INCOMPLETE
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
