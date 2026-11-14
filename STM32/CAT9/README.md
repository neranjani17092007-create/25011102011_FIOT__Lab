# Continuous Lab Evaluation 9: UART Command-Controlled LED

## 1. Overview

Receives newline-terminated commands over USART1, echoes characters, and controls the active-LOW PC13 LED. A bounded 32-byte command buffer handles overlong input.

## 2. Objective

- Initialize HSI, GPIO and USART1 at 9600.
- Print startup text and ready prompt.
- Receive and echo one byte.

## 3. Hardware and Software Used

- STM32F103C8T6 / Blue Pill
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- USB-UART adapter with 3.3 V logic and serial terminal
- STM32CubeMX and Keil MDK-ARM; see the [detailed report](project_files/README.md) for build setup.

## 4. Main Peripherals Used

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PA9 | USART1 TX | 3.3 V USB-UART RX | Echo and replies, 9600 8N1 |
| PA10 | USART1 RX | 3.3 V USB-UART TX | Commands, 9600 8N1 |
| PC13 | GPIO output | Active-LOW LED | LOW=ON; HIGH=OFF |
| PA13 / PA14 | SWD | ST-LINK | Programming |
| GND | Ground | All connected devices | Common logic reference |


## 5. Summary of Actual Implementation

At startup the program prints HELLO and WORLD, waits 3 seconds, and prints the command prompt. Characters are echoed and buffered until CR or LF. LED ON writes PC13 LOW; LED OFF writes HIGH; LED TOGGLE toggles. Unknown and overlong commands receive explicit replies. Empty terminators, including LF following CR, are ignored.

## 6. Circuit and Pin Reference

Circuit PNG upload pending. Use the verified pin table above.

System clock: Internal HSI 8 MHz; PLL disabled; AHB/APB1/APB2 undivided.

## 7. Links to Artifacts and Detailed Documentation

- [Detailed Project Engineering Report](project_files/README.md)
- [Primary Application Source](project_files/Core/Src/main.c)
- [STM32CubeMX Configuration](project_files/BluePill_UART1.ioc)
- [Keil Project](project_files/MDK-ARM/BluePill_UART1.uvprojx)

## 8. Key Observations

- Commands are case-sensitive and must match the spaces shown: LED ON, LED OFF, LED TOGGLE.
- The 32-byte buffer holds at most 31 characters plus the terminator. Once overflow is detected, the line is rejected at its terminator.
- Disable terminal local echo because the firmware already echoes typed characters.

## 9. Learning Outcomes

- Explain the pin assignments and the program sequence from the actual source.
- Distinguish configured peripherals from those used in the application loop.
- Compare physical output with the [expected behavior](project_files/README.md#14-expected-output).

## 10. Verification Note

Source and configuration were inspected; the firmware was not rebuilt or tested on a board during this documentation update. Diagrams describe code-based connections and explicitly noted hardware assumptions.

## 11. Conclusion

Example interaction: send `LED ON` followed by Enter; receive `LED turned ON`. Other replies include `LED turned OFF`, `LED toggled`, `Unknown command`, and `Command too long`.
