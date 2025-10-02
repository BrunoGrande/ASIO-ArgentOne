# Argent One + FlexASIO + RS_ASIO Setup

This repository provides configuration and instructions for using your **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch to enable ASIO). The system allows you to run Guitar Rig, Rocksmith, and standard Windows audio (YouTube, Discord, etc.) simultaneously.

- RS_ASIO: https://github.com/mdias/rs_asio :contentReference[oaicite:0]{index=0}  
- FlexASIO: https://github.com/dechamps/FlexASIO :contentReference[oaicite:1]{index=1}  
- FlexASIO GUI: https://github.com/flipswitchingmonkey/FlexASIO_GUI :contentReference[oaicite:2]{index=2}  

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
- **FlexASIO** acts as a universal ASIO driver, bridging between ASIO hosts and Windows audio backends (WASAPI, etc.), enabling multiple applications to use the same interface. :contentReference[oaicite:3]{index=3}  
- **RS_ASIO** injects ASIO support into **Rocksmith 2014**, replacing or augmenting its WASAPI audio handling. :contentReference[oaicite:4]{index=4}  

With this setup, you’ll be able to:

- Use Guitar Rig (or other VST / amp simulators) via ASIO through FlexASIO  
- Play Rocksmith with ASIO (via RS_ASIO)  
- Let Windows audio (e.g. browser, system sounds) co-exist with ASIO apps  

---

## Prerequisites

- Windows PC  
- **Armer Argent One** interface, installed and functioning  
- **FlexASIO** (driver) installed  
- **FlexASIO GUI** (optional but helpful)  
- **RS_ASIO** patch for Rocksmith 2014  
- Basic comfort editing `.toml` and `.ini` text files  
- .NET Desktop Runtime 6.x installed (for the FlexASIO GUI) :contentReference[oaicite:5]{index=5}  

---

## Included Config Files

You should find in this repo:

- `FlexASIO.toml` — configuration for FlexASIO  
- `RS_ASIO.ini` — configuration for RS_ASIO  

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
