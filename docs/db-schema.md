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

## Row-Level Security
- `scans`: users can `select`/`insert` only rows where `user_id = auth.uid()`.
- `profiles`: users can `select`/`update` only their own row.
