# Warehouse Operations Benchmarking & Compliance Tool

> An open-source self-assessment diagnostic for warehouse operators, supply chain leaders, and auditors. Score your operation across 15 sub-areas and 106 controls aligned to the DTAC Warehouse SOP framework, then receive an instant compliance tier, sub-area heatmap, KPI gap analysis, and prioritised remediation plan.

**Designed by [FreightFox.ai](https://freightfox.ai) — freight intelligence platform for Indian manufacturers.**

---

## What it does

A four-step browser-based wizard that:

1. **Captures the assessor and warehouse details** — organisation, sector, location, footprint, SKU count, contact information.
2. **Runs a 106-control self-assessment** scored on a 0–3 maturity scale, organised under three pillars (Inbound Operations, Order Fulfilment, Warehouse Maintenance).
3. **Collects 10 operational KPIs** — picking accuracy, stock accuracy, customs clearance time, cold chain integrity, expired stock, and others — each with SOP-derived target thresholds.
4. **Generates a benchmarking report** with overall compliance %, maturity tier (Best-in-Class → Critical Gap), pillar scores, sub-area heatmap, top-5 strengths and gaps, KPI actual-vs-target table, and an auto-prioritised remediation plan.

The report can be downloaded as JSON or printed/saved as PDF. All assessments are silently captured to a Google Sheet for benchmarking and follow-up.

## Live demo

Embedded on Google Sites at: *(add your URL once published)*

Direct GitHub Pages URL: *(add your URL once published)*

## Framework

The control framework is derived from the **Warehouse Standard Operating Procedures (2020)** published by the Indo-Pacific Health & Supply Chain Data & Technical Assistance Centre (DTAC), in consultation with The mSupply Foundation and Beyond Essential Systems.

The original SOPs were written for National Medical Stores in Pacific Island Countries managing health commodities. For this tool, the framework has been generalised to apply to any warehouse operation — pharmaceutical, FMCG, manufacturing, e-commerce, 3PL, cold chain, or hazardous goods.

### Three pillars, fifteen sub-areas

| Pillar | Sub-areas |
|---|---|
| **A — Inbound Operations** | Preparation for Goods Receipt · Customs Clearance · Goods Arrival · Goods Inspection · Goods Receipt (System Entry) · Supplier Payments |
| **B — Order Fulfilment** | Order Processing · Picking & Assembling · Dispatch & Distribution |
| **C — Warehouse Maintenance** | Warehouse Storage · Storage Conditions · Stocktakes · Expiring Items · Supplementary Orders · Security |

### Maturity tiers

| Range | Tier |
|---|---|
| 90–100% | Best-in-Class |
| 75–89% | Compliant |
| 60–74% | Developing |
| 40–59% | Needs Improvement |
| 0–39% | Critical Gap |

## Architecture

A single self-contained `index.html` file. No build step, no framework, no dependencies beyond Google Fonts (loaded via `@import` from CSS).

```
warehouse-benchmark/
├── index.html              # The entire tool — HTML, CSS, JS in one file
├── apps_script_capture.gs  # Google Apps Script backend for data capture
└── README.md               # This file
```

Data flow on the gate unlock and report generation steps:

```
Browser (HTML tool)
    │
    │ POST application/text  (no-cors)
    ▼
Google Apps Script Web App
    │
    │ writes row(s)
    ▼
Google Sheet (in your Drive folder)
    ├── Leads tab        ← gate unlocks
    └── Assessments tab  ← report generations
```

## Setup

### 1. Configure data capture (one-time, ~10 minutes)

The tool ships with a placeholder for the Apps Script Web App URL. Until configured, the tool runs normally but skips data capture silently.

**Step 1 — Create the Apps Script project**

1. Open [script.google.com](https://script.google.com) → **New project**
2. Delete the default placeholder code
3. Open `apps_script_capture.gs` from this repo, copy all of it, paste into the editor
4. Rename the project to *Warehouse Benchmark Capture*
5. Save (Ctrl+S / Cmd+S)

**Step 2 — Set your Drive folder**

At the top of the Apps Script, the `FOLDER_ID` is set to a default. Change it to the ID of your own Drive folder:

```javascript
const FOLDER_ID = 'YOUR_DRIVE_FOLDER_ID_HERE';
```

The folder ID is the long alphanumeric string at the end of your Drive folder URL: `https://drive.google.com/drive/folders/THIS_PART_HERE`

**Step 3 — Test the script**

Run `testCreateSpreadsheet`, `testWriteLead`, and `testWriteAssessment` from the function dropdown. First run will request permissions:

- *Review permissions* → choose your account → *Advanced* → *Go to Warehouse Benchmark Capture (unsafe)* → *Allow*

(GitHub flags any unverified Apps Script as "unsafe" — your own scripts are always flagged this way.)

**Step 4 — Deploy as a Web App**

1. Click **Deploy** → **New deployment**
2. Click the cog icon → **Web app**
3. Settings:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Click **Deploy** → copy the **Web app URL**

**Step 5 — Wire it into the HTML**

Open `index.html`, find this block near the top of the `<script>` section:

```javascript
const APPS_SCRIPT = {
  enabled: true,
  url: 'PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE'
};
```

Replace the placeholder with the URL from Step 4. Save.

### 2. Host the HTML

The file is ~115 KB — too large for inline embedding in Google Sites (~10 KB cap). Host it anywhere that serves static HTML over HTTPS.

**GitHub Pages** (recommended):

1. Push `index.html` to a public GitHub repo
2. Settings → Pages → Source: *Deploy from a branch* → branch: `main` → folder: `/ (root)`
3. Live at `https://your-username.github.io/repo-name/`

**Netlify Drop** (no account needed for first 24 hrs):

1. Drag `index.html` to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Live URL appears immediately

### 3. Embed on Google Sites

1. Open your Site → **Insert** → **Embed** → **By URL** tab
2. Paste your hosted URL
3. Choose **Embed page** (not preview)
4. Resize the frame: ~1600 px tall, full width
5. Publish

## Data captured

### Leads tab (one row per gate unlock)

`Timestamp · Name · Email · Phone · User Agent · Source URL`

### Assessments tab (one row per report generation)

`Timestamp · Assessment ID · Name · Email · Phone · Designation · Organisation · Sector · Warehouse · Location · Size · SKUs · Date · Notes · Overall % · Tier · Pillar A/B/C % · 15 sub-area %s · 10 KPI values · Raw Responses (JSON) · User Agent · Source URL`

The Raw Responses column stores all 106 control scores as JSON, so individual control-level patterns can be analysed downstream.

## Updating the tool

**HTML changes:**
- *GitHub Pages*: edit `index.html` in the repo, commit. Live in ~1 minute. Hard-refresh your browser to bypass cache.
- *Netlify*: drag the new file to the Deploys tab.

**Apps Script changes:**
- Edit and save the script
- **Deploy → Manage deployments → pencil icon → Version: New version → Deploy**
- The Web App URL stays the same; no need to update the HTML

The Google Sites embed pulls the live URL on every page view, so no re-embedding is needed.

## Methodology notes

- **Equal weighting per control.** No single control is privileged over another. If your business model treats certain controls (e.g. cold chain integrity, controlled substance handling) as critical, treat low scores there as priority regardless of overall percentage.
- **N/A handling.** Controls marked Not Applicable are excluded from both numerator and denominator — score percentages remain meaningful even when entire sub-areas are skipped.
- **Snapshot in time.** A score reflects today's state. Operations drift; quarterly re-assessment is recommended.
- **Self-reported.** Results reflect the assessor's perception. An external audit may surface different findings.
- **Not a regulatory submission.** This tool does not replace WHO Good Storage and Distribution Practices (GSDP), GMP, GDP, ISO 9001, FSSAI, or any jurisdiction-specific compliance audit. It is a complement, not a substitute.

For deeper interpretation, gap remediation, or implementation support, please consult internal subject-matter experts or [reach out to the FreightFox team](https://freightfox.ai).

## Use cases

- **Initial baseline.** Establish where your warehouse stands today.
- **Internal audit prep.** Run before a scheduled audit to spot likely findings.
- **Vendor/3PL evaluation.** Have prospective logistics partners self-assess.
- **Quarterly maturity tracking.** Re-run every 90 days; compare in your Sheet.
- **M&A or due diligence.** First-pass operational review of a target warehouse network.
- **Training tool.** Walk team leaders through the controls in their section.

## Browser support

Tested on Chrome, Edge, Firefox, Safari (desktop and mobile). Requires a modern browser with `fetch`, `async/await`, and CSS Grid support — anything from 2019 onwards.

## Privacy & security

- All assessment scoring runs client-side in the browser
- Contact details (name, email, phone) and assessment results are POSTed to the configured Apps Script endpoint and stored in the tool operator's Google Drive
- No third-party analytics, no cookies, no localStorage
- The Apps Script Web App is deployed under your own Google account — you have full control over the data

## License

Open source — released for free use by the warehouse and supply chain community. Embed it on your own internal site, share with peers, fork and adapt for sector-specific extensions.

If you build something interesting on top of this, we'd love to see it.

## Credits

- **Source framework:** Warehouse Standard Operating Procedures, 2020. Indo-Pacific Health & Supply Chain Data & Technical Assistance Centre (DTAC).
- **Co-authors of original SOPs:** The mSupply Foundation; Beyond Essential Systems.
- **Tool design & engineering:** [FreightFox.ai](https://freightfox.ai)

## Feedback

Issues, feature requests, and contributions welcome via the GitHub repo. For implementation support or sector-specific adaptations, reach out to the FreightFox team directly.
