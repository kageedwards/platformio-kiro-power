# Project Initialization Workflow

Step-by-step procedure for creating a new PlatformIO project. This steering file ensures consistent, reproducible project setup regardless of context. Load this whenever a user wants to create, scaffold, or initialize a new PlatformIO project.

## Initialization Sequence

Follow these steps in order. Do not skip steps. Do not proceed to the next step until the current step succeeds.

### Step 1: Verify Prerequisites

Run both commands and confirm success before continuing:

```bash
pio --version
pio system info
```

If `pio` is not found, guide the user through installation (see POWER.md Onboarding). Do not proceed until `pio --version` returns a valid version string.

#### IDE IntelliSense Check

After confirming `pio` is available, check whether the user has C/C++ IntelliSense set up in their IDE. This power runs in Kiro IDE (a VS Code fork). Kiro may not have the Microsoft Marketplace, so extensions like PlatformIO IDE and Microsoft C/C++ may not be directly searchable.

Ask the user (once, at the start of the first project setup):
- "Do you have a C/C++ language server extension installed (clangd or Microsoft C/C++)? This is needed for IntelliSense — code completion, go-to-definition, and error highlighting in your source files."

If they don't, or aren't sure, guide them:

1. **Try clangd first** (recommended for Kiro). Search the extension marketplace for "clangd". If found, install it. If not found in the marketplace, the VSIX can be downloaded from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd) and installed via Command Palette → "Extensions: Install from VSIX...".
2. **Alternative: Microsoft C/C++.** If the user prefers or needs the Microsoft C/C++ extension (e.g., for PlatformIO IDE extension compatibility), it must be sideloaded as a VSIX from the [Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools). Note: this extension has platform-specific builds — the user must download the correct one for their OS/architecture.
3. **PlatformIO IDE extension** is optional. This power uses the CLI directly. If the user wants the GUI integration, it can also be sideloaded as a VSIX, but it depends on the Microsoft C/C++ extension.

Do not block project creation on this — the user can set up IntelliSense later. But mention it early so they're aware. The compilation database setup in Step 11 will complete the IntelliSense configuration.

### Step 2: Determine Project Root Directory

Apply these rules in order:

1. **Default is the workspace root.** Unless the user specifies otherwise, the project root is the workspace root directory.
2. **Ask the user if the workspace root is non-empty.** If the workspace root contains files or directories beyond dotfiles (`.git`, `.gitignore`, `.vscode`, `.kiro`, etc.), present the user with what's there and ask:
   - "This directory already has content. Should I initialize the PlatformIO project here, or would you prefer a subdirectory?"
3. **If the user specifies a directory**, use it. Create it if it doesn't exist.
4. **Never silently pick a subdirectory.** The agent must not invent a project directory name without the user's explicit approval.

Store the resolved project root path. All subsequent commands use `-d <project_root>` or `cwd` set to this path.

### Step 3: Identify Target Board

The user may provide a board name, a chip name, a dev board product name, or a vague description. Resolve it to a valid PlatformIO board ID:

1. **If the user gave an exact board ID** (e.g., `esp32dev`, `megaatmega2560`), validate it:
   ```bash
   pio boards <board_id> --json-output
   ```
   Confirm the output contains the board. If not, search more broadly.

2. **If the user gave a partial or informal name** (e.g., "ESP32", "Arduino Mega", "Pico"), search:
   ```bash
   pio boards <search_term> --json-output
   ```
   Present the matching boards to the user and ask them to confirm which one. Include the board ID, MCU, frequency, flash, and RAM from the output.

3. **If multiple boards match**, always ask the user to pick one. Never guess.

4. **If no boards match**, or the user's board is a newer revision not yet in the PlatformIO registry, proceed to Step 3b.

5. **Record the confirmed board ID** for use in subsequent steps.

### Step 3b: Board Not in Registry — Research and Resolve

When the user's specific board or revision is not found in `pio boards`, do not give up or silently substitute a different board. Follow this procedure:

1. **Search online** for the board's PlatformIO compatibility. Use web search with queries like:
   - `"<board name>" PlatformIO platformio.ini`
   - `"<board name>" PlatformIO custom board definition`
   - `"<board name>" site:<manufacturer-domain>`
   Check the manufacturer's official documentation, community forums (community.platformio.org, forum.arduino.cc), and GitHub for guidance.

2. **Determine the resolution path.** There are two outcomes:

   **Path A — Compatible existing board definition.** The board is electrically and pin-compatible with an existing PlatformIO board (e.g., a V4 revision that uses the same MCU and pin layout as a V3 already in the registry). In this case:
   - Use the compatible board ID for `pio project init`.
   - Note the substitution to the user and explain why it works.
   - If there are known differences (different pin assignments, additional peripherals, changed GPIOs), document them in a comment in `platformio.ini` or the source file.

   **Path B — Custom board definition required.** The board has a different MCU, pin mapping, flash size, or other hardware differences that make existing definitions incorrect. In this case:
   - Create a `boards/` directory in the project root.
   - Create a `<board_name>.json` file based on information from the manufacturer's documentation or community sources.
   - The JSON must include at minimum: `build` (mcu, core, f_cpu, flash_mode, extra_flags, partitions if applicable), `frameworks`, `name`, `upload` (flash_size, maximum_size, maximum_ram_size, speed), and `vendor`.
   - If the board requires a custom pin variant (different pin assignments from the base MCU board), also create a `variants/<board_variant>/pins_arduino.h` with the correct pin definitions, and set `board_build.variants_dir` and `board_build.variant` in `platformio.ini`.
   - Use the custom board ID (filename without `.json`) in `platformio.ini`.

3. **Always cite your sources.** Tell the user where the board definition or compatibility information came from (manufacturer wiki, GitHub repo, forum post, etc.) so they can verify.

4. **If you cannot find reliable information**, tell the user. Do not fabricate board definitions. Suggest they check the manufacturer's documentation or community forums, and offer to use the closest compatible board as a starting point with caveats noted.

### Step 4: Determine Framework

Each board supports one or more frameworks. The correct framework depends on the board and the user's intent.

1. **Check supported frameworks** from the board JSON output (Step 3). The `frameworks` array lists what's available.
2. **Common mappings:**
   | Framework | Typical Use |
   |-----------|-------------|
   | `arduino` | Most common for hobbyist/maker boards (ESP32, AVR, STM32, RP2040) |
   | `espidf` | ESP-IDF for ESP32 (advanced, FreeRTOS-based) |
   | `stm32cube` | STM32 HAL/LL (advanced STM32 development) |
   | `mbed` | ARM Mbed OS (Nucleo, mbed-enabled boards) |
   | `freedom-e-sdk` | SiFive RISC-V boards |
   | `zephyr` | Zephyr RTOS (cross-platform) |
3. **If the board supports only one framework**, use it without asking.
4. **If the board supports multiple frameworks**, ask the user which one they want. Default suggestion should be `arduino` if available, as it's the most common starting point.
5. **Record the confirmed framework.**

### Step 5: Initialize the Project

Run `pio project init` with all resolved parameters:

```bash
pio project init -d <project_root> --board <board_id> -O "framework=<framework>"
```

Verify the command succeeds. It should create:
- `platformio.ini`
- `src/`
- `include/`
- `lib/`
- `test/`

If the command fails, read the error output and troubleshoot. Common issues:
- Platform not installed → PlatformIO will auto-install it, but network is required.
- Invalid board ID → go back to Step 3.

### Step 6: Generate Boilerplate Source File

`pio project init` creates the directory structure but does NOT create a source file. You must create the correct entry point file based on the framework.

**Select the correct template from the section below and write it to the project.**


#### Arduino Framework → `src/main.cpp`

For boards using `framework = arduino`. This covers ESP32, ESP8266, AVR (Uno, Mega, Nano), STM32 with Arduino, RP2040 with Arduino, SAMD, nRF52 with Arduino, etc.

```cpp
#include <Arduino.h>

void setup() {
    Serial.begin(115200);
    while (!Serial) {
        delay(10);  // Wait for serial port (native USB boards)
    }
    Serial.println("Ready");
}

void loop() {
    // Main application loop
}
```

**Board-specific notes for Arduino boilerplate:**
- **AVR boards (Uno, Mega, Nano):** `while (!Serial)` is unnecessary (hardware UART), but harmless. Can be omitted for minimal code.
- **ESP32/ESP8266:** `while (!Serial)` is unnecessary (hardware UART via USB bridge), but harmless.
- **Native USB boards (Leonardo, Micro, SAMD21, RP2040, ESP32-S2/S3):** `while (!Serial)` is important — without it, early serial output is lost.
- **Baud rate:** 115200 is the safe default. Match `monitor_speed` in `platformio.ini`.

#### ESP-IDF Framework → `src/main.c`

For ESP32 boards using `framework = espidf`.

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"

static const char *TAG = "main";

void app_main(void) {
    ESP_LOGI(TAG, "Application started");

    while (1) {
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

#### STM32Cube Framework → `src/main.c`

For STM32 boards using `framework = stm32cube`.

```c
#include "stm32f4xx_hal.h"

void SystemClock_Config(void);
void Error_Handler(void);

int main(void) {
    HAL_Init();
    SystemClock_Config();

    while (1) {
        HAL_Delay(1000);
    }
}

void SystemClock_Config(void) {
    // Clock configuration is board-specific.
    // Refer to the STM32CubeMX generated code or the reference manual
    // for your specific MCU (e.g., RM0390 for STM32F446).
}

void Error_Handler(void) {
    __disable_irq();
    while (1) {
    }
}
```

**Note:** STM32Cube boilerplate is highly MCU-specific. The `SystemClock_Config` function must be tailored to the exact chip. For production projects, generate the initial configuration with STM32CubeMX and import it. The skeleton above compiles but uses default clock settings.

#### Mbed Framework → `src/main.cpp`

For ARM boards using `framework = mbed`.

```cpp
#include "mbed.h"

int main() {
    printf("Ready\n");

    while (1) {
        ThisThread::sleep_for(1000ms);
    }
}
```

#### Freedom-E-SDK (RISC-V) → `src/main.c`

For SiFive boards using `framework = freedom-e-sdk`.

```c
#include <stdio.h>
#include <metal/cpu.h>
#include <metal/led.h>

int main(void) {
    printf("Ready\n");

    while (1) {
        // Main application loop
    }

    return 0;
}
```

#### Zephyr RTOS → `src/main.c`

For boards using `framework = zephyr`.

```c
#include <zephyr/kernel.h>
#include <zephyr/sys/printk.h>

int main(void) {
    printk("Ready\n");

    while (1) {
        k_msleep(1000);
    }

    return 0;
}
```

#### Fallback (Unknown Framework) → `src/main.c`

If the framework is not listed above, generate a minimal C entry point:

```c
int main(void) {
    while (1) {
        // Main application loop
    }
    return 0;
}
```

Then inform the user that the boilerplate is minimal and they may need to add framework-specific includes and initialization.

### Step 7: Peripheral-Aware Boilerplate Enhancement

The generic boilerplate from Step 6 gets the project compiling, but many boards have on-board peripherals that define their purpose — LoRa radios, OLED displays, IMUs, GPS modules, environmental sensors, etc. A user choosing a specific board (rather than a generic dev board) almost certainly intends to use those peripherals.

After writing the generic boilerplate, follow this procedure:

1. **Identify on-board peripherals.** Use the information gathered during board identification (Step 3/3b):
   - The `connectivity` array in the board JSON (e.g., `["wifi", "bluetooth", "lora"]`).
   - The board's product page or datasheet (if consulted during Step 3b).
   - Common knowledge about the board family (e.g., Heltec LoRa boards have an SSD1306 OLED and SX1262 radio; Adafruit Feather Sense has a suite of sensors).

2. **Classify the board:**
   - **General-purpose dev board** (e.g., `esp32dev`, `uno`, `nucleo_f446re`): No special on-board peripherals beyond the MCU and basic I/O. The generic boilerplate from Step 6 is sufficient. Skip to Step 8.
   - **Feature board** (e.g., Heltec WiFi LoRa 32, Adafruit Feather Sense, TTGO T-Beam, LilyGO T-Display): Has specific on-board peripherals that are central to the board's purpose. Continue to sub-step 3.

3. **Ask the user about peripheral initialization.** Present what you know about the board's on-board peripherals and ask:
   - "This board has [LoRa radio / OLED display / GPS / etc.] on board. Would you like the boilerplate to include initialization code for any of these, or would you prefer to start with the minimal skeleton and add peripheral support later?"

4. **If the user wants peripheral support**, for each requested peripheral:
   - **Search the PlatformIO registry** for the appropriate library:
     ```bash
     pio pkg search "type:library <peripheral> platform:<platform>"
     ```
   - **Prefer well-maintained, popular libraries.** For common peripherals:
     - LoRa (SX1276/SX1262): `jgromes/RadioLib` (multi-radio, actively maintained) or board-specific libraries
     - OLED SSD1306: `thingpulse/ESP8266 and ESP32 OLED driver for SSD1306 displays` or board vendor libraries
     - GPS (u-blox): `mikalhart/TinyGPSPlus`
     - IMU (MPU6050/BMI270): `adafruit/Adafruit MPU6050` or vendor-specific
     - Environmental (BME280): `adafruit/Adafruit BME280 Library`
   - **Check for board-specific community libraries** that wrap multiple peripherals into a single API. These are often the best option for feature boards (e.g., `ropg/heltec_esp32_lora_v3` for Heltec boards wraps radio, display, button, LED, and battery monitoring).
   - **Add the library to `lib_deps`** in `platformio.ini`.
   - **Update the boilerplate source** to include initialization for the peripheral. Follow the library's examples for the correct init sequence. Pay attention to:
     - Pin assignments (use the board's defined pin constants, not hardcoded numbers)
     - Power control pins (many boards gate peripheral power via a GPIO — e.g., Heltec's `Vext` pin must be driven LOW to power the OLED and sensors)
     - SPI/I2C bus configuration specific to the board's wiring

5. **If the user declines or you're unsure**, keep the generic boilerplate. Add a comment noting the available peripherals:
   ```cpp
   // This board has on-board: [LoRa SX1262, SSD1306 OLED, LED on GPIO 35]
   // Add peripheral libraries to lib_deps in platformio.ini when ready.
   ```

6. **Rebuild after any changes** to verify the enhanced boilerplate compiles:
   ```bash
   pio run -d <project_root>
   ```

### Step 8: Configure platformio.ini Defaults

After `pio project init` creates the base `platformio.ini`, verify it contains the correct environment and add standard defaults if missing:

```ini
[env:<board_id>]
platform = <auto-populated by pio init>
board = <board_id>
framework = <framework>
monitor_speed = 115200
```

The `monitor_speed` line is important — without it, `pio device monitor` defaults to 9600 baud, which won't match the 115200 in the boilerplate source.

**Do not add options the user didn't ask for.** Keep the ini minimal. Additional configuration (build flags, lib_deps, debug settings) should be added as needed during development, not at init time.

### Step 9: Set Up .gitignore

If a `.gitignore` doesn't already exist in the project root, create one:

```
.pio/
.vscode/.browse.c_cpp.db*
.vscode/c_cpp_properties.json
.vscode/launch.json
.vscode/ipch
```

If a `.gitignore` already exists, append the `.pio/` entry if it's not already present. Do not overwrite existing content.

### Step 10: Verify Build

Run a build to confirm everything is wired up correctly:

```bash
pio run -d <project_root>
```

If the build succeeds, the project is ready. Report success to the user with:
- The project root path
- The board and framework
- The entry point file created
- Confirmation that the build passed

If the build fails, read the error output and fix the issue before reporting success. Common causes:
- Missing platform (auto-installs on first build, may need network)
- Incorrect framework for the board
- Boilerplate includes that don't match the framework version

### Step 11: Set Up IntelliSense (Compilation Database)

After a successful build, generate the compilation database so the C/C++ language server can provide accurate IntelliSense. This step is essential if the user is using clangd. It also benefits the Microsoft C/C++ extension.

1. **Create the compilation database helper script.** PlatformIO's `compiledb` target does not include toolchain header paths by default, which causes clangd to report false errors on standard library and toolchain headers. Fix this by adding an extra script.

   Add to `platformio.ini` (in the `[env]` or specific environment section):
   ```ini
   extra_scripts = pre:pre_compiledb.py
   ```

   Create `pre_compiledb.py` in the project root:
   ```python
   import os
   Import("env")

   # Include toolchain paths in compilation database
   env.Replace(COMPILATIONDB_INCLUDE_TOOLCHAIN=True)

   # Place compile_commands.json in the project root (where clangd expects it)
   env.Replace(COMPILATIONDB_PATH=os.path.join(
       env.subst("$PROJECT_DIR"), "compile_commands.json"
   ))
   ```

2. **Generate the compilation database:**
   ```bash
   pio run -d <project_root> -t compiledb
   ```
   This creates `compile_commands.json` in the project root.

3. **Add `compile_commands.json` to `.gitignore`** (it's machine-specific and regenerated on demand):
   ```
   compile_commands.json
   ```

4. **For clangd users**, create a `.clangd` config file in the project root. This is **required**, not optional — without it, clangd will fail to index the standard library and produce false errors throughout the project.

   **Why this is necessary:** PlatformIO cross-compilation toolchains use compiler targets (like `xtensa-esp32-elf`) and flags that clangd does not recognize. clangd is built on LLVM, which does not include an xtensa backend. When clangd encounters an unknown target or unrecognized flags, it fails to parse standard library headers entirely, producing errors like "incomplete due to errors" in the clangd log and false squiggles like "No matching member function for call to 'println'" in the editor. The fix is to override the target to a known LLVM triple and strip the unrecognized flags.

   **The `.clangd` config must do two things:**
   1. **Add a known `--target`** so clangd can parse standard library and toolchain headers. Use a generic triple like `x86_64-linux-gnu` — this does not affect the actual build (which still uses the real cross-compiler via `pio run`), it only tells clangd's analyzer how to interpret the headers.
   2. **Remove unrecognized compiler flags** that the cross-toolchain passes but clangd/LLVM does not understand.

   Select the correct `.clangd` config based on the project's platform:

   **ESP32 (Xtensa) — `esp32dev`, `esp32-s3`, `esp32-c3` (xtensa variants), Heltec, TTGO, etc.:**
   ```yaml
   CompileFlags:
     Add:
       - --target=x86_64-linux-gnu
     Remove:
       - -mtext-section-literals
       - -mlongcalls
       - -fstrict-volatile-bitfields
       - -free
       - -fipa-pta
       - -fno-jump-tables
       - -fno-tree-switch-conversion
       - -mfix-esp32-psram-cache-issue
       - -mfix-esp32-psram-cache-strategy=memw
   ```

   **ESP32-C3 / ESP32-C6 / ESP32-H2 (RISC-V variants):**
   ```yaml
   CompileFlags:
     Add:
       - --target=riscv32-unknown-elf
     Remove:
       - -fstrict-volatile-bitfields
       - -free
       - -fipa-pta
       - -fno-jump-tables
       - -fno-tree-switch-conversion
       - -march=rv32imc_zicsr_zifencei
       - -mabi=ilp32
   ```
   Note: clangd has RISC-V support, so a RISC-V target triple works here. Adjust `-march` removal if clangd recognizes the specific extension string in your toolchain version.

   **AVR (Arduino Uno, Mega, Nano):**
   ```yaml
   CompileFlags:
     Add:
       - --target=avr
     Remove:
       - -mmcu=*
   ```
   Note: clangd has basic AVR support. The `-mmcu` flag specifies the exact chip (e.g., `atmega328p`) and may or may not be recognized depending on your clangd version. If you still see errors, switch to `--target=x86_64-linux-gnu` as a fallback.

   **ARM (STM32, nRF52, RP2040, SAMD, Teensy):**
   ```yaml
   CompileFlags:
     Add:
       - --target=arm-none-eabi
     Remove:
       - -mfloat-abi=*
       - -mfpu=*
   ```
   Note: clangd has good ARM support. Most ARM flags are recognized. Only remove flags that cause specific errors in your setup.

   **SiFive / RISC-V (HiFive, freedom-e-sdk):**
   ```yaml
   CompileFlags:
     Add:
       - --target=riscv32-unknown-elf
     Remove:
       - -fstrict-volatile-bitfields
   ```

   **Fallback (any platform with clangd errors):**
   If you're unsure which flags to remove, or the platform-specific config above doesn't resolve all errors:
   ```yaml
   CompileFlags:
     Add:
       - --target=x86_64-linux-gnu
     Remove:
       - -*
   ```
   This nuclear option removes all flags and uses a generic target. It sacrifices some cross-compilation-specific diagnostics but guarantees clangd can parse the code. Refine from here by adding back flags that clangd recognizes.

   **Diagnosing remaining issues:** If clangd still shows errors after applying the config:
   - Check the clangd log (Output panel → "clangd" channel) for "unknown argument" or "unknown target" messages.
   - Inspect `compile_commands.json` for the specific flags being passed.
   - Add any unrecognized flags to the `Remove` list in `.clangd`.

5. **Verify IntelliSense.** Tell the user to open `src/main.cpp` (or `.c`) and check:
   - Framework headers (e.g., `#include <Arduino.h>`) resolve without errors
   - Hovering over functions shows signatures
   - Go-to-definition (F12) navigates to framework/library source

   If IntelliSense still shows errors, the most common causes are:
   - No C/C++ language server extension installed (see Step 1 IDE check)
   - `compile_commands.json` not generated (re-run `pio run -t compiledb`)
   - Toolchain flags not recognized by clangd (update `.clangd` Remove list)

## Multi-Board Projects

If the user wants to target multiple boards in one project:

1. Gather all board IDs and frameworks (Steps 3-4 for each board).
2. Initialize with multiple `--board` flags:
   ```bash
   pio project init -d <project_root> --board <board1> --board <board2> -O "framework=<framework>"
   ```
3. The boilerplate source file should use the framework common to all boards. If boards use different frameworks, this is an advanced setup — discuss with the user before proceeding.
4. Add `monitor_speed = 115200` to the `[env]` (common) section rather than per-environment.
5. Build all environments: `pio run -d <project_root>` (builds all), or build each individually to verify.

## Summary Checklist

Before reporting project creation as complete, verify all of the following:

- [ ] `pio --version` confirmed working
- [ ] IDE IntelliSense extension status checked with user (clangd or Microsoft C/C++)
- [ ] Project root directory confirmed with user
- [ ] Board ID validated against `pio boards` (or custom board definition created with sources cited)
- [ ] Framework confirmed (auto-selected or user-chosen)
- [ ] `pio project init` succeeded
- [ ] Correct boilerplate source file written (framework-matched)
- [ ] On-board peripherals identified and user consulted on peripheral initialization
- [ ] `platformio.ini` has `monitor_speed = 115200`
- [ ] `.gitignore` includes `.pio/`
- [ ] `pio run` build succeeded
- [ ] Compilation database generated (`pio run -t compiledb`) with `pre_compiledb.py` for toolchain paths
- [ ] `compile_commands.json` added to `.gitignore`
- [ ] `.clangd` config created with correct `--target` and flag removals for the platform (required for clangd users)
