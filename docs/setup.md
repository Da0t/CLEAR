# Setup

Local development setup for each component.

## Prerequisites
- Node.js 20+ and npm
- Python 3.11+
- Expo CLI (`npm install -g expo-cli`) — optional, `npx expo` also works
- Supabase CLI (`brew install supabase/tap/supabase`)

## Backend
```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # then fill in real values
uvicorn app.main:app --reload
```

## ML
```bash
cd ml
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Training and inference scripts live under `ml/training/` and `ml/inference/`.

## Mobile
```bash
cd mobile
npm install
npx expo start
```
Scan the QR code with the Expo Go app on your phone.

## Supabase (local)
```bash
cd supabase
supabase start         # spins up local Postgres + Studio
supabase db reset      # applies migrations + seed
```
