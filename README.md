# Mat Time

A single-file BJJ class planner + scoreboard-buzzer timer. Built to run from your phone at the gym — install once, works offline, edits persist locally.

Live: **https://jeremy-yap.github.io/mat-time/**

## Features

- Plan a class as a list of named segments with mm:ss durations
- Edit / reorder / delete segments on the fly (drag the `≡` handle to reorder)
- Big sticky timer card with play / pause / prev / next
- **Scoreboard buzzer** between segments — synthesized in Web Audio, no audio file to load
- **Locked-screen survival**: buzzers fire on time even with the phone locked in your pocket
- Volume slider (tap the speaker icon to test the buzzer)
- Long-press the countdown to restart the current segment
- Auto-saves to `localStorage` — your last plan is always there
- Default plan loaded on first visit: the 90-min closed-guard passing class
- Installs to iOS / Android home screen as a PWA, works fully offline

## Use

1. Open `https://jeremy-yap.github.io/mat-time/` on your phone.
2. In Safari: **Share → Add to Home Screen.** In Chrome on Android: **menu → Install app.**
3. Launch from the home-screen icon (gets you the standalone window without Safari chrome).
4. Edit the plan to suit the day's class. Hit play when class starts.

### Controls

| Action | How |
|--------|-----|
| Start / pause | Big coral button |
| Previous / next segment | Side buttons |
| Reset current segment | Long-press the countdown (~0.7s) |
| Jump to a segment | Double-tap its row |
| Edit name | Tap the name field |
| Edit duration | Tap the duration field, type `mm:ss` or just minutes |
| Reorder | Press and drag the `≡` handle |
| Delete | Tap `×` — undo within 5s via the toast |
| Add segment | Tap `+ Add segment` |
| Restore default plan | Tap "Reset to default plan" at the bottom |
| Test buzzer | Tap the speaker icon next to the volume slider |

### iOS caveats — read before using in class

- **Silent Mode**: the physical mute switch on the side of your iPhone silences Web Audio. Keep the ringer **on**, or use AirPods.
- **Low Power Mode**: aggressively suspends background tabs. Disable it before class.
- **Standalone install required for full reliability**: the screen wake lock and background-audio behaviour are much better when the app is launched from the home-screen icon than from a regular Safari tab.
- **First-time audio unlock**: the buzzer needs a user tap to initialize. The first tap of Play does this for you — but if you set up the plan without ever tapping Play, the very first buzz will be slightly delayed. Tap the speaker icon to pre-warm if you want.

## Deploy

```bash
cd "/Users/jeremy.yap/Claude Code/apr 1/mat-time"
git init -b main
git add .
git commit -m "Initial commit"
gh repo create mat-time --public --source=. --remote=origin --push
# Then in repo Settings → Pages → Deploy from a branch: main / root
```

To ship updates: edit, commit, push. Bump `CACHE_NAME` in `sw.js` (e.g. `mat-time-v1` → `mat-time-v2`) so phones evict the old cache and pick up the new files.

## Local development

```bash
cd "/Users/jeremy.yap/Claude Code/apr 1/mat-time"
python3 -m http.server 8080
# open http://localhost:8080/ in Chrome
# or open http://<your-mac-lan-ip>:8080/ on your phone, same wifi
```

## Files

| File | Purpose |
|------|---------|
| `index.html` | The whole app — markup, styles, JS, default plan |
| `manifest.webmanifest` | PWA metadata so iOS treats it as a real app when installed |
| `sw.js` | Service worker — caches the shell so the app works offline |
| `icon-192.png`, `icon-512.png` | Home-screen icons (tatami-grid motif) |
| `icon-source.svg` | The source SVG used to render the icons via `qlmanage` |

## How the locked-screen buzzer trick works

Three layered techniques (see [comments in `index.html`](./index.html)):

1. **Web Audio pre-scheduling.** When you tap Play, the app walks the entire plan and schedules each buzz at its absolute future `AudioContext.currentTime`. These events fire even when JavaScript itself is suspended.
2. **Silent audio loop.** A 2-second buffer of zeros plays on repeat from the moment you tap Play. This keeps the iOS audio session alive, preventing the OS from suspending the audio context.
3. **Screen Wake Lock + MediaSession metadata.** Foreground keeps the screen on; the media-session API tells iOS "this tab is playing media" so it stays alive in the background.
