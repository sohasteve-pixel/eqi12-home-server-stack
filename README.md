# Beelink EQi12 Home Server Stack

A tested Docker Compose stack for running a home server on a mini PC (Beelink EQi12, Intel Core i3-1215U). Includes nginx, PostgreSQL 17, Redis 8 and Jellyfin with sensible defaults, health checks and persistent volumes.

> 📊 **Looking for the raw data behind these claims?** See [eqi12-measurement-data](https://github.com/sohasteve-pixel/eqi12-measurement-data) — the open power / codec / storage / network measurement logs this stack was validated against.

## What's included

| Service | Image | Port | Purpose |
|---|---|---|---|
| nginx | `nginx:1.27-alpine` | 8080 | Reverse proxy / static serving |
| PostgreSQL | `postgres:17-alpine` | 5432 | Application database |
| Redis | `redis:8-alpine` | 6379 | Cache / session store |
| Jellyfin | `jellyfin/jellyfin:10` | 8096 | Media server with Intel QSV ready |

## Quick start

```bash
git clone https://github.com/sohasteve-pixel/eqi12-home-server-stack.git
cd eqi12-home-server-stack
cp .env.example .env
# edit .env and set real passwords
docker compose up -d
docker compose ps
```

## Why this stack

This is not a generic example. It is the stack actually running on a measured [Beelink EQi12 home server](https://homelabtoolkit.com/build/beelink-eqi12-windows-home-server/) and verified through repeated restart cycles, Docker auto-start tests and a 12.57-hour stability window.

- **Power:** 12W idle at the wall with all services running — about $20/year at US average electricity rates. See the [home server power cost calculator](https://homelabtoolkit.com/build/home-server-power-cost-guide/) with the full measured dataset.
- **Recovery:** verified 5/5 normal-restart cycles, 5/5 shutdown WOL cycles and 3/3 AC-loss recovery cycles. See the [Docker auto-start guide](https://homelabtoolkit.com/build/docker-desktop-auto-start-home-server/).
- **Media:** Jellyfin with Intel Quick Sync hardware transcoding verified at ~13W for two simultaneous 4K-to-1080p streams. See the [Jellyfin Intel QSV setup guide](https://homelabtoolkit.com/build/jellyfin-intel-qsv-windows/) and the [QSV codec matrix](https://homelabtoolkit.com/lab/intel-i3-1215u-qsv-codec-support/).

## Notes

- `restart: unless-stopped` is set on every service so the stack returns after a host reboot.
- Health checks are defined for every service so `docker compose ps` shows real readiness, not just "running".
- The Jellyfin `devices` block for `/dev/dri` (Intel QSV) is commented out — uncomment after confirming your platform exposes the iGPU.
- The `.env` file is gitignored. Never commit real passwords.

## License

MIT — see [LICENSE](LICENSE).
