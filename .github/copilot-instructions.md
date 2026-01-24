# GitHub Copilot Instructions for rickballv4 ZMK Keyboard

## Overview

This file enables AI coding assistants to generate features aligned with the rickballv4 keyboard's architecture and style. It is based only on actual, observed patterns from the codebase — not invented practices.

The rickballv4 is a **split wireless mechanical keyboard** built on **ZMK Firmware** (Zephyr RTOS-based). It features:
- Split design (left/right halves)
- Wireless Bluetooth connectivity (nRF52840-based)
- Matrix scanning via SPI shift registers (74HC595)
- Rotary encoder support on both sides
- VIK (Versatile Input Keyboard) modular peripheral system
- ZMK Studio runtime configuration support
- RGB underglow hardware (disabled by default for battery life)

## Technology Stack

### Core Technologies
- **Devicetree (DTS/DTSI)**: Hardware description and configuration
- **Kconfig**: Build-time feature configuration
- **YAML**: Build matrix and project metadata
- **ZMK Firmware**: Keyboard firmware framework on Zephyr RTOS
- **West**: Meta-tool for multi-repository management

### Key Dependencies
- ZMK from rianadon fork (main branch)
- zmk-fingerpunch-vik for VIK module system
- vik-core module for VIK functionality
- Zephyr RTOS (via ZMK)

### Hardware
- **Board**: cosmos_lemon_wireless (nRF52840)
- **Matrix**: 7 rows × 14 columns (7×7 per side)
- **GPIO Expansion**: 74HC595 shift register via SPI
- **Encoders**: EC11 rotary encoders (left and right)
- **LEDs**: 49× WS2812B RGB LEDs via SPI
- **Communication**: Bluetooth split (right=central, left=peripheral)

## File Category Reference

### Build Configuration
**Purpose**: Define CI/CD build matrix and GitHub Actions workflow

**Files**:
- [build.yaml](build.yaml) - Build matrix with shield/board combinations
- [.github/workflows/build.yml](.github/workflows/build.yml) - GitHub Actions workflow

**Key Conventions**:
- Separate build entry for each shield variant (left, right, settings_reset)
- Right side includes full features (Studio, VIK modules)
- Left side minimal build
- Space-separated shields for multi-shield builds
- CMake overrides via `cmake-args` for build-specific features

### West Manifest
**Purpose**: Multi-repository dependency management

**Files**:
- [config/west.yml](config/west.yml) - Main manifest with ZMK reference
- [config/deps.yml](config/deps.yml) - Project-specific dependencies (VIK)

**Key Conventions**:
- Use rianadon fork for ZMK (not upstream)
- Pin to `main` branch
- Import chain: project → ZMK → VIK modules
- Self path in `config/` directory

### Zephyr Module
**Purpose**: Declare repository as Zephyr module

**Files**:
- [zephyr/module.yml](zephyr/module.yml)

**Key Conventions**:
- Minimal configuration: depends + board_root only
- Lists vik-core dependency
- `board_root: .` exposes boards/ directory

### Shield Metadata
**Purpose**: ZMK shield declaration and feature listing

**Files**:
- [boards/shields/rickballv4/rickballv4.zmk.yml](boards/shields/rickballv4/rickballv4.zmk.yml)

**Key Conventions**:
- Lists all features (even if disabled in config)
- Declares siblings (left/right variants)
- URL points to keyboard designer's repo

### Kconfig Files
**Purpose**: Shield detection and default configuration

**Files**:
- [boards/shields/rickballv4/Kconfig.shield](boards/shields/rickballv4/Kconfig.shield) - Shield detection
- [boards/shields/rickballv4/Kconfig.defconfig](boards/shields/rickballv4/Kconfig.defconfig) - Default config

**Key Conventions**:
- Use `def_bool $(shields_list_contains,...)` for shield detection
- Conditional blocks for per-side config
- Right side sets keyboard name and central role
- Both sides enable ZMK_SPLIT
- Comment endif with condition

### Devicetree Shield Includes
**Purpose**: Shared hardware description for both halves

**Files**:
- [boards/shields/rickballv4/rickballv4.dtsi](boards/shields/rickballv4/rickballv4.dtsi) - Main hardware definition
- [boards/shields/rickballv4/rickballv4-layouts.dtsi](boards/shields/rickballv4/rickballv4-layouts.dtsi) - Physical layouts

**Key Conventions**:
- Define VIK macros before SPI config (`VIK_SPI_REG_START`, `VIK_SPI_CS_PREFIX`)
- Shift register always at `reg = <0>` on SPI bus
- Matrix scanning: col2row diode direction, direct GPIO rows, shifter columns
- Encoders disabled in base (enabled per side in overlays)
- Chosen nodes: `zmk,kscan`, `zmk,physical-layout`
- GPIO hogging to disable ext_power for battery life

### Devicetree Shield Overlays
**Purpose**: Per-side (left/right) hardware configuration

**Files**:
- [boards/shields/rickballv4/rickballv4_left.overlay](boards/shields/rickballv4/rickballv4_left.overlay)
- [boards/shields/rickballv4/rickballv4_right.overlay](boards/shields/rickballv4/rickballv4_right.overlay)

**Key Conventions**:
- Include base .dtsi with quotes
- Boot magic key position: 0 (left), 43 (right)
- Enable respective encoder (left_encoder, right_encoder)
- Right side adds `col-offset = <7>` to matrix transform
- Minimal overrides only (don't redefine full nodes)

### Devicetree Board Overlays
**Purpose**: Board-specific hardware (LEDs, VIK connector, pinctrl)

**Files**:
- [boards/shields/rickballv4/boards/cosmos_lemon_wireless.overlay](boards/shields/rickballv4/boards/cosmos_lemon_wireless.overlay)

**Key Conventions**:
- Pinctrl with default and sleep states
- VIK connector gpio-map with inline comments
- VIK aliases: `vik_i2c: &i2c0`, `vik_spi: &spi1`
- WS2812 on separate SPI bus (spi3)
- LED strip timing: 4MHz, 0xE0=1, 0x80=0, GRB color order
- Chosen node for zmk,underglow

### Keymap Files
**Purpose**: Key layout, layers, behaviors, and macros

**Files**:
- [boards/shields/rickballv4/rickballv4.keymap](boards/shields/rickballv4/rickballv4.keymap)

**Key Conventions**:
- Include order: behaviors, bindings, mouse support
- Custom behavior naming: snake_case label, UPPERCASE label property
- ASCII art comments for visual layout
- Hold-tap timing: tapping-term, quick-tap, require-prior-idle
- Sensor-bindings on one line (left encoder, right encoder)
- Layer naming: default_layer, extra1, extra2, etc.
- Transparent keys with `&trans`

### Shield Configuration
**Purpose**: Compile-time feature configuration

**Files**:
- [boards/shields/rickballv4/rickballv4_left.conf](boards/shields/rickballv4/rickballv4_left.conf)
- [boards/shields/rickballv4/rickballv4_right.conf](boards/shields/rickballv4/rickballv4_right.conf)

**Key Conventions**:
- Comment grouping for related features
- Explicit disabling with `=n` (don't rely on defaults)
- Required: `CONFIG_SPI=y`, `CONFIG_EC11=y`, `CONFIG_ZMK_POINTING=y`
- Power savings: RGB and ext_power disabled
- Studio locking disabled

### Physical Layout Definition
**Purpose**: ZMK Studio visual layout representation

**Files**:
- [boards/shields/rickballv4/rickballv4-layouts.dtsi](boards/shields/rickballv4/rickballv4-layouts.dtsi)

**Key Conventions**:
- Coordinates in hundredths (100 = 1u)
- Rotation in tenths of degrees
- Leading equals, leading commas, trailing semicolon
- Links to matrix transform
- Rotation point (rx, ry) for rotated thumb keys

## Feature Scaffold Guide

### Adding a New Layer
1. Add layer to keymap file after existing layers:
   ```dts
   new_layer {
       bindings = <
           // ASCII art layout comment
           &kp ...  // Key bindings
       >;
       sensor-bindings = <&inc_dec_kp ... &inc_dec_kp ...>;
   };
   ```
2. Reference layer in other layers with `&mo N` or `&lt N KEY`

### Adding a Custom Behavior
1. Define behavior in behaviors block:
   ```dts
   my_behavior: my_custom {
       compatible = "zmk,behavior-TYPE";
       label = "MY_CUSTOM";
       #binding-cells = <N>;
       // Behavior-specific properties
   };
   ```
2. Use in keymap: `&my_behavior` with required parameters

### Adding VIK Module Support
1. Add module to right side build.yaml:
   ```yaml
   shield: rickballv4_right vik_MODULE_NAME
   ```
2. Increment `VIK_SPI_REG_START` in rickballv4.dtsi if module uses SPI
3. Module configuration comes from VIK library (no local changes needed)

### Enabling RGB Underglow
1. Change .conf files:
   ```properties
   CONFIG_ZMK_RGB_UNDERGLOW=y
   CONFIG_WS2812_STRIP=y
   CONFIG_ZMK_EXT_POWER=y
   ```
2. Disable or remove ext_power_hog in rickballv4.dtsi
3. Add RGB keybindings to keymap:
   ```dts
   #include <dt-bindings/zmk/rgb.h>
   // In layer: &rgb_ug RGB_TOG
   ```

### Modifying Matrix Layout
1. Update row-gpios/col-gpios in rickballv4.dtsi
2. Update matrix transform map
3. Update physical layout coordinates
4. Keep all three synchronized (same key count and order)

### Adding Encoder Functions
1. Modify sensor-bindings in keymap layers:
   ```dts
   sensor-bindings = <&inc_dec_kp KEY1 KEY2 &inc_dec_kp KEY3 KEY4>;
   ```
2. Can use any behavior, not just key presses
3. One binding per encoder (left, right)

## Integration Rules

### Hardware Abstraction Domain
- ✅ **REQUIRED**: All hardware must be described in devicetree
- ✅ **REQUIRED**: Matrix scanning via `zmk,kscan-gpio-matrix`
- ✅ **REQUIRED**: Physical layouts for Studio support
- ⚠️ **CONSTRAINT**: No runtime hardware detection
- ⚠️ **CONSTRAINT**: SPI chip selects managed via `VIK_SPI_CS_PREFIX`

### Input Processing Domain
- ✅ **REQUIRED**: All input via ZMK behavior system
- ✅ **REQUIRED**: Behaviors define `#binding-cells`
- ⚠️ **CONSTRAINT**: No raw keycodes, only behavior invocations
- ⚠️ **CONSTRAINT**: Hold-tap must specify timing parameters
- ⚠️ **CONSTRAINT**: Macros use `macro-one-param` with placeholders

### Split Keyboard Communication Domain
- ✅ **REQUIRED**: Right side is central role
- ✅ **REQUIRED**: Separate .overlay files per side
- ✅ **REQUIRED**: Right side adds col-offset to transform
- ⚠️ **CONSTRAINT**: Single shared keymap for both halves
- ⚠️ **CONSTRAINT**: Keyboard name only on central (right)

### Peripheral Integration Domain
- ✅ **REQUIRED**: VIK connector with gpio-map
- ✅ **REQUIRED**: VIK_SPI_REG_START and VIK_SPI_CS_PREFIX macros
- ⚠️ **CONSTRAINT**: Shift register must be reg=0 on SPI bus
- ⚠️ **CONSTRAINT**: VIK modules added as secondary shields

### Build System Domain
- ✅ **REQUIRED**: West workspace with config/ as self path
- ✅ **REQUIRED**: Separate build entries for left and right
- ⚠️ **CONSTRAINT**: Use rianadon ZMK fork
- ⚠️ **CONSTRAINT**: Studio only on central (right) side

### Configuration Management Domain
- ✅ **REQUIRED**: Per-side .conf files
- ✅ **REQUIRED**: Explicit feature disabling with =n
- ⚠️ **CONSTRAINT**: Power-hungry features disabled by default
- ⚠️ **CONSTRAINT**: Split configuration via Kconfig conditionals

### LED Underglow Domain
- ✅ **REQUIRED**: WS2812 via SPI with specific timing
- ✅ **REQUIRED**: Separate SPI bus for LEDs (not vik_spi)
- ⚠️ **CONSTRAINT**: Disabled by default for battery life
- ⚠️ **CONSTRAINT**: Must remove ext_power_hog to enable

### ZMK Studio Support Domain
- ✅ **REQUIRED**: Physical layout with key attributes
- ✅ **REQUIRED**: Coordinates in hundredths, rotation in tenths
- ⚠️ **CONSTRAINT**: Studio only on central, enabled via build.yaml
- ⚠️ **CONSTRAINT**: Requires studio-rpc-usb-uart snippet

## Example Prompt Usage

### Example 1: Add a Symbol Layer
**Prompt**: "Add a symbol layer with common programming symbols accessible via layer tap on space"

**Expected Output**:
1. Add new layer in [boards/shields/rickballv4/rickballv4.keymap](boards/shields/rickballv4/rickballv4.keymap):
   ```dts
   symbols_layer {
       bindings = <
           // ╭─────────────────────────────────────────────────╮ ╭─────────────────────────────────────────────────╮
                &kp EXCL  &kp AT    &kp HASH  &kp DLLR  &kp PRCNT &trans        &kp CARET &kp AMPS  &kp ASTRK &kp LPAR  &kp RPAR  &trans
           // ├─────────────────────────────────────────────────┤ ├─────────────────────────────────────────────────┤
                &trans    &kp LBRC  &kp RBRC  &kp LPAR  &kp RPAR  &trans        &kp MINUS &kp EQUAL &kp LBKT  &kp RBKT  &kp BSLH  &trans
           // ... more rows
       >;
       sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp C_VOL_UP C_VOL_DN>;
   };
   ```
2. Update default_layer space key to layer-tap:
   ```dts
   &lt 1 SPACE  // Assuming symbols_layer is layer 1
   ```

### Example 2: Enable Mouse Emulation
**Prompt**: "Configure mouse scroll wheel emulation on encoders"

**Expected Output**:
Update sensor-bindings in a layer of [boards/shields/rickballv4/rickballv4.keymap](boards/shields/rickballv4/rickballv4.keymap):
```dts
sensor-bindings = <&inc_dec_kp MWHL_UP MWHL_DOWN &inc_dec_kp MWHL_UP MWHL_DOWN>;
```
(Note: Already includes mouse headers in keymap file)

### Example 3: Add Bluetooth Profile Switching
**Prompt**: "Add a layer for Bluetooth profile management with clear all on bottom-left"

**Expected Output**:
Add layer to [boards/shields/rickballv4/rickballv4.keymap](boards/shields/rickballv4/rickballv4.keymap):
```dts
bluetooth_layer {
    bindings = <
        // ╭─────────────────────────────────────────────────╮ ╭─────────────────────────────────────────────────╮
             &bt BT_SEL 0 &bt BT_SEL 1 &bt BT_SEL 2 &bt BT_SEL 3 &bt BT_SEL 4 &trans        &trans &trans &trans &trans &trans &trans
        // ├─────────────────────────────────────────────────┤ ├─────────────────────────────────────────────────┤
             &bt BT_CLR   &trans       &trans       &trans       &trans       &trans        &trans &trans &trans &trans &trans &trans
        // ... more rows
    >;
};
```

### Example 4: Add Trackball Module
**Prompt**: "Add support for the PMW3610 trackball module"

**Expected Output**:
Update [build.yaml](build.yaml) right side entry:
```yaml
- board: cosmos_lemon_wireless
  shield: rickballv4_right vik_pmw3610
  snippet: studio-rpc-usb-uart
  cmake-args: -DCONFIG_ZMK_STUDIO=y
```
(Note: No other changes needed; VIK module library provides configuration)

## Common Pitfalls

### ❌ Don't: Modify Left and Right Configs Separately
Unless the feature is truly side-specific, keep .conf files identical. Most features apply to both halves.

### ❌ Don't: Forget Col-Offset on Right Side
Right side must have `col-offset = <7>` in overlay, or key positions will be wrong.

### ❌ Don't: Mix Up Matrix Transform and Physical Layout
These are different:
- Matrix transform: electrical scanning (RC coordinates)
- Physical layout: visual appearance (X/Y coordinates)
Keep them synchronized but don't confuse them.

### ❌ Don't: Enable RGB Without Removing Power Hog
Enabling `CONFIG_ZMK_RGB_UNDERGLOW=y` won't work if ext_power_hog is still forcing power off.

### ❌ Don't: Add VIK Modules to Left Side
VIK peripherals go on central (right) side only in build.yaml.

### ❌ Don't: Forget to Include Required Headers
Keymaps need behavior headers, Kconfig needs endif comments, devicetree needs compatible strings.

## Quick Reference

### Common Devicetree Compatible Strings
- `zmk,kscan-gpio-matrix` - Matrix scanner
- `zmk,gpio-595` - 74HC595 shift register
- `zmk,behavior-hold-tap` - Hold-tap behavior
- `zmk,behavior-mod-morph` - Modifier-morphing key
- `zmk,behavior-tap-dance` - Multi-tap sequences
- `zmk,physical-layout` - Studio layout
- `zmk,matrix-transform` - Matrix mapping
- `alps,ec11` - Rotary encoder
- `worldsemi,ws2812-spi` - RGB LEDs
- `sadekbaroudi,vik-connector` - VIK connector

### Common Kconfig Options
- `CONFIG_SPI` - SPI bus support
- `CONFIG_ZMK_SPLIT` - Split keyboard
- `CONFIG_ZMK_SPLIT_ROLE_CENTRAL` - Central role (right)
- `CONFIG_EC11` - Encoder support
- `CONFIG_ZMK_POINTING` - Mouse/pointing devices
- `CONFIG_ZMK_RGB_UNDERGLOW` - RGB lighting
- `CONFIG_ZMK_STUDIO` - Studio support
- `CONFIG_ZMK_EXT_POWER` - External power rail

### File Creation Order for New Shield
1. `boards/shields/NAME/Kconfig.shield` - Shield detection
2. `boards/shields/NAME/NAME.zmk.yml` - Metadata
3. `boards/shields/NAME/NAME.dtsi` - Base hardware
4. `boards/shields/NAME/NAME_left.overlay` - Left config
5. `boards/shields/NAME/NAME_right.overlay` - Right config
6. `boards/shields/NAME/NAME_left.conf` - Left features
7. `boards/shields/NAME/NAME_right.conf` - Right features
8. `boards/shields/NAME/NAME.keymap` - Keymap
9. `boards/shields/NAME/NAME-layouts.dtsi` - Physical layout
10. `boards/shields/NAME/Kconfig.defconfig` - Defaults
11. `boards/shields/NAME/boards/BOARD.overlay` - Board-specific
12. `build.yaml` - Build matrix entries
