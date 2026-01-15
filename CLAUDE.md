# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file, mobile-friendly web radio launcher for streaming Icecast/Shoutcast stations. The entire application is contained in `index.html` - a self-contained HTML file with embedded CSS and JavaScript, designed to be deployed to a homelab server and accessed via mobile devices.

## Architecture

**Single-File Application Structure:**
- Lines 1-19: Deployment instructions and usage notes (commented)
- Lines 20-191: HTML structure and embedded CSS with mobile-first design
- Lines 192-237: HTML markup for header, station grid, player controls, and toast notifications
- Lines 238-629: JavaScript application logic

**Key JavaScript Architecture:**
- Lines 239-255: Station configuration array (the primary data structure defining available radio streams)
- Lines 257-272: DOM element references
- Lines 274-280: LocalStorage keys and configuration constants
- Lines 286-330: UI update functions (toast, status, transport controls, now-playing display)
- Lines 331-381: Icecast metadata parsing and now-playing refresh logic
- Lines 383-404: Station selection and stream management
- Lines 406-471: Audio playback control (startStream, play, stop functions)
- Lines 505-526: Initial render and DOM setup
- Lines 528-551: State restoration from localStorage
- Lines 553-576: Inline self-tests (console assertions)
- Lines 578-624: Event listeners for UI controls and audio element

**External Dependencies:**
- Streams served from `https://jellyfin.trout-pancake.ts.net/radio/*.mp3` (Icecast endpoints)
- Status API: `https://jellyfin.trout-pancake.ts.net/radio/status-json.xsl` for track metadata
- Uses standard Web Audio API, MediaSession API, LocalStorage API

**State Management:**
- LocalStorage persists: last selected station URL, volume level
- In-memory state: `current` (selected station object), `isPlaying` (boolean)
- No external state management library used

## Development Workflow

**Testing:**
The application includes inline console-based assertions that run on page load (lines 553-576). To view test results, open browser developer console.

**Local Development:**
Serve the file with any HTTP server. For HTTPS (required for some mobile features like MediaSession):
```bash
# Python 3
python3 -m http.server 8000

# With SSL (for full PWA features)
# You'll need to configure SSL certificates for local testing
```

**Deployment:**
Simply copy `index.html` to your web server. Optional PWA features require creating:
- `manifest.webmanifest` (template at lines 638-651)
- `sw.js` service worker (template at lines 658-694)

## Modifying Station List

The primary customization point is the `stations` array (lines 239-255). Each station object requires:
- `emoji`: Display emoji (string)
- `name`: Station name (string)
- `url`: Direct Icecast/Shoutcast stream URL (must be HTTPS)

**Important:** After modifying stations, update the `ICECAST_STATUS_URL` constant (line 279) if using a different Icecast server for metadata.

## Key Technical Constraints

1. **Autoplay Restrictions:** Modern browsers block autoplay. Users must tap a station to initiate audio (see lines 420-430)
2. **CORS Requirements:** Streams must serve proper CORS headers for `<audio crossorigin="anonymous">` (line 232)
3. **HTTPS Required:** MediaSession API and service workers require HTTPS in production
4. **Mobile-First:** Design uses `env(safe-area-inset-*)` for notch/home indicator spacing (lines 57, 117)
5. **No Build Process:** Pure HTML/CSS/JS - edit the file directly, no transpilation or bundling

## Metadata System

The application polls Icecast's JSON status endpoint (lines 354-381) to display current track information:
- Extracts mount point from stream URL (lines 331-339)
- Parses Icecast JSON to find matching source (lines 346-352)
- Updates UI and MediaSession metadata (lines 368-377)
- Polls every 15 seconds (line 280)

**Note:** The Icecast JSON format can return either a single source object or an array, handled by `normalizeSources()` (lines 341-344)

## Common Modifications

**Changing Station URLs:**
Edit the `stations` array at line 239. Each entry must have `emoji`, `name`, and `url`.

**Adjusting Colors/Theme:**
CSS custom properties are defined in `:root` (lines 36-44). Modify these for global color changes.

**Polling Frequency:**
Change `NOWPLAYING_POLL_MS` constant at line 280 (default: 15000ms).

**Volume Default:**
See `restoreState()` function at line 531 (default: 0.9).
