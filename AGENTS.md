# AGENTS.md — Busan Travel Guide Project

Interactive travel guide web application and offline PDF handbook for a 5-day Busan trip, deployed via GitHub Pages at `https://roychihcheng.github.io/busan-travel-guide/`.

## Architecture & Layout

* `index.html`: Single-page responsive guide styled with Tailwind CSS (CDN) and custom CSS classes (`.kr-street-sign`, `.glass-nav`, `.img-box`).
* `images/`: Authentically sourced, locally bundled JPEG assets (max 1200x800, quality 85). Never hotlink external image APIs (e.g. Wikimedia) in HTML to prevent HTTP 429 blocks.
* `busan-travel-guide-2026.pdf` & `釜山極致自由行詳細攻略與實景路引指南.pdf`: Offline PDF handbooks linked directly in the header and footer.
* `validate_tags.py`: DOM tag balance validator extending `html.parser.HTMLParser`.
* `make_pdf.py`: Headless Chromium/Edge script to render `index.html` to PDF.

## Development & Verification Commands

All commands run from repo root (`C:\Users\RoyWang\Desktop\釜山`):

### 1. Validate HTML DOM Tag Balance
Always run after modifying `index.html` to guarantee 0 unclosed or mismatched tags:
```bash
python -c "import sys; from validate_tags import TagValidator; v = TagValidator(); v.feed(open('index.html', encoding='utf-8').read()); sys.exit(0 if v.validate() else 1)"
```

### 2. Regenerate Offline PDF Handbooks
Whenever `index.html` is updated, sync the PDFs:
```bash
python -c "import subprocess, shutil; cmd = ['C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe', '--headless=new', '--disable-gpu', '--no-sandbox', '--virtual-time-budget=10000', '--print-to-pdf=temp.pdf', '--no-pdf-header-footer', 'index.html']; subprocess.run(cmd, timeout=60); shutil.copyfile('temp.pdf', 'busan-travel-guide-2026.pdf'); shutil.copyfile('temp.pdf', '釜山極致自由行詳細攻略與實景路引指南.pdf'); import os; os.remove('temp.pdf')"
```

### 3. Deploy to GitHub Pages
GitHub Pages builds from `main` branch root `/`:
```bash
git add index.html busan-travel-guide-2026.pdf "釜山極致自由行詳細攻略與實景路引指南.pdf" README.md images/ AGENTS.md
git commit -m "update: sync itinerary content and offline PDF"
git push origin main
```

## Content & Wayfinding Conventions

Every attraction and dining entry MUST follow the **5-Point Field Wayfinding Anatomy**:
1. 🚇 **Line & Code**: Busan Metro line badge (🔴 Line 1 `#F05A28`, 🟢 Line 2 `#3CB44A`, 🔵 Donghae `#0054A6`, 🟣 BGL `#782F8E`) + station code.
2. 🚪 **Exit & Directory**: Station exit number + exact text on the station directory sign.
3. 🪧 **Physical Street Sign**: Road plate format in `.kr-street-sign` (e.g. `[ GUNAM-RO 41BEON-GIL ]`).
4. 🏬 **Exact Local Storefront**: Korean name (`한글`) + English translation + stall/unit number.
5. 🧭 **Visual Turning Cue**: Physical landmark visible upon exiting or arrival.

Every destination/restaurant MUST include paired navigation buttons:
* **Google Maps**: `https://www.google.com/maps/search/?api=1&query={encoded_query}`
* **Naver Map**: `https://map.naver.com/p/search/{encoded_korean_query}` (Korean ground navigation standard)

Every dining entry MUST feature dual-plan dining: **【首選名店】** paired with 1–2 **【超強備案】** in the same district.

## Pitfalls & Gotchas

* **Headless Browser PDF Timeout**: Headless Chrome hangs on Windows if user profile locks conflict or without rendering allowance. Always supply `--no-sandbox`, `--disable-gpu`, and `--virtual-time-budget=10000`.
* **Windows PDF File Lock (Error 0x5)**: If the target PDF is open in Adobe Acrobat or Edge, writes fail with `Access Denied`. Always write to a temporary file (`temp.pdf`) first before replacing.
* **Synchronous PDF Updates**: Modifying `index.html` without re-rendering and committing the PDF creates version drift between the live website and downloaded handbook. Always update both together.
