# CanLII PDF Downloader

Bulk-download CanLII decision PDFs (and per-year JSON metadata) for any jurisdiction.

Files are saved under:

```
data/<state>/<db>/<year>/<decision>.pdf
data/<state>/<db>/<year>.json
```

Example: `data/on/onlrb/2026/2026canlii63962.pdf`

---

## 1. Setup (after clone)

```bash
git clone https://github.com/zainiqbal-ml1/scrapper.git
cd scrapper

python3 -m venv .venv
source .venv/bin/activate          # macOS / Linux
# .venv\Scripts\activate             # Windows

pip install --upgrade pip
pip install -r requirements.txt
```

**You also need:**
- Google Chrome (for cookie / captcha when not using Tor import)
- Optional: [Tor Browser](https://www.torproject.org/) for `--tor` mode or the browser extension

On first run, `session.py` is created automatically from `session.py.template`.

---

## 2. Run the Python scraper

### Interactive (easiest)

```bash
python run.py
```

Follow the prompts: jurisdiction → database(s) → years → rate → Tor yes/no.

### One-liner examples

```bash
# Ontario, one database, one year
python run.py --juris on --db onlrb --years 2026 --rate 0.3-0.5

# Through Tor (recommended for bulk)
python run.py --tor --juris on --db onlrb --years 2026 --rate 0.3-0.5

# Multiple databases / year range
python run.py --juris on --db onca onsc --years 2020-2024 --rate 0.2
```

**Rate tip:** use a low rate with Tor (`0.1–0.5` req/s). Higher rates get blocked faster.

The scraper skips PDFs already on disk and resumes after a block.

---

## 3. Tor workflow (recommended)

Best results: solve captcha once in **Tor Browser**, import the cookie, then scrape on the **same exit** (do not rotate IP).

### Step A — solve captcha in Tor Browser

1. Open Tor Browser → go to `https://www.canlii.org/`
2. Solve the captcha
3. Open a listing page and confirm PDFs open
4. In Tor Browser: **Tools → Web Developer → Web Console**, run:
   ```js
   copy(navigator.userAgent)
   copy(document.cookie)
   ```

### Step B — import cookie into the scraper

```bash
python import_tor_cookie.py
```

Paste the User-Agent and cookie when asked.

### Step C — run (no `--new-ip`)

```bash
python run.py --tor --juris on --db onlrb --years 2026 --rate 0.3-0.5
```

**Important:** do **not** use `--new-ip` after importing a Tor Browser cookie, or the cookie will not match the exit IP.

---

## 4. Force a fresh cookie

If the session is blocked or stale:

```bash
# Clear cached cookie state
rm -f .cookie_state.json

# Option 1 — re-import from Tor Browser (best with --tor)
python import_tor_cookie.py

# Option 2 — harvest via Chrome (no Tor)
python auto_refresh.py
```

Then run `python run.py` again.

### Stuck on "exit unchanged"?

That means Tor could not rotate the exit IP automatically (control port not available). Either:

- Use **Tor Browser → New Identity** manually, then `python import_tor_cookie.py`, **or**
- Run **without** `--new-ip` after importing a cookie from Tor Browser

---

## 5. Tor Browser extension (no Python)

For downloading directly inside Tor Browser:

```bash
./install_tor_extension.sh
```

Then **quit and reopen Tor Browser** completely.

1. Open a CanLII listing page, e.g. `https://www.canlii.org/on/onlrb/nav/date/2026/`
2. Solve captcha if shown
3. Click the extension icon → **Start**
4. PDFs save to `Downloads/canlii/<db>/<year>/`

When blocked: **Tor menu → New Identity**, solve captcha again — the extension reopens the page and resumes.

---

## 6. Other useful commands

```bash
python canlii_scraper.py --list-jurisdictions
python canlii_scraper.py --juris on --list-dbs
python canlii_scraper.py --check
python set_session.py          # paste a Copy-as-cURL export
```

Optional API key (metadata only) — copy `.env.example` to `.env` and set `CANLII_API_KEY`.

---

## 7. macOS permissions (optional, for auto-captcha)

If you want the scraper to auto-solve the slider in Chrome:

- **System Settings → Privacy & Security → Screen Recording** — enable Terminal (or Cursor)
- **System Settings → Privacy & Security → Accessibility** — enable Terminal (or Cursor)
- **Chrome → View → Developer → Allow JavaScript from Apple Events**

Without these, a Chrome window opens and you solve the captcha manually.

---

## Local files (not in git)

| File | Purpose |
|------|---------|
| `session.py` | Live cookie + user-agent |
| `data/` | Downloaded PDFs and JSON |
| `.env` | Optional API key |
| `.cookie_state.json` | Cookie / exit cache |
