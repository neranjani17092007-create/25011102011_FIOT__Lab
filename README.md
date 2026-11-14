# 25011102011_FIOT__Lab
Foundations of IoT — Continuous Assessment Tasks (CATs)
This repository contains the firmware implementations, hardware configurations, and compiled deliverables for the Foundations of IoT laboratory evaluations.

<img width="533" height="434" alt="image" src="https://github.com/user-attachments/assets/1fa275f3-7364-431d-b2f1-3d2e25677935" />


8051 Microcontroller (Atmel AT89S52)
All 8051 tasks are implemented in bare-metal Embedded C with direct register-level access (without third-party libraries or RTOS):

Evaluation	Topic / Experiment	Description	Deliverables

CAT2-Hardware Bring-Up & LED Blink	Basic GPIO output register access and software delay loop toggling LEDs at ~1 Hz.	main.c, STARTUP.A51, .uvproj, .hex

CAT3-Timer & Button State Controller	Multi-mode state machine (Running LED, Binary Counter, Alternating, Flashing) with hardware debouncing.	main.c, STARTUP.A51, .uvproj, .hex

CAT4-Matrix Keypad, 7-Segment & LEDs	4×4 keypad input scanning with corresponding 7-segment numerical display and multi-mode LED actuation.	main.c, STARTUP.A51, .uvproj, .hex

CAT5-PWM, Waveforms & Stepper Motor	Hardware timer-driven non-blocking LED fading (PWM), variable-frequency square waves, pulse trains, and stepper sequencing.	Task folders, .c, STARTUP.A51, .uvproj, .hex

CAT6-UART Serial Communications	Hardware UART serial string transmission at 9600 baud and full-duplex interactive echo terminal.	TASK_A/, TASK_B/, .c, STARTUP.A51, .uvproj, .hex

CAT7-16×2 LCD Driver & Keypad Movement	HD44780 4-bit LCD driver with string/number formatting and keypad-controlled cursor/character movement.	lcd_driver_and_keypad_movement.c, STARTUP.A51, .uvproj, .hex


STM32 Microcontrollers (STM32F103 Blue Pill & STM32F401RE Nucleo)
All STM32 projects are configured via STM32CubeMX and built with ARM Keil MDK (using STM32 HAL & CMSIS drivers):

Evaluation	Topic / Experiment	Target Hardware	Deliverables

CAT8-GPIO Input/Output & Switch LED	STM32F103 (Blue Pill)	Switch bounce handling, short-press and long-press event classification.

CAT9-UART Command-Controlled LED	STM32F103 (Blue Pill)	Hardware UART receiver decoding commands to control peripheral state.

CAT10-Wemos D1 Wi-Fi Modem Interface	STM32F103 + ESP8266	Serial AT command communication with Wemos D1 module.

CAT11-HTTP Web Server via Wemos AT	STM32F103 + ESP8266	TCP server on port 80, parsing +IPD, and serving static HTML page.

CAT12-GPIO & User Input Programming	STM32F401RE (Nucleo)	Hardware clock bring-up, user push-button, and onboard LED control.

CAT13-BME280 Sensor Interfacing via I²C	STM32F401RE (Nucleo)	Bosch BME280 sensor driver with compensated temperature, pressure & humidity reading.

CAT14-BME280 & ESP8266 MQTT Telemetry	STM32F401RE + ESP8266	Custom ESP8266 AT API publishing BME280 telemetry over MQTT.

CAT15-ToF Distance Sensing & MQTT Dashboard	STM32F401RE (Nucleo) + ToF Sensor + Wemos D1 Mini (ESP8266)	ToF distance measurement, UART communication between STM32 and ESP8266, MQTT telemetry, and real-time Node-RED dashboard visualization.
