# Home automation

## General principle

The general principle is based on a centralised ECU (Raspbery Pi) with an optional coprpocessor (STM32) used to manage low speed IO.
It shall communicate with satellite modules dedicated to specific tasks (like input, output and so on).

The main ECU shall communicate with an high speed link (Ethernet / WiFi) to the gateway.
A communication link shall be established with the satellite modules even by wire (RS-485) or wireless (NRF24L01, 2.4 gHz link).

The system is based on a Home Assistant server able to centralize different variables (temperature, lightining, ...),
commands (switch light on or off, enable a swimming pool filter, ...) or video streams (IP camera).
This server allows the system administrator to manage the system with a local computer and
users to interact with it through terminals (smartphone based, iOS or Android).

![System global diagram overview](system_global_diagram_overview.drawio.svg)

Read more about the Rapberry pi motherboard [here](./01_raspberry_pi_motherboard/01_documentation/raspberry_pi_motherboard.md)

Read more about the satellite module [here](./02_module_satellite/01_documentation/module.md)
