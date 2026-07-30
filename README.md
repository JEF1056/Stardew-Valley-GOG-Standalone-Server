# Stardew Valley GOG Standalone Dedicated Server

A lightweight, standalone Docker setup for hosting a **Stardew Valley** dedicated server using local **GOG Linux** game files without requiring Steam credentials, Steam authentication sidecars, or online lobbies.

---

## Features

- 🎮 **Offline & LAN Multiplayer**: Host multiplayer games directly over LAN or IP address without needing Steam login or GOG Galaxy online authentication.
- 🚀 **Standalone Docker Setup**: No `steam-auth` or `discord-bot` containers needed. Runs purely as a single container.
- 🖥️ **Web Admin GUI & REST API**: Control your server via a built-in web interface (`http://localhost:5800`) or REST API (`http://localhost:8080`).
- ⚡ **SMAPI & Mod Support**: Pre-configured with SMAPI 4.5.2 and JunimoServer host automation.
- 🍎 **Apple Silicon & x86_64 Compatible**: Pre-configured with `platform: linux/amd64` to run smoothly on macOS (Apple Silicon via Rosetta 2) and Linux/Windows.

---

## Prerequisites

1. **Docker**: Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) (macOS / Windows) or [Docker Engine](https://docs.docker.com/engine/install/) (Linux).
2. **GOG Linux Installer**: Stardew Valley GOG Linux installer script (e.g. `stardew_valley_1_6_15_*.sh`).

---

## Quick Start

### 1. Extract GOG Linux Game Files

Extract the `data/noarch/game/` folder from your GOG Linux installer `.sh` file and copy it into the Docker volume `stardew-valley-gog-standalone_game-data`:

```bash
# Create the volume
docker volume create stardew-valley-gog-standalone_game-data

# Extract GOG installer payload
unzip -q "/path/to/stardew_valley_1_6_15_*.sh" "data/noarch/game/*" -d /tmp/gog_game

# Copy game files into the Docker volume
docker run --rm \
  -v stardew-valley-gog-standalone_game-data:/data/game \
  -v /tmp/gog_game/data/noarch/game:/source \
  alpine sh -c "cp -rp /source/* /data/game/ && chmod -R 777 /data/game"

# Clean up temporary extraction folder
rm -rf /tmp/gog_game
```

### 2. Add Steamworks.NET DLL (For GOG Compatibility)

Download `Steamworks.NET` 20.0.0 and place `Steamworks.NET.dll` into the game volume:

```bash
curl -sL https://github.com/rlabrecque/Steamworks.NET/releases/download/20.0.0/Steamworks.NET-Standalone_20.0.0.zip -o /tmp/Steamworks.zip
unzip -p /tmp/Steamworks.zip OSX-Linux-x64/Steamworks.NET.dll > /tmp/Steamworks.NET.dll

docker run --rm \
  -v stardew-valley-gog-standalone_game-data:/data/game \
  -v /tmp/Steamworks.NET.dll:/tmp/Steamworks.NET.dll \
  alpine cp /tmp/Steamworks.NET.dll /data/game/Steamworks.NET.dll

rm -f /tmp/Steamworks.zip /tmp/Steamworks.NET.dll
```

### 3. Configure Environment & Server Settings

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Ensure `.env` contains:

```env
IMAGE_VERSION=preview
VNC_PASSWORD="admin"
ALLOW_INSECURE_SETUP=true
```

Ensure `.local-container/settings/server-settings.json` has IP connections enabled:

```json
{
  "Server": {
    "AllowIpConnections": true
  }
}
```

### 4. Start the Server

Start the container in background mode:

```bash
docker compose up -d
```

View server startup logs:

```bash
docker compose logs -f
```

---

## How Players Connect

1. Launch Stardew Valley on client machines.
2. Select **Co-op** -> **Join**.
3. Choose **Join LAN Game...** (or **Join via IP**).
4. Enter the host server's IP address (or `localhost` if running locally).

---

## Web Interfaces & Ports

- **VNC Web Administration UI**: `http://localhost:5800` (Password: `admin`)
- **REST API**: `http://localhost:8080/status`
- **Game Server UDP Ports**: `24642` and `27015`

---

## Managing the Server

- **View Logs**: `docker compose logs -f`
- **Restart Server**: `docker compose restart`
- **Stop Server**: `docker compose down`
