# Embedded Coding Standards

Idiomatic conventions and style guidelines for C, C++, Assembly, Rust, and Python in embedded contexts. Load this when writing or reviewing firmware code.

## C (Embedded C — C99/C11)

The lingua franca of embedded development. Used across all platforms.

### Style Conventions
- **Naming:**
  - Functions: `snake_case` — `uart_init()`, `spi_transfer_byte()`
  - Types/structs: `snake_case_t` — `sensor_data_t`, `gpio_config_t`
  - Constants/macros: `UPPER_SNAKE_CASE` — `MAX_BUFFER_SIZE`, `LED_PIN`
  - Global variables: prefix with `g_` — `g_system_state`
  - Static file-scope variables: prefix with `s_` — `s_rx_buffer`
  - Enum values: `UPPER_SNAKE_CASE` with type prefix — `GPIO_MODE_INPUT`, `GPIO_MODE_OUTPUT`
- **Indentation:** 4 spaces (no tabs). Some projects use 2 spaces — match existing style.
- **Braces:** Allman or K&R — be consistent within a project. K&R is more common in embedded.
- **Line length:** 80-100 characters max.
- **File organization:** One module per `.c`/`.h` pair. Header guards or `#pragma once`.

### Embedded-Specific Practices
```c
// Use fixed-width integers (stdint.h) — ALWAYS
#include <stdint.h>
uint8_t  register_value;    // NOT: unsigned char
int16_t  temperature;       // NOT: int (size varies by platform!)
uint32_t timestamp_ms;      // NOT: unsigned long

// Use volatile for hardware registers and ISR-shared variables
volatile uint8_t g_isr_flag = 0;

// Use const for read-only data (goes to flash, saves RAM)
static const uint8_t lookup_table[] = { 0x00, 0x01, 0x03, 0x07 };

// Use static for file-scope functions and variables (encapsulation)
static void process_rx_buffer(void);
static uint8_t s_rx_buffer[64];

// Bit manipulation — use named macros, not magic numbers
#define STATUS_READY  (1U << 0)
#define STATUS_ERROR  (1U << 1)
#define STATUS_BUSY   (1U << 2)

// Register access patterns
#define REG_SET_BIT(reg, bit)   ((reg) |= (bit))
#define REG_CLR_BIT(reg, bit)   ((reg) &= ~(bit))
#define REG_TOG_BIT(reg, bit)   ((reg) ^= (bit))
#define REG_CHK_BIT(reg, bit)   ((reg) & (bit))
```

### ISR (Interrupt Service Routine) Rules
- Keep ISRs as short as possible. Set a flag, copy data to a buffer, return.
- Never call blocking functions (delay, printf, malloc) from an ISR.
- Variables shared between ISR and main code must be `volatile`.
- Use atomic operations or disable interrupts briefly when accessing multi-byte shared variables.
- On AVR: use `ISR(VECTOR_NAME)` macro. On ARM: standard function with correct vector table entry.

### Memory Management
- **Avoid `malloc`/`free`** in production embedded code. Heap fragmentation is fatal on systems without an MMU.
- Use static allocation, memory pools, or ring buffers.
- Stack size is limited (often 1-4 KB on AVR, 4-8 KB on ARM). Avoid large local arrays.
- Use `sizeof()` instead of hardcoded sizes.

### Header File Template
```c
#ifndef MODULE_NAME_H
#define MODULE_NAME_H

#include <stdint.h>
#include <stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif

// Public types
typedef struct {
    uint16_t value;
    uint8_t  status;
} sensor_reading_t;

// Public function declarations
void module_init(void);
bool module_read(sensor_reading_t *reading);

#ifdef __cplusplus
}
#endif

#endif // MODULE_NAME_H
```

## C++ (Embedded C++ — C++11/14/17)

Used with Arduino framework, ESP-IDF (partially), and Mbed OS.

### Style Conventions
- **Naming:**
  - Classes: `PascalCase` — `TemperatureSensor`, `LoRaRadio`
  - Methods: `camelCase` — `readTemperature()`, `sendPacket()`
  - Member variables: `m_` prefix or trailing `_` — `m_baudRate` or `baudRate_`
  - Constants: `k` prefix + PascalCase — `kMaxRetries`, `kDefaultBaud`
  - Namespaces: `lowercase` — `sensors`, `comms`
  - Enums (scoped): `PascalCase` — `enum class Mode { Input, Output, Analog }`
- **Use `enum class`** over plain `enum` for type safety.
- **Use `constexpr`** over `#define` for compile-time constants.
- **Use `nullptr`** over `NULL` or `0`.

### Embedded-Specific C++ Practices
```cpp
// RAII for hardware resources
class SpiDevice {
public:
    explicit SpiDevice(uint8_t cs_pin) : m_cs_pin(cs_pin) {
        pinMode(m_cs_pin, OUTPUT);
        digitalWrite(m_cs_pin, HIGH);
    }

    void beginTransaction() {
        SPI.beginTransaction(m_settings);
        digitalWrite(m_cs_pin, LOW);
    }

    void endTransaction() {
        digitalWrite(m_cs_pin, HIGH);
        SPI.endTransaction();
    }

    // Transfer with automatic CS management
    uint8_t transfer(uint8_t data) {
        beginTransaction();
        uint8_t result = SPI.transfer(data);
        endTransaction();
        return result;
    }

private:
    uint8_t m_cs_pin;
    SPISettings m_settings{1000000, MSBFIRST, SPI_MODE0};
};

// Use constexpr for compile-time computation
constexpr uint32_t kBaudRate = 115200;
constexpr size_t kBufferSize = 256;
constexpr uint8_t kMaxSensors = 8;

// Templates for type-safe register access (zero-cost abstraction)
template<typename T>
inline void writeRegister(volatile T* reg, T value) {
    *reg = value;
}
```

### What to Avoid in Embedded C++
- **Exceptions** (`throw`/`catch`): Disabled by default on most embedded toolchains. Use error codes or `std::optional`.
- **RTTI** (`dynamic_cast`, `typeid`): Adds code size. Disable with `-fno-rtti`.
- **STL containers** (`std::vector`, `std::map`): Use heap allocation. Prefer fixed-size arrays, `std::array`, or custom containers.
- **`std::string`**: Heap-allocated. Use `char[]` buffers or `String` (Arduino) sparingly.
- **Virtual functions**: Acceptable but add vtable overhead. Use sparingly on very constrained MCUs (AVR).
- **`new`/`delete`**: Same concerns as `malloc`/`free`. Use placement new or static allocation.

### Arduino Framework Conventions
- `setup()` and `loop()` are the entry points. Keep `loop()` non-blocking.
- Use `millis()` for timing, not `delay()`. Delay blocks everything.
- Prefer `digitalWriteFast()` (if available) over `digitalWrite()` for time-critical GPIO.
- Use `F()` macro for string literals on AVR to keep them in flash: `Serial.println(F("Hello"))`.

## Assembly (ARM Thumb-2, AVR, RISC-V)

Used for startup code, critical timing, and hardware-specific operations.

### General Conventions
- **Comment every instruction** or logical block. Assembly is write-once, read-never without comments.
- **Use meaningful labels:** `uart_tx_byte:`, `delay_loop:`, not `L1:`, `L2:`.
- **Document register usage** at the top of each function:
  ```asm
  ; uart_send_byte - Send one byte via UART
  ; Input:  r0 = byte to send
  ; Output: none
  ; Clobbers: r1, r2
  ```
- **Preserve callee-saved registers.** Push/pop per the platform ABI.
- **Use `.section` directives** to place code and data correctly (`.text`, `.data`, `.bss`, `.rodata`).

### ARM Thumb-2 (STM32, SAMD, nRF52)
```asm
    .syntax unified
    .thumb
    .global my_function

    .section .text
my_function:
    push    {r4-r7, lr}     @ Save callee-saved registers
    @ ... function body ...
    pop     {r4-r7, pc}     @ Restore and return
```

### AVR Assembly
```asm
; Inline ASM in C (GCC AVR)
; Disable interrupts atomically
cli                         ; Clear global interrupt flag
; ... critical section ...
sei                         ; Set global interrupt flag
```

### RISC-V Assembly
```asm
    .global _start
    .section .text

_start:
    la      sp, _stack_top  # Load stack pointer
    call    main            # Call C main
    j       .               # Infinite loop if main returns
```

### When to Use Assembly
- Startup code and vector tables (often provided by the framework)
- Precise timing loops (bit-banging protocols)
- DSP inner loops where compiler output is insufficient
- Accessing special CPU registers (interrupt control, cache management)
- **In all other cases, prefer C/C++.** Modern compilers optimize better than most hand-written assembly.

## Rust (Embedded Rust — no_std)

Growing ecosystem for embedded. Strong safety guarantees.

### Style Conventions (follows standard Rust conventions)
- **Naming:**
  - Functions/methods: `snake_case` — `read_sensor()`, `init_uart()`
  - Types/traits: `PascalCase` — `SensorReading`, `UartConfig`
  - Constants: `UPPER_SNAKE_CASE` — `MAX_BUFFER_SIZE`
  - Modules: `snake_case` — `mod serial_comms;`
  - Crate names: `kebab-case` in Cargo.toml, `snake_case` in code
- **Use `#![no_std]` and `#![no_main]`** for bare-metal targets.
- **Use PAC (Peripheral Access Crate)** or HAL crate for your MCU. Don't write raw register access.

### Embedded Rust Patterns
```rust
#![no_std]
#![no_main]

use panic_halt as _;  // Panic handler
use cortex_m_rt::entry;
use stm32f4xx_hal::{pac, prelude::*};

#[entry]
fn main() -> ! {
    let dp = pac::Peripherals::take().unwrap();
    let rcc = dp.RCC.constrain();
    let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();

    let gpioa = dp.GPIOA.split();
    let mut led = gpioa.pa5.into_push_pull_output();

    loop {
        led.toggle();
        cortex_m::asm::delay(8_000_000);
    }
}
```

### Key Crates for Embedded Rust
- `cortex-m`, `cortex-m-rt` — ARM Cortex-M runtime
- `embedded-hal` — Hardware abstraction traits
- `defmt` — Efficient logging for embedded (replaces `println!`)
- `heapless` — Fixed-capacity collections (no allocator needed)
- `nb` — Non-blocking abstraction
- `riscv`, `riscv-rt` — RISC-V runtime

### PlatformIO + Rust
PlatformIO has experimental Rust support. For production Rust embedded, consider using `cargo` with `probe-rs` or `cargo-embed` directly, and use PlatformIO for the C/C++ portions of mixed projects.

## Python (MicroPython / CircuitPython)

Interpreted Python for microcontrollers. Rapid prototyping, education, and scripting.

### Style Conventions (PEP 8 adapted for embedded)
- **Naming:** Standard PEP 8 — `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
- **Imports:** Group stdlib, third-party, then local. On MicroPython, minimize imports to save RAM.
- **Docstrings:** Use them for public functions, but keep brief — memory is limited.

### MicroPython Best Practices
```python
import machine
import time

# Use const() for integer constants (optimized by MicroPython compiler)
_LED_PIN = const(2)
_BAUD_RATE = const(115200)

# Pre-allocate buffers
_rx_buffer = bytearray(256)

# Use machine module for hardware access
led = machine.Pin(_LED_PIN, machine.Pin.OUT)
uart = machine.UART(1, baudrate=_BAUD_RATE)

# Prefer memoryview for zero-copy buffer slicing
mv = memoryview(_rx_buffer)

# Use micropython.schedule() for ISR-safe deferred calls
import micropython

def _isr_handler(pin):
    micropython.schedule(_process_interrupt, pin)

def _process_interrupt(pin):
    # Heavy processing here, outside ISR context
    pass
```

### Memory Optimization
- Use `gc.collect()` periodically in long-running loops.
- Use `micropython.mem_info()` to monitor heap usage.
- Freeze modules into firmware for RAM savings.
- Use `const()` for integer constants.
- Avoid string concatenation in loops — use `bytes` or `bytearray`.

## Cross-Language Best Practices for Embedded

1. **Fixed-width integers everywhere.** `int` is 16 bits on AVR, 32 bits on ARM. Use `uint8_t`, `int32_t`, etc.
2. **Document units in variable names or comments:** `timeout_ms`, `temperature_celsius_x100`, `voltage_mv`.
3. **Defensive programming:** Check return values, validate inputs, handle error cases. Silent failures are the norm in embedded — nothing throws an exception when SPI returns garbage.
4. **State machines** are the preferred architecture for embedded main loops. Avoid deep call stacks and complex control flow.
5. **Minimize global mutable state.** Pass data through function parameters or well-defined module interfaces.
6. **Power awareness:** Every line of code has a power cost. Idle loops should use sleep modes. Peripherals should be disabled when not in use.
7. **Reproducible builds:** Pin all dependency versions in `platformio.ini`. Use `lib_deps` with version specifiers (`@^1.2.0`).
