# 🎉 Birthday Countdown

A birthday countdown site for friends and family. It runs on GitHub Pages and has a built-in **admin panel**, so you can add, edit and delete people without touching any code.

- 🌐 **Live site:** https://birthday-wish00.github.io/wishesh/
- 🔐 **Admin panel:** https://birthday-wish00.github.io/wishesh/admin.html

## Features

- Live countdowns (days / hours / minutes / seconds), sorted by the next birthday
- A "Next birthday" spotlight with a big countdown. On the day, it switches to a 🎂 celebration with confetti and a *Send wishes on WhatsApp* button
- Search, plus filters for *This month* and *Next 30 days*
- Shows the age each person is turning, a progress bar, and an optional note per person
- Floating balloons, twinkling stars and glass cards. Works on mobile
- Optional background music that starts on the first tap or click anywhere on the page (no music button)
- **Admin panel**: add / edit / delete people, change the title and subtitle, bulk import (the old `Name` + `DD MM YYYY` text format also works) and export a backup
- **Telegram reminders** (optional) for birthdays today and tomorrow, sent daily by GitHub Actions

## How the admin panel saves to GitHub

There's no server or database. All data is stored in **`data.json`** in this repo.

```
admin.html ──(your token)──▶ GitHub API: update data.json ──▶ new commit on main
                                                                  │
live site ◀── GitHub Pages redeploys (≈1 min) ◀── "Deploy static content" workflow
```

1. You open `admin.html` and paste a GitHub token (one-time setup, see below).
2. The panel reads `data.json` through the GitHub API. You make your changes, then click **☁️ Save to GitHub**.
3. The save is a normal **Git commit** (e.g. `Admin: add Rahul Das`). The existing Pages workflow redeploys the site, and the change goes live in about a minute.
4. Every change is kept in the Git history, so you can always see or restore an older version.

### Create the admin token (one-time, about 1 minute)

1. Go to **GitHub → Settings → Developer settings → [Fine-grained tokens → Generate new token](https://github.com/settings/personal-access-tokens/new)**.
2. **Repository access:** *Only select repositories* → `birthday-wish00/wishesh`.
3. **Permissions → Repository permissions → Contents:** *Read and write*.
4. Generate it, copy it, and paste it into the admin panel's login screen.

The token is stored only in your browser (localStorage, or sessionStorage if you untick *Remember me*) and is only sent to `api.github.com`. Anyone can open `admin.html`, but without a token that can write to this repo they can't change anything. Don't share your token. If it leaks, delete it on GitHub and make a new one.

## Telegram reminders (optional)

`.github/workflows/reminders.yml` runs every day at 08:00 IST. It messages you when someone's birthday is today or tomorrow.

1. Create a bot with [@BotFather](https://t.me/BotFather) and copy its token. Send your bot a message, then get your chat id (for example from [@userinfobot](https://t.me/userinfobot)).
2. In the repo go to **Settings → Secrets and variables → Actions → New repository secret** and add:
   - `TELEGRAM_TOKEN`
   - `TELEGRAM_CHAT_ID`
3. Test it from **Actions → Birthday reminders → Run workflow**.

The token is stored as a GitHub secret, so it's **never visible** on the website. If no secrets are set, the workflow just skips sending.

## Background music (optional)

Upload an mp3 to the repo (e.g. `music.mp3`), then enter `music.mp3` under **Admin → Site settings → Background music URL** and save.

Browsers don't allow sound until the visitor interacts with the page, so the music starts on their first tap, click or key press anywhere. There's no music button: `<body onclick="playMusic()">` in `index.html` does it, and `assets/app.js` adds the same listeners for taps that never reach `<body>`, plus a retry if the browser refuses the first attempt. The song URL also sits on the `<audio>` tag in `index.html` as a fallback, so a tap works even before `data.json` has loaded — whatever is in **Background music URL** wins once the data arrives, and clearing it turns the music off. The song loops until the tab is closed, and visitors can still pause it from their phone's media controls.

> Use the plain `https://raw.githubusercontent.com/<user>/<repo>/main/<file>.mp3` form for the URL. The `https://github.com/<user>/<repo>/raw/refs/heads/main/<file>.mp3` form redirects to a `…/refs/heads/main/…` raw URL that GitHub intermittently returns 404 for, which leaves the music silent.

## Files

| File | Purpose |
|---|---|
| `index.html`, `assets/style.css`, `assets/app.js` | Public countdown page |
| `admin.html`, `assets/admin.css`, `assets/admin.js` | Admin panel (GitHub API) |
| `assets/common.js` | Shared date helpers |
| `data.json` | **All the data**: people and site settings |
| `scripts/remind.mjs`, `.github/workflows/reminders.yml` | Daily Telegram reminders |
| `.github/workflows/static.yml` | Deploys the site to GitHub Pages on every push to `main` |

## Run locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```
