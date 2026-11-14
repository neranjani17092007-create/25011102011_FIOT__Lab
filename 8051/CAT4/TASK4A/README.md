# TASK4A — Keypad, seven-segment display and LED patterns

## Aim

Read numeric keys, display the selected digit, and select an LED or pattern.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P2.3, P2.2, P2.1 | Scanned keypad drive lines |
| P2.7–P2.4 | COL1–COL4 inputs |
| P0 | Seven-segment pattern byte |
| P1.1–P1.4 | Four LED outputs |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

keypad_scan() drives P2 with 0xF7, 0xFB and 0xFD, confirms a press after delay(), and waits for release. display_digit() writes active-HIGH segment codes for 0–9. Keys 1–4 select one LED; 5 runs the LEDs; 6 alternates two pairs; 7 counts 0–15; 8 flashes all LEDs; 9 clears P1 and stops pattern execution. Key 0 displays 0 and sets mode 9 without clearing P1.

## Pseudocode

```text
SCAN keypad and update digit
DISPATCH mode: single LED / running / alternate / binary / flash
REPEAT
```

## Expected output and verification

The digit and LED behavior follow the selected numeric key. Key 0 stops the active mode while retaining the last LED output; key 9 explicitly clears LEDs.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

Only the numeric-key combinations present in the code are handled; this is not a complete 16-key decoder. Blocking delays and release waits reduce responsiveness, especially during the full binary count. Segment codes imply conventional active-HIGH/common-cathode wiring; confirm actual segment order and the required Port 0 pull-ups/driver circuit.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK4A.uvproj](TASK4A.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK4A.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
