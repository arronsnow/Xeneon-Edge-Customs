# 🌌 Galactic HUD — Xeneon Edge Widget

A Star Wars–themed Imperial holographic HUD for the Corsair Xeneon Edge secondary display. Built as a single self-contained HTML file served via GitHub Pages and loaded through iCUE's iFrame widget.

![Layout: Video | Calendar + Weather | Sensors]

---

## Features

- **Live clock** — hours and minutes, updates every second
- **Galactic Standard Calendar** — real-world date displayed as Star Wars year/month/day
- **Galactic Standard Year (GSY)** — calculated relative to the Battle of Yavin (May 25, 1977)
- **Seconds progress bar** along the bottom of the calendar panel
- **Live local weather** — temperature, feels like, humidity, wind speed, conditions, 4-hour forecast (Open-Meteo API, no key required)
- **Holographic Death Star video** — left panel, VP9 WebM with true alpha transparency (black background removed via FFmpeg chromakey)
- **CPU & GPU gauges** — load % and temperature, animated arc gauges updating every 2 seconds (via LibreHardwareMonitor)
- **Aurebesh decoder animation** — every 30 seconds all text scrambles into Star Wars alien script, holds, then decodes back to English
- **Fully fluid layout** — uses `vw`/`vh` units, adapts to any resolution iCUE provides
- **Imperial HUD aesthetic** — dark blue, cyan glow, scanlines, star field, corner brackets

---

## Layout

```
┌─────────────────────────────────────────────────────────────┐
│  LEFT (33%)          │  CENTER (33%)       │  RIGHT (33%)   │
│                      │  ┌───────────────┐  │  CPU LOAD      │
│  Death Star          │  │  CLOCK + DATE │  │  CPU TEMP      │
│  Hologram Video      │  ├───────────────┤  │  ──────────    │
│  (transparent bg)    │  │    WEATHER    │  │  GPU LOAD      │
│                      │  └───────────────┘  │  GPU TEMP      │
└─────────────────────────────────────────────────────────────┘
```

---

## Requirements

### LibreHardwareMonitor *(for CPU/GPU sensors)*
- Download: https://github.com/LibreHardwareMonitor/LibreHardwareMonitor/releases
- Must be run as **Administrator**
- Enable: `Options → Web Server → Run Web Server`
- Enable: `Options → Run On Windows Startup`
- Enable: `Options → Start Minimized`
- Serves sensor data at: `http://localhost:8085/data.json`

### GitHub Account *(for hosting)*
- Free at https://github.com
- Repository must be **Public** for GitHub Pages on the free tier

### iCUE
- Use the **iFrame** widget
- Point it at your GitHub Pages URL

---

## Setup

### 1. Host on GitHub Pages

1. Create a public GitHub repository (e.g. `Xeneon-Edge-Customs`)
2. Upload `starwars_index.html` to the repo
3. Go to **Settings → Pages → Source → Deploy from branch → main**
4. Your widget will be live at:
   ```
   https://<your-username>.github.io/<repo-name>/starwars_index.html
   ```

### 2. Add to iCUE

1. Open iCUE → click your **Xeneon Edge** device
2. Go to **Widgets**
3. Drag the **iFrame** widget onto the display
4. Set size to **XL**
5. Paste your GitHub Pages URL into the iFrame field
6. Press Enter

### 3. LibreHardwareMonitor

1. Extract and run `LibreHardwareMonitor.exe` as Administrator
2. `Options → Web Server` → check **Run Web Server**
3. `Options` → check **Run On Windows Startup** and **Start Minimized**
4. Verify at: http://localhost:8085/data.json

---

## Fixing Sensor Data (Mixed Content)

The widget is served over **HTTPS** (GitHub Pages) but LibreHardwareMonitor runs on **HTTP** (localhost). Browsers block this by default. To fix it in Chrome:

1. Go to: `chrome://flags/#unsafely-treat-insecure-origin-as-secure`
2. Add `http://localhost:8085` to the field
3. Set the flag to **Enabled**
4. Click **Relaunch**

> **Alternative:** Open the HTML file directly in Chrome (`Ctrl+O`) instead of via GitHub Pages. Local `file://` URLs don't have mixed content restrictions.

---

## How It Works

### Tech Stack
- Pure HTML/CSS/JavaScript — no frameworks, no build tools
- All assets embedded as base64 (font + video) — fully self-contained
- Fluid layout via `vw`/`vh` and `clamp()` — no fixed pixel sizes

### Data Sources

| Data | Source | Auth |
|------|--------|------|
| Weather | [Open-Meteo](https://open-meteo.com) | None |
| Reverse geocoding | [Nominatim / OpenStreetMap](https://nominatim.openstreetmap.org) | None |
| CPU/GPU sensors | LibreHardwareMonitor `localhost:8085` | None |
| Clock / date | `JavaScript Date` | None |
| Orbitron font | Google Fonts | None |

### Video Transparency

The original MP4 had a black background. FFmpeg was used to remove it:

```bash
ffmpeg -i input.mp4 \
  -vf "colorkey=0x000000:0.35:0.1" \
  -c:v libvpx-vp9 -pix_fmt yuva420p \
  -b:v 800k -crf 20 -an \
  output_alpha.webm
```

### Font Loading

The Aurebesh OTF font is injected into a dynamically created `<style>` tag as a base64 data URI. `document.fonts.load()` is used to confirm the font is parsed before the decoder animation fires.

### Aurebesh Decoder

Every 30 seconds, all text elements scramble character-by-character into Aurebesh then back to English.

| Config | Value | Description |
|--------|-------|-------------|
| `HOLD` | `30000` ms | How long each mode (English/Aurebesh) is held |
| `STEP` | `45` ms | Delay between each character reveal |

---

## File Structure

```
Xeneon-Edge-Customs/
├── starwars_index.html   # Main widget (self-contained, ~8MB)
├── README.md             # This file
```

### What's Embedded in the HTML

| Asset | Format | Purpose |
|-------|--------|---------|
| `Aurebesh.otf` | base64 OTF | Star Wars alien script font |
| `hologram_alpha.webm` | base64 VP9 WebM | Death Star video with alpha transparency |

---

## Troubleshooting

**Widget is blank/white in iCUE**
- Visit the GitHub Pages URL in Chrome first to confirm it loads
- GitHub Pages can take ~2 minutes after a commit to go live
- Make sure your repo is set to **Public**

**Sensors show "SENSOR ARRAY OFFLINE"**
- Check LibreHardwareMonitor is running (look in system tray)
- Confirm `Options → Web Server` is checked
- Visit `http://localhost:8085/data.json` in Chrome to verify it's serving
- Apply the Chrome mixed content flag fix above

**Aurebesh font not showing**
- The font is embedded — no internet needed
- Wait the full 30 seconds for the first cycle to fire
- If testing locally, open directly in Chrome (`Ctrl+O`)

**Weather stuck on "ACQUIRING SENSOR DATA"**
- Allow location access when the browser prompts
- Requires internet access to reach open-meteo.com
- iCUE's embedded browser may not support geolocation — test in Chrome first

**Video not playing**
- Video is embedded in the file — no external file needed
- Muted autoplay is required and should be allowed by default in Chromium

---

## Built With

- [Claude](https://claude.ai) — AI assistant (Anthropic)
- [Open-Meteo](https://open-meteo.com) — weather API
- [LibreHardwareMonitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor) — sensor data
- [Aurebesh font](https://aurebesh.org) — Star Wars script
- [FFmpeg](https://ffmpeg.org) — video alpha channel processing
- [GitHub Pages](https://pages.github.com) — free static hosting

---

*May the Force be with you.* 🚀