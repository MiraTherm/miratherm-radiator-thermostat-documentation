# MiraTherm Radiator Thermostat Software Design
This document describes the software design for the MiraTherm Radiator Thermostat. It includes the overall architecture, task structure, synchronization mechanisms, and description of used design patterns. The software is written in C and utilizes FreeRTOS for real-time task scheduling and resource management.

The architecture follows a modular design that separates hardware abstraction, core control logic, and user interface (UI) management. The structure aims to enable future integration of control algorithms and the Matter-over-Thread standard with minimal effort.

## Architecture Diagram
![Architecture](./../diagrams/mt-rt-sw-architecture.svg)

## Architecture Components

The software is divided into specific FreeRTOS tasks, each responsible for distinct domain logic. These tasks are typically started at system boot in the `main` function and then run indefinitely. There is only one instance of each task, following the singleton design pattern.

### Synchronization & Data Structures
Tasks communicate using event queues and mutex-protected structures to ensure thread safety. These synchronization mechanisms are created and passed to tasks as parameters before the scheduler is started. This prevents deadlocks and priority inversion problems and enables flexible priority adjustments.

#### Event Queues

To ensure the correct order of program flow, tasks communicate using event queues for signaling and synchronizing state changes (e.g., using requests). Each event queue is intended for one-way communication between two tasks. For bidirectional communication, two event queues are used.

#### Model Structures

Mutexes protect shared data between tasks, and each mutex is wrapped in a `Model` structure together with the data it protects:

- **SensorModel**: Holds the latest sensor readings (temperature, SoC, motor current).

- **ConfigModel**: Holds configuration settings. It is automatically saved to non-volatile memory by `StorageTask` on every update.

- **SystemModel**: Holds volatile runtime data (system state, current operational mode, target temperature, current time slot, etc.).

### Application Layer (Tasks)

- **SystemTask**: Acts as the central coordinator. It manages the high-level state machine, coordinates transitions between operational modes, and assigns the target temperature. It communicates with other tasks using events (e.g., `EVT_SYS_INIT_END`, `EVT_ADAPT_REQ`, `EVT_ADAPT_END`), reads configuration from `ConfigModel`, and manages `SystemModel`.

- **InputTask**: Interprets hardware signals from `RotaryEncoder` and `Buttons`, converting interrupts and GPIO states into logical input events (e.g., `EVT_MENU_BTN`, `EVT_CTRL_WHEEL_DELTA`) and sends them to the `ViewPresenterTask`.

- **ViewPresenterTask**: Manages the User Interface using the Routed MVP pattern (detailed below). Receives input events from `InputTask`, accesses shared Models (`SensorModel`, `ConfigModel`, `SystemModel`), and renders UI using LVGL. Communicates with `SystemTask` primariy through direct readings/writings of `SystemModel`.

- **SensorTask**: Periodically reads raw data from hardware sensors (e.g., temperature), converts it to internal units (e.g., raw data to °C as float), and updates the shared `SensorModel`.

- **StorageTask**: Manages non-volatile memory interaction, ensuring configuration data is saved to the internal Flash via an EEPROM emulation layer. It communicates with `SystemTask` using events to ensure sequential program flow.

- **MaintenanceTask**: Handles command events from `SystemTask` for non-standard operations such as valve adaptation (calibration) and descaling routines. The task is currently mocked and implements only a mocked valve adaptation routine.

- **ControlTask**: Executes the control loop. The task is currently not implemented.

### Hardware Abstraction Layer (HAL)

- **LVGLDisplayPort**: Port of SH1106-based 128x64 display for LVGL library. It can be reimplemented in order to replace the display with another one.

- **Motor**: Software abstraction for DRV8833 or similar motor drivers. Handles states: Coast, Forward, Backward, Brake.

- **RotaryEncoder**: Quadrature decoder driver for control wheel input.

- **Buttons**: Interrupt-based driver for Left, Middle and Right buttons.

### Sensors (ADC DMA)
ADC and internal temperature sensor measurements have no own abstraction layers. Due to usage of DMA channels of a single ADC all measurements and calculations are executed in `SensorTask` for performance optimization and code clarity.

## Routed Model View Presenter Pattern

The User Interface implementation follows a customized version of the Model View Presenter (MVP) design pattern to separate concerns and improve extensibility and maintainability.

This implementation introduces a Router component for centralized navigation and input event forwarding, which is why we refer to it as "Routed MVP" (R-MVP). The key adaptations from classical MVP are partial state-driven navigation (instead of user-driven) and hierarchical event delegation through the router.

### R-MVP Components

The whole UI logic is encapsulated within the `ViewPresenterTask`, which consists of the following components:

- **Router**: Manages screen navigation and page transitions based on system state. Each page is represented by one or more presenters and their corresponding views. The router:
    - reads `SystemModel` to fetch `SystemState_t`
    - initializes/deinitializes presenters
    - forwards input events from `InputTask` to the active presenter
    - sends events via `vp2system_event_queue` to initiate state transitions if needed.

- **Presenters**: Handle presentation logic and user interactions. They initialize views or nested presenters, receive input events from the router, read/write Models, and instruct views to update display elements.

- **Views**: Pure rendering components using LVGL. Views are responsible for updating/rendering the graphical elements on the display.

- **Models**: Shared thread-safe data structures (`SensorModel`, `ConfigModel`, `SystemModel`) passed to `ViewPresenterTask` as parameters. HAL functions (e.g., RTC) are also accessed as models.

Presenters or views can be nested to implement wizard workflows.

> ℹ️ Note: For simplicity and to reduce abstraction overhead, no interfaces are used in this implementation. Presenters call views directly. Since all page content created differs significantly from each other, there is no risk of code duplication. Interfaces (e.g., `I*View_t`) can be selectively introduced in the future if necessary.

### R-MVP Data Flow

The data flow within the R-MVP pattern is as follows:

1. **Input Events**: `InputTask` generates input events and sends them to the `Router` via the input event queue.
2. **Router**: The router receives input events and forwards them to the currently active `Presenter`.
3. **Presenter Logic**: The active presenter processes the input event and reads/writes the shared `Models` (with mutex protection).
4. **View Update**: The presenter instructs the corresponding `View` to render the updated data.

### Differences from Classical MVP

Reference: [Mike Potel's MVP paper](https://bit.ly/dF4gNn).

Key deviations from classical MVP:

1. **Centralized Router**: Instead of presenters directly managing navigation between views, a router component is introduced to centralize page navigation logic and input event forwarding. The router makes routing decisions based on current `SystemState_t` (read from `SystemModel`), enabling partial state-driven navigation rather than user-driven navigation (as in classical MVP).
2. **Hierarchical Structure**: Since input events do not come from views, presenters are considered to be on a higher hierarchy level than views. Presenters are initialized and deinitialized by the router based on the currently active page. Nested presenters are initialized by their parent presenters. Finally, views are initialized by their corresponding presenters.
3. **Separated Input Handling**: Input events flow through the router to the active presenter, rather than being handled by views directly (as in classical MVP). This enables more flexible input handling independent of UI framework callbacks.

### Interaction between R-MVP and `SystemTask`

As described before, the `ViewPresenterTask` implements the R-MVP pattern and interacts with the `SystemTask` in two ways:
- using events when one of the tasks needs to notify the other about a specific occurrence,
- reading and writing `SystemModel` for system-state-dependent routing and displaying or adjusting system variables.

The interaction points with the `ViewPresenterTask` can be clearly identified in the diagram of the `SystemTask` state machine:

![SystemTask State Machine](./../diagrams/mt-rt-sw-system-task-state-machine.svg)

For example, during `STATE_INIT`, `ViewPresenterTask` waits for `EVT_SYS_INIT_END` event from `SystemTask` before rendering. After that, `SystemTask` waits for the end of the COD (configuration on device) signaled by `EVT_COD_END` from `ViewPresenterTask`.

It is noticeable that `ViewPresenterTask` does not receive any events besides `EVT_SYS_INIT_END`. This is because after initialization, the router of `ViewPresenterTask` reads `SystemState_t` from `SystemModel` directly to determine corresponding pages to be rendered. (Assignment of multiple pages to multiple system states is also possible.)