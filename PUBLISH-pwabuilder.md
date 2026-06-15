# Publishing Steel Talon to Google Play with PWABuilder

You already have the right tools: **GitHub**, **VS Code**, and **PWABuilder**.
The plan: host the game as a PWA on **GitHub Pages**, let **PWABuilder** wrap it into a
signed Android `.aab` (a Trusted Web Activity), then upload that to **Google Play Console**.

This folder is already a complete PWA:
```
index.html        the game
manifest.json     PWA manifest (name, icons, landscape, fullscreen)
sw.js             service worker (offline support)
icons/            app icons (192, 512, maskable, favicon)
```

---

## Step 1 — Put the files on GitHub Pages (VS Code)

**Use a user-site repo so Digital Asset Links work at the domain root.** Name the repo
**exactly** `YOURUSERNAME.github.io` (replace with your real GitHub username). This makes the
site live at `https://YOURUSERNAME.github.io/` and lets you serve `/.well-known/assetlinks.json`
at the root — which the Android app needs (more in Step 4).

1. On GitHub, create a new **public** repo named `YOURUSERNAME.github.io`.
2. In VS Code: open this folder, then **Source Control** panel → **Initialize Repository** →
   stage all → commit ("initial") → **Publish/Push** to the repo you just made (branch `main`).
   (Or via terminal:)
   ```bash
   git init
   git add .
   git commit -m "Steel Talon PWA"
   git branch -M main
   git remote add origin https://github.com/YOURUSERNAME/YOURUSERNAME.github.io.git
   git push -u origin main
   ```
3. On GitHub: repo **Settings → Pages**. For a `username.github.io` repo it builds the root
   automatically. If you see a Source option, choose **Deploy from a branch → main → / (root)**.
4. Wait 1–2 minutes, then open `https://YOURUSERNAME.github.io/` — the game should load.

> Prefer a project repo (e.g. `steel-talon`)? It works, but the site is at
> `username.github.io/steel-talon/` and the asset-links file STILL has to live at
> `username.github.io/.well-known/assetlinks.json` (the origin root). The user-site repo
> above avoids that headache, so it's the recommended path.

## Step 2 — Sanity-check the PWA

Open the live URL in **Chrome → F12 → Application tab**:
- **Manifest** should show "Steel Talon", landscape, the icons.
- **Service Workers** should show `sw.js` activated.
If both look good, you're ready to package.

## Step 3 — Package with PWABuilder

1. Go to **https://www.pwabuilder.com**, paste `https://YOURUSERNAME.github.io/`, **Start**.
2. It scores your manifest/service worker/security — should be green.
3. Click **Package For Stores → Android (Google Play)**.
4. In **Android options**:
   - **Package ID**: `com.YOURUSERNAME.steeltalon` — pick carefully, it **can never change**
     once published.
   - **App name**: Steel Talon. **Version**: 1.0.0, **Version code**: 1.
   - **Signing key**: choose **Create new** (PWABuilder generates one).
5. **Download** the zip. It contains:
   - `app-release-bundle.aab` → upload this to Play.
   - `app-release-signed.apk` → for testing on a device/emulator.
   - **`signing.keystore` + `signing-key-info.txt`** → **BACK THESE UP AND NEVER LOSE THEM.**
     They're the only way to ship updates.
   - `assetlinks.json` → used in Step 4.

## Step 4 — Deploy Digital Asset Links (removes the URL bar)

The Android app must prove it owns your site, or it shows a browser address bar.

1. From the PWABuilder zip, take `assetlinks.json`.
2. In your repo, create the folder + file: **`.well-known/assetlinks.json`** (note the leading dot).
3. Commit & push. Confirm it loads at:
   `https://YOURUSERNAME.github.io/.well-known/assetlinks.json`
4. **Important:** Google re-signs your app with its own key (Play App Signing). After Step 5
   you must come back and ALSO add Play's signing fingerprint to this file:
   Play Console → your app → **Setup → App integrity → App signing key certificate → SHA-256** →
   add it as a second entry in the `sha256_cert_fingerprints` array (comma-separated), then
   push again. Skipping this is the #1 reason the address bar still appears.

## Step 5 — Upload to Google Play Console

1. Create a **Google Play developer account** ($25 one-time, two-step verification required) at
   https://play.google.com/console.
2. **Create app** → name, language, **App** type = Game, Free.
3. Pick a track — start with **Internal testing** to confirm it installs and runs, then promote
   to **Production** later.
4. **Upload the `.aab`.** Accept **Play App Signing** when prompted.
5. Complete the required sections before you can roll out:
   - **Store listing**: short/long description, your icon, and at least 2 **landscape** phone
     screenshots (grab them from a device or emulator running the game).
   - **Content rating** questionnaire (note: it has mild combat).
   - **Data safety** form (this game collects nothing — declare accordingly).
   - **Target audience**: choose **older users (not children)** — Google policy currently does
     not allow PWA/TWA apps to target children.
   - Privacy policy URL (you can host a simple one on the same GitHub Pages site).
6. Go back and finish **Step 4.4** (add Play's signing SHA-256 to assetlinks.json).
7. **Review release → Roll out.** First review usually takes a few days.

---

## Notes & gotchas

- **Target API level:** new Google Play apps must target Android 16 (API 36) from
  **Aug 31, 2026**. PWABuilder's current packaging targets a recent API; if Play ever rejects
  for target API, just re-package on PWABuilder (it tracks the latest) and upload a new version.
- **Updating later:** bump the version code, re-package on PWABuilder **using the same
  `signing.keystore`** (upload it in the Android options), and upload the new `.aab`.
- **Offline:** the service worker caches everything, so the installed app runs without internet.
- **Test before submitting:** install the `.apk` from the zip on an Android phone (enable
  "install unknown apps") to confirm it runs full-screen with no address bar once assetlinks
  is live.
