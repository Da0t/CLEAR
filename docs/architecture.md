# Architecture

High-level system design for CLEAR — the Skin Lesion Identification App.

## Components

```
┌─────────────┐      HTTPS       ┌─────────────┐      import      ┌─────────────┐
│   Mobile    │ ───────────────► │   Backend   │ ───────────────► │     ML      │
│ (Expo/RN)   │ ◄─────────────── │  (FastAPI)  │ ◄─────────────── │  (PyTorch)  │
└─────────────┘                  └──────┬──────┘                  └─────────────┘
                                        │
                                        │ auth / db / storage
                                        ▼
                                 ┌─────────────┐
                                 │  Supabase   │
                                 └─────────────┘
```

## Request flow: a single scan

1. User signs in on the mobile app (Supabase Auth).
2. User captures or picks a lesion photo.
3. Mobile uploads the image to the backend with the user's auth token.
4. Backend validates the token, preprocesses the image, and calls `ml/inference/predict.py`.
5. ML returns `{label, confidence}`.
6. If inference succeeds, backend stores the image in Supabase Storage.
7. Backend writes a row to the `scans` table (the `label` value goes into the `prediction` column).
8. Backend returns `{label, confidence, image_url, signed_image_url, scan_id}` to the mobile app.
9. Mobile displays the prediction and confidence, then shows the saved result in history.

> Disclaimer status: Phase 1 copy avoids medical certainty, but the full first-launch disclaimer/onboarding flow is deferred to Phase 4 UX polish.

## Boundaries
- **Mobile never imports ML code.** It only talks to the backend over HTTP.
- **Backend never trains.** Training happens offline in `ml/training/`; the backend only loads saved weights via `ml/inference/`.
- **ML never touches Supabase.** Persistence is the backend's job.

## Data
- `profiles` — one per user (mirrors `auth.users`)
- `scans` — one per prediction, owned by a user
- See [db-schema.md](db-schema.md) for column details.

## Security notes
- Row-Level Security (RLS) on `scans` so users can only read their own rows.
- Backend uses Supabase service role only for trusted server-side operations.
- Images stored in a private Supabase Storage bucket; signed URLs issued on demand.
