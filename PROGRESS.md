# Progress

Status key: ✅ done · 🔄 in progress · ⬜ not started · ⏸ waiting for owner

_Last updated: 2026-09-25 (session 1)_

## Next step

⏸ Owner to answer the Phase 0 step 1 questions (which computer, how we work, phone details, app name).

## Open issues

- The GitHub repo `mohnish-cyber/Security-Cam` is **public**. The plan wants it **private**. Switch it before we add Firebase files.
- The "Fill these in" section of `MASTER_PROMPT.md` is still blank.
- Claude is running in a cloud computer, not on the owner's PC. We must pick how to work (see `docs/PLAN-REVIEW.md`, section 1).

## Phase 0: Set up the computer and run a first app 🔄

- ✅ Read the master plan and check it against current official docs → `docs/PLAN-REVIEW.md`
- ⏸ Step 1: find out the owner's computer OS and what is installed
- ⬜ Install Git, Flutter, Android Studio (SDK + command-line tools), Node.js, VS Code (optional)
- ⬜ `flutter doctor` shows no Android problems
- ⬜ Developer options + USB debugging on both phones; `flutter devices` sees them
- ⬜ Choose app name and package ID
- ⬜ Create the Flutter project (folders, lints, `.gitignore`, memory files) and first commit
- ⬜ Private GitHub repo as backup
- ⬜ Home screen with the app name runs on both phones
- ⬜ **Done when:** app opens on both phones; CLAUDE.md has the exact run command

## Phase 1: Accounts and device setup ⬜

- ⬜ Firebase project (Spark plan), Android app, SHA-1/SHA-256, FlutterFire CLI
- ⬜ Google + email/password sign-in, sign-out, password reset
- ⬜ Onboarding (3 screens)
- ⬜ Mode selection (Camera / Viewer)
- ⬜ Camera registers itself in Firestore (name, model, Android version, battery, charging, online, last seen)
- ⬜ Simple heartbeat so the viewer can show "offline" (needed for this phase's "Done when")
- ⬜ Viewer camera list (online/offline, battery, rename, remove)
- ⬜ Firestore rules + emulator tests
- ⬜ **Done when:** camera shows on viewer in seconds; shows offline within ~2 min of closing

## Phase 2: Live streaming ⬜

- ⬜ 2a spike: one shared camera source for live view, motion and recording
- ⬜ WebRTC live view with Firestore signaling, reconnect, cleanup
- ⬜ Live screen: full screen, landscape, status, mute, SD/HD, snapshot
- ⬜ 2+ viewers at once (or documented limits)
- ⬜ 2b: TURN server (compare options, owner chooses)
- ⬜ **Done when:** video in ~3 s on Wi-Fi and mobile data, <1 s delay, recovers after Wi-Fi off/on

## Phase 3: Always-on camera ⬜

- ⬜ Foreground service, wake lock, battery-optimisation request, phone-brand help screen
- ⬜ Optional auto-start after reboot
- ⬜ Dim-screen mode
- ⬜ Heartbeat every ~60 s; viewer shows last seen
- ⬜ Low battery / not charging / overheating alerts
- ⬜ **Done when:** runs overnight (8+ h) dimmed and still streams in the morning

## Phase 4: Motion detection and alerts ⬜

- ⬜ Blaze plan + budget alert (owner, with cost warning first)
- ⬜ Motion detection (grayscale frames, sensitivity, cooldown, warm-up)
- ⬜ Snapshot → Storage → Firestore event → Cloud Function push with thumbnail
- ⬜ Detection on/off and schedule per camera
- ⬜ Unit tests for motion logic
- ⬜ **Done when:** alert within ~5 s; moving curtain on Low doesn't spam

## Phase 5: Recording and event history ⬜

- ⬜ Clip recording (default 20 s) from shared source, compress, upload with retry queue
- ⬜ Events timeline (by day, thumbnails, filter, play, download, share, delete)
- ⬜ Auto-delete after 7 days; show storage used
- ⬜ **Done when:** yesterday's clip plays; 7-day-old clips vanish

## Phase 6: Remote controls ⬜

- ⬜ Push-to-talk, siren, switch camera, flashlight
- ⬜ Commands over data channel, Firestore fallback
- ⬜ **Done when:** all controls work with viewer on mobile data

## Phase 7: Polish, security, release ⬜

- ⬜ Design pass, security review, performance on oldest phone
- ⬜ Privacy policy + in-app account deletion
- ⬜ Signing key (backed up), Play Console, internal testing, store listing, Data safety
- ⬜ **Done when:** Play internal-testing build works end-to-end on both phones
