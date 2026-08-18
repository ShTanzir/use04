# DiaBo Preview Template

This is **not** the DiaBo app itself — it's the lightweight Android project that
DiaBo's "▶ Real Build" feature targets via GitHub Actions `workflow_dispatch`.

## How it fits together
1. User taps "▶ Real Build" in the DiaBo app.
2. The app base64-encodes the current `MainActivity.java` + `activity_main.xml`
   and calls `POST /repos/{owner}/DiaBoPreviewTemplate/actions/workflows/diabo-preview-build.yml/dispatches`
   with those as inputs, plus a unique `build_id`.
3. `.github/workflows/diabo-preview-build.yml` injects the code, runs
   `./gradlew assembleDebug`, boots an emulator, installs the APK, screenshots it.
4. The DiaBo app polls `GET /repos/{owner}/DiaBoPreviewTemplate/actions/runs`,
   finds the matching run, then downloads the `apk-{build_id}` and
   `screenshot-{build_id}` artifacts once the run completes.

## Setup
1. Create a **public** GitHub repo (public = free unlimited Actions minutes;
   private works too but consumes your monthly quota) and push this folder's
   contents to it.
2. Generate a **fine-grained GitHub Personal Access Token** scoped to:
   - This repo only
   - `Actions: Read and write`
   - `Contents: Read` (for artifact download)
3. Paste that token + `owner/repo` into DiaBo's Settings → GitHub Integration.

## Security notes
- The workflow only ever writes into `MainActivity.java` and `activity_main.xml`
  — nothing else in the repo is touched by user input.
- A 200KB size guard rejects oversized payloads before Gradle runs.
- `timeout-minutes: 12` and a `concurrency` group prevent runaway or duplicate jobs.
- No secrets are ever passed into the injected-code path.
