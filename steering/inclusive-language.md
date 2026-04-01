# Inclusive Language Guide for Embedded Development

The embedded and electronics industry has historically used terminology rooted in oppressive metaphors. When writing new code, documentation, comments, variable names, or configuration, use inclusive alternatives. This guide follows the [IETF Terminology, Power, and Oppressive Language](https://datatracker.ietf.org/doc/draft-knodel-terminology/) recommendations and the [Inclusive Naming Initiative](https://inclusivenaming.org/).

## Core Principle

**Functionality and correctness always come first.** Use inclusive terminology in code you author — variable names, function names, comments, documentation, and configuration you control. Never alter external URLs, library API names, register identifiers, branch names, protocol-level constants, or any other external interface to conform to inclusive naming. Breaking a link or a build to avoid a word is the wrong tradeoff.

When external libraries, protocol specs, or hardware APIs enforce legacy terminology, use their exact terms at the interface boundary. Wrap them with inclusive naming in your own code when practical.

## Terminology Replacements

### Master / Slave → Controller / Peripheral (or alternatives)

| Legacy Term | Preferred Alternative | Context |
|-------------|----------------------|---------|
| master | controller, primary, leader, coordinator | SPI bus, I2C bus, general architecture |
| slave | peripheral, secondary, follower, worker, target | SPI device, I2C device, general architecture |
| master/slave | controller/peripheral, primary/secondary, leader/follower | Bus topology, replication, architecture |

**In SPI context:** Use controller/peripheral. The [OSHWA resolution](https://www.oshwa.org/a-resolution-to-redefine-spi-signal-names/) formally redefined SPI signal names:
- MOSI → SDO (Serial Data Out)
- MISO → SDI (Serial Data In)
- SS → CS (Chip Select)

**In I2C context:** Use controller/target. The [I2C specification revision](https://www.nxp.com/docs/en/user-guide/UM10204.pdf) from NXP adopted controller/target terminology.

**In code:**
```cpp
// Preferred
void spi_controller_init(void);
uint8_t i2c_read_from_target(uint8_t address);
typedef enum { ROLE_CONTROLLER, ROLE_PERIPHERAL } bus_role_t;

// Avoid
// void spi_master_init(void);
// uint8_t i2c_read_from_slave(uint8_t address);
```

**Interface boundary example** — when an Arduino library uses legacy naming:
```cpp
// The Wire library uses legacy API names — we can't change that.
// But our wrapper uses inclusive terminology.
bool read_target_register(uint8_t target_addr, uint8_t reg, uint8_t *data) {
    Wire.beginTransmission(target_addr);  // Library API — legacy naming internally
    Wire.write(reg);
    Wire.endTransmission();
    Wire.requestFrom(target_addr, (uint8_t)1);
    if (Wire.available()) {
        *data = Wire.read();
        return true;
    }
    return false;
}
```

### Blacklist / Whitelist → Blocklist / Allowlist

| Legacy Term | Preferred Alternative |
|-------------|----------------------|
| blacklist | blocklist, denylist, reject list |
| whitelist | allowlist, permit list, accept list |

**In code:**
```cpp
// Preferred
const uint8_t allowed_addresses[] = { 0x3C, 0x68, 0x76 };
bool is_address_blocked(uint8_t addr);

// Avoid
// const uint8_t whitelisted_addresses[] = { ... };
// bool is_blacklisted(uint8_t addr);
```

**In configuration:**
```ini
# Preferred
; platformio.ini
build_flags = -D ALLOWED_BAUD_RATE=115200

# Avoid
; build_flags = -D WHITELISTED_BAUD_RATE=115200
```

### Male / Female → Plug / Socket (Connectors)

| Legacy Term | Preferred Alternative |
|-------------|----------------------|
| male connector | plug, pin connector, Type-A |
| female connector | socket, receptacle, Type-B |

**In documentation and comments:**
```cpp
// Preferred: "Connect the USB plug to the board's USB socket"
// Avoid: "Connect the male USB to the female USB port"
```

### Other Terms

| Legacy Term | Preferred Alternative | Context |
|-------------|----------------------|---------|
| sanity check | confidence check, coherence check, validation | Code review, testing |
| dummy | placeholder, stub, sample, mock | Test data, variables |
| crippled | limited, restricted, reduced | Feature descriptions |
| grandfathered | legacy, exempt, pre-existing | Policy, compatibility |
| native | built-in, default, host | Platform descriptions |
| abort | cancel, stop, terminate, halt | Process control |

## When You Must Use Legacy Terminology

Correctness and interoperability take absolute priority. These situations are not exceptions — they are the rule at interface boundaries:

1. **URLs and branch names:** If a repo's default branch is `master`, link to `master`. Don't guess or substitute `main` — a broken link helps nobody.
2. **Library APIs:** If `Wire.h` exposes `requestFrom()` or `SPI.begin()` uses `MSBFIRST`, use those exact identifiers. Wrap them in your own functions with inclusive naming if you want, but never rename the call itself.
3. **Hardware register names:** If a chip's register is `SPI_MASTER_CR`, use that exact name in register access code. The compiler only knows what it knows.
4. **Protocol specifications and datasheets:** When citing or implementing a spec that says "master," use its terminology for accuracy. In your own surrounding prose and variable names, use inclusive terms.
5. **Existing codebases:** When modifying legacy code, update terminology in the files you touch when practical and when it won't cause confusion. Don't refactor an entire codebase just for naming — do it incrementally.
6. **Interoperability with old devices:** Some devices expose legacy terminology in their command sets, status registers, or configuration interfaces. Use whatever the device requires to function correctly.

## Agent Behavior

When generating code, comments, documentation, or variable/function names:

1. **Functionality first, always.** Never change a URL, API call, register name, branch reference, or external identifier to be more inclusive. If it breaks, it's wrong — full stop.
2. **Use inclusive terminology by default** in new code and documentation that you control.
3. **When referencing external APIs** that use legacy terms, use the API's exact names at the call site. Use inclusive names for your own wrappers, variables, and comments.
4. **When explaining protocols** (SPI, I2C), use controller/peripheral/target terminology. Mention legacy names in parentheses on first reference for clarity, then use inclusive terms throughout.
5. **Don't silently rename** external API calls, register names, URLs, or branch references — that breaks things.
6. **In `platformio.ini` and build configuration**, use inclusive terms for custom defines, comments, and environment names. Never alter library or platform identifiers.

## References

- [IETF: Terminology, Power, and Oppressive Language](https://datatracker.ietf.org/doc/draft-knodel-terminology/)
- [Inclusive Naming Initiative](https://inclusivenaming.org/)
- [OSHWA: Resolution to Redefine SPI Signal Names](https://www.oshwa.org/a-resolution-to-redefine-spi-signal-names/)
- [Oppressive Language Repository](https://github.com/mbmccormick/oppressivelanguage)
