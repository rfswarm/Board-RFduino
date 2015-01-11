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

Second you need to enable the stand-alone ST-Link mode of the discovery board by removing the two `CN2` jumpers, found somewhere in the upper right part of the board. This disconnects the ST-Link programmer from the micro-controller part of the port and enables direct access through the pin-header `CN3`, also labled `SWD`. The white dot tells you the position of pin1.

The RFduino module supports the Serial Wire Debug (SWD) interface. To access the device the following four lines need to be connected with the STM32x-discovery board:
```
                 RFduino module    STM32Fx-discovery
common ground:       GND <-----------> GND
supply voltage:      VDD <-----------> 3V
SWD clock:       FACTORY <-----------> SWCLK (CN3, pin2)
SWD data I/O:      RESET <-----------> SWDIO (CN3, pin4)
```

### Software
Debugging and programming the RFduino module works well with [[OpenOCD]]. 

We suggest to use a fairly recent version, best use the upstream version from their [git repository](http://sourceforge.net/p/openocd/code/ci/master/tree/). Version 0.9.0-dev-* is reported to work.

### Programming the Device
To program the RFduino module, just go to your RIOT application and type:
```
make flash
```
and voila, the new firmware should be flashed onto your device.

### Resetting the Device
As the RFduino module does not provide a reset button, RIOT includes a target to reset the board. To do that, just type
```
make reset
```
and your board will reboot.

### Programming the device manually
For OpenOCD to work correctly, you need the following configuration file (which you can also find in `RIOTDIR/boards/rfduino/dist/openocd.cfg`:

```
$ cat RIOTDIR/boards/rfduino/openocd.cfg
# nRF51822 Target
source [find interface/stlink-v2.cfg]

transport select hla_swd

set WORKAREASIZE 0x4000
source [find target/nrf51.cfg]

# use hardware reset, connect under reset
#reset_config srst_only srst_nogate
```

You can now program your device by doing the following:

1. start openocd with: `openocd -d3 -f RIOTDIR/boards/rfduino/dist/openocd.cfg`
2. open a new terminal an connect with telnet: `telnet 127.0.0.1 4444`
3. do the following steps to flash (only use bank #0 starting from address 0):

```
> flash banks
#0 : nrf51.flash (nrf51) at 0x00000000, size 0x00040000, buswidth 1, chipwidth 1
#1 : nrf51.uicr (nrf51) at 0x10001000, size 0x000000fc, buswidth 1, chipwidth 1

> halt
target state: halted
target halted due to debug-request, current mode: Thread 
xPSR: 0x61000000 pc: 0x00000e1a msp: 0x20001b2c

> flash write_image erase PATH-TO-YOUR-BINARY/YOUR-BINARY.bin 0
wrote xxx bytes from file PATH-TO-YOUR-BINARY/YOUR-BINARY.bin in xx.yys (x.yyy KiB/s)

> reset
```

### Using UART

The UART pins are configured in `boards/rfduino/include/periph_conf.h`.
The default values are PIN 1 and 2.

The default Baud rate is `115 200`.

It is also possible to use the USB Shield for RFduino (RFD22121) as UART adapter.
The PIN numbering is a bit confusing, here is an example on how to use GPIO0 (AREF) and GPIO1
as UART pins you have to change `boards/rfduino/include/periph_conf.h`:

```
#define UART_PIN_RX 0
#define UART_PIN_TX 1
```

### Troubleshooting
#### Compilation error (cpu/nrf51822/startup.c: LED_RED_TOGGLE undeclared)
For a quick fix just comment out the LED_RED_TOGGLE call at line 103 and recompile.

```
// LED_RED_TOGGLE;
```

#### Unlocking the flash memory (error erasing or writing to flash at address...)

If you holding a new device in your hands, there is a high change that your device's flash memory is locket and RIOT's make flash command will fail, saying something about erasing the flash was not possible.

A solution for this is to reset the chips code memory and user information registers. Just follow these steps:

Follow the steps described above for manually flashing the device:

1. start openocd with: `openocd -d3 -f RIOTDIR/boards/rfduino/dist/openocd.cfg`
2. open a new terminal an connect with telnet: `telnet 127.0.0.1 4444`
3. type `halt` to stop the device
4. type `nrf51 mass_erase` to reset the code memory
5. all done, make flash should now work as expected.

