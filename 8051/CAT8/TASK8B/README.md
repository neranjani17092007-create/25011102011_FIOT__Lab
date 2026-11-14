# TASK8B — Debounced LED toggle

## Aim

Toggle the LED once for each stable button press.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P3.2 | Active-LOW button input |
| P1.0 | LED output |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

Timer 0 mode 1 reloads 0xFC66 and increments ms_ticks. get_ms() masks ET0 while copying the counter. Each raw state change restarts debounceTime; after 30 unchanged ticks, buttonState is updated. Only a stable LOW transition toggles the LED.

## Pseudocode

```text
READ button and time
ON raw change: restart debounce timer
AFTER 30 stable ticks: accept state
IF newly pressed: invert LED
```

## Expected output and verification

The first accepted press turns the output HIGH, the next LOW; releasing or holding the button does not repeatedly toggle it.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

This is an 8051 task. Debounce timing assumes the intended approximately 1 ms tick. ms_ticks is modified in the ISR but is not declared volatile; compiler optimization should be considered when validating the firmware.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK8B.uvproj](TASK8B.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK8B.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
