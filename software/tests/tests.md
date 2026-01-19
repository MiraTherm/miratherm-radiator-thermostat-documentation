# Integration Tests
This document describes integration test scenarios and results for the MiraTherm radiator thermostat project.

## General prerequisites

All tests require the hardware setup shown below:
![Test Hardware Setup](../../electronics/diagrams/mt-rt-sw-dev-hw-block-diagram.svg)

Follow the software project README to set up the development environment: https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/README.md

You can view the MCU pin configuration by opening the [.ioc file](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/mt-rt.ioc) in STM32CubeMX.

Unless otherwise specified in the test scenarios, please ensure the following before starting each test: 
- If settings are stored in permanent memory, a factory reset should be performed before each test (`Menu` -> `Factory Reset`)

## Driver test scenarios

- For these test scenarios the MCU software must be compiled with the `TEST` and `DRIVER_TEST` flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h).

|ID|Name|REQ|Prerequisites|Test scenarios|Expected behavior|
|--|----|---|-------------|--------------|-----------------|
|1|Display Driver and Measurements Test|1.x, 5.x, 6.x, 8.x| - |Start the hardware and observe the display.|The display should show the following output: ![Driver test display output](./img/driver_test_screen.jpg)<br> The values of `RE` (rotary encoder), `M` (motor current), and `B%` (battery SoC) should match the screenshot exactly. `BV` may vary slightly. `T` may vary due to ambient temperature and measurement error.|
|2| Rotary Encoder Driver Test | 2.x | Test 1 was passed.|Rotate the encoder one tick to the left, then two ticks to the right.|The displayed `RE:0` should change in the following order: 0 → -1 → 0 → 1.|
|3| Buttons Driver Test | 3.x | Test 2 was passed.|Press, briefly hold, and release each button (left: `Mode`, middle: `Go`, right: `Menu`).|While a button is held, the main color should change to white and return to black on release. This applies to each button.|
|4| Motor Driver Test | 4.x, 5.x | Test 3 was passed.|1. Press and hold the `Go` button until the valve reaches its end stop.<br>2. Press and release `Mode` once.<br>3. Repeat step 1 in the opposite direction.|1. On `Go` button press, the motor should begin moving the valve. `M` should be greater than zero: $30-50 mA$ at start, $10-25 mA$ during movement, and $100–130 mA$ at the end stop.<br>2. The label of the `Go` button should toggle between `Go: F` and `Go: R`.|

### Driver test results

|ID|Name|Commit Hash|Status|Comments|
|--|-----------|-----------|------|--------|
|1|Display Driver and Measurements Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|2|Rotary Encoder Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|3|Buttons Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||
|4|Motor Driver Test|8051fed567254ebcf559fd5246106ed6269e8fe9|✅Passed||

## Integration test scenarios

- For these test scenarios the MCU software must be compiled without any flags set in [tests.h](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Inc/tests.h).

|ID|Name|REQ|Prerequisites|Test scenarios|Expected behavior|
|--|----|---|-------------|--------------|-----------------|
|5|Home Display Page| 14, 31 | - | Skip COD (Configuration on Device) routine. To do this, press the middle button 8 times after a device reset/restart. Wait 10 seconds until mocked adaptation is done. | The display should show the following output: ![Home screen](./img/home_screen.jpg)<br> The ambient temperature (value on the right side) may vary.|
|6|Target Temperature Range, Resolution and Valve States| 20 | - | 1. Same as in test 5.<br> 2. Spin the control wheel clockwise to change the displayed target temperature (value on the left side) to $29.5°$ and then spin one tick further. 2. Spin the control wheel counterclockwise to change the target temperature to $5.0°$, then spin one tick further. | 1. Same as in test 5<br> 2. The target temperature value should change to $29.5°$ in $0.5°$ steps and then to `ON`.<br> 3. The target temperature value should change to $5.0°$ and then to `OFF`. |
|7|Mode Switch| 14, 17, 18, 31 | - | 1. Same as in test 5.<br> 2. Switch to the Manual Mode using the left button. <br> 3. Set the target temperature or valve state in the Manual Mode to another value (e.g., `OFF`, see test 6, step 2). Then change back to the Auto Mode and back again to the Manual Mode.| 1. Same as in test 5.<br> 2. The left button hint should change from `Auto` to `Manual` signalizing the mode switch. The end time of the time slot (day time value on the right side) should disappear. <br> 3. After the target temperature in the Manual Mode has been changed, the switch to the Auto Mode should change the displayed target temperature value back to $20°C$ and the further switch to the Manual Mode should restore the temperature value or valve state set in the step 3.|
|8|COD: Date and Time Setup| 9, 10, 31 | Device reset. | 1. Use control wheel to set Year, Month, Day, Hour, Minute. Press Middle button to confirm each. (The set time should not be 12:0x for further verification)<br> 2. Select `On` for `Summer Time`, skip the following COD steps (press the middle button twice) and wait 10 seconds.| 1. Software requests date/time input. Button hint `O` for the middle button is always visible. Button hint `<` is not visible on the first field (year) of the date selection page. <br>  2. After the COD the home screen with the set time in the upper left corner appears.|
|9|COD: Daily Schedule Setup| 9, 10, 11, 12.x 13, 20 | Test 8 passed. Device reset. | 1. Leave the date and time setting as is. (Press the middle button 6 times.). <br> 2. Within 1 minute:<br> Edit the schedule. Check all options available for the number of time slots per day. Then select 3 time slots per day.<br> 3. Within 3 minutes:<br> Set the following values for the time slots: <br>  (1/3): 00:00-12:05 - $21°C$ <br> (2/3): 12:05-22:00 - $OFF (<5°C)$ <br> (3/3): 22:00-23:59 - $ON (>29.5°C)$  <br> 4. Start the installation, wait until the mocked adaptation has been performed, and then wait until 12:05–12:06 p.m.| 1. After the date and time setting you should see the question `Edit schedule?` <br>  2. Options 3, 4 or 5 should be available for the time slots number. <br>3. 00:00 should be automatically applied as start time of the 1/3 time slot, 23:59 for the end of the 3/3 time slot. The end time of a previous time slot should be automatically applied as the end of the following time slot. <br> 4. The user should be asked to begin the installation, accept it and wait until the home screen appears. At 12:06 the target temperature value (in the Auto Mode) should change from $21°C$ to $OFF$.|
|10|Boost-Mode| 17, 19.x, 31 | - | 1. Same as in test 5.<br> 2. Press the middle button.<br> 3. Press the middle button again.<br> 4. Press the middle button again and wait until the countdown ends. | 1. Same as in test 5.<br> 2. You should see the text `Boost Mode` with a countdown from 300 seconds and `X` as hint for the middle button.<br> 3. You should change back to the home screen.<br> 4. You should see the Boost Mode screen again and switch to the home screen when the countdown ends. |
|11|Temperature Offset| 15, 21, 23, 31 | - | 1. Same as in test 5.<br> 2. Press the right button to enter the menu, then scroll to `Temp offset` and press the middle button.<br> 3. Spin the control wheel clockwise to change the value to $-15.0$. <br> 4. Spin the control wheel counterclockwise to change the value to $+15.0$. <br> 5. Press the middle button to save the setting, wait minimum 3 seconds, restart the device and repeat steps 1 and 2. | 1. Same as in test 5.<br> 2. You should see the selection `Temp Offset` $0.0°C$ ($0.0°C$ is the factory default value) and `O` as hint for the middle button.<br> 3. The temperature offset value should change to $-15.0°C$ in $0.5°C$ steps. <br> 4. The temperature offset value should change to $+15.0°C$.<br> 5. After a device restart, the initial temperature offset value in the `Temp offset` menu item should be $+15.0°C$ as it has been set before the device restart.|
|12|Factory Reset| 23, 24, 31 | The test should be performed directly after the last step in test 11.| 1. Use the left button to go back to the menu. <br> 2. Scroll to `Factory reset` and press the middle button. <br> 3. Press the middle button to choose `No`. <br> 4. Go to `Factory reset` screen again and choose `Yes`. <br> 5. Skip the COD (see test 5) and then go to the `Temp offset` screen (see test 11, step 2).| 1. After pressing the left button with the hint `<` the software should change back to the menu. <br> 2. The selection `Factory reset?` with `No` as the default option and `Yes` as the second option should be opened. <br> 3. The software should change back to the menu screen. <br> 4. The device should go to the COD after the factory reset. <br> 5. The factory default value of $0.0°C$ should be shown for the temperature offset.|

### Integration test results

|ID|Name|Commit Hash|Status|Comments|
|--|-----------|-----------|------|--------|