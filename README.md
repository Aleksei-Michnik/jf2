# jf2 - Dockerized Jellyfin Media Server

A Dockerized Jellyfin media server setup designed for local network access in a WSL (Windows Subsystem for Linux) environment.

## Overview

This project provides a containerized Jellyfin media server configuration that:
- Runs Jellyfin in a Docker container for easy deployment and management
- Supports multiple media locations per category (movies, TV shows, music, etc.)
- Mounts media volumes from the Windows host machine via WSL
- Provides web interface access on the local network
- Offers a reproducible and portable media server setup

## Features

- **Docker-based deployment**: Easy setup and teardown with Docker Compose
- **WSL Integration**: Seamless volume mounting from Windows host drives
- **Multiple Media Sources**: Support for media spread across multiple drives/locations
- **Local Network Access**: Web interface accessible from any device on the network
- **Persistent Configuration**: Server settings and metadata stored in mounted volumes

## Prerequisites

- Docker and Docker Compose installed
- WSL2 environment (for Windows hosts)
- Network access to the host machine

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/jf2.git
   cd jf2
   ```

2. Copy the environment template:
   ```bash
   cp .env.example .env
   ```

3. Configure your media paths in `docker-compose.yml` (see [Media Configuration](#media-configuration))

4. Start the container:
   ```bash
   docker compose up -d
   ```

5. Access Jellyfin at `http://<host-ip>:8096`

## Project Structure

```
jf2/
├── docker-compose.yml    # Docker Compose configuration
├── config/               # Jellyfin configuration (mounted volume)
├── cache/                # Jellyfin cache (mounted volume)
├── .env.example          # Environment variables template
├── .env                  # Your local environment (git-ignored)
└── README.md             # This file
```

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and adjust the values:

| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | 1000 | User ID for file permissions |
| `PGID` | 1000 | Group ID for file permissions |
| `TZ` | UTC | Timezone |
| `JELLYFIN_PORT` | 8096 | Web interface port |
| `JELLYFIN_HTTPS_PORT` | 8920 | HTTPS port (optional) |

### Media Configuration

Media paths are configured directly in `docker-compose.yml` under the `volumes` section. This allows maximum flexibility for multiple media locations.

#### Adding Media Paths

Edit `docker-compose.yml` and add volume mounts in this format:
```yaml
volumes:
  - /host/path:/media/library_name:ro
```

The `:ro` suffix makes the mount read-only (recommended for media files).

#### Example: Multiple Drives

```yaml
volumes:
  # Movies from different drives
  - /mnt/d/Movies:/media/movies_d:ro
  - /mnt/e/Movies:/media/movies_e:ro
  - /mnt/f/Movies/4K:/media/movies_4k:ro
  
  # TV Shows from multiple locations
  - /mnt/d/TVShows:/media/tvshows_d:ro
  - /mnt/e/Series:/media/tvshows_e:ro
  
  # Music collections
  - /mnt/d/Music:/media/music_main:ro
  - /mnt/e/Music/FLAC:/media/music_lossless:ro
```

#### WSL Path Format

In WSL, Windows drives are mounted under `/mnt/`:
- `C:\Users\Name\Videos` → `/mnt/c/Users/Name/Videos`
- `D:\Media\Movies` → `/mnt/d/Media/Movies`

### Volume Mounts

The following volumes are automatically configured:
- `/config`: Jellyfin server configuration and database
- `/cache`: Transcoding cache and temporary files

## Usage

### Starting the Server
```bash
docker compose up -d
```

### Stopping the Server
```bash
docker compose down
```

### Viewing Logs
```bash
docker compose logs -f jellyfin
```

### Updating Jellyfin
```bash
docker compose pull
docker compose up -d
```

### Restarting After Config Changes
```bash
docker compose down
docker compose up -d
```

## Network Access

Once running, Jellyfin will be accessible at:
- **Local**: `http://localhost:8096`
- **Network**: `http://<your-ip>:8096`

To find your IP address in WSL:
```bash
hostname -I | awk '{print $1}'
```

## Hardware Acceleration

For hardware transcoding, uncomment the relevant sections in `docker-compose.yml`:

### Intel/AMD GPU
```yaml
devices:
  - /dev/dri:/dev/dri
```

### NVIDIA GPU
```yaml
runtime: nvidia
environment:
  - NVIDIA_VISIBLE_DEVICES=all
```

## Troubleshooting

### Permission Issues
Ensure `PUID` and `PGID` in `.env` match your user:
```bash
id -u  # Your user ID
id -g  # Your group ID
```

### Media Not Showing
1. Check volume mounts are correct in `docker-compose.yml`
2. Verify paths exist: `ls /mnt/d/your/path`
3. Check container logs: `docker compose logs jellyfin`

### Container Won't Start
```bash
docker compose logs jellyfin
docker compose down
docker compose up -d
```

## License

This project is provided as-is for personal use.

## Contributing

Feel free to submit issues and pull requests for improvements.
