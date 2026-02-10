# Gorilla-Tag-Quest-Mods
# Gorilla Tag Quest 3 Modloader (offline toolkit)

> Private/educational use only. Expect bans on public lobbies. Every Gorilla Tag patch may break this.

## What this repo provides
- `scripts/quest-modloader.sh`: end-to-end patcher that pulls the live Quest APK from a connected headset, injects the BepInEx IL2CPP shim, rebuilds, signs, installs, and pushes the BepInEx payload + doorstop config.
- `config/doorstop_config.ini`: default Doorstop/BepInEx preloader config tuned for IL2CPP on Android.
- `plugins/UtilitaPort/`: minimal BepInEx + Utilla plugin stub to gate features to modded rooms; use as a starting point to port Utilita or other menus.

## Requirements
- Host: Linux/WSL/macOS with `adb`, `java`, `curl`, `unzip`, `python3`, Android build-tools (`zipalign`, `apksigner`), and `keytool`.
- Device: Quest 3 with Gorilla Tag installed (same version you want to patch). USB debugging enabled.
- BepInEx IL2CPP Android build zipped as `vendor/BepInEx_UnityIL2CPP_Android_ARM64.zip` (or set `BEPINEX_URL` to a direct download URL so the script can fetch it).

## Quick start
```bash
# 1) Connect Quest 3 via USB, enable debugging.
# 2) Place the BepInEx IL2CPP Android zip in vendor/ OR export BEPINEX_URL to download automatically.
# 3) Run the pipeline (downloads apktool automatically):
./scripts/quest-modloader.sh
# Outputs: build/gtag-modded.apk and installs it on the headset.
```

## Script knobs (env vars)
- `APP_ID` (default `com.AnotherAxiom.GorillaTag`)
- `WORKDIR` (default `./build`)
- `BEPINEX_URL` (override to a specific BepInEx IL2CPP Android build)
- `BEPINEX_ZIP` (use a local zip instead of downloading)
- `OUTPUT_APK` (default `build/gtag-modded.apk`)
- `PUSH_BEPINEX` (set `0` to skip pushing payload after install)

## How it works (abridged)
1) Pulls `base.apk` (and OBB if present) from the device.
2) Decodes with apktool and forces `android:extractNativeLibs="true"` so the shim can replace `libil2cpp.so`.
3) Replaces `lib/arm64-v8a/libil2cpp.so` with BepInEx's shim, preserving the original as `libil2cpp.so.orig`.
4) Drops `doorstop_config.ini` into `assets/`.
5) Rebuilds, zipaligns, signs with a local debug keystore, and installs.
6) Pushes the BepInEx payload + doorstop config to `/sdcard/Android/data/<app>/files/` so the shim can find it at runtime.

## Porting Utilita/menus
- Place Gorilla Tag IL2CPP dump + Utilla + UnityEngine DLLs in a folder and point `GTAG_DLLS` to it when building `plugins/UtilitaPort/UtilitaPort.csproj`.
- Add your menu code under `ToggleFeatures()`; keep everything disabled when `!e.isModded` to avoid server-side flags.
- Build with `dotnet build plugins/UtilitaPort/UtilitaPort.csproj` and drop the resulting DLL into `/sdcard/Android/data/<app>/files/BepInEx/plugins/`.

## Ban/safety notes
- Use **modded/private** lobbies only; public lobbies are near-certain bans.
- Every Gorilla Tag update replaces `libil2cpp.so`/`global-metadata.dat`; rerun the pipeline after each patch.
- Do not redistribute signed APKs; share patch scripts instead.

## Troubleshooting
- "Failed to locate base.apk": make sure Gorilla Tag is installed and visible to `adb shell pm path com.AnotherAxiom.GorillaTag`.
- "Missing zipalign/apksigner": install Android SDK build-tools and ensure they’re on PATH.
- Black screen on launch: likely wrong BepInEx build; override `BEPINEX_URL`/`BEPINEX_ZIP` with the correct IL2CPP Android package for the game’s Unity version.
- App won’t install: uninstall the official build first, or sign with the same key you used previously (Quest side-by-side installs require matching signatures).

## Heads-up on legality
Modifying the APK breaches the game’s TOS and Meta’s distribution terms. Proceed at your own risk; you’re responsible for account/device bans or data loss.
