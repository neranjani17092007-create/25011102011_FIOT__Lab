# TASK5A — Timer 0 interrupt LED blink

## Aim

Toggle P1.0 after 1000 Timer 0 interrupts.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P1.0 | LED output |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

TMOD=0x01 selects Timer 0 mode 1. Each interrupt reloads TH0/TL0 to 0xFC66, increments count and toggles LED when count reaches 1000. ET0, EA and TR0 enable the interrupt and timer; the foreground loop is empty.

## Pseudocode

```text
ON Timer 0 overflow: reload; increment count
IF count >= 1000: invert LED; clear count
```

## Expected output and verification

With the intended approximately 1 ms tick, the LED changes state about once per second, giving an approximately two-second complete ON/OFF cycle.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

This task is timer-based blinking, not PWM fading. LED is not explicitly initialized in main(). Timing depends on the actual oscillator and interrupt overhead.

For a classic 12-clock 8051 timer, nominal overflow time is `(65536 − reload) × 12 / f_osc`. At 11.0592 MHz, `0xFC66` gives approximately 1 ms and `0xDC00` gives 10 ms before interrupt/reload overhead. Confirm the board oscillator before treating these as real-time measurements.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK5A.uvproj](TASK5A.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK5A.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
