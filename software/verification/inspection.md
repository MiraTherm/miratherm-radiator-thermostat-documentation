# Verification by Inspection

For some of the requirements verification by inspection was performed and documented in this section.

## Inspection Scenarios

|REQ|Name|Verification|Success Criteria|
|--|--|--|--|
|22| Automatic Summer/Winter Time Switching | Inspection of the source code in [set_date_time_presenter.c](https://github.com/MiraTherm/miratherm-radiator-thermostat-software/blob/main/Core/Src/Presenters/set_date_time_presenter.c) | Usage of native STM32 HAL RTC functions for date/time setting. |

## Inspection Results
|REQ|Name|Commit Hash|Status|Comments|
|--|--|--|--|--|
|22| Automatic Summer/Winter Time Switching |016659f10fa078166a2549f68c3efff718f45566|✅Passed| |