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
* After the installation of `esptool`, the firmware can be downoladed from the [micropython](https://micropython.org/download/esp8266/) website and flashed on the nodeMCU 
  * ereasing the microcontroller flash
  ```
  esptool.py --port /dev/ttyUSB0 erase_flash
  ```
  * writing the actual firmware (here MYFIRMWARE.bin) with the correct baud rate (better explicitly identifying the port)
  ```
  esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 MYFIRMWARE.bin 

  ```

## Accessing the Micropython console via USB
If the previous process has been succesful, it is now possible to interact with the ESP through the USB (actually serial) interface. In practice it will be possible to open a serial connection towards the ESP8266 and interactively run micropython commands on it through a REPL (Read Evaluate Print Loop).

In order to do that it is necessary to have a terminal emulator installed on the machine (if missing install it trough the software manager). A possible choice is `picocom`

```
picocom /dev/ttyUSB0 -b115200
```

When the connectio is established the user will recognize the standard Python prompt which will allow to interactively run python code

```python
>>> print('hello esp8266!')
hello esp8266!
```
Even if REPL is extremely stripped down, it will store a limited command history (accessible with the up arrow) and, more importantly, it supports TAB completion for python commands and objects.





