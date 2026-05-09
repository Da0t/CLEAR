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

> Note: `.python-version` pins exactly `3.13.3` for `pyenv`-style tools. Brew installs the latest 3.13.x, which is fine — any 3.13.x works.

### Optional — emulators / simulators
Skip if testing on a physical phone with Expo Go.
- **iOS Simulator (macOS only)**: install Xcode from the App Store; the simulator is bundled.
- **Android Emulator (any OS)**: install Android Studio and create a virtual device.

## Switching between environments

The repo has two Python venvs (`backend/.venv` for FastAPI, `ml/.venv` for PyTorch) plus the mobile Node project. Only one Python venv can be active per terminal.

Both venv directories are named `.venv` (this is the convention most tooling auto-detects), but each is created with `--prompt`, so the shell prompt distinguishes them: `(backend)` vs `(ml)`.

### How to switch venvs
From any directory in the repo:

**Windows (PowerShell):**
```powershell
deactivate; .\backend\.venv\Scripts\Activate   # → (backend)
deactivate; .\ml\.venv\Scripts\Activate        # → (ml)
```

**macOS / Linux:**
```bash
deactivate; source backend/.venv/bin/activate   # → (backend)
deactivate; source ml/.venv/bin/activate        # → (ml)
```

If no venv is currently active, drop the leading `deactivate;` — it's a function defined only when a venv is active, so it errors otherwise. The first activation in a fresh terminal is just the second half of the line.

### Verify which environment is active
When a venv is active, your prompt is prefixed with `(backend)` or `(ml)`. If you ever want to confirm via the actual Python path:

**Windows (PowerShell):**
```powershell
(Get-Command python).Source
$env:VIRTUAL_ENV         # empty if no venv active
```

**macOS / Linux:**
```bash
which python
echo $VIRTUAL_ENV
```

The path tells you which venv (e.g. `...\backend\.venv\...` vs `...\ml\.venv\...`).

### Why a venv may auto-activate when you open a terminal
If a venv activates by itself when you open a terminal in this folder, it's almost certainly **VS Code's Python extension** running the activation script for the workspace's selected interpreter (the one shown in the bottom-bar Python selector). VS Code stores this selection in user-level state, not in the repo, so it follows you across terminals inside that workspace.

To confirm it's VS Code (and not a PowerShell profile or shell hook): open a regular PowerShell window **outside VS Code**, `cd` into the project, and run `$env:VIRTUAL_ENV`. If it's empty there but populated inside VS Code's terminal, the activation is coming from VS Code.

To change which venv VS Code auto-activates: click the Python version in the bottom-right status bar → "Select Interpreter" → pick `backend\.venv\Scripts\python.exe` or `ml\.venv\Scripts\python.exe`.

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
python -m venv .venv --prompt backend
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # then fill in real values
uvicorn app.main:app --reload --host 0.0.0.0
```

**Windows (PowerShell):**
```powershell
cd backend
python -m venv .venv --prompt backend
.\.venv\Scripts\Activate
pip install -r requirements.txt
copy .env.example .env   # then fill in real values
uvicorn app.main:app --reload --host 0.0.0.0
```

> `--host 0.0.0.0` binds uvicorn to all network interfaces so your phone (running Expo Go) can reach the backend over Wi-Fi. Without it, uvicorn only listens on `127.0.0.1` and the phone gets `Network request timed out`. Set `EXPO_PUBLIC_API_URL` in `mobile/.env` to `http://<your-laptop-ip>:8000` (find it with `ipconfig` on Windows or `ifconfig` on macOS).
>
> **Wi-Fi gotcha:** many networks (school, office, public hotspots) isolate clients from each other and block phone-to-laptop traffic, so even with `--host 0.0.0.0` the phone still times out. The simplest workaround is to tether your laptop to your phone's hotspot — both devices end up on the phone's private network and can talk to each other. Your laptop IP changes when you switch networks, so re-run `ipconfig`/`ifconfig` and update `EXPO_PUBLIC_API_URL` after switching.

## ML

**macOS / Linux:**
```bash
cd ml
python -m venv .venv --prompt ml
source .venv/bin/activate
pip install -r requirements.txt
```

**Windows (PowerShell):**
```powershell
cd ml
python -m venv .venv --prompt ml
.\.venv\Scripts\Activate
pip install -r requirements.txt
```

### Already have a venv? Retrofit it without recreating
The `--prompt` flag on `python -m venv` only takes effect at venv creation. If you created your venvs before this convention existed (so your prompt shows `(.venv)` instead of `(backend)` / `(ml)`), don't recreate — just add a single line to `pyvenv.cfg`:

- `backend/.venv/pyvenv.cfg` → add `prompt = backend`
- `ml/.venv/pyvenv.cfg` → add `prompt = ml`

The next time you activate the venv, the new prompt prefix takes effect. No reinstall needed.

Training and inference scripts live under `ml/training/` and `ml/inference/`.
Always run them from the **project root** (not from inside `ml/`) so Python can find the package:

```bash
# from project root:
python -m ml.training.train
```

## HAM10000 Dataset

The image data is gitignored and must be downloaded locally by each developer.

### 1. Create a Kaggle account
Sign up at [kaggle.com](https://www.kaggle.com).

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

## Cross-role setup

The repo expects two developers, each focused on one side. If you usually work on one side and want to step into the other, here's the *additional* setup you'd need.

### Backend / ML dev → also wants to run mobile
You already have Python 3.13 + venvs. To add mobile:
1. Install Node 20+ (Windows: `scoop install nodejs-lts`; macOS: `brew install node`).
2. Install the Expo Go app on your phone.
3. Follow the [Mobile](#mobile) section above (`cd mobile && npm install && ...`).

You don't need a global TypeScript install — `mobile/package.json` brings its own. VS Code's TypeScript support picks it up from the workspace.

### Mobile dev → also wants to run backend or ML
You already have Node + the mobile app running. To add backend / ML:
1. Install Python 3.13 (Windows: `scoop install python`; macOS: `brew install python@3.13`).
2. Follow the [Backend](#backend) and/or [ML](#ml) sections above.

Each side has its own venv and its own `.env`. Activating one venv does not affect the other.
