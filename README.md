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

The song is on the page's `<audio>` tag and also bundled with the site (`assets/music/DEVIL.mp3`, 4 MB). The bundled copy is served from the same host as the page, which matters: a URL on GitHub's raw hosts can 404 or be blocked by a network, and either way the page goes silent with nothing on screen to explain why. If the URL on the `<audio>` tag ever fails, `assets/app.js` moves on to the next copy of the same song, in this order:

1. the URL on the `<audio>` tag in `index.html`
2. `assets/music/DEVIL.mp3`
3. `https://raw.githubusercontent.com/all-drama/Nxnx/main/DEVIL.mp3`

Browsers don't allow sound until the visitor interacts with the page, so the music starts on their first tap, click or key press anywhere. There's no music button. `index.html` wires it the plain way:

```html
<body onclick="document.getElementById('lagu').play()">
  …
  <audio id="lagu" src="…/DEVIL.mp3" autoplay="true" loop preload="auto"></audio>
```

`assets/app.js` adds matching listeners on `document` (`pointerdown`, `pointerup`, `touchend`, `click`, `keydown`) as a backstop for taps that never reach `<body>` and for key presses, retries if the browser refuses the first attempt, and swaps in the next copy of the song when a URL fails. Because the tap calls `play()` directly, a tap always restarts the song — including after a pause from the phone's media controls. Clearing **Background music URL** in the admin panel leaves the page with no song to play, so it stays silent.

> URL forms: `https://github.com/<user>/<repo>/raw/refs/heads/main/<file>` redirects to a `…/refs/heads/main/…` raw URL that GitHub intermittently returns **404** for ([community discussion #53538](https://github.com/orgs/community/discussions/53538), [#146968](https://github.com/orgs/community/discussions/146968)). The plain `https://raw.githubusercontent.com/<user>/<repo>/main/<file>` form is the reliable one, which is why it sits last in the list above rather than first.

## Files

| File | Purpose |
|---|---|
| `index.html`, `assets/style.css`, `assets/app.js` | Public countdown page |
| `admin.html`, `assets/admin.css`, `assets/admin.js` | Admin panel (GitHub API) |
| `assets/common.js` | Shared date helpers |
| `assets/music/DEVIL.mp3` | The bundled background song (see above) |
| `data.json` | **All the data**: people and site settings |
| `scripts/remind.mjs`, `.github/workflows/reminders.yml` | Daily Telegram reminders |
| `.github/workflows/static.yml` | Deploys the site to GitHub Pages on every push to `main` |

## Run locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```
