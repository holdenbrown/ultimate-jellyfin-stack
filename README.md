# The Ultimate Jellyfin Stack!

Welcome to my Jellyfin stack repository! This repository showcases my Docker Compose setup for managing various media-related services using Docker containers. The compose file is meant to be changed to each users liking as I know not everyone has the same requirements. Hope you enjoy!

## Overview

This Plex Stack includes the following services:

- **Jellyfin:** Media server for streaming movies and TV shows.
- **Radarr:** Movie management and automation.
- **Sonarr:** TV show management and automation.
- **Readarr:** Used to grab books and audiobooks.
- **Lidarr:** Used to grab music.
- **Kapowarr** Used to grab comics.
- **Prowlarr:** Indexer manager for Radarr and Sonarr.
- **Jellyseerr:** Request management and monitoring for Jellyfin.
- **Gluetun:** VPN container with WireGuard support for secure browsing.
- **Qbittorrent:** BitTorrent client with VPN support.
- **Tdarr:** Pre-transcodes your media to decrease file sizes
- **Bazarr:** Subtitle management for movies and TV shows.
- **Autobrr:** Used to grab torrents immediately as they are released.
- **Flaresolverr:** Used as a proxy server to bypass Cloudflare and DDoS-GUARD protection.
- **Dozzle:** Used to view the logs of any container.
- **Wizarr:** Used to create links that can be sent to users so they can be invited to your media server.
- **Homarr:** Used as a dashboard for docker containers with integrations for the *arr, torrent, and Jellyfin apps.
- **Decluttarr:** Used to maintain/clean your *arr app queues and downloads.
- **Jellystat:** Used to monitor the usage of each user or content on your jellyfin server.

![containers](https://github.com/user-attachments/assets/855014b1-2716-4370-975d-a02564df881e)

## Dependencies

1. Linux
2. Docker / Docker Compose
3. OPTIONAL: Portainer - Docker GUI

## How to Use - Using portainer
1. Create a new stack using the Repository build method
2. Add this link `https://github.com/Wh1rr/ultimate-jellyfin-stack/blob/main/docker-compose.yml` as repository URL
3. Add your environment variables using the below examples.

## Example of Environment variables in Portainer
Keep in mind some variable names have changed since this screenshot was taken
![jellyfin-stack](https://github.com/user-attachments/assets/1b984a6f-df7f-46c9-9dd1-54c62b6854a6)

  
File location examples:
- {MEDIA_SHARE} = ~/Jellyfin/share
- {BASE_PATH} = ~/Jellyfin/home

To allow hardlinking to work (which you will definitely want!) you will have to use the same root folder in all of your container path. In this example we use "/share", so in the container it will look like "/share/downloads/tv"

An example folder structure:  
![image](https://github.com/DonMcD/ultimate-plex-stack/assets/90471623/2003ac26-a929-4ff6-ad67-e35fc51fb51a)
  
- Feel free to expand your folders to also include "books" or "music" as you need for your setup
  
## Starr apps
Setting up the starr apps might be a bit confusing the first time, but to keep it simple:
1. Prowlarr manages the indexes for *some* starr apps (radarr and sonarr)
2. The following starr apps make use of the download client (qBittorent) to download their respective content type:
   1. Radarr - Movies
   2. Sonarr - TV Shows
   3. Lidarr - Music
   4. Readarr - Books
   5. Kapowarr - Comics
3. Jellyseer is a platform which combines the request system in radarr and sonarr to make a nice UI/UX to find and auto download the content.
  
Anytime you reference your media folder in a container you want the path to look like /share/media/tv instead of /tv like a lot of the default guides say, if you do end up mapping the path as /tv hardlinking will not work

## Network layout

The Docker Compose file is the source of truth for container networking.

- qBittorrent is routed through Gluetun with `network_mode: "service:gluetun"`. This keeps torrent traffic behind the VPN.
- Radarr and Sonarr use normal Docker bridge networking for metadata lookups and app traffic. They use explicit DNS servers so metadata lookups do not depend on VPN or Tailscale DNS.
- Prowlarr and Flaresolverr use normal Docker bridge networking. This keeps indexer and Cloudflare-solver traffic reliable without exposing qBittorrent outside the VPN.
- Profilarr is disabled by default because this stack prefers native Radarr/Sonarr quality profiles and custom formats.

For AI-assisted profile generation, use [docs/arr-profile-prompt.md](docs/arr-profile-prompt.md). It contains the minimal system context needed to create practical 1080p H.265/HEVC profiles and optional 2160p profiles without making downloads too strict.

## Recommendations

1. Get familiar with reverse proxies
2. Install portainer to manage and monitor containers
3. Use Cloudflare tunnels

