# Decisions

Lightweight log of meaningful technical decisions. Newest at the top.

## Template
```
## YYYY-MM-DD — Title
**Context:** what was the question?
**Decision:** what did we choose?
**Why:** what were the alternatives and why did we reject them?
```

---

## 2026-04-08 — Repo structure
**Context:** Needed a clear top-level layout before scaling beyond a single component.
**Decision:** `mobile/`, `backend/`, `ml/`, `docs/`, `supabase/` at root, plus `PROJECT.md` and `AGENTS.md`.
**Why:** Each folder maps to one responsibility. Keeps the three runtimes (RN, FastAPI, PyTorch) cleanly separable and matches the phased build plan.

## 2026-04-08 — Supervised baseline before RL
**Context:** Tempting to jump into RL since it's a learning goal.
**Decision:** Ship a working supervised classifier first; defer RL to a later phase for decision-support tasks (not the classifier itself).
**Why:** RL on top of an unstable pipeline would conflate two sources of failure. A solid supervised baseline gives us a measurable reference point.
