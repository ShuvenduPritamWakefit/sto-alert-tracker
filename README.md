# STO Alert Tracker — Automated Daily Run

Runs `sto_alert_tracker_shuvendu.ipynb` automatically **every day at 06:21 AM IST**
using GitHub Actions. No server or laptop needs to be on.

---

## ⚠️ IMPORTANT: This repo MUST be Private

The notebook contains hard-coded credentials (Snowflake login, a Google
service-account private key). If this repo is **Public**, bots will scrape those
keys within minutes and Google/Snowflake will auto-revoke them — breaking the job.

👉 When you create the repo on GitHub, choose **Private**.

---

## What the notebook does (the flow)

1. Connects to **Snowflake** (`query_execute` tries schema `SCM` → `PUBLIC` → `CRM`).
2. Connects to **Google Sheets** via a service account.
3. Pulls `wf_master_warehouse` (id, warehouse_code, plant_code) from Snowflake.
4. Reads the `cc_output` tab of the STO Alerts sheet, keeps **today → +7 days**.
5. Merges the warehouse master, filters to warehouses listed in `input_config`.
6. Writes raw data to the `sto_alert_raw` tab.
7. Computes `stress_level` (inventory ÷ design holding capacity) + helper keys.
8. Writes the final result to the `sto_alert` tab.

It's a pure **read → transform → write** job — ideal for unattended automation.

---

## Repo structure

```
sto-alert-tracker/
├── .github/
│   └── workflows/
│       └── sto_alert.yml          # the schedule + run steps (the automation)
├── sto_alert_tracker_shuvendu.ipynb  # your notebook (runs unchanged)
├── requirements.txt               # pinned deps + papermill
├── .gitignore
└── README.md
```

---

## How the schedule works

GitHub Actions cron is **always in UTC**.

| Local (IST) | UTC   | Cron expression |
|-------------|-------|-----------------|
| 06:21 AM    | 00:51 | `51 0 * * *`    |

> Note: GitHub's scheduled runs are best-effort and can be delayed a few minutes
> under load. For a daily alert that's fine. If you ever need exact timing,
> a server cron job is the alternative.

---

## One-time setup (step by step)

You need: a GitHub account, and `git` installed on your Mac.

### 1. Create a Private repo on GitHub
- Go to <https://github.com/new>
- Repository name: `sto-alert-tracker`
- Visibility: **Private**  ← required
- Do **not** add a README/.gitignore (we already have them)
- Click **Create repository**

### 2. Push this folder to GitHub
Open Terminal and run (copy your repo URL from the GitHub page):

```bash
cd "/Users/shuvendu.pritam/Desktop/Shuvendu Pritam Das/sto-alert-tracker"

git init
git add .
git commit -m "Initial commit: STO alert tracker + daily GitHub Actions schedule"
git branch -M main
git remote add origin https://github.com/<YOUR_USERNAME>/sto-alert-tracker.git
git push -u origin main
```

### 3. Confirm Actions is enabled
- In your repo: **Settings → Actions → General**
- Under "Actions permissions" make sure **Allow all actions** is selected.

### 4. Test it right now (don't wait for 6:21 AM)
- Go to the **Actions** tab → **STO Alert Tracker (Daily)** → **Run workflow**.
- Watch the run. Green check = success.
- If it fails, open the run → download the **executed-notebook** artifact to
  see exactly which cell errored.

That's it. From now on it runs automatically every day at 06:21 AM IST.

---

## Updating the notebook later

Whenever you change the notebook locally, push the update:

```bash
cd "/Users/shuvendu.pritam/Desktop/Shuvendu Pritam Das/sto-alert-tracker"
cp "../sto_alert_tracker_shuvendu.ipynb" .   # if you edited the original
git add .
git commit -m "Update notebook logic"
git push
```

## Changing the run time

Edit the cron in `.github/workflows/sto_alert.yml`, remembering to convert
**IST → UTC** (subtract 5h30m), then commit & push.

## Troubleshooting

- **Run failed** → download the `executed-notebook` artifact; the failed cell
  shows the traceback.
- **Credentials rejected** → the Snowflake password or Google key may have been
  rotated; update the notebook and push.
- **Schedule didn't fire** → GitHub disables scheduled workflows in repos with
  no activity for 60 days; any push re-enables them. Also confirm the repo
  isn't Public (keys revoked).
