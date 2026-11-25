# Rathan's UK IT Job Bot — Setup Guide

This is the same automation as Yuvaraj's and Vineela's bots (the same `run_jobs.py`), set up for Rathan in London:

| | |
|---|---|
| **Roles (priority)** | 1st Line IT Support / Service Desk → Junior Network Engineer → Junior NOC Engineer. Also IT Support Technician, Helpdesk Analyst, Network Support, Junior Cyber/SOC Analyst, Infrastructure Support, Technical Support, Desktop Support, IT graduate schemes |
| **Experience** | Keeps jobs asking 0–2 years. Drops 3+ years, and Senior, Lead, Manager, 3rd line and Architect titles. Apprenticeships and student placements are dropped too |
| **Areas** | London, Hertfordshire, Surrey, Reading/Berkshire, and remote anywhere in the UK. Manchester, Birmingham etc. are dropped |
| **Flags on the job card (never a rejection)** | ⚠️ SC/DV clearance (these jobs are also ranked lower) · 🛂 no sponsorship / right to work · 📜 CCNA/CompTIA mentioned · 🌱 entry-level friendly · 🚗 driving licence · 🌙 shifts |
| **Sources** | LinkedIn, Indeed UK, Google Jobs, the **Reed.co.uk official API**, and careers pages for BT, Sky, Virgin Media O2, Computacenter, Capgemini, Accenture, IBM, Microsoft, Cisco and Sopra Steria (via Firecrawl) |
| **Schedule (UK time)** | 8 AM (overnight digest), 11 AM, 2 PM, 5 PM, 8 PM, plus `/scan` any time. Quiet from 8:30 PM to 7:30 AM. Summer time is handled automatically |

In IST his scans land on the half hour (for example 1:30 PM), so they never overlap with Yuvaraj's bot (8/11/2/5/8/11) or Vineela's (9/12/3/6/9).

His details live in **`profile.json`**, **`master_resume.md`** and **`selftest_cases.json`**. `run_jobs.py` is shared with the other two bots.

---

## Step 1: Create the repository (in YOUR GitHub account)

1. Go to https://github.com/new
2. Set **Repository name** to `job-automator-rathan`.
3. Choose **Public**, as you decided. Public repos get unlimited free Actions minutes.
   - His phone and email are not in the files; they come from secrets (Step 6).
   - The job database is never committed.
   - Note: his resume content (`master_resume.md`) will be publicly readable, much like a LinkedIn profile.
4. Don't add a README, .gitignore or license.
5. Click **Create repository**.

## Step 2: Push the folder (PowerShell)

Unzip `job-automator-rathan.zip`, for example to `C:\Users\<you>\Projects\job-automator-rathan`. Then run:

```powershell
cd C:\Users\<you>\Projects\job-automator-rathan
git init
git add .
git commit -m "Rathan job bot: UK IT support / network roles, London + remote UK, Reed + Firecrawl"
git branch -M main
git remote add origin https://github.com/yuvarajkaleru/job-automator-rathan.git
git push -u origin main
```

Check on GitHub that `.github/workflows/job_runner.yml` is there.

## Step 3: Create his Telegram bot (on Rathan's phone)

1. In Telegram, open **@BotFather** (blue tick) and send `/newbot`.
2. For the name, send `Rathan Job Alerts`.
3. For the username, send something ending in `bot`, for example `rathan_jobs_alert_bot`.
4. Copy the **token** it sends; this is `TELEGRAM_BOT_TOKEN`.
5. Open the new bot, tap **Start**, and send `hi`.
6. In a browser, open `https://api.telegram.org/bot<TOKEN>/getUpdates`.
7. Find `"chat":{"id":...`. That number is `TELEGRAM_CHAT_ID`.

## Step 4: Get a Reed API key (free, about 2 minutes)

1. Go to https://www.reed.co.uk/developers/jobseeker
2. Sign in, or create a Reed account with Rathan's email.
3. Register for the **Jobseeker API**. Reed shows or emails an **API key**; this is `REED_API_KEY`.

The bot searches Reed for his 12 role keywords within about 45 miles of London. It only fetches the full description for new jobs that already match his titles and areas.

## Step 5: Get a Firecrawl API key

1. Go to https://www.firecrawl.dev, click **Sign up**, and use Rathan's Google account. The free plan needs no card.
2. Open **Dashboard → API Keys** and copy the key (`fc-...`); this is `FIRECRAWL_API_KEY`.

The bot visits 4 careers pages a day, taking the 10 companies in turn. It stops at 450 credits a month.

**Check the careers URLs once.** Open each URL in `companies.txt` in a browser; each one should show a list of jobs. If a page doesn't, search on the company's site (e.g. "support", United Kingdom) and paste the resulting URL into `companies.txt`. Big company sites change their addresses often. The Telegram scan summary tells you when a page found "no matching links".

## Step 6: Add the secrets

On GitHub, open **job-automator-rathan → Settings → Secrets and variables → Actions → New repository secret** and add:

| Name | Value |
|---|---|
| `TELEGRAM_BOT_TOKEN` | token from Step 3 |
| `TELEGRAM_CHAT_ID` | chat id from Step 3 |
| `CLAUDE_CODE_TOKEN` | the same value as in your own repo |
| `RESUME_EMAIL` | `rathantudugani@gmail.com` |
| `RESUME_PHONE` | `+44 7748 204452` |
| `REED_API_KEY` | key from Step 4 |
| `FIRECRAWL_API_KEY` | key from Step 5 |

## Step 7: First test run

1. Open the **Actions** tab and enable workflows if GitHub asks.
2. Click **Rathan UK IT Job Bot → Run workflow**, choose mode **scrape**, and run it.
3. After about 5–10 minutes he gets two lists in Telegram (🔥 VERY RECENT and 📅 LAST 48 HOURS) and a scan summary showing every source, including Reed and the careers pages.
4. Tap **Apply** on a job, then run the workflow again with mode **approvals**, or wait for Step 9. His tailored CV PDF, DOCX and cover letter will arrive. They use UK spelling and mention no visa details.

## Step 8: GitHub token for the scheduler

1. Go to https://github.com/settings/personal-access-tokens/new
2. Set **Name** to `rathan-relay` and **Expiration** to 1 year.
3. Under **Repository access**, choose **Only select repositories** → `job-automator-rathan`.
4. Under **Permissions → Repository permissions**, set **Actions** to **Read and write**.
5. Click **Generate** and copy the token.

## Step 9: Cloudflare Worker (a separate worker)

1. Go to **Workers & Pages → Create → Create Worker**, name it `rathan-job-relay`, and click **Deploy**.
2. Click **Edit code**, paste all of `cloudflare-worker.js`, and click **Deploy**.
   - This worker reads the **UK clock**, so 8 AM means 8 AM in London in both winter and summer.
3. Go to **Settings → Variables and Secrets** and add:
   - `TELEGRAM_BOT_TOKEN`: Secret
   - `CHAT_ID`: Text, his chat id
   - `GH_TOKEN`: Secret, the token from Step 8
   - `GH_REPO`: Text, `yuvarajkaleru/job-automator-rathan`
4. Go to **Settings → Triggers → Cron Triggers** and add `* * * * *`.

## Step 10: Test from his phone

`/status` (shows the next scan time in UK time) · `/scan` · `/why` · `/stats` · `/export` · `/help`

---

## Make his CV stronger (edit `master_resume.md`)

The bot will **never** claim skills that aren't in his master resume. Many UK 1st line jobs ask for:

**Active Directory, Microsoft 365 / Office 365, Windows 10/11, ServiceNow (or another ticketing tool), ITIL basics, PowerShell, Intune, VMware.**

If he has genuinely used any of these at Entain, at university, or in a home lab, add them under TECHNICAL SKILLS. The bot will then use them on matching jobs. Add certifications (CCNA, CompTIA) as soon as he earns them.

## Changing things later

| Want to… | Edit |
|---|---|
| Add or remove job titles | `profile.json → search_terms` (the first 4 also run on LinkedIn) and `reed.terms` |
| Accept more years | `experience_max_years` |
| Add a town | the matching `keywords` list in `profile.json → locations` |
| Skip clearance jobs entirely | change the clearance flag's `points` from -8 to -100 |
| Watch another company | `companies.txt` |
| Change scan times | `profile.json → schedule` **and** `SCAN_HOURS_IST` in the worker **and** the cron line in the workflow |

Run `python run_jobs.py selftest` after editing the filters. It checks 35 example jobs from `selftest_cases.json`.
