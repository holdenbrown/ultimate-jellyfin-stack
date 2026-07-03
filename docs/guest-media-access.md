# Guest Media Access

This stack uses Jellyfin for playback and Jellyseerr for requests.

Guest workflow:

1. Open Jellyseerr / Media Requests.
2. Sign in with the guest's Jellyfin account.
3. Search for a movie or show.
4. Request either the normal 1080p option or the 4K option.
5. Watch approved/downloaded media in Jellyfin.

Jellyfin itself does not provide a native Radarr/Sonarr request queue with quality-profile selection. Jellyseerr is the request UI and is mapped to the native Arr profiles.

## Request Profile Mapping

Jellyseerr should expose:

- Movies, normal: Radarr `1080p HEVC Preferred`
- Movies, 4K: Radarr `2160p HEVC Preferred`
- Shows, normal: Sonarr `1080p HEVC Preferred`
- Shows, 4K: Sonarr `2160p HEVC Preferred`

The profiles prefer H.265/HEVC/x265 but keep the minimum custom format score at `0`, so x264/H.264 fallback releases can still download when an HEVC release is unavailable.

## Tailscale Security Model

Use Tailscale for private remote access only. Do not expose the Arr apps, Prowlarr, Portainer, qBittorrent, or Gluetun to guests.

Guest-accessible services:

- Jellyfin: TCP `8096`
- Jellyseerr / Media Requests: TCP `5055`

Operator-only services:

- Radarr: TCP `7878`
- Sonarr: TCP `8989`
- Prowlarr: TCP `9696`
- Portainer: TCP `9000` / `9443`
- qBittorrent: TCP `8081`

qBittorrent must remain behind Gluetun using `network_mode: "service:gluetun"`. Tailscale is for management and viewing access; Gluetun is still the VPN boundary for torrent traffic.

## Recommended Tailscale ACL

Prefer tagging the server as `tag:media-server`, then granting guests only Jellyfin and Jellyseerr.

```json
{
  "groups": {
    "group:media-guests": [
      "guest1@example.com",
      "guest2@example.com"
    ],
    "group:media-admins": [
      "admin@example.com"
    ]
  },
  "tagOwners": {
    "tag:media-server": [
      "group:media-admins"
    ]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["group:media-guests"],
      "dst": ["tag:media-server:8096,5055"]
    },
    {
      "action": "accept",
      "src": ["group:media-admins"],
      "dst": ["tag:media-server:*"]
    }
  ],
  "ssh": []
}
```

If you do not use tags, replace `tag:media-server` with the server's Tailscale IP after enrollment. Tags are cleaner because the policy survives IP/device churn.

## Server-Side Guardrail

The live server also has a host firewall guard for Tailscale traffic:

- `/usr/local/sbin/media-tailscale-firewall.sh`
- `media-tailscale-firewall.service`

The service allows inbound `tailscale0` traffic only to TCP `8096` and `5055`, then drops other inbound Tailscale traffic. This protects the stack even if the tailnet ACL is temporarily too broad.

This firewall does not change LAN access, Docker bridge traffic, or qBittorrent's Gluetun/VPN routing.
