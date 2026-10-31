# Home Base widget: handoff (Oct 31, 2026)

Repo: `gmeeks328/home-base-widget`, served by GitHub Pages at `https://gmeeks328.github.io/home-base-widget/`. Push to `main` to deploy (live in about a minute, sometimes up to 10).

## Pages

| File | What it is | Used as |
|---|---|---|
| `index.html` | Home Base widget: weather strip, Schedule / Inbox / Radar tabs | The iPhone home screen widget in WidgetWeb |
| `radar-widget.html` | Radar only, static, ◀ ▶ frame stepping, no location prompt | Optional second widget |
| `radar.html` | Full interactive radar: play/pause, scrubber, live-location blue tag, style switcher | Open in Safari |

## How WidgetWeb works (learned the hard way)

- The widget is a screenshot of the page. Nothing animates, and it can't ask for location without a prompt every refresh.
- Taps on `<a href="#name">` reload the page with that hash. WidgetWeb's cache ignores the hash, so the page reads `location.hash` itself. Tabs are `#schedule`, `#inbox`, `#radar`; radar frames are `#radar` (latest), `#radar1`, `#radar2`... (each step is about 10 minutes earlier).
- The page must define `__wweb2WaitMillisecondsToWidgetIsReady`. It is called repeatedly and ALWAYS receives `currentDelayMs=0`, so any time cap must be tracked inside the page. Return `0` when ready; any other number means "ask again in that many ms". (A version that always returned 400 made the Radar tab hang for the full ~30 s limit.)
- WidgetWeb keeps a saved copy of the page. After a push: set the widget URL to `.../home-base-widget/?v=N` (bump N), tap Update in the app, then refresh the widget. The Reload button on the widget fetches fresh.
- The widget's visible area is about 350 x 365 pt (1,050 x 1,095 px). The page viewport can be taller, so layout code caps at `WIDGET_H = 365`.
- iOS widgets get about 30 MB memory. Tile layers use `keepBuffer: 0` to stay light.
- The in-app log is at `http://127.0.0.1:18080/WWebWidgetExt_applog.txt` (on the phone).

## Map and radar

- Base map: Esri `World_Topo_Map` (keyless). Greens are boosted per pixel on a canvas (`RecolorTerrain`, `GREEN_BOOST` in `radar.html` / `radar-widget.html` / `index.html`). Only green hues are touched. If tile pixels can't be read (CORS), it falls back to a mild `saturate(1.25)`.
- Radar: RainViewer public feed (`api.rainviewer.com/public/weather-maps.json`), color scheme 6, falls back to scheme 2 if tiles fail. Native zoom 7.
- `radar.html` has a blue "you are here" tag via the browser's geolocation, with the town name from OpenStreetMap Nominatim (rounded coordinates only). The widget pages deliberately skip this and center on Waxahachie. Use `?lat=&lon=&name=` on `radar-widget.html` to center elsewhere.
- Other styles in `radar.html` (Satellite, Light gray) use Esri tiles with the green name tags. The topographic style hides the name tags because its tiles already carry names.
- Esri and RainViewer cannot be fetched from the build sandbox, so map colors were tuned from phone screenshots, not seen directly.

## Data and refresh

- `index.html` holds snapshot data: `EVENTS`, `DATA_THROUGH`, `INBOX` (counts only, no subjects), and the "Updated" stamp. Weather is fetched live from Open-Meteo, with the `WX` snapshot as fallback.
- Scheduled task "Home Base dashboard refresh" runs daily at 6:30 AM Central. It rebuilds the Home Base artifact (`d92e1968-e75b-468f-9189-3aa177f21304`) and, as of Oct 31, also updates the data above in this repo and pushes. First run with the repo step: Nov 1.

## Open items

- Check Nov 1 morning that the scheduled run pushed (look for a "Daily data refresh" commit) and that the widget's "Updated" time moved.
- Optional: raise radar opacity so rain stands out (`RADAR_OPACITY`, currently 0.85).
- Optional: Tomorrow.io as the widget's weather source instead of Open-Meteo.
- Dead code: `radar.html` still contains unused terrain-classification leftovers from earlier color experiments; harmless.
