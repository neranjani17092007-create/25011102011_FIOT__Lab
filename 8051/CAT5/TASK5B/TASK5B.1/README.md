# TASK5B.1 — Timer 0 software PWM LED fading

## Aim

Generate a 100-slot PWM cycle and automatically sweep its duty value.

## Circuit diagram

![Timer 0 software PWM fading — circuit connections](circuit.png)

[Open or download circuit.png](circuit.png)

This diagram follows the port assignments in `main.c`. It is a proposed pin-level connection diagram, not a verified record of the physical lab wiring. Board header numbers, component values and module variants must be checked against the hardware. The LED interface shown is a suitable non-inverting current driver for the code's active-HIGH LED interpretation; it does not establish that such a driver was used in the lab. `R*` denotes a current-limiting resistor whose value is unconfirmed.

## Source and hardware

This description follows the committed [main.c](main.c). It describes expected behavior from source inspection, not a new hardware test. The code uses 8051 register definitions and Keil C51 interrupt syntax.

| Signal | Connection / purpose |
| --- | --- |
| P1.0 | PWM LED output |

Pin names are MCU port bits, not development-board header numbers. Match the board wiring and logic polarity; use current-limiting resistors for LEDs and compatible electrical levels for connected devices.

## Program logic

Timer 0 mode 1 reloads 0xFFF7. The ISR sets LED HIGH when pwm_count < duty and LOW otherwise. After 100 slots, pwm_count resets; after 10 PWM cycles, duty moves one step toward 100 or 0. direction reverses at the endpoints.

## Pseudocode

```text
ON timer interrupt: output (pwm_count < duty)
EVERY 100 slots: complete PWM cycle
EVERY 10 cycles: adjust duty up/down
```

## Expected output and verification

The logical duty setting rises from 0 to 100 and falls back, producing a repeating fade for an active-HIGH LED if the timing is adequate.

Build the project, run it on the intended board, and compare the output with this behavior. Record actual timings or observations separately; stored build artifacts alone do not verify the current source on hardware.

## Implementation notes

This is software PWM driven by Timer 0. 0xFFF7 leaves only nine timer counts before overflow; ISR execution can dominate this interval. Do not infer a measured PWM frequency or perfectly linear brightness from the reload alone.

## Files and build steps

- [main.c](main.c): application source.
- [STARTUP.A51](STARTUP.A51): 8051 startup code.
- [TASK5B.1.uvproj](TASK5B.1.uvproj): Keil project.
- `Objects/` and `Listings/`: stored build outputs and listings; rebuild before relying on them.

1. Open `TASK5B.1.uvproj` in Keil µVision with C51 support.
2. Confirm the selected device, oscillator setting, include paths and output settings against the hardware.
3. Build/rebuild and inspect compiler messages. Generate a HEX file if required by the board programmer.
4. Program the board using its supported programmer, connect the signals above, and check the expected behavior.
