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

## Backing the stack up

Docker does not back up your volumes. Export each service with its own tool and restore-test the result, rather than copying `/var/lib/docker/volumes` while the stack is running:

```bash
docker compose exec -T database pg_dump -U app app > backup-postgres-$(date +%Y%m%d).sql
docker compose exec -T cache redis-cli -n 0 BGSAVE
docker compose cp cache:/data/dump.rdb ./backup-redis-$(date +%Y%m%d).rdb
```

Measured on this stack: nginx returns to healthy in 0.660 s, PostgreSQL in 0.833 s and Redis in 2.042 s after a container restart, with persistent data intact. The full drill, exact commands and raw evidence are in the [Docker Compose backup and restore drill](https://homelabtoolkit.com/build/docker-compose-backup-restore-drill/). To work out how many generations to keep and how much disk they cost, use the [backup retention planner](https://homelabtoolkit.com/tools/backup-retention-planner/).

## Related reading

If you are deciding what to run on this class of hardware before committing to a stack:

- [Windows mini PC home server roadmap](https://homelabtoolkit.com/build/windows-mini-pc-home-server-roadmap/) — the end-to-end path from bare metal to a running server.
- [Docker on Windows vs Proxmox on the same mini PC](https://homelabtoolkit.com/compare/windows-docker-vs-proxmox-same-mini-pc/) — measured trade-offs, not opinion.
- [Proxmox on a mini PC](https://homelabtoolkit.com/build/proxmox-on-mini-pc-eqi12/) — the alternative hypervisor route.
- [Mini PC home server buying guide](https://homelabtoolkit.com/build/mini-pc-home-server-buying-guide/) — ports, NICs and NVMe choices that actually matter.
- [NAS vs mini PC running cost](https://homelabtoolkit.com/tools/nas-vs-mini-pc-cost/) — total cost over the service life.
- [Home Assistant capacity estimator](https://homelabtoolkit.com/tools/home-assistant-capacity/) — sizing if you plan to add Home Assistant.
- [Wake-on-LAN checklist generator](https://homelabtoolkit.com/tools/wol-checklist-generator/) — an ordered pre-deployment checklist for the verified 5/5 shutdown-WOL cycles this stack documents.
- [Windows home server port audit](https://homelabtoolkit.com/build/windows-home-server-port-audit/) — every listening TCP socket and firewall rule on the host that runs this stack, measured.
- [Port audit checklist generator](https://homelabtoolkit.com/tools/port-audit-checklist-generator/) — turn the audit method into an ordered checklist for your own host.

## Notes

- `restart: unless-stopped` is set on every service so the stack returns after a host reboot.
- Health checks are defined for every service so `docker compose ps` shows real readiness, not just "running".
- The Jellyfin `devices` block for `/dev/dri` (Intel QSV) is commented out — uncomment after confirming your platform exposes the iGPU.
- The `.env` file is gitignored. Never commit real passwords.

## License

MIT — see [LICENSE](LICENSE).
