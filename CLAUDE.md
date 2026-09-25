# CLAUDE.md

Project memory for Claude. Keep under ~150 lines.

## Start of every session

1. Read `CLAUDE.md` (this file), `PROGRESS.md`, and `MASTER_PROMPT.md`.
2. Tell the owner in 3 lines where we are and what's next.
3. Follow the working rules in `MASTER_PROMPT.md` section 1 (beginner owner, one task at a time, phase by phase).

## What this is

An Android app (Flutter) that turns a spare phone into a home security camera.
One app, two modes: **Camera** (spare phone) and **Viewer** (main phone).
Full plan: `MASTER_PROMPT.md`. Plan check with current docs: `docs/PLAN-REVIEW.md`.

## Key decisions

| Topic | Decision | Status |
| --- | --- | --- |
| Framework | Flutter + Dart, Android first | fixed by plan |
| State / navigation | Riverpod / go_router | fixed by plan |
| Backend | Firebase (Auth, Firestore, Functions TS, FCM, Storage, Crashlytics), region near India | fixed by plan |
| Live video | WebRTC (`flutter_webrtc`), Firestore signaling, TURN with short-lived credentials | fixed by plan |
| Where Claude works | cloud session vs owner's computer | **not decided** |
| App name / package ID | — | **not decided** |
| Minimum Android version | — | decide in Phase 0 |

## Environment notes

- Cloud sessions run in a temporary Linux container. It is wiped later, so everything important must be committed and pushed.
- Cloud sessions cannot see the owner's phones over USB.

## Commands

_Not set yet. Add the exact run/build/test commands once the Flutter project exists._

## Coding rules (short version of MASTER_PROMPT.md section 7)

- Feature-first folders: `lib/features/…`, `lib/core/…`.
- Run `flutter analyze` and tests before every commit.
- No secrets in the repo (keystores, service-account keys, passwords, TURN secrets).
- Tell the owner before adding a package, and before anything that costs money.
- Debug logger off in release. Never log video, images, locations, or personal data.
