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

## 2026-05-07 — Auth handled directly by mobile, not proxied through backend
**Context:** Backend had `/auth/register` and `/auth/login` stub endpoints intended to proxy calls to Supabase Auth. Meanwhile, the mobile app already initialised a Supabase JS client capable of auth directly.
**Decision:** Mobile talks to Supabase Auth directly. The backend validates the user's JWT on each request but never proxies auth calls. The backend `/auth` router was deleted.
**Why:** The proxy adds a network hop and a maintenance burden with no security benefit. Supabase's JS SDK is purpose-built for this pattern.

## 2026-05-07 — Backend uses Supabase service-role key
**Context:** Two options for backend↔Supabase: (a) forward the user's JWT and rely on RLS, or (b) use the service-role key and enforce user filtering in application code.
**Decision:** Backend uses the service-role key (bypasses RLS). The backend must always filter by the user ID extracted from the validated JWT — it is solely responsible for ensuring users only see their own data.
**Why:** Service-role approach is simpler to reason about in a small API. RLS insert policy would require forwarding the JWT through every Supabase call, adding complexity without additional safety when the backend already validates the JWT.

## 2026-05-07 — Phase 1 uses binary classifier (suspicious / non_suspicious)
**Context:** The canonical label set has 7 HAM10000 classes. Training a 7-class model as the very first step adds complexity before the pipeline is validated end-to-end.
**Decision:** Phase 1 trains a binary ResNet18 (`num_classes=2`) using the grouping: `suspicious` = {melanoma, basal_cell_carcinoma, actinic_keratosis}; `non_suspicious` = everything else.
**Why:** A working binary classifier validates the full pipeline (data → training → inference → API → mobile) with minimal ML complexity. Phase 2 upgrades to 7 classes once the pipeline is proven.

## 2026-05-07 — squamous_cell_carcinoma and seborrheic_keratosis deferred to Phase 3
**Context:** These two labels are in the ISIC Archive but not in HAM10000 (SCC is folded into actinic_keratosis; seborrheic keratosis has no separate HAM10000 class).
**Decision:** Both labels are excluded from the canonical set until Phase 3 when a dataset that supports them is added. The database constraint and label table will be updated in migration `0003`.
**Why:** Including them now would create canonical classes with zero training examples, which silently breaks training.

## 2026-04-08 — Supervised baseline before RL
**Context:** Tempting to jump into RL since it's a learning goal.
**Decision:** Ship a working supervised classifier first; defer RL to a later phase for decision-support tasks (not the classifier itself).
**Why:** RL on top of an unstable pipeline would conflate two sources of failure. A solid supervised baseline gives us a measurable reference point.

## 2026-04-08 — Repo structure
**Context:** Needed a clear top-level layout before scaling beyond a single component.
**Decision:** `mobile/`, `backend/`, `ml/`, `docs/`, `supabase/` at root, plus `PROJECT.md` and `AGENTS.md`.
**Why:** Each folder maps to one responsibility. Keeps the three runtimes (RN, FastAPI, PyTorch) cleanly separable and matches the phased build plan.
