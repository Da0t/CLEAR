# Setup

Local development setup for each component.

## Prerequisites
- Node.js 20+ and npm
- Python 3.13.3 (see `.python-version` at the project root)
- Expo CLI (`npm install -g expo-cli`) — optional, `npx expo` also works
- Supabase CLI:
  - macOS: `brew install supabase/tap/supabase`
  - Windows: `scoop install supabase` or download from [supabase.com/docs/guides/cli](https://supabase.com/docs/guides/cli)

## Backend
**macOS / Linux:**
```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # then fill in real values
uvicorn app.main:app --reload
```

**Windows (PowerShell):**
```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate
pip install -r requirements.txt
copy .env.example .env   # then fill in real values
uvicorn app.main:app --reload
```

## ML

**macOS / Linux:**
```bash
cd ml
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**Windows (PowerShell):**
```powershell
cd ml
python -m venv .venv
.\.venv\Scripts\Activate
pip install -r requirements.txt
```

Training and inference scripts live under `ml/training/` and `ml/inference/`.
Always run them from the **project root** (not from inside `ml/`) so Python can find the package:

```bash
# from project root:
python -m ml.training.train
```

## Mobile
```bash
cd mobile
npm install
cp .env.example .env   # then fill in real values
npx expo start
```
Scan the QR code with the Expo Go app on your phone.

## Supabase (local)
```bash
supabase start         # spins up local Postgres + Studio
supabase db reset      # applies all migrations + seed
```

Run these from the project root (not inside `supabase/`).
