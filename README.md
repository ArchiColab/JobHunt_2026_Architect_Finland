# 🏗️ Finnish Architecture Job Scraper

A Google Colab notebook that scrapes architecture and BIM job listings from Finnish job boards and exports them to Excel.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ArchiColab/JobHunt_2026_Architect_Finland/blob/main/finland_job_scraper_2026.ipynb)


---

## 📋 What it does

Searches for architecture-related jobs across four Finnish job sites using the keywords **arkkitehti**, **projektiarkkitehti**, **BIM**, and **architect**, then exports all results to a structured Excel file with one sheet per source.

| Source | Method | Notes |
|---|---|---|
| [Duunitori](https://duunitori.fi) | Playwright (async) | Blocks plain requests — uses headless browser |
| [Työmarkkinatori](https://tyomarkkinatori.fi) | Playwright (async) | React SPA — needs JS rendering |
| [Jobly](https://www.jobly.fi/en) | `requests` + BeautifulSoup | Server-rendered HTML |
| [SAFA](https://www.safa.fi/koulutus-ja-ura/tyopaikat/) | `requests` + BeautifulSoup | All listings on one page, no keyword filter needed |

> **Note:** Duunitori uses Cloudflare bot detection. Even headless Playwright is blocked — the scraper cell runs but returns 0 results. A working alternative (via Google Custom Search API or Anthropic API) is documented in the notebook comments.

---

## 🚀 Quick start

**1. Open in Colab**

Click the badge at the top of this page, or go to [colab.research.google.com](https://colab.research.google.com) → File → Open notebook → GitHub → paste this repo URL.

**2. Run cells in order**

```
Cell 1 → Mount Google Drive
Cell 2 → Install dependencies (Playwright, BeautifulSoup, pandas — ~2 min)
Cell 3 → Set keywords & shared config
Cell 4 → Scrape Duunitori       ⚠️ blocked by Cloudflare (see notes)
Cell 5 → Scrape Työmarkkinatori
Cell 6 → Scrape Jobly
Cell 7 → Scrape SAFA
Cell 8 → Combine, deduplicate & clean
Cell 9 → Export to Excel + download
```

**3. Download your results**

Cell 9 auto-downloads an `.xlsx` file with:
- **All Jobs** sheet — every listing combined
- One sheet per source (Duunitori, Työmarkkinatori, Jobly, SAFA)

Columns: `source · keyword · title · company · location · posted · deadline · url · scraped_at`

---

## ⚙️ Customise keywords

In **Cell 3**, edit the `Keywords` field:

```python
Keywords = "arkkitehti, projektiarkkitehti, BIM, architect"
```

Separate keywords with commas. The scraper searches each keyword on every site.

---

## 🛠️ Technical notes

### Why Playwright instead of Selenium?

Selenium requires a ChromeDriver binary that must version-match Chrome exactly. In Colab this causes a `WebDriverException: exit code 1` crash because the binary paths change between runtime updates. Playwright bundles its own Chromium — no version matching, no path guessing.

### Why `nest_asyncio`?

Colab's IPython kernel runs a permanent `asyncio` event loop. Playwright's synchronous API (`sync_playwright`) tries to create its own loop and crashes. The fix is `nest_asyncio.apply()` + `async_playwright`, which runs inside the existing loop rather than fighting it.

### Why does Duunitori return 0 results?

Duunitori sits behind Cloudflare bot detection. It checks `navigator.webdriver`, plugin fingerprints, and timing patterns. Even a properly launched headless Chromium is detected and served a JS challenge page with no job cards. Alternatives that work:

- **Anthropic API with web_search tool** — requires an Anthropic API key from [console.anthropic.com](https://console.anthropic.com). NOT DONE!!!
- **Google search** — TEST. NOT DONE!!!

---

## 📦 Dependencies

| Package | Purpose |
|---|---|
| `playwright` | Headless browser for JS-rendered sites |
| `nest_asyncio` | Allows async Playwright to run in Colab's event loop |
| `requests` | HTTP requests for plain HTML sites |
| `beautifulsoup4` | HTML parsing |
| `pandas` | Data cleaning and deduplication |
| `openpyxl` | Excel export |

All installed automatically by Cell 2.

---

## 📁 Output example

| source | keyword | title | company | location | posted | deadline | url |
|---|---|---|---|---|---|---|---|
| Työmarkkinatori | arkkitehti | Arkkitehti | Ingenjörsbyrå Kronqvist Ab | Pietarsaari | | 09.04.2026 | https://tyomarkkinatori.fi/... |
| SAFA | all | Kokenut arkkitehti | Arco Architecture | Helsinki | | 26.05.2026 | https://www.safa.fi/tyopaikat/... |
| Jobly | BIM | BIM-koordinaattori | YIT Suomi Oy | Espoo | 12.5. | | https://www.jobly.fi/... |

---

## ⚠️ Disclaimer

This tool is intended for personal job search use only. Always respect each site's `robots.txt` and terms of service. The scraper includes polite delays between requests (1.5–2 seconds) to avoid overloading servers.

---

## 🤝 Contributing

Pull requests welcome — especially fixes for the Duunitori Cloudflare issue or selector updates if any site changes its HTML structure.

---

*Built for job hunting in the Finnish architecture market. Good luck! 🇫🇮*
