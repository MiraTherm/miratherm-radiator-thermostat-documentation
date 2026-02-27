# MiraTherm radiator thermostat documentation

This repository contains project documentation including specifications for the MiraTherm Radiator Thermostat — an MCU-based radiator thermostat prototype. The software implements basic consumer functions and could be used as a base for research, development and production of smart heating controllers or thermostats.

The MiraTherm Radiator Thermostat is an interdisciplinary development that includes the following areas: mechanics, electronics, software, and control algorithms.

## Related Repositories

| Repository | Description |
|---|---|
| [miratherm-radiator-thermostat-mechanics](https://github.com/MiraTherm/miratherm-radiator-thermostat-mechanics) | Development of the thermostat's power transmission mechanism for proper function with commonly used radiator valves, followed by the design of an enclosure. |
| [miratherm-radiator-thermostat-electronics](https://github.com/MiraTherm/miratherm-radiator-thermostat-electronics) | Development of the thermostat's PCB and its integration with mechanical components. |
| [miratherm-radiator-thermostat-software](https://github.com/MiraTherm/miratherm-radiator-thermostat-software) | Development of the thermostat's software and its integration with PCB components. |

## Documents

### Software

| Document | Type | Description |
|---|---|---|
| [MP1-Exposé](software/MP1-Exposé%20-%20Radiator%20Thermostat%20Software/README.md) | Exposé | Initial project proposal for software development of a smart radiator thermostat. |
| [MP1-Specification](software/MP1-Specification%20-%20Radiator%20Thermostat%20Software/README.md) | Requirement Specification | Defines functional and non-functional requirements for the MCU-based thermostat software. |
| [MP1-Paper](software/MP1-Paper%20-%20Radiator%20Thermostat%20Software/README.md) | Master Project 1 Paper | IEEE-style paper presenting the technical design of the Routed MVP (R-MVP) pattern for decoupling UI logic in the thermostat software. |
| [Software Design](software/design/README.md) | Design Document | Describes the overall software architecture, FreeRTOS task structure, synchronization mechanisms, and design patterns. |
| [Verification](software/verification/README.md) | Verification | Driver and integration test scenarios with results, and inspection reports for verifying requirements. |
| [Diagrams](software/diagrams/) | Diagrams | Source files (DrawIO) and exports (SVG, PNG, PDF) for software architecture and design diagrams. |

## License

This documentation is licensed under the CC-BY-4.0 license. See the [LICENSE](LICENSE) file for details.

Copyright (c) 2025 MiraTherm.
