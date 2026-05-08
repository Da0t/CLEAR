# Setup

Local development setup for each component.

## Prerequisites

### All platforms
- **Python 3.13.3** (see `.python-version` at the project root)
- **Node.js 20+** and npm
- **Supabase CLI** (cloud project access)
- **Expo Go app** on your phone — install from the iOS App Store or Google Play. Required to run the mobile app on a real device.

### Windows install (via Scoop — recommended)
```powershell
scoop install python
scoop install nodejs-lts
scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
scoop install supabase
```

### macOS install (via Homebrew)
```bash
brew install python@3.13 node
brew install supabase/tap/supabase
```

After `brew install python@3.13`, use `python3` rather than `python` in commands — Homebrew does not create a bare `python` symlink by default.

### Optional — emulators / simulators
Skip if testing on a physical phone with Expo Go.
- **iOS Simulator (macOS only)**: install Xcode from the App Store; the simulator is bundled.
- **Android Emulator (any OS)**: install Android Studio and create a virtual device.

## Switching between environments

The repo has two Python venvs (`backend/.venv` for FastAPI, `ml/.venv` for PyTorch) plus the mobile Node project. Only one Python venv can be active per terminal.

### Verify which environment is active
When a Python venv is active, your prompt is prefixed with `(.venv)`. To check *which* one:

**Windows (PowerShell):**
```powershell
(Get-Command python).Source
```

**macOS / Linux:**
```bash
which python
```

The path tells you which venv (e.g. `...\backend\.venv\...` vs `...\ml\.venv\...`).

### Deactivate
```
deactivate
```

### Common pitfalls
1. **`cd backend` does not activate the venv.** Always run the activation command after `cd`.
2. **Run ML scripts from the project root**, not from inside `ml/`. Use `python -m ml.training.train` so package imports resolve.
3. **One venv per terminal.** Open separate terminal tabs for parallel backend / ML work.
4. **VS Code interpreter.** The bottom-bar Python selector picks one interpreter for the workspace. Switch it to match the file you're editing.

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

**macOS / Linux:**
```bash
cd mobile
npm install
cp .env.example .env   # then fill in real values
npx expo start
```

**Windows (PowerShell):**
```powershell
cd mobile
npm install
copy .env.example .env   # then fill in real values
npx expo start
```

Scan the QR code with the Expo Go app on your phone. Both your phone and your computer must be on the same Wi-Fi network.

## Supabase (local)
```bash
supabase start         # spins up local Postgres + Studio
supabase db reset      # applies all migrations + seed
```

Run these from the project root (not inside `supabase/`).
