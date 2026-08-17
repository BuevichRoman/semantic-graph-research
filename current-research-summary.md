# Current Research State

```text
STATUS: COMPLETED
Current Methodology: v2.3
```

Актуальное кумулятивное состояние исследования. Историческая сводка ранней
стадии — [`research-summary.md`](research-summary.md) (`STATUS: HISTORICAL`).

---

## 1. Нормативные документы

```text
methodology/research-methodology-v2.3.md
methodology/prototype-structures-v2.3.md
```

- [`methodology/research-methodology-v2.3.md`](methodology/research-methodology-v2.3.md)
  — методология исследования: Evidence, Claim, Confidence, Research Pass,
  Semantic Synthesis, три набора статусов (раздел 9.1), lifecycle синтеза
  (раздел 23.1).
- [`methodology/prototype-structures-v2.3.md`](methodology/prototype-structures-v2.3.md)
  — нормативная спецификация протоструктур и JSON Schema.

Остальные файлы в `methodology/` (`v1-ru`, `v2`, `v2-rev2`, `v2.2`) — исторические
редакции.

---

## 2. Состав корпуса

```text
Research:      research-passes/
MCR:           mcr/
Experiments:   experiments/
Sources:       source-docx/
```

- [`research-passes/`](research-passes/) — Research Passes RP-01 … RP-37.
- [`mcr/`](mcr/) — Methodology Change Requests MCR-01 … MCR-07.
- [`experiments/`](experiments/) — эксперименты по кристаллизации и scan'ы правил.
- `source-docx/` — исходные документы методологии.

---

## 3. Current confirmed results

### 1. Authentication — CONFIRMED

```text
Taxi Frontend
    ──AUTHENTICATES_WITH──>
Core Backend
```

**Research Result:** `CONFIRMED`

**Source:**
[`research-passes/rp-37-authentication-credential-value-flow-v0.4.md`](research-passes/rp-37-authentication-credential-value-flow-v0.4.md)

Механизм в кратком виде: frontend получает `auth_hash` через `POST /auth`,
обменивает его на пару `token` + `u_hash` через `POST /token`, и передаёт эту
пару в последующие API calls; backend разбирает `u_hash` относительно `token` и
восстанавливает authenticated User.

Полный Evidence chain и provenance (frontend/backend snapshots, конкретные файлы
источников) — в разделах 8 и 9 указанного RP-37 v0.4. Здесь они не дублируются.

RP-37 v0.3 и RP-37 reconciliation v0.1 остаются историческими research records
и не являются источником итогового Claim.

---

Других подтверждённых результатов на текущий момент в кумулятивное состояние не
включено. Это не полный реестр всех исторических результатов корпуса, а текущее
состояние, сознательно передаваемое дальше — для выделения Authentication
Platform.
