# The Warden's Labyrinth – Mobile Build Guide

## Overview
This repository contains **The Warden's Labyrinth**, a browser‑based action‑arcade game built with Vite + React + TanStack.  The original project uses **SSR** (via `@Anti Gravity).  To run the game as a native Android application we:
1. **Generated a static SPA build** (no server required).
2. **Configured Capacitor** to serve the SPA from the Android assets folder.
3. **Fixed the mobile UI layout** so the canvas occupies ~65 % of the screen height and controls the remaining ~35 %.
4. **Forced landscape orientation** on launch (Clash‑of‑Clans style).
5. **Added responsive CSS variables** to adapt the layout for portrait/landscape.

The result is an APK that launches directly into the game, looks great on phones and tablets, and respects the desired orientation.

---

## Prerequisites
| Tool | Version |
|------|---------|
| Node.js | `>=18` |
| npm (or `pnpm`/`yarn`) | latest |
| Android Studio | 2022.2+ (includes Android SDK) |
| Capacitor CLI | `>=4` |
| Vite | `7.x` |

> **Tip**: All commands below assume you are in the project root `c:\Users\Waqar\AI‑Warden‑Run`.

---

## 1. Install dependencies
```bash
npm ci               # installs exact versions from package-lock.json
# or
pnpm install
```
---

## 2. Build the static SPA
The project ships an npm script that builds the SPA and renames the entry file to `index.html`:
```bash
npm run build:spa
```
- Output is placed in `dist/client/`.
- The generated files are:
  - `dist/client/index.html`
  - `dist/client/assets/main-<hash>.js`
  - `dist/client/assets/main-<hash>.css`
- **Important**: The script also updates the `<script>` and `<link>` tags to point to the new hashed asset names.

---

## 3. Capacitor configuration
`capacitor.config.json` already points to the SPA output folder:
```json
{
  "appId": "com.wardenslabyrinth.game",
  "appName": "The Warden's Labyrinth",
  "webDir": "dist/client",
  "bundledWebRuntime": false,
  "plugins": {
    "ScreenOrientation": {
      "orientation": "sensorLandscape"
    }
  }
}
```
The `ScreenOrientation` plugin forces the activity to stay in landscape mode.

---

## 4. Copy assets to Android and sync
```bash
npx cap copy android   # copies dist/client → android/app/src/main/assets/public
npx cap sync android   # updates Android project (plugins, config, Gradle files)
```
Both commands complete in < 2 seconds (see task logs).

---

## 5. UI layout fixes (WardenGame.tsx)
The component now enforces a clean, non‑scrolling layout:
- **CSS variables** (defined in a `<style>` tag) control canvas height:
  ```css
  :root {
    --canvas-h: 65vh;          /* mobile portrait – canvas takes 65 % of viewport */
    --canvas-max-h: 65vh;
  }
  @media (min-width: 1024px) {
    :root {
      --canvas-h: auto;        /* desktop / large tablets – let flex grow */
      --canvas-max-h: calc(100vh - 120px);
    }
  }
  ```
- The outer wrapper uses `flex flex-col h-screen overflow-hidden` so no part of the UI is cut off.
- Buttons (dash, fire, swap weapon, freeze, grenade) remain on the right side; the joystick stays on the left.
- The header *"THE WARDEN'S LABYRINTH"* is hidden on small screens via the `.portrait-warning` overlay, preserving space.
- `overflow:hidden` on the root container ensures no accidental scrolling.

---

## 6. Forcing landscape orientation
The `AndroidManifest.xml` includes:
```xml
<activity
    android:screenOrientation="sensorLandscape"
    android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|smallestScreenSize|screenLayout|uiMode"
    android:name=".MainActivity"
    ...>
```
No additional Java/Kotlin code is required.

---

## 7. Build & run the APK
```bash
# From the project root
cd android
./gradlew assembleDebug   # builds app-debug.apk
# Or open the project in Android Studio and press ▶ Run
```
Install the generated APK on a device:
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```
The app should launch directly in landscape, showing the game canvas and controls without any clipping.

---

## 8. Troubleshooting
| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Blank screen / `main‑xxxx.js not found` | Stale `index.html` still referencing old asset hash. | Re‑run `npm run build:spa` then `npx cap copy android`.
| UI elements cut off | CSS variables not applied (e.g., custom CSS removed). | Verify the `<style>` block in `WardenGame.tsx` near line 1578 contains the `--canvas-h` definitions.
| App stays in portrait | `android:screenOrientation` missing or overridden. | Ensure the `<activity>` tag in `android/app/src/main/AndroidManifest.xml` matches the snippet above.
| Controls not responding on touch devices | `pointerdown` listeners missing on buttons. | Confirm the button markup (`onPointerDown`) is present and `engineRef.current` is defined.

---

## 9. Clean rebuild (optional)
If you ever need to start from a clean slate:
```bash
# Remove Android assets
rm -rf android/app/src/main/assets/public
# Clean Vite build
rm -rf dist
# Re‑install dependencies (optional)
npm ci
# Repeat steps 2‑6
npm run build:spa && npx cap copy android && npx cap sync android
```
---

## 10. License & Credits
- **Game code** – © 2024 The Warden’s Labyrinth team (MIT License).
- **Assets** – Fonts from Google Fonts (Inter), icons from Font Awesome, sound assets from `@/lib/audio`.
- **Capacitor** – Apache 2.0 license (see `android/LICENSE` for details).

---

*Generated by Antigravity – an advanced agentic coding assistant.*
