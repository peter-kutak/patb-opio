% OPIO(1) opio v2.1
% Pat Beirne <patb@pbeirne.com>
% Oct 2021

# NAME
opio - Control GPIO pins on OrangePi. A replacement for WiringPi

# SYNOPSIS
**opio** [-2 | -9] readall {or status} 

**opio** [-2 | -9] readallx {or statusx}

**opio** [-2 | -9] exports

**opio** [-2 | -9] leds

**opio** mode *pin* [ in | out | alt ]

**opio** [ -d ] read *pin*

**opio** [ -d ] write *pin* [ 1 | 0 | on | off ]

# DESCRIPTION
**opio** allows access to the GPIO pins of OrangePi single-board computers. This version is designed speciifically for the

- OrangePi i96
- OrangePi 2G-iot

Running **opio** without any parameters will show its usage. **opio** requires 'su' permissions, so must be run as 'root' or via 'sudo'.

# COMMANDS
**readall** or **status**
: Displays the state of the gpio pins in a grid format. The list includes all the pins used in the on-board 40 pin connector. For each pin, the listing shows the gpio pin number, its alternate function, its i96 pin name, its current _mode_ and _value_, and the corresponding pin number on the 40 pin connector.

**readallx** or **statusx**
: Creates a similar chart, but includes the RDA pin names and Linux device driver names. 

**leds** 
: Creates a smaller chart, for the interesting I/O pins which are _not_ part of the 40 pin connector. On the i96 board, there are 3 LEDs which can be controlled via **opio**

**exports**
: Print a list of current entries in /sys/class/gpio, indicating which pins have been exported (prepared for read/write). If a gpio pin exists on the 40 pin connector, the pin number is listed.

**mode**
: Sets the _mode_ for a pin as either 'in', 'out' or 'alternate-function'. Normally **opio** will create an export for this gpio pin, and then set the direction. If you set the 'alt' function, the export will be removed. With the **-d** option, the export is not created, but the 'in'/'out'/'alt' _mode_ setting will still be done.

- **mode** with a pin number and no set-mode request, will simply return the current _mode_ (in, out, alt, in\*, alt\*).

**read**
: Returns the current _value_ (1/0) of the gpio pin, if possible. If the pin is in 'alt' _mode_, it is changed to 'in' before the _value_ is read.

**write**
: Attempts to write the given _value_ into the given pin. If the pin is in 'alt' _mode_, it is changed to 'out' before the _value_ is asserted.

# OPTIONS
**-d**
: Use low-level access to control the pins. Without the **-d** option, **opio** tries to use the 'export' gpio mechanism (at /sys/class/gpio). The **-d** option only applies to **mode, read** and **write**

**-2**
: Use the pin assignments for the OrangePi-2G-iot. If you wish to make this option persistent, create a file named /etc/OrangePi-2g-iot. 

**-9**
: Use the pin assignments for the OrangePi-i96. If you wish to make this option persistent, create a file named /etc/OrangePi-i96. In the absence of the *-9* and *-2* option, the program will attempt to auto-detect the board version.

## Mode
The microcontroller used on these boards presents i/o pins which can be set to 'general usage' GPIO as 'input' or 'output, or alternatively, set to operate in a specific way (uart, i2c, i2s, pcm, etc). **opio** refers the the current _mode_ of each pin one of these: either GPIO 'in', GPIO 'out', or 'alt'ernate function. 

The 'in\*' or 'out\*' are marked with an asterisk to indicate that the pin is in GPIO _mode_, but _not_ listed in the current 'export' list.

As a convenience, the **read** and **write** commands will automatically set the _mode_ on the GPIO pin and create an export. So, generally, the **mode** command is only *required* if you wish to change a pin back to its 'alt' function.

# HERITAGE
This program is styled after the **gpio** program written by Gordon Henderson for the Raspberry Pi.
Unlike the original **gpio** program, this one does not implement:

- export...........exports are created automatically by **mode, read** and **write**
- pwm, clk.........this microcontroller does not have PWM or CLK type pins
- aread, awrite....these boards have not connected and ADC/DAC pins

# NUMBERING
There are 4 naming schemes used to identify pins in these boards. **opio** uses exclusively the Linux gpio device driver numbers, the first on this list:

- Linux gpio numbers (from /sys/class/gpio)
- I/O connector pin numbers (1-40)
- RDA microcontroller pin names (like GPIOA_C23)
- i96 pin names (like GPIOB)

You can explore the correspondence between these naming schemes with **opio readall** and **opio readallx**

# HIGH LEVEL/LOW LEVEL ACCESS
Normally, **opio** will access the GPIO pins through the Linux gpio device driver, and the corresponding files at `/sys/class/gpio`. This is _high level_ access.

If you wish to bypass the Linux gpio driver, add the `-d` option to the **opio** command line and the reads/writes will be done at a _low level_, directly on the machine's registers.

The commands **readall, readallx** and **leds** are always done using _low level_ access. The **exports** command is always done with _high level_ commands.

Be careful about making changed with the **-d** option. Some linux gpio drivers will cache the direction and value, so changes you make with the **-d** option may not be reflected in the export folder.

# EXAMPLES

display a chart of the pin assignments of the 40 pin connector:

    opio readall     

list the currently 'exported' pins: 

    opio exports


set gpio 15 to an output; also create an export for gpio15
...then flash the pin (gpio15 Linux number): 

    opio mode 15 out    
    opio write 15 on
    sleep 2
    opio write 15 off   

set gpio 15 to an output; do *not* create an export

    opio -d mode 15 out 

write to gpio15 pin, bypassing the export mechanism

    opio -d write 15 on  


**NOTE** The more recent /dev/gpio driver is not yet available on these boards, since they're running the 3.xx kernels.

**NOTE** The 2G-IOT board uses the I2C1 bus to communicate with the modem chip. This is i2c-0 in the kernel, and is pins 3 & 5 on the 40 pin connector. Do not use these pins to connect to peripherals. And do not use **opio** to modify the _mode_ of these pins.

**NOTE** The 2G-IOT board uses the I2C3 bus to communicate with the LCD. If you are using an LCD in the socket, do _not_ change the mode on pins 38 & 40.

**NOTE** To create a man page, run `pandoc README.md -s -t man -o opio.1` For local usage, you can run `pandoc README.md -s --css pandoc.css -o opio.html` 
