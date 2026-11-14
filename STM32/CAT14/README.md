# Continuous Lab Evaluation 14: BME280 MQTT Telemetry through ESP8266 AT Firmware

## 1. Overview

Reads BME280 data, builds JSON on the STM32, and uses ESPLib to send MQTT packets through an ESP8266 AT modem over USART2.

## 2. Objective

- Initialize I2C1 and USART2 at 115200.
- Probe and configure BME280.
- Initialize AT modem and join configured Wi-Fi.

## 3. Hardware and Software Used

- STM32F401RET6 / NUCLEO-F401RE
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- BME280 breakout with compatible 3.3 V logic; I2C pull-ups provided by module or external circuit
- ESP8266/Wemos board with a stable supply appropriate to its power input
- STM32CubeMX and Keil MDK-ARM; see the [detailed report](project_files/README.md) for build setup.

## 4. Main Peripherals Used

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PB6 / PB7 | I2C1 SCL / SDA | BME280 | 100 kHz; 0x76 or fallback 0x77 |
| PA2 | USART2 TX | ESP8266 RX | 115200 8N1 commands and MQTT packet data |
| PA3 | USART2 RX | ESP8266 TX | AT responses |
| PA5 / PC13 | LED / button configuration | Onboard peripherals | Not used by telemetry loop |
| GND | Ground | All connected devices | Common logic reference |


## 5. Summary of Actual Implementation

Probe both BME280 addresses and choose 0x77 only when 0x76 fails and 0x77 responds. Initialize and configure the sensor, stopping in Error_Handler on setup failure. Initialize the ESP8266 in station mode, join the configured network, open TCP to broker.emqx.io:1883 and send MQTT CONNECT with client ID STM32_Publisher, clean session and keepalive 500. Each loop retries the sensor read, formats bounded JSON, publishes to stm32/kh/data with QoS0 on successful formatting, and delays 1000 ms.

## 6. Circuit and Pin Reference

Circuit PNG upload pending. Use the verified pin table above.

System clock: HSI-derived 84 MHz PLL clock; APB1 42 MHz and APB2 84 MHz.

## 7. Links to Artifacts and Detailed Documentation

- [Detailed Project Engineering Report](project_files/README.md)
- [Primary Application Source](project_files/Core/Src/main.c)
- [STM32CubeMX Configuration](project_files/NUCLEO_MQTT_1.ioc)
- [Keil Project](project_files/MDK-ARM/NUCLEO_MQTT_1.uvprojx)

## 8. Key Observations

- USART2 uses PA2/PA3, which may also be connected to the Nucleo ST-LINK virtual COM interface. Check board solder-bridge routing to avoid multiple TX drivers on PA3.
- The main loop does not call MQTTTimer() or MQTT_pingReq(), and does not implement an explicit Wi-Fi/broker reconnection state machine.
- The custom library encodes MQTT remaining length in one byte; large payloads are unsupported. Do not assume arbitrary JSON size is safe.
- Network credentials are intentionally omitted from documentation. They must be configured locally in the firmware.

## 9. Learning Outcomes

- Explain the pin assignments and the program sequence from the actual source.
- Distinguish configured peripherals from those used in the application loop.
- Compare physical output with the [expected behavior](project_files/README.md#14-expected-output).

## 10. Verification Note

Source and configuration were inspected; the firmware was not rebuilt or tested on a board during this documentation update. Diagrams describe code-based connections and explicitly noted hardware assumptions.

## 11. Conclusion

On successful connectivity, a subscriber to `stm32/kh/data` receives JSON with keys `temperature`, `pressure`, and `humidity`. Pressure is hPa; the other units are degrees C and percent RH. No measured sample values are asserted.
