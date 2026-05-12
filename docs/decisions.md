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

## 2026-05-11 — Keep model checkpoints out of the public repo
**Context:** The Phase 1 binary checkpoint is useful locally, but the repository is public and should stay lightweight and reviewable.
**Decision:** Keep `.pt` checkpoint files gitignored. Share model artifacts out-of-band or retrain from the committed training code when a new machine needs them.
**Why:** Recruiters can evaluate the implementation, metrics, and app flow without downloading large binary weights, and we avoid accidentally treating an early experimental model as a polished distributable artifact.

## 2026-05-11 — Clean up Supabase advisor warnings
**Context:** After Phase 1 wiring, Supabase advisors flagged a live `public.rls_auto_enable()` SECURITY DEFINER function as externally executable and flagged direct `auth.uid()` calls in RLS policies as a performance issue.
**Decision:** Add migrations to revoke client execute privileges from `rls_auto_enable()` when present and to rewrite RLS policies with `(select auth.uid())`.
**Why:** These are small hardening/performance fixes that keep the public project cleaner before Phase 2 without changing app behavior.

## 2026-05-11 — Phase 1 phone flow verified
**Context:** The first real-device Phase 1 test covered backend inference, image upload, Supabase scan persistence, signed history thumbnails, and binary label display.
**Decision:** Treat Phase 1 wiring as verified. Keep unsupported non-PNG/non-JPEG uploads rejected for now, with friendlier app copy. Validate/infer before storing so rejected images do not leave scan objects behind.
**Why:** The MVP path is working end to end, and constraining uploads to PNG/JPEG keeps storage and image decoding simple until Phase 2/UX polish.

## 2026-05-11 — Grant backend table privileges
**Context:** Real-device Phase 1 testing reached `POST /predictions`, but the scan insert failed with Postgres error `42501` because `service_role` did not have table privileges on `public.scans`.
**Decision:** Add migration `0004_grant_backend_table_privileges.sql`, granting `service_role` schema usage plus `SELECT` and `INSERT` on `public.scans`.
**Why:** The backend still validates the user's JWT and filters by that user ID, while Supabase PostgREST needs explicit table privileges before the service-role key can save and read scan rows.

## 2026-05-11 — Phase 1 binary baseline metrics
**Context:** The first full Phase 1 binary ResNet18 was trained on HAM10000 using the lesion-grouped split and weighted cross-entropy.
**Decision:** Use the epoch-5 checkpoint at `ml/models/lesion_classifier_binary.pt` as the initial Phase 1 local baseline. Held-out test metrics: accuracy `0.7505`; `non_suspicious` precision `0.9243`, recall `0.7494`, F1 `0.8277`, support `1189`; `suspicious` precision `0.4302`, recall `0.7550`, F1 `0.5481`, support `298`.
**Why:** This is good enough to validate the end-to-end product pipeline. The `suspicious` recall is intentionally much stronger than precision because the weighted loss pushes the model to avoid missing suspicious examples, but the low precision means the output must remain framed as experimental classification, not diagnosis.

## 2026-05-11 — Weighted loss for Phase 1 binary training
**Context:** The Phase 1 binary HAM10000 grouping is still imbalanced: most rows map to `non_suspicious`, while `suspicious` is the minority class.
**Decision:** `ml/training/train.py` uses class-weighted cross-entropy based on the training split counts.
**Why:** A plain loss would let the model get deceptively high accuracy by leaning toward the majority class. Weighted loss is simpler than weighted sampling for the MVP and keeps the dataset split unchanged.

## 2026-05-09 — Lesion-grouped train/val/test splits for HAM10000
**Context:** HAM10000 has 10,015 images of 7,470 unique lesions — about 18% of lesions have multiple images of the same physical spot (different angles or visits, sharing a `lesion_id`). A naive random image-level split would put the same lesion in both train and test.
**Decision:** `ml/training/prepare_ham10000.py` splits at the lesion level: all images of one lesion go to one split. Splits are stratified by canonical label so rare classes (`df`, `vasc`) keep similar proportions across train/val/test. Ratios: 70/15/15 with seed 42.
**Why:** Image-level splits would let the model memorize specific lesions instead of learning patterns, producing inflated test metrics that don't generalize. Lesion-grouped splits are the standard practice for HAM10000 and the only way to get a fair evaluation.

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
**Decision:** Both labels are excluded from the canonical set until Phase 3 when a dataset that supports them is added. The database constraint and label table will be updated in a future Phase 3 migration.
**Why:** Including them now would create canonical classes with zero training examples, which silently breaks training.

## 2026-04-08 — Supervised baseline before RL
**Context:** Tempting to jump into RL since it's a learning goal.
**Decision:** Ship a working supervised classifier first; defer RL to a later phase for decision-support tasks (not the classifier itself).
**Why:** RL on top of an unstable pipeline would conflate two sources of failure. A solid supervised baseline gives us a measurable reference point.

## 2026-04-08 — Repo structure
**Context:** Needed a clear top-level layout before scaling beyond a single component.
**Decision:** `mobile/`, `backend/`, `ml/`, `docs/`, `supabase/` at root, plus `PROJECT.md` and `AGENTS.md`.
**Why:** Each folder maps to one responsibility. Keeps the three runtimes (RN, FastAPI, PyTorch) cleanly separable and matches the phased build plan.
