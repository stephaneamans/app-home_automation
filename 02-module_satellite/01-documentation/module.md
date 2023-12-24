# 2 - Home automation satellite module


## 2.1 - Principle

The module is based around a microrontroller used to manage the RF tranceiver and all the inputs and outputs.
Additionaly it used to manage the power supply to optimize the power consumption of the module.
It compute all needed informations to perform the expected tasks.

When the module is used as an input it acquire informations (digital or analog) which are computed (filtering, debouncing, ...) by the microcontroller.
The data is then packed in a dedicated protocol to be sent to the master over the RF tranceiver.

When the module is used as an output, it receive informations which are unpacked to extract relevant payload.
The payload is computed by the microcontroller to perform the expected task.
The output is then activated or unactiveted (actuator) to pilot any peripheral.

![Module global diagram overview](module_global_diagram_overview.drawio.svg)

The module shall be used either as an input module, output module or both input ans output.

The power module is used to supply needed voltage domain (3.3V), the power consumption optimizations are done by the microcontroller unit.


## 2.2 - Hardware architecture

The hardware architecture is shown in the figure 2.2, declination of the module principle described in the section one

![Module global diagram overview](module_hardware_architecture.drawio.svg)

### NRF24L01 RF tranceiver function

This RF tranceiver is a 2.4 GHz multichannel transmit and receive module.
It can be found as a SOM (system on module) with an external antenna or a PCB printed antenna.

It is powered  wit a +3.3V power supply, and uses an SPI interface to communicate with master.

**Interfaces**:

SPI_SCK  : SPI clock, function input

SPI_MOSI : SPI slave data in, function input

SPI_MISO : SPI slave data out, function output

NRF24L01_CS : SPI chip select, function input

NRF24L01_CE : chip enable, function input

NRF24L01_IRQ : interrupt line, function output


### Clock function

This optional clock source can be used to enhance the precision of the RTC and microcontroller accuracy.

**Interfaces**:

Clock : clock source for MCU and RTC

### SWD serial wire debug function

This serial wire debug connector is a 4 wires connector.

It is used to plug a serial wire debug probe to perform microcontroller flash programming and provides numbers of debug features

**Interfaces**:

SWD_CLK : module input serial wire debug clock : function ouput

SWD_IO : serial wire debug data line : bidirectional

VCC  : +3.3V MCU, power line

GND : Ground, power line

### Extensions

This termination is a 6 wires port used to extend the module to welcome an optional daughter board.

This optional custom board can enhance the module functionalities.

**Interfaces**:

EXT_GPIO1 : general purpose IO, bidirectional

EXT_GPIO2 : general purpose IO, bidirectional

EXT_I2C_SDA : I2C data line, bidirectional

EXT_I2C_SCL : I2C clock line module output, function input

VCC  : +3.3V MCU, power line

GND : Ground, power line

### Runtime debug

This is a 5 wires connector used to perform runtime debug actions.

It can be separated into two different functionalities:

 - UART is used through a dedicated external converter to perform ``printf`` like functionalities to a terminal.

 - GPIOs are used to triggers and/or output states to an external analyser (oscilloscope, digital annalyser).

**Interfaces**:

DBG_TX : UART data out, module input

DBG_RX : UART data in, module output

GND : Ground, power line

DBG_GPIO1 : general purpose IO, bidirectional

DBG_GPIO2 : general purpose IO, bidirectional

### Digital input

This function shall convert any external binary signal to a safe 3.3V / 0V signal usable by the MCU to detect an external peripheral ON / OFF state.

**Interfaces**:

DIG_IN1 : module general purpose input, function output

DIG_IN2 : module general purpose input, function output

DIG_IN3 : module general purpose input, function output

DIG_IN4 : module general purpose input, function output

### Analog input

This function shall be used to convert any external analog input voltages to a safe ADC range signal usable by the MCU.
This can reflect any analog peripheral like humidity, temperature, light sensor or monitor input battery level.

**Interfaces**:

ADC1 : module analog input, function output

ADC2 : module analog input, function output

ADC3 : module analog input, function output

ADC4 : module analog input, function output

### Digital output

This function shall deliver safely 3.3V ON / OFF voltages.

**Interfaces**:

DIG_OUT1 : module general purpose output, function input

DIG_OUT2 : module general purpose output, function input

### Switching output

This function shall provide a contact normaly open or normaly closed based around relays.
A mecanism shall be developped to switch on and off the relays regarding the function input state.

**Interfaces**:

RLY1 : relay command, function input

RLY2 : relay command, function input

RLY3 : relay command, function input

RLY4 : relay command, function input

### Power supply

This function shaal convert a module input voltage to a safe and regulated voltage domain to power all components on the board.

**Interfaces**:

+3.3V : power supply

### Microcontroller

The microcontroller shall centralize and drive all peripherals.

**Interfaces**:

| Signal name  | Signal description           | Direction  |
|--------------|------------------------------|------------|
| SPI_SCK      | SPI clock                    | MCU output |
| SPI_MOSI     | SPI master out               | MCU output |
| SPI_MISO     | SPI master in                | MCU input  |
| SPI_CS       | SPI chip select              | MCU output |
| NRF24L01 CE  | NRF24L01 chip enable         | MCU output |
| NRF24L01 IRQ | NRF24L01 interrupt line      | MCU input  |
| MCU_CLK      | clock source for MCU and RTC | MCU bidir  |
| SWD_CLK      | serial wire debug clock      | MCU input  |
| SWD_IO       | serial wire debug data       | MCU bidir  |
| EXT_GPIO1    | general purpose IO           | MCU bidir  |
| EXT_GPIO2    | general purpose IO           | MCU bidir  |
| EXT_I2C_SDA  | I2C data line                | MCU bidir  |
| EXT_I2C_SCL  | I2C clock                    | MCU output |
| DBG_TX       | debug UART data out          | MCU output |
| DBG_RX       | debug UART data in           | MCU input  |
| DBG_GPIO1    | debug general purpose IO     | MCU bidir  |
| DBG_GPIO2    | debug general purpose IO     | MCU bidir  |
| DIG_IN1      | general purpose input        | MCU input  |
| DIG_IN2      | general purpose input        | MCU input  |
| DIG_IN3      | general purpose input        | MCU input  |
| DIG_IN4      | general purpose input        | MCU input  |
| ADC1         | analog input                 | MCU input  |
| ADC2         | analog input                 | MCU input  |
| ADC3         | analog input                 | MCU input  |
| ADC4         | analog input                 | MCU input  |
| DIG_OUT1     | general purpose output       | MCU output |
| DIG_OUT2     | general purpose output       | MCU output |
| RLY1         | relay command                | MCU output |
| RLY2         | relay command                | MCU output |
| RLY3         | relay command                | MCU output |
| RLY4         | relay command                | MCU output |




