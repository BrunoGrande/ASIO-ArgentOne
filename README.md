# Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using the **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO). The setup allows running Guitar Rig, Rocksmith, and standard Windows audio (YouTube, Discord, etc.) simultaneously without conflicts.

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
8. [Argent One Hardware Details](#argent-one-hardware-details)
9. [Credits & References](#credits--references)

---

## Overview

* **Argent One** is an audio interface by Armer.
* **FlexASIO** functions as a universal ASIO driver, bridging between ASIO hosts and Windows audio backends (WASAPI, etc.), enabling multiple applications to share the same interface.
* **RS_ASIO** injects ASIO support into **Rocksmith 2014**, replacing or augmenting its WASAPI audio handling.

With this setup it is possible to:

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
* Ability to edit `.toml` and `.ini` files
* .NET Desktop Runtime 6.x installed (for the GUI)

---

## Included Config Files

* `FlexASIO.toml` — FlexASIO configuration
* `RS_ASIO.ini` — RS_ASIO configuration

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

> The FlexASIO GUI does **not** support the `sampleRate` field. Leave it commented or remove it entirely.

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
   Download from the [official releases](https://github.com/dechamps/FlexASIO/releases/).
   Run the installer so that `FlexASIO.dll` and related files are installed.

2. **Install RS_ASIO**
   Download from the [RS_ASIO releases](https://github.com/mdias/rs_asio/releases/).
   Copy `RS_ASIO.dll`, `RS_ASIO.ini`, and related files into the Rocksmith game folder.

3. **Place configuration files**

   * Copy `FlexASIO.toml` to the user profile directory (e.g. `C:\Users\<YourName>\FlexASIO.toml`)
   * Replace the `RS_ASIO.ini` in the Rocksmith folder with the one from this repository

4. **Set up Windows audio**

   * Set Argent One as the default playback & capture device
   * Select 48 kHz in device properties. This is the recommended rate for Rocksmith; other applications may use different sample rates if required
   * Disable audio enhancements and exclusive control

5. **Launch FlexASIO GUI (optional)**

   * Ensure `sampleRate` is commented or removed to avoid crashes
   * Use the GUI to confirm device selection and buffer size

6. **Launch Rocksmith**

   * RS_ASIO will hook into the game, detect FlexASIO, and use the provided configuration

7. **Launch Guitar Rig / other ASIO apps**

   * Select `FlexASIO` as the ASIO driver
   * Both apps will share the Argent One interface

8. **Play & monitor**
   Guitar, game audio, and system sounds should now work simultaneously.

---

## Testing & Latency Tuning

* Start with `bufferSizeSamples = 512` (balanced)
* If crackles/pops occur, increase to `1024` or higher
* For lower latency, experiment with `256` or `128`
* Review `RS_ASIO-log.txt` to verify buffer settings and check for xruns
* Monitor CPU / DPC latency with tools like LatencyMon
* Ensure all devices are configured at 48 kHz to avoid resampling overhead

---

## Streaming & ASIO Bypass

When RS_ASIO handles Rocksmith’s audio output through ASIO, the Windows audio stack is bypassed. This prevents capture by streaming/recording software. Options include:

* Using WASAPI output in RS_ASIO (instead of ASIO) so the game audio goes through Windows mixer
* Employing virtual audio routing tools (e.g. VB-Audio Cable, VoiceMeeter)
* For simple streaming setups, enabling WASAPI output is the most straightforward solution

---

## Troubleshooting

| Symptom                 | Likely Cause                        | Suggested Fix                                          |
| ----------------------- | ----------------------------------- | ------------------------------------------------------ |
| GUI crashes on start    | `sampleRate` present in TOML        | Remove or comment out `sampleRate`                     |
| Rocksmith silent        | Driver mismatch or misconfiguration | Check `RS_ASIO-log.txt` and verify `Driver = FlexASIO` |
| Crackles / glitches     | Buffer too small, USB/power issues  | Increase buffer, disable USB power saving              |
| Latency too high        | Buffer too large                    | Reduce buffer gradually                                |
| No game audio in stream | ASIO bypassing Windows              | Enable WASAPI output or use virtual audio routing      |
| GUI shows no devices    | Device disabled in Windows settings | Enable device in Windows Sound Settings                |

---

## Argent One Hardware Details

* [Product Page](https://armer.com.br/produtos/interface-de-audio-usb-armer-argent-one/)
* [Official Manual (PDF)](https://drive.google.com/file/d/15iVYalthiWtdguu4y76YLcfhcIaOtVLd/view?usp=sharing)

### Key Features

* Two combo inputs (XLR / ¼” TRS).
* +48V Phantom Power for condenser microphones (Channel 1).
* Channel 2 switchable between **Instrument (INST)** and **Line**.
* Direct Monitoring switch (MON) for zero-latency monitoring.
* Balanced TRS outputs for studio monitors.
* Dedicated headphone output with its own gain control.

### Important Behavior Notes

* If **Mic (Ch1)** and **Instrument/Line (Ch2)** are used simultaneously, the inputs are summed to **mono**.
* Enabling **INST** on Channel 2 reconfigures it to accept high-impedance instruments (e.g. guitar, bass). Disabling it sets the channel as a balanced mono line input, ideal for keyboards, synthesizers, or mixers.
* When **Direct Monitor** is active, input signals are routed directly to outputs with zero latency. If the DAW also routes inputs to outputs, users may hear doubled signals (echo effect).
* Phantom Power supplies 48V only to Channel 1. Use with condenser microphones that require it, and disable when not in use to avoid unwanted noise.

---

## Credits & References

* **RS_ASIO** — patch for Rocksmith 2014 ASIO support
* **FlexASIO** — universal ASIO driver using PortAudio and Windows audio backends
* **FlexASIO GUI** — helper GUI for editing `FlexASIO.toml`
* **Armer Argent One** — hardware details based on [official manual] and manufacturer specs
