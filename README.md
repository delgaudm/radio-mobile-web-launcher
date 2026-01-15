# Radio Mobile Web Launcher

A mobile-friendly, single-file web application for streaming Icecast radio stations. Dynamically discovers stations from your Icecast server and displays live track metadata with smart polling.

![Version](https://img.shields.io/badge/version-1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

- **Dynamic Station Discovery** - Automatically fetches available stations from Icecast JSON API
- **Live Track Metadata** - Shows current playing track on all station cards with auto-updates
- **Smart Polling** - Updates every 10s while playing, 30s when stopped, pauses when tab hidden
- **Mobile-First Design** - Optimized for phone screens with PWA support
- **Bluetooth Audio** - Streams play through your phone's Bluetooth output
- **Lock Screen Controls** - Media Session API integration for native playback controls
- **Persistent Selection** - Remembers your last station across page refreshes
- **Zero Dependencies** - Pure HTML/CSS/JavaScript, no build process required

## Quick Start

### Requirements

- An Icecast streaming server with JSON status endpoint
- Web server with HTTPS (required for MediaSession API and PWA features)

### Basic Setup

1. **Configure the server URL:**

   Edit `index.html` and update the constants (around line 279):
   ```javascript
   const ICECAST_STATUS_URL = 'https://your-server.com/radio/status-json.xsl';
   const BASE_SERVER_URL = 'https://your-server.com/radio';
   ```

2. **Deploy to your web server:**
   ```bash
   # Copy index.html to your web server
   cp index.html /var/www/html/radio/
   ```

3. **Access from your mobile device:**
   - Visit `https://your-server.com/radio/index.html`
   - iOS Safari: Share → Add to Home Screen
   - Android Chrome: ⋮ → Add to Home screen

That's it! The app will automatically discover your stations and start displaying them.

## Configuration

### Server URLs

If your Icecast server uses a reverse proxy (like Caddy or nginx), you may need to configure URL translation:

```javascript
const INTERNAL_SERVER_URL = 'http://0.0.0.0:8000';  // Icecast's internal URL
const BASE_SERVER_URL = 'https://your-server.com/radio';  // Public-facing URL
```

The app will automatically translate internal URLs to public URLs for streaming.

### Polling Intervals

Adjust how often track metadata refreshes:

```javascript
const POLL_INTERVAL_PLAYING = 10000;  // 10s when playing (default)
const POLL_INTERVAL_STOPPED = 30000;  // 30s when stopped (default)
```

### Station Name Display

Station names are derived from Icecast mount points:
- `/rock.mp3` → "Rock"
- `/classic-rock.mp3` → "Classic Rock"
- `/seventies-soul.mp3` → "Seventies Soul"

Names are automatically prettified by capitalizing words and replacing hyphens/underscores with spaces.

## Optional: PWA Features

For a full Progressive Web App experience with offline support:

1. **Create `manifest.webmanifest`:**
   ```json
   {
     "name": "Radio Launcher",
     "short_name": "Radio",
     "start_url": "./index.html",
     "display": "standalone",
     "background_color": "#070a12",
     "theme_color": "#070a12",
     "icons": [
       {"src": "./icons/icon-192.png", "sizes": "192x192", "type": "image/png"},
       {"src": "./icons/icon-512.png", "sizes": "512x512", "type": "image/png"}
     ]
   }
   ```

2. **Create `sw.js` (service worker):**

   See the commented template at the bottom of `index.html` (lines 658-694)

3. **Uncomment the PWA lines** in `index.html` `<head>`:
   ```html
   <link rel="manifest" href="./manifest.webmanifest">
   <script>if ('serviceWorker' in navigator) navigator.serviceWorker.register('./sw.js');</script>
   ```

## Architecture

### How It Works

1. **Initial Load:**
   - Fetches `status-json.xsl` from Icecast server
   - Parses available sources and transforms into station objects
   - Renders station grid with current track metadata

2. **Polling:**
   - Smart interval based on playback state (10s/30s)
   - Pauses when tab is hidden (Page Visibility API)
   - Updates all station cards and player display

3. **State Management:**
   - Last selected station stored in localStorage by mount point
   - Volume level persisted across sessions
   - Restored on page load if station still available

### File Structure

```
radio-mobile-web-launcher/
├── index.html           # Single-file application (entire app)
├── CLAUDE.md            # Development guide for AI assistants
├── README.md            # This file
└── docs/
    └── plans/
        └── 2026-01-15-dynamic-station-discovery-design.md
```

## Browser Compatibility

- **Modern browsers** with ES6+ support
- **iOS Safari** 12+
- **Android Chrome** 80+
- **Desktop browsers** (Chrome, Firefox, Safari, Edge)

**Note:** MediaSession API and service workers require HTTPS in production.

## Troubleshooting

### Stations Not Loading

1. Check browser console for errors
2. Verify Icecast status URL is accessible: `curl https://your-server.com/radio/status-json.xsl`
3. Ensure CORS headers are set on Icecast server
4. Confirm the audio elements have `crossorigin="anonymous"` attribute

### Autoplay Blocked

Modern browsers block autoplay until user interaction. The app handles this by:
- Requiring a tap to start playback
- Showing "Tap Play to start" toast if autoplay fails
- This is expected behavior and cannot be bypassed

### Track Metadata Not Updating

1. Verify Icecast is serving track metadata in JSON response
2. Check that `title` field exists in source objects
3. Confirm polling is active (check console for fetch requests)
4. Try manual refresh with "Retry" button

## Development

### Local Testing

```bash
# Serve locally
python3 -m http.server 8000

# Visit in browser
open http://localhost:8000/index.html
```

### Running Tests

Open browser console after page load. All assertions should pass silently. Any failures will appear as warnings.

### Making Changes

The entire application is in `index.html`. Key sections:
- Lines 36-189: CSS styles
- Lines 238-290: Configuration and constants
- Lines 291-820: JavaScript application logic

See `CLAUDE.md` for detailed architecture notes.

## License

MIT License - feel free to use, modify, and distribute.

## Credits

Built with vanilla HTML/CSS/JavaScript. No frameworks, no dependencies, no build tools.

Designed for homelab enthusiasts who want a simple, reliable way to stream their Icecast stations on mobile devices.

## Version History

### v1.0 (2026-01-15)
- Initial release
- Dynamic station discovery from Icecast JSON API
- Smart polling with visibility-aware intervals
- Live track metadata on all station cards
- Mobile-first responsive design
- LocalStorage persistence
- MediaSession API integration
