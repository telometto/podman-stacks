# Docker Compose Services: WireGuard, Firefox, and qBittorrent

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
  - [Software Requirements](#software-requirements)
  - [Environment Variables](#environment-variables)
- [Service-Specific Instructions](#service-specific-instructions)
  - [WireGuard](#wireguard)
  - [Firefox](#firefox)
  - [qBittorrent](#qbittorrent)
- [Setup Instructions](#setup-instructions)
- [Important Ports](#important-ports)
- [Persistent Data](#persistent-data)
- [Other notes](#other-notes)

## Overview

This repository contains a `docker-compose.yml` file that sets up the following services:

- **WireGuard**: A VPN service.
- **Firefox**: The Firefox web browser.
- **qBittorrent**: The qBittorrent client.

Each of these services has been configured to operate in a specific manner as delineated in the `docker-compose.yml` file.

## Prerequisites

### Software Requirements

Ensure that either **Option #1** or **Option #2** are installed on your system before proceeding.

- **Option #1**: Podman and Podman Compose
- **Option #2**: Docker and Docker Compose

### Environment Variables

The compose file relies on the following environment variables which **must** be set:

- `TZ`: The time zone.
- `QBIT_DATA_PATH`: The download directory for qBittorrent.

#### *Note*

For those unfamiliar with `.env` files, see the example below. Create and place a `.env` file in the same directory as your `docker-compose.yml` to securely load environment variables.

```env
# example .env contents
TZ=Europe/Vienna
QBIT_DATA_PATH=/path/to/your/torrent/files
```

## Service-Specific Instructions

### WireGuard

- **Privileged Mode**: Required to operate.
- **sysctl Flag**: `net.ipv4.conf.all.src_valid_mark=1` is required if WireGuard is run as a client.
  
### Firefox

- **Network Mode**: Must be set to `network_mode: "service:wireguard"` to route traffic through the WireGuard container.
- **Shared Memory**: `shm_size: "1gb"` is allocated for adequate shared memory.
- **Hardware Acceleration**: `DRINODE` and `devices` are required to use HW accelerated graphics from a GPU.

### qBittorrent

- **Network Mode**: Similar to Firefox, must be set to `network_mode: "service:wireguard"`.

## Setup Instructions

### IMPORTANT
If you're running an SELinux-enabled distro, you will need to add `:z` or `:Z` at the end of the path(s).
If you are *not* running an SELinux-enabled distro, you can remove them if you want. Or leave them; it doesn't matter, since the flags will get ignored anyway.

### Manual Start

1. Clone the repository.
2. Navigate to the directory containing `docker-compose.yml`.
3. Execute `podman-compose up -d` or `docker-compose up -d` to initiate the services in detached mode.

### Auto-Start Services

#### Docker
To automatically restart your containers when they exit or the Docker daemon restarts, you can uncomment the `restart` directive(s) in the `docker-compose.yml` file under each service, like this:

```yaml
  wireguard:
    ...
    restart: unless-stopped
  firefox:
    ...
    restart: unless-stopped
  qbittorrent:
    ...
    restart: unless-stopped
```

#### Podman
For Podman, auto-restart isn't supported natively in the same way as Docker. However, you can use `systemd` services to achieve similar functionality.

Be sure to replace anything within the brackets (`< >`) - *and remove the brackets themselves, too* - with the appropriate info.

Create a `systemd` service file, for example:

```bash
# enable the linger service if it isn't already
sudo loginctl enable-linger <your-user>

# deploy the stack
podman-compose up -d

# check that the container is running properly
podman ps # gets running containers

# stop the container(s)
podman stop -a

# automatically generate the systemd file(s)
# NB! this will generate the files inside the current directory!
podman generate systemd --new --files --name wireguard
podman generate systemd --new --files --name qbittorrent
podman generate systemd --new --files --name firefox

ls -l *.service # ensure the files have been created

cat container-<service_name>.service # if you want to look at the contents

# remove the containers
# this command might have to be run twice; be sure to check the output after the first run
podman system prune # press 'y' when prompted

# create a dir for user systemd files
mkdir -p $HOME/.config/systemd/user

# copy the service files over
# NB! if you're using SELinux, you need to also pass the '-Z' flag!
sudo mv -v container-wireguard.service /etc/systemd/system/
mv -v container-*.service $HOME/.config/systemd/user/

# restart the systemd daemon
sudo systemctl daemon-reload
systemctl --user daemon-reload

# start the service(s)
sudo systemctl start container-wireguard.service
systemctl --user start container-firefox.service
systemctl --user start container-qbittorrent.service

# check their status
sudo systemctl status container-wireguard.service
systemctl --user status container-firefox.service
systemctl --user status container-qbittorrent.service

# if everything seems to be running properly, enable the services at startup
sudo systemctl enable container-wireguard.service
systemctl --user enable container-firefox.service
systemctl --user enable container-qbittorrent.service
```

## Important Ports

- WireGuard: Utilizes the qBittorrent and Firefox ports for its operation.
- Firefox: Omits port configurations when using `network_mode`.
- qBittorrent: Also omits port configurations when using `network_mode`.

## Persistent Data

Persistent data for each service is stored in a designated volume under `./<service_name>/config`.

## Other notes
If you are experiencing 'Unauthorized access' when trying to access the qBittorrent web UI, you need to navigate to where the qBittorrent config files live and edit the `qBittorrent.conf` file.
If the line exists, and says `WebUI\HostHeaderValidation=true`, change it to `WebUI\HostHeaderValidation=false`; if not, just add the line yourself.