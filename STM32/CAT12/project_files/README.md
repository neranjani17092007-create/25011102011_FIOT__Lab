# Continuous Lab Evaluation 12: Nucleo LED Blink Timing

## 1. Experiment Title

Nucleo LED Blink Timing using STM32F401RET6 / NUCLEO-F401RE.

## 2. Aim

Repeats six slow LED toggles, a 500 ms pause, and six fast LED toggles on PA5. The uploaded program is a timed blink sequence rather than button-controlled toggling.

## 3. Hardware Components

- STM32F401RET6 / NUCLEO-F401RE
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table

## 4. Software Tools

- STM32CubeMX: `NUCLEO_LED_blink.ioc`
- Keil MDK-ARM: `MDK-ARM/NUCLEO_LED_blink.uvprojx`
- STM32 HAL and CMSIS sources included in the project

## 5. Theory

GPIO and UART behavior depends on pin mode and active level. A logical HIGH or LOW describes the electrical output; visible LED state also depends on its wiring. HAL calls provide peripheral access while SysTick supplies software timing.

## 6. Actual Pin Configuration

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PA5 | GPIO output | Nucleo user LED | Toggled by firmware |
| PC13 | B1 falling-edge EXTI configuration | Board user button | Configured; not used by the blink loop |
| PA2 / PA3 | USART2 TX / RX | Board serial interface | 115200 8N1 configured, no application traffic |
| GND | Ground | All connected devices | Common logic reference |

The connections are derived from the source and configuration, rather than a photographed lab assembly. Check board routing and module power pins before wiring. MCU port labels are not connector pin numbers.

## 7. Circuit Diagram

Circuit PNG upload pending. Use the pin table above.



## 8. CubeMX / Peripheral Configuration

- System clock: HSI 16 MHz through PLL M=16, N=336, P=4 gives 84 MHz; APB1 42 MHz, APB2 84 MHz.
- Configuration: [NUCLEO_LED_blink.ioc](NUCLEO_LED_blink.ioc).
- Peripheral assignments and operational rates are listed above; the actual initialization in `main.c` is the basis for runtime settings.

1. Open [NUCLEO_LED_blink.uvprojx](MDK-ARM/NUCLEO_LED_blink.uvprojx) with the matching STM32 device pack installed.
2. Keep `Core/`, `Drivers/`, `MDK-ARM/` and the IOC together. The reorganization preserves these internal relative paths.
3. Check the device and clock settings, build, and review compiler/linker diagnostics. Existing output files may predate the source.
4. Program the intended board using ST-LINK. Make connections from the pin table and confirm the stated logic levels.
5. Compare the observed behavior with the expected output below.

## 9. Key APIs Used

- `HAL_GPIO_TogglePin()`
- `HAL_Delay()`
- `HAL_UART_Init()`

## 10. Source Code Logic

Each slow iteration toggles PA5 and delays 1000 ms; six iterations follow. HAL_Delay(500) adds a pause. A second six-iteration loop toggles with a 250 ms delay. PA5 starts LOW, and six toggles return it to its previous level.

Read [main.c](Core/Src/main.c) for the complete implementation.

## 11. Algorithm

1. Initialize HAL and 84 MHz system clock.
2. Configure PA5 output; initialize USART2.
3. Toggle PA5 six times, 1000 ms apart.
4. Wait 500 ms.
5. Toggle PA5 six times, 250 ms apart.
6. Repeat sequence.

## 12. Pseudocode

```text
1. Initialize HAL and 84 MHz system clock
2. Configure PA5 output; initialize USART2
3. Toggle PA5 six times, 1000 ms apart
4. Wait 500 ms
5. Toggle PA5 six times, 250 ms apart
6. Repeat sequence
```

## 13. Flowchart

Flowchart PNG upload pending. Refer to the algorithm and pseudocode above.

## 14. Expected Output

The PA5 LED alternates slowly for three full cycles, pauses, then alternates quickly for three full cycles. The pattern repeats.

## 15. Actual Observations

No new physical test was performed for this update. The behavior described here is inferred from the uploaded source. Add measured readings, terminal captures and board-specific observations after running the project.

## 16. Troubleshooting

1. Check board power, shared GND and the exact MCU port names before interpreting a missing output.
2. Open the matching project and rebuild; do not assume stored build artifacts reflect the current source.

## 17. Limitations and Implementation Notes

- Six toggles correspond to three complete ON/OFF cycles, not six complete blinks.
- The nominal loop duration is 8000 ms plus execution overhead: 6000 + 500 + 1500 ms.
- PC13 and USART2 are configured but do not affect the shown application loop.

## 18. Result and Verification

Reviewed against the uploaded `Core/Src/main.c`, `Core/Inc/main.h`, `Core/Src/stm32f4xx_hal_msp.c` and `NUCLEO_LED_blink.ioc`. This documentation update does not claim a new Keil build, hardware run, measured waveform or successful network connection.

The uploaded program implements the behavior described above. Hardware success remains to be confirmed against the expected output.

## 19. Viva Questions with Answers

1. **How many complete blinks are six toggles?**  
   Three, because a complete cycle needs two state changes.

2. **Is HAL_Delay non-blocking?**  
   No. The foreground waits for the requested interval.

3. **Does B1 change the pattern?**  
   Not in the uploaded application loop.

## 20. File Descriptions

| File / folder | Purpose |
|---|---|
| [Core/Src/main.c](Core/Src/main.c) | Main application and peripheral initialization |
| [Core/Inc/main.h](Core/Inc/main.h) | Public declarations and GPIO macros |
| `Core/Src/*hal_msp.c` | Peripheral clocks and alternate-function GPIO setup |
| [NUCLEO_LED_blink.ioc](NUCLEO_LED_blink.ioc) | STM32CubeMX configuration |
| [MDK-ARM/NUCLEO_LED_blink.uvprojx](MDK-ARM/NUCLEO_LED_blink.uvprojx) | Keil project |
| `Drivers/` | Included HAL and CMSIS dependencies |

[Return to task overview](../README.md)
