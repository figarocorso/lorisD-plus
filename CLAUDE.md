# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

lorisD+ is a Chrome extension (Manifest V3) that pranks users by replacing webpage images with themed alternatives. It's designed to be subtle and escalate over time, making it harder for victims to detect.

## File Structure

- `manifest.json` - Extension configuration (Manifest V3)
- `imageReplacer.js` - Content script that runs on all webpages
- `incrementImageReplace.js` - Service worker (background script)
- `options.html` + `options.js` - Extension settings UI
- `lorisImages.js` - Default image library
- `defaultOptions.js` - Initial settings loaded on install
- `css/style.css` - Styles for options page and replacement effects
- `images/` - Extension icons and test images

## Architecture

### Core Components

**Content Scripts** (runs on all web pages):
- `imageReplacer.js` - Main replacement logic, runs on every page load
  - Asynchronously loads settings from Chrome storage
  - Polls for new images every 3 seconds using `setInterval`
  - Handles three replacement strategies: image URL replacement, "censored" CSS overlays, and "spin in place" animations
  - Uses CSS `object-fit:cover` and `object-position:50% 35%` to scale replacement images while preserving original dimensions

**Service Worker** (background script):
- `incrementImageReplace.js` - Gradually increases replacement probability over time
  - Sets up a Chrome alarm that fires every 5 minutes
  - Checks if enough time has passed based on `incrementInterval` (default: 24 hours)
  - Increases `imgReplaceProb` by `incrementValue` each interval, capped at 1.0
  - Runs once on installation to load default options and open options page

**Image Libraries**:
- `lorisImages.js` - Loris image URLs (default)
- Custom library support via textarea input in options

**Options Page**:
- `options.html` + `options.js` - User-facing configuration interface
  - Custom image library with URL validation using regex
  - Image preview/validation UI with next/prev controls

### Data Flow

1. **Installation**: `incrementImageReplace.js` loads `defaultOptions.js` into Chrome storage
2. **Page Load**: `imageReplacer.js` reads settings from Chrome storage, then runs replacement logic
3. **Background Timer**: Every 5 minutes, service worker checks if probability should increase
4. **Settings Changes**: Options page updates Chrome storage, which content scripts read on next page load

### Key Settings Structure

```javascript
{
  settings: {
    imageReplacement: {
      enableImgReplace: boolean,
      imgReplaceProb: number (0-1),
      imgLibraryName: string ("loris" or "custom"),
      imgLibrary: array,
      customImageLibrary: array,
      incrementValue: number (0-1),
      incrementInterval: number (milliseconds),
      messageForVictim: string,
      lastUpdate: timestamp
    }
  }
}
```

## Testing the Extension

**Load unpacked extension in Chrome**:
1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select this repository directory

**Test replacement logic**:
1. Open the extension's options page (right-click icon → Options, or it auto-opens on install)
2. Adjust replacement probability to 100% for immediate testing
3. Set `incrementValue` to 0 to disable auto-escalation during testing
4. Visit any image-heavy website to see replacements

**Test custom images**:
1. Select "custom" library in options
2. Paste comma-separated image URLs in textarea
3. Use next/prev buttons to preview each URL
4. Valid URL count shows below textarea

## Debugging

**View service worker logs**:
1. Navigate to `chrome://extensions/`
2. Click "Inspect views: service worker" under lorisD+
3. Console logs from `incrementImageReplace.js` appear here

**View content script logs**:
- Open DevTools on any webpage
- Content script logs from `imageReplacer.js` appear in the page's console

**Reset extension state**:
- Remove and re-add the extension to clear `chrome.storage.sync` and reset to defaults

## Important Implementation Notes

- This extension is designed as a prank that manipulates browser content without user awareness. Code contributions should not enhance stealth capabilities or make detection more difficult.
- The extension uses `chrome.storage.sync` which has quota limits (100KB total, 8KB per item)
- Content script runs at `document_end` to ensure DOM is ready
- Image replacement logic tracks `numImages` to avoid re-processing images on each interval
- CSS animations use randomized durations (0.5s - 20s) and directions for "spinInPlace" mode
- "Censored" mode traverses up the DOM tree to find parent `<div>` for overlay placement
- Replacement images must be publicly accessible URLs with appropriate CORS headers or same-origin policy
