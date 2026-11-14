# Continuous Lab Evaluation 13: BME280 I2C Readout and UART Telemetry

## 1. Experiment Title

BME280 I2C Readout and UART Telemetry using STM32F401RET6 / NUCLEO-F401RE.

## 2. Aim

Uses the Bosch BME280 driver over I2C1 and sends temperature, pressure and humidity as comma-separated text. USART1 is used for an external receiver and USART2 for the PC.

## 3. Hardware Components

- STM32F401RET6 / NUCLEO-F401RE
- ST-LINK or onboard ST-LINK programmer
- USB cable and connections shown in the pin table
- BME280 breakout with compatible 3.3 V logic; I2C pull-ups provided by module or external circuit

## 4. Software Tools

- STM32CubeMX: `NUCLEO_BME280_test.ioc`
- Keil MDK-ARM: `MDK-ARM/NUCLEO_BME280_test.uvprojx`
- STM32 HAL and CMSIS sources included in the project

## 5. Theory

The BME280 driver converts raw sensor data using calibration parameters. I2C1 reads/writes registers through HAL adapter functions. Firmware treats temperature as degrees C, divides pressure in Pa by 100 for hPa, and uses relative humidity in percent. I2C pull-ups and compatible voltage levels are required.

## 6. Actual Pin Configuration

| Pin | Function | Connection | Purpose |
|---|---|---|---|
| PB6 | I2C1 SCL | BME280 SCL | 100 kHz clock |
| PB7 | I2C1 SDA | BME280 SDA | Bidirectional data |
| PA9 | USART1 TX | External receiver / Wemos RX | 9600 8N1 CSV and greeting |
| PA10 | USART1 RX | Receiver TX if needed | Configured; not read by loop |
| PA2 / PA3 | USART2 TX / RX | PC serial interface | 115200 8N1; CSV transmitted on PA2 |
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

- `HAL_I2C_IsDeviceReady()`
- `HAL_I2C_Mem_Read()`
- `HAL_I2C_Mem_Write()`
- `bme280_init()`
- `bme280_get_sensor_data()`
- `HAL_UART_Transmit()`

## 10. Source Code Logic

The code probes 0x76 and 0x77, but selects 0x76 for the driver. It configures 1× oversampling, filtering off, 1000 ms standby and normal mode. Each iteration delays 1000 ms, sends HELLO FROM NUCLEO on USART1, reads compensated data when rslt is OK, converts pressure to hPa, and formats temperature,pressure,humidity with two decimal places. Valid formatted data is sent to both UARTs.

Read [main.c](Core/Src/main.c) for the complete implementation.

## 11. Algorithm

1. Initialize I2C1, USART1 and USART2.
2. Probe 0x76/0x77; initialize sensor at 0x76.
3. Configure normal mode and 1x oversampling.
4. Delay 1000 ms; send greeting via USART1.
5. If status OK: read and format T,P,H CSV.
6. Transmit CSV to both UARTs; repeat.

## 12. Pseudocode

```text
1. Initialize I2C1, USART1 and USART2
2. Probe 0x76/0x77; initialize sensor at 0x76
3. Configure normal mode and 1x oversampling
4. Delay 1000 ms; send greeting via USART1
5. If status OK: read and format T,P,H CSV
6. Transmit CSV to both UARTs; repeat
```

## 13. Flowchart

Flowchart PNG upload pending. Refer to the algorithm and pseudocode above.

## 14. Expected Output

USART1 sends `HELLO FROM NUCLEO` and, on successful reads, a line shaped like `<temperature_C>,<pressure_hPa>,<humidity_percent>`. USART2 sends the CSV readings. Actual measurements require hardware testing.

## 15. Actual Observations

No new physical test was performed for this update. The behavior described here is inferred from the uploaded source. Add measured readings, terminal captures and board-specific observations after running the project.

## 16. Troubleshooting

1. Check board power, shared GND and the exact MCU port names before interpreting a missing output.
2. Open the matching project and rebuild; do not assume stored build artifacts reflect the current source.
3. For garbled serial data, match the documented baud rate and 8N1 settings, cross TX to RX, and check voltage compatibility.
4. For sensor failure, inspect check76/check77 and rslt, confirm PB6/PB7 wiring and pull-ups, and verify that the selected device address matches the sensor.

## 17. Limitations and Implementation Notes

- Address probing does not automatically select 0x77. The driver remains configured for 0x76.
- The IOC does not record the explicit 9600 baud override for USART1, while main.c does. Preserve/check 9600 if regenerating code.
- After a failed read, rslt remains nonzero and the outer condition skips subsequent reads; no recovery is implemented in this file.
- The CSV variable name mqttData does not mean this STM32 program publishes MQTT. No broker connection is present in this task.

## 18. Result and Verification

Reviewed against the uploaded `Core/Src/main.c`, `Core/Inc/main.h`, `Core/Src/stm32f4xx_hal_msp.c` and `NUCLEO_BME280_test.ioc`. This documentation update does not claim a new Keil build, hardware run, measured waveform or successful network connection.

The uploaded program implements the behavior described above. Hardware success remains to be confirmed against the expected output.

## 19. Viva Questions with Answers

1. **Why shift the I2C address left?**  
   The STM32 HAL memory-transfer functions expect the 7-bit address in the shifted address field.

2. **Why divide pressure by 100?**  
   The floating-point Bosch result is in Pa; division gives hPa.

3. **Does detecting 0x77 select it?**  
   No. This program still passes the primary 0x76 address to the driver.

## 20. File Descriptions

| File / folder | Purpose |
|---|---|
| [Core/Src/main.c](Core/Src/main.c) | Main application and peripheral initialization |
| [Core/Inc/main.h](Core/Inc/main.h) | Public declarations and GPIO macros |
| `Core/Src/*hal_msp.c` | Peripheral clocks and alternate-function GPIO setup |
| [NUCLEO_BME280_test.ioc](NUCLEO_BME280_test.ioc) | STM32CubeMX configuration |
| [MDK-ARM/NUCLEO_BME280_test.uvprojx](MDK-ARM/NUCLEO_BME280_test.uvprojx) | Keil project |
| `Drivers/` | Included HAL and CMSIS dependencies |

[Return to task overview](../README.md)
