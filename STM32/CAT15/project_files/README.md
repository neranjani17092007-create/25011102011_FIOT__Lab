# Continuous Lab Evaluation 15: BME280 UART-to-MQTT Gateway

## 1. Experiment Title

BME280 UART-to-MQTT Gateway using STM32F401RET6 / NUCLEO-F401RE.

## 2. Aim

Combines the STM32 BME280 CSV sender with a Wemos Arduino sketch that validates each line, converts it into JSON, and publishes MQTT. The uploaded files implement environmental telemetry; they do not contain ToF distance acquisition.

## 3. Hardware Components

- STM32F401RET6 / NUCLEO-F401RE
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- BME280 breakout with compatible 3.3 V logic; I2C pull-ups provided by module or external circuit
- ESP8266/Wemos board with a stable supply appropriate to its power input

## 4. Software Tools

- STM32CubeMX: `NUCLEO_BME280_test.ioc`
- Keil MDK-ARM: `MDK-ARM/NUCLEO_BME280_test.uvprojx`
- STM32 HAL and CMSIS sources included in the project
- Arduino IDE with ESP8266 board support, PubSubClient and SoftwareSerial for the Wemos sketch

## 5. Theory

The BME280 driver converts raw sensor data using calibration parameters. I2C1 reads/writes registers through HAL adapter functions. Firmware treats temperature as degrees C, divides pressure in Pa by 100 for hPa, and uses relative humidity in percent. I2C pull-ups and compatible voltage levels are required. MQTT then transports an application payload to a named topic through a broker. CAT14 constructs packets on STM32 using an AT modem; CAT15 delegates JSON validation and MQTT to the Wemos sketch.

## 6. Actual Pin Configuration

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PB6 / PB7 | I2C1 SCL / SDA | BME280 | 100 kHz; selected address 0x76 |
| PA9 | USART1 TX | Wemos GPIO14 / D5 | 9600 baud receive-only SoftwareSerial |
| PA10 | USART1 RX | Unconnected in this one-way link | Configured but unused |
| PA2 / PA3 | USART2 TX / RX | PC serial interface | 115200 8N1 |
| GND | Common reference | STM32, sensor and Wemos GND | Required shared reference |
| GND | Ground | All connected devices | Common logic reference |

The connections are derived from the source and configuration, rather than a photographed lab assembly. Check board routing and module power pins before wiring. MCU port labels are not connector pin numbers.

## 7. Circuit Diagram

Circuit PNG upload pending. Use the pin table above.



## 8. CubeMX / Peripheral Configuration

- System clock: HSI-derived 84 MHz PLL clock; APB1 42 MHz and APB2 84 MHz.
- Configuration: [NUCLEO_BME280_test.ioc](NUCLEO_BME280_test.ioc).
- Peripheral assignments and operational rates are listed above; the actual initialization in `main.c` is the basis for runtime settings.

1. Open [NUCLEO_BME280_test.uvprojx](MDK-ARM/NUCLEO_BME280_test.uvprojx) with the matching STM32 device pack installed.
2. Keep `Core/`, `Drivers/`, `MDK-ARM/` and the IOC together. The reorganization preserves these internal relative paths.
3. Check the device and clock settings, build, and review compiler/linker diagnostics. Existing output files may predate the source.
4. Program the intended board using ST-LINK. Make connections from the pin table and confirm the stated logic levels.
5. Verify sensor address and compensation/float formatting support before interpreting readings.

## 9. Key APIs Used

- `bme280_get_sensor_data()`
- `HAL_UART_Transmit()`
- `SoftwareSerial()`
- `strtod()`
- `isfinite()`
- `PubSubClient::connect()`
- `PubSubClient::publish()`

## 10. Source Code Logic

The STM32 probes sensor addresses but selects 0x76, configures normal mode, and sends a greeting plus CSV about once per second on USART1 at 9600. CSV is also mirrored to USART2 at 115200. Wemos receives on GPIO14/D5, rejects the greeting and malformed lines, requires three finite values within the programmed ranges, and constructs JSON. It connects to test.mosquitto.org:1883, forms a chip-ID-based client/topic, services mqtt.loop(), and publishes the latest pending fresh sample. Wi-Fi and broker retries run at 15 s and 5 s intervals respectively.

Read [main.c](Core/Src/main.c) for the complete implementation.
- [Wemos MQTT sketch](Wemos_BME280_MQTT.ino)

## 11. Algorithm

1. STM32: initialize BME280 and both UARTs.
2. STM32: send greeting and valid CSV each loop.
3. Wemos: read complete line on D5 at 9600.
4. Validate 3 finite, in-range values; build JSON.
5. Reconnect as needed and service MQTT client.
6. Publish latest fresh sample; repeat.

## 12. Pseudocode

```text
1. STM32: initialize BME280 and both UARTs
2. STM32: send greeting and valid CSV each loop
3. Wemos: read complete line on D5 at 9600
4. Validate 3 finite, in-range values; build JSON
5. Reconnect as needed and service MQTT client
6. Publish latest fresh sample; repeat
```

## 13. Flowchart

Flowchart PNG upload pending. Refer to the algorithm and pseudocode above.

## 14. Expected Output

Wemos USB serial at 115200 prints `Subscribe to: stm32-demo/nucleo-bme280-<chipid>/readings`. Subscribe to that exact topic on the configured broker. JSON keys are `temperature_c`, `pressure_hpa`, and `humidity_percent`; successful publication prints `Published`.

## 15. Actual Observations

No new physical test was performed for this update. The behavior described here is inferred from the uploaded source. Add measured readings, terminal captures and board-specific observations after running the project.

## 16. Troubleshooting

1. Check board power, shared GND and the exact MCU port names before interpreting a missing output.
2. Open the matching project and rebuild; do not assume stored build artifacts reflect the current source.
3. For garbled serial data, match the documented baud rate and 8N1 settings, cross TX to RX, and check voltage compatibility.
4. For sensor failure, inspect check76/check77 and rslt, confirm PB6/PB7 wiring and pull-ups, and verify that the selected device address matches the sensor.
5. For absent MQTT output, check Wi-Fi, the broker/topic documented for this task, and whether the ESP8266 firmware is AT firmware (CAT14) or the supplied Arduino sketch (CAT15).

## 17. Limitations and Implementation Notes

- Use Wemos_BME280_MQTT.ino as Arduino firmware on the ESP8266. This differs from CAT14's AT-firmware approach.
- The receive-only SoftwareSerial pin is GPIO14 (D5 on D1 mini), not the Wemos hardware RX pin. No Wemos TX-to-PA10 wire is required.
- The STM32 driver remains fixed at 0x76 and can stop retrying reads after rslt becomes an error.
- Only the latest pending reading is kept; samples older than 5 seconds are dropped. MQTT reconnection clears pending data and resynchronizes serial input.
- The sketch contains configured Wi-Fi values; documentation omits them. No Node-RED flow or ToF driver is present in the uploaded task.

## 18. Result and Verification

Reviewed against the uploaded `Core/Src/main.c`, `Core/Inc/main.h`, `Core/Src/stm32f4xx_hal_msp.c` and `NUCLEO_BME280_test.ioc`. This documentation update does not claim a new Keil build, hardware run, measured waveform or successful network connection.

The uploaded program implements the behavior described above. Hardware success remains to be confirmed against the expected output.

## 19. Viva Questions with Answers

1. **Why is D5 used instead of hardware RX?**  
   The sketch explicitly constructs receive-only SoftwareSerial on GPIO14.

2. **How is the greeting handled?**  
   The strict three-value CSV parser rejects it.

3. **Where are the JSON and MQTT connection handled?**  
   In the Wemos Arduino sketch; the STM32 sends CSV.

## 20. File Descriptions

| File / folder | Purpose |
|---|---|
| [Core/Src/main.c](Core/Src/main.c) | Main application and peripheral initialization |
| [Core/Inc/main.h](Core/Inc/main.h) | Public declarations and GPIO macros |
| `Core/Src/*hal_msp.c` | Peripheral clocks and alternate-function GPIO setup |
| [NUCLEO_BME280_test.ioc](NUCLEO_BME280_test.ioc) | STM32CubeMX configuration |
| [MDK-ARM/NUCLEO_BME280_test.uvprojx](MDK-ARM/NUCLEO_BME280_test.uvprojx) | Keil project |
| `Drivers/` | Included HAL and CMSIS dependencies |
| [Wemos_BME280_MQTT.ino](Wemos_BME280_MQTT.ino) | ESP8266 serial parser and MQTT gateway |

[Return to task overview](../README.md)
