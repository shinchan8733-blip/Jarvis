# Jarvis — Android Studio project

A real native Android app (WebView shell around the Jarvis voice-coach web
app) that Android Studio will compile into an actual signed/installable
`.apk` on your machine.

## Open it

1. Unzip `JarvisAndroid.zip` somewhere on disk.
2. Android Studio → **File → Open** → select the unzipped `JarvisAndroid`
   folder (the one containing `settings.gradle`).
3. Android Studio will say the Gradle wrapper is missing and offer to
   create/download it — accept that (or "Sync with Gradle wrapper" if
   prompted). First sync will download Gradle 8.7 + the Android Gradle
   Plugin, so do this on wifi.
4. Once sync finishes: **Run ▶** with a device or emulator selected, or
   **Build → Build Bundle(s) / APK(s) → Build APK(s)** to get an installable
   `app-debug.apk` under `app/build/outputs/apk/debug/`.

That APK is a real app: full-screen, its own launcher icon ("Jarvis"), no
browser UI, and it works fully offline (fonts are the only thing that need
wifi, and they fall back gracefully without it).

## How it's built

`MainActivity.kt` is a single-screen app that loads
`app/src/main/assets/index.html` — the same Jarvis routine-coach web app —
into a full-screen `WebView`. Two things are wired up specifically for the
native shell:

- **Text-to-speech** — works out of the box; modern Android WebView ships
  `window.speechSynthesis` on top of the system TTS engine.
- **Mic ("Ask Jarvis") button** — needs the `RECORD_AUDIO` permission, which
  the app requests on first launch, and `WebChromeClient.onPermissionRequest`
  forwards that grant into the page so `webkitSpeechRecognition` works.

## Editing your routine

Edit `app/src/main/assets/index.html` — the `ROUTINE` array near the top of
the `<script>` block, entries are `{ id, start, end, title, desc }` in
24-hour time. Matching lines in `MOTIVATION[id]` give that step its own
pep-talk lines. After editing, just re-run the app (no need to touch any
Kotlin/Gradle files).

## Or: let GitHub build the APK for you (no local Gradle/SDK needed)

A workflow at `.github/workflows/build.yml` is already included. It builds a
debug APK in the cloud on every push and uploads it as a downloadable
artifact — useful if you'd rather not wait on a local Gradle/SDK sync at all.

1. Create a new **public or private** repo on GitHub (empty, no README).
2. From inside the unzipped `JarvisAndroid` folder:
   ```
   git init
   git add .
   git commit -m "Jarvis app"
   git branch -M main
   git remote add origin <your-repo-url>
   git push -u origin main
   ```
3. On GitHub, open the **Actions** tab of the repo — a "Build Jarvis APK" run
   starts automatically (takes a couple of minutes: it sets up the Android
   SDK from scratch each time).
4. When it finishes, open the run → scroll to **Artifacts** → download
   `jarvis-debug-apk` (a zip containing `app-debug.apk`).
5. Copy that APK to your phone and tap it to install (you'll need to allow
   "install unknown apps" for whatever app you used to open it — Android will
   prompt you the first time).

Re-run any time by pushing new commits, or manually from the Actions tab via
**Run workflow** (the workflow also listens for `workflow_dispatch`).

## Publishing beyond your own phone

The debug APK from step 4 above installs fine by sideloading (copy it to the
phone, or `adb install app-debug.apk`). If you ever want it on the Play
Store, you'd switch to a signed **release** build (Android Studio →
**Build → Generate Signed Bundle / APK**), which needs a keystore — Android
Studio walks you through creating one the first time.
