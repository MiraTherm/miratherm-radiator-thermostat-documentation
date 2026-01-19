# Verification by Inspection

For some of the requirements verification by inspection was performed and documented in this section.

## Inspection Scenarios

|REQ|Name|Verification|Success Criteria|
|--|--|--|--|
|22| Automatic Summer/Winter Time Switching | Inspection of the source code in [set_date_time_presenter.c](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Src/Presenters/set_date_time_presenter.c) | Usage of native STM32 HAL RTC functions for date/time setting. |
|30| Hardware Component Replaceability | Inspection of hardware-related source code files in the [repository](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/tree/main) | Hardware-specific code is decoupled from the main application logic, allowing for minimal changes to the core codebase if a component is swapped. |

## Inspection Results
|REQ|Name|Commit Hash|Status|Comments|
|--|--|--|--|--|
|22| Automatic Summer/Winter Time Switching |016659f10fa078166a2549f68c3efff718f45566|✅Fully passed| |
|30| Hardware Component Replaceability |016659f10fa078166a2549f68c3efff718f45566|⚠️Partially passed| The abstraction layers are successfully implemented for:<br> - ✅[Buttons](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Drivers/buttons/) <br> - ✅[Rotary Encoder](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/tree/main/Drivers/rotary_encoder.c) <br> - ✅[Motor Driver](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/tree/main/Drivers/motor) <br>- ✅Display can be replaced by changing [lv_conf.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Drivers/lv_conf.h) and [lvgl_port_display.c](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Drivers/lvgl_port_display/lvgl_port_display.c), whereas its resolution should not be smaller than 128x64 for proper UI rendering. UI elements are placed using LVGL anchors relative to screen sections (center, top left, middle left, etc.). If further UI adjustments are needed, files in [/Core/Src/Views](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/tree/main/Core/Src/Views) can be changed.<br> ⚠️ ADC and internal temperature sensor measurements have no own abstraction layers. Due to usage of DMA channels of a single ADC all measurements and calculations are executed in [sensor_task.c](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Src/sensor_task.c) for performance optimization and code clarity.|
