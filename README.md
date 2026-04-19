# MJSR Clinic Dispensary Portal

Full medical-records and medicine-stock system for the MJSR onsite clinic.  
Follows the standard Samsons Group architecture: GitHub Pages + Google Apps Script + Google Sheets.

## What you get

- **5 portal tabs**: Dashboard, New Visit, Visit Log, Stock Levels, Config
- **186 medicines** pre-parsed from the March 2026 stock register, auto-categorised into 18 therapeutic categories, 5 expiry alerts already flagged
- **Full visit capture**: 4 patient types, vitals (BP/pulse/temp/SpO2/RR), chief complaint, symptoms, diagnosis, treatment, medicines with qty (auto-deducts stock), follow-up flag, referral-to field
- **Category-based par levels** with per-item overrides (editable in Config tab)
- **Auto low-stock email alerts** to bilal.tariq@malamjabbaresort.com when any item hits MIN after a dispense
- **Audit log, CSV exports, soft-delete for visits**

## Deployment steps

### 1. GitHub repo
1. In the `samsons-digital` org, create a new repo named `mjsr-clinic` (public)
2. Upload all files preserving the folder structure:
   ```
   mjsr-clinic/
   ├── index.html
   ├── Code.gs              (not served - kept here for reference)
   ├── assets/
   │   ├── styles.css
   │   ├── app.js
   │   └── seed_medicines.js
   └── data/
       └── medicines_seed.json
   ```
3. Settings > Pages > Source: `main` branch > `/ (root)` > Save
4. Your portal will be at `https://samsons-digital.github.io/mjsr-clinic/`

### 2. Google Sheet + Apps Script
1. Create a new Google Sheet named `MJSR Clinic Master Sheet`
2. Copy the **Sheet ID** from the URL (`/spreadsheets/d/THIS_PART/edit`)
3. Extensions > Apps Script
4. Delete the default `Code.gs` content, paste in the contents of `Code.gs` from this bundle
5. Replace `__REPLACE_WITH_SHEET_ID__` at the top with your Sheet ID
6. Save (Ctrl+S), name the script `MJSR Clinic Backend`
7. **First deployment only**: Deploy > New Deployment > gear icon > **Web app**
   - Description: `v1.0 initial`
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Deploy > Authorise > copy the **Web app URL**
8. ⚠️ **For all future code changes**: use Deploy > **Manage Deployments** > edit pencil > Version: **New Version** > Deploy (this keeps the same URL).  NEVER click "New Deployment" after this first time.

### 3. Wire up the frontend
1. Open `assets/app.js`
2. Replace `REPLACE_WITH_APPS_SCRIPT_URL` (line near the top) with your Web app URL from step 2.7
3. Commit and push to GitHub

### 4. First-run setup
1. Open the portal — it will show "Offline" until you bootstrap
2. Go to the **Config** tab, enter PIN `1234`, click **Remember for session**
3. Click **Bootstrap Sheets** — this creates all 5 sheet tabs with headers and seeds category pars into the Config sheet
4. Click **Seed 186 Medicines** — this loads every item from the March 2026 register into Medicine_Master
5. Refresh the portal, go to Dashboard — you should see 186 SKUs, 5 expiry alerts, 74 out-of-stock items (all matching the March register)

### 5. Test a sample visit
- Go to **New Visit** tab
- Select doctor, patient type, name
- Add a medicine, qty 1
- Submit
- Check Dashboard — should show 1 visit, 1 medicine dispensed
- Check the Sheet — row added to Patient_Visits and Stock_Movements, balance decremented in Medicine_Master

## Key settings to adjust

In the `Config` sheet tab directly (no redeployment needed):

| Key | Value |
|---|---|
| `DOCTORS` | Comma-separated list of doctor names for the dropdown |
| `ALERT_EMAIL` | Who gets low-stock alerts (default: bilal.tariq) |
| `NOTIFY_GM` | GM email for any escalations |
| `LOW_STOCK_ALERTS_ENABLED` | `TRUE` or `FALSE` |

## Technical notes

- All fetch() calls use `Content-Type: text/plain` with JSON in body (established pattern)
- All sheet writes use Date objects, never formatted strings
- Admin PIN stored in session only, never localStorage
- No em dashes used anywhere
- Soft-delete pattern for visits (DeletedAt/DeletedBy columns, never hard-removed)
- 5 pre-flagged expiry items from March register: Inj Transim, Cap Betrall 250mg, Inj Atropin, Inj Dextro 25%, Inj Adrnalin

## Category par defaults (pre-seeded, editable in Config tab)

| Category | MIN | PAR |
|---|---|---|
| Antibiotic | 30 | 80 |
| Analgesic / NSAID | 50 | 150 |
| Gastrointestinal | 30 | 100 |
| Anti-allergy | 20 | 60 |
| Fluid / Electrolyte | 5 | 15 |
| Steroid | 10 | 30 |
| Cardiac / BP | 20 | 60 |
| Anti-diabetic | 20 | 60 |
| CNS / Psychiatric | 15 | 40 |
| Supplement | 15 | 45 |
| Respiratory | 20 | 60 |
| Dermatology | 3 | 10 |
| Eye / Ear | 3 | 10 |
| Injection (other) | 5 | 20 |
| Consumable | 10 | 40 |
| Antiemetic | 15 | 40 |
| Other Rx | 10 | 30 |
| Other | 5 | 20 |

