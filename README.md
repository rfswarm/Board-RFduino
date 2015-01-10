# Board-RFduino
RIOT OS support

## Overview

The RFduino is a finger-tip sized wireless enabled microcontroller. The RFduino module is utilizing Nordics NRF51822QFAA. The SoC features 16Kb of RAM, 256Kb of flash ROM and comes on top of the usual micro-controller peripherals with a 2.4GHz radio that supports both Nordics proprietary ShockBurst as well as Bluetooth Low Energy (BLE).

The board is available at well known global chip distributors (Mouser, Micro Center, Arrow etc.)
[http://www.rfduino.com/sales](http://www.rfduino.com/sales/index.html)

## Hardware

| MCU 		| NRF51822QFAA 		|
|:------------- |:--------------------- |
| Family	| ARM Cortex-M0 	|
| Vendor	| Nordic Semiconductor	|
| RAM		| 16Kb	|
| Flash		| 256Kb				|
| Frequency	| 16MHz |
| FPU		| no				|
| Timers	| 3 (2x 16-bit, 1x 32-bit [TIMER0])	|
| ADCs		| 1x 10-bit (8 channels)		|
| UARTs		| 1 				|
| SPIs		| 2					|
| I2Cs		| 2 				|
| Vcc		| 1.8V - 3.6V			|
| Datasheet 	| [Datasheet](http://www.rfduino.com/wp-content/uploads/2014/03/rfduino.datasheet.pdf) (pdf file) |
| Reference Manual | [Reference Manual](http://www.100y.com.tw/pdf_file/39-Nordic-NRF51822.pdf) |

##  Flashing and Debugging
The RFduino module comes without any on-board programming and flashing capabilities. It supports however to be programmed using of-the-shelf programmers as Segger's JLink or STM's STLink.

A very simple and affordable way to program and debug the RFduino module is to the integrated ST-Link/V2 programmer of any STM32Fx-discovery board. The needed steps are described in the following sections. If you want to use a stand-alone ST-Link adapter, you just simply have to alter the wiring to fit for your programmer, the software part is identical.

### ST-Link USB Hardware
First of all make sure the your ST-Link device is detected and can be accessed properly. In Linux you might have to adept your `udev` rules accordingly:
```
wget https://raw.githubusercontent.com/texane/stlink/master/49-stlinkv2.rules
sudo cp 49-stlinkv2.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```
now replug the usb cable and flash.

Have a look at the 'Setting up udev rules' section in this [README file](https://github.com/texane/stlink/blob/master/README) if you need help.

Second you need to enable the stand-alone ST-Link mode of the discovery board by removing the two `CN2` jumpers, found somewhere in the upper right part of the board. This disconnects the ST-Link programmer from the micro-controller part of the port and enables direct access through the pin-header `CN3`, also labled `SWD`.

The RFduino module supports the Serial Wire Debug (SWD) interface. To access the device the following four lines need to be connected with the STM32x-discovery board:
```
                 RFduino module    STM32Fx-discovery
common ground:       GND <-----------> GND
supply voltage:      VDD <-----------> 3V
SWD clock:           SWD <-----------> SWCLK (CN3, pin2)
SWD data I/O:      SWDIO <-----------> SWDIO (CN3, pin4)
