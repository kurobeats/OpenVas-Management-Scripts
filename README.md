# OpenVAS / Greenbone Community Edition Management Scripts

Scripts for managing Greenbone Community Edition (formerly OpenVAS) on systemd-based Linux distributions.

## Overview

These scripts are compatible with **Greenbone Community Edition 22.4+**, which uses a modern architecture with the following components:

- **openvasd** - The Notus Scanner daemon for local security checks
- **ospd-openvas** - OSP (Open Scanner Protocol) wrapper for the OpenVAS Scanner
- **gvmd** - Greenbone Vulnerability Manager daemon (central management service)
- **gsad** - Greenbone Security Assistant daemon (web interface)
- **greenbone-feed-sync** - Unified feed synchronization tool

> **Note:** These scripts have been updated from the legacy OpenVAS 8/9 architecture. The old components (`openvassd`, `openvasmd`, `openvas-nvt-sync`, etc.) are no longer used in modern GCE installations.

## Requirements

- Linux distribution with systemd (Debian, Ubuntu, Fedora, CentOS, Kali, etc.)
- Greenbone Community Edition 22.4 or later installed
- Root or sudo access for setup operations

## Installation

1. Clone or download this repository:
   ```bash
   git clone <repository-url>
   cd OpenVas-Management-Scripts
   ```

2. Make scripts executable:
   ```bash
   chmod +x openvas-* gvm-check-setup
   ```

3. (Optional) Add to PATH:
   ```bash
   sudo cp openvas-* gvm-check-setup /usr/local/bin/
   ```

## Usage

### Initial Setup

Run the setup script to configure your system:

```bash
sudo ./openvas-setup
```

This will:
- Create the `gvm` user and group
- Configure Redis for OpenVAS
- Set up directory permissions
- Configure GPG for feed validation
- Set up PostgreSQL database
- Configure sudo for scanning
- Synchronize feeds (VTs, SCAP, CERT, GVMD data)
- Create an admin user
- Enable and start services

### Starting Services

```bash
sudo ./openvas-start
```

Starts all GCE services in the correct order:
1. openvasd (Notus Scanner)
2. ospd-openvas (OSP daemon)
3. gvmd (Vulnerability Manager)
4. gsad (Web interface)

### Stopping Services

```bash
sudo ./openvas-stop
```

Stops all services in reverse dependency order.

### Updating Feeds

```bash
sudo ./openvas-feed-update
```

Synchronizes all feed data from the Greenbone Community Feed:
- VT data (Vulnerability Tests / NASL scripts)
- SCAP data (CPE/CVE information)
- CERT data (DFN-CERT and CERT-Bund advisories)
- GVMD data (Scan configs, port lists, report formats)

> **Note:** Feed updates may take a while, especially the first time. The services will automatically load the new data.

### Checking Setup

```bash
sudo ./gvm-check-setup
```

Verifies that your installation is complete and ready to use. Checks:
- Required binaries are installed
- Redis configuration
- PostgreSQL database
- Directory permissions
- GPG setup for feed validation
- Services are running
- Feed data is present
- Users are configured

## Accessing the Web Interface

After starting services, access the Greenbone Security Assistant at:

```
http://127.0.0.1:9392
```

Default credentials (created by `openvas-setup`):
- Username: `admin`
- Password: Generated during setup (displayed in output)

## Architecture Changes

### Legacy (OpenVAS 8/9) vs Modern (GCE 22.4+)

| Legacy Component | Modern Replacement | Purpose |
|-----------------|-------------------|---------|
| `openvassd` | `openvas` + `ospd-openvas` | Scanner daemon |
| `openvasmd` | `gvmd` | Vulnerability manager |
| `openvasad` | *removed* | Administrator (functionality merged) |
| `gsad` | `gsad` | Web interface (updated) |
| `openvas-nvt-sync` | `greenbone-feed-sync` | Feed synchronization |
| `openvas-scapdata-sync` | `greenbone-feed-sync` | SCAP data sync |
| `openvas-certdata-sync` | `greenbone-feed-sync` | CERT data sync |
| `openvas-mkcert` | `gvm-manage-certs` | Certificate management |
| SQLite database | PostgreSQL | Database backend |

## Troubleshooting

### Services won't start

Check service status:
```bash
systemctl status openvasd
systemctl status ospd-openvas
systemctl status gvmd
systemctl status gsad
```

View logs:
```bash
tail -f /var/log/gvm/gvmd.log
tail -f /var/log/gvm/ospd-openvas.log
tail -f /var/log/gvm/gsad.log
```

### Feed sync fails

Ensure GPG is configured:
```bash
ls -la /etc/openvas/gnupg/
```

Run feed sync manually with verbose output:
```bash
sudo greenbone-feed-sync --verbose
```

### Database connection issues

Verify PostgreSQL is running:
```bash
sudo -u postgres pg_isready
```

Check database exists:
```bash
sudo -u postgres psql -l | grep gvmd
```

### Redis connection issues

Verify Redis socket exists:
```bash
ls -la /run/redis-openvas/redis.sock
```

Test Redis connection:
```bash
redis-cli -s /run/redis-openvas/redis.sock ping
```

## File Descriptions

| File | Description |
|------|-------------|
| `openvas-setup` | Initial system setup and configuration |
| `openvas-start` | Start all GCE services |
| `openvas-stop` | Stop all GCE services |
| `openvas-feed-update` | Update all feed data |
| `gvm-check-setup` | Verify installation completeness |
| `redis.conf` | Redis configuration template for OpenVAS |

## License

These scripts are released under the GNU General Public License v3.0. See LICENSE file for details.

## Resources

- [Greenbone Community Documentation](https://greenbone.github.io/docs/latest/)
- [Greenbone Community Forum](https://forum.greenbone.net/)
- [OpenVAS Scanner Repository](https://github.com/greenbone/openvas-scanner)
- [GVM Daemon Repository](https://github.com/greenbone/gvmd)

## Contributing

Contributions are welcome! Please ensure your changes are compatible with the latest Greenbone Community Edition release.
