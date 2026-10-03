# VodiWalker — Base44 Dev Notes

## What this is
A single-process FastAPI app (VodiWalker VPN management panel) written in Python.
All UI is server-rendered HTML inside `main.py` / `pages.py` (no separate frontend build).
Persists state to a JSON file in the `data/` directory.

## Running it
- `docker compose -f docker-compose.base44.yml up -d` — builds nothing; uses `python:3.11-slim`,
  bind-mounts the repo at `/app`, installs `requirements.txt` on startup, then runs
  `uvicorn main:app --host 0.0.0.0 --port 3000 --reload` (live reload via WatchFiles).
- Web entry point is host port **3000** (mapped from container 3000; `PORT=3000` env).
- Healthcheck: `GET /health` → `{"status":"ok",...}`.
- Data volume `vodiwalker_data` persists `/app/data` (state + auto-generated secret key).

## Environment / credentials
- **No external credentials are required to boot.** The panel starts fine with defaults.
- `SECRET_KEY` — auto-generated and persisted to `data/vodiwalker_secret.key` if unset.
- `ADMIN_USERNAME` / `ADMIN_PASSWORD` — default to `admin` / `admin` (overridable via env).
- `TELEGRAM_BOT_TOKEN` — **optional**. Without it the Telegram bot stays off (logged as a warning,
  not an error). Provide it via the Secrets dashboard only if you want the bot feature.
- `RAILWAY_VOLUME_MOUNT_PATH` — set to `/app/data` in compose so state persists in the volume.

## Key files
- `main.py` — the whole FastAPI app: config, auth, sessions, link/sub generation, all routes,
  and the embedded dashboard/login HTML. Very large (~9400 lines).
- `pages.py` / `telegram_bot.py` / `nodes.py` / `outbound_proxy.py` / `tcp_relay.py` /
  `relay_vless.py` / `xhttp_siz10.py` — feature modules imported at startup, each guarded with
  try/except so a missing/broken module degrades gracefully instead of crashing the app.

## Verifying it works
- `curl http://localhost:3000/health` → JSON `status: ok`.
- Browse to `/` → redirects to `/login` (unauthenticated) or `/dashboard` (logged in).
- Default login: username `admin`, password `admin`.
