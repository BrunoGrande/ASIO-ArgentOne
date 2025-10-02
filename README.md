# Argent One + FlexASIO + RS_ASIO Setup

This repository contains the configuration and instructions to use your **Armer Argent One** audio interface with **FlexASIO** (universal ASIO driver) and **RS_ASIO** (Rocksmith 2014 patch for ASIO support). The goal is to allow you to use Guitar Rig, Rocksmith, and regular Windows audio (YouTube, Discord, etc.) together without driver conflicts.

> ⚠️ *Placeholders* below (`<LINK_TO_PROJECT>`) should be replaced with the actual GitHub or release URLs.

---

## Table of Contents

1. [Overview](#overview)  
2. [Prerequisites](#prerequisites)  
3. [Configuration Files](#configuration-files)  
4. [Step-by-Step Setup](#step-by-step-setup)  
5. [Testing & Latency Tuning](#testing--latency-tuning)  
6. [Troubleshooting](#troubleshooting)  
7. [License & Credits](#license--credits)

---

## Overview

- **Argent One** is your audio interface (Armer brand).  
- **FlexASIO** is a generic ASIO driver that uses Windows audio backends (WASAPI, etc.) to allow multiple audio apps to share a single interface.  
- **RS_ASIO** is a patch that injects ASIO support into **Rocksmith 2014**. It allows Rocksmith to use ASIO drivers instead of default WASAPI. :contentReference[oaicite:0]{index=0}  

With this setup, you can run:
- Guitar Rig (or any VST-based amp sim) using ASIO via FlexASIO  
- Rocksmith via RS_ASIO  
- Regular Windows audio (YouTube, Discord) at the same time  

You avoid needing Voicemeeter or juggling drivers manually.

---

## Prerequisites

- Windows PC  
- **Armer Argent One** interface, installed and working in Windows  
- **FlexASIO** installed (put `FlexASIO.dll` etc in system)  
- **FlexASIO GUI** for easier editing of settings (optional, but helpful)  
- **RS_ASIO** patch for Rocksmith 2014 (`RS_ASIO.dll`, etc)  
- Basic familiarity with editing `.toml` and `.ini` files  

---

## Configuration Files

This repo includes two key config files:

- `FlexASIO.toml` — holds the FlexASIO configuration  
- `RS_ASIO.ini` — config for RS_ASIO in Rocksmith  

Below is the version that works for your setup using **Argent One** as both input and output:

### FlexASIO.toml
```toml
backend = "Windows WASAPI"
# sampleRate = 48000     # leave commented (GUI may crash if this is present)
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
