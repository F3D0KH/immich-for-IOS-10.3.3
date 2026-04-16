# Immich Lite — Immich viewer for legacy devices (iOS 10 / WebKit 4)

A lightweight, single-file web client for [Immich](https://immich.app) built specifically for old devices that cannot run the official interface — such as the iPad 4 (2012) running iOS 10.3.3.

---

## How it works

The official Immich web app is a modern React SPA that relies on ES2020+ JavaScript, CSS Grid, and other APIs that old WebKit engines simply do not support. When an iPad 4 tries to open it, the page either fails silently or renders broken — because its WebKit 4 engine (shipped with iOS 10) has no idea what to do with that code.

Immich Lite solves this by sitting in front of Immich as a reverse proxy powered by **nginx**. Here is what happens on every request:

```
Old Device (iOS 10)
       │
       │  HTTP request → port 8090
       ▼
  immich-lite (nginx)
       │
       ├─ /api/* → proxied to your Immich server (port 2283)
       │            All API calls reach Immich unchanged.
       │
       └─ /*     → serves index.html (Immich Lite UI)
                    Written in ES5-only JavaScript with no
                    frameworks, no fetch(), no arrow functions —
                    everything old Safari can understand.
```

Your old device talks to `immich-lite` on port 8090. nginx forwards every `/api/` request straight to your real Immich server, so all authentication, photo data, and media files come directly from your own instance. The only thing immich-lite serves is the HTML/JS/CSS shell.

The UI is intentionally minimal — no build tools, no dependencies, just one HTML file.

---

## Features

- **All Photos** — paginated timeline of your entire library, grouped by month
- **Albums** — browse any album from your Immich account
- **Private Folder** — PIN-protected access to your locked folder via session elevation
- **Timeline scrubber** — right-side year/month bar for quick navigation
- **Lightbox** — tap any photo or video to view it full-screen
- **Infinite scroll** — loads the next page automatically as you scroll down

---

## Requirements

- A running [Immich](https://immich.app) instance on your local network
- Docker (for the nginx container)
- Any device with a browser — tested on iPad 4 / iOS 10.3.3 / Safari

---

## Setup

### 1. Download the files

Download `index.html`, `nginx.conf.template`, and `compose.yaml` from this repository.

### 2. Configure compose.yaml

Open `compose.yaml` and set `IMMICH_URL` to the address of your Immich server:

```yaml
services:
  immich-lite:
    image: nginx:alpine
    container_name: immich-lite
    restart: unless-stopped
    ports:
      - 8090:80
    environment:
      - IMMICH_URL=http://192.168.1.129:2283  # <- your Immich address here
    volumes:
      - /opt/immich-lite:/usr/share/nginx/html:ro
      - /opt/immich-lite/nginx.conf.template:/etc/nginx/templates/default.conf.template:ro

networks:
  immich_default:  # <- immich lite need connect to real immich server
    external: true
```

> **Important:** Make sure the `immich-lite` container is on the same Docker network as your Immich stack (`immich_default`). If your Immich uses a different network name, update it in `compose.yaml` accordingly.

### 3. Start the container once to create the directory

```bash
docker compose up -d
```

This creates `/opt/immich-lite/` on your host.

### 4. Copy the files into place

```bash
sudo cp index.html nginx.conf.template /opt/immich-lite/
```

### 5. Restart the container

```bash
docker compose restart immich-lite
```

Open `http://<your-server-ip>:8090` on your old device. That's it.

---

## Usage

### Signing in

1. Open `http://<your-server-ip>:8090` in your browser.
2. Enter your Immich **email** and **password**.
3. Optionally enter an **API Key** (generated in Immich → Account Settings → API Keys). If left empty, the session token from login is used automatically.
4. Tap **Connect**. Your credentials are saved in `localStorage` so you only need to sign in once.
5. To sign out, tap **Sign out** on the login screen (shown when credentials exist).

### All Photos tab

- Displays your entire library grouped by month, newest first.
- Scroll down to load more — new pages are fetched automatically.
- Tap any thumbnail to open it full-screen.
- Videos play directly in the lightbox.
>> **NOTE** : Sometimes video in fullscreen don't start right away, so to start them, press once on button in the player.

### Albums tab

- Appears automatically after the album list is fetched.
- Tap **Albums ▾** to open the dropdown and pick any album.
- The tab label updates to the selected album name.

### Private (Locked Folder) tab

- Tap **🔒 Private** to open the PIN prompt.
- Enter the PIN you set in Immich (Account Settings → Account → Locked Folder PIN).
- On success, the session is elevated and your locked assets are loaded.
- The tab changes to **🔓 Private** while unlocked.
- Tap it again to lock and return to All Photos.
- The PIN is never stored — you will be prompted again after a page refresh.

### Timeline scrubber

- The vertical bar on the right shows years and month initials.
- Tap any label to jump directly to that month.

### Lightbox

- Tap any photo or video to open it full-screen.
- Tap **×** in the top-right corner to close.

---

## Privacy & Data

**All of your data stays on your own server.**

Immich Lite is a static HTML file served by nginx. It makes API calls directly to your own Immich instance — there are no third-party servers, no analytics, no telemetry, and no external requests of any kind.

- Your photos and videos are fetched from **your Immich server** and displayed directly in your browser.
- Your login credentials are stored only in your **browser's localStorage** on your device.
- Your PIN is **never stored** anywhere — it is sent once to your Immich server for session elevation and then discarded.
- This project has no backend of its own. The author has no access to your data, your server, or your account.

---
## Tested devices:
- Ipad 4 older
- Iphone 7 Plus newer
- Iphone XS newer
- PC of course
- Iphone 3G (IN FUTURE) it's gonna be cool right?

## Proofs of work on IPAD

<img width="1529" height="958" alt="IMG_1" src="https://github.com/user-attachments/assets/5b4cd952-5e57-4faa-a21a-7f0ccb2a439e" />




<img width="1536" height="1914" alt="IMG_2" src="https://github.com/user-attachments/assets/b2b4cf76-0174-4639-a2c4-b2e32737adce" />

## License

MIT
