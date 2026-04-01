# ⚡ PlatformIO — Kiro Power

> **⚠️ Experimental:** This power is under active development. Steering content, structure, and behavior may change without notice. Feedback and contributions are welcome.

A community-built [Kiro Power](https://kiro.dev) that gives your AI assistant deep knowledge of the [PlatformIO](https://platformio.org/) ecosystem — project management, building, flashing, debugging, serial monitoring, library management, and device discovery across 1500+ boards, 50+ platforms, and 20+ frameworks.

## What It Does

When this power is active, Kiro understands how to:

- Initialize and configure PlatformIO projects (`platformio.ini`)
- Build, upload, and monitor firmware via the `pio` CLI
- Search and install libraries from the PlatformIO Registry
- Discover connected devices and installed platforms/boards
- Debug firmware with J-Link, ST-Link, CMSIS-DAP, OpenOCD, and more
- Work with serial protocols (UART, SPI, I2C, RS-232, USB, 1-Wire)
- Work with network protocols (TCP, UDP, MQTT, CoAP, LoRa/LoRaWAN)
- Follow idiomatic coding standards for C, C++, Rust, Python, and Assembly in embedded contexts
- Look up datasheets and technical reference manuals from trusted sources
- Use inclusive language in new code and documentation

## Steering Files

Context is loaded on demand to keep things lean. The power includes eight steering guides:

| File | Covers |
|---|---|
| `cli-reference` | Full `pio` CLI command reference — build, upload, debug, monitor, packages, and all flags |
| `project-structure` | `platformio.ini` config, multi-environment setups, library deps, build flags, scripting |
| `coding-standards` | Embedded coding conventions for C, C++, Assembly, Rust, and Python |
| `serial-protocols` | RS-232, USB (CDC/HID), SPI, I2C/TWI, UART, 1-Wire best practices |
| `network-protocols` | TCP, UDP, MQTT, CoAP, LoRa/LoRaWAN on constrained devices |
| `debugging-guide` | Unified Debugger setup, debug probes, GDB workflows |
| `datasheet-lookup` | Finding datasheets and TRMs for MCUs and peripherals |
| `inclusive-language` | Inclusive terminology for embedded dev (with guidance on legacy API boundaries) |

## Prerequisites

- Python 3.6+
- PlatformIO Core CLI (`pio`) in your PATH

```bash
# Install via pip
pip install platformio

# Verify
pio --version
```

## Installation

Copy or clone this power into your Kiro powers directory, then enable it from the Kiro Powers panel.

## License

This project is licensed under the [GNU General Public License v3.0](LICENSE).
