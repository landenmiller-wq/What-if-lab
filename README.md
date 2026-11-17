# Decision Lab: see the odds before you decide

An AI-assisted financial decision-support app for small businesses. You compare real business decisions (hiring, borrowing, repricing, expanding, cutting costs) side by side. For each one, the app shows the **probability of running out of cash**, the projected financial statements, key ratios and automatic risk flags, plus an AI advisor that explains the results in plain English.

> Informational tool only, not financial advice. Sample companies are fictional.

---

## 1. The problem

Small-business owners make their highest-stakes financial decisions (hire, borrow, expand, change prices) mostly on gut feel:

- Financial statements describe the past, not what a decision will do to the business.
- A single spreadsheet forecast hides the real question: *how likely is it that this goes wrong?*
- Most owners can't read ratios like debt service coverage (DSCR) or the cash conversion cycle, and can't afford a CFO.
- Profitable businesses still fail by running out of **cash**. Growth, inventory and loan payments drain cash before profit shows up.

**Decision Lab turns the owner's own numbers into a side-by-side comparison of options, with the odds of running out of cash front and center, and explains them in plain language.**

## 2. Technology stack

| Layer | Choice | Why |
|---|---|---|
| Calculation engine | Plain JavaScript module (`src/engine.js`), no dependencies | Runs in the browser *and* in Node, so every formula is unit-tested |
| UI | Vanilla HTML/CSS/JS, single page | Opens with no install or build server; easy to demo and share |
| Charts | Chart.js 4 (bundled inline) | Interactive tooltips, works offline |
| AI | Claude: built-in when opened in claude.ai, or the Anthropic Messages API with the user's own key when downloaded | Explains results; never calculates them |
| Testing | `node:test` (17 unit tests) + Playwright/headless Chrome (end-to-end) | Checks the math and the full user flow |
| AI dev tools | Claude (Anthropic) used to design, code, test and debug the app | |

## 3. Features → requirements

| Requirement | How Decision Lab covers it |
|---|---|
| Financial data analysis | **Financial health today** tab: trends, margins, strengths, concerns, anomaly detection (robust z-scores) |
| Forecasting | Driver-based monthly projection of income statement, cash flow and balance sheet over 12–36 months, plus a **Monte Carlo** simulation (500–2,500 futures) |
| AI insights | **AI advisor**: "Explain my options", "Pre-mortem" (how the riskiest option could fail), "Explain today's health", free-form Q&A, and plain-English or analyst mode |
| Ratios and metrics | Gross, operating and net margin, ROA, ROE, current, quick and cash ratios, debt-to-equity, debt-to-assets, interest coverage, DSCR, DSO, DIO, DPO, cash conversion cycle and runway. Each shows its formula and a plain-language meaning |
| Interactive dashboard | Verdict tiles, cash fan chart, cash-out probability curve, comparison table with deltas vs baseline, monthly projection table |
| Risk identification | Rule-based flags (cash-out odds, funding gap, DSCR < 1.25×, leverage, liquidity, margin erosion, losses, slow collections, runway) for today *and* for each option |
| Sample data | Three fictional companies with distinct stories (growing café, struggling retailer, cash-burning startup) |
| Interactive inputs | 11 decision types with editable parameters; editable history, balance sheet and assumptions; CSV import and export; every change recomputes instantly |
| Calculated vs AI clearly separated | Results are labeled **Calculated** or **Rule-based**. AI text sits in a purple **AI-generated** panel and shows the exact data sent to the AI. An **automatic number check** flags any figure in the AI's text that doesn't match a calculated value |
| Disclaimers | Persistent banner, AI panel note, footer, and "How it works" page |

### Decision types
Hire staff · Take a loan · Change prices (with price elasticity) · Buy equipment · Open a location or product line · Marketing campaign · Cut operating costs · Change payment terms or inventory · Supplier cost change · Stress test: lose sales · One-time income or expense

## 4. How the model works (short version)

1. **Learn from history.** Fit a log-linear trend to revenue to get monthly growth, how uncertain that growth estimate is (its standard error), and how much revenue normally swings around the trend. Gross margin is the average of the last 6 months; operating expenses are the average of the last 3. Days to collect, days of inventory and days to pay suppliers (DSO/DIO/DPO) come from today's balance sheet.
2. **Project monthly.** Revenue = trend × volume effects × price effects. COGS scales with units sold, so a price change moves both revenue and margin. Operating expenses grow with cost inflation. Then depreciation, interest from amortizing loans, and tax on positive profit only. Working capital scales with activity. Cash = previous cash + operating cash flow − capex + new borrowing − principal repaid.
3. **Simulate.** Each simulated future draws its own growth rate, persistent month-to-month revenue swings (correlation 0.6, centered on the trend) and margin noise, all sized from the company's own history. **Every option uses the same random futures**, so differences come from the decision, not luck.
4. **Flag risks** with transparent thresholds (see the app's "How it works" tab).

## 5. Running it

### Option A: just open the file (no install)
Download `decision-lab.html` and double-click it. It opens in any modern browser (Chrome, Edge, Safari, Firefox), works offline, and needs no server. Your edits are saved in that browser.

**Turning on the AI advisor outside Claude:** go to **AI advisor → Turn on AI in this copy**, paste an Anthropic API key (from console.anthropic.com), and click **Save key**. The key is stored only in your browser and sent only to `api.anthropic.com`; usage is billed to your Anthropic account. Without a key, the advisor shows clearly labeled rule-based summaries instead.

### Option B: put it on the web with a shareable link (free)
- **Netlify Drop:** go to app.netlify.com/drop, then drag a folder containing the file renamed to `index.html`. You get a public link in seconds.
- **GitHub Pages:** create a repository, upload the file as `index.html`, then go to Settings → Pages → Deploy from branch (main). The site appears at `https://<username>.github.io/<repo>/`.

Each visitor enters their **own** API key if they want AI. Never put your key into the file you publish.

### Option C: develop it further
```bash
npm install          # installs Chart.js
node tools/build.js  # regenerates sample data and builds dist/index.html
```
Edit files in `src/`, rebuild, and refresh the browser.

**Tests:**
```bash
node --test test/engine.test.js     # 17 unit tests for the financial math
python3 tools/browser_test.py       # end-to-end UI test (needs Playwright + Chrome)
python3 tools/ai_test.py            # AI flow with a simulated AI, including the number check
python3 tools/api_test.py           # your-own-key AI mode against a simulated Anthropic API
```

## 6. Five-minute demo script

1. **Set the scene (30 s).** "Owners make big decisions on gut feel. Profitable businesses still die from running out of cash."
2. **Brightside Coffee (1.5 min).** The decision is *second café or wholesale push?*
   - Point to the verdict tiles: current course 0%, café **≈8%** chance of running out of cash, wholesale 0%.
   - Cash outlook chart: the café dips to about $13k in mid-2027. Click **Option A** to show the uncertainty band.
   - Open the loan card and raise the loan from $180k to $300k. The risk drops to 0%. *"A bigger loan buys safety, at the cost of more debt. Look at debt-to-equity in the table."*
3. **Northwind Outdoor (1 min).** Switch company. Doing nothing → **100%** cash-out by month 9. The discount option makes it *worse* (margin falls about 7 points and cash runs out in month 2). Cost reset plus leaner inventory → about **4%**. *"The intuitive fix, discounting, is the dangerous one."*
4. **AI advisor (1 min).** Click **Explain my options**, then **Pre-mortem**. Point out the purple AI label, "See exactly what data was sent", and the **number check** result.
5. **Trust and transparency (1 min).** Show the **Financial health today** ratios with formulas, then **Your data**: edit a revenue figure live and watch everything update. Mention CSV import, the 17 unit tests and the disclaimers.

## 7. Project structure
```
src/engine.js              calculation engine: ratios, projection, Monte Carlo, risk rules
src/app.js                 user interface, charts, data editing, AI advisor, number check
src/styles.css             design system (light and dark)
src/index.template.html    page shell
src/samples.js             generated sample companies
tools/gen-samples.js       sample-data generator (seeded and reproducible)
tools/build.js             bundles everything into dist/index.html
test/engine.test.js        unit tests
tools/browser_test.py      end-to-end test
tools/ai_test.py           AI-path test with a simulated AI
dist/index.html            the finished single-file app
```

## 8. Limitations and next steps
- Trend extrapolation can't foresee seasonality beyond what's in the history, or shocks larger than the uncertainty setting.
- Taxes, depreciation and loan terms are simplified.
- Ideas: connect to accounting software (QuickBooks/Xero), add seasonality, add a "lender's view" loan-readiness mode, and offer a PDF board or lender report.
