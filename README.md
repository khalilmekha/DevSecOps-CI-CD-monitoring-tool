# DevSecOps CI/CD Vulnerability Watch Tool

> **Automated collection, analysis and distribution of DevSecOps & CI/CD vulnerabilities**  
> University project — USTHB, Faculty of Computer Science  
> Module: Technological Watch & Databases | 2025/2026

---

## Overview

This tool automates **DevSecOps vulnerability monitoring** by continuously scraping multiple sources, storing and analyzing the data, then distributing results through a web dashboard, PDF reports, and email alerts — all without manual intervention.

---

## Features

- 🔍 **Multi-source collection** — OSV API, GitHub Issues API, Docker & Kubernetes security bulletins
- 📊 **Dashboard** — global stats, severity breakdown, vulnerability trends over time
- 🔎 **Search & filtering** — by severity, component, date range
- 🧠 **NLP analysis** — automatic summaries and keyword extraction (NLTK + TF-IDF)
- 📄 **PDF report generation** — stats, charts, and tables via fpdf2
- 📧 **Email alerts** — subscribe for immediate or periodic reports via SMTP
- ⚙️ **Scheduled automation** — runs collectors every X hours in a background thread
- 🛠 **Admin panel** — trigger collection manually, manage automation settings

---

## Architecture

```
Sources                   Backend                  Output
────────                  ───────                  ──────
OSV API        ──┐
GitHub API     ──┼──▶  Collectors  ──▶  SQLite  ──▶  Flask  ──▶  Web Dashboard
Docker/K8s     ──┘        │                              │
HTML scraping             ▼                              ├──▶  PDF Reports
                       Analyzer                          └──▶  Email Alerts
                    (pandas + NLTK)
```

### Pipeline stages

1. **Collect** — HTTP requests to OSV & GitHub APIs, BeautifulSoup scraping for Docker/K8s pages. All records normalized to: title, description, severity, CVSS score, dates, URL.
2. **Store** — SQLite with `INSERT OR IGNORE` on unique keys — no duplicates across runs.
3. **Analyze** — pandas/numpy for stats and trends, NLTK + TF-IDF for NLP summaries, Matplotlib for charts.
4. **Distribute** — Flask web app + fpdf2 PDF generation + SMTP email alerts.
5. **Automate** — `schedule` library runs collectors in a background thread at configurable intervals.

---

## Project Structure

```
.
├── app.py                          # Flask app entry point & routes
├── database.py                     # SQLite access layer (VulnerabilityDB)
├── analyze.py                      # Stats, trends, NLP analysis
├── charts.py                       # Chart generation + PDF reports
├── automation.py                   # Scheduler & background automation
├── email_alerts.py                 # SMTP alert subscriptions
├── config.py                       # Dev / Prod / Test configurations
├── collectors/
│   ├── base_collector.py           # Shared collector logic
│   ├── osv_github_collector.py     # OSV API + GitHub Issues scraper
│   └── docker_k8s_collector.py     # Docker & Kubernetes HTML scraper
├── scrapers/
│   └── nvd_scraper.py              # (planned) NVD scraper
├── templates/                      # Jinja2 HTML templates
│   ├── index.html                  # Dashboard
│   ├── search.html                 # Search & filter
│   ├── reports.html                # Analysis & NLP
│   ├── alerts.html                 # Email subscription
│   └── admin.html                  # Admin panel
├── static/
│   ├── css/style.css
│   └── js/script.js
├── data/
│   └── vulnerabilities.db          # SQLite database
├── requirements.txt
└── .gitignore
```

---

## Getting Started

### Prerequisites

- Python 3.9+
- pip
- A Gmail account (or any SMTP provider) for email alerts

### Installation

```bash
git clone https://github.com/YOUR_USERNAME/DevSecOps-CI-CD-monitoring-tool.git
cd DevSecOps-CI-CD-monitoring-tool
pip install -r requirements.txt
```

Download required NLTK data (first run only):

```python
python -c "import nltk; nltk.download('punkt'); nltk.download('stopwords')"
```

### Environment variables

Create a `.env` file at the root:

```env
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password
ENABLE_AUTOMATION=true
AUTOMATION_INTERVAL_HOURS=4
SECRET_KEY=change-me-in-production
```

> **Note:** For Gmail, use an [App Password](https://myaccount.google.com/apppasswords), not your regular password.

### Run locally

```bash
python app.py
```

Visit `http://localhost:5000`

### Run with Gunicorn (production)

```bash
gunicorn app:app
```

---

## Deployment on Render

1. Push this repo to GitHub
2. Create a new **Web Service** on [Render](https://render.com)
3. Set:
   - **Build command:** `pip install -r requirements.txt`
   - **Start command:** `gunicorn app:app`
4. Add your environment variables in the Render dashboard
5. Deploy — the app stays live even when your local machine is off

---

## Tech Stack

| Purpose | Tool |
|---|---|
| Web framework | Flask + Jinja2 |
| Database | SQLite + SQLAlchemy |
| Scraping | requests, BeautifulSoup, lxml |
| Data analysis | pandas, numpy, scikit-learn |
| NLP | NLTK, TF-IDF |
| Charts | Matplotlib, Plotly, Seaborn |
| PDF generation | fpdf2, Pillow |
| Email | smtplib / python-dotenv |
| Automation | schedule |
| Production server | Gunicorn |

---

## Possible Improvements

- Add NVD, ExploitDB, CISA KEV as sources
- Replace NLTK with spaCy or an LLM/RAG pipeline for better summaries
- Migrate from SQLite to PostgreSQL for multi-instance deployments
- Replace BeautifulSoup with Scrapy for more robust scraping
- Real-time dashboard with WebSockets
- CSV/Excel export
- ML-based severity filtering

---

---

## License

Academic project — USTHB. All rights reserved.
