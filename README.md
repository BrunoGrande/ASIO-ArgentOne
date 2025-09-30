# ASIO-ArgentOne

Custom ASIO driver (and dev toolkit) for the **Armer Argent** USB Audio 2.0 interface.

> Goal: deliver a stable, low-latency ASIO device for Windows DAWs, with correct 16/24-bit formats, proper clocking, and clear channel names—without needing OEM software.

---

## Device profile (from USB descriptors)

* **Vendor / Product:** VID `0x2F6E` (SMMMPLUS Electronic Technology Co., Ltd.), PID `0x4E06`
* **Strings:** Manufacturer = “Armer Argent”, Product = “Armer Argent”
* **USB:** 2.0 High-Speed (480 Mb/s), **UAC2** (Audio Function 2.0)
* **I/O:** 2-in / 2-out (stereo)
* **Sample formats:** PCM
* **Bit depths:** 16-bit and 24-bit (via alternate interface settings)
* **Clocking:** Clock Source units report **host-programmable** frequency
* **Feature Unit:** Mute (master) + Volume (per channel)
* **Power:** Bus-powered, 100 mA

**Endpoints**

* **IN (capture)**: `0x81` isochronous, **async**

  * 16-bit `wMaxPacketSize=0x007C` (124B)
  * 24-bit `wMaxPacketSize=0x00BA` (186B)
* **OUT (playback)**: `0x04` isochronous, **adaptive**

  * 16-bit `wMaxPacketSize=0x007C` (124B)
  * 24-bit `wMaxPacketSize=0x00BA` (186B)

> Note: Extension Unit `wExtensionCode = 0x0BDA` hints at a Realtek-based codec path. Not required for functionality, but useful when troubleshooting.

---

## Windows sound panel formats observed

* **2 channels, 16-bit**: 44.1 kHz, 48 kHz, 96 kHz, 192 kHz
* **2 channels, 24-bit**: 44.1 kHz, 48 kHz, 96 kHz, 192 kHz

---

## Hardware notes (visual inspection)

* GAIA VISION **UA02A**
* **HEF4053BT** (x2) — analog switch/multiplexer
* **TXD24174** (marking seen)
* USB identifies as **Composite / USB Audio 2.0** under Windows with `usbaudio2.sys`

---

## What we’re building (phased)

1. **ASIO Wrapper (WASAPI/KS path)**
   Expose an ASIO device that talks to Windows’ class driver (`usbaudio2.sys`) through WASAPI Exclusive / Kernel Streaming. Fast to ship and stable; the OS handles alt-settings and clocking.

2. **Native UAC2 mapping (KS pin-accurate)**
   Map UAC2 terminals/feature units to ASIO channels and controls. Provide proper safety offset, low buffer sizes, and device-side mute/volume where supported.

3. **(Optional) Direct USB engine (WinUSB)**
   Research path that bypasses the class driver for potentially lower latency. Requires robust isochronous IN/OUT scheduling and error recovery—higher complexity and risk.

---

## Features (current / planned)

* ⏳ Detect **only** VID `0x2F6E` / PID `0x4E06`
* ⏳ ASIO device name: **“Argent One (ASIO)”**
* ⏳ Stereo I/O with stable channel labels: **In L/R, Out L/R**
* ⏳ 16-bit and 24-bit PCM; buffer sizes from ~32–2048 frames (host-dependent)
* ⏳ Mute/Volume bridged to device Feature Unit (when exposed)
* ⏳ Host-set sample rates (44.1/48/88.2/96/176.4/192 kHz)
* ⏳ Safety-offset tuning for popular DAWs (REAPER, Cubase, Live, Studio One)
* ⏳ Installer that registers ASIO under `HKLM\SOFTWARE\ASIO`

---

## Project structure

```
ASIO-ArgentOne/
├─ src/
│  ├─ asio/     # ASIO SDK wrapper + driver class (no Steinberg code included)
│  ├─ core/     # device enumeration, streaming engine, ring buffers
│  ├─ ks/       # WDM-KS (Kernel Streaming) plumbing for WASAPI/KS path
│  ├─ usb/      # (optional) WinUSB user-mode ISO engine experiments
│  └─ util/     # logging, GUIDs, registry helpers
├─ installer/   # Inno Setup or WiX scripts + .reg templates
├─ tools/       # descriptor dumps, latency tests, debug apps
├─ docs/        # design notes, troubleshooting, protocol sketches
└─ README.md
```

---

## Requirements

* **Windows 10/11 x64**
* **Visual Studio 2022** (Desktop development with C++)
* **Windows SDK + WDK** (matching the OS build)
* **Steinberg ASIO SDK 2.3+**

  * Download separately from Steinberg and accept its license.
  * Set `ASIOSDK_DIR` to the SDK root (SDK files are **not** in this repo).

---

## Building

1. Install VS2022, Windows SDK, and WDK.
2. Download the **ASIO SDK** and set `ASIOSDK_DIR` (e.g., `C:\SDKs\ASIO`).
3. Open `ASIO-ArgentOne.sln`.
4. Build **Release x64**.

**Artifacts**

* `ArgentOneASIO.dll` (ASIO driver)
* Optional CLI/test tools under `tools/bin`

---

## Installing (developer mode)

### Place the DLL

```
C:\Program Files\Steinberg\ASIO\ArgentOneASIO\ArgentOneASIO.dll
```

### Register in the registry

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\ASIO\Argent One (ASIO)]
"CLSID"="{5A07E2F8-72A5-4F47-8E31-5F3E3C0FB0D3}"
"Description"="Argent One (ASIO)"
"Dll"="C:\\Program Files\\Steinberg\\ASIO\\ArgentOneASIO\\ArgentOneASIO.dll"
"Manufacturer"="Argent"
"Version"="1.0.0"
```

> Replace the CLSID with the one compiled into the driver (kept in `src/asio/Guids.h`).

### Verify in a DAW

Open your DAW → Audio Device → ASIO → **Argent One (ASIO)**.
Select 16/24-bit and the desired sample rate in the driver panel.

---

## How it works (high level)

* **Enumeration**: match `USB\VID_2F6E&PID_4E06`, open **UAC2** pins via KS.
* **Streaming**: lock-free ring buffers per direction; double/triple buffering based on block size.
* **Clocking**: host requests the sample rate; the device advertises host-programmable clock.
* **Formats**: expose **16-bit** and **24-bit** ASIO sample types; channel layout **FL/FR**.
* **Controls**: link mute/volume to the device Feature Unit when available; otherwise fall back to software gain.
* **Latency reporting**: `bufferSize / sampleRate + safetyOffset` (backend-specific constant).

---

## Known good settings (starting points)

* **48 kHz** → **64–128** samples
* **96 kHz** → **128–256** samples
* **192 kHz** → **256–512** samples

If you hear crackles, increase the buffer, prefer rear-panel USB ports, and disable “USB selective suspend” while testing.

---

## Troubleshooting

* **Device not listed in DAW**
  Check `HKLM\SOFTWARE\ASIO` and confirm the DLL path and x64 build.

* **No audio / silence**
  Ensure the DAW’s sample rate matches the device rate. Try **48 kHz** first, then **44.1 kHz**.

* **Glitches under load**
  Increase buffer size; disable USB selective suspend; avoid front-panel hubs.

---

## Developer notes

* **Endpoints**: IN `0x81` (async), OUT `0x04` (adaptive); alt-settings provide 16-bit and 24-bit subslots.
* **Safety offset**: tune `kDefaultSafetyFrames` in `src/asio/AsioDriver.cpp`; DAWs may add hidden offsets—verify via loopback.
* **Channel naming**:

  * Capture: `Input 1 (L)`, `Input 2 (R)`
  * Playback: `Output 1 (L)`, `Output 2 (R)`
* **Logging**: set `ARGENT_LOG=1` to write verbose logs to `%PROGRAMDATA%\ArgentOne\logs`.

---

## Roadmap

* Device control panel (tray) for rate/bit-depth/buffer switching
* Per-rate safety-offset presets
* Direct WinUSB engine prototype with ISO scheduling
* Firmware notes (if alt-setting tweaks are ever required)

---

## License

* **This repository:** MIT (see `LICENSE`)
* **Steinberg ASIO SDK:** subject to Steinberg’s license (not included)

---

## Credits

* USB descriptor dump & engineering notes by the **Argent** team over their site and manual for the equipement
* ASIO plumbing inspired by public SDK samples (no Steinberg code included here)

---

### Quick checklist

* [ ] `ArgentOneASIO.dll` placed in `C:\Program Files\Steinberg\ASIO\ArgentOneASIO\`
* [ ] Registry entry under `HKLM\SOFTWARE\ASIO\Argent One (ASIO)` with the correct CLSID and path
* [ ] **Argent One (ASIO)** selected in your DAW
* [ ] Started at **48 kHz / 128 samples**, then tuned down as stable
