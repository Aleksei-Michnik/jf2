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

5. Access Jellyfin at `http://localhost:8096` or `http://<host-ip>:8096`

6. Complete the Setup Wizard (see [Initial Setup](#initial-setup) below)

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

#### Setting Up Media Path Symlinks

Docker Desktop for WSL2 requires paths to be accessible from the WSL home directory for reliable volume mounting. Create symlinks from your home directory to Windows drive paths:

```bash
# Create symlinks for Films
ln -s /mnt/h/Films ~/Films          # For H:\Films
ln -s /mnt/e/Films ~/Films_E        # For E:\Films

# Create symlinks for TV Shows
ln -s /mnt/c/Users/Lenovo/Shows ~/Shows_C   # For C:\Users\Lenovo\Shows
ln -s /mnt/e/Shows ~/Shows_E        # For E:\Shows
ln -s /mnt/h/Shows ~/Shows_H        # For H:\Shows
```

Then mount these symlinks in `docker-compose.yml`:
```yaml
volumes:
  - /home/yourusername/Films:/media/films:ro
  - /home/yourusername/Films_E:/media/films_e:ro
  - /home/yourusername/Shows_C:/media/shows_c:ro
  - /home/yourusername/Shows_E:/media/shows_e:ro
  - /home/yourusername/Shows_H:/media/shows_h:ro
```

After updating docker-compose.yml, restart the container:
```bash
docker compose down && docker compose up -d
```

Verify mounts are accessible inside the container:
```bash
docker exec jf2 ls -la /media/
```

### Volume Mounts

The following volumes are automatically configured:
- `/config`: Jellyfin server configuration and database
- `/cache`: Transcoding cache and temporary files

## Initial Setup

After starting the container for the first time, you need to complete the Jellyfin Setup Wizard.

### Step 1: Access the Setup Wizard

Open your browser and navigate to:
- **Local**: `http://localhost:8096`
- **Network**: `http://<your-ip>:8096` (find IP with `hostname -I | awk '{print $1}'`)

If you see a "Select Server" screen with old entries:
1. Delete any stale server entries
2. Click "Add Server"
3. Enter `http://localhost:8096` and click "Connect"

If you see a login page but have no credentials, go directly to:
- `http://localhost:8096/web/#/wizardstart.html`

### Step 2: Setup Wizard Steps

1. **Select Language** - Choose your preferred display language

2. **Create Admin User** - Set your username and password (remember these!)

3. **Add Media Libraries**:
   - Click "Add Media Library"
   - Select content type (e.g., "Movies", "TV Shows", "Music")
   - Click the "+" button to add a folder
   - Select from available paths (your mounted media folders):
     - `/media/films` - Movies from your Films folder
     - Add more paths as configured in `docker-compose.yml`
   - Configure metadata settings (language, country)
   - Click "OK" to save

4. **Preferred Metadata Language** - Select language for movie/show metadata

5. **Remote Access** - Configure if you want to access from outside your network
   - Enable "Allow remote connections to this server" for network access
   - Keep "Enable automatic port mapping" unchecked for local-only access

6. **Complete Setup** - Click "Finish" to complete the wizard

### Step 3: Post-Setup Configuration

After completing the wizard:

1. **Scan Libraries**: Go to Dashboard → Libraries → click "Scan All Libraries"
2. **Configure Users**: Add additional users in Dashboard → Users
3. **Set up Transcoding**: Configure hardware acceleration in Dashboard → Playback → Transcoding
4. **Install Plugins**: Browse available plugins in Dashboard → Plugins → Catalog

### Browser Cache Issues

If you encounter connection errors after restarting the container:
1. Clear your browser's local storage for the Jellyfin site
2. Or open in an incognito/private window
3. Or manually delete stale server entries and re-add `http://localhost:8096`

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

### Server Inaccessible After Host Restart (Docker Port Binding Issue)

**Symptoms:**
- Container shows as "running" and "healthy" in `docker ps`
- Connection refused when accessing `http://localhost:8096` or `http://<host-ip>:8096`
- Samsung TV shows server in list but cannot connect
- Browser shows "Unable to connect to the selected server"

**Root Cause:**
After a Windows/WSL2 host restart, Docker's port forwarding (docker-proxy) may not properly re-establish even though the container appears healthy. The container is running internally but the port bindings to the host are broken.

**Diagnosis:**
```bash
# Check if container is running
docker ps -a --filter "name=jf2"
# Shows: Up X minutes (healthy)

# Check if port is actually listening
ss -tlnp | grep 8096
# If empty - port is NOT listening despite container being "healthy"

# Check docker-proxy processes
ps aux | grep docker-proxy
# If no docker-proxy for port 8096 - port forwarding is broken
```

**Solution:**
Simply restart the container to re-establish port bindings:

```bash
# Restart the container
docker compose restart

# Wait for startup (10-15 seconds)
sleep 10

# Verify port is now listening
ss -tlnp | grep 8096
# Should show: LISTEN 0 4096 *:8096 *:*

# Verify server is accessible
curl -s http://localhost:8096/System/Info/Public | jq .
```

**Prevention:**
Add a startup script or systemd service to restart Docker containers after WSL2 starts:

```bash
# Add to ~/.bashrc or create a startup script
if ! ss -tlnp | grep -q ":8096"; then
    echo "Jellyfin port not listening, restarting container..."
    cd ~/jf2 && docker compose restart
fi
```

**Key Insight:**
Docker's health check only verifies the container's internal state, not the host port bindings. A container can be "healthy" internally while its ports are unreachable from the host.

## Issues Handling

### Issue: Samsung TV Jellyfin App Playback Error

**Symptoms:**
- Web interface at `http://192.168.1.173:8096` plays films correctly
- Samsung TV Jellyfin app (installed via Tizen) shows library and can browse content
- Playback fails with error: "Playback Error: There was an error processing the request"
- TV app was installed using: `docker run --rm ghcr.io/georift/install-jellyfin-tizen 192.168.1.152`

**Environment:**
- Jellyfin server running in Docker on WSL2
- Windows host IP: `192.168.1.173`
- Samsung TV IP: `192.168.1.152`
- Docker container internal IP: `172.19.0.2`

#### Investigation Method

**Step 1: Log Analysis**
```bash
docker logs jf2 --tail 100 | grep -i -E "(error|fail|transcode|samsung|tizen|playback)"
```
- Found transcoding was working correctly for web playback
- No Samsung/Tizen specific errors in logs
- FFmpeg commands executing successfully

**Step 2: Configuration Review**
Examined key configuration files:
- `config/config/network.xml` - Network settings
- `config/config/encoding.xml` - Transcoding settings
- `config/config/system.xml` - Server settings

**Step 3: Network Architecture Analysis**
```bash
# Check Docker container IP
docker exec jf2 hostname -I
# Output: 172.19.0.2

# Check Windows host IPs
powershell.exe -Command "ipconfig" | grep "IPv4"
# Found: 192.168.1.173 (LAN), 172.19.0.1 (Docker), etc.
```

**Step 4: Root Cause Identification**
The issue was in `config/config/network.xml`:
- `EnablePublishedServerUriByRequest` was `false`
- `PublishedServerUriBySubnet` was empty
- `EnableUPnP` was `false`

#### Diagnosis

**Root Cause:** Docker NAT URL Translation Issue

When the Samsung TV requests media playback:
1. TV connects to `192.168.1.173:8096` ✓ (browsing works)
2. TV requests media stream
3. Jellyfin returns stream URLs with internal Docker IP `172.19.0.2:8096`
4. TV cannot reach `172.19.0.2` ✗ → **Playback Error**

The web browser works because:
- It runs on the same machine or network segment
- Docker port forwarding handles the translation transparently
- Browser requests go through `localhost` or the host IP directly

The Samsung TV fails because:
- It's on a different device (192.168.1.152)
- It receives URLs pointing to unreachable Docker internal network
- No NAT translation occurs for the returned stream URLs

#### Solution

**Modified `config/config/network.xml`:**

1. **Published Server URI** (Critical Fix):
```xml
<EnablePublishedServerUriByRequest>true</EnablePublishedServerUriByRequest>
<PublishedServerUriBySubnet>
  <string>192.168.1.0/24=http://192.168.1.173:8096</string>
</PublishedServerUriBySubnet>
```
This tells Jellyfin to return `http://192.168.1.173:8096` for all clients on the 192.168.1.x network.

2. **Local Network Subnets**:
```xml
<LocalNetworkSubnets>
  <string>192.168.1.0/24</string>
</LocalNetworkSubnets>
```

3. **Enable UPnP** (for better device discovery):
```xml
<EnableUPnP>true</EnableUPnP>
```

**Application Steps:**
```bash
# Stop container to prevent config overwrite
docker stop jf2

# Fix file permissions (owned by root due to PUID=0)
sudo chmod 666 config/config/network.xml

# Edit the file with the changes above
# (or use the web UI: Dashboard → Networking)

# Restart container
docker start jf2

# Verify configuration loaded
docker logs jf2 --tail 30 | grep -i subnet
# Should show: Used LAN subnets: ["192.168.1.0/24"]
```

#### Verification

After applying the fix:
```bash
# Check config persisted
grep -A1 "PublishedServerUriBySubnet" config/config/network.xml
# Should show: 192.168.1.0/24=http://192.168.1.173:8096

# Check logs for proper initialization
docker logs jf2 --tail 30 | grep -i -E "(subnet|publish|upnp)"
# Should show:
# - Defined LAN subnets: ["192.168.1.0/24"]
# - Used LAN subnets: ["192.168.1.0/24"]
# - Starting NAT discovery
```

**Test from Samsung TV:**
1. Open Jellyfin app
2. Select any movie
3. Press Play
4. Playback should now work

#### If Issue Persists

1. **Clear TV app cache**: See [Clearing App Cache on Samsung TV](#clearing-app-cache-on-samsung-tv-ru7172-and-similar-2019-models) below
2. **Re-login on TV**: Sign out and sign back in to refresh session
3. **Check Windows Firewall**: Ensure port 8096 is allowed from 192.168.1.152
4. **Verify IP hasn't changed**: Re-run `powershell.exe -Command "ipconfig"` to confirm 192.168.1.173

#### Key Learnings

1. **Docker NAT Limitation**: Containers don't know their external IP; they return internal Docker network IPs in API responses
2. **Published Server URI**: Essential for any Docker/NAT setup where clients connect from different network segments
3. **Config File Permissions**: Jellyfin runs as root (PUID=0) in this setup, so config files are owned by root
4. **Config Persistence**: Jellyfin may rewrite config files on startup; stop container before editing

---

### Issue: Samsung TV Jellyfin App Version Mismatch Playback Error

**Symptoms:**
- Samsung TV Jellyfin app shows library content correctly (can browse movies/shows)
- Pressing Play results in error: "Playback Error: There was an error processing the request"
- Web browser playback works fine
- No errors in Jellyfin server logs when playback fails on TV
- TV was previously working but stopped after server update

**Reference:** GitHub Issues [jellyfin-tizen#343](https://github.com/jellyfin/jellyfin-tizen/issues/343) and [jellyfin-tizen#359](https://github.com/jellyfin/jellyfin-tizen/issues/359)

#### Root Cause

**Version mismatch** between the Samsung TV Tizen app and the Jellyfin server. The Tizen app is built with a specific version of jellyfin-web that must match (or be compatible with) the server version.

Example:
- Server version: 10.11.5
- TV app built with: jellyfin-web 10.10.z (older, incompatible)

#### Solution: Reinstall Samsung TV Jellyfin App with Matching Version

**Step 1: Check Your Server Version**
```bash
curl -s http://localhost:8096/System/Info/Public | grep -oP '"Version":"\K[^"]+'
# Example output: 10.11.5
```

**Step 2: Install Matching TV App Version**

Using the `georift/install-jellyfin-tizen` tool, reinstall with the correct version:

```bash
# For server version 10.11.x (recommended for 10.11.5):
docker run --rm ghcr.io/georift/install-jellyfin-tizen 192.168.1.152 Jellyfin-10.11.z

# For server version 10.10.x:
docker run --rm ghcr.io/georift/install-jellyfin-tizen 192.168.1.152 Jellyfin-10.10.z

# Default (uses latest stable jellyfin-web):
docker run --rm ghcr.io/georift/install-jellyfin-tizen 192.168.1.152
```

Replace `192.168.1.152` with your Samsung TV's IP address.

**Available build variants:**
- `Jellyfin` - Default build (latest stable)
- `Jellyfin-10.11.z` - For server 10.11.x
- `Jellyfin-10.10.z` - For server 10.10.x
- `Jellyfin-TrueHD` - With TrueHD audio support
- `Jellyfin-secondary` - Secondary installation (different app ID)

See all available builds at: https://github.com/jeppevinkel/jellyfin-tizen-builds/releases

**Step 3: Clear TV App Cache**

After reinstalling, clear the app cache on the TV (see section below).

**Step 4: Re-login on TV**

Open the Jellyfin app on the TV and sign in again with your credentials.

#### Samsung TV App Settings

| Setting | Value |
|---------|-------|
| **Server Address** | `192.168.1.173` |
| **Port** | `8096` |
| **Full URL** | `http://192.168.1.173:8096` |

---

### Clearing App Cache on Samsung TV (RU7172 and Similar 2019+ Models)

For Samsung TVs from 2019 onwards (RU series, TU series, etc.) that don't have an "Apps" menu in main settings, use **Device Care** instead:

#### Method 1: Via Device Care (Recommended for RU7172)

1. **Press Home** button on remote
2. **Navigate to Settings** (gear icon)
3. **Go to Support**
4. **Select Self Diagnostics**
5. **Choose TV Device Manager**
6. **Select Show App List** under **Manage Storage**
6. **Find and select Jellyfin** in the app list
7. **Click View Details**
8. **Select Clear Cache** to remove temporary files
9. *(Optional)* Select **Clear Data** to reset the app completely (will require re-login)

#### Method 2: Via Apps Panel

1. **Press Home** button on remote
2. **Navigate to Apps** (bottom row of icons)
3. **Go to Settings** (gear icon in top-right of Apps panel)
4. **Find Jellyfin** in the installed apps list
5. **Select it and choose Delete** or look for cache options

#### Method 3: Reinstall the App

If cache clearing doesn't help, completely reinstall the app:

```bash
# The install command will automatically replace/update the existing app
docker run --rm ghcr.io/georift/install-jellyfin-tizen 192.168.1.152 Jellyfin-10.11.z
```

#### Method 4: Cold Restart the TV

Sometimes a simple cache clear isn't enough:

1. **Turn off** the TV completely (not standby)
2. **Unplug** the power cord for 30 seconds
3. **Plug back in** and turn on
4. Try the Jellyfin app again

#### Tips

- **Frequency**: Clear cache every 2-3 months for optimal performance
- **After Updates**: Always clear cache after updating either the server or TV app
- **Version Sync**: Keep server and TV app versions aligned (both 10.11.x or both 10.10.x)

---

## License

This project is provided as-is for personal use.

## Contributing

Feel free to submit issues and pull requests for improvements.
