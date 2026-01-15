# Integration Tests
This document describes integration test procedures and results for the MiraTherm radiator thermostat project.

## General prerequisites

All tests require the hardware setup shown below:
![Test Hardware Setup](../../electronics/diagrams/mt-rt-sw-dev-hw-block-diagram.svg)

Follow the software project README to set up the development environment: https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/README.md

You can view the MCU pin configuration by opening the [.ioc file](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/mt-rt.ioc) in STM32CubeMX.

## Test procedures

|ID|Name|REQ|Prerequisites|Test procedure|Expected behavior|
|--|----|---|-------------|--------------|-----------------|
|1|Display Driver and Measurements Test|1.x, 5.x, 6.x, 8.x|The MCU software is compiled with the `TEST` and `DRIVER_TEST` flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h).|Start the hardware and observe the display.|The display should show the following output: ![Display output](./img/photo_2026-01-15_10-52-52.jpg)<br> The values of `RE` (rotary encoder), `M` (motor current), and `B%` (battery SoC) should match the screenshot exactly. `BV` may vary slightly. `T` may vary due to ambient temperature and measurement error.|
|2| Rotary Encoder Driver Test | 2.x | The MCU software is compiled with the `TEST` and `DRIVER_TEST` flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h). Test 1 was passed.|Rotate the encoder one tick to the left, then two ticks to the right.|The displayed `RE:0` should change in the following order: 0 → -1 → 0 → 1.|
|3| Buttons Driver Test | 3.x | The MCU software is compiled with the `TEST` and `DRIVER_TEST` flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h). Test 2 was passed.|Press, briefly hold, and release each button (left: `Mode`, middle: `Go`, right: `Menu`).|While a button is held, the main color should change to white and return to black on release. This applies to each button.|
|4| Motor Driver Test | 4.x, 5.x | The MCU software is compiled with the `TEST` and `DRIVER_TEST` flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h). Test 3 was passed.|1. Press and hold the `Go` button until the valve reaches its end stop.<br>2. Press and release `Mode` once.<br>3. Repeat step 1 in the opposite direction.|1. On `Go` button press, the motor should begin moving the valve. `M` should be greater than zero — up to ~50 mA at start, around ~25 mA during movement, and 100–130 mA at the end stop.<br>2. The label of the `Go` button should toggle between `Go: F` and `Go: R`.|


## Test results

|ID|Name|Commit Hash|Status|Comments|
|--|-----------|-----------|------|--------|
|1|Display Driver and Measurements Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|2|Rotary Encoder Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|3|Buttons Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|4|Motor Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||