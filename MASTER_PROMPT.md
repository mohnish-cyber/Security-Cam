# MASTER PROMPT: Build my home security camera app

> Keep this file in the root of the project folder. It is the single source of truth for the project.
> Before the first session, fill in the section "Fill these in" below.

---

## Fill these in (edit before first use)

- **My computer's operating system:** [Windows 11 / Windows 10 / macOS — write yours]
- **Camera phone (the spare/old one):** [brand + model + Android version, e.g. "Redmi Note 7, Android 10"]
- **Viewer phone (my main one):** [brand + model + Android version]
- **App name:** [leave blank if you want suggestions]
- **My name or nickname (for the package ID):** [e.g. "rahul"]

---

## 1. About me and how you must work with me

I have **zero** app development experience. I have never written code or used a terminal, Git, Android Studio, or Firebase. You are both my **developer** and my **teacher**.

Rules for working with me:

1. Explain everything in simple English with short sentences. The first time you use a technical word, explain it in one line.
2. Do all the coding, commands, and file editing yourself. Only ask me to do things you truly cannot do, such as installing programs with a graphical installer, clicking inside websites (Firebase Console, Google Cloud, Play Console), physical actions on my phones, or entering payment details.
3. When I must do something manually, give me numbered steps with exact button names and what I should see on screen. Give me **one task at a time** and wait for me to reply "done" before continuing.
4. Before installing software, changing system settings, or deleting anything, tell me in one line what you are doing and why.
5. Work strictly **phase by phase** (Section 6). Never start the next phase until the current phase passes its "Done when" checklist on my real phones and I reply "phase complete".
6. At the end of each phase, give me: a plain-words summary of what we built, a test checklist for me to try on my phones, any known limitations, and then make a Git commit.
7. When something breaks: read the full error, explain the cause simply, fix it, and tell me how to check the fix. If you are still stuck after two attempts, stop and explain my options.
8. Never claim something works if you have not verified it. If only I can test it on the phone, say so clearly.
9. Keep replies short and focused. I will ask if I want more detail.
10. **Warn me before any step that could cost money**, with an estimated cost in Indian rupees where possible.
11. Any step that needs my input or confirmation must be done interactively, one step at a time. Do not put such steps inside a dynamic workflow, because workflows cannot pause for my input. Workflows are fine for large self-contained jobs (for example, a code-wide security audit or bug sweep).

---

## 2. Project memory files (create and keep updated)

- **CLAUDE.md**: key decisions, tech stack, folder structure, the exact commands to run and build the app, and coding conventions. Keep it under about 150 lines.
- **PROGRESS.md**: checklist of all phases and steps with status (done / in progress / not started), open issues, and "next step".
- **docs/SETUP.md**: a record of every manual setup step I did (Firebase project name, which Google account, where config files live) so I could redo it. **Never write passwords, private keys, or secret values in any file.**

At the start of every new session: read CLAUDE.md, PROGRESS.md, and this file, then tell me in 3 lines where we are and what's next.

Update PROGRESS.md at the end of every session, even if a phase isn't finished.

---

## 3. The product

**Name:** Use the name from "Fill these in". If blank, suggest 5 original names in Phase 0 and let me choose. The app must have its **own** name, logo, colours, and design. Do not copy any existing app's name, branding, text, screenshots, or UI (it is inspired by the general idea of apps like AlfredCamera, but must be clearly its own product).

**One sentence:** An Android app that turns a spare old phone into a home security camera that I can watch live and get motion alerts from on my main phone.

**One app, two modes** (chosen after sign-in, changeable later in settings):

- **Camera mode**: runs on the spare phone, usually plugged into a charger and on home Wi-Fi. Streams live video when a viewer opens it, detects motion, records clips, sends alerts, and plays siren or talk audio.
- **Viewer mode**: runs on my main phone, on Wi-Fi or mobile data. Shows my cameras, plays live video, receives alerts, browses recorded events, and controls cameras.

One account can have many camera phones and several viewer phones.

**Target users:** ordinary families in India, many with old or low-end Android phones and patchy internet. So the app must work on old phones, use little mobile data, have a very simple UI, and work for viewers on 4G.

**Language:** English UI first, but keep all user-facing text in localization files so Hindi and Marathi can be added later.

---

## 4. Technical decisions (fixed unless you give me a strong reason to change)

- **Framework:** Flutter (latest stable) with Dart. **Android first.** Do not build iOS now, but avoid choices that would block iOS later.
- **Minimum Android version:** the oldest version that all chosen libraries support (ideally Android 7.0 / API 24). Tell me the final minimum and why, and check that my camera phone is supported.
- **State management:** Riverpod. **Navigation:** go_router.
- **Backend:** Firebase
  - Firebase Authentication (Google sign-in + email/password)
  - Cloud Firestore (devices, signaling, events, settings)
  - Cloud Functions in TypeScript (push notifications, cleanup jobs, TURN credentials)
  - Firebase Cloud Messaging (push notifications)
  - Cloud Storage for Firebase (snapshots and video clips)
  - Firebase Crashlytics (crash reports)
  - Use a region close to India (e.g. asia-south1, Mumbai) wherever supported.
- **Live video:** WebRTC using the `flutter_webrtc` package.
  - Signaling through Firestore.
  - STUN: a public STUN server for early testing.
  - TURN: required for real-world use, because many Indian mobile networks use carrier-grade NAT, which blocks direct connections. In Phase 2b, compare (a) running coturn on a small VPS and (b) a managed TURN service, with cost and effort for each, and let me choose. **Never hardcode long-lived TURN credentials in the app**; generate short-lived credentials from a Cloud Function.
- **Background running:** an Android foreground service (a Flutter package such as `flutter_foreground_task`, or native Kotlin code if needed), with the correct camera and microphone foreground-service types and permissions for newer Android versions.
- If any package named here is deprecated, abandoned, or clearly worse than an alternative, tell me before switching. Verify current package versions and APIs from their official docs rather than relying on memory.

---

## 5. Critical technical gotchas (read carefully)

1. **Only one component can use the camera at a time.** Live streaming, motion detection, and recording must all share the **same camera source** (the WebRTC local video track). They must not open the camera separately. Early in Phase 2, run a short experiment ("spike") comparing options, such as capturing frames from the track at low FPS, a native Android video sink/processor, and recording via the WebRTC media recorder. Pick the most reliable approach, explain the trade-off to me, and only then build on it.
2. **Camera mode must run for hours or days.** Use a foreground service with a persistent notification and a partial wake lock. Ask the user to exempt the app from battery optimization, with a help screen for aggressive phone brands (Xiaomi/Redmi/POCO, Realme, Oppo, Vivo, OnePlus, Samsung). Auto-reconnect after Wi-Fi drops. Offer an optional auto-start after the phone reboots. Send a heartbeat so viewers can see whether the camera is online.
3. **Newer Android rules:** camera foreground services must be started while the app is visible (Android 14+). Declare the correct foreground-service types and permissions. Request the notification permission on Android 13+. Handle every permission being denied with a friendly explanation screen.
4. **Screen:** offer a "dim screen" mode (black overlay, lowest brightness) that saves power and prevents screen burn-in while the camera keeps running.
5. **Heat and battery:** alert the viewer if the camera phone is overheating, not charging, or low on battery.
6. **Mobile data:** use adaptive bitrate. Default live view around 480p to save data, with an HD option.
7. **Old phones:** keep motion detection cheap (low-resolution grayscale frames, about 1–2 checks per second).
8. **Privacy and ethics:** the app must **never** be hidden or stealthy. Camera mode always shows a persistent notification and a clear on-screen indicator that it is active. Do not build any hidden-camera or disguised mode. This prevents misuse as spyware and keeps the app within Google Play policies.
9. **Security:** every piece of data is private to its owner. Firestore and Storage security rules must allow a user to access **only their own** devices, events, and files. Every Cloud Function must check who is calling it.

---

## 6. Build phases

Each phase has a goal, what to build, and a **"Done when"** checklist that I test on my real phones.

### Phase 0: Set up my computer and run a first app

- Detect my operating system and check what is already installed.
- Guide me through installing: Git, Flutter SDK, Android Studio (with Android SDK and command-line tools), Node.js (needed for Firebase tools), and VS Code (optional, for viewing code). Run `flutter doctor` until the Android parts show no problems.
- Teach me to enable Developer options and USB debugging on both phones, connect them by USB, and confirm with `flutter devices`.
- Decide the app name and package ID (e.g. `com.<myname>.<appname>`). Explain that the package ID can't be changed after publishing.
- Create the Flutter project with a clean feature-first folder structure, lint rules, `.gitignore` (confirm secrets are excluded), CLAUDE.md, PROGRESS.md, and docs/SETUP.md. Initialise Git and make the first commit.
- Recommend creating a **private** GitHub repository as a backup, and guide me through it.
- Show a simple home screen with the app name on both phones.

**Done when:** the app opens on both phones, and CLAUDE.md contains the exact command I use to run it again.

### Phase 1: Accounts and device setup

- Guide me to create a Firebase project on the free Spark plan, add the Android app (including SHA-1 and SHA-256 fingerprints for Google sign-in), and connect it with the FlutterFire CLI.
- Sign-in with Google and with email/password; sign-out; password reset.
- A short onboarding (about 3 screens) explaining how the app works.
- Mode selection screen: "Use this phone as a Camera" / "Use this phone as a Viewer", with a one-line explanation of each.
- Camera mode registers the device in Firestore: editable name (e.g. "Living room"), phone model, Android version, battery %, charging status, online status, last seen.
- Viewer mode shows a list of my cameras with online/offline status and battery; I can rename or remove a camera.
- Firestore security rules so users can only access their own data, with rules tests using the Firebase Emulator.

**Done when:** after signing in with the same account on both phones, the camera appears on the viewer's list within a few seconds, and shows "offline" within about 2 minutes of the camera app closing.

### Phase 2: Live streaming

- **2a (spike):** solve gotcha 5.1 (shared camera source). Report findings to me in plain words before continuing.
- WebRTC live view: the camera prepares its local stream; when the viewer taps a camera, a session is created and offer/answer/ICE candidates are exchanged through Firestore. Handle reconnection, camera offline, timeouts, and cleanup of old signaling documents.
- Viewer live screen: full screen, landscape support, connection status, mute/unmute camera audio, SD/HD toggle, and a snapshot button that saves to the phone gallery.
- Support at least 2 viewers watching the same camera at once, or explain the limits.
- **2b:** set up TURN (see Section 4), then test with the viewer on mobile data and the camera on home Wi-Fi.

**Done when:** live video starts within about 3 seconds on the same Wi-Fi **and** on mobile data, delay is under about 1 second, and the stream recovers by itself after I turn the camera phone's Wi-Fi off and on.

### Phase 3: Always-on camera

- Foreground service with persistent notification, wake lock, battery-optimization request, and a help screen for aggressive phone brands.
- Optional auto-start after reboot (a toggle in settings).
- Dim-screen mode.
- Heartbeat every ~60 seconds; the viewer shows last-seen time.
- Alerts to the viewer for low battery, not charging, and overheating.

**Done when:** the camera phone runs overnight (8+ hours) with the screen dimmed and still streams live in the morning.

### Phase 4: Motion detection and alerts

- **Before this phase**, tell me that Cloud Functions (and, for new projects, Cloud Storage) need the pay-as-you-go Blaze plan. Help me upgrade and set a **budget alert** (e.g. ₹200/month) first. Explain the expected cost at my small testing scale.
- Motion detection by comparing low-resolution grayscale frames. Sensitivity setting (Low / Medium / High). Cooldown between alerts (default 60 seconds). Ignore the first few seconds after the camera starts.
- On motion: capture a snapshot → upload to Storage → create an event in Firestore → a Cloud Function sends a push notification with the thumbnail to all my viewer phones. Tapping the notification opens that event.
- From the viewer: turn detection on/off per camera, and set a schedule (e.g. only 10 PM – 6 AM).
- Unit tests for the motion-detection logic.

**Done when:** walking in front of the camera produces a notification on the viewer within about 5 seconds, and a slightly moving curtain on Low sensitivity does not trigger alerts constantly.

### Phase 5: Recording and event history

- When motion is detected, record a clip (default 20 seconds, adjustable 10–60 seconds) from the shared camera source, compress it, and upload it. Use a retry queue if the internet drops.
- Viewer events timeline: grouped by day, thumbnails, filter by camera, play the clip, download to gallery, share, delete.
- Auto-delete clips older than 7 days using a scheduled Cloud Function, to control storage cost. Show how much storage is used.

**Done when:** I can open and play yesterday's clip, and clips older than 7 days disappear automatically.

### Phase 6: Remote controls

- Two-way talk: a push-to-talk button on the viewer; my voice plays on the camera phone's speaker.
- Siren: a loud alarm on the camera phone that stops after 30 seconds or when I press stop.
- Switch front/back camera and turn the flashlight on/off remotely (if the phone supports it).
- Send commands over the WebRTC data channel when connected, and through Firestore when not.

**Done when:** every control works from the viewer while it is on mobile data.

### Phase 7: Polish, security, and release

- Design pass: consistent theme, light and dark mode, app icon, splash screen, empty states, friendly error messages, large tap targets, readable text.
- Security review of the whole project: Firestore and Storage rules, auth checks in every Cloud Function, no secrets in code or Git history, outdated or vulnerable dependencies. (This is a good task for a dynamic workflow.)
- Performance check on my oldest phone.
- A plain-language privacy policy (what data is collected, where it's stored, how long, how to delete it) and in-app account deletion.
- Release: create the upload signing key and explain how to back it up safely (losing it is a serious problem), set up a Google Play Console developer account (tell me the fee first), publish to the internal testing track, and help me write the store listing and fill in the Data safety form.

**Done when:** the app installed from Play internal testing works end-to-end on both phones.

### Future ideas (do NOT build unless I ask)

Premium subscription with Google Play Billing, person/pet detection with on-device ML, motion detection zones, multi-camera grid view, iOS version, web viewer, Hindi and Marathi translations.

---

## 7. Code quality rules

- Feature-first folders: `lib/features/` (auth, camera, viewer, events, settings) and `lib/core/` (services, models, theme, utils).
- Small files, clear names, and short comments explaining **why** in simple words.
- Handle errors everywhere. The app must never crash because the internet drops.
- Run `flutter analyze` and the tests before every commit, and fix warnings.
- Commit after each working step with a clear message. Never leave the main branch broken.
- Don't add a package without telling me why. Prefer popular, well-maintained packages.
- Use a debug logger that is off in release builds. Never log video, images, locations, or personal data.
- Never commit service account keys, keystores, passwords, or other secrets.

---

## 8. Costs and accounts I may need

Always tell me **before** any step that could cost money. Expected items:

- Claude subscription (I already have this).
- Firebase: free Spark plan to start; Blaze (pay-as-you-go, needs a card) from Phase 4. Set a budget alert before upgrading.
- TURN server: a small VPS or managed TURN service (from Phase 2b); compare options for me.
- Google Play Console developer account: one-time fee (Phase 7).

---

## 9. Start now

In your first reply:

1. Confirm you have read this entire file by summarising the project in 5 short bullets.
2. List anything in this plan that is unclear, outdated, or risky, with your suggested fix.
3. Detect my operating system and check which tools are already installed.
4. Begin Phase 0, step 1.
