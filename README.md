# Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using your **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO). The system allows you to run Guitar Rig, Rocksmith, and standard Windows audio (YouTube, Discord, etc.) simultaneously.

- [RS_ASIO](https://github.com/mdias/rs_asio)  
- [FlexASIO](https://github.com/dechamps/FlexASIO)  
- [FlexASIO GUI](https://github.com/flipswitchingmonkey/FlexASIO_GUI)  

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

- **Argent One** is your audio interface (by Armer).  
- **FlexASIO** acts as a universal ASIO driver, bridging between ASIO hosts and Windows audio backends (WASAPI, etc.), enabling multiple applications to share the same interface.  
- **RS_ASIO** injects ASIO support into **Rocksmith 2014**, replacing or augmenting its WASAPI audio handling.

With this setup, you’ll be able to:

- Use Guitar Rig (or other VST / amp simulators) via ASIO through FlexASIO  
- Play Rocksmith with ASIO (via RS_ASIO)  
- Let Windows audio (e.g. browser, system sounds) coexist with ASIO apps  

---

## Prerequisites

- Windows PC  
- **Armer Argent One** interface, installed and functional  
- **FlexASIO** installed  
- **FlexASIO GUI** (optional, but helpful)  
- **RS_ASIO** applied to Rocksmith 2014  
- Comfort editing `.toml` and `.ini` files  
- .NET Desktop Runtime 6.x installed (for the GUI)  

---

## Included Config Files

- `FlexASIO.toml` — FlexASIO configuration  
- `RS_ASIO.ini` — RS_ASIO configuration  

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
