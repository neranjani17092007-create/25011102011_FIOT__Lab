# TASK5C — Keypad-controlled stepper sequencing

## Aim

Select a step interval using keys 1–3 while Timer 0 advances a one-coil sequence.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P1.0–P1.3 | Coil sequence outputs to a motor driver |
| P2.7–P2.4 | Keypad rows in descending order |
| P2.3–P2.0 | Keypad columns in descending order |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

keypad_scan() drives one row LOW and returns a key; only the first row's first three columns map to 1, 2 and 3. set_speed() selects step_interval=1, 5 or 10. Timer 0 reloads 0xDC00; after step_interval interrupts, step_motor() shifts coil right and wraps 0x01 to 0x08.

## Pseudocode

```text
INITIALIZE coil=0x08 and timer
FOREGROUND: scan key; select interval
ISR: count overflows; when interval reached, output next coil
```

## Expected output and verification

The sequence is 0x08 → 0x04 → 0x02 → 0x01 → 0x08. At 11.0592 MHz with a classic 12-clock timer, nominal step intervals are 10, 50 and 100 ms for keys 1, 2 and 3.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

Use a suitable motor driver and supply; GPIO signals must not directly power motor windings. Rotation direction depends on coil wiring. There is no explicit debounce or release wait. step_interval is shared with the ISR but is not declared volatile in this source.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK5C.uvproj](TASK5C.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK5C.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
