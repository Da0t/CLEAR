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

## HAM10000 Dataset

The image data is gitignored and must be downloaded locally by each developer.

### 1. Create a Kaggle account
Sign up at [kaggle.com](https://www.kaggle.com) — a school email gives you free GPU access.

### 2. Get a Kaggle API token
- Go to kaggle.com → avatar (top right) → Settings → **API Tokens** tab
- Click **"Generate New Token"** and copy the token string it shows you

### 3. Save the token (Git Bash)
```bash
mkdir -p ~/.kaggle && echo YOUR_TOKEN > ~/.kaggle/access_token
```
Replace `YOUR_TOKEN` with the actual token. Do not share this token — it gives access to your Kaggle account.

### 4. Install the Kaggle CLI and download
Run from the **project root** with the ML venv active:

**Windows (PowerShell):**
```powershell
ml\.venv\Scripts\Activate
ml\.venv\Scripts\pip install kaggle
ml\.venv\Scripts\kaggle datasets download -d kmader/skin-cancer-mnist-ham10000 -p ml\data\raw\ham10000 --unzip
```

**macOS / Linux:**
```bash
source ml/.venv/bin/activate
pip install kaggle
kaggle datasets download -d kmader/skin-cancer-mnist-ham10000 -p ml/data/raw/ham10000 --unzip
```

This downloads ~6 GB and takes around 10 minutes. When complete, `ml/data/raw/ham10000/` should contain:
```
ham10000/
├── HAM10000_images_part_1/   — 5,000 images
├── HAM10000_images_part_2/   — 5,015 images
└── HAM10000_metadata.csv     — labels for all 10,015 images
```
The other CSV files in the download (`hmnist_*.csv`) are not used and can be ignored.

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
