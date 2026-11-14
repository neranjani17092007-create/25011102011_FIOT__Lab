# TASK8C — Short-press toggle and long-press blink

## Aim

Classify a debounced button press on release and select steady or blinking LED behavior.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P3.2 | Active-LOW button input |
| P1.0 | LED output |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

Timer 0 reloads 0xFC66 to maintain ms_ticks. After 30 stable ticks, a press records pressStart. On the debounced release, duration is measured: less than 1000 ticks toggles the LED, or stops blinking and forces LOW; 1000 ticks or more enters blinkMode, sets LED HIGH and records lastBlink. Blinking toggles the LED every 250 ticks.

## Pseudocode

```text
DEBOUNCE button
ON press: save start time
ON release: classify duration
SHORT: toggle, or stop blinking and turn OFF
LONG: enter blinking
WHILE blinking: toggle every 250 ticks
```

## Expected output and verification

A short press toggles the steady LED. A long press starts blinking only after release. A later short press stops blinking and turns the LED OFF. Nominal blinking has a 500 ms full cycle.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

This is 8051 code. Timings assume approximately 1 ms ticks. ms_ticks lacks volatile despite ISR writes. The 16-bit tick counter wraps; very long holds spanning its full range cannot be classified reliably by this duration calculation.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK8C.uvproj](TASK8C.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK8C.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
