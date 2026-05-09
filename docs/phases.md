# Build Phases

This document is the master build plan for CLEAR. Each phase has a clear goal, a definition of done, and a note on what the next phase unlocks. Start small, validate, then expand.

---

## Phase 0 — Foundation (do this before writing any ML or product code)

**Goal:** Every component can talk to every other component. No features yet — just plumbing.

### Supabase
- [x] Create Supabase project (cloud) and link it locally with the CLI
- [x] Apply migration `0001_init.sql` (profiles + scans tables, RLS)
- [x] Apply migration `0002_...sql` (profile auto-creation trigger, schema constraints)
- [x] Create a private Supabase Storage bucket called `scan-images`
- [x] Verify: sign up a test user → profile row appears automatically in `public.profiles`

### Backend
- [x] Wire `config.py` — Supabase client initialised using `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`
- [x] Add image upload helper: receive bytes → upload to `scan-images` bucket → return storage path (signed URLs are issued on demand; `scans.image_url` stores the path)
- [x] Add JWT validation middleware: extract user ID from the Supabase JWT on every request
- [x] Verify: `GET /health` returns `{"status": "ok"}`

### Mobile
- [x] Implement login/sign-up screen using Supabase JS client directly
- [x] Store session JWT; attach it as `Authorization: Bearer <token>` on every backend API call
- [x] Create `mobile/.env` from `mobile/.env.example`; fill in real Supabase + API values
- [x] Verify: sign in on device → session persists across app restarts

### ML
- [x] Confirm Python package imports work: `python -m ml.training.train` from project root
- [x] Write `ml/preprocessing.py` — single `get_transforms(split)` function used by both training and inference (resize to 224×224, ImageNet normalize)
- [x] Verify: `from ml.preprocessing import get_transforms` works without errors

**Phase 0 done when:** a logged-in user can hit the backend from the phone and the backend can authenticate the request and return a stub response without errors.

---

## Phase 1 — Binary Classifier MVP

**Goal:** End-to-end working product with a simple "suspicious / not suspicious" answer.

### ML
- [ ] Download HAM10000 dataset; place in `ml/data/raw/ham10000/`
- [ ] Write `ml/training/prepare_ham10000.py` — reads HAM10000 CSVs, applies canonical label translation, writes `ml/data/splits/ham10000.csv` (`image_path,label` columns)
- [ ] For binary training, map labels using the binary grouping (see `ml/data/README.md` Rule 5)
- [ ] Train ResNet18 with `num_classes=2`; save checkpoint to `ml/models/lesion_classifier_binary.pt`
- [ ] Evaluate on the held-out test split; record accuracy and per-class metrics in `docs/decisions.md`
- [ ] Wire `ml/inference/predict.py` to load the checkpoint, apply `get_transforms("val")`, run forward pass, return `{label, confidence}`

### Backend
- [ ] Wire `backend/app/services/inference.py` to call `ml/inference/predict.py`
- [ ] `POST /predictions`: receive image → upload to storage → run inference → insert scan row → return `{label, confidence, image_url, scan_id}`
- [ ] `GET /scans`: fetch all scans for the authenticated user from Supabase, return list

### Mobile
- [ ] `ScanScreen`: open camera / image picker → send image to `POST /predictions` → display result
- [ ] `HistoryScreen`: call `GET /scans` → show list of past scans with label and confidence
- [ ] Display label as a human-readable string (e.g. `suspicious` → "Needs a closer look")

**Phase 1 done when:** a user can photograph a skin lesion on their phone, get a "suspicious" or "not suspicious" result, and see their scan history.

---

## Phase 2 — Full HAM10000 Classifier (7 classes)

**Goal:** Replace the binary classifier with one that names the specific lesion type.

### ML
- [ ] Retrain with all 7 HAM10000 canonical labels (`num_classes=7`)
- [ ] Address class imbalance — HAM10000 is ~67% nevus; use weighted sampling or loss weighting; document choice in `docs/decisions.md`
- [ ] Save checkpoint to `ml/models/lesion_classifier_ham10000.pt`
- [ ] Evaluate per-class precision/recall; record in `docs/decisions.md`

### Backend
- [ ] Update `MODEL_PATH` in `.env` to point to new checkpoint
- [ ] No other backend changes needed — inference interface is the same

### Database
- No migration needed. `0002_profile_trigger_and_constraints.sql` already includes all Phase 1 binary labels (`suspicious`, `non_suspicious`) and all 7 HAM10000 canonical labels in the `prediction` CHECK constraint. The next migration will be `0004` (`0003` was applied in Phase 0 to lock down the `handle_new_user` RPC).

### Mobile
- [ ] Update label → display string mapping for all 7 classes (e.g. `melanoma` → "Melanoma", `nevus` → "Common Mole")
- [ ] Optionally show a short description of each lesion type

**Phase 2 done when:** the app returns one of the 7 specific lesion names instead of just "suspicious."

---

## Phase 3 — Multi-Dataset Expansion

**Goal:** Improve model coverage by adding more datasets, unlocking `squamous_cell_carcinoma` and `seborrheic_keratosis` which HAM10000 lacks.

### Candidate datasets
- **ISIC Archive** — adds `squamous_cell_carcinoma`, `seborrheic_keratosis`, more `nevus` diversity
- Other public dermatology datasets (document each addition in `docs/decisions.md`)

### Process for each new dataset
1. Place raw files in `ml/data/raw/<dataset-name>/`
2. Add a translation table in `ml/data/README.md` under "Dataset → canonical translation"
3. Write a preparation script in `ml/training/prepare_<dataset>.py`
4. Merge splits, retrain, evaluate
5. Write a new migration to update the `prediction` CHECK constraint

**Phase 3 done when:** the model achieves solid baseline performance across all canonical labels and coverage gaps are documented.

---

## Phase 4 — Confidence Thresholding & UX Polish

**Goal:** Make the app more responsible — surface uncertainty to the user clearly.

- [ ] Add a confidence threshold (e.g. < 0.6) below which the app says "image unclear — try again"
- [ ] Show confidence as a visual indicator (progress bar, color-coded)
- [ ] Store `model_version` alongside each scan (add a migration) so history shows which model made the prediction
- [ ] Consider adding a "not a medical device" disclaimer screen on first launch

---

## Phase 5 — Reinforcement Learning (deferred)

Deferred per `docs/decisions.md` (2026-04-08). RL is a learning goal for later — potentially for image-quality scoring or adaptive decision thresholds. Only revisit after Phase 3 is stable.

---

## Dataset progression at a glance

| Phase | Dataset(s) | Classes | Model output |
|-------|-----------|---------|--------------|
| 1 | HAM10000 | 2 (binary) | suspicious / non_suspicious |
| 2 | HAM10000 | 7 | all HAM10000 canonical labels |
| 3 | HAM10000 + ISIC + others | 9 | all canonical labels |
