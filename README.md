# ESP8266
Material for the programming of ESP8266 based boards using micropython

## Flashing firmware on nodeMCU
Under linux the firmware can be flashed in the command line by using [esptool](https://github.com/espressif/esptool).

* Connect the board to the PC via micro USB
* Since the board is already equipped of a USB-serial interface, it should be readily visible as USB interface

```
lsusb
```
* If the interface is not listed among the detected USB devices remamber that the quality/type of USB cable is extremely important!
* To flash the firmware it is necessary to identify the device which normally shows up as

```
/dev/ttyUSB0
```
* 


