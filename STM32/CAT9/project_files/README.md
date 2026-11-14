# Continuous Lab Evaluation 9: UART Command-Controlled LED

## 1. Experiment Title

UART Command-Controlled LED using STM32F103C8T6 / Blue Pill.

## 2. Aim

Receives newline-terminated commands over USART1, echoes characters, and controls the active-LOW PC13 LED. A bounded 32-byte command buffer handles overlong input.

## 3. Hardware Components

- STM32F103C8T6 / Blue Pill
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- USB-UART adapter with 3.3 V logic and serial terminal

## 4. Software Tools

- STM32CubeMX: `BluePill_UART1.ioc`
- Keil MDK-ARM: `MDK-ARM/BluePill_UART1.uvprojx`
- STM32 HAL and CMSIS sources included in the project

## 5. Theory

GPIO and UART behavior depends on pin mode and active level. A logical HIGH or LOW describes the electrical output; visible LED state also depends on its wiring. HAL calls provide peripheral access while SysTick supplies software timing.

## 6. Actual Pin Configuration

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PA9 | USART1 TX | 3.3 V USB-UART RX | Echo and replies, 9600 8N1 |
| PA10 | USART1 RX | 3.3 V USB-UART TX | Commands, 9600 8N1 |
| PC13 | GPIO output | Active-LOW LED | LOW=ON; HIGH=OFF |
| PA13 / PA14 | SWD | ST-LINK | Programming |
| GND | Ground | All connected devices | Common logic reference |

The connections are derived from the source and configuration, rather than a photographed lab assembly. Check board routing and module power pins before wiring. MCU port labels are not connector pin numbers.

## 7. Circuit Diagram

Circuit PNG upload pending. Use the pin table above.



## 8. CubeMX / Peripheral Configuration

- System clock: Internal HSI 8 MHz; PLL disabled; AHB/APB1/APB2 undivided.
- Configuration: [BluePill_UART1.ioc](BluePill_UART1.ioc).
- Peripheral assignments and operational rates are listed above; the actual initialization in `main.c` is the basis for runtime settings.

1. Open [BluePill_UART1.uvprojx](MDK-ARM/BluePill_UART1.uvprojx) with the matching STM32 device pack installed.
2. Keep `Core/`, `Drivers/`, `MDK-ARM/` and the IOC together. The reorganization preserves these internal relative paths.
3. Check the device and clock settings, build, and review compiler/linker diagnostics. Existing output files may predate the source.
4. Program the intended board using ST-LINK. Make connections from the pin table and confirm the stated logic levels.
5. Compare the observed behavior with the expected output below.

## 9. Key APIs Used

- `HAL_UART_Receive()`
- `HAL_UART_Transmit()`
- `HAL_GPIO_WritePin()`
- `HAL_GPIO_TogglePin()`

## 10. Source Code Logic

At startup the program prints HELLO and WORLD, waits 3 seconds, and prints the command prompt. Characters are echoed and buffered until CR or LF. LED ON writes PC13 LOW; LED OFF writes HIGH; LED TOGGLE toggles. Unknown and overlong commands receive explicit replies. Empty terminators, including LF following CR, are ignored.

Read [main.c](Core/Src/main.c) for the complete implementation.

## 11. Algorithm

1. Initialize HSI, GPIO and USART1 at 9600.
2. Print startup text and ready prompt.
3. Receive and echo one byte.
4. Accumulate up to 31 command characters.
5. On CR/LF: dispatch command or report error.
6. Reset buffer state; repeat.

## 12. Pseudocode

```text
1. Initialize HSI, GPIO and USART1 at 9600
2. Print startup text and ready prompt
3. Receive and echo one byte
4. Accumulate up to 31 command characters
5. On CR/LF: dispatch command or report error
6. Reset buffer state; repeat
```

## 13. Flowchart

Flowchart PNG upload pending. Refer to the algorithm and pseudocode above.

## 14. Expected Output

Example interaction: send `LED ON` followed by Enter; receive `LED turned ON`. Other replies include `LED turned OFF`, `LED toggled`, `Unknown command`, and `Command too long`.

## 15. Actual Observations

No new physical test was performed for this update. The behavior described here is inferred from the uploaded source. Add measured readings, terminal captures and board-specific observations after running the project.

## 16. Troubleshooting

1. Check board power, shared GND and the exact MCU port names before interpreting a missing output.
2. Open the matching project and rebuild; do not assume stored build artifacts reflect the current source.
3. For garbled serial data, match the documented baud rate and 8N1 settings, cross TX to RX, and check voltage compatibility.

## 17. Limitations and Implementation Notes

- Commands are case-sensitive and must match the spaces shown: LED ON, LED OFF, LED TOGGLE.
- The 32-byte buffer holds at most 31 characters plus the terminator. Once overflow is detected, the line is rejected at its terminator.
- Disable terminal local echo because the firmware already echoes typed characters.

## 18. Result and Verification

Reviewed against the uploaded `Core/Src/main.c`, `Core/Inc/main.h`, `Core/Src/stm32f1xx_hal_msp.c` and `BluePill_UART1.ioc`. This documentation update does not claim a new Keil build, hardware run, measured waveform or successful network connection.

The uploaded program implements the behavior described above. Hardware success remains to be confirmed against the expected output.

## 19. Viva Questions with Answers

1. **Why reserve a byte in the command buffer?**  
   To append the null terminator needed for C string comparisons.

2. **Why ignore empty lines?**  
   It prevents a CRLF pair from being interpreted as two commands.

3. **What does GPIO_PIN_RESET do here?**  
   It drives PC13 LOW, which turns on the active-LOW LED.

## 20. File Descriptions

| File / folder | Purpose |
|---|---|
| [Core/Src/main.c](Core/Src/main.c) | Main application and peripheral initialization |
| [Core/Inc/main.h](Core/Inc/main.h) | Public declarations and GPIO macros |
| `Core/Src/*hal_msp.c` | Peripheral clocks and alternate-function GPIO setup |
| [BluePill_UART1.ioc](BluePill_UART1.ioc) | STM32CubeMX configuration |
| [MDK-ARM/BluePill_UART1.uvprojx](MDK-ARM/BluePill_UART1.uvprojx) | Keil project |
| `Drivers/` | Included HAL and CMSIS dependencies |

[Return to task overview](../README.md)
