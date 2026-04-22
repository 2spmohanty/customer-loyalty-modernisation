# Stored Procedures & Triggers — Migration Assessment

>**Document:** 05-stored-procedures-triggers.md  
**Status:** ```Blocked — confirmed in Month 1 Discovery```  
**Last Updated:** 22 Apr 2026  

---

## Why This Document Exists

Business logic embedded in the database layer —
stored procedures, triggers, scheduled jobs —
must be catalogued before schema design begins.
Gate G1 does not pass without it.

---

## Classification Framework

| Class | Definition | Count (TBC M1) | Action |
|-------|------------|----------------|--------|
| Simple | Single table, audit / logging only | TBC | Atlas Database Trigger |
| Moderate | Multi-step, called by application | TBC | Application service layer |
| Complex | Multi-table, transactions, core business logic | TBC | Application layer + full test cycle |
| Scheduled | Batch jobs — point expiry, tier recalculation | TBC | Atlas Scheduled Trigger |
| Unknown | No documentation, no owner | TBC | Freeze — do not migrate until understood |

```
Unknown class = Gate G1 blocker.
Nothing proceeds until every item is classified.
```

---

## MongoDB Equivalents

| RDBMS Concept | MongoDB Equivalent | Notes |
|---------------|--------------------|-------|
| Stored procedure | Application service layer | Preferred — testable, version controlled |
| Stored procedure (simple) | Atlas Function | Lightweight, stateless only |
| Scheduled job | Atlas Scheduled Trigger | Replaces SQL Agent / pg_cron |
| AFTER INSERT / UPDATE trigger | Atlas Database Trigger | Async — does not block write |
| BEFORE DELETE trigger | Application validation | No pre-event trigger in MongoDB |
| Audit trigger | Atlas Database Trigger → audit collection | Direct replacement |
| FK constraint | Application validation + $jsonSchema | No native FK in MongoDB |
| Database view | MongoDB View — aggregation pipeline | Read-only |

---

## Delivery Impact

```
BEST CASE — few simple triggers, no procedures:
  M1 audit completes in 1 week
  No timeline impact

MODERATE — 10–20 procedures, mixed triggers:
  M1 audit: 2–3 weeks
  Consumes part of limited refactoring budget
  Monitor closely at G1 review

WORST CASE — 100+ procedures, complex logic:
  Gate G1 delayed
  Escalate to steering committee immediately
  Options: defer to backlog / increase budget
  Peak season cutover gate at risk
```

---

## Open Items

| # | Item | Owner | Required By |
|---|------|-------|-------------|
| 1 | Full stored procedure inventory | Customer DB team | End of M1 |
| 2 | Full trigger inventory | Customer DB team | End of M1 |
| 3 | Classification of all items | Consulting + Customer | G1 gate |
| 4 | Owner assigned per complex item | Customer Engineering | G1 gate |
| 5 | Atlas Trigger vs app layer decision per item | Consulting | G1 gate |

---

