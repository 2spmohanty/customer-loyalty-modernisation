# Delivery Roadmap

>**Document:** 03-roadmap.md .  
**Status:** ```DRAFT``` .  
**Last Updated:** 21 April 2026 .





---

## Delivery Philosophy

```
Value every month — not a big-bang reveal at Month 6.
Each phase has a gate. No phase proceeds without it.
Rollback is available at every step.
Legacy RDBMS stays live until Month 5 gate is passed.
```

---

## Timeline at a Glance

```
May 2026        Jun 2026        Jul 2026        Aug 2026        Sep 2026        Oct 2026
────────────────────────────────────────────────────────────────────────────────────────
M1 DISCOVERY    M2 BUILD        M3 MIGRATION    M4 SCALE        M5 CUTOVER      M6 HANDOVER
│               │               │               │               │               │
├─ Data audit   ├─ APIs live    ├─ 32M migrated ├─ 50% traffic  ├─ 100% Atlas   ├─ Team owns it
├─ Schema ✓     ├─ CDC active   ├─ Shadow mode  ├─ 2× load ✓    ├─ Loyalty API  ├─ Legacy plan
├─ Shard key    ├─ EU zone up   ├─ 5% traffic   ├─ GDPR audit   ├─ 5× load ✓    ├─ Backlog hand
└─ GDPR agreed  └─ Upskill →   └─ Rollback ✓   └─ Runbooks →   └─ Read-only DB └─ Closed ✓
│               │               │               │               │               │
[G1]            [G2]            [G3]            [G4]            [✓ Stable 2wk]  [✓ Ops sign-off]
```

---

## Workstream Map

```
WORKSTREAM          M1      M2      M3      M4      M5      M6
────────────────────────────────────────────────────────────────
Data Audit          ████▌
Schema Design       ████████
Shard Key           ███▌
GDPR Strategy       ████
Infra Provision     ██▌ ████
Bulk Migration              ██▌ ████
CDC → Atlas                 ████████████████████▌
Reverse CDC                                         ████████▌
Shadow Mode                         ████████▌
Traffic 5%                          ████
Traffic 50%                                 ████▌
Traffic 100%                                        ████████▌
5× Load Sim                                         ████
GDPR Audit                                  ████
Loyalty API                                         ████
Team Upskilling             ██▌ ████████████████████████████
Ops Runbooks                                ████████▌
Stabilisation                                               ████
────────────────────────────────────────────────────────────────
GATES               G1      G2      G3      G4      ✓       ✓


```

---

## Month-by-Month

### Month 1 — Discovery + Foundation
| Deliverable | Detail |
|-------------|--------|
| Data audit | All 45+ tables mapped — edge cases and business logic identified |
| Schema design | Target document model validated and signed off |
| Shard key | `{ customer_id: 1 }` confirmed for 32M records + 5× peak |
| GDPR strategy | EU zone confirmed with DPO — eu-west-1 or eu-central-1 |
| Infra | Atlas cluster provisioned across required regions |
| CDC tooling | DMS vs Debezium confirmed based on source RDBMS type |

**Gate G1:** Panel sign-off on document schema and multi-region topology

---

### Month 2 — Build + Pilot
| Deliverable | Detail |
|-------------|--------|
| Core APIs | Customer profile APIs live on MongoDB Atlas |
| CDC active | Pipeline starts at beginning of M2 — dual-write operational |
| Bulk migration | CDC pipeline extracts full historical dataset into Atlas |
| EU zone | Isolated and GDPR controls verified |
| Upskilling | Customer team embedded with delivery team |

**Gate G2:** p95 latency validated at ≤30ms on representative dataset

---

### Month 3 — Migration + Validation
| Deliverable | Detail |
|-------------|--------|
| Full migration | 32M records reconciled — 12M active + 20M inactive |
| Shadow mode | Atlas reads compared vs legacy — zero customer exposure |
| 5% traffic | First real traffic on Atlas via API Gateway |
| Rollback tested | Procedure documented and verified |

**Gate G3:** Zero data inconsistency · EU records confirmed in EU region

---

### Month 4 — Scaling + Hardening
| Deliverable | Detail |
|-------------|--------|
| 50% traffic | Gateway weighted routing — non-EU first, EU after audit |
| 2× load | Performance validated at average peak load |
| GDPR audit | Compliance confirmed — DPO sign-off received |
| Runbooks | Drafted and owned by customer ops team |

**Gate G4:** p95 ≤30ms at 50% load · No GDPR findings outstanding

---

### Month 5 — Cutover + Peak Readiness
| Deliverable | Detail |
|-------------|--------|
| 100% cutover | All traffic on Atlas — gateway fully shifted |
| Loyalty API | Live on Atlas — committed deliverable ✓ |
| Legacy | Moved to read-only — reverse CDC active as safety net |
| 5× simulation | Peak season load test run and passed |
| New features | Shipping on new architecture |

**Gate:** 2 weeks stable · 5× load passed · No rollback triggered

---

### Month 6 — Stabilisation + Handover
| Deliverable | Detail |
|-------------|--------|
| Team autonomous | Customer team leads — consulting team shadows only |
| CDC decommissioned | Dual-write and reverse CDC both stopped |
| Legacy plan | Decommission scoped and handed over — Month 9–12 |
| Backlog | Items prioritised and documented |
| Engagement | Formally closed |

**Gate:** Ops signs off on readiness · Consulting team exits

---

## Gates Summary

| Gate | Month | Pass Criteria |
|------|-------|---------------|
| G1 | May | Schema signed off · Multi-region topology agreed |
| G2 | Jun | p95 ≤30ms on representative dataset |
| G3 | Jul | Zero data inconsistency · EU records in EU region |
| G4 | Aug | p95 ≤30ms at 50% load · GDPR audit clean |
| ✓ | Sep | 2 weeks stable · 5× load passed |
| ✓ | Oct | Ops sign-off · Team autonomous |

```
No gate passed = no progression to next phase.
Gate failure triggers scope review — not timeline compression.
```

---

## Rollback Strategy

```
Every phase has an explicit rollback trigger.
Rollback is a gateway weight change — seconds, not hours.

Phase       Rollback trigger              Action
────────────────────────────────────────────────────────
Shadow      Discrepancy rate > 0          Stop shadow — investigate
5% traffic  Error rate or latency spike   Gateway → 0% Atlas, 100% legacy
50% traffic Same as above                 Gateway → 0% Atlas, 100% legacy
100% cutover Same — within 2-week window  Reverse CDC active — flip gateway
Post 2 weeks Legacy decommission halted   Reverse CDC extends until resolved
```

---

## Critical Path

```
These items block everything downstream.
Delay here = delay to cutover gate.

  DPO engagement            → must complete Month 1
  Schema sign-off (G1)      → blocks all build work
  CDC pipeline active       → must start Month 2
  Bulk migration complete   → must complete Month 3
  GDPR audit (G4)           → blocks EU traffic shift
  5× load data strategy     → must be agreed Month 2
                               (DPO sign-off on test data)
```

---

## What Is Not In Scope (Month 1–6)

```
  Full application refactoring
    Legacy SQL patterns removed from app code
    → Backlog: Month 7+

  Legacy RDBMS decommission
    Requires all integrations validated
    → Backlog: Month 9–12

  Additional region expansion
    Beyond EU + primary region
    → Backlog: post go-live
```

---
