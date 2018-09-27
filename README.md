# MICROPYTHON ON ESP8266 GUIDE FOR PYTHON PROGRAMMERS
## Introduction
- Simple guide to get python users with some background in microcontrollers started with micropython on esp8266.
-  Most if not all of the material in this guide has been copied from the [official documentation for the micropython on esp8266](https://docs.micropython.org/en/latest/esp8266/esp8266/tutorial/)
- I use this as a refresher after my brain memory has been f*cked up by lectures of communication skills at school.
## Table Of Content
* [Flashing the firmware to the board](#flashing-the-firmware-to-the-board)
* [Getting a Python REPL prompt](#getting-a-python-repl-prompt)
  * REPL over serial
  * WebREPL, prompt over wifi
* [Running script on start up](#start-up-scripts).
* [Network basics - connecting to a wireless AP / starting an AP](#network-basics).
  * Creating interface intances
  * Configuration and connecting to an AP
* [Using Ampy to upload code to Esp8266 via usb serial](#using-ampy-to-upload-code-to-esp8266-via-usb-serial)
* [GPIO pins - accessing, configuring, reading, setting pin values.](#gpio-pins)
    * Accessing and configuring a pin
    * Reading and setting pin values
* [External interrupts](#external-interrupts)
* [Pulse Width Modulation](#pulse-width-modulation)
    * PWM examples
* [Analog to digital conversion.](#analog-to-digital-conversion)
* [Contributing](#contributing)

##  Flashing the firmware to the board
* `pip install esptool`
* Then we erase the flash on the board first `sudo esptool.py --port /dev/ttyUSB0 erase_flash`
* Download the latest esp8266 [micropython firmware](http://micropython.org/download/#esp8266) 
* Flash the firmware to the board `sudo esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 micropython/esp8266-20180511-v1.9.4.bin`

## Getting a Python REPL prompt
### REPL over serial
- The REPL is always available on the UART0 serial peripheral, which is connected to the pins GPIO1 for TX and GPIO3 for RX. 
- The baudrate of the REPL is 115200. If your board has a USB-serial convertor on it then you should be able to access the REPL directly from your PC. Otherwise you will need to have a way of communicating with the UART.
- To access the prompt over USB-serial you need to use a terminal emulator program. On Windows `TeraTerm` is a good choice, on Mac you can use the built-in screen program, and Linux has `picocom` and minicom. Of course, there are many other terminal programs that will work, so pick your favourite!
- Picocom example `picocom /dev/ttyUSB0 -b 115200`
### WebREPL, prompt over wifi
- WebREPL client is hosted at http://micropython.org/webrepl . Alternatively, you can install it locally from the the GitHub repository https://github.com/micropython/webrepl

## Start up scripts
- There are two files that are treated specially by the ESP8266 when it starts up: `boot.py` and `main.py`. 
- The boot.py script is executed first (if it exists) and then once it completes the main.py script is executed.
- You can create these files yourself and populate them with the code that you want to run when the device starts up.

## Network basics
- Board has 2 WIFI interfaces - Access point `AP_IF` and another to connect to the station `STA_IF`.
- You can then use sockets to communicate with other devices on many internet protocols. 
### Creating interface intances
```python
import network
sta_if = network.WLAN(network.STA_IF)
ap_if = network.WLAN(network.AP_IF)
```
- To check interfaces status 
```python
sta_if.active()
ap_if.active()
#returns boolean
```
- check the network settings of the interface, returned values are: IP address, netmask, gateway, DNS
```python
ap_if.ifconfig() # returns ('192.168.4.1', '255.255.255.0', '192.168.4.1', '8.8.8.8')
```
### Configuration and connecting to an AP
- upon fresh install `AP_IF` is active and `STA_IF` is inactive.
```python
#First activate the station interface:
sta_if.active(True)
#Then connect to your WiFi network:
sta_if.connect('<your ESSID>', '<your password>')
#To check if the connection is established use:
sta_if.isconnected()
#Once established you can check the IP address:
sta_if.ifconfig()
#You can then disable the access-point interface if you no longer need it:
ap_if.active(False)
```
- Here is a function you can run (or put in your boot.py file) to automatically connect to your WiFi network:
```python
def do_connect():
    import network
    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.active(True)
        sta_if.connect('<essid>', '<password>')
        while not sta_if.isconnected():
            pass
    print('network config:', sta_if.ifconfig())
```
## Using Ampy to upload code to Esp8266 via usb serial
### Install ampy 
- You can clone the project `git clone https://github.com/adafruit/ampy.git` or use pip to install `pip install adafruit-ampy or pip3 install adafruit-ampy`.
### Usage
- run `ampy --help` to see usage
```
Usage: ampy [OPTIONS] COMMAND [ARGS]...

  ampy - Adafruit MicroPython Tool

  Ampy is a tool to control MicroPython boards over a serial connection.
  Using ampy you can manipulate files on the board's internal filesystem and
  even run scripts.

Options:
  -p, --port PORT    Name of serial port for connected board.  Can optionally
                     specify with AMPY_PORT environment variable.  [required]
  -b, --baud BAUD    Baud rate for the serial connection (default 115200).
                     Can optionally specify with AMPY_BAUD environment
                     variable.
  -d, --delay DELAY  Delay in seconds before entering RAW MODE (default 0).
                     Can optionally specify with AMPY_DELAY environment
                     variable.
  --version          Show the version and exit.
  --help             Show this message and exit.

Commands:
  get    Retrieve a file from the board.
  ls     List contents of a directory on the board.
  mkdir  Create a directory on the board.
  put    Put a file or folder and its contents on the...
  reset  Perform soft reset/reboot of the board.
  rm     Remove a file from the board.
  rmdir  Forcefully remove a folder and all its...
  run    Run a script and print its output.
```
- For example to upload a script to the board `ampy --port /dev/ttyUSB0 --baud 115200 put script.py`
## GPIO pins
- Not all pins are available to use, in most cases only pins 0, 2, 4, 5, 12, 13, 14, 15, and 16 can be used.
### Accessing and configuring a pin
- pins are available in the machine module, so make sure you import that first. Then you can create a pin using:
```python
pin = machine.Pin(0, machine.Pin.IN, machine.Pin.PULL_UP)
#machine.Pin.IN/machine.Pin.OUT
```
-You can either use PULL_UP or None for the input pull-mode. If it’s not specified then it defaults to None, which is no pull resistor. GPIO16 has no pull-up mode.
### Reading and setting pin values
- To read a pin value
```python
pin.value()
#returns 0 or 1
```
- To set value of a pin
```python
pin = machine.Pin(0, machine.Pin.OUT)
#Then set its value using:
pin.value(0)
pin.value(1)
#Or:
pin.off()
pin.on()
```
#### Nodemcu gpio pins image example
<img src="https://res.cloudinary.com/dyuhcszpn/image/upload/v1538044653/nodemcu_pins.png" alt="node mcu" height=350, width=300>

### External interrupts
- All pins except number 16 can be configured to trigger a hard interrupt if their input changes. You can set a callback function to be executed on the trigger.
- A hard interrupt will trigger as soon as the event occurs and will interrupt any running code, including Python code.
- As such your callback functions are limited in what they can do (they cannot allocate memory, for example) and should be as short and simple as possible.
- our simple callback
```python
def callback(p):
    print('pin change', p)
```
- Lets create 2 pins and configure them as input
```python
from machine import Pin
p0 = Pin(0, Pin.IN)
p2 = Pin(2, Pin.IN)
#An finally we need to tell the pins when to trigger, and the function to call when they detect an event:
p0.irq(trigger=Pin.IRQ_FALLING, handler=callback)
p2.irq(trigger=Pin.IRQ_RISING | Pin.IRQ_FALLING, handler=callback)
```
- We set pin 0 to trigger only on a falling edge of the input (when it goes from high to low), and set pin 2 to trigger on both a rising and falling edge. - After entering this code you can apply high and low voltages to pins 0 and 2 to see the interrupt being executed.

## Pulse Width Modulation
- a way to get an artificial analog `output` on a digital pin. It achieves this by rapidly toggling the pin from low to high.
- There are two parameters associated with this: the `frequency of the toggling`, and the `duty cycle`.
- The duty cycle is defined to be how long the pin is high compared with the length of a single period (low plus high time). Maximum duty cycle is when the pin is high all of the time, and minimum is when it is low all of the time.
- Duty cycle of a pin is between 0 (all off) and 1023 (all on), with 512 being a 50% duty. Values beyond this min/max will be clipped.
- On the ESP8266 the pins 0, 2, 4, 5, 12, 13, 14 and 15 all support PWM. The limitation is that they must all be at the same frequency, and the frequency must be between 1Hz and 1kHz.
```python 
import machine
p12 = machine.Pin(12)
#Then create the PWM object using:
pwm12 = machine.PWM(p12)
#You can set the frequency and duty cycle using:
pwm12.freq(500)
pwm12.duty(512)
pwm12 #returns PWM(12, freq=500, duty=512)
#You can also call the freq() and duty() methods with no arguments to get their current values.
#The pin will continue to be in PWM mode until you deinitialise it using:
pwm12.deinit()
```
### PWM examples
#### Fading and LED
- Let’s use the PWM feature to fade an LED. Assuming your board has an LED connected to pin 2.

```python
#create an LED-PWM object
led = machine.PWM(machine.Pin(2), freq=1000)
#we will use timing and some math, so import these modules:
import time, math
#Then create a function to pulse the LED:
def pulse(l, t):
    for i in range(20):
        l.duty(int(math.sin(i / 10 * math.pi) * 500 + 500))
    time.sleep_ms(t)
#For a nice effect you can pulse many times in a row:
for i in range(10):
    pulse(led, 20)
```
#### Controlling a hobby servo motor
- Hobby servo motors can be controlled using PWM. 
- They require a frequency of 50Hz and then a duty between about 40 and 115, with 77 being the centre value. 
```python
servo = machine.PWM(machine.Pin(12), freq=50)
servo.duty(40)
servo.duty(115)
servo.duty(77)
```

## Analog to Digital Conversion
- The ESP8266 has a single pin (separate to the GPIO pins) which can be used to read analog voltages and convert them to a digital value.
```python
import machine
adc = machine.ADC(0)
adc.read() #returns an interger
```
- The values returned from the read() function are between 0 (for 0.0 volts) and 1024 (for 1.0 volts). Please note that this input can only tolerate a maximum of 1.0 volts and you must use a voltage divider circuit to measure larger voltages.

## Contributing
- You can add more awesome short guides for the esp8266.
- fork or clone the repo and make the changes.
- If everything is OK. push your changes and make a pull request.
