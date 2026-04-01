# PlatformIO CLI Reference

Complete command reference for the PlatformIO CLI (`pio`). Use this when you need to execute any PlatformIO operation.

## Project Management

### pio project init
Initialize a new project or reinitialize an existing one.

```bash
pio project init [OPTIONS]

Options:
  -d, --project-dir PATH    Project directory (default: CWD)
  -b, --board ID            Board identifier (repeatable for multi-env)
  -O, --project-option      Set platformio.ini option (repeatable)
  --ide                     Generate IDE project files (atom, clion, codeblocks,
                            eclipse, emacs, netbeans, qtcreator, sublimetext, vim,
                            visualstudio, vscode)
  --sample-code             Generate sample "main" files
```

Examples:
```bash
# Multi-board project
pio project init -b esp32dev -b megaatmega2560 -O "framework=arduino"

# With specific framework and build flags
pio project init -b nucleo_f446re -O "framework=stm32cube" -O "build_flags=-DUSE_HAL_DRIVER"

# RISC-V project
pio project init -b sifive-hifive1-revb -O "framework=freedom-e-sdk"
```

### pio project config
Dump the computed project configuration (after interpolation).

```bash
pio project config [--json-output]
```

### pio project metadata
Extract build metadata (includes, defines, compiler paths) for IDE integration.

```bash
pio project metadata -e <environment> [--json-output]
```

## Building

### pio run
Build, upload, or execute project targets.

```bash
pio run [OPTIONS]

Options:
  -e, --environment NAME    Process specific environment(s) (repeatable)
  -t, --target NAME         Process specific target(s) (repeatable)
  --list-targets            List available targets for the project
  --upload-port PORT        Override upload port
  --monitor-port PORT       Override monitor port
  -d, --project-dir PATH   Project directory
  -j, --jobs N              Parallel build jobs (default: CPU count)
  -s, --silent              Suppress progress output
  -v, --verbose             Detailed build output
```

Built-in targets:
| Target | Description |
|--------|-------------|
| (none) | Build only |
| `upload` | Build and upload firmware |
| `monitor` | Start serial monitor after build |
| `clean` | Clean build files |
| `envdump` | Dump build environment variables |
| `compiledb` | Generate `compile_commands.json` |

Examples:
```bash
# Build all environments
pio run

# Build and upload specific environment
pio run -e esp32dev -t upload

# Build, upload, and monitor in one shot
pio run -e esp32dev -t upload -t monitor

# Clean build artifacts
pio run -t clean

# Verbose build to see compiler commands
pio run -v

# List all available targets
pio run --list-targets

# Override upload port
pio run -t upload --upload-port /dev/cu.usbserial-0001

# Parallel build with 8 jobs
pio run -j 8
```

## Device Discovery & Monitoring

### pio device list
List connected serial devices.

```bash
pio device list [OPTIONS]

Options:
  --serial        List serial ports (default)
  --logical       List logical devices
  --mdns          List mDNS services
  --json-output   Output as JSON
```

Example output:
```
/dev/cu.usbserial-0001
----------
Hardware ID: USB VID:PID=10C4:EA60 SER=0001
Description: CP2102 USB to UART Bridge Controller
```

### pio device monitor
Interactive serial port monitor.

```bash
pio device monitor [OPTIONS]

Options:
  -p, --port PORT           Serial port
  -b, --baud RATE           Baud rate (default: 9600)
  --parity {N,E,O,S,M}     Parity (default: N)
  --rtscts                  Enable RTS/CTS flow control
  --xonxoff                 Enable XON/XOFF flow control
  --rts {0,1}               Initial RTS state
  --dtr {0,1}               Initial DTR state
  --encoding CODEC          Output encoding (default: UTF-8)
  -f, --filter NAME         Apply output filter (repeatable)
  --eol {CR,LF,CRLF}       End-of-line character
  --raw                     Disable input/output transformations
  -q, --quiet               Suppress status messages
```

Available filters:
| Filter | Description |
|--------|-------------|
| `default` | Remove typical terminal control codes |
| `colorize` | Apply ANSI colors to output |
| `debug` | Print what is sent and received |
| `direct` | No transformations |
| `hexlify` | Show hex representation |
| `log2file` | Log output to a file |
| `nocontrol` | Remove all control codes |
| `printable` | Show only printable characters |
| `send_on_enter` | Send text on Enter key |
| `time` | Add timestamp to each line |
| `esp32_exception_decoder` | Decode ESP32 exception stack traces |
| `esp8266_exception_decoder` | Decode ESP8266 exception stack traces |

Examples:
```bash
# Basic monitor at 115200 baud
pio device monitor -b 115200

# Monitor with timestamp and hex output
pio device monitor -b 115200 -f time -f hexlify

# ESP32 with exception decoding
pio device monitor -b 115200 -f esp32_exception_decoder

# Specific port with DTR/RTS control (important for some boards)
pio device monitor -p /dev/cu.usbserial-0001 -b 9600 --dtr 0 --rts 0

# Log to file while monitoring
pio device monitor -b 115200 -f log2file
```

## Board & Platform Discovery

### pio boards
List supported development boards.

```bash
pio boards [OPTIONS] [FILTER]

Options:
  --installed     Only boards from installed platforms
  --json-output   Output as JSON
```

Output columns: ID, MCU, Frequency, Flash, RAM, Name

Examples:
```bash
# All boards (1500+)
pio boards

# Filter by keyword
pio boards esp32
pio boards atmega
pio boards stm32
pio boards risc-v
pio boards pic32
pio boards raspberry

# Only installed boards (fast, no network)
pio boards --installed

# JSON for parsing
pio boards --installed --json-output
```

### pio platform (via pio pkg)
Manage development platforms.

```bash
# List installed platforms
pio pkg list -g -t platform

# Search for platforms
pio pkg search "type:platform esp32"
pio pkg search "type:platform risc-v"

# Install a platform
pio pkg install -g -p "platformio/espressif32"
pio pkg install -g -p "platformio/atmelavr"

# Update platforms
pio pkg update -g
```

## Package & Library Management

### pio pkg search
Search the PlatformIO Registry (15,000+ libraries, 50+ platforms).

```bash
pio pkg search [OPTIONS] [QUERY]

Options:
  -p, --page N              Page number
  -s, --sort METHOD         Sort: relevance, popularity, trending, added, updated
```

Search qualifiers:
| Qualifier | Example | Description |
|-----------|---------|-------------|
| `type:library` | `type:library json` | Filter by package type |
| `type:platform` | `type:platform esp` | Search platforms |
| `type:tool` | `type:tool openocd` | Search tools |
| `tier:official` | `tier:official` | Official PlatformIO packages |
| `tier:verified` | `tier:verified` | Verified third-party packages |
| `name:"Exact Name"` | `name:"ArduinoJson"` | Search by exact name |
| `keyword:KEY` | `keyword:lora` | Search by keyword |
| `framework:NAME` | `framework:arduino` | Filter by framework compatibility |
| `platform:NAME` | `platform:espressif32` | Filter by platform compatibility |
| `owner:NAME` | `owner:adafruit` | Filter by package owner |
| `header:NAME` | `header:Wire.h` | Search by included header file |

Examples:
```bash
# Find LoRa libraries for Arduino
pio pkg search "type:library keyword:lora framework:arduino"

# Find I2C sensor libraries for ESP32
pio pkg search "type:library keyword:i2c platform:espressif32"

# Find libraries by a specific author
pio pkg search "type:library owner:adafruit"

# Search by header file (find who provides a header)
pio pkg search "type:library header:SPI.h"

# Trending libraries
pio pkg search "type:library" --sort trending

# Search for SX1276 LoRa chip libraries
pio pkg search "type:library SX1276"

# Search for PIC32 compatible libraries
pio pkg search "type:library platform:microchippic32"
```

### pio pkg install
Install packages into the current project or globally.

```bash
pio pkg install [OPTIONS]

Options:
  -l, --library SPEC        Install library (repeatable)
  -p, --platform SPEC       Install platform (repeatable)
  -t, --tool SPEC           Install tool (repeatable)
  -e, --environment NAME    Target specific environment
  --no-save                 Don't update platformio.ini
  -g, --global              Install globally
  -f, --force               Force reinstall
  --skip-dependencies       Skip transitive dependencies
```

Package specification formats:
```bash
# Registry (recommended — pin with ^)
pio pkg install -l "bblanchon/ArduinoJson@^6.21.0"

# Registry latest
pio pkg install -l "adafruit/Adafruit BME280 Library"

# Git repository
pio pkg install -l "https://github.com/sandeepmistry/arduino-LoRa.git"

# Git with specific tag
pio pkg install -l "https://github.com/sandeepmistry/arduino-LoRa.git#v0.8.0"

# Local folder (symlink — edits reflect immediately)
pio pkg install -l "symlink:///path/to/my/library"

# Local folder (copy)
pio pkg install -l "file:///path/to/my/library"
```

### pio pkg list
List installed packages for the current project.

```bash
pio pkg list [OPTIONS]

Options:
  -g, --global              List global packages
  -v, --verbose             Show detailed info
  --json-output             Output as JSON
```

### pio pkg update
Update project or global packages.

```bash
pio pkg update [OPTIONS]

Options:
  -g, --global              Update global packages
  -f, --force               Force update
```

### pio pkg show
Show detailed package information from the registry.

```bash
pio pkg show OWNER/NAME

# Example
pio pkg show "bblanchon/ArduinoJson"
```

## Debugging

### pio debug
Launch the PlatformIO Unified Debugger.

```bash
pio debug [OPTIONS]

Options:
  -e, --environment NAME    Debug specific environment
  -d, --project-dir PATH   Project directory
  --interface {gdb}         Debug interface (default: gdb)
  --load-mode {always,modified,changed,manual}  Firmware load mode
  -v, --verbose             Verbose output
```

Examples:
```bash
# Start debug session
pio debug

# Debug specific environment
pio debug -e nucleo_f446re

# Skip firmware reload (faster restart)
pio debug --load-mode manual

# Verbose for troubleshooting probe issues
pio debug -v
```

## Testing

### pio test
Run unit tests on hardware or native platform.

```bash
pio test [OPTIONS]

Options:
  -e, --environment NAME    Test specific environment
  -f, --filter PATTERN      Run tests matching pattern
  -i, --ignore PATTERN      Ignore tests matching pattern
  --upload-port PORT        Override upload port
  -v, --verbose             Verbose output
```

Examples:
```bash
# Run all tests
pio test

# Run tests for specific environment
pio test -e native

# Filter specific test suite
pio test -f test_sensors

# Ignore desktop-only tests when targeting hardware
pio test -i test_desktop
```

## Static Analysis

### pio check
Run static code analysis (cppcheck, pvs-studio, etc.).

```bash
pio check [OPTIONS]

Options:
  -e, --environment NAME    Check specific environment
  --filter PATTERN          Check specific source files
  --severity LEVEL          Minimum severity: low, medium, high
  --flags FLAGS             Extra checker flags
  -v, --verbose             Verbose output
```

## System

### pio system info
Display system-wide PlatformIO information.

```bash
pio system info
```

### pio system prune
Remove unused data (cached packages, old platforms, etc.).

```bash
pio system prune [--force] [--dry-run]
```

## Tips for Agents

- Always use `--json-output` when you need to parse results programmatically.
- Chain targets: `pio run -t upload -t monitor` is the most common embedded workflow.
- Use `-e` to target specific environments in multi-board projects.
- `pio run --list-targets` shows what's available for the current project.
- `pio device list --json-output` gives you reliable port paths for upload/monitor.
- When a board isn't found, install its platform first: `pio pkg install -g -p "platformio/<platform-name>"`.
- `pio pkg search` is your gateway to 15,000+ libraries with headers, examples, and compatibility data.
