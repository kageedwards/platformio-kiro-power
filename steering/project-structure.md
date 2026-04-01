# PlatformIO Project Structure & Configuration

Reference for project layout, `platformio.ini` configuration, multi-environment setups, and advanced build scripting. Load this when setting up or restructuring a project.

## Directory Layout

```
project/
├── platformio.ini          # Project configuration (required)
├── src/                    # Main source code
│   └── main.cpp            # Entry point (setup/loop for Arduino, main for others)
├── include/                # Project header files
│   └── config.h            # Project-wide configuration
├── lib/                    # Project-specific (private) libraries
│   ├── MyDriver/
│   │   ├── MyDriver.h
│   │   └── MyDriver.cpp
│   └── README
├── test/                   # Unit tests
│   ├── test_sensors/
│   │   └── test_main.cpp
│   └── test_comms/
│       └── test_main.cpp
├── data/                   # SPIFFS/LittleFS filesystem data (ESP)
├── boards/                 # Custom board definitions (JSON)
├── extra_scripts/          # Custom SCons build scripts
└── .pio/                   # Build output (gitignore this)
    └── build/
        └── <env>/          # Per-environment build artifacts
```

### Key Directories
- **`src/`** — Main application source. PlatformIO compiles everything here.
- **`include/`** — Header files automatically added to the include path. Use for project-wide headers.
- **`lib/`** — Private libraries. Each subdirectory is treated as a separate library with its own namespace. PlatformIO's Library Dependency Finder (LDF) resolves includes automatically.
- **`test/`** — Unit test suites. Each subdirectory is a test suite. Run with `pio test`.
- **`data/`** — Filesystem data uploaded with `pio run -t uploadfs`.
- **`.pio/`** — Build artifacts. Always add to `.gitignore`.

## platformio.ini Configuration

### Section: [platformio]
Global project settings.

```ini
[platformio]
; Default environments to build (if not specified with -e)
default_envs = esp32dev, megaatmega2560

; Custom source directory (default: src)
src_dir = firmware/src

; Custom include directory (default: include)
include_dir = firmware/include

; Custom lib directory (default: lib)
lib_dir = firmware/lib

; Custom build directory (default: .pio/build)
build_dir = .pio/build

; Custom data directory for filesystem (default: data)
data_dir = data

; Extra library storage directories
lib_extra_dirs =
    ~/shared_libs
    ../common_libs
```

### Section: [env] (Common) and [env:NAME] (Specific)

Options in `[env]` apply to all environments. Options in `[env:NAME]` override for that specific environment.

```ini
; Common settings for all environments
[env]
framework = arduino
monitor_speed = 115200
lib_deps =
    bblanchon/ArduinoJson@^6.21.0

; ESP32 environment
[env:esp32dev]
platform = espressif32
board = esp32dev
build_flags =
    -D BOARD_ESP32
    -D WIFI_ENABLED

; ATmega2560 environment
[env:megaatmega2560]
platform = atmelavr
board = megaatmega2560
build_flags =
    -D BOARD_MEGA
```

### Core Options

```ini
[env:myenv]
; Platform and board (required)
platform = espressif32          ; Development platform
board = esp32dev                ; Board identifier
framework = arduino             ; Framework (arduino, espidf, mbed, etc.)

; Build options
build_flags =
    -D MY_DEFINE=42             ; Preprocessor define
    -Wall                       ; Enable all warnings
    -Wextra                     ; Extra warnings
    -O2                         ; Optimization level
    -std=gnu++17                ; C++ standard
build_unflags = -Os             ; Remove default flags
build_src_flags = -Wno-unused   ; Flags only for src/ files
build_src_filter =
    +<*>                        ; Include all by default
    -<test/>                    ; Exclude test directory
    +<main_${PIOENV}.cpp>       ; Dynamic source selection

; Upload options
upload_protocol = esptool       ; Upload protocol
upload_port = /dev/cu.usbserial-0001
upload_speed = 921600           ; Upload baud rate

; Monitor options
monitor_speed = 115200
monitor_port = /dev/cu.usbserial-0001
monitor_filters = esp32_exception_decoder, time
monitor_rts = 0                 ; RTS signal state
monitor_dtr = 0                 ; DTR signal state

; Debug options
debug_tool = jlink              ; Debug probe
debug_init_break = tbreak setup ; Break at setup() for Arduino
debug_speed = 4000              ; SWD/JTAG speed in kHz

; Test options
test_framework = unity          ; Test framework
test_ignore = test_desktop      ; Ignore test suites

; Library options
lib_deps =
    bblanchon/ArduinoJson@^6.21.0
    sandeepmistry/LoRa@^0.8.0
    https://github.com/me-no-dev/ESPAsyncWebServer.git
lib_extra_dirs = ~/my_libs
lib_ldf_mode = deep+            ; Library Dependency Finder mode
```

### Library Dependency Finder (LDF) Modes
| Mode | Description |
|------|-------------|
| `off` | No automatic dependency resolution |
| `chain` | Parse includes in src, then follow chain (default) |
| `deep` | Parse all includes in all source files |
| `chain+` | Like chain, but also parse library examples |
| `deep+` | Like deep, but also parse library examples |

### Interpolation and Custom Sections

```ini
; Custom section for shared values
[common]
build_flags =
    -D VERSION=1.0.0
    -D DEBUG_LEVEL=1
lib_deps =
    bblanchon/ArduinoJson@^6.21.0

; Reference common values with ${section.option}
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
build_flags =
    ${common.build_flags}
    -D BOARD_ESP32
lib_deps =
    ${common.lib_deps}
    sandeepmistry/LoRa@^0.8.0

[env:megaatmega2560]
platform = atmelavr
board = megaatmega2560
framework = arduino
build_flags =
    ${common.build_flags}
    -D BOARD_MEGA
lib_deps = ${common.lib_deps}
```

### Environment Variables in platformio.ini

```ini
; Use system environment variables with sysenv prefix
[env:myenv]
build_flags = -D API_KEY=\"${sysenv.MY_API_KEY}\"
upload_port = ${sysenv.UPLOAD_PORT}
```

## Multi-Environment Strategies

### Platform-Specific Source Code

Use `build_src_filter` or preprocessor defines:

```ini
[env:esp32dev]
build_flags = -D PLATFORM_ESP32
build_src_filter = +<*> -<platform_avr/>

[env:megaatmega2560]
build_flags = -D PLATFORM_AVR
build_src_filter = +<*> -<platform_esp32/>
```

Or use conditional compilation in code:
```cpp
#ifdef PLATFORM_ESP32
    #include "wifi_manager.h"
#elif defined(PLATFORM_AVR)
    #include "serial_manager.h"
#endif
```

### Shared Libraries with Platform Overrides

```ini
[common]
lib_deps =
    bblanchon/ArduinoJson@^6.21.0

[env:esp32dev]
lib_deps =
    ${common.lib_deps}
    WiFi                        ; ESP32 built-in
    sandeepmistry/LoRa@^0.8.0

[env:megaatmega2560]
lib_deps =
    ${common.lib_deps}
    SPI                         ; AVR built-in
```

## Custom Board Definitions

Place custom board JSON files in the `boards/` directory at the project root.

```json
// boards/my_custom_board.json
{
    "build": {
        "core": "esp32",
        "extra_flags": "-DARDUINO_MY_BOARD",
        "f_cpu": "240000000L",
        "f_flash": "40000000L",
        "flash_mode": "dio",
        "mcu": "esp32"
    },
    "connectivity": ["wifi", "bluetooth"],
    "frameworks": ["arduino", "espidf"],
    "name": "My Custom ESP32 Board",
    "upload": {
        "flash_size": "4MB",
        "maximum_ram_size": 327680,
        "maximum_size": 4194304,
        "speed": 460800
    },
    "url": "https://example.com/my-board",
    "vendor": "My Company"
}
```

Use it in `platformio.ini`:
```ini
[env:my_custom_board]
platform = espressif32
board = my_custom_board
framework = arduino
```

## Advanced Build Scripting (extra_scripts)

PlatformIO uses SCons under the hood. Custom Python scripts can hook into the build process.

```ini
[env:myenv]
extra_scripts =
    pre:extra_scripts/pre_build.py
    post:extra_scripts/post_build.py
```

Example pre-build script (auto-generate version header):
```python
# extra_scripts/pre_build.py
Import("env")
import datetime

def generate_version_header(source, target, env):
    with open("include/version.h", "w") as f:
        f.write(f'#define BUILD_DATE "{datetime.datetime.now().isoformat()}"\n')
        f.write(f'#define BUILD_ENV "{env["PIOENV"]}"\n')

env.AddPreAction("buildprog", generate_version_header)
```

Example post-build script (copy firmware to custom location):
```python
# extra_scripts/post_build.py
Import("env")
import shutil

def copy_firmware(source, target, env):
    firmware = str(target[0])
    shutil.copy(firmware, "output/firmware.bin")

env.AddPostAction("buildprog", copy_firmware)
```

## Filesystem (SPIFFS/LittleFS)

For ESP32/ESP8266, upload files to the on-chip filesystem:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
board_build.filesystem = littlefs    ; or spiffs
```

```bash
# Upload filesystem image
pio run -t uploadfs

# Build filesystem image only
pio run -t buildfs
```

Place files in the `data/` directory. They'll be accessible via the filesystem API in code.

## Platform Packages Override

Override default toolchain or framework versions:

```ini
[env:myenv]
platform = espressif32
platform_packages =
    ; Use specific toolchain version
    toolchain-xtensa-esp32@8.4.0+2021r2
    ; Use framework from git
    framework-arduinoespressif32@https://github.com/espressif/arduino-esp32.git#2.0.14
```

## .gitignore Template

```
.pio/
.vscode/.browse.c_cpp.db*
.vscode/c_cpp_properties.json
.vscode/launch.json
.vscode/ipch
```

Keep `platformio.ini` and all source directories in version control. Never commit `.pio/`.
