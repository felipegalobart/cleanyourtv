# Epson EF-12 (Android TV 14 mini laser projector)

Epson's EF-12 with Askey's Android TV module (board `sti6200d101`, Amlogic), firmware
UKG3.250826.001.20261009 (`u-3.03-U-0`, October 2026) — the sibling of the LS800's HA90 module
([`examples/epson-ls800`](../epson-ls800)), on the same UKG3.250826.001 base. Both halves work: the
debloat list and the Clean Your TV launcher as Home. Applied and reboot-checked on 27 October 2026.

| File | What it is |
|---|---|
| `debloat.txt` | The 6 packages that were disabled, grouped and commented, with what to keep and why (`build-and-install.sh` disables the stock launcher on top) |
| `config.json` | Dock order and hidden tiles; push it with `tools/theme.sh` |

All commands run from the repo root, with the projector's IP in `TV_HOST`.

## Turning ADB on

Settings → Device Preferences → About → press OK on **Build** 7 times, then Developer options →
USB debugging on. This firmware offers **Wireless debugging** (TLS) with a pairing code, and while
USB debugging is on it also runs plain network ADB on port 5555 (`service.adb.tcp.port=5555`):

1. Developer options → Wireless debugging → on → **Pair device with pairing code**.
2. `adb pair <ip>:<pairing port> <code>` with Google's
   [platform-tools](https://developer.android.com/tools/releases/platform-tools).
3. After pairing, port 5555 accepts the paired key with no on-screen prompt, so `tools/tv.py`
   (port 5555) works straight away — no `adb tcpip` needed, and 5555 comes back by itself after a
   reboot while USB debugging stays on.

## Install, from scratch or again

```bash
TV_HOST=<projector-ip> tools/disable.sh examples/epson-ef12/debloat.txt
TV_HOST=<projector-ip> launcher/build-and-install.sh
TV_HOST=<projector-ip> tools/theme.sh examples/epson-ef12
```

Building needs JDK 17 and the Android SDK (`ANDROID_HOME`, platform 34). On Android 14,
`build-and-install.sh` pushes the APK and runs `pm install -r` itself, so no on-screen confirmation
is needed. Private DNS works on this unit (Android 14).

## Undo

```bash
TV_HOST=<projector-ip> tools/restore.sh examples/epson-ef12/debloat.txt com.google.android.tvlauncher
TV_HOST=<projector-ip> tools/theme.sh --reset
```

One app: `TV_HOST=<projector-ip> tools/venv/bin/python tools/tv.py 'pm enable com.google.android.apps.mediashell'`,
with the package name from `debloat.txt`.

## Change the dock or the wallpaper

Edit `dock_apps` in `config.json` (four package names, left to right; `tools/tv.py 'pm list packages'`
lists them), then push it again with `tools/theme.sh examples/epson-ef12`. For a wallpaper, put a
`wallpaper.jpg` (1920x1080) next to `config.json` and push the same way; `themes/README.md` has the
dim and rotation options.

## Boot straight into an HDMI input

With a stick or console as the main source, two keys in `config.json` make every power-on land on
it instead of Home — verified on this unit with a Fire TV stick on HDMI 2, including from a full
power cut (the last-used input survives on disk):

```json
"live_tv_package": "com.droidlogic.android.tv",
"boot_to_live_tv": true
```

The droidlogic HDMI app opens the **last-used** input, not a fixed one, once per boot; Home on the
remote goes back to the launcher. The folder's `config.json` ships without these keys on purpose —
without a source plugged in, every boot would land on a blank input.

## Remote buttons

Home, Back, D-pad, Settings and volume (held, with normal key repeat) work without the stock
launcher, checked on the real remote. Two firmware behaviours, shared with the LS800:

- **YouTube** is hard-coded by Epson's framework to YouTube Music, which forwards to YouTube. Keep
  YouTube Music enabled, or the button opens its Play Store page. It cannot be remapped.
- **Mic / voice search** runs through `com.google.android.katniss`, which the firmware re-enables on
  its own (see `debloat.txt`), so it keeps working whether you want it or not.
- **Apps** (`ALL_APPS`): Android 14 sends it only to a system launcher, so expect it dead with
  tvlauncher disabled, as on the LS800; the new launcher's grid lists every app.

## When you are done

Switch Wireless debugging and USB debugging off in Developer options and revoke the debugging
authorisations. On this firmware, USB debugging off closes port 5555 at once; check it, restart,
and check again:

```bash
tools/venv/bin/python tools/adb-closed.py <projector-ip>
```
