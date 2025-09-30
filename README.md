
# ASIO-ArgentOne

Custom ASIO driver (and dev toolkit) for the **Armer Argent** USB Audio 2.0 interface.

> Goal: deliver a stable, low-latency ASIO device for Windows DAWs, with correct 16/24-bit formats, proper clocking, and clear channel names—without needing OEM software.

---

## Device profile (from USB descriptors)

* **Vendor / Product:** VID `0x2F6E` (SMMMPLUS), PID `0x4E06`
* **Strings:** Manufacturer = “Armer Argent”, Product = “Armer Argent”
* **USB:** 2.0 High-Speed (480 Mb/s), UAC2 (Audio Function 2.0)
* **I/O:** 2-in / 2-out (stereo)
* **Bit depths:** 16-bit and 24-bit (alternate settings)
* **Sample formats:** PCM
* **Endpoints:**

  * IN (capture): `0x81` isochronous, async, 16-bit (`wMaxPacketSize=0x007C`) and 24-bit (`0x00BA`)
  * OUT (playback): `0x04` isochronous, adaptive, 16-bit (`0x007C`) and 24-bit (`0x00BA`)
* **Clocking:** Clock Source units report **host-programmable** frequency
* **Feature Unit:** Mute (master) + Volume (per channel)
* **Power:** Bus-powered, 100 mA

> Inference: Extension Unit `wExtensionCode = 0x0BDA` suggests a Realtek-based codec path. Not required for functionality, but useful for troubleshooting.

---

## Project structure

```
ASIO-ArgentOne/
├─ /src/
│  ├─ asio/           # ASIO SDK wrapper + driver class (no Steinberg code included)
│  ├─ core/           # device enumeration, streaming engine, ring buffers
│  ├─ ks/             # WDM-KS (Kernel Streaming) plumbing for WASAPI/KS path
│  ├─ usb/            # (optional) WinUSB user-mode ISO path experiments
│  └─ util/           # logging, GUIDs, registry helpers
├─ /installer/        # Inno Setup or WiX installer scripts + .reg templates
├─ /tools/            # descriptor dumps, latency tests, debug apps
├─ /docs/             # design notes, troubleshooting, protocol sketches
└─ README.md
```

---

## What we’re building (phased)

1. **ASIO Wrapper (WASAPI/KS path)**

   * Expose an ASIO device that talks to Windows’ `usbaudio2.sys` via WASAPI Exclusive / Kernel Streaming.
   * Fast to ship, stable, and already handles clocking/alt-settings.

2. **Native UAC2 map (KS pin-accurate)**

   * Map UAC2 terminals/feature units to ASIO channels and controls.
   * Proper safety-offset, low buffer sizes, and device-side mute/volume.

3. **(Optional) Direct USB engine (WinUSB)**

   * Bypass class driver for research-grade latency.
   * Requires robust isochronous IN/OUT scheduling and error recovery.
   * Only if we need sub-class-driver latencies—and we accept higher complexity.

---

## Features (current / planned)

* ✅ Detect only **VID 0x2F6E / PID 0x4E06** (avoid hijacking other devices)
* ✅ ASIO device name: **“Argent One (ASIO)”**
* ✅ Stereo in/out with stable channel labels: **In L/R, Out L/R**
* ✅ 16-bit and 24-bit PCM, buffer sizes from 32–2048 frames (host-dependent)
* ✅ Mute/Volume bridged to device Feature Unit (where supported)
* ⏳ Sample-rate set via host (44.1/48/88.2/96/176.4/192 kHz)
* ⏳ Safety offset tuning for popular DAWs (REAPER, Cubase, Live, Studio One)
* ⏳ Installer that registers ASIO under `HKLM\SOFTWARE\ASIO`

---

## Requirements

* **Windows 10/11** (x64)
* **Visual Studio 2022** (Desktop development with C++)
* **Windows SDK + WDK** (matching your OS version)
* **Steinberg ASIO SDK 2.3+**

  * You must download it from Steinberg and accept their license.
  * Set env var `ASIOSDK_DIR` to the SDK root (no SDK files in this repo).

---

## Building

1. Install VS2022, Windows SDK, and WDK.
2. Download the **ASIO SDK**, set `ASIOSDK_DIR` (e.g., `C:\SDKs\ASIO`).
3. Clone this repo and open `ASIO-ArgentOne.sln`.
4. Build **Release x64**.

Artifacts:

* `ArgentOneASIO.dll` (drop-in ASIO driver)
* Optional CLI test tools under `/tools/bin`

---

## Installing (developer mode)

### 1) Place the DLL

Copy the driver to the common ASIO location:

```
C:\Program Files\Steinberg\ASIO\ArgentOneASIO\ArgentOneASIO.dll
```

### 2) Register in the registry

Create a `.reg` with your final paths/CLSID:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\ASIO\Argent One (ASIO)]
"CLSID"="{5A07E2F8-72A5-4F47-8E31-5F3E3C0FB0D3}"
"Description"="Argent One (ASIO)"
"Dll"="C:\\Program Files\\Steinberg\\ASIO\\ArgentOneASIO\\ArgentOneASIO.dll"
"Manufacturer"="Argent"
"Version"="1.0.0"
```

> Replace the CLSID with the one compiled into the driver. We keep the CLSID in `/src/asio/Guids.h`.

### 3) Verify in a DAW

Open your DAW → Audio Device → ASIO → **Argent One (ASIO)**.
Select 16/24-bit and the desired sample rate in the driver panel.

---

## How it works (high level)

* **Enumeration:** we match **USB\VID_2F6E&PID_4E06** and open the **UAC2** pins through KS.
* **Streaming:** ring buffers per direction; double or triple buffering depending on requested block size.
* **Clocking:** we request the rate on the host side; device advertises host-programmable clock.
* **Formats:** we expose **16-bit** and **24-bit** ASIO sample types; channel layout **FL/FR**.
* **Controls:** mute/volume linked to the device Feature Unit when available; falls back to software gain.
* **Latency reporting:** bufferSize / sampleRate + safetyOffset (configured per backend).

---

## Known good settings (starting points)

* **48 kHz, 64-128 samples**: most USB 2.0 HS devices are stable here.
* **96 kHz, 128-256 samples**: fine on modern chipsets.
* **192 kHz**: start at **256-512 samples** and lower as stable.

If you hear crackles, increase the buffer or disable CPU C-States / USB selective suspend for tests.

---

## Troubleshooting

* **Device not listed in DAW**

  * Check registry key under `HKLM\SOFTWARE\ASIO`
  * Ensure `ArgentOneASIO.dll` path is correct and x64 build is used

* **No audio / silence**

  * Confirm the DAW sample rate matches the device’s current rate
  * Try 48 kHz first; then 44.1 kHz

* **Glitches under load**

  * Try bigger buffer sizes
  * Disable “USB selective suspend” in Windows Power Options
  * Prefer rear-panel USB ports (direct to chipset)

---

## Developer notes

* **Endpoints**

  * IN `0x81` (async), OUT `0x04` (adaptive)
  * Alt-settings provide **16-bit** and **24-bit** subslots

* **Safety offset**

  * Tunable in `/src/asio/AsioDriver.cpp` → `kDefaultSafetyFrames`
  * DAWs may add their own hidden offsets; verify with loopback tests

* **Channel naming**

  * Capture: `Input 1 (L)`, `Input 2 (R)`
  * Playback: `Output 1 (L)`, `Output 2 (R)`

* **Logging**

  * Set `ARGENT_LOG=1` env var to enable verbose logs in `%PROGRAMDATA%\ArgentOne\logs`

---

## Roadmap

* Device panel (tray) for rate/bit-depth/buffer switching
* Per-rate safety-offset presets
* Direct WinUSB engine prototype with ISO scheduling
* Firmware notes (if we ever need alt-setting tweaks)

---

## License

* **This repository:** MIT (see `LICENSE`)
* **Steinberg ASIO SDK:** subject to Steinberg’s license (you must obtain it yourself; not included here)

---

## Credits

* USB descriptor dump & reverse notes by the **Argent** team
* ASIO plumbing inspired by public SDK samples (no Steinberg code included here)

---

### Quick checklist (before you open your DAW)

* [ ] Copied `ArgentOneASIO.dll` to `C:\Program Files\Steinberg\ASIO\ArgentOneASIO\`
* [ ] Wrote registry entry under `HKLM\SOFTWARE\ASIO\Argent One (ASIO)` with the correct CLSID and path
* [ ] Selected **Argent One (ASIO)** in your DAW
* [ ] Started at **48 kHz / 128 samples**, then tuned down

---

If you want, I can tailor this README with your exact company/author info and drop in your repo’s folder structure and CLSID you’re using.
