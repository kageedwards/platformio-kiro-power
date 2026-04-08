# ⚡ PlatformIO — Kiro Power

> **⚠️ Experimental:** This power is under active development. Steering content, structure, and behavior may change without notice. Feedback and contributions are welcome.

A community-built [Kiro Power](https://kiro.dev) that gives your AI assistant deep knowledge of the [PlatformIO](https://platformio.org/) ecosystem — project management, building, flashing, debugging, serial monitoring, library management, and device discovery across 1500+ boards, 50+ platforms, and 20+ frameworks.

## What It Does

When this power is active, Kiro understands how to:

- **Set up new projects** with a formalized, repeatable workflow — board validation, framework-correct boilerplate, peripheral-aware scaffolding, and build verification
- **Handle boards not in the PlatformIO registry** by researching online and creating custom board definitions with proper pin variants
- **Configure IDE IntelliSense** for Kiro and other VS Code forks via clangd and compilation database generation
- Build, upload, and monitor firmware via the `pio` CLI
- Search and install libraries from the PlatformIO Registry
- Discover connected devices and installed platforms/boards
- Debug firmware with J-Link, ST-Link, CMSIS-DAP, OpenOCD, and more
- Work with serial protocols (UART, SPI, I2C, RS-232, USB, 1-Wire)
- Work with network protocols (TCP, UDP, MQTT, CoAP, LoRa/LoRaWAN)
- Follow idiomatic coding standards for C, C++, Rust, Python, and Assembly in embedded contexts
- Look up datasheets and technical reference manuals from trusted sources
- Use inclusive language in new code and documentation

## Project Initialization

The power includes a formalized 11-step project init workflow that ensures consistent results regardless of context. When you ask Kiro to create a new PlatformIO project, it will:

1. Verify `pio` CLI prerequisites and check your IDE's C/C++ IntelliSense setup
2. Confirm the project root directory (defaults to workspace root, asks before using non-empty directories)
3. Validate the target board against `pio boards` — or research online and create a custom board definition if the board isn't in the registry
4. Resolve the correct framework and generate framework-matched boilerplate source code
5. Identify on-board peripherals (LoRa, OLED, GPS, etc.) and offer to include initialization code with appropriate libraries
6. Configure `platformio.ini` defaults, `.gitignore`, and verify the build
7. Generate a `compile_commands.json` compilation database for clangd IntelliSense

See `steering/project-init.md` for the full procedure.

## IDE Setup (Kiro / VS Code Forks)

Kiro is a VS Code fork and may not have direct access to the Microsoft Marketplace. This power uses the PlatformIO CLI directly, so the PlatformIO IDE extension is optional. For C/C++ IntelliSense:

- **clangd** (recommended) — open-source, may be available in Kiro's extension search. If not, sideload the VSIX from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd). Requires `compile_commands.json` (generated automatically during project init).
- **Microsoft C/C++** — sideload the VSIX from the [Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools). Required if you also want the PlatformIO IDE extension GUI.

## Steering Files

Context is loaded on demand to keep things lean. The power includes nine steering guides:

| File | Covers |
|---|---|
| `project-init` | **Formalized 11-step project initialization workflow** — directory selection, board validation (including custom board definitions for boards not in the registry), framework resolution, peripheral-aware boilerplate, IntelliSense setup |
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
