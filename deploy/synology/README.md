# AgenticSeek on Synology DSM 7.1

This deployment is intended for Synology NAS systems using the legacy `docker-compose` v1 command. It keeps the AgenticSeek backend private on the Docker network and exposes only the frontend port to the LAN.

## Layout

- Frontend: Nginx + production React build, exposed on `${FRONTEND_PORT}` (default `3100`)
- Backend: private Docker-network service on port `7777`
- SearXNG: private Docker-network service on port `8080`
- Valkey/Redis: private Docker-network service on port `6379`
- Workspace: NAS bind mount configured by `WORKSPACE_DIR`

Browser requests to `/api/*` are proxied by Nginx to the backend, so backend port 7777 is not published on the NAS.

## First-time setup

From the repository root on the NAS:

```sh
cd deploy/synology
cp .env.example .env
cp config.ini.example config.ini
```

Edit `.env` and set:

- `NAS_HOST` to the NAS hostname or LAN IP
- `FRONTEND_PORT` to an unused NAS port (default `3100`)
- `WORKSPACE_DIR` to the desired persistent workspace directory
- `SEARXNG_SECRET_KEY` to a long random value
- `OPENROUTER_API_KEY` (or whichever provider key you use)

`config.ini` is intentionally untracked. Edit it to choose the runtime provider/model without changing the repository's root `config.ini`.

Create the workspace directory if necessary:

```sh
mkdir -p /volume1/docker/agenticseek/workspace
```

Validate the rendered Compose configuration:

```sh
sudo docker-compose config
```

Build and start:

```sh
sudo docker-compose build
sudo docker-compose up -d
```

Watch startup:

```sh
sudo docker-compose logs -f backend
```

AgenticSeek is ready when the backend startup finishes. Open:

```text
http://NAS_HOST:FRONTEND_PORT
```

For the default port this is `http://NAS_HOST:3100`.

## Useful commands

```sh
sudo docker-compose ps
sudo docker-compose logs --tail=100 backend
sudo docker-compose restart backend
sudo docker-compose down
```

Do not publish backend port 7777 directly to the LAN or Internet. AgenticSeek can execute shell commands within its workspace and should remain behind the frontend proxy and a trusted LAN/VPN boundary.
