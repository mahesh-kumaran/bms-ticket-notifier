# BMS Ticket Notifier

Automatically monitors multiple [BookMyShow](https://in.bookmyshow.com) movies for ticket availability and sends you an email alert when something changes.

Runs every n minutes via GitHub Actions. (You can set it via cron jobs, tho GH Actions doesn't kick in accurately)

## How It Works

1. Fetches showtimes from the BookMyShow API for each watch in `bms_watches.json`
2. Compares results with the previous check (stored in `bms_state.json`)
3. Sends an HTML email via [Resend](https://resend.com) if anything changed (new shows, dates opening, availability updates)

## Setup

### 1. Fork this repository

### 2. Set GitHub Secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|--------|-------------|
| `RESEND_API_KEY` | API key from [resend.com](https://resend.com) |
| `RESEND_FROM_EMAIL` | Email address to send notifications (use your own domain email or anything@resend.dev) |
| `RESEND_TO_EMAIL` | Email address to receive notifications |

### 3. Configure watches in `bms_watches.json`

Edit `bms_watches.json` in your repo. Add one object per movie watch:

| Field | Description | Example |
|-------|-------------|---------|
| `id` | Unique watch key used in state storage | `kalki-chennai` |
| `name` | Friendly label for logs | `Kalki - Chennai` |
| `url` | BookMyShow ticket page URL | `https://in.bookmyshow.com/movies/chennai/.../ET00123456` |
| `dates` | Optional dates (YYYYMMDD, comma-separated). Empty = from URL/default API response. | `20260710,20260711` |
| `theatre` | Optional theatre substring filter (comma-separated) | `PVR,IMAX` |
| `time_period` | Optional period filter | `evening,night` |

**Time periods:** `morning` (6–12), `afternoon` (12–16), `evening` (16–19), `night` (19–24)

### 4. Trigger the workflow

Go to **Actions → BMS Ticket Checker** and click **Run workflow**, or wait for it to run automatically every 30 minutes.

## Local Usage

Requires Python 3.14+ and [uv](https://docs.astral.sh/uv/).

```bash
uv sync --frozen

# edit bms_watches.json with one or more movies
export RESEND_API_KEY="re_..."
export RESEND_TO_EMAIL="you@example.com"
export RESEND_FROM_EMAIL="python@resend.dev"

uv run main.py
```

`bms_watches.json` is now the source of truth for movie URLs and per-movie filters (`dates`, `theatre`, `time_period`).

## Notifications

You'll receive an email when:
- A new showtime is added
- A date opens for booking
- Seat availability changes (e.g. sold out → available)

Emails show a summary of what changed and the current status of all monitored shows, grouped by theatre.
