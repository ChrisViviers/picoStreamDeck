# Pico Stream Deck

A DIY stream deck built on a Raspberry Pi Pico, with a 3×3 grid of buttons, a volume knob and a 0.96" OLED screen. It plugs in over USB as an ordinary keyboard, so there are no drivers to install, and it works on Windows, macOS and Linux.

- **9 buttons** in a 3×3 matrix that send hotkeys (F13–F19) or media keys
- **Volume knob** that controls your computer's system volume
- **OLED screen** showing the last button pressed and a live volume bar
- Written in **CircuitPython**, with every button easy to remap

## Parts

| Part | Qty | Notes |
| --- | --- | --- |
| Raspberry Pi Pico | 1 | Pico, Pico W, Pico 2 or Pico 2 W |
| Tactile momentary push buttons | 9 | 12 mm buttons can take snap-on keycaps |
| [0.96" yellow/blue 128×64 OLED (I2C, SSD1306)](https://www.pishop.co.za/store/yellow-blue-double-colour-128x64-oled-lcd-display-module-for-arduino-096-i2c-iic-spi-serial) | 1 | Pins labelled GND, VDD, SCK, SDA |
| 10 kΩ linear potentiometer | 1 | Marked "B10K", not "A" (logarithmic) |
| Perfboard or breadboard, hookup wire, 3D Printed Mount | – | |
| USB data cable | 1 | Must carry data, not charge-only |

## Wiring

| Part | Connection | Pico pin (physical pin #) |
| --- | --- | --- |
| Row 1 | buttons 1, 2, 3 | GP6 (9) |
| Row 2 | buttons 4, 5, 6 | GP7 (10) |
| Row 3 | buttons 7, 8, 9 | GP8 (11) |
| Column 1 | buttons 1, 4, 7 | GP9 (12) |
| Column 2 | buttons 2, 5, 8 | GP10 (14) |
| Column 3 | buttons 3, 6, 9 | GP11 (15) |
| OLED GND | ground | GND (38) |
| OLED VDD | power | 3V3 OUT (36) |
| OLED SCK | I2C clock | GP5 (7) |
| OLED SDA | I2C data | GP4 (6) |
| Pot left leg | reference voltage | ADC_VREF (35) |
| Pot middle leg | wiper | GP26 (31) |
| Pot right leg | ground | AGND (33) |

Buttons are numbered left to right, top row first. Connect one leg of each button to its row wire and the other to its column wire. Use diagonally opposite legs, because four-leg tactile buttons have pairs that are always connected.

If you fit diodes, put one between each button and its row wire with the **striped end facing the row**.

The pot's middle leg must go to GP26, one of the Pico's analog pins. ADC_VREF and AGND give a steadier reading than the regular 3V3 and GND pins. If the knob works backwards, swap the outer legs.

## Installation

### 1. Install Thonny and CircuitPython

1. Download and install [Thonny](https://thonny.org).
2. Hold the **BOOTSEL** button on the Pico while plugging it into your computer, then let go.
3. In Thonny, open **Tools → Options → Interpreter** and choose **CircuitPython (generic)**.
4. Click **Install or update CircuitPython (UF2)** in the bottom right of that window.
5. Select the Pico as the target volume, choose the variant that matches your board, and click **Install**.
6. When it finishes, the Pico restarts and appears as a USB drive called **CIRCUITPY**. Back in the Interpreter settings, select the Pico's port and click OK.

### 2. Install the libraries

The deck needs three Adafruit libraries:

- `adafruit_hid` (folder)
- `adafruit_display_text` (folder)
- `adafruit_displayio_ssd1306.mpy` (single file)

**Option A: from the library bundle.** Open `boot_out.txt` on CIRCUITPY to see your CircuitPython version. Download the matching bundle from [circuitpython.org/libraries](https://circuitpython.org/libraries), unzip it, and copy the three items above from its `lib` folder into a `lib` folder on CIRCUITPY.

### 3. Upload the code

Open `src/code.py` in Thonny, choose **File → Save as… → CircuitPython device**, and save it as `code.py`. You can also drag the file onto the CIRCUITPY drive. The Pico restarts and runs it straight away.

Your drive should look like this:

```
CIRCUITPY/
├── code.py
└── lib/
    ├── adafruit_hid/
    ├── adafruit_display_text/
    └── adafruit_displayio_ssd1306.mpy
```

## Testing

The `tests` folder has two scripts for checking your wiring before running the full deck. Open one in Thonny and press **Run** (F5). Output appears in Thonny's shell, and the Pico's onboard LED lights while a button is held.

| Script | What it tests | Wiring |
| --- | --- | --- |
| `tests/button_test.py` | A single loose button | One leg to GP15 (pin 20), diagonal leg to GND (pin 18) |
| `tests/matrix_test.py` | The whole 3×3 matrix | As in the wiring table above |

The matrix test prints lines like `Button 5 (row 2, col 2) pressed`. If a whole row or column is dead, that wire isn't reaching the Pico. If buttons report the wrong number, two row or column wires are swapped.

## Configuration

All settings are at the top of `src/code.py`.

**`BUTTONS`** sets each button's on-screen name (10 characters max) and action. An action is either a tuple of keycodes for a key or shortcut, or a media key:

```python
("Scene 1", (Keycode.F13,)),                               # single key
("Mic Mute", (Keycode.CONTROL, Keycode.SHIFT, Keycode.M)), # shortcut
("Next", ConsumerControlCode.SCAN_NEXT_TRACK),             # media key
```

| Setting | Default | What it does |
| --- | --- | --- |
| `VOL_STEPS` | `50` | Volume steps on your OS. Use 50 for Windows, 16 for macOS |
| `SYNC_ON_BOOT` | `True` | At startup, set the computer's volume to match the knob |
| `OLED_ADDRESS` | `0x3C` | I2C address of the display. Some modules use `0x3D` |

## Binding buttons in your apps

Most apps have a hotkey screen where you click a box and press a key. Press a deck button instead.

- **OBS Studio:** Settings → Hotkeys, click the box next to an action, press a deck button, then Apply.
- **Discord:** User Settings → Keybinds → Add a Keybind, choose an action, click Record Keybind and press the button.
- **AutoHotkey (Windows):** launch programs or websites, for example `F13::Run "notepad.exe"`.

If an app ignores F13–F24, change that button to send an ordinary shortcut instead.

## How the volume knob works

USB devices can't set an exact volume level. They can only press "volume up" and "volume down". The Pico measures how far the knob has turned and sends the matching number of presses. At startup it drops the volume to zero and brings it up to the knob's position. If you change the volume another way, turn the knob fully left or right to re-sync it.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `ImportError: no module named 'board'` | The Pico is running MicroPython. Install CircuitPython (step 1) |
| `ImportError: no module named 'machine'` | You're running a MicroPython script. Use the scripts in this repo |
| "Incompatible .mpy file" | Your library bundle doesn't match your CircuitPython version |
| Screen stays blank | Check SCK and SDA aren't swapped, then try `OLED_ADDRESS = 0x3D` |
| Volume flickers with the knob still | Change `0.7` in `knob_level()` to `0.9` |
| Knob works backwards | Swap the pot's outer legs |
| Volume jumps on a Mac | Set `VOL_STEPS = 16` |

## Ideas for next steps

- GP12–GP15 are still free for a second knob, a rotary encoder or more buttons
- Swap the tactile buttons for Cherry MX-style mechanical switches
- 3D print a case

## License

MIT. See [LICENSE](LICENSE).
