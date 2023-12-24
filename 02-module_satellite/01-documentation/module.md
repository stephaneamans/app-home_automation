# 2 - Home automation satellite module


## 2.1 - Principle

The module is based around a microrontroller used to manage the communication (RF or wire) and all the inputs and outputs.
Additionaly it used to manage the power supply to optimize the power consumption of the module.
It compute all needed informations to perform the expected tasks.

When the module is used as an input it acquire informations (digital or analog) which are computed (filtering, debouncing, ...) by the microcontroller.
The data is then packed in a dedicated protocol to be sent to the master over the RF tranceiver.
The inputs are analog inputs used to monitor the module's input voltage, temperature sensor and light sensor.
They can also be digital input from the external world, these input are protected and can be polarised to work following R/2R methodology.

When the module is used as an output, it receive informations which are unpacked to extract relevant payload.
The payload is computed by the microcontroller to perform the expected task.
The output is then activated or unactiveted (actuator) to pilot any peripheral, they are decentralized on the extension board.

An optional external source clock can be used to make the RTC more accurate.

An optional serial wire debug connector is used to plug the debugger to flash and debug the microcontroler. Only in developement phase, not in production.

![Module global diagram overview](module_global_diagram_overview.drawio.svg)

The module shall be used either as an input module, output module or both input ans output.

The power module is used to supply needed voltage domain (3.3V), the power consumption optimizations are done by the microcontroller unit.


## 2.2 - Hardware architecture

### 2.2.1 - Main module

The hardware architecture is shown in the figure 2.2, declination of the module principle described in the section one.

The module is composed of two boards. The module board with the RF tranceiver, the MCU, some input interfaces the power supply and so on as shown in the figure below.
The extension board provide additional optional functionalities like, relay contact, light sensor, humidity sensor and so on.

![Module global diagram overview](module_hardware_architecture.drawio.svg)

#### NRF24L01 RF tranceiver submodule

This RF tranceiver is a 2.4 GHz multichannel transmit and receive module.
It can be found as a SOM (system on module) with an external antenna or a PCB printed antenna.

It is powered  with a +3.3V power supply, and uses an SPI interface to communicate with its master.

A chip enable (CE) pin is used to enable the module, during the TX and RX sequences it allows to reduce module consumption when it is not used.
This pin is driven low at start-up to avoid accidental module activation.

An interrupt pin is used to inform master when an event occurs (Rx, en of Tx and so on) it can be enabled or disabled by software.

Interfaces :

| Signal name  | Signal description    |
|--------------|-----------------------|
| SPI_SCK      | SPI clock             |
| SPI_MOSI     | SPI slave data in     |
| SPI_MISO     | SPI slave data out    |
| NRF24L01_CS  | SPI chip select       |
| NRF24L01_CE  | chip enable           |
| NRF24L01_IRQ | interrupt line        |
| +3.3V        | +3.3V MCU, power line |
| GND          | ground, power line    |


#### Clock submodule

This optional clock source can be used to enhance the precision of the RTC and microcontroller accuracy.

Interfaces :

| Signal name  | Signal description          |
|--------------|-----------------------------|
| RTC_CLK_IN   | clock input source for RTC  |
| RTC_CLK_OUT  | clock output source for RTC |


#### SWD serial wire debug submodule

This serial wire debug connector is a 4 wires connector.

It is used to plug a serial wire debug probe to perform microcontroller flash programming and provides numbers of debug features.

+3.3V power supply is used to indicate the debugger that the power supply is enabled.

Interfaces :

| Signal name  | Signal description             |
|--------------|--------------------------------|
| SWD_CLK      | module serial wire debug clock |
| SWD_IO       | serial wire debug data line    |
| +3.3V        | +3.3V MCU, power line          |
| GND          | Ground, power line             |


#### MAX487 tranceiver submodule

This is an RS485 transmissison through a dedicated physical interface from UART.

It is powered  with a +5V power supply, and is used to provide a wire communication mean.

Two pins are be used to enable or disable the driver RX and TX stages.
/MAX487_RE pin is driven high and MAX487_DE pin is driven low to avoid accidental driver activation at start-up.

Interfaces :

| Signal name  | Signal description     |
|--------------|------------------------|
| UART_TX      | UART transmission line |
| UART_RX      | UART reception line    |
| /MAX487_RE   | Driver RX enable       |
| MAX487_DE    | Driver TX enable       |
| +5V          | +5V power line         |
| GND          | ground, power line     |


#### Digital input submodule

This function shall convert any external binary signal to a safe 3.3V / 0V signal usable by the MCU to detect an external peripheral ON / OFF state.
It shall provide a polarisation of the line to allow R / 2R external systems. This polarisation shall be enabled or disabled with the MCU.

The principle of R / 2R is:

![Module global diagram overview](R_2R_principle.drawio.svg)

When the tamper switch is open, the input is HiZ.

When the alarm switch is open, the input is $$\displaystyle\frac{R_2}{R_1+R_2}$$

When all switch are closed, the input value depends on R2.

The input stage shall be implemented like this:

![Module global diagram overview](input_stage.drawio.svg)

The polarisation stage is driven by the MCU to switch on a 12V power supply on the input line. In this case and if no input charge is available, the line is driven high (+12V).

The level adaptation stage convert the +12V input level to a secure MCU compatible level (lower than 3.3V).

The input can be driven directly with the peripheral, in this case th polarisation is external a shall be switched off by the MCU.

Interfaces :

| Signal name   | Signal description     |
|---------------|------------------------|
| EXT_INPUT1    | general purpose input  |
| EXT_INPUT2    | general purpose input  |
| EXT_INPUT3    | general purpose input  |
| EXT_INPUT4    | general purpose input  |
| DRIVE_CMD1_EN | General purpose output |
| DRIVE_CMD2_EN | General purpose output |
| DRIVE_CMD3_EN | General purpose output |
| DRIVE_CMD4_EN | General purpose output |

Where DRIVE_CMDx_EN are the polarisation stage enable command and EXT_INPUTx are the ADC input after level adaptation for each input line.


#### Analog input submodule

This function shall be used to convert any external analog input voltages to a safe ADC range signal usable by the MCU.
This can reflect any analog peripheral and it is used in the application to measure:
  - AD_12V : battry input level.
  - AD_TEMP_CTN : temperature sensitive resistor value.
  - AD_LIGHT : light sensor. 

For safety reason, the sensor will be powered with the MCU voltage (+3.3V) and measured on an ADC input.

Interfaces :

| Signal name  | Signal description    |
|--------------|-----------------------|
| AD_12V       | analog input          |
| AD_TEMP_CTN  | analog input          |
| AD_LIGHT     | analog input          |
        

#### Power supply submodule

This function shall convert a module input voltage to a safe and regulated voltage domain to power all components on the board.

Three voltage domains are needed.

The +12V is used directly from he external supply, it is used as main power and is propagated to the external board (for example to driver relays).

The +5V is the communication domain, it is optional and shall be activated or implemented only if the MAX487 (RS-485 link) is used.

The +3.3V is the digital MCU domain, it is powered directly from the +12V power supply to be independant from the +5V regulator.

Each regulator is an LDO designed with all filtering needed.

Interfaces :

| Signal name  | Signal description         |
|--------------|----------------------------|
| +12V         | main external power supply |
| +5V          | communication power supply |
| +3.3V        | digital power supply       |


#### Microcontroller submodule

The microcontroller shall centralize and drive all peripherals.

Interfaces :

| Pin | Signal name     | Pin configuration             | Signal description                | Pin affectation  |
|-----|-----------------|-------------------------------|-----------------------------------|------------------|
|  1  | NC              | General purpose input         | Not connected                     | PC13             |
|  2  | RTC_CLK_IN      | Clock source input            | Clock source for RTC              | PC14 / OSC32_IN  |
|  3  | RTC_CLK_OUT     | Clock source output           | Clock source for RTC              | PC15 / OSC32_OUT |
|  4  | 3.3V_POWER      |                               | MCU main power                    | VBAT             |
|  5  | 3.3V_POWER      |                               | MCU main power                    | VREF+            |
|  6  | 3.3V_POWER      |                               | MCU main power                    | VDD/VDDA         |
|  7  | GND             |                               | MCU ground path                   | VSS/VSSA         |
|  8  | NRF24L01_CE     | General purpose output        | NRF24L01 chip enable              | PF0              |
|  9  | NRF24L01_IRQ    | General purpose input         | NRF24L01 interrupt line           | PF1              |
|  10 | \RST            | Reset                         | MCU reset                         | NRST             |
|  11 | EXT_INPUT1      | General purpose input         | Module external input             | PA0              |
|  12 | EXT_INPUT2      | General purpose input         | Module external input             | PA1              |
|  13 | EXT_INPUT3      | General purpose input         | Module external input             | PA2              |
|  14 | EXT_INPUT4      | General purpose input         | Module external input             | PA3              |
|  15 | DRIVE_CMD1_EN   | General purpose output        | External input drive enable       | PA4              |
|  16 | DRIVE_CMD2_EN   | General purpose output        | External input drive enable       | PA5              |
|  17 | DRIVE_CMD3_EN   | General purpose output        | External input drive enable       | PA6              |
|  18 | DRIVE_CMD4_EN   | General purpose output        | External input drive enable       | PA7              |
|  19 | HW_VERSION0     | General purpose input         | Hardware version (0 or 1)         | PB0              |
|  20 | HW_VERSION1     | General purpose input         | Hardware version (0 or 1)         | PB1              |
|  21 | HW_VERSION2     | General purpose input         | Hardware version (0 or 1)         | PB2              |
|  22 | AD_12V          | Analog input                  | Monitor 12V power supply          | PB10 / ADC_IN11  |
|  23 | AD_TEMP_CTN     | Analog input                  | Monitor analog temperature sensor | PB11 / ADC_IN15  |
|  24 | AD_LIGHT        | Analog input                  | Monitor light sensor              | PB12 / ADC_IN16  |
|  25 | SPI_SCK         | SPI2 master output            | SPI master clock                  | PB13 / SPI2_SCK  |
|  26 | SPI_MISO        | SPI2 master input             | SPI master data in                | PB14 / SPI2_MISO |
|  27 | SPI_MOSI        | SPI2 master output            | SPI master data out               | PB15 / SPI2_MOSI |
|  28 | NRF24L01_CS     | SPI2 master output            | SPI master chip select            | PA8              |
|  29 | 5V_EN           | General purpose output        | Enable communication power supply | PA9              |
|  30 | \MAX487_RE      | General purpose output        | Enable RS-485 driver reception    | PC6              |
|  31 | MAX487_DE       | General purpose output        | Enable RS-485 driver transmission | PC7              |
|  32 | NC              | General purpose input         | Not connected                     | PA10             |
|  33 | NC              | General purpose input         | Not connected                     | PA11             |
|  34 | NC              | General purpose input         | Not connected                     | PA12             |
|  35 | SWD_IO          | debug SWD bidirectional       | Serial wire debug data            | PA13 / SWDIO     |
|  36 | SWD_CLK         | debug SWD input               | Serial wire debug clock           | PA14 / SWCLK     |
|  37 | NC              | General purpose input         | Not connected                     | PA15             |
|  38 | NC              | General purpose input         | Not connected                     | PD0              |
|  39 | NC              | General purpose input         | Not connected                     | PD1              |
|  40 | DBG0            | General purpose output        | Pin used for debug purpose        | PD2              |
|  41 | DBG1            | General purpose output        | Pin used for debug purpose        | PD3              |
|  42 | HW_VERSION3     | General purpose input         | Hardware version (0 or 1)         | PB3              |
|  43 | NC              | General purpose input         | Not connected                     | PB4              |
|  44 | EXT_GPIO        | General purpose bidirectional | Module extension GPIO             | PB5              |
|  45 | UART_TX         | UART1 line output             | UART1 data transmission           | PB6 / USART1_TX  |
|  46 | UART_RX         | UART1 line input              | UART1 data reception              | PB7 / USART1_RX  |
|  47 | I2C_SCL         | I2C1 master output            | I2C1 clock                        | PB8 / I2C1_SCL   |
|  48 | I2C_SDA         | I2C1 master bidirectional     | I2C1 data line                    | PB9 / I2C1_SDA   |


#### Extensions submodule

This termination is a 6 wires port used to extend the module to welcome an optional daughter board.

This optional custom board can enhance the module functionalities.

Interfaces :

| Signal name  | Signal description    |
|--------------|-----------------------|
| EXT_GPIO     | general purpose IO    |
| I2C_SDA      | I2C data line         |
| I2C_SCL      | I2C clock line        |
| +3.3V        | +3.3V MCU, power line |
| +12V         | +12V MCU, power line  |
|  GND         | ground, power line    |


### 2.2.2 - Extension board

This hardware is used to extend the module's functionalities.
The general principle is composed of an I2C slave component to interface an analog discrete functionality as illustrated in figure 2.5.

![Module global diagram overview](module_extension_hardware_architecture.drawio.svg)


#### Relay switching extension

This extension board is composed of a single or several relays commutable through an I2C GPIO expander.
This function shall provide contacts normaly open or normaly closed based around relays.

The overall architecture is given in figure 2.6.

![Module global diagram overview](relays_extension_hardware.drawio.svg)

It is powered though the main 12V power supply.

A local regulator provides digital 3.3V power supply used by the GPIO expander and the NVM component.

A I2C NVM component is mainly used to store the extension board type and version.
It can also be used to store runtime application data.

An I2C GPIO expander switch on and off the relays by positionnong the corresponding pin high or low.
In the following truth table, the contact state depends on the relay output contact used (normaly closed or normaly open).

| Signal state | Relay position            |
|--------------|---------------------------|
| 0            | Relay off (not activated) |
| 1            | Relay on (activated)      |

The relays are powered with the main 12V and a voltage domain adaptation shall be implemented to be adressed with the GPIO expander.

Input (module / extension board link), this termination is a 6 wires interface used to ensure the communication and power supply between the module and the extension board:

| Pin number | Signal name  | Signal description                  |
|------------|--------------|-------------------------------------|
|     1      | +12V         | +12V power supply for relays        |
|     2      | +3.3V        | +3.3V power supply for digital      |
|     3      | EXT_IO_AN0   | External digital IO or analog input |
|     4      | I2C_SCL      | I2C bus clock line                  |
|     5      | I2C_SDA      | I2C bus data line                   |
|     6      | GND          | Power ground                        |

Output connector, for each relay implemented (this can vary) :

| Output name  | Output description                            |
|--------------|-----------------------------------------------|
| COMx         | Common relay contact                          |
| NOx          | Normaly open contact between the common pin   |
| NCx          | Normaly closed contact between the common pin |
