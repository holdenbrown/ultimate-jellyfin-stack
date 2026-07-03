# Radarr/Sonarr Profile Prompt

Use this prompt when asking an AI assistant to create or update native Radarr and Sonarr quality profiles and custom formats for this stack.

```text
You are configuring native Radarr and Sonarr quality profiles for a Jellyfin media stack with a request workflow.

System context:
- Docker Compose is the source of truth.
- qBittorrent is the only active service that must share Gluetun's network namespace, using `network_mode: "service:gluetun"`.
- Radarr and Sonarr are normal bridge-network containers on `starr`, `proxy`, and `gluetun_network`; their metadata/API traffic should not depend on Gluetun DNS.
- Prowlarr manages indexers for Radarr and Sonarr and connects to qBittorrent through the stack network.
- Jellyseerr is the user-facing request/search layer for Jellyfin users. The intended experience is that a user can search for a movie or show and request it in either a 1080p HEVC profile or a 2160p/4K HEVC profile.
- IPTorrents is the preferred tracker/indexer and should be prioritized through Prowlarr/Radarr/Sonarr settings where practical. Other configured indexers may remain as fallback so requests do not fail when IPTorrents has no suitable release.
- Profilarr is disabled by default. Prefer native Radarr/Sonarr quality profiles and custom formats unless explicitly asked to sync external profile packs.

Goal:
- Create a simple request-ready system where Jellyfin/Jellyseerr users can choose between `1080p HEVC Preferred` and `2160p HEVC Preferred`.
- Prefer H.265/HEVC releases for both formats without making H.265 mandatory.
- Favor good 1080p HEVC releases for normal use.
- Provide a 2160p/4K HEVC option for movies and shows where 4K is wanted.
- Prioritize IPTorrents results when available, but keep fallback indexers/results eligible so requests can still complete.
- Avoid profiles so strict that downloads fail when a suitable HEVC release is unavailable.
- Prioritize indexer/download reliability and tracker/IP tolerance over aggressive quality scoring. Do not create rules that unnecessarily block common legitimate releases.
- Make the profile names clean and user-facing so they are understandable in request tools.

Radarr profiles:
1. `1080p HEVC Preferred`
   - Allowed qualities: 1080p WEBDL, 1080p WEBRip, 1080p Bluray.
   - Optional fallback qualities: 720p WEBDL/WEBRip/Bluray if availability matters more than strict 1080p.
   - Prefer HEVC/x265/H.265 with a positive custom format score.
   - Allow AVC/x264/H.264 fallback with neutral score.
   - Do not require HEVC by setting a positive minimum custom format score.
   - Avoid Remux unless storage use is intentionally high.
   - This should be the default movie request profile in Jellyseerr.

2. `2160p HEVC Preferred`
   - Allowed qualities: 2160p WEBDL, 2160p WEBRip, 2160p Bluray.
   - Prefer HEVC/x265/H.265.
   - Prefer HDR10/HDR10+ when useful, but avoid hard-blocking SDR fallback.
   - Avoid Dolby Vision-only releases unless there is HDR10 fallback compatibility.
   - Avoid Remux by default unless explicitly requested.
   - This should be exposed as the 4K movie request option in Jellyseerr.

Sonarr profiles:
1. `1080p HEVC Preferred`
   - Allowed qualities: 1080p WEBDL, 1080p WEBRip, 1080p Bluray.
   - Prefer HEVC/x265/H.265 with a positive custom format score.
   - Keep x264/H.264 as fallback.
   - Prefer season packs only when they do not reduce quality or delay availability too much.
   - This should be the default show request profile in Jellyseerr.

2. `2160p HEVC Preferred`
   - Allowed qualities: 2160p WEBDL, 2160p WEBRip, 2160p Bluray.
   - Prefer HEVC/x265/H.265.
   - Prefer HDR10/HDR10+ with fallback-safe scoring.
   - Avoid Dolby Vision-only releases without HDR10 fallback.
   - This should be exposed as the 4K show request option in Jellyseerr.

Custom format guidance:
- Use one simple HEVC preference custom format matching release titles containing `x265`, `h265`, or `HEVC`.
- Score HEVC around +100 for 1080p profiles.
- Score HEVC around +100 to +200 for 2160p profiles.
- Keep `minimum custom format score` at 0 so non-HEVC releases can still download.
- Add small positive scores for trusted sources/groups only if already known to work well with the configured indexers.
- Add negative scores only for clearly unwanted releases such as CAM/TS, 3D, upscaled, VVC if unsupported, or Dolby Vision without HDR10 fallback.

Indexer/request guidance:
- Prefer IPTorrents via Prowlarr indexer priority or equivalent Radarr/Sonarr indexer priority settings.
- Do not hard-code IPTorrents as the only allowed indexer unless explicitly asked; keep other configured indexers as fallback.
- If using release profiles, tags, or indexer flags, make them soft preferences rather than hard blocks unless the release is clearly unwanted.
- Ensure Jellyseerr can present both 1080p and 4K request choices for movies and shows, mapped to the matching Radarr/Sonarr quality profiles.

Output requested:
- Exact Radarr profile names, allowed qualities, cutoff, custom formats, and scores.
- Exact Sonarr profile names, allowed qualities, cutoff, custom formats, and scores.
- Which Jellyseerr request profiles/settings should map to each Radarr/Sonarr profile.
- How to prioritize IPTorrents while preserving fallback indexer behavior.
- A short explanation of why the scoring prefers HEVC but still allows fallback downloads.
- Any assumptions about Jellyfin client compatibility for 4K, HDR, Dolby Vision, and audio codecs.
```
