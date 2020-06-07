# ESP8266
Material for the programming of ESP8266 based boards using micropython

## Flashing firmware on nodeMCU
Under linux the firmware can be flashed in the command line by using [esptool](https://github.com/espressif/esptool).

* Connect the board to the PC via micro USB
* Since the board is already equipped of a USB-serial interface, it should be readily visible as USB interface

```
lsusb
```
* If the interface is not listed among the detected USB devices remember that the quality/type of USB cable is extremely important (I've been trying at least three of them before getting the board recognized by the PC)!
* To flash the firmware it is necessary to identify the device which normally shows up as

```
/dev/ttyUSB0
```
* After the installation of `esptool`, the firmware can be downoladed from the [micropython](https://micropython.org/download/esp8266/) website and flashed on the nodeMCU 
  * ereasing the microcontroller flash
  ```
  esptool.py --port /dev/ttyUSB0 erase_flash
  ```
  * writing the actual firmware (here MYFIRMWARE.bin) with the correct baud rate (better explicitly identify the port)
  ```
  esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 MYFIRMWARE.bin 

  ```

## Accessing the Micropython console via USB
If the previous process has been succesful, it is now possible to interact with the ESP through the USB (actually serial) interface. In practice it will be possible to open a serial connection towards the ESP8266 and interactively run micropython commands on it through a REPL (Read Evaluate Print Loop).

In order to do that it is necessary to have a terminal emulator installed on the machine (if it missing install it trough the software manager). A possible choice is `picocom`. 

```
picocom /dev/ttyUSB0 -b115200
```

When the connection is established the user will recognize the standard Python prompt, which will allow to interactively run python code

```python
>>> print('hello esp8266!')
hello esp8266!
```
Even if the REPL is extremely stripped down, it will store a limited command history (accessible with the up arrow) and, more importantly, it supports TAB completion for python commands and objects.

To disconnect from the REPL type CTRL-a+CTRL-x


## Modes of operation
As for standard Python, on the ESP board the commands can be executed either interactively through the console, or "programmatically" running `.py` scripts which are present on the microcontroller flash memory.

To run micropython scripts it is necessary to be able to "transfer" the code to the ESP memory. A possible way to perform this operation is to use the command line utility [**ampy**](https://learn.adafruit.com/micropython-basics-load-files-and-run-code/install-ampy). the utility also allow some basic operations like listing the files present in the flash memory. 

For example, the command: 

```
ampy --port /dev/ttyUSB0 run test
```

will run the script `test.py` on the ESP board attached to the ttyUSB0 device.

The other ampy commands are:

* get
* ls
* put
* rm 
* run

I've found some issues when I use ampy and, in parallel, I'm interacting with the microcontroller throuch the REPL. To have ampy running smoothly (for example to put files on the ESP) it is better to disconnect from the REPL. In a sense the serial connection is already in use and this creates problems.

For ESP8266-based boards, before using a tool like ampy you might need to disable debug output on the board. This operation can be performed directly in the REPL

```python
>>> import esp
>>> esp.osdebug(None)
```

As an alternative the code can be included in the ESP boot script which will automatically executed when the board is powered-up.

There are two important files that MicroPython looks for in the root of its filesystem.  These files contain MicroPython code that will be executed whenever the board is powered up or reset (i.e. it 'boots').  These files are:

* /boot.py - This file is run first on power up/reset and should contain low-level code that sets up the board to finish booting.  You typically don't need to modify boot.py unless you're customizing or modifying MicroPython itself.  However it's interesting to look at the contents of the file to see what happens when the board boots up.  Remember you can use the ampy get command to read this and any other file!
* /main.py - If this file exists it's run after boot.py and should contain any main script that you want to run when the board is powered up or reset.

The main.py script is what you can use to have your own code run whenever a MicroPython board powers up.  Just like how an Arduino sketch runs whenever the Arduino board has power, writing a main.py to a MicroPython board will run that code whenever the MicroPython board has power.

The typical way of deploying a `main.py` file is first to interactively test the code in the REPL console, then to organize the code in a .py file with a text editor on the PC and, finally, to transfer the file  to main.py on a connected MicroPython board with ampy's put command:

```
ampy --port /serial/port put test.py /main.py
```

## Networking
The WSP8266 is provided of wi-fi capabilities. The microcontroller can serve either as access point (i.e. generating its own wireless network) or can connect to a router (station mode). The two modes of operation are managed by two interfaces which can be configured by using the `network` module


```python
>>> import network
>>> sta_if = network.WLAN(network.STA_IF)
>>> ap_if = network.WLAN(network.AP_IF)
```

By default, the ESP8266 is configured in access point mode, so the AP_IF interface is active while the STA_IF interface is inactive. To activate the station mode, disable the AP model, check the connection status and, finally, get the network configuration it is necessary to issue the following commands:

```python
>>> sta_if.active(True)
>>> ap_if.active(False)
>>> sta_if.connect('<your ESSID>', '<your password>')
>>> sta_if.isconnected()
>>> sta_if.ifconfig()
```
It is useful to wrap the connection steps inside a function which could be put (and called!) inside the `boot.py` file


```python
def do_connect():
    import network
    sta_if = network.WLAN(network.STA_IF)
    ap_if = network.WLAN(network.AP_IF)
    ap_if.active(False)
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.active(True)
        sta_if.connect('<essid>', '<password>')
        while not sta_if.isconnected():
            pass
    print('network config:', sta_if.ifconfig())

```

## MQTT
[MQTT](https://en.wikipedia.org/wiki/MQTT) is a lightweight *publisher-subscriber* communication protocol well suited for the exchange of information in low band high latency contexts. It is therefore a popular way to exchange information and commands in IoT networks.

The geeral idea is that a central dispatching unit (the so called *broker*) acts a s a sort of communication hub (think about it as a virtual notice-board) where messages (and commands) coming from the different devices are published (displayed). The devices in the network subscribe (and publish) to specific message topics, which are posted on the broker's notice-board. When a new message is published on a specific topic, the subscribers are notified.  MQTT is implemented over TCP/IP so the ESP8266 can act as a MQTT client posting and receiving messages from a MQTT broker.

The messages can be sensor readings or commands triggering the digital IO of the board.

MQTT capability can be implemented on the WSP8266 by using several libraries. Apparently, the community has been developing simple solutions (the more recent seems to be [umqtt.simple2] (https://github.com/fizista/micropython-umqtt.simple2)) or more robust versions (robust in the sense that they re-try to send data in case of conncetion losses) [mqtt.robust2](https://github.com/fizista/micropython-umqtt.robust2). 

But, how can one make the library availabe on the ESP8266? 

* install the library with `upip`. `upip`, read "micropip", is the micropython analogous of `pip`, the utility which allows to install software from python repos.
* upload the library as .py file on the ESP flash by using `ampy`

The first solution would be more linear, but I've encounter errors, as far as I understand due to the limited memory space of the microcontroller. In my case the second strategy was succesfull

These are the steps:

* Download the python file from the [umqtt.simple2] (https://github.com/fizista/micropython-umqtt.simple2)) GIT repository saving it as `MQTTsimple2.py` (or the name you prefer)
* Disconnect form the REPL (see above)
* Load the library on the ESP8266 by using `ampy`
 ```
 ampy --port /dev/ttyUSB0 put MQTTsimple2.py
 ```
* After connecting to the REPL
 * configure the network interface to be "online"
 * configure the MQTT client by defyning the IP of the MQTT broker (in my case 192.168.2.51) and the unique ID of the client

```python
>>> from MQTTsimple2 import MQTTClient
>>> mqtt_server = '192.168.2.51'
>>> import machine
>>> import ubinascii
>>> client_id = ubinascii.hexlify(machine.unique_id())
```
 * connect to the broker (if the connect command is successfull it will return `0`)

```python
>>> client = MQTTClient(client_id, mqtt_server)
>>> client.connect()
```
 * publish something on the broker. Here the topic is `test` and the message is `Alive`

```python
>>> client.publish(b'test', 'Alive')
```
... note the *b* before the "topic" string 










