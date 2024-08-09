+++
title = 'How to Measure Temperature Using An Arduino and RTD'
date = 2024-07-25T10:58:28-07:00
draft = false
author = "Reuben De Souza"
+++

// Insert Top level image

# What is an RTD

A _Resistance Temperature Dectector_ (RTD) is a simple sensor whose resistance changes as its temperature changes. The RTD we will be using today is the [PT100](https://www.adafruit.com/product/3290), from a well respected hardware company, _Adafruit_. The **PT** indicates that the "resistor" inside is just a small strip of platinum, and the **100** indicates at 0 degrees celsius the platinum will have a resistance of 100 ohms. Platinum is used in the resistor because it is stable, provides repeatable results, and boasts a broad temperature range.

// insert RTD picture

The method to calculate the resistance of the platinum is slightly more complicated than applying a voltage, measuring the corresponding current, then using Ohm's Law `R = V/I` to solve for `R`. Instead, a second reference resistor (RREF) is placed in series with the RTD. A voltage is applied to the pair, and the voltage of each individual resistor is measured. The ratio of these two voltages, which also respresents the ratio of resistance due to the constant current, can be multiplied by the known RREF to obtain the RTD's resistance. This process allows for accurate readings at low resistances, and recognizes small changes in resistance, improving the accuracy significantly. To accomplish this, a separate chip is necessary: Adafruit's [MAX31865 Amplifier](https://www.adafruit.com/product/3328). This amplifier includes the required chip to do the above measurement process, and the peripherals to communicate its reading to the Arduino using Serial Peripheral Interface [(SPI)](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all). SPI is a standardized way to use the [Serial](https://learn.sparkfun.com/tutorials/serial-communication/all) communication method, which makes talking to different external chips simple and customary. To find out more about the amplifier consult the [datasheet](https://learn.adafruit.com/adafruit-max31865-rtd-pt100-amplifier/downloads).

# Pinout

![pinout](/intel_arduino_blog/images/top_down_board.jpg)

The black box in the middle is the actual chip (also known as an integrated circuit). All the other parts are used to support the chip's functions, such as the reference resistor, labeled _RREF_.

### To connect to an Arduino we need to use the following pins:

#### Power Pins

-   VIN: (**V**oltage **In**), the board needs 3V DC to operate, but an onboard voltage regulator allows for an Arduino's 5V power pin to connect seamlessly
-   GND: The ground pin, a feature for every circuit

#### SPI Logic Pins

-   CLK: The **Cl**oc**k** pin, vital to any communication protocol
-   SDO: **S**erial **D**ata **O**ut, also known as Microcontroller In Sensor Out, used for data sent from the _amplifier_ to the _Arduino_
-   SDI: **S**erial **D**ata **I**n, also known as Microcontroller Out Sensor In, used for data sent from the _Arduino_ to the _amplifier_
-   CS: **C**hip **S**elect, when low the amplifier is active

These 4 pins are the heart of the SPI protocol and are all we need, besides power and ground to communicate between the amplifier and Arduino.

# Configuring the board

By default the board is set up for the 4 wire sensor. Since our PT100 only has 3 wires, a little soldering is necessary to configure thecircuits correctly. For those unfamiliar with soldering, its purpose is to make electrical connections that are a little more permanat than twisting two wires together. I'd recomend looking at a tutorial and practicing on some spare parts before attempting the real board. Here's a guide that might [help](https://learn.adafruit.com/adafruit-guide-excellent-soldering)
![jumpers](/intel_arduino_blog/images/top_down_board.jpg)

There are three jump pads that could be configured, we need to do the following:

-   {{<color color="red">}}Solder closed the jumper labeled 2/3 Wire {{</color>}}
-   {{<color color="green">}}Leave jumper labeled 2 Wire untouched {{</color>}}
-   {{<color color="purple">}}Cut the small tab just to the right of the _4_ on the pad labeled "24 3", then solder the rigthmost and middle pads closed.{{</color>}}

Tips for 3 pad jumper:

-   Cutting the tab can be difficult and does scratch the surrounding area slightly, but the circuit should still work fine
-   Use a small amount of solder when connecting the middle and right pads and have a solder sucker ready in case the left pad accidentely gets connected
-   My first time trying to get the solder to lay correctly took quite a bit of trial and error, but just be careful to not overheat the chip

# Board Assembly

![connector pins](/intel_arduino_blog/images/solder_pins.jpg)

Why have the extra wire or two? Improved accuracy. In order to obtain very precise readings of low resistance, the resistance of the wire connecting the piece of platinum to the board must also be accounted for, since it could be a couple ohms, resulting in up to a degree celsius change. By having that extra wire, we can measure the resistance of the leads and subtract that from our total measured resistance. If you want more information Omega has a good explanation [here.](https://www.omega.com/en-us/resources/rtd-2-3-4-wire-connections)

![wire resistance](/intel_arduino_blog/images/three_wires.jpg)

You can see the difference in resistances between the wires in the 3-wire PT100 configuration.
For example, if the total measrued resistance is 104 ohms, the 2 ohms would be subtracted to get that the real resistance of the platinum is 102 ohms.

### Wiring it Up!

**4 Wires**
Typically, the wire that connect together have matching colors, but to double check connect a multimeter to see which ones are actually _pairs_.
Then, insert one pair of matching wires into one terminal and the same with the other pair. It doesn't matter which wire of the _pair_ goes on the inside or outside. Use a 2.4 mm flathead screwdriver to

You should not have soldered any of the jumper so your final board should look similar to this.

**3 Wires**
Again, check to make sure the matching colors have around a 2 ohm between them. This time insert the _matching pair_ in the right terminal with the **F+** label in between the terminal and the punch hole. The solo wire should be placed in the other terminal, in either slot.

![complete three wire](/intel_arduino_blog/images/three_wired.jpg)

You should have soldered the "2/3 Wire" jump pad and the right two pads of the jumper with the directly flower to the right.

**2 Wire**
This is the simplest. Just place both wires in one of the terminal. Then either solder closed the jumpers next to the RTD terminal block or put little wires in the right and left terminal blocks to short them together.

# Arduino Connection

### SPI Wiring

On an Arduino board:

-   Connect **Vin** to the 5V or 3V power pin, for most Arduinos the 5V works well.
-   Connect **GND** to a command ground pin, also labeled **GND**
-   Connect **CLK** to **Digital #13**
-   Connect **SDO** to **Digital #12**
-   Connect **SDI** to **Digital #11**
-   Connect **CS** to **Digital #10**

Other digital pins can be used, as long as the code reflects the changes. You'll see what I mean when we look at the library.

### Download Adafruit_MAX31865 Library

In the Arduino IDE, open up the library manager:

![library manager](/intel_arduino_blog/images/library_manager_menu.png)

Search for the Adafruit MAX31865 library:

![search bar](/intel_arduino_blog/images/search_max31865.png)

Click **Install**!

Alternatively, in Platformio add the following line under the `[env:]` section of the platformio.ini file:

```
lib_deps = adafruit/Adafruit MAX31865 library@^1.6.2
```

This will autmatically install the library for you!

### Run Example

In the Arduino IDE, go to **File->Examples->Adaruit MAX31865 library->max31865**
The example file should pop up.

![load demo](/intel_arduino_blog/images/load_demo.png)

The example can also be accessed in Platformio under the following filepath:
**.pio->libdeps->board_name->Adafruit MAX31865 library->examples->max31865->max31865.ino**

![load demo](/intel_arduino_blog/images/example_platformio.png)

You can also download this file [here].

Right underneath, the instantiation of the class, two `#define` are made. It's important to change those accordingly based on if you are using the PT100 or PT1000. For the PT100 my section looks like this:

```cpp
// The value of the Rref resistor. Use 430.0 for PT100 and 4300.0 for PT1000
#define RREF      430.0
// The 'nominal' 0-degrees-C resistance of the sensor
// 100.0 for PT100, 1000.0 for PT1000
#define RNOMINAL  100.0
```

Now you can upload your code, open the serial monitor, and hopefully your temperature readings will be displayed every second!
