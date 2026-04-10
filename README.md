# TP Daily Actions Dashboard

A personal dashboard for Shopify Trust & Safety analysts, live at [nish-tp-daily.quick.shopify.io](https://nish-tp-daily.quick.shopify.io).

## What it does

Tracks your daily Trust Platform work in real time:

- **Personal Queue** — current count of open/unresolved tickets assigned to you (mirrors "My Unresolved" in TP)
- **New Tickets** — tickets assigned to you today
- **Total Today** — enforcement actions you've taken today
- **Action Breakdown** — bar chart of action types taken today (False Positive, Suspend, Request Docs, etc.)
- **Sidebar** — last 14 days of daily action counts, clickable to view that day's stats
- **Calendar** — monthly view with activity indicators
- **Notes** — color-tagged notes saved to quick.db, persist across sessions

## How it works

### Dashboard
Built as a single-page app hosted on [Quick](https://quick.shopify.io) (Shopify's internal static site platform). Uses:
- `quick.dw` — BigQuery queries for historical data (action breakdown, 14-day sidebar history)
- `quick.db` — real-time stats written by the background tracker, read by the dashboard on load
- Auto-refreshes every 5 minutes

### Background tracker (`~/.claude/daily-ticket-tracker.sh`)
A shell script that runs every 15 minutes via launchd (`com.nish.daily-ticket-tracker`). It uses Claude Code with MCP access to:
1. Query the Trust Platform API for your live "My Unresolved" PQ count, new tickets, and action count
2. Write the results to `quick.db` (collection: `tp-realtime-stats`)

The dashboard reads from `quick.db` first (real-time TP data), falling back to BigQuery for historical days.

## Setup

### Dashboard
```bash
cd nish-tp-daily
quick deploy . nish-tp-daily --force
```

### Background tracker
The tracker requires:
- Claude Code CLI (`claude`) in PATH
- MCP servers configured in `~/.claude.json`:
  - `trust-platform` — Trust Platform MCP
  - `quick-nish-tp-daily` — Quick MCP for nish-tp-daily site

Install the launchd job:
```bash
cp com.nish.daily-ticket-tracker.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.nish.daily-ticket-tracker.plist
```

To run manually at any time:
```bash
bash ~/.claude/daily-ticket-tracker.sh
```

## Files

- `index.html` — the entire dashboard (HTML/CSS/JS, single file)
