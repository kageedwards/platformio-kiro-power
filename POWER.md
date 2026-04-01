---
name: "platformio"
displayName: "PlatformIO"
description: "Comprehensive embedded development guide for PlatformIO CLI. Covers project management, building, flashing, debugging, serial monitoring, library management, and device discovery across all supported platforms and boards."
keywords: ["platformio", "pio", "embedded", "firmware", "microcontroller"]
author: "Community"
---

# PlatformIO

## Overview

PlatformIO is a cross-platform, cross-architecture, multi-framework professional tool for embedded systems and IoT development. It supports 1500+ boards, 50+ development platforms, and 20+ frameworks out of the box. This power gives you fluent command of the PlatformIO CLI so you can manage projects, discover hardware, build, flash, debug, monitor, and manage libraries for any target platform.

When working on a PlatformIO project, you should always start by reading the workspace `platformio.ini` to understand the project's environments, target boards, frameworks, and dependencies before taking any action.

## Available Steering Files

This power uses lazy-loaded steering files to keep context lean. Load them as needed:

- **cli-reference** — Complete PlatformIO CLI command reference. Build, upload, debug, monitor, device discovery, package management, project init, and all flags/options. Load this when you need to run any `pio` command.
- **serial-protocols** — Best practices and standards for RS-232, USB (CDC/HID), SPI, I2C/TWI, UART, and 1-Wire serial communication protocols across embedded platforms. Load when working with hardware communication buses.
- **network-protocols** — Best practices for TCP, UDP, and LoRa/LoRaWAN networking on embedded devices. Covers socket programming on constrained devices, MQTT, CoAP, and radio configuration. Load when working with networked firmware.
- **coding-standards** — Idiomatic coding conventions and style guidelines for C, C++, Assembly, Rust, and Python in embedded contexts. Load when writing or reviewing firmware code.
- **project-structure** — PlatformIO project layout, `platformio.ini` configuration reference, multi-environment setups, library dependency management, build flags, and advanced scripting. Load when setting up or restructuring a project.
- **debugging-guide** — Unified Debugger configuration, debug tool setup (J-Link, ST-Link, CMSIS-DAP, OpenOCD, etc.), GDB usage, and common debugging workflows. Load when debugging firmware.
- **datasheet-lookup** — Methodical approach to finding datasheets, technical reference manuals, and peripheral documentation for MCUs and external devices from trusted sources. Load when you need chip or peripheral specifications.
- **inclusive-language** — Inclusive terminology guide for embedded development. Covers replacements for historically oppressive terms (master/slave, blacklist/whitelist, male/female connectors, etc.) with clear guidance on when legacy terms are acceptable at API boundaries. Load when writing new code, documentation, or reviewing naming conventions.

## Onboarding

### Prerequisites

- Python 3.6+ (PlatformIO Core is Python-based)
- `pio` CLI available in PATH (verify: `pio --version`)

### Installation

```bash
# Via pip (recommended for CLI-only usage)
pip install platformio

# Or via installer script
curl -fsSL -o get-platformio.py https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py
python3 get-platformio.py
```

After installation, ensure shell commands are available:
```bash
# Add to PATH if needed (macOS/Linux)
export PATH="$HOME/.platformio/penv/bin:$PATH"
```

### Verification

```bash
pio --version
pio system info
```

## Quick Workflows

### Discover What's Installed

```bash
# List installed platforms and their packages
pio pkg list --global

# List boards from installed platforms only
pio boards --installed

# List ALL supported boards (1500+), optionally filter
pio boards esp32
pio boards atmega
pio boards risc-v

# List connected devices (serial ports)
pio device list

# JSON output for programmatic use
pio boards --installed --json-output
pio device list --json-output
```

### Create a New Project

```bash
# Interactive — will prompt for board and framework
pio project init

# Explicit board and framework
pio project init --board esp32dev --project-option "framework=arduino"
pio project init --board megaatmega2560 --project-option "framework=arduino"
pio project init --board sifive-hifive1-revb --project-option "framework=freedom-e-sdk"
```

### Build, Upload, Monitor

```bash
# Build default environment
pio run

# Build specific environment
pio run -e esp32dev

# Build and upload
pio run -t upload

# Upload to specific environment and port
pio run -e esp32dev -t upload --upload-port /dev/cu.usbserial-0001

# Build, upload, then immediately monitor serial
pio run -t upload -t monitor

# Monitor serial output (standalone)
pio device monitor
pio device monitor --baud 115200 --filter esp32_exception_decoder
```

### Debug

```bash
# Start debug session (GDB)
pio debug

# Debug specific environment
pio debug -e esp32dev

# Debug with verbose output
pio debug -v
```

### Library Management

```bash
# Search the PlatformIO Registry (15,000+ libraries)
pio pkg search "type:library DHT"
pio pkg search "type:library keyword:lora framework:arduino"
pio pkg search "type:library keyword:i2c platform:espressif32"

# Install a library into the current project
pio pkg install --library "adafruit/Adafruit BME280 Library"
pio pkg install -l "sandeepmistry/LoRa@^0.8.0"

# List project dependencies
pio pkg list

# Update all project dependencies
pio pkg update
```

## Agent Behavior Guidelines

When working on a PlatformIO project, follow these practices:

1. **Read `platformio.ini` first.** Always understand the project's target environments, boards, frameworks, and existing dependencies before suggesting changes.

2. **Query the environment.** Use `pio boards --installed` and `pio device list` to understand what platforms are available and what hardware is connected. Use `pio pkg list` to see current dependencies. Don't assume — ask the toolchain.

3. **Respect platform differences.** Communication protocols, register layouts, memory constraints, and toolchain quirks vary widely even among boards on the same platform. Always verify against the specific board and MCU.

4. **Use the registry.** When a user needs a driver or library, search with `pio pkg search` first. Registry entries contain headers, examples, and compatibility metadata that are invaluable.

5. **Look up datasheets on demand.** When you need chip-specific information (register maps, electrical characteristics, peripheral capabilities), use the datasheet-lookup steering file's methodology to find authoritative sources.

6. **Match the language idiom.** Embedded projects span C, C++, Rust, Python, and ASM. Load the coding-standards steering file and follow the conventions for whichever language the project uses.

7. **Protocol awareness.** When working with SPI, I2C, UART, RS-232, USB, TCP, UDP, or LoRa, load the relevant steering file. Protocol configuration is highly device-specific and getting it wrong can damage hardware or produce silent failures.

8. **Don't guess upload ports.** Use `pio device list` to discover connected devices. If multiple devices are connected, ask the user which one to target.

9. **Prefer `pio run -t upload -t monitor`** for the common flash-and-watch workflow. It chains build, upload, and serial monitor in one command.

10. **Use `--json-output`** when you need to parse CLI output programmatically for board IDs, device paths, or package metadata.

11. **Use inclusive language — without breaking anything.** In code you write (variable names, function names, comments, documentation), prefer inclusive terminology (controller/peripheral, allowlist/blocklist, plug/socket). But never alter external URLs, library API calls, register names, branch names, or protocol-level identifiers to match — correctness and interoperability always come first. Load the inclusive-language steering file for the full guide.

## Troubleshooting

### "pio: command not found"
The PlatformIO CLI is not in PATH. Add `$HOME/.platformio/penv/bin` to PATH or activate the virtual environment.

### Upload fails with "No serial port found"
Run `pio device list` to check if the device is detected. Common causes:
- Missing USB driver (CP2102, CH340, FTDI)
- Device not connected or in bootloader mode
- Permission issues on Linux (`sudo usermod -a -G dialout $USER`)

### Build fails with "Unknown board ID"
The board's platform is not installed. Install it:
```bash
pio pkg install -p "platformio/espressif32"
```
Or let PlatformIO auto-install by running `pio run` in a project with the board declared in `platformio.ini`.

### Library not found
Search the registry with different terms:
```bash
pio pkg search "type:library <keyword>"
```
Libraries can also be installed from Git URLs or local paths if not in the registry.

### Debug probe not detected
Verify the debug tool is configured in `platformio.ini`:
```ini
debug_tool = jlink    ; or stlink, cmsis-dap, etc.
debug_port = /dev/ttyACM0  ; if needed
```
Run `pio debug -v` for verbose output to diagnose connection issues.

---

**CLI Tool:** `pio`
**Documentation:** [docs.platformio.org](https://docs.platformio.org/en/latest/)
**Registry:** [registry.platformio.org](https://registry.platformio.org/)
