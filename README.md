# PS/2 USB Dual-Mode Keyboard

A QMK-based keyboard firmware that supports both USB and PS/2 protocols. Switch between modes with a hardware toggle!

[![License](https://img.shields.io/badge/license-GPL--2.0-blue.svg)](https://opensource.org/licenses/GPL-2.0) [![QMK](https://img.shields.io/badge/QMK-firmware-blue.svg)](https://qmk.fm/) [![Platform](https://img.shields.io/badge/platform-RP2040-green.svg)](https://www.raspberrypi.com/products/rp2040/) [![Version](https://img.shields.io/badge/version-2.0-brightgreen.svg)](https://github.com/BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard/releases)

## 🎉 What's New in v2.0

**Complete Ground-Up Rewrite!**

Version 1.x was a proof-of-concept that "worked" but used hacky workarounds. Version 2.0 is a **complete architectural rewrite** using proper QMK conventions.

**Major Changes:**

- ✅ **Proper host driver implementation** - Uses QMK's `host_driver_t` system instead of manual hacks
- ✅ **Better code organization** - Lookup tables instead of giant switch statement, separate files for different concerns
- ✅ **Full modifier support** - Shift, Ctrl, Alt, GUI now work correctly
- ✅ **Special key support** - Print Screen and Pause/Break with proper multi-byte sequences
- ✅ **Rock-solid reliability** - Proper driver switching eliminates flaky behavior
- ✅ **NO_USB_STARTUP_CHECK** - Works without USB connection
- ✅ **Comprehensive debug output** - Full uprintf logging with hex codes
- ✅ **Clean, maintainable code** - Follows QMK best practices

See CHANGELOG.md for complete details.

## Features

- 🔌 **Dual Protocol Support**: USB HID and PS/2 device modes
- 🔄 **Hardware Mode Switch**: Toggle between USB and PS/2 with a physical switch
- ⌨️ **Complete PS/2 Implementation**:
    - Full Scan Code Set 2 support (all standard keys)
    - Extended scancodes (F13-F24, multimedia, browser controls, power management)
    - International keyboard support (Japanese, Korean layouts)
    - Make/Break scan codes with automatic E0 prefix handling
    - Special keys (Print Screen, Pause/Break) with complex multi-byte sequences
    - Typematic repeat (auto-repeat when key held)
    - Proper timing and idle state handling
- 🎮 **QMK Powered**: Built on QMK framework, adaptable to any QMK-compatible microcontroller
- 📦 **Portable Code**: Uses QMK's GPIO abstraction layer for easy porting
- 🔧 **Robust & Tested**: Extensively tested with real PS/2 hosts

## Hardware Requirements

- **Microcontroller**: Any QMK-compatible board (currently configured for RP2040, but adaptable to AVR, STM32, etc.)
- **PS/2 Connector**: 6-pin Mini-DIN female connector
- **Mode Switch**: SPST switch or jumper
- **Button**: Single momentary switch (demo uses 'A' key)
- **Level Shifter** (recommended): Bidirectional logic level converter or transistor-based shifter

### ⚠️ Important: Voltage Level Considerations

**Different microcontrollers operate at different voltage levels, while PS/2 protocol typically operates at 5V.**

- **3.3V MCUs** (RP2040, STM32, ESP32): Need level shifting or rely on PS/2 host accepting 3.3V
- **5V MCUs** (AVR like ATmega32U4): Can connect directly to PS/2 without level shifters
- **PS/2 Standard**: 5V logic levels

**Options:**

1. **Direct Connection (for 5V microcontrollers)**
    
    - AVR-based boards (Pro Micro, Arduino Leonardo, etc.) can connect directly
    - No level shifter needed
    - Simplest solution
2. **Direct Connection (for 3.3V microcontrollers)** - Works with many modern PS/2 hosts
    
    - 3.3V output is often sufficient for PS/2 receivers
    - Many MCUs have 5V tolerant inputs when configured with pull-ups
    - This is what the demo uses with RP2040, and it works reliably in most cases
    - ⚠️ Some older PS/2 hosts may not reliably detect 3.3V logic high
3. **Bidirectional Level Shifter** (recommended for 3.3V MCUs in production)
    
    - Use a 3.3V ↔ 5V bidirectional level shifter (e.g., TXS0102, BSS138-based)
    - Ensures proper voltage levels in both directions
    - Necessary for guaranteed compatibility with all PS/2 hosts
    - Protects MCU from potential 5V signals on misconfigured hosts
4. **Simple Transistor Level Shifter**
    
    - 2x N-channel MOSFETs (BSS138 or similar) for clock and data
    - 4x 10kΩ pull-up resistors
    - Cost-effective DIY solution

**For this demo project**, direct connection works fine with modern computers. For production keyboards intended for wide compatibility, use a proper level shifter with 3.3V microcontrollers, or choose a 5V microcontroller.

### Pin Configuration

|Function|Default Pin|Description|
|---|---|---|
|Button|GP15|Single key input (direct pin)|
|PS/2 Clock|GP16|PS/2 clock line (bidirectional)|
|PS/2 Data|GP17|PS/2 data line (bidirectional)|
|Mode Switch|GP14|HIGH = USB mode, LOW = PS/2 mode|
|PS/2 Mouse Clk|GP18|Reserved for future mouse support|
|PS/2 Mouse Data|GP19|Reserved for future mouse support|

**Note**: Pin assignments are configured in `config.h` and `info.json` and can be changed for different microcontrollers. The current configuration uses RP2040 GPIO naming (GPxx), but the same pins can be adapted to other MCU naming schemes (e.g., PD2, PB3 for AVR).

### PS/2 Connector Pinout

![PS/2-Pinout](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse1.mm.bing.net%2Fth%2Fid%2FOIP.26t6hR8LtDg6UF4Z_CXtUwHaFA%3Fpid%3DApi%26ucfimg%3D1&f=1&ipt=2a8778ad5c083ed91012ae51ea8c5f98157f2c1856d839ca5e63038d2b3a54288&ipo=images)

**Wiring:**

- Pin 1 (Data) → GP17
- Pin 3 (GND) → GND
- Pin 4 (VCC) → 5V (VBUS)
- Pin 5 (Clock) → GP16

## Software Setup

### Prerequisites

1. [QMK Firmware](https://docs.qmk.fm/#/newbs_getting_started) installed
2. QMK CLI configured
3. ARM GCC toolchain

### Installation

#### Quick Install (Recommended)

1. **Clone this repository into your QMK keyboards directory:**

```bash
cd ~/qmk_firmware/keyboards
mkdir -p bjl  # Create the bjl directory first
cd bjl
git clone https://github.com/BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard.git ps2demo
```

#### Sparse Checkout (Minimal Files)

Or if you want to keep your QMK directory clean, use **sparse checkout** to only get the keyboard files:

```bash
cd ~/qmk_firmware/keyboards
mkdir -p bjl/ps2demo  # Create both directories
cd bjl/ps2demo
git init
git remote add origin https://github.com/BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard.git
git config core.sparseCheckout true
echo "ps2demo/*" >> .git/info/sparse-checkout
git pull origin main
```

This will only download the firmware files (`.c`, `.h`, `.json`, `.mk`) and skip documentation/extras.

### Compilation and Flashing

2. **Compile the firmware:**

```bash
qmk compile -kb bjl/ps2demo -km default
```

3. **Flash to your RP2040:**

```bash
# Put your board into bootloader mode (double-tap reset or hold BOOTSEL while plugging in)
qmk flash -kb bjl/ps2demo -km default
```

Alternatively, manually copy the `.uf2` file from `.build/` to the RPI-RP2 drive that appears.

**Pre-built firmware**: Check the [Releases](https://github.com/BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard/releases) page for pre-compiled `.uf2` files.

## Usage

### USB Mode

1. Set the mode switch to HIGH (or disconnect from GND)
2. Connect via USB
3. The keyboard will enumerate as a standard USB HID device
4. Press the button to send 'A' key

### PS/2 Mode

1. Set the mode switch to LOW (connect to GND)
2. Connect the PS/2 cable to your computer
3. Power the RP2040 via USB (for power only) or external 5V source
4. Press the button to send PS/2 scan codes:
    - Make code: `0x1C` (key down)
    - Break code: `0xF0 0x1C` (key up)
5. Hold the button for typematic repeat (auto-repeat after 500ms)

### Mode Switching

You can switch modes on-the-fly:

1. Toggle the mode switch
2. The firmware detects the change after 50ms debounce
3. Mode transition happens automatically:
    - **PS/2 → USB**: Disables typematic, restores USB driver, clears keyboard state
    - **USB → PS/2**: Saves USB driver, activates PS/2 driver, initializes PS/2 protocol
4. Debug output shows the transition (if console is enabled)

## Project Structure

```
bjl/ps2demo/
├── keymaps/
│   └── default/
│       └── keymap.c       # Keymap definition (single 'A' key)
├── config.h               # Pin definitions and configuration
├── info.json              # QMK keyboard metadata and USB IDs
├── kb.c                   # Main keyboard logic and mode switching (~140 lines)
├── kb.h                   # Keyboard header and layout definitions
├── ps2_keyboard.c         # PS/2 protocol implementation (~640 lines)
├── ps2_keyboard.h         # PS/2 protocol header (~60 lines)
├── ps2_scancodes.h        # Lookup tables and scancode definitions (~370 lines)
├── ps2_mouse.c            # PS/2 mouse (placeholder for future)
├── ps2_mouse.h            # PS/2 mouse header (placeholder)
└─── rules.mk              # Build configuration

```

**Total Core Code**: ~1,210 lines (well-organized and maintainable)

## PS/2 Protocol Implementation

This firmware implements PS/2 Scan Code Set 2 as a **device** (keyboard), including:

### Transmission Format

Each byte sent over PS/2 consists of 11 bits:

```
[Start] [D0] [D1] [D2] [D3] [D4] [D5] [D6] [D7] [Parity] [Stop]
   0     LSB                              MSB    Odd      1
```

### Timing Characteristics (v2.0 Improved)

- Clock frequency: ~10 kHz (50μs half-period)
- Inter-byte delay: 2ms (prevents receiver overload)
- Idle state: Both clock and data HIGH with 4x period stabilization
- Debounce: 50ms for mode switch

### Key Features

- **Make Codes**: Sent when key is pressed
- **Break Codes**: Two-byte sequence (`0xF0` + scan code) sent when key is released
- **E0 Extended Codes**: Automatic handling for navigation, arrows, multimedia keys
- **Complete Key Support**:
    - All standard keys (A-Z, 0-9, symbols, modifiers)
    - Function keys (F1-F24)
    - Navigation cluster (Insert, Delete, Home, End, Page Up/Down)
    - Arrow keys (Up, Down, Left, Right)
    - Numeric keypad (full support)
    - Multimedia keys (Volume, Play/Pause, Next/Previous, Stop)
    - Browser controls (Back, Forward, Refresh, Home, Search, Favorites)
    - Application launchers (Mail, Calculator, My Computer)
    - Power management (Power, Sleep, Wake)
    - International keys (Japanese and Korean keyboard support)
    - **Special keys** (Print Screen, Pause/Break with complex multi-byte sequences)
- **Typematic Repeat** (Enhanced in v2.0):
    - Delay: 500ms before repeat starts
    - Rate: ~30 repeats per second (33ms interval)
    - Works with all keys including E0-prefixed extended keys
    - Preserves E0 prefix during repeats
    - Properly disabled during mode transitions
    - Uses complete mapping structure for any keycode

## Debugging

### Serial Debug Output

The firmware includes comprehensive debug output via USB serial (CONSOLE feature enabled):

```bash
# Monitor serial output
qmk console
```

Example output:

```
================================
Mode switch: PS/2
================================
[PS2] PS/2 driver activated
[DEBUG] Key pressed: keycode=0x0004 (PS/2 mode)
[PS2] Key pressed: keycode=0x0004, scancode=0x1C
[PS2] Typematic repeat: keycode=0x0004, scancode=0x1C
[DEBUG] Key released: keycode=0x0004 (PS/2 mode)
[PS2] Key released: keycode=0x0004, scancode=0x1C

[DEBUG] Key pressed: keycode=0x00E1 (PS/2 mode)
[PS2] Modifier pressed: 0x02 (scancode: 0x12)
[DEBUG] Key released: keycode=0x00E1 (PS/2 mode)
[PS2] Modifier released: 0x02 (scancode: 0x12)

================================
Mode switch: USB
================================
[USB] USB driver restored
```

All events show:

- QMK keycode in hex
- PS/2 scancode in hex
- E0 prefix status
- Current mode (USB/PS/2)

### Testing with Python

To verify PS/2 output, use the included `ps2_decoder.py` script on a second Raspberry Pi Pico:

1. Upload `ps2_decoder.py` to a second Pico using Thonny or mpremote
2. Wire GP16 (Clock) and GP17 (Data) to your keyboard's PS/2 lines
3. Run the script and press keys

The decoder will show:

```
✓ KEY DOWN: A
  Break prefix (0xF0)
✓ KEY UP:   A
```


See [QUICKSTART.md](QUICKSTART.md#for-testingdebugging) for detailed instructions.

## Customization

### Adding More Keys

The firmware includes **complete PS/2 Scan Code Set 2 support**, so you can add any standard keyboard key to your keymap!

Edit `kb.h` and to define the physical layout:

```c
// This the physical layout of a 3-key keyboard.
#define KEYMAP( \
    k00 \ k01 \ k02 \
) { \
    { k00, k01, k02 } \
}
```

Edit `keymaps/default/keymap.c` and `info.json` to add more keys:

```c
// Example: 3-key keyboard
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT_ortho_1x3(
        KC_A, KC_B, KC_C
    )
};
```
See [SCANCODES.md](SCANCODES.md) for detailed instructions.



**No additional coding required!** The firmware automatically:

- Maps QMK keycodes to PS/2 scancodes via lookup tables
- Sends E0 prefix for extended keys
- Handles special multi-byte sequences (Print Screen, Pause)
- Handles typematic repeat for all keys

### Supported Key Categories

All of these work out of the box:

|Category|Example Keys|Notes|
|---|---|---|
|**Letters**|KC_A through KC_Z|Standard|
|**Numbers**|KC_1 through KC_0|Standard|
|**Function Keys**|KC_F1 through KC_F24|F13-F24 supported|
|**Modifiers**|KC_LSFT, KC_LCTL, KC_LALT, KC_LGUI|Left and right|
|**Navigation**|KC_HOME, KC_END, KC_PGUP, KC_PGDN|Auto E0 prefix|
|**Arrows**|KC_UP, KC_DOWN, KC_LEFT, KC_RGHT|Auto E0 prefix|
|**Editing**|KC_INS, KC_DEL, KC_BSPC|Auto E0 prefix|
|**Multimedia**|KC_MUTE, KC_VOLU, KC_VOLD, KC_MPLY|Auto E0 prefix|
|**Browser**|KC_WSCH, KC_WHOM, KC_WBAK, KC_WFWD|Auto E0 prefix|
|**Applications**|KC_MAIL, KC_CALC, KC_MYCM|Auto E0 prefix|
|**Power**|KC_PWR, KC_SLEP, KC_WAKE|Auto E0 prefix|
|**Keypad**|KC_P0 through KC_P9, KC_PPLS, KC_PMNS|Full numpad|
|**International**|KC_INT1 through KC_INT5, KC_LNG1, KC_LNG2|Japanese/Korean|
|**Special**|KC_PSCR, KC_PAUS|Multi-byte sequences|

See SCANCODES.md for the complete mapping table.

### Adjusting Typematic Rate

Modify the typematic settings in `ps2_keyboard.c`:

```c
} typematic_state = {
    ...
    .delay_ms = 500,  // Default 500ms delay
    .rate_ms = 33,     // Default ~30Hz repeat rate
    ...
};
```

## Technical Details

### Why PS/2 Device Mode?

Most PS/2 projects implement **host** mode (reading from a PS/2 keyboard). This project implements **device** mode, making the microcontroller _act as_ a PS/2 keyboard.

**Why build this?**

- I recently learned how the PS/2 protocol works and wanted to see if I could implement it
- Many FPGA development boards and retro computers have PS/2 ports
- Having dual-mode support means you can use the same keyboard on your modern PC (USB) and your FPGA dev board or vintage computer (PS/2)
- **Less desk clutter** - No need to keep multiple keyboards around
- **PS/2 keyboards are getting harder to find** - Most modern keyboards are USB-only, and finding quality PS/2-compatible keyboards is increasingly difficult

**Practical use cases:**

- **FPGA/CPLD development** - Many dev boards (ALTERA DE1, spartan 3E, etc.) have PS/2 ports
- **Retro computing and vintage hardware** - 486, Pentium, and early 2000s systems
- **Embedded systems** that use PS/2 input
- **Industrial equipment** with PS/2 interfaces
- **Testing and development** without needing multiple keyboards
- **One keyboard for everything** - Modern work machine and vintage/embedded projects

### Challenges Solved (v2.0)

1. ✅ **Idle State Management**: Lines must be properly released and stabilized between transmissions
2. ✅ **Typematic Repeat**: Firmware handles key repeat timing with E0 prefix preservation
3. ✅ **Mode Switching**: USB driver properly saved and restored during transitions
4. ✅ **State Management**: Clean state transitions prevent phantom repeats
5. ✅ **Special Keys**: Complex multi-byte sequences for Print Screen and Pause
6. ✅ **Code Organization**: Lookup tables instead of giant switch statements

### Architecture

```
┌─────────────────────────────────────────────┐
│           QMK Core (matrix scan)            │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
         ┌────────────────┐
         │  Mode Switch   │ ◄─── GP14 (Hardware)
         │   (kb.c)       │
         └────────┬───────┘
                  │
        ┌─────────┴──────────┐
        │                    │
        ▼                    ▼
┌──────────────┐    ┌──────────────────┐
│  USB Driver  │    │  PS/2 Driver     │
│  (QMK Core)  │    │ (ps2_keyboard.c) │
└──────────────┘    └─────────┬────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
            ┌──────────────┐   ┌──────────────┐
            │ Lookup Tables│   │ GPIO Bit-bang│
            │(scancodes.h) │   │CLK/DATA: GP16│
            └──────────────┘   │     /GP17    │
                               └──────────────┘
```

### Future Enhancements

- ☐ Host-to-device command handling (LED updates, scan code set switching)
- ☐ PS/2 mouse device implementation (pins already allocated)
- ☐ Software toggle via keypress instead of hardware switch
- ☐ Testing and support for more microcontrollers (AVR, STM32, etc.)

## Troubleshooting

### Keyboard not recognized in PS/2 mode

- Check wiring (especially clock and data)
- Verify 5V power to PS/2 port
- Ensure mode switch is set to LOW (PS/2 mode)
- Check debug console for initialization messages
- **Voltage level issues**: If your PS/2 host doesn't recognize the keyboard, it may not accept 3.3V logic levels. Try using a bidirectional level shifter (3.3V ↔ 5V)

### Keys not registering

- Monitor serial debug output (`qmk console`)
- Verify button is connected to correct GPIO (GP15)
- Check that firmware is in correct mode
- If in PS/2 mode, ensure the host computer recognizes the device first

### Typematic repeat not working

- Ensure you're holding the key for > 500ms
- Check debug output for "Typematic repeat" messages
- Verify `ps2_keyboard_task()` is being called regularly in `housekeeping_task_kb()`

### USB not working after switching from PS/2

- **This was fixed in v2.0!** Update to the latest version
- If still having issues, check that `original_usb_driver` is being saved during mode switch
- Monitor debug output for "USB driver restored" message

### Intermittent PS/2 behavior

- **Improved in v2.0** with better idle state management
- This can still be a voltage level issue
- Add a bidirectional level shifter between RP2040 and PS/2 connector
- Ensure good connections (no loose wires)
- Check that pull-up resistors are properly configured

### Buffer Overflow Warning

If you see `[PS2] WARNING: Send buffer full!` messages:

- You're typing faster than PS/2 can transmit (unlikely with single button)
- Check that `ps2_keyboard_task()` is being called regularly
- Increase `PS2_SEND_BUFFER_SIZE` if needed (currently 32 bytes)

## License

This project is licensed under the GPL-2.0 License - see the LICENSE file for details.

This license matches QMK Firmware's GPL-2.0 license for maximum compatibility.

## Credits

- **Author**: Betzalel J. Lewis
- **Framework**: [QMK Firmware](https://qmk.fm/)
- **Hardware**: Raspberry Pi RP2040
- **Testing**: Community feedback and extensive real-world testing

## References

- [PS/2 Protocol Specification](https://www.avrfreaks.net/sites/default/files/PS2%20Keyboard.pdf)
- [QMK Documentation](https://docs.qmk.fm/)
- [RP2040 Datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)
- Complete Scancode Reference - Full PS/2 Scan Code Set 2 table
- Quick Start Guide - Step-by-step setup instructions

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

**Areas where contributions would be especially appreciated:**

- PS/2 mouse device implementation
- Host-to-device command handling
- Testing and porting to other microcontrollers (AVR, STM32, ESP32, etc.)
- Testing with vintage computers

---

**Note**: This is a demonstration project showing PS/2 device implementation. For production use, you would want to add full keyboard matrix support, host command handling, and more robust error handling.

## Star History

If you find this project useful, please consider giving it a star! ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard&type=Date)](https://star-history.com/#BuzzL123/QMK-PS2-USB-Dual-Mode-Keyboard&Date)
