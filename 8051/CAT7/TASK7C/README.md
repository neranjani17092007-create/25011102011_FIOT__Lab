# TASK7C — Keypad-controlled LCD character movement

## Aim

Move X within the visible 2×16 LCD area using directional numeric keys.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P2.0 / P2.1 | LCD RS / EN |
| P2.4–P2.7 | LCD D4–D7 |
| P1.0–P1.3 | Keypad rows |
| P1.4–P1.7 | Keypad columns |
| LCD R/W | Ground for write-only operation |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

keypad_getkey() drives one row LOW and decodes a conventional 4×4 key map. The main loop confirms a key after delay(20), erases the old X, updates its row/column, redraws it, and waits for release.

## Pseudocode

```text
START X at row 0, column 0
CONFIRM key
2: up; 8: down; 4: left; 6: right
CLAMP movement to rows 0..1 and columns 0..15
REDRAW X; wait for key release
```

## Expected output and verification

Each accepted directional press moves X one position. Movement stops at screen edges; other keys do not change its coordinates.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

Debouncing and release waiting are blocking. Holding a key does not produce automatic repeated movement. The delay values are loop-based, not measured time.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK7C.uvproj](TASK7C.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK7C.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
