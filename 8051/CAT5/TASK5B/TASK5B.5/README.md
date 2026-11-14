# TASK5B.5 — Burst waveform output

## Aim

Generate a repeating sequence of toggling and forced-LOW intervals.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P1.1 | Burst output |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

Timer 1 mode 1 reloads 0xFC66. For counts 0–9 the ISR toggles OUTPUT; counts 10–19 force LOW; counts 20–29 toggle again. count then wraps to zero.

## Pseudocode

```text
ON interrupt: reload
IF count in 10..19: output LOW
ELSE: invert output
INCREMENT count modulo 30
```

## Expected output and verification

At the nominal 1 ms tick: 10 toggles, 10 LOW intervals, then 10 toggles. Each group of 10 toggles represents five square-wave cycles.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

The toggling block at counts 20–29 directly joins the next block at counts 0–9. Consequently, after startup, the repeating waveform has 20 consecutive toggling intervals between 10-interval LOW gaps. It is not separated into two independent bursts per cycle.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK5B.5.uvproj](TASK5B.5.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK5B.5.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
