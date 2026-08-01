# Morningside — Handoff

A first-person life sim. Six places to live, 21 shops you can walk into, jobs,
cooking, a two-storey house, flights between towns, and five animated shorts
playing in the cinema.

Everything is one file: `index.html`, about 324 KB, ~5,900 lines.

---

## 1. What you actually have

| | |
|---|---|
| **Engine** | three.js r128, loaded from a CDN |
| **Build step** | None. Open the file in a browser and it runs. |
| **Assets** | None. Every texture, building and character is generated in code at load. |
| **Save data** | Auto-saves to `localStorage` on sleep, on landing in a new town, and when a game begins. A **Continue** button on the title screen picks up where you left off. |
| **Dependencies** | Exactly one: three.js — now **vendored locally** as `three.min.js` (r128), no CDN. |

That last point is the important one for making an app. There is no npm, no
bundler, no asset pipeline, no server. It is a single static page.

### Content

- **6 towns** — Downtown, The Coast, The Countryside, The Desert, The Forest,
  The Rainforest
- **21 shop interiors**, each with its own furniture and a working counter
- **21 jobs**, one per shop, with a station-based shift you physically walk
- **10 recipes** and a pantry that runs down
- **A two-storey house** with a themed interior per town, a dimmer, and an
  optional household of up to three other people
- **An airport** in every town, with six gates and a flyable cabin sequence
- **Fruit Films** — five animated shorts, embedded as base64

---

## 2. Running it

```
open index.html
```

That is genuinely it. For a local server (needed for some phone testing):

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

---

## 3. Turning it into an app

Three routes, easiest first.

### Route A — PWA (installable web app)

Cheapest by far. Users "Add to Home Screen" and it behaves like an app: own
icon, no browser chrome, works offline.

**You need to add three things:**

1. **`manifest.json`**

```json
{
  "name": "Morningside",
  "short_name": "Morningside",
  "start_url": "./index.html",
  "display": "fullscreen",
  "orientation": "landscape",
  "background_color": "#05070b",
  "theme_color": "#0d1117",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

2. **A service worker** (`sw.js`) that caches `index.html` and the three.js
   file so it works offline.

3. **Two lines in `<head>`:**

```html
<link rel="manifest" href="manifest.json">
<script>navigator.serviceWorker?.register('sw.js')</script>
```

**Before you do this, vendor three.js locally.** Right now it loads from
`cdnjs.cloudflare.com`, so the game will not start offline. Download
`three.min.js` (r128, ~600 KB), put it next to `index.html`, and change the
script tag to `<script src="three.min.js"></script>`. Do not upgrade to a newer
three.js version without testing — see *Known constraints* below.

Host anywhere static: GitHub Pages, Netlify, Cloudflare Pages, Vercel. All
free. HTTPS is required for service workers.

### Route B — Capacitor (real iOS / Android app)

If you want it in the App Store or Play Store.

```
npm install -g @capacitor/cli
npx cap init Morningside com.yourname.morningside
# put index.html + three.min.js in a folder called www/
npx cap add ios
npx cap add android
npx cap sync
npx cap open ios      # opens Xcode
npx cap open android  # opens Android Studio
```

Capacitor wraps the page in a native shell. The game needs no native plugins —
no camera, no GPS, no notifications — so this is close to a straight wrap.

**Things to set in the native config:**
- Lock orientation to landscape
- Disable the bounce/overscroll on iOS
- Set the status bar to hidden
- Set the background colour to `#05070b` so there is no white flash on launch

You will need an Apple Developer account ($99/yr) to ship on iOS. Google Play
is a one-time $25.

### Route C — Electron (desktop app)

Overkill for this, but simple if you want a Mac/Windows download. Roughly 15
lines of `main.js` plus `electron-builder`. The download will be ~150 MB
because Electron ships a whole browser.

---

## 4. What needs doing before shipping

These are real gaps, in the order I would fix them.

### Must do

1. ~~**Vendor three.js locally.**~~ ✅ Done. `three.min.js` (r128) sits next to
   `index.html` and the script tag loads it directly — no CDN, works offline.

2. ~~**Add saving.**~~ ✅ Done. See the `SAVING` block near the bottom of
   `index.html`. It snapshots a whitelist of plain `state` fields to
   `localStorage` (skipping the live `Vector3`s like `sitting`/`returnTo`),
   saves on sleep / on landing / on a fresh start, and rebuilds the world on
   load via `applyHomeTheme()`, `applyHouseLight()`, `addHousehold()`. The
   title screen shows a **Continue** button when a save exists.

3. **Test on a real phone.** The mobile path exists (joystick, touch look,
   fullscreen, a `DENS` density dial that halves prop counts) but has been
   tested far less than desktop.

### Should do

4. **An audio pass.** There is no sound at all. Footsteps, a door, the fire,
   traffic, a kettle. This would do more for the feel than any visual work.

5. **A pause menu.** Currently ESC only frees the mouse. Needs a proper
   pause with volume, the light setting, and quit-to-title.

6. **Reduce the shop interiors' repetition.** Several shops share layout DNA.
   Fine now, more obvious once people play longer.

### Nice to have

7. Rain and weather — the cloud system in the sky shader is already there to
   build on
8. A driveable car
9. More TV channels (the format is documented in the code)

---

## 5. How the code is laid out

One file, in this order. Line numbers are approximate and will drift.

| Line | Section |
|---|---|
| 19 | Procedural textures — asphalt, sand, wood, tile, all drawn to canvas |
| 192 | Renderer, camera, resize handling, fullscreen |
| 227 | `player` and `state` — the two objects that hold everything |
| 299 | `makePerson()` — randomised characters |
| 399 | `room()` and shop interior helpers |
| 493 | `buildHome()` — the two-storey house |
| 1032 | The 21 shop interiors |
| 1814 | `SHOPS` — what each shop sells |
| 1937 | The TV and its channels |
| 2129 | Fruit Films — the embedded shorts |
| 2621 | House lighting / dimmer |
| 2660 | Per-town house styling |
| 2704 | Household — solo vs family |
| 2875 | Cooking, pantry, groceries |
| 3102 | Sitting |
| 3144 | Jobs and shifts |
| 3396 | The six towns |
| 4634 | The airport and the flight |
| 4897 | Moving between worlds |
| 5207 | Input — keyboard, mouse, touch |
| 5323 | Movement and collision |
| 5416 | Day/night cycle and sky shader |
| 5584 | The main loop |

### The two ideas worth understanding

**Worlds.** Every place — a town, a shop, your house — is a "world": its own
scene, its own collision boxes, its own interaction points. `getWorld(id)`
builds one on first use and caches it. Only one renders at a time.

```js
{ scene, colliders: [Box3], hotspots: [{p, r, label, run, y}], spawn, ... }
```

**Hotspots.** Everything you can interact with is a hotspot: a position, a
radius, a label, and a function. Each frame the nearest one within reach wins
and becomes the E key. That is the whole interaction system.

Adding something new to the world is usually one `hotspot()` call.

---

## 6. Known constraints

**three.js r128 specifically.** The code uses only geometry available in r128.
Two things have already bitten here: `CapsuleGeometry` does not exist in r128
(it caused a black screen in the surf shop), and newer versions changed colour
management, which would shift every colour in the game. If you upgrade, budget
a day and check every interior.

**No asset pipeline is a feature, not a gap.** Everything being generated in
code is why the file is 324 KB instead of hundreds of megabytes, and why it
loads instantly. Resist adding image files unless you genuinely need them.

**Point-light shadows are expensive.** Only the first two lamps in any room
cast shadows, deliberately. Raising that limit will hurt framerate on phones.

**Fruit Films is base64.** It sits inside `FRUIT_FILMS_B64` and is decoded to
a Blob URL on first watch. Adding more films this way is fine; adding video
this way is not — a 5 MB clip becomes ~7 MB of text.

---

## 7. Testing

There is a headless test harness that runs the whole game in Node with a
stubbed three.js — no browser, no GPU. It catches the class of bug that is
invisible until you walk into it.

It checks, among other things:
- Every interior can be entered *and* left
- Every interaction is reachable from ground you can actually stand on
- No spawn point is inside furniture or on top of an exit
- Every job station sits on open floor
- Every seat faces the thing it should
- No collision box is stranded outside its world
- Brightness in every room falls in a sensible band

This is worth rebuilding if you keep developing. It found real bugs repeatedly
— an invisible building parked at the centre of four maps, an entire wardrobe
rendering on the wrong floor, 21 rooms where pressing E threw you straight
back outside.

The stub needs to model rotation on bounding boxes. An earlier version did not,
and that is exactly what hid the invisible-building bug.

---

## 8. Credits and licensing

- **three.js** — MIT licence, needs an attribution line somewhere in the app
- **Fruit Films** — yours
- Everything else — yours

If you ship commercially, the only third-party obligation is the three.js MIT
notice. Put it in an About screen.
