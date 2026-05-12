# Database Schema

Authoritative schema is defined by SQL migrations in [../supabase/migrations/](../supabase/migrations/). This file is the human-readable summary.

## profiles
Mirrors `auth.users` with app-specific fields.

| column     | type        | notes                          |
|------------|-------------|--------------------------------|
| id         | uuid (PK)   | references `auth.users.id`     |
| email      | text        |                                |
| created_at | timestamptz | default `now()`                |

## scans
One row per lesion prediction.

| column     | type        | notes                              |
|------------|-------------|------------------------------------|
| id         | uuid (PK)   | default `gen_random_uuid()`        |
| user_id    | uuid (FK)   | references `profiles.id`           |
| image_url  | text        | path in Supabase Storage           |
| prediction | text        | predicted lesion class             |
| confidence | float4      | 0.0 – 1.0                          |
| created_at | timestamptz | default `now()`                    |

### Possible future columns
- `lesion_type` — broader category grouping
- `model_version` — which model produced the prediction
- `notes` — user-supplied free text
- `follow_up_required` — bool for triage

## Constraints & Triggers (added in `0002`)

**`scans_confidence_range`** — `confidence IS NULL OR (confidence >= 0 AND confidence <= 1)`

**`scans_prediction_valid`** — `prediction` must be `NULL` or one of:
- Phase 1 binary: `suspicious`, `non_suspicious`
- Phase 2 HAM10000: `melanoma`, `nevus`, `basal_cell_carcinoma`, `actinic_keratosis`, `benign_keratosis`, `dermatofibroma`, `vascular_lesion`

**`handle_new_user` trigger** — fires `AFTER INSERT ON auth.users`; inserts a matching row into `public.profiles` automatically on sign-up.

## Row-Level Security
- `scans`: users can `select`/`insert` only rows where `user_id = (select auth.uid())`.
- `profiles`: users can `select`/`update` only their own row via `(select auth.uid())`.

> **RLS vs service-role:** RLS policies protect direct client access only. The backend uses the Supabase service-role key, which bypasses RLS. Migration `0004` grants that role `SELECT`/`INSERT` on `public.scans`; the backend still enforces user ownership in application code by filtering on the user ID extracted from the JWT.
