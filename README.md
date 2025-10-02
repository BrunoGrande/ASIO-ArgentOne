# Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using your **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO). The system allows you to run Guitar Rig, Rocksmith, and standard Windows audio (YouTube, Discord, etc.) simultaneously.

* [RS_ASIO](https://github.com/mdias/rs_asio)
* [FlexASIO](https://github.com/dechamps/FlexASIO)
* [FlexASIO GUI](https://github.com/flipswitchingmonkey/FlexASIO_GUI)

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Included Config Files](#included-config-files)
4. [Setup Instructions](#setup-instructions)
5. [Testing & Latency Tuning](#testing--latency-tuning)
6. [Streaming & ASIO Bypass](#streaming--asio-bypass)
7. [Troubleshooting](#troubleshooting)
8. [Credits & References](#credits--references)

---

## Overview

* **Argent One** is your audio interface (by Armer).
* **FlexASIO** acts as a universal ASIO driver, bridging between ASIO hosts and Windows audio backends (WASAPI, etc.), enabling multiple applications to share the same interface.
* **RS_ASIO** injects ASIO support into **Rocksmith 2014**, replacing or augmenting its WASAPI audio handling.

With this setup, you’ll be able to:

* Use Guitar Rig (or other VST / amp simulators) via ASIO through FlexASIO
* Play Rocksmith with ASIO (via RS_ASIO)
* Let Windows audio (e.g. browser, system sounds) coexist with ASIO apps

---

## Prerequisites

* Windows PC
* **Armer Argent One** interface, installed and functional
* **FlexASIO** installed
* **FlexASIO GUI** (optional, but helpful)
* **RS_ASIO** applied to Rocksmith 2014
* Comfort editing `.toml` and `.ini` files
* .NET Desktop Runtime 6.x installed (for the GUI)

---

## Included Config Files

* `FlexASIO.toml` — FlexASIO configuration
* `RS_ASIO.ini` — RS_ASIO configuration

Below is the version tested for **Argent One (input & output)**:

### FlexASIO.toml

```toml
backend = "Windows WASAPI"
# sampleRate = 48000     # GUI may crash if uncommented
bufferSizeSamples = 512

[input]
device = "Microfone (2- Armer Argent)"
channels = 1
wasapiExclusiveMode = false
wasapiAutoConvert = true

[output]
device = "Fones de ouvido (2- Armer Argent)"
wasapiExclusiveMode = false
wasapiAutoConvert = true
```

> Note: The GUI currently does **not** support the `sampleRate` field (it throws an exception if present). Keep it commented or omit it entirely.

### RS_ASIO.ini

```ini
[Config]
EnableWasapiOutputs = 0
EnableWasapiInputs = 0
EnableAsio = 1

[Asio]
BufferSizeMode = driver

[Asio.Output]
Driver = FlexASIO
BaseChannel = 0
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0

[Asio.Input.0]
Driver = FlexASIO
Channel = 0
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0

[Asio.Input.1]
Driver =
Channel =
EnableSoftwareEndpointVolumeControl = 0
EnableSoftwareMasterVolumeControl = 0
```

---

## Setup Instructions

1. **Install FlexASIO**
   Download from its official releases page. `<LINK_TO_FLEXASIO_RELEASES>`
   Run the installer so `FlexASIO.dll` etc are correctly placed.

2. **Install RS_ASIO**
   Download from its releases page. `<LINK_TO_RS_ASIO_RELEASES>`
   Copy `RS_ASIO.dll`, `RS_ASIO.ini`, etc. into the Rocksmith game folder.

3. **Place configuration files**

   * Copy `FlexASIO.toml` to your user profile directory (e.g. `C:\Users\<YourName>\FlexASIO.toml`)
   * Place `RS_ASIO.ini` in the Rocksmith folder

4. **Set up Windows audio**

   * Mark Argent One as default playback & capture device
   * Set sample rate to 48 kHz (if option available)
   * Turn off enhancements, exclusive control, etc.

5. **Launch FlexASIO GUI (optional)**

   * If GUI fails, ensure `sampleRate` is commented out or removed
   * Use GUI to confirm device selection and buffer settings

6. **Launch Rocksmith**

   * RS_ASIO will inject into the game, detect `FlexASIO`, and apply your configurations

7. **Launch Guitar Rig / other ASIO apps**

   * Select `FlexASIO` as the ASIO driver
   * They will share the Argent One interface

8. **Play & monitor**
   You should hear your guitar, game audio, and system sounds together.

---

## Testing & Latency Tuning

* Start with `bufferSizeSamples = 512` (balanced)
* If crackles/pops appear, increase to `1024` or more
* For lower latency, try `256` or `128` (if system handles it)
* Inspect `RS_ASIO-log.txt` for buffer negotiation and xruns
* Monitor CPU / DPC latency spikes
* Ensure all sample rates are 48 kHz to avoid hidden resampling overhead

---

## Streaming & ASIO Bypass

When RS_ASIO handles the game’s audio output, it bypasses the Windows audio stack. That means streaming/recording software may not “see” the game’s audio. To address this:

* Use WASAPI output in RS_ASIO instead of ASIO (then game audio passes through the Windows mixer)
* Use virtual audio routing (e.g. virtual cables or advanced routing apps) to capture ASIO audio
* For easier streaming, enabling WASAPI output is usually the least complex path

---

## Troubleshooting

| Symptom                 | Likely Cause                        | Suggested Fix                                          |
| ----------------------- | ----------------------------------- | ------------------------------------------------------ |
| GUI crashes on start    | `sampleRate` present in TOML        | Comment out or remove `sampleRate`                     |
| Rocksmith silent        | Driver mismatch or RS_ASIO issue    | Check `RS_ASIO-log.txt`, verify `Driver = FlexASIO`    |
| Crackles / glitches     | Buffer too low, system interference | Increase buffer, disable USB power saving, etc.        |
| Latency too high        | Buffer too large                    | Lower buffer gradually, test stability                 |
| No game audio in stream | ASIO bypassing Windows              | Enable WASAPI output or route with virtual audio tools |

---

## Credits & References

* **RS_ASIO** — patch for Rocksmith 2014 ASIO support
* **FlexASIO** — universal ASIO driver using PortAudio and Windows audio backends
* **FlexASIO GUI** — helper GUI for editing `FlexASIO.toml`
