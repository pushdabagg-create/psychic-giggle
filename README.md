# psychic-giggle
Below is the one‑shot shell script that will write the fully expanded project files locally, initialize a git repo, create a PRIVATE GitHub repo pushdabagg-create/lunar-lab via gh, and push the initial commit. Save it as create_and_push_lunar_lab_private.sh, inspect/edit if desired, make it executable, and run it locally. gh must be authenticated on your machine.

Important safety notes before running:
- Do NOT put any API keys or secrets into the files this script writes; use .env and keep it out of version control.
- Inspect files (especially README_FOR_NON_PROGRAMMERS.md, LICENSE, and any scripts) before pushing.
- This will create and push a private repository under pushdabagg-create named lunar-lab.

create_and_push_lunar_lab_private.sh
#!/usr/bin/env bash
set -euo pipefail

OWNER="pushdabagg-create"
REPO="lunar-lab"
VISIBILITY="private"
BRANCH="main"
WORKDIR="${REPO}"

echo "This script will create a local folder '${WORKDIR}', populate it with project files, commit, and create a PRIVATE GitHub repo ${OWNER}/${REPO} (via gh) and push the initial commit."
echo "If you want to review files before pushing, abort now (Ctrl-C)."

rm -rf "${WORKDIR}"
mkdir -p "${WORKDIR}"
cd "${WORKDIR}"

# README.md
cat > README.md <<'EOF'
Short description
This repository contains a pipeline that combines lunar and solar cycle data with historical stock and cryptocurrency prices, builds features, trains models, runs threshold sweeps, and produces ranked candidate dates and assets. It includes a paper-trade scaffold (Alpaca), CCXT crypto scaffolding, a Streamlit proof-of-concept dashboard (phase 1), and a non-technical packet.

Quickstart (recommended — Docker)
1. Copy .env.example -> .env and add Alpaca / exchange keys only if you will paper-trade (optional).
2. Build Docker:
   docker build -t lunar-lab:latest .
3. Run one-off analysis (this may take time; adapt tickers/time range):
   docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python data_fetch.py --start 1995-11-10 --end 2025-11-10
   docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python blend_lunar_solar_model.py --hold 3 --tickers SP500
   docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python threshold_sweep.py --dataset outputs/dataset.parquet --hold_days 3
4. Inspect outputs/ for CSVs and plots.

See README_FOR_NON_PROGRAMMERS.md for a full non-technical guide.
EOF

# README_FOR_NON_PROGRAMMERS.md (full non-technical packet)
cat > README_FOR_NON_PROGRAMMERS.md <<'EOF'
Non‑Technical Step‑by‑Step Packet

1) Short summary (one paragraph)
- This program studies the Moon and the Sun (their phases, distances, and solar activity) together with historical stock and cryptocurrency prices to find times when certain assets historically moved in a predictable way after specific lunar/solar events. It produces ranked lists of dates and assets that, historically, had the best risk/reward for short multi‑day holds, provides a paper‑trade scaffold to test trades safely, and includes a simple demo so non‑technical users can verify behavior. It is evidence‑driven, not a guarantee of future profit.

2) Plain‑English description of what it does
- It gathers three kinds of information:
  A) lunar data (new/full moons, illumination fraction, distance),
  B) solar activity (sunspots, radio flux F10.7, recorded flares, geomagnetic indices if available),
  C) asset price history (stocks and cryptocurrencies).
- It finds days where Moon/Sun conditions historically correlate with positive (or negative) asset moves, tests whether those correlations are statistically reliable, trains machine‑learning models to learn which combinations of features are predictive, and produces a ranked list of the most promising asset + date combinations.
- It includes a paper‑trading routine to place simulated trades for testing before you trade real money.

3) Safety and important warnings (read first)
- Nothing is guaranteed. Historical patterns can and do fail. Never treat any output as a guarantee.
- Start with extensive paper trading for several weeks or months and only consider real money after consistent, repeatable results.
- Use small position sizes initially, strict stop‑loss rules, and never trade money you can’t afford to lose.
- The system can generate many signals. Use recommended exposure limits (e.g., 1–5% of account equity per trade) and limit concurrent positions.

4) What you need (plain list)
- Internet connection.
- A computer (Windows, Mac, or Linux).
- Option A (recommended): Docker Desktop (no need to install Python modules locally).
- Option B (alternative): Python 3.10+ and ability to run commands in a terminal.
- Optional for live paper trading: Alpaca paper trading account (free) for US equities; for crypto you may use an exchange supported by CCXT (Binance, Coinbase, Kraken, etc.), but only provide API keys to your environment — never paste them in public places.

5) Quickstart — easiest way (Docker)
- Create a file named .env in the project folder with your Alpaca paper keys and optional CCXT exchange keys; otherwise skip keys for dry-run.
  Example .env contents:
  ALPACA_API_KEY=your_key_here
  ALPACA_SECRET_KEY=your_secret_here
  ALPACA_BASE_URL=https://paper-api.alpaca.markets
  TICKERS=SPY,QQQ,BTC-USD,ETH-USD
  MODE=dry_run
  DRY_RUN=true
- Build the Docker image:
  docker build -t lunar-lab:latest .
- Run the analysis once (this downloads data; may take time for many tickers):
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python data_fetch.py --start 1995-11-10 --end 2025-11-10
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python blend_lunar_solar_model.py --hold 3 --tickers SP500
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python threshold_sweep.py --dataset outputs/dataset.parquet --hold_days 3
- Look inside the outputs/ folder after it finishes.

6) Quickstart — alternative (Python on your computer)
- Install Python 3.10+
- In the project folder:
  pip install -r requirements.txt
- Create .env (optional).
- Run:
  python data_fetch.py --start 1995-11-10 --end 2025-11-10
  python blend_lunar_solar_model.py --hold 3 --tickers SP500
  python threshold_sweep.py --dataset outputs/dataset.parquet --hold_days 3
- Wait for files to be written to outputs/ (may take minutes–hours depending on machine & network).

7) What files the program creates and how to read them (non‑technical)
- outputs/moon_daily.csv — daily lunar features (date, illumination, phase, distance).
- outputs/price_matrix.parquet — cached adjusted close prices for all tickers processed.
- outputs/dataset.parquet — merged feature dataset with labels for model training.
- outputs/event_returns.csv — for each event and asset, returns at days -5..+5 and cumulative windows.
- outputs/stock_summary.csv — per asset and time-window summaries: n_events, average return, win rate, p-values.
- outputs/ranked_top_cum3.csv — most promising assets for a typical 3‑day hold (ranked by historical effect and corrected p‑values).
- outputs/backtest_top.csv — simple backtest summary showing how a simple rule would have performed historically.
- outputs/threshold_results.csv — shows how different model probability cutoffs affect trades and P&L.
- outputs/plots/*.png — visualization charts you can open in any image viewer.
- model_artifacts/ — saved model and scaler used for live inference.

8) How to interpret the ranked list (practical)
- Open outputs/ranked_top_cum3.csv in Excel or a text editor.
- Important columns:
  - symbol — asset ticker (e.g., AAPL or BTC-USD).
  - date — event date the list refers to.
  - expected_return or mean_ret — average historical return for the event window.
  - n_events — how many historical events contributed (more is better).
  - p_adj — multiple-test corrected p-value (lower is better; values < 0.01 suggest strong historical evidence).
  - model_prob — model’s probability for a positive outcome (higher is more confident).
- Practical rule of thumb: focus on rows with n_events >= 8–10, p_adj <= 0.01, and model_prob >= 0.90 (conservative) or >= 0.95 (very conservative). The repo may include a filtered CSV for 99% filters.

9) Example — how a non‑programmer can act on a signal (manual steps)
- Suppose ranked_top_cum3.csv has a row:
  date: 2025-12-02, symbol: XYZ, model_prob: 0.95, suggested_hold: 3 days
- Manual steps:
  1. Open your Alpaca paper account (or crypto exchange demo account).
  2. Search for ticker XYZ in the broker UI.
  3. Create a BUY order for the dollar amount you want (recommended: 1–5% of your account equity).
  4. Set a stop‑loss about 2% below entry and a take‑profit about 4% above entry (defaults used by scripts).
  5. Note the entry date and plan to close after the suggested hold (or use a bracket order so TP/SL closes automatically).
  6. Record the trade in a spreadsheet (entry, exit, P&L). Track results for at least 30–90 trades before changing rules.

10) How to use the paper‑trading scaffold (non‑technical)
- Option A (fully automatic simulation): run lunar_vol_trader.py or lunar_multi_alpaca_trader.py in DRY_RUN to simulate orders (no real trades).
  Example:
  python lunar_vol_trader.py --dry-run
- Option B (semi-automatic/manual): generate an “action list” CSV with recommended entries and amounts; a helper or you enter orders in the broker’s web UI.
- Option C (auto paper-trade): supply Alpaca paper API keys in .env and run the trader with --place flag (only after extended dry-run).

11) Money & position sizing explained plainly
- Percent per trade: default 5% of account equity. You can change this in .env or config.
- Reinvest rule: by default, 25% of realized profits are automatically allocated to increase the reinvestment pool; the rest is considered withdrawn profit.
- Simple rule for beginners: use very small amounts in paper demo. If going live, start very small and increase only after long, consistent positive paper-trading results.

12) Daily routine for a non‑programmer (simple)
- Step 1 (once daily): Run the analysis or ask your helper to run it; check outputs/ for new results.
- Step 2: Open ranked_top_cum3.csv and outputs/threshold_results.csv.
- Step 3: Choose up to 1–2 top signals that match your risk tolerance.
- Step 4: Place paper orders manually (or use the scaffold if validated).
- Step 5: Record results and repeat daily. After 30–90 trades evaluate performance.

13) How “99%” filters work (plain language)
- Statistical confidence: a 99% flag means the pattern was statistically strong in history after correcting for multiple testing. It’s not a guarantee — think “very strong historical evidence.”
- Model confidence: 99% model probability means the trained model assigns a 99% chance of a positive outcome (based on historical patterns). This is not a guarantee.
- The repo provides separate lists for model-only, stat-only, and combined filters.

14) Troubleshooting (very simple)
- If the program does not run: check Docker is installed or that Python and dependencies from requirements.txt are installed. Look at the terminal errors (last 20 lines are often diagnostic).
- If outputs/ is empty: the job may still be running or a download failed; re-run and watch logs.
- If paper trades do not appear: confirm ALPACA_API_KEY and ALPACA_SECRET_KEY are in .env and ALPACA_BASE_URL points to the paper endpoint.
- If too many signals: raise the model_prob threshold (e.g., only act on model_prob >= 0.95) or lower max concurrent positions.

15) FAQ (short)
- Q: Will this guarantee profit? A: No. It produces statistically supported suggestions and simulated backtests; you must validate and trade carefully.
- Q: How long before I can trust results? A: Paper trade for months and perform walk‑forward tests. Reliable edges typically require extensive validation.
- Q: Can I change risk settings? A: Yes — edit .env or the user-friendly config. Ask a helper if needed.

16) Glossary (very simple)
- Backtest: running the strategy on historical data to see how it would have performed.
- Paper trading: simulated trading using a demo account to test strategy.
- Bracket order: a buy order with attached stop-loss and take-profit so the trade automatically closes.
- P&L: profit and loss.
- p-value / p_adj: statistical measure that tells us how likely a result is due to chance. Lower is better.
- model_prob: the probability the model assigns to a positive outcome.

17) Checklist for a safe first run (copy this as a checklist)
- [ ] Install Docker (recommended) or Python + dependencies.
- [ ] Create .env and add Alpaca paper keys if you plan to paper trade (do NOT commit it).
- [ ] Run docker build and run the analysis once.
- [ ] Open outputs/ and view ranked_top_cum3.csv and the plots.
- [ ] Choose 1–2 signals and paper trade manually with small amounts.
- [ ] Record all trades in a spreadsheet for 30–90 trades before making live changes.
- [ ] If results are consistently good in paper trading, consider gradual live testing.

18) How to get help
- If you need someone to run commands: ask a technically capable friend or hire a freelancer to run Docker/Python steps.
- If you want me to produce a “one‑click” script or hosted run (I cannot run it for you), I can produce more scripts, Docker images, or deployment instructions.

19) One‑page quick reference (printable)
- Create .env (keys) → Build Docker → Run analysis → Open outputs/ranked_top_cum3.csv → Pick up to 2 signals/day → Paper trade with 1–5% equity and TP/SL → Record results → Repeat.

20) Example simple daily script for a helper to run (copy this to send someone technical)
- Build & run once (in project folder):
  docker build -t lunar-lab:latest .
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python data_fetch.py --start 1995-11-10 --end 2025-11-10
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python blend_lunar_solar_model.py --hold 3 --tickers SP500
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python threshold_sweep.py --dataset outputs/dataset.parquet --hold_days 3
- Run trader daemon in dry-run (paper simulation):
  docker run --env-file .env --rm -v "$(pwd)/outputs":/app/outputs lunar-lab:latest python lunar_vol_trader.py --dry-run

21) Final plain advice
- Treat this tool like a scientific instrument: use it to gather evidence, not as a “get rich quick” button. Verify with robust paper trading, use strict risk controls, and always use stop losses.
EOF

# .env.example
cat > .env.example <<'EOF'
# If you want to enable Alpaca paper trading, fill these out. Otherwise you can ignore.
ALPACA_API_KEY=
ALPACA_SECRET_KEY=
ALPACA_BASE_URL=https://paper-api.alpaca.markets

# CCXT exchange keys (example variables; use your exchange's key names)
CCXT_EXCHANGE=
CCXT_API_KEY=
CCXT_SECRET=

# General settings
MODE=dry_run
TICKERS=SPY,QQQ,BTC-USD,ETH-USD
PERCENT_PER_TRADE=0.05
MAX_CONCURRENT=5
REINVEST_PERCENT=0.25
MODEL_PROB_THRESHOLD=0.95
EOF

# requirements.txt
cat > requirements.txt <<'EOF'
pandas==2.2.2
numpy==1.26.5
scikit-learn==1.3.2
xgboost==1.7.6
yfinance==0.2.25
requests==2.34.0
tqdm==4.66.2
matplotlib==3.8.1
pyyaml==6.0.1
skyfield==2.4
python-dateutil==2.8.2
python-dotenv==1.0.0
alpaca-trade-api==3.0.0
ccxt==3.0.0
streamlit==1.30.0
EOF

# .gitignore
cat > .gitignore <<'EOF'
outputs/
.env
*.pyc
__pycache__/
model_artifacts/
data/
*.ipynb_checkpoints
EOF

# Dockerfile
cat > Dockerfile <<'EOF'
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PYTHONUNBUFFERED=1
CMD ["bash"]
EOF

# LICENSE (MIT)
cat > LICENSE <<'EOF'
MIT License

Copyright (c) 2025 pushdabagg-create

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF

# data_fetch.py (full expanded)
cat > data_fetch.py <<'EOF'
#!/usr/bin/env python3
"""
data_fetch.py
Fetch S&P-500 tickers or provided tickers, download price data (stocks & crypto),
compute daily lunar ephemeris features (illumination, phase angle, distance),
and save outputs required by downstream pipeline.
"""
import os
import argparse
from datetime import datetime, timedelta
import math
import warnings

import pandas as pd
import numpy as np
import requests
import yfinance as yf
from skyfield.api import load

OUT = "outputs"
DATA = "data"
os.makedirs(OUT, exist_ok=True)
os.makedirs(DATA, exist_ok=True)

WIKI_SP500 = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"

def fetch_sp500_tickers():
    try:
        r = requests.get(WIKI_SP500, timeout=30)
        r.raise_for_status()
        df = pd.read_html(r.text)[0]
        tickers = df['Symbol'].tolist()
        tickers = [t.replace('.', '-') for t in tickers]
        return tickers
    except Exception as e:
        raise RuntimeError(f"Unable to fetch S&P500 tickers: {e}")

def download_prices(tickers, start, end):
    start_iso = pd.to_datetime(start).strftime("%Y-%m-%d")
    end_iso = (pd.to_datetime(end) + timedelta(days=1)).strftime("%Y-%m-%d")
    print(f"Downloading {len(tickers)} tickers from {start_iso} to {end_iso} ...")
    df = yf.download(tickers, start=start_iso, end=end_iso, group_by='ticker', progress=False, threads=True)
    if isinstance(df.columns, pd.MultiIndex):
        out = {}
        for t in tickers:
            try:
                out[t] = df[(t, 'Adj Close')]
            except Exception:
                continue
        mat = pd.DataFrame(out)
    else:
        mat = df['Adj Close'] if 'Adj Close' in df.columns else df
    mat = mat.sort_index()
    mat.to_parquet(os.path.join(OUT, "price_matrix.parquet"))
    print("Saved price_matrix.parquet")
    return mat

def compute_moon_daily(start, end):
    print("Computing lunar ephemeris daily features...")
    ts = load.timescale()
    eph = load('de421.bsp')
    earth, moon, sun = eph['earth'], eph['moon'], eph['sun']
    start_dt = pd.to_datetime(start)
    end_dt = pd.to_datetime(end)
    dates = pd.date_range(start_dt, end_dt, freq='D')
    rows = []
    for dt in dates:
        t = ts.utc(dt.year, dt.month, dt.day, 0, 0, 0)
        astromoon = earth.at(t).observe(moon).apparent()
        astrosun = earth.at(t).observe(sun).apparent()
        elong = astromoon.separation_from(astrosun).degrees
        illum = (1 + math.cos(math.radians(elong))) / 2.0
        distance_km = float(earth.at(t).observe(moon).distance().km)
        if elong < 20:
            phase = "New Moon"
        elif elong <= 90:
            phase = "Waxing"
        elif elong <= 160:
            phase = "Waxing Gibbous"
        elif elong <= 200:
            phase = "Full Moon"
        elif elong <= 260:
            phase = "Waning Gibbous"
        elif elong <= 340:
            phase = "Waning"
        else:
            phase = "New Moon"
        rows.append({"date": dt.date(), "illumination": float(illum), "phase_angle_deg": float(elong), "phase":