# PlatformIO Debugging Guide

Unified Debugger configuration, debug tool setup, GDB usage, and common debugging workflows. Load this when debugging firmware.

## Overview

PlatformIO's Unified Debugger abstracts away the complexity of configuring GDB, OpenOCD, J-Link GDB Server, and other debug backends. It works across 300+ boards with zero-configuration for supported debug probes.

## Debug Configuration in platformio.ini

```ini
[env:myenv]
platform = ststm32
board = nucleo_f446re
framework = arduino

; Debug configuration
debug_tool = stlink             ; Debug probe type
debug_init_break = tbreak setup ; Where to break on start (Arduino)
; debug_init_break = tbreak main ; For non-Arduino frameworks
debug_speed = 4000              ; SWD/JTAG clock speed in kHz
debug_build_flags = -O0 -g3    ; Disable optimization, max debug info
debug_load_mode = always        ; always | modified | changed | manual
```

### debug_tool Options

Common debug probes and their identifiers:

| Tool | ID | Interface | Common Boards |
|------|----|-----------|---------------|
| ST-Link | `stlink` | SWD | STM32 Nucleo, Discovery |
| ST-Link V3 | `stlink` | SWD | Newer STM32 boards |
| J-Link | `jlink` | SWD/JTAG | Universal (commercial) |
| CMSIS-DAP | `cmsis-dap` | SWD | ARM mbed boards, RP2040 Debug Probe |
| Black Magic Probe | `blackmagic` | SWD/JTAG | Open-source probe |
| ESP-Prog | `esp-prog` | JTAG | ESP32 |
| Raspberry Pi (OpenOCD) | `raspberrypi-swd` | SWD | RP2040 Pico |
| Atmel-ICE | `atmel-ice` | JTAG/debugWIRE | AVR, SAM |
| PICkit | `pickit3` | ICSP | PIC microcontrollers |
| RISC-V OpenOCD | `olimex-arm-usb-ocd-h` | JTAG | SiFive, GD32V |
| Built-in USB JTAG | `esp-builtin` | USB-JTAG | ESP32-S3, ESP32-C3 |

### debug_load_mode Options

| Mode | Description | Use Case |
|------|-------------|----------|
| `always` | Upload firmware before every debug session | Default, safest |
| `modified` | Upload only if binary changed | Faster iteration |
| `changed` | Upload only if source changed | Fastest, may miss changes |
| `manual` | Never auto-upload | When using external programmer |

### debug_build_flags

For meaningful debugging, disable compiler optimizations:

```ini
; Full debug info, no optimization
debug_build_flags = -O0 -ggdb3

; Or use PlatformIO's debug build type
build_type = debug
```

Note: `-O0` significantly increases binary size. Some constrained MCUs (AVR with 32KB flash) may not fit debug builds. Use `-Og` (optimize for debugging) as a compromise.

## Starting a Debug Session

### CLI

```bash
# Start GDB debug session
pio debug

# Debug specific environment
pio debug -e nucleo_f446re

# Verbose output (useful for diagnosing probe issues)
pio debug -v

# Skip firmware upload (use existing firmware)
pio debug --load-mode manual
```

### Common GDB Commands

Once in the GDB session:

```gdb
# Execution control
continue (c)          # Resume execution
step (s)              # Step into function
next (n)              # Step over function
finish                # Run until current function returns
until <line>          # Run until specific line

# Breakpoints
break main            # Break at function
break main.cpp:42     # Break at file:line
break loop            # Break at Arduino loop()
watch variable_name   # Break when variable changes (hardware watchpoint)
info breakpoints      # List all breakpoints
delete <num>          # Delete breakpoint by number
disable <num>         # Temporarily disable breakpoint

# Inspection
print variable        # Print variable value
print/x variable      # Print in hex
print *pointer        # Dereference pointer
display variable      # Auto-print on every stop
info locals           # Show all local variables
info registers        # Show CPU registers
x/16xb 0x20000000    # Examine 16 bytes at address in hex

# Stack
backtrace (bt)        # Show call stack
frame <num>           # Switch to stack frame
up / down             # Navigate stack frames

# Memory
x/4xw 0x40021000     # Examine 4 words at peripheral register address
set *0x40021014 = 0x1 # Write to memory/register directly

# Peripheral registers (if SVD file loaded)
info registers        # Shows named peripheral registers

# Reset and restart
monitor reset halt    # Reset MCU and halt
load                  # Re-upload firmware
```

## Platform-Specific Debug Setup

### STM32 (ST-Link)

Most STM32 Nucleo and Discovery boards have an onboard ST-Link. Zero configuration needed.

```ini
[env:nucleo_f446re]
platform = ststm32
board = nucleo_f446re
framework = arduino
debug_tool = stlink
debug_init_break = tbreak setup
```

For external ST-Link with custom boards:
```ini
debug_tool = stlink
debug_port = /dev/ttyACM0    ; If multiple ST-Links connected
```

### ESP32 (JTAG)

ESP32 debugging requires a JTAG probe (ESP-Prog, J-Link, or built-in USB-JTAG on S3/C3).

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
debug_tool = esp-prog
debug_init_break = tbreak setup
debug_speed = 5000

; ESP32-S3 with built-in USB JTAG
[env:esp32s3]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
debug_tool = esp-builtin
```

ESP32 JTAG pin connections (ESP-Prog):
| ESP32 Pin | JTAG Signal |
|-----------|-------------|
| GPIO 12 | TDI |
| GPIO 13 | TCK |
| GPIO 14 | TMS |
| GPIO 15 | TDO |

**Important:** These GPIO pins cannot be used for other purposes during debugging.

### AVR (debugWIRE / JTAG)

AVR debugging is limited. Most ATmega boards use debugWIRE (single-wire) via Atmel-ICE or AVR Dragon.

```ini
[env:megaatmega2560]
platform = atmelavr
board = megaatmega2560
framework = arduino
debug_tool = atmel-ice
; Note: debugWIRE disables the RESET pin. Use JTAG on ATmega2560.
```

**Caveat:** AVR debugging support in PlatformIO is limited compared to ARM. For AVR, `Serial.print()` debugging is often more practical.

### RISC-V (SiFive, GD32V)

```ini
[env:sifive-hifive1-revb]
platform = sifive
board = sifive-hifive1-revb
framework = freedom-e-sdk
debug_tool = olimex-arm-usb-ocd-h
; Or use the onboard J-Link on HiFive1 Rev B
debug_tool = jlink
```

### RP2040 (Raspberry Pi Pico)

```ini
[env:pico]
platform = raspberrypi
board = pico
framework = arduino
debug_tool = cmsis-dap        ; Raspberry Pi Debug Probe
; Or use picoprobe
debug_tool = raspberrypi-swd
```

## Advanced Debug Techniques

### Custom GDB Init Commands

```ini
[env:myenv]
debug_extra_cmds =
    set print pretty on
    set print array on
    set pagination off
    define hook-stop
        info locals
    end
```

### SVD Files for Peripheral Register Viewing

SVD (System View Description) files provide named register maps for MCU peripherals. PlatformIO auto-loads them for supported boards.

```ini
; Override SVD file path if needed
debug_svd_path = path/to/MCU.svd
```

### Custom Debug Server

```ini
[env:myenv]
debug_tool = custom
debug_server =
    ${platformio.packages_dir}/tool-openocd/bin/openocd
    -f
    interface/stlink.cfg
    -f
    target/stm32f4x.cfg
debug_port = localhost:3333
```

### Semihosting (printf via debug probe)

Redirect `printf` output through the debug probe instead of UART:

```ini
[env:myenv]
debug_extra_cmds =
    monitor arm semihosting enable
build_flags =
    -D USE_SEMIHOSTING
    --specs=rdimon.specs
    -lrdimon
```

## Debugging Without a Probe

When hardware debuggers aren't available:

### Serial Print Debugging
The most common approach. Use structured output:
```cpp
#define DEBUG_PRINT(fmt, ...) Serial.printf("[%lu] " fmt "\n", millis(), ##__VA_ARGS__)

DEBUG_PRINT("Sensor value: %d, status: 0x%02X", value, status);
```

### LED Debugging
For boards without serial output:
```cpp
// Blink patterns to indicate state
void blink_error(uint8_t code) {
    for (uint8_t i = 0; i < code; i++) {
        digitalWrite(LED_PIN, HIGH);
        delay(200);
        digitalWrite(LED_PIN, LOW);
        delay(200);
    }
    delay(1000);
}
```

### ESP32 Exception Decoder
Decode stack traces from crashes:
```ini
monitor_filters = esp32_exception_decoder
```

### Core Dumps (ESP32)
```ini
build_flags = -D CONFIG_ESP_COREDUMP_ENABLE_TO_FLASH
```

## Common Debug Issues

### "Error: unable to find a matching CMSIS-DAP device"
- Check USB connection and driver installation.
- Verify the probe is recognized: `pio device list`.
- Try a different USB port or cable.

### "Error: target not halted"
- Reset the target manually (press reset button).
- Reduce debug speed: `debug_speed = 1000`.
- Check power supply to the target board.

### "Program too large for debug build"
- Use `-Og` instead of `-O0` for smaller debug builds.
- Reduce debug info level: `-g1` instead of `-g3`.
- Remove unused libraries.

### Breakpoints not hitting
- Ensure `build_type = debug` or `debug_build_flags = -O0 -g`.
- Compiler optimization can reorder or eliminate code.
- Hardware breakpoint limit: ARM Cortex-M has 2-8 hardware breakpoints depending on the core. Software breakpoints require writable flash.

### Variables showing "optimized out"
- Compile with `-O0` to prevent optimization.
- Mark variables as `volatile` if they're being optimized away.
- Use `__attribute__((used))` to prevent the compiler from removing unused variables.
