# Continuous Lab Evaluation 10: STM32-Wemos AT Command Test

## 1. Overview

Sends one AT command to an ESP8266 running AT firmware, then forwards each received modem byte to a PC serial monitor. This is a connectivity test, not a Wi-Fi join, MQTT client or HTTP server.

## 2. Objective

- Initialize USART1 and USART2 at 115200.
- Wait 2 seconds; clear USART2 overrun flag.
- Print startup message to PC.

## 3. Hardware and Software Used

- STM32F103C8T6 / Blue Pill
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- ESP8266/Wemos board with a stable supply appropriate to its power input
- USB-UART adapter with 3.3 V logic and serial terminal
- STM32CubeMX and Keil MDK-ARM; see the [detailed report](project_files/README.md) for build setup.

## 4. Main Peripherals Used

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PA2 | USART2 TX | ESP8266 RX | AT command, 115200 8N1 |
| PA3 | USART2 RX | ESP8266 TX | Modem response, 115200 8N1 |
| PA9 | USART1 TX | USB-UART adapter RX | PC monitor, 115200 8N1 |
| PA10 | USART1 RX | Adapter TX (optional) | Configured; not read by application |
| GND | Ground | All connected devices | Common logic reference |


## 5. Summary of Actual Implementation

After a 2-second wait the code clears the USART2 overrun flag, prints a startup message on USART1, sends AT followed by CRLF on USART2, and prints the same startup message again. The main loop blocks waiting for one USART2 byte and forwards it through USART1.

## 6. Circuit and Pin Reference

Circuit PNG upload pending. Use the verified pin table above.

System clock: Internal HSI 8 MHz; PLL disabled.

## 7. Links to Artifacts and Detailed Documentation

- [Detailed Project Engineering Report](project_files/README.md)
- [Primary Application Source](project_files/Core/Src/main.c)
- [STM32CubeMX Configuration](project_files/BluePill_Wifi_esp8266.ioc)
- [Keil Project](project_files/MDK-ARM/BluePill_Wifi_esp8266.uvprojx)

## 8. Key Observations

- The duplicate startup message is present in main.c; it is not evidence of two AT transmissions.
- There is no PC-to-Wemos forwarding loop. Typing in the PC terminal will not send arbitrary AT commands through this firmware.
- The blocking HAL_MAX_DELAY receive can wait indefinitely if the module stays silent.
- Use compatible AT firmware and a stable supply appropriate to the actual Wemos/module power input.

## 9. Learning Outcomes

- Explain the pin assignments and the program sequence from the actual source.
- Distinguish configured peripherals from those used in the application loop.
- Compare physical output with the [expected behavior](project_files/README.md#14-expected-output).

## 10. Verification Note

Source and configuration were inspected; the firmware was not rebuilt or tested on a board during this documentation update. Diagrams describe code-based connections and explicitly noted hardware assumptions.

## 11. Conclusion

The PC should show the startup banner twice. An AT-ready module at the matching baud rate is expected to respond with OK, with possible command echo depending on its configuration.
