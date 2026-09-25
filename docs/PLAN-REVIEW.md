# Plan review

Checked on 2026-09-25 against official docs, pub.dev, GitHub source code and Google/Firebase pricing pages.
Money: ₹96 = 1 US dollar (ECB rate, 24 Sep 2026).

**Summary.** The plan is sound: Flutter, Firebase and WebRTC are still the right choices, and all named packages are alive and current. The main changes are: (1) this Claude session runs in the cloud, not on your computer; (2) the Blaze paid plan is needed from Phase 2b, not Phase 4; (3) "auto-start after reboot" cannot work on newer Android phones; and (4) the camera phone should keep its screen on (dimmed), not off.

> **How sure are these facts?** Sections 1–5 were each researched by one agent that fetched every source listed. A second "try to disprove it" check did not run yet (usage limit), and three topics (TURN servers, other app packages, Play Store and Indian law) are still to do. See section 6.

---

## 1. Fix before we start (Phase 0–1)

### 1.1 Claude is running in the cloud, not on your computer
- This session runs on a temporary Linux computer in the cloud (Ubuntu 24.04). It **cannot** install programs on your PC, see your phones over USB, or run the app on them.
- The plan's Phase 0 assumes Claude runs on your computer.
- **Option A (recommended): run Claude Code on your own computer.** This matches the plan. You get USB debugging, live error logs from the phone, and fast "hot reload" (see a code change on the phone in about a second). This matters a lot for the hard parts (camera, WebRTC, background running).
- **Option B: stay in the cloud.** I write the code and build the app file (APK = the Android install file) in the cloud or on GitHub. You download it and install it on each phone. Nothing to install on your PC, but every test is slower, and we can't see the phone's logs live. We'd need a fixed signing key kept in encrypted secrets, so Google sign-in keeps working.

### 1.2 Your computer must be strong enough (Option A)
- Android Studio needs 64-bit Windows 10+ or macOS 12+, at least 8 GB RAM (16 GB is better), and a lot of disk space. Budget **about 30 GB free** on the main drive. Windows PCs with ARM chips are not supported. [Android Studio install](https://developer.android.com/studio/install)
- With 8 GB RAM, we test on real phones only (no phone emulator) and give the build less memory.

### 1.3 The install list needs updating
- **Flutter 3.47.5** (Dart 3.13.4) is the current stable version. [releases](https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json)
- The easiest official route is **VS Code + the Flutter extension**, which downloads Flutter for you. So VS Code is not really optional. [Flutter quick install](https://docs.flutter.dev/install/quick)
- In Android Studio's SDK Manager, also install: Android API 36 platform, Command-line Tools, NDK, CMake. On Windows, the phone maker's USB driver may be needed. [Android setup](https://docs.flutter.dev/platform-integration/android/setup)
- **Node.js 24 LTS** (not 20, which is end-of-life; not 26 yet). [Node schedule](https://raw.githubusercontent.com/nodejs/Release/main/schedule.json), [Cloud Functions runtimes](https://cloud.google.com/functions/docs/runtime-support)
- **Java 21** is needed by the Firebase Emulator (used for security-rules tests in Phase 1). Android Studio's built-in Java can be used. [firebase-tools v15](https://github.com/firebase/firebase-tools/releases/tag/v15.0.0)
- Don't accept Android Studio's "upgrade Android Gradle Plugin" offer. Flutter pins its own build tool versions. [Flutter gradle_utils](https://raw.githubusercontent.com/flutter/flutter/3.47.5/packages/flutter_tools/lib/src/android/gradle_utils.dart)

### 1.4 Minimum Android version: **Android 7.0 (API 24)**
- Flutter 3.47 supports Android 7.0 and newer only. Google sign-in (google_sign_in_android) also needs 24. So 24 is the floor. [Flutter supported platforms](https://docs.flutter.dev/reference/supported-platforms)
- **Both phones must run Android 7.0 or newer.** A spare phone on Android 6 or older cannot be the camera.

### 1.5 The GitHub repo is public
- `mohnish-cyber/Security-Cam` is **public**. The plan asks for private. Switch it before we add Firebase files or personal details.

### 1.6 Phase 1 needs the heartbeat early
- Phase 1's test ("shows offline within ~2 minutes") needs a heartbeat, but the plan builds it in Phase 3. We'll build a simple one in Phase 1: the camera writes "last seen" every ~60 s, and the viewer shows "offline" after 2 minutes of silence.

### 1.7 Google sign-in has a new API
- google_sign_in **7.x** changed how sign-in works (`initialize()` once, then `authenticate()`). Old tutorials are wrong. [changelog](https://raw.githubusercontent.com/flutter/packages/main/packages/google_sign_in/google_sign_in/CHANGELOG.md), [FlutterFire docs](https://firebase.google.com/docs/auth/flutter/federated-auth)
- Firebase needs the SHA-1/SHA-256 "fingerprint" (a short ID of the key that signs the app) of **every** signing key: debug key, a fixed test key, and in Phase 7 the upload key **and** Google Play's app-signing key. Missing one = sign-in fails with a confusing "canceled" error. [client auth](https://developers.google.com/android/guides/client-auth)

### 1.8 Password reset still works
- Firebase Dynamic Links shut down in Aug 2025, but password-reset emails still work. For privacy, Firebase no longer says whether an email has an account, so the app should say "If an account exists, we've emailed a link." [FAQ](https://firebase.google.com/docs/dynamic-links/dynamic-links-deprecation-faq)

---

## 2. Changes to the plan by phase

### Phase 2: Live streaming
- **Blaze plan is needed here, not in Phase 4.** The TURN-credentials Cloud Function can't be deployed on the free Spark plan. Cloud Storage now needs Blaze for **every** project (since 3 Feb 2026). → Move "upgrade to Blaze + budget alert" to the start of Phase 2b. [Functions](https://firebase.google.com/docs/functions/get-started), [Storage changes](https://firebase.google.com/docs/storage/faqs-storage-changes-announced-sept-2024)
- **flutter_webrtc 1.6.2+hotfix.3 is current and active.** It is ready for Play's 16 KB rule. Pin the exact version; releases come fast. [pub.dev](https://pub.dev/packages/flutter_webrtc)
- **Shared camera (gotcha 5.1), spike plan:**
  - `captureFrame()` makes a full-size JPEG each call. Fine for the snapshot button, **too slow for motion detection**.
  - Best option: a **small Kotlin (native Android) add-on** that attaches to the camera track inside flutter_webrtc and checks a tiny grayscale frame 1–2 times per second. Only a "motion score" goes back to Flutter.
  - flutter_webrtc's recorder works, but it records at 1–6 Mbps with no setting (a 20 s clip at 720p ≈ 15 MB). It may record nothing if audio is on and no viewer is connected. Test both in the spike.
- **Old phones:** on Android 7–9 with MediaTek/Unisoc chips (common in India), video encoding runs on the CPU (software VP8). Capture at 640×480 or 720p at 10–15 fps, never the default 720p at 30 fps. [encoder source](https://raw.githubusercontent.com/webrtc-sdk/webrtc/m150_release/sdk/android/api/org/webrtc/HardwareVideoEncoderFactory.java)
- **SD/HD toggle:** the camera size can't change while running. Capture once at 720p, then send a smaller or larger stream per viewer (≈225 MB/hour at SD, 500 kbps).
- **2 viewers = 2 separate encodings** on the old phone. Limit to 2 viewers; the second gets SD. Test for heat.
- **Camera errors are silent:** flutter_webrtc only writes camera failures to the log. Add a "no frames for 10 s → restart camera, then alert viewer" check.
- **Pre-roll (seconds before motion) isn't possible** with built-in tools. v1 clips start when motion is detected (about 1–2 s late). A 3–5 s pre-roll is a Phase 5 stretch goal.

### Phase 3: Always-on camera
- **Keep the screen on, dimmed, as the default.** Reasons:
  - WebRTC lives inside the app's main screen. If that screen is closed (Back or swipe away from recent apps), the camera stops, even with the notification still showing.
  - Since March 2026, Google Play warns users about apps that hold a "partial wake lock" (keeps the CPU awake) for 2+ hours with the screen off. Screen-on time doesn't count. [Play vitals](https://developer.android.com/google/play/vitals/excessive-wakelock)
- **flutter_foreground_task 11.0.3** is current and a good choice. Keep all camera work in the main app; the service only keeps the app alive. [pub.dev](https://pub.dev/packages/flutter_foreground_task)
- **"Auto-start after reboot" must change** to "**Resume after reboot**": after a reboot, show a "Tap to resume camera" notification and tell the viewer "camera rebooted, needs a tap". Real auto-start only works on Android 10 and older. [Android 15 changes](https://developer.android.com/about/versions/15/behavior-changes-15), [background start rules](https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start)
- Turn **off** the plugin's own auto-restart options, because a restart from the background can't reopen the camera on Android 11+.
- **Privacy switches:** on Android 12+, if someone turns off the camera/mic switch in quick settings, the app gets black video with no error. Detect long black runs and alert the viewer.
- **Overheating:** battery_plus has no temperature. Read battery temperature from Android directly (works on all versions), e.g. warn at ~45 °C.
- **Phone-brand help screen:** dontkillmyapp.com is out of date for HyperOS/ColorOS. We'll write steps and check them on your real phones. Show it on the **viewer** phone too, otherwise alerts may not arrive.
- Test one night **on battery** too (power cuts are when "Doze" battery saving kicks in).

### Phase 4: Motion alerts
- **Notification thumbnail:** the phone downloads the image without logging in, so the Cloud Function must create a short-lived (1 hour) signed link. That needs one extra permission on Google Cloud. Never make the storage public. [FCM image](https://firebase.google.com/docs/cloud-messaging/android/send-image)
- Send alerts as **high-priority notifications** and add a "Send test alert" button.

### Phase 5: Recording
- **Storage in Mumbai has no free tier** (free tier is US regions only). Mumbai costs about ₹1.9 per GB per month, plus ~₹11.5 per GB when clips are downloaded. You choose: Mumbai (data stays in India, faster) or US (small free tier). This can't be changed later. [Storage pricing](https://cloud.google.com/storage/pricing), [locations](https://firebase.google.com/docs/storage/locations)
- **Deleted files are kept (and billed) 7 more days** by default ("soft delete"). Turn it off on the clips bucket and use a built-in "delete after 7 days" rule instead of only a function. [soft delete](https://cloud.google.com/storage/docs/soft-delete)
- Record small: e.g. 640×360 at ~600 kbps ≈ 1.5 MB per 20 s clip, instead of compressing afterwards.

### Phase 6: Remote controls
- All needed features exist: data channel, camera switch, torch, ICE restart.
- Two-way talk: ask for mic permission **before** the first connection (known bug), use push-to-talk that mutes the camera's mic while you speak (avoids echo), force speakerphone.
- Android 17 blocks background sound unless a visible screen or foreground service is running. The siren must play only while camera mode is running.

### Phase 7: Release
- Add Play Console "foreground service" declarations (camera, microphone) with a short demo video, and a one-line "safety app" reason for the battery exemption. [Play FGS declaration](https://support.google.com/googleplay/android-developer/answer/13392821)
- Add **App Check** (free; stops fake apps from using your Firebase). [App Check](https://firebase.google.com/docs/app-check)
- Build account deletion as a callable Cloud Function (the old "user deleted" trigger doesn't exist in the functions type we must use in Mumbai).
- Target API 36 (Android 16) is required on Play since 31 Aug 2026; Flutter already uses it. UI must work edge-to-edge and with the new "predictive back". [target SDK](https://developer.android.com/google/play/requirements/target-sdk)
- Android developer verification reaches India in 2027. USB installs are not affected. [developer verification](https://developer.android.com/developer-verification)

---

## 3. Costs

| When | What | Expected cost |
| --- | --- | --- |
| Phase 0–2a | Firebase Spark (free) | ₹0 |
| Phase 2b → | Firebase Blaze (needs a card) | Under ~₹50/month for 1 camera at test scale. $300 free credit may be offered. |
| Phase 2b → | TURN server | **Not checked yet** (see section 6) |
| Phase 5 → | Clip storage in Mumbai | ~₹1.9/GB/month + ~₹11.5/GB downloaded; a few GB ⇒ roughly ₹5–30/month |
| Phase 7 | Play Console developer account | One-time fee, **not checked yet** |

Important about money:
- **A budget alert does not stop spending.** It only emails you. Google's "auto-disable billing" trick can delete your data, so we won't use it. We'll set alerts at 50/90/100% and keep costs low by design. [budgets](https://cloud.google.com/billing/docs/how-to/budgets)
- **India billing rules:** a Cloud billing account with an Indian address must pass an **identity check within 30 days**, or it can be suspended. Use a card that supports e-mandates (recurring auto-debit). [billing account](https://cloud.google.com/billing/docs/how-to/create-billing-account)
- Hidden deploy costs (Cloud Build, Artifact Registry) are tiny at our scale; we'll set the auto-cleanup policy after the first function deploy.

---

## 4. Current versions (checked 2026-09-25)

| Name | Version | Source |
| --- | --- | --- |
| Flutter (stable) / Dart | 3.47.5 / 3.13.4 | [releases](https://storage.googleapis.com/flutter_infra_release/releases/releases_linux.json) |
| Flutter default minSdk / targetSdk | 24 / 36 | [supported platforms](https://docs.flutter.dev/reference/supported-platforms) |
| Android Studio | Quail 4 (2026.1.4) Patch 1 | [install](https://developer.android.com/studio/install) |
| Node.js LTS | 24.21.0 | [nodejs.org](https://nodejs.org/dist/index.json) |
| firebase-tools | 15.31.0 (Java 21 for emulator) | [npm](https://registry.npmjs.org/firebase-tools/latest) |
| firebase-functions / firebase-admin | 7.4.0 / 14.5.0 (use 13.x for tests for now) | [releases](https://github.com/firebase/firebase-functions/releases/tag/v7.0.0) |
| flutterfire_cli | 1.4.1 | [pub.dev](https://pub.dev/packages/flutterfire_cli) |
| firebase_core / firebase_auth / cloud_firestore | 4.15.0 / 6.7.0 / 6.10.0 | [pub.dev](https://pub.dev/packages/firebase_core) |
| firebase_messaging / firebase_storage / firebase_crashlytics / cloud_functions | 16.7.0 / 13.6.0 / 5.4.0 / 6.5.0 | [pub.dev](https://pub.dev/packages/firebase_messaging) |
| google_sign_in | 7.2.0 | [pub.dev](https://pub.dev/packages/google_sign_in) |
| flutter_webrtc | 1.6.2+hotfix.3 | [pub.dev](https://pub.dev/packages/flutter_webrtc) |
| flutter_foreground_task | 11.0.3 | [pub.dev](https://pub.dev/packages/flutter_foreground_task) |
| flutter_riverpod / riverpod_generator | 3.4.3 / 4.0.9 | [pub.dev](https://pub.dev/packages/flutter_riverpod) |
| go_router | 18.0.1 (Flutter Favorite, by flutter.dev) | [pub.dev](https://pub.dev/packages/go_router) |
| permission_handler | 13.0.2 | [pub.dev](https://pub.dev/packages/permission_handler) |
| gal (save to gallery) | 2.3.3 | [pub.dev](https://pub.dev/packages/gal) |
| flutter_local_notifications | 22.3.1 | [pub.dev](https://pub.dev/packages/flutter_local_notifications) |
| wakelock_plus / screen_brightness / battery_plus | 1.8.0 / 2.1.11 / 7.1.1 | [pub.dev](https://pub.dev/packages/wakelock_plus) |

We will add packages with `flutter pub add` at build time, so they get the newest version, not these numbers from memory.

---

## 5. Things checked and fine

- Flutter + Dart, Riverpod (now 3.x), go_router: current and maintained.
- flutter_webrtc: active, sponsored, works on Android 7+, ready for Play's 16 KB rule.
- Firebase in Mumbai (asia-south1): Firestore, Storage, Scheduler and Cloud Functions (2nd gen) all work there.
- Firestore free daily quota (50k reads, 20k writes) also applies in Mumbai. A 60 s heartbeat is 1,440 writes per camera per day: fine.
- Camera and microphone foreground services have no time limit.
- Doze (Android's deep sleep) is off while the phone charges.
- LiveKit (a video server) is not needed for 1–2 viewers.

---

## 6. Still to check

These are needed later (Phase 2b and Phase 7), not now. The research hit a usage limit and will be finished in a later session:

- **TURN servers:** Cloudflare, Metered, Twilio and similar managed services vs a small server (VPS = virtual private server) in India. Prices, free allowances and effort (for Phase 2b).
- **Other packages:** localization setup, and a final check of every package's minimum Android version.
- **Play Store and law:** developer fee in ₹, new-account testing rules, spyware/stalkerware policy, India's data-protection law (DPDP) (for Phase 7).
- **Second check** of sections 1–5 by an agent trying to disprove them.
