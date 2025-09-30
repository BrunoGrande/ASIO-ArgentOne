Custom ASIO Driver for Armer Argent One

This repository contains the source code and documentation for a custom ASIO driver for the Armer Argent One audio interface. The goal is to provide a low‑latency, fully‑featured driver on Windows without relying on generic wrappers like ASIO4ALL.

Device overview

The Argent One is a 2‑in/2‑out USB audio interface. It provides two analogue inputs, two analogue outputs and a headphone output. Key features from the official manual include:

Channel 1: combo XLR/¼″ input with 48 V phantom power for condenser microphones.

Channel 2: ¼″ input with a toggleable INST mode for high‑impedance instruments such as guitars.

Front‑panel gain controls, signal/clipping indicators and MON button for direct monitoring (routes inputs directly to outputs with zero latency).

Balanced TRS outputs and a headphone output with its own volume control.

Supports 16‑ or 24‑bit resolution at 44.1 kHz, 48 kHz, 96 kHz and 192 kHz sample rates.

Internally, the interface uses a Gaia Vision UA02A USB audio codec (UAC2), two HEF4053BT analogue switch ICs for signal routing and additional support ICs. These components have been identified via board inspection but are not documented publicly.

Driver design goals

ASIO compliance: implement a compliant IASIO interface as defined by Steinberg's ASIO SDK, supporting 2 input and 2 output channels.

Low latency: use event‑driven WASAPI or WDM/KS backends to achieve sub‑10 ms latency. Allow adjustable buffer sizes (64–2048 frames).

Sample rates & resolution: report support for 44.1–192 kHz at 24‑bit resolution; reject unsupported rates.

Channel mapping: map CH1/CH2 to the driver’s input channels. Provide an option to duplicate the instrument input to both left and right channels when running in mono mode.

Control panel: provide a small configuration utility to select buffer size, sample rate and mono mixing. Hardware features such as phantom power, INST and direct monitor remain physical buttons as per the manual.

Open source: all code should be openly licensed to encourage community contributions.

Development plan

Environment setup

Download and install the Steinberg ASIO SDK (available from Steinberg after accepting their licence).

Set up a C++17 build environment (Visual Studio 2019 or later). Optional: use CMake for build configuration.

Familiarise yourself with example drivers provided in the SDK.

Device enumeration

Use the Windows API (IMMDeviceEnumerator) or the Win32 SetupAPI to locate the Argent One based on its VID/PID.

Query the USB audio descriptors to confirm supported sample rates and bit depths.

ASIO wrapper implementation

Define a class ArgentASIODriver implementing the IASIO interface.

Implement the required methods: init, start, stop, getChannels, getBufferSize, getSampleRate, setSampleRate, createBuffers, disposeBuffers, bufferSwitch, etc.

Use WASAPI exclusive mode to open the Argent One endpoint at the desired sample rate and buffer size. Alternatively, implement a WDM/Kernel Streaming (KS) backend for lower latency.

Buffer management

Create double buffers for input and output, sized according to the selected ASIO buffer length.

In the bufferSwitch callback, copy data between the WASAPI/KS buffers and the ASIO buffers.

Configuration utility

Provide a Windows GUI application (e.g., using Win32 API or Qt) that writes user preferences (sample rate, buffer size, mono mixing) to the registry or a configuration file. The driver should read these preferences at startup.

Testing

Test with audio applications (Reaper, Cubase) and Rocksmith using the RS_ASIO wrapper. Measure round‑trip latency and adjust buffer sizes accordingly.

Test sample rate changes, device hot‑plugging and error handling.

Repository structure
ASIO-ArgentOne/
├── src/              # C++ source files for the ASIO driver
├── control-panel/    # Code for the configuration utility
├── docs/             # Documentation and further reading
└── README.md         # This file
