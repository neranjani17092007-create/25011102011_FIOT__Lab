# TASK3A — Button-controlled LED pattern modes

## Aim

Select four LED patterns with an active-LOW push button and interrupt-assisted software debouncing.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P1.0–P1.3 | Four LED outputs |
| P3.2 / INT0 | Active-LOW button input |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

Timer0_ISR() reloads Timer 0 with 0xFC66 and increments ms_ticks. INT0_ISR() disables EX0 and sets buttonEvent. The main loop confirms a LOW after 30 ticks, advances the mode, and requires a stable HIGH release for 30 ticks before re-enabling INT0. LED_Display() preserves the upper nibble of P1. get_ms() temporarily masks ET0 while copying the 16-bit counter.

## Pseudocode

```text
INITIALIZE timer and falling-edge INT0
ON confirmed press: cycle mode 1 → 2 → 3 → 4 → 1
EVERY 250 ticks: update selected pattern
AFTER stable release: re-enable INT0
```

## Expected output and verification

Startup leaves the four outputs LOW. Mode 1 runs 0x01, 0x02, 0x04, 0x08; mode 2 counts 0–15; mode 3 alternates 0x0A/0x05; mode 4 alternates 0x0F/0x00.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

Debouncing is implemented in software, not by a documented hardware debounce circuit. Pattern scheduling is non-blocking. The 30/250 timing values assume the intended approximately 1 ms timer tick.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK3A.uvproj](TASK3A.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK3A.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
