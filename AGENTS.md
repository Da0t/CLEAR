# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project context
CLEAR — see [PROJECT.md](PROJECT.md) for full vision, scope, and phased plan. This is an **educational + technical project**, not a medical device. Predictions are experimental and must never be framed as medical certainty.

## Repository layout
```
root/
├── PROJECT.md     # vision, scope, phases
├── AGENTS.md      # this file
├── mobile/        # React Native + Expo + TypeScript app
├── backend/       # Python FastAPI server
├── ml/            # PyTorch training & inference
├── docs/          # architecture, setup, build phases, design decisions
└── supabase/      # Supabase config, SQL migrations, seed data
```

## Guiding principles
1. **Clarity over cleverness** — the user is learning each layer. Prefer simple, readable code over abstractions.
2. **Phase discipline** — work within the current phase. Don't add Phase 4 features while Phase 2 is incomplete.
3. **Separation of concerns** — `mobile/`, `backend/`, `ml/` stay cleanly decoupled. The mobile app talks to the backend; the backend calls into `ml/inference`.
4. **No premature optimization** — no caches, queues, or scaling work until there's a real bottleneck.
5. **No medical claims** — UI copy and API responses must avoid implying diagnosis. Always frame as "experimental classification".

## Current phase
Phase 2 (full HAM10000 classifier): Phase 1 binary MVP is verified end-to-end on a real phone. Current work should focus on the 7-class HAM10000 model, Phase 2 label display, and related evaluation notes. Do not add Phase 3 datasets or Phase 4 UX features until the supervised 7-class baseline is working.

## Conventions

### Backend (`backend/`)
- FastAPI + Pydantic
- Routes live in `app/routers/`, business logic in `app/services/`, data shapes in `app/models/`
- Config via `app/config.py` reading from `.env` (template in `.env.example`)

### ML (`ml/`)
- `models/` = PyTorch network architectures (NOT data shapes)
- `training/` = training scripts, kept runnable standalone
- `inference/` = lightweight prediction code imported by the backend
- `data/` = datasets (gitignored content)

### Mobile (`mobile/`)
- Expo + TypeScript
- Screens in `src/screens/`, reusable UI in `src/components/`, API/Supabase clients in `src/lib/`
- **Design system: read [mobile/DESIGN.md](mobile/DESIGN.md) before touching any UI.** Visual reference at `mobile/design-preview.html` (open in browser).

### Supabase (`supabase/`)
- SQL migrations in `supabase/migrations/`
- Seed data in `supabase/seed.sql`
- Local config in `supabase/config.toml` (managed by Supabase CLI)

## What to avoid
- Adding RL code before the supervised baseline works
- Storing real secrets in `.env.example`
- Mixing data models (Pydantic) with ML models (nn.Module) — they live in different `models/` folders for a reason
- Generating long docs unless asked
- Bypassing the backend by calling Supabase directly from the mobile app for sensitive operations

## When in doubt
Check [PROJECT.md](PROJECT.md), then ask the user. Favor small, reviewable changes over large sweeping ones.
