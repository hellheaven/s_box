# AGENTS.md

## What this is

Design assets + a hardware test/debug page for a **数字证书柜 (Digital Certificate Cabinet)** — an Android touchscreen kiosk with motorized locks, RFID tags, and serial-port peripherals.

## Repo layout

```
resource/
  lock.html            -- JS→Android bridge test harness (WebView-based)
  数字证书柜-原型/      -- UI mockups (10 PNG screens: login, face rec, cert store/retrieve, etc.)
```

No source code, build system, tests, CI, or package manager files exist here.

## Key facts about `resource/lock.html`

- **Character encoding**: Uses `gb231ian` (`<meta charset="gb2312">`). Opening in a browser without gb2312 support will show garbled Chinese.
- **Must run in Android WebView** — most features require `window.jsInterface.invokeMethods()`, which is provided by the native Android app.
- **External JS dependencies** loaded from `js/` (relative, not present in repo): `jquery-3.4.1.min.js`, `ise-fe-audio.js`, `isc-fe-common-android.js`.
- **Serial port**: Hardcoded to `/dev/ttyS2`.
- **Heartbeat**: Sends `heartBeatPongToAndroidClient` every 10s to prevent the Android client from restarting the WebView when offline.
- **Callback naming convention**: Android invokes JS callbacks by action name + `Callback` suffix (e.g., `openLockCallback`, `getLockStCallback`, `reportTagStateCallback`, `reportDoorLockStateCallback`).
- **Author** ("CaiLin") spans 2020–2024; code may have stale assumptions.

## PNG mockups

Reference images only. Located in `resource/数字证书柜-原型/`. ~28MB total.
