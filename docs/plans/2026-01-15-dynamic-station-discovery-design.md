# Dynamic Station Discovery Design

**Date:** 2026-01-15
**Status:** Approved

## Overview

Transform the radio launcher from hardcoded stations to dynamic discovery via Icecast JSON API. The application will fetch station metadata on load and continuously poll for "now playing" track updates.

## Current State

- 15 hardcoded stations with emoji icons
- Manual station list maintenance required
- Track metadata polling for player only
- Static display, no station-level track previews

## Target State

- Dynamic station discovery from Icecast JSON endpoint
- Text-only station cards (no emojis)
- Track metadata visible on all station cards (truncated preview)
- Smart polling based on playback state and visibility
- Selection persistence by mount identifier

## Architecture

### Data Fetching

**Initial Load:**
1. Fetch `https://jellyfin.trout-pancake.ts.net/radio/status-json.xsl` with cache-busting
2. Parse `icestats.source` (handle both single object and array with existing `normalizeSources()`)
3. Transform each source into station object:
   - `name`: `server_name` if available, else prettified mount name
   - `mount`: Raw mount path (e.g., `/rock.mp3`)
   - `url`: Constructed by replacing internal URL with public Caddy-proxied URL
   - `nowPlaying`: Truncated `title` field (30 chars), empty string if no title
4. Populate global `stations` array
5. Call `render()` to build grid

**URL Transformation:**
```javascript
url: source.listenurl.replace('http://0.0.0.0:8000', 'https://jellyfin.trout-pancake.ts.net/radio')
```

Handles Caddy proxy path (`/radio`) that cannot be configured in Icecast hostname.

**Error Handling:**
- Network/parse errors: Show centered error message with "Retry" button
- Empty sources: Show "No stations available" (no retry)
- Malformed sources: Skip invalid entries, log warning, continue processing

### Polling Strategy

**Smart Intervals:**
- 10s when `isPlaying === true` (active playback)
- 30s when station selected but stopped (`current !== null && !isPlaying`)
- Paused when tab hidden (Page Visibility API)

**Polling Updates:**
1. Fetch status JSON with cache-busting
2. Update `stations` array `nowPlaying` fields by matching mount
3. Update all station card `.now-playing` divs in DOM
4. Update player now-playing display for selected station
5. On error: Show toast, keep existing data, continue polling

**Visibility Handling:**
- `visibilitychange` event listener
- Pause polling when hidden, resume at appropriate interval when visible

### UI Changes

**Station Cards:**

Remove emoji, add secondary track info line:

```html
<button class="station" data-mount="/rock.mp3">
  <div class="label">Rock</div>
  <div class="now-playing">Artist - Song Title (trun...</div>
</button>
```

**CSS Updates:**
- Remove `.emoji` styles
- Add `.now-playing`: 12px, muted color, margin-top 4px, ellipsis overflow
- Hide `.now-playing` div when empty (no track info available)

**Error State:**
```html
<div class="error-state">
  <div class="error-message">Unable to load stations from server</div>
  <button class="btn primary" id="btnRetry">Retry</button>
</div>
```

### State Management

**LocalStorage:**
- Change `mike_radio:lastStationUrl` → `mike_radio:lastStationMount`
- Store mount path (e.g., `/rock.mp3`) instead of full URL
- Keep `mike_radio:volume` unchanged

**Selection Restoration:**
1. After fetching stations, read `lastStationMount` from localStorage
2. Find station in array where `station.mount === lastStationMount`
3. If found: set `current`, update UI, set status "Ready"
4. If not found: clear localStorage silently, leave idle

### Utilities

**Mount Prettification:**

```javascript
function prettifyMount(mount) {
  return mount
    .replace(/^\//, '')              // Remove leading /
    .replace(/\.mp3$/, '')           // Remove extension
    .split(/[-_]/)                   // Split on hyphens/underscores
    .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(' ');
}
```

Examples:
- `/rock.mp3` → "Rock"
- `/mashups-and-mixtapes.mp3` → "Mashups And Mixtapes"

## Implementation Structure

### New Constants

```javascript
const BASE_SERVER_URL = 'https://jellyfin.trout-pancake.ts.net/radio';
const INTERNAL_SERVER_URL = 'http://0.0.0.0:8000';
const POLL_INTERVAL_PLAYING = 10000;
const POLL_INTERVAL_STOPPED = 30000;
const TRACK_TRUNCATE_LENGTH = 30;
```

### New Global Variables

```javascript
let pollTimer = null;
let currentPollInterval = null;
```

### New Functions

1. `prettifyMount(mount)` - Transform mount to display name
2. `fetchStations()` - Async fetch and transform JSON
3. `renderError()` - Display error state with retry
4. `updatePollingInterval()` - Adjust poll rate based on state
5. `handleVisibilityChange()` - Pause/resume on visibility change
6. `updateStationCards()` - Refresh card track info during polls

### Modified Functions

1. `render()` - Iterate dynamic array, update card HTML structure
2. `refreshNowPlaying()` - Update all cards + player
3. `chooseStation()` - Save mount to localStorage
4. `restoreState()` - Restore by mount, handle missing station
5. `runTests()` - Add prettifyMount tests, URL transformation tests
6. `startNowPlayingPolling()` - Replace with smart polling

### Initialization Flow

```javascript
async function init() {
  await fetchStations();
  document.addEventListener('visibilitychange', handleVisibilityChange);
  startNowPlayingPolling();
}

init();
```

## Testing

**Automated (Console Assertions):**
- `prettifyMount()` transformations
- URL replacement logic
- Existing helper functions (keep relevant tests)

**Manual Checklist:**
1. Initial load fetches and displays 16 stations
2. Cards show truncated track info
3. Tapping station starts playback
4. Player updates track info
5. Polling at 10s while playing
6. Polling at 30s when stopped with selection
7. Tab switch pauses/resumes polling
8. Page refresh restores last station
9. Network error shows retry, retry works
10. localStorage persists volume and mount

## Rollback Plan

Git commit `25d7f78` contains the working hardcoded version. To rollback:

```bash
git checkout 25d7f78 -- index.html
```

## Migration Notes

**No Breaking Changes:**
- LocalStorage key change won't break existing users (just won't restore last station on first load)
- All existing functionality preserved
- UI changes are purely visual (no interaction model changes)

**Future Enhancements:**
- Server-side Icecast configuration to eliminate URL translation
- Album art support (if Icecast extended metadata available)
- Listener count display
- Station filtering/search (if station count grows significantly)
