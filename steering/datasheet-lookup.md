# Datasheet & Technical Reference Lookup

Methodical approach to finding datasheets, technical reference manuals, and peripheral documentation for MCUs and external devices. Load this when you need chip or peripheral specifications.

## Lookup Strategy

When you need technical specifications for a chip, module, or peripheral, follow this priority order:

### 1. Identify the Exact Part Number

Before searching, determine the full part number. Examples:
- MCU: `ATmega2560-16AU`, `ESP32-WROOM-32E`, `STM32F446RET6`, `FE310-G002`
- LoRa: `SX1276IMLTRT`, `SX1262IMLTRT`, `RFM95W`
- Sensors: `BME280`, `MPU6050`, `ADS1115`
- Interfaces: `MAX3232`, `CP2102N`, `CH340G`, `FT232RL`
- Memory: `W25Q128JVSIQ`, `AT24C256`

Sources for part numbers:
- Board schematic or silkscreen markings
- `platformio.ini` board definition → MCU field
- `pio boards <filter> --json-output` → mcu field
- Module vendor product page

### 2. Trusted Datasheet Sources (Priority Order)

#### Manufacturer Websites (Primary — Always Prefer)
These are the authoritative sources. Always check here first.

| Manufacturer | URL Pattern | Covers |
|-------------|-------------|--------|
| Microchip (+ Atmel) | `https://www.microchip.com/en-us/search?searchQuery=<part>` | AVR, PIC, SAM, AT series |
| Espressif | `https://www.espressif.com/en/support/documents/technical-documents` | ESP32, ESP8266, ESP32-S/C/H series |
| STMicroelectronics | `https://www.st.com/en/search.html?q=<part>` | STM32, STM8 |
| Texas Instruments | `https://www.ti.com/search?searchterm=<part>` | MSP430, CC series, analog ICs |
| Nordic Semiconductor | `https://www.nordicsemi.com/Products` | nRF52, nRF53, nRF91 |
| NXP | `https://www.nxp.com/search?keyword=<part>` | LPC, i.MX, Kinetis |
| SiFive | `https://www.sifive.com/documentation` | FE310, RISC-V cores |
| Semtech | `https://www.semtech.com/products/wireless-rf/lora-connect` | SX1276, SX1262, LoRa chipsets |
| Bosch Sensortec | `https://www.bosch-sensortec.com/products/` | BME280, BMP390, BMI270 |
| InvenSense (TDK) | `https://invensense.tdk.com/products/` | MPU6050, ICM series |
| FTDI | `https://ftdichip.com/products/` | FT232, FT2232 USB bridges |
| Silicon Labs | `https://www.silabs.com/search` | CP2102, EFM32, EFR32 |
| WCH (Nanjing) | `https://www.wch-ic.com/products/` | CH340, CH9102 USB bridges |
| Analog Devices (+ Maxim) | `https://www.analog.com/en/search.html?q=<part>` | MAX232, ADCs, DACs |
| Raspberry Pi | `https://www.raspberrypi.com/documentation/microcontrollers/` | RP2040, RP2350 |

#### Aggregator Sites (Secondary — When Manufacturer Site Fails)

| Site | URL | Notes |
|------|-----|-------|
| Octopart | `https://octopart.com/search?q=<part>` | Links to manufacturer datasheets, shows availability |
| AllDatasheet | `https://www.alldatasheet.com/view.jsp?Searchword=<part>` | Large archive, may have older revisions |
| Mouser | `https://www.mouser.com/search/refine?keyword=<part>` | Distributor with datasheet links |
| DigiKey | `https://www.digikey.com/en/products/result?keywords=<part>` | Distributor with datasheet links |

### 3. What to Look For in a Datasheet

Depending on your task, focus on these sections:

#### For Pin Configuration
- Pinout diagram / pin assignment table
- Alternate function mapping (which pins support SPI, I2C, UART, PWM, ADC)
- Electrical characteristics (max current per pin, voltage levels)

#### For Communication Peripherals
- Register map and bit field descriptions
- Timing diagrams (setup time, hold time, clock requirements)
- Supported modes and speeds
- DMA channel assignments
- Interrupt sources and vectors

#### For Power Management
- Operating voltage range
- Current consumption (active, idle, sleep, deep sleep modes)
- Power-on reset (POR) thresholds
- Brown-out detection (BOD) configuration

#### For Memory
- Flash size and page/sector erase sizes
- RAM size and layout (SRAM, DRAM, TCM)
- Memory map (peripheral base addresses)
- Bootloader memory regions

#### For Clock Configuration
- Internal oscillator accuracy
- External crystal requirements
- PLL configuration ranges
- Clock tree diagram

### 4. Technical Reference Manual vs Datasheet

Many MCU families have two documents:

| Document | Contains | When to Use |
|----------|----------|-------------|
| **Datasheet** | Pinout, electrical specs, package info, ordering codes | Hardware design, pin selection |
| **Technical Reference Manual (TRM)** | Detailed peripheral registers, programming model, timing | Firmware development, driver writing |

For example, STM32F446:
- Datasheet: `STM32F446xC/E` — pinout, electrical characteristics
- Reference Manual: `RM0390` — 1800+ pages of peripheral details

ESP32:
- Datasheet: `ESP32` — module specs, pin definitions
- Technical Reference Manual: `ESP32 TRM` — register-level peripheral documentation

### 5. Application Notes and Errata

Always check for:
- **Errata sheets** — Known silicon bugs and workarounds. Critical for production firmware.
- **Application notes** — Manufacturer-provided example implementations and design guidance.
- **Migration guides** — When moving between chip revisions or families.

Search pattern: `<manufacturer> <part> errata` or `<manufacturer> <part> application note`

### 6. Module vs Chip Documentation

Many development boards use modules (e.g., ESP32-WROOM-32E) that contain a chip (ESP32-D0WD-V3). You may need both:

- **Module datasheet:** Physical dimensions, antenna specs, recommended PCB layout, pin mapping
- **Chip datasheet/TRM:** Register-level programming, peripheral capabilities, electrical limits

### 7. Community Resources (Tertiary — For Practical Guidance)

When official docs are insufficient for practical implementation:

| Resource | URL | Best For |
|----------|-----|----------|
| Arduino Reference | `https://www.arduino.cc/reference/en/` | Arduino API documentation |
| ESP-IDF Docs | `https://docs.espressif.com/projects/esp-idf/en/latest/` | ESP32 framework-level docs |
| STM32 HAL Docs | `https://www.st.com/en/embedded-software/stm32cube-mcu-mpu-packages.html` | STM32 HAL/LL driver reference |
| Random Nerd Tutorials | `https://randomnerdtutorials.com/` | ESP32/ESP8266 practical tutorials |
| Adafruit Learning | `https://learn.adafruit.com/` | Sensor/module hookup guides |

## Agent Workflow for Datasheet Lookup

When you need technical information about a component:

1. **Identify the exact part number** from the project context (schematic, platformio.ini, user description).
2. **Search the manufacturer's website first** using web search: `"<part number> datasheet site:<manufacturer-domain>"`.
3. **Fetch the relevant page** to find the direct PDF link or HTML documentation.
4. **If manufacturer site fails**, try Octopart or distributor sites.
5. **Extract only the specific information needed** — don't try to summarize entire datasheets.
6. **Cite the source** — always tell the user where the information came from and link to the document.
7. **Flag uncertainty** — if you can't verify a specification from an authoritative source, say so explicitly.

### Example Search Queries
```
"SX1276 datasheet site:semtech.com"
"ATmega2560 technical reference site:microchip.com"
"ESP32-S3 technical reference manual site:espressif.com"
"BME280 datasheet site:bosch-sensortec.com"
"STM32F446 reference manual RM0390"
"FE310-G002 manual site:sifive.com"
"PIC18F46K22 datasheet site:microchip.com"
```

## Common Peripheral Datasheets Quick Reference

These are frequently needed in PlatformIO projects:

### LoRa Radio Modules
- **SX1276/77/78/79** — Semtech, 137-1020 MHz, LoRa + FSK
- **SX1261/62/68** — Semtech, next-gen LoRa, lower power
- **RFM95/96/97/98** — HopeRF modules using SX127x (same registers)
- **Ra-01/Ra-02** — Ai-Thinker modules using SX127x

### USB-to-Serial Bridges
- **CP2102/CP2102N** — Silicon Labs, common on ESP32 DevKit
- **CH340/CH340G** — WCH, common on cheap Arduino clones
- **FT232RL/FT2232H** — FTDI, professional grade
- **CH9102** — WCH, newer, ESP32-S2/S3 boards

### Common Sensors
- **BME280** — Bosch, temperature + humidity + pressure (I2C/SPI)
- **MPU6050** — InvenSense, 6-axis IMU (I2C)
- **ADS1115** — TI, 16-bit ADC (I2C)
- **DS18B20** — Maxim/Analog Devices, temperature (1-Wire)
- **DHT22/AM2302** — Aosong, temperature + humidity (custom 1-wire)
- **INA219** — TI, current/power monitor (I2C)

### Display Controllers
- **SSD1306** — Solomon Systech, 128x64 OLED (I2C/SPI)
- **ILI9341** — Ilitek, 240x320 TFT LCD (SPI)
- **ST7789** — Sitronix, 240x240/320 TFT LCD (SPI)
