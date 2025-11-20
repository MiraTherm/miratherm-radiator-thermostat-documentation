# MiraTherm Radiator Thermostat Software Architecture
This document describes the software architecture for the MiraTherm Radiator Thermostat. The software will be written in C and utilizes FreeRTOS for real-time task scheduling and resource management.

The architecture follows a modular design separating hardware abstraction, core control logic, and user interface (GUI) management. The structure aims to enable future integration of control algorithms and the Matter-over-Thread standard with minimal effort.

## Architecture Diagram
![Architecture](./../diagrams/mt-rt-sw-architecture.png)

## System Components

### Application Layer (Tasks)
The software is divided into specific FreeRTOS tasks, each responsible for distinct domain logic:

**SystemTask**: Acts as the central coordinator. It manages the high-level state machine, coordinates transitions between operational modes, and assigns target temperatures. Therefore, it communicates with other tasks using events (e.g., `EVT_ADAPT_REQ`, `EVT_ADAPT_END`, `ENT_INST_REQ`), reads configurations from `ConfigMutex`, and writes current system state to `SystemStateMutex`.

**InputTask**: Interprets hardware signals from the rotary encoder and push buttons, converting interrupts and GPIO states into logical input events (e.g., `EVT_MENU_BTN`, `EVT_CTRL_WHEEL_DELTA`) and sends them to the `ViewPresenterTask`.

**ViewPresenterTask**: Manages the User Interface. It utilizes the LVGL library for rendering, handles routing of manual commands from the `InputTask`, and presents the system state to the user. Therefore, it writes to `ConfigMutex`, reads from `SystemStateMutex`, and sends command events to `SystemTask`.

**SensorTask**: Periodically reads raw data from hardware sensors (e.g., temperature), converts them to internally used units (e.g., raw data to °C in float), and updates the shared `SystemStateMutex`.

**ControlTask**: Executes the control loop and receives related values (e.g., current temperature and target temperature) from `SystemStateMutex`. The calculation of the temperature setpoint is currently mocked as a placeholder for future control algorithms integration.

**MaintenanceTask**: Handles command events from `SystemTask` for non-standard operations such as valve adaptation (calibration) and descaling routines. Moreover, it sets fully opened (`EVT_OPEN_STATE` 100%) or fully closed (`EVT_CLOSED_STATE` 0%) positions. Therefore, it interacts with the `ControlTask` to stop the control loop and use `ValveFacade` to move the valve to specific positions as part of these procedures.

**StorageTask**: Manages non-volatile memory interaction, ensuring configuration data is saved to the internal Flash via an EEPROM emulation layer. It clears saved data upon receiving `EVT_CFG_RST_REQ` during factory reset routine.

### Synchronization & Data Structures
Thread safety is maintained using Event Queues and Mutexes for shared data structures:

**SystemStateMutex**: Holds volatile runtime data (Current Temperature, Target Temperature, Battery SoC, Current System Mode etc.).

**ConfigMutex**: Holds configuration settings.

**SleepLock**: A mechanism utilizing FreeRTOS pre-sleep processing to prevent the STM32WB55 from entering deep sleep while critical tasks (like motor movement or UI interaction) are active. It manages a variable containing bitflags `*_LOCK` that represent whether specific operations are ongoing, ensuring the system remains responsive during these periods.

### Hardware Abstraction Layer (HAL)
**Display**: Driver for SH1106 OLED display (128×64) via I²C.

**MotorDriver**: Driver for a DRV8833 module. Handles states: Coast, Forward, Backward, Brake.

#### Sensors
**TemperatureSensor**: Abstraction for temperature sensor integrated into the STM32WB55 microcontroller.

**SoCSensor**: ADC driver for measuring power supply voltage to calculate battery charge level.

**MotorCurrentSensor**: Shunt resistor ADC driver for measuring motor current consumption during valve operation.

#### Inputs
**RotaryEncoder**: Quadrature decoder driver for control wheel input.

**Buttons**: Interrupt-based driver for Mode, Centered, and Menu buttons.

#### ValveFacade 
**ValveFacade** is high-level abstraction layer sitting on top of the `MotorDriver` and `MotorCurrentSensor`, allowing tasks to set valve positions (0-100%) rather than managing raw motor inputs.