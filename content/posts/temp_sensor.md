+++
title = 'MAX31865 and PT100 RTD'
date = 2024-07-25T10:58:28-07:00
draft = false
+++

This article will show you how to set up the PT100 Resistance Temperature Detector (RTD) with the **MAX31865** Sensor Amplifier.

# Basic Principles

RTDs contain a resistor that changes its resistance as the temperature flucuates. In the PT100, the resistor is a small strip of platinum, which will have a resistance of 100 ohms at 0 degrees celsius, which explains the name. Adafruit has a PT1000 that will have a resistance of 1000 ohms at 0 degrees C. The difference between the two is accuracy, since the same change in tempereature will cause a larger change in resistance on the PT1000, it will be more accurate. However, Platinum RTD's are inherently accurate so saving a few dollars by getting the PT100 will have little affect for most projects.

If we just attached our RTD to an Arduino, we wouldn't have a good way to measure the changes. Therefore, we need an amplifier to get the best accurary out of the probe. Adafruit's MAX31865 will not only amplify the signal, but also regulate other factors that go a bit beyond the scope of this article. Some RTD's also have more that the two wires that help account for lost voltage and improve accuracy; the one in the article has 3 wires, and the MAX amplifier can make use of this. To talk to the Arduino, the board uses the standard serial peripheral interface (SPI) and can be powered with 3V-5V, and can provide 3.3V for up to 100 mA as well.

# Pinout

![pinout](/intel_arduino_blog/images/board_pinouts.jpg)

The actual MAX31865 chip is a tiny integrated circuit while the board contains all the other components necessary to make it function. It also has several pins that make connecting to the board simple.

### Power Pins

-   Vin: The chips need 3V DC to operate, but an onboard voltage regulator allows up to 5V to allow common 5V microcontrollers (Arduinos) to connected smoothly.
-   GND: The common ground pin
-   3V3: This ouputs 3.3V and up to 100mA of current to be drawn. Not needed for basic temperature sensing.

### SPI Logic Pins

-   CLK: The _clock_ pin for SPI
-   SDO: **S**erial **D**ata **O**ut, also known as Microcontroller In Sensor Out, used for data sent from the _MAX_ to the _Arduino_
-   SDI: **S**erial **D**ata **I**n, also known as Microcontroller Out Sensor In, used for data sent from the _Arduino_ to the _MAX_
-   CS: **C**hip **S**elect, when low the MAX chip is active

Connecting multiple MAX31865's or other SPI devices? Have them share a CLK, SDO, and SDI pins but keep seperate CS pins and only activate one chip at a time.

-   RDY: the data-**ready** indicator pin, can be used to speed up reads for custom drivers. Adafruit's Arduino library doesn't make use of this pin.

# Sensor Terminal Blocks and Configuration Jumpers

![terminals](/intel_arduino_blog/images/board_terminals.jpg)

There are _four_ contacts that connect the RTD sensors to the board. All contacts don't need to be used since the board supports 2, 3, or 4 wire sensors. Additionally, the 3 and 4 wire sensor can be used as a 3 or 2 wire sensor by not connecting the extra wires, only sacrificing a little accurary.

![jumpers](/intel_arduino_blog/images/board_jumpers.jpg)
By default the board is set up for the 4 wire sensor, but the 2 and 3 wire set up only requires a little soldering.

**3-wire**

-   Solder closed the jumper labeled **2/3 Wire**
-   Leave jumper labeled **2 Wire** untouched
-   Cut the small tab in between the _2_ and _4_ on the last jumper, then solder the rigthmost and middle pads closed.

Tips for 3 pad jumper:

-   Cutting the tab can be difficult and does scratch the surrounding area slightly so just be mindful
-   Use a small amount of solder when connecting the middle and right pads and have a solder sucker ready in case the left pad accidentely gets connected
-   My first time trying to get the solder to lay correctly took quite a bit of trial and error, but just be careful to not overheat the chip

**2-wire**

-   Solder closed the jumper labeled **2/3 Wire**
-   solder closed the jumper labeled **2 Wire**
-   Leave the 3 pad jumper untouched

# Board Assembly

**Let's add the parts to the board**
The header strip is easiest to connect using a breadboard. Be sure to insert the **long pins down**.

![connector pins](/intel_arduino_blog/images/solder_pins.jpg)

Solder all pins for the most consistent electrical contact. If you need help soldering, Adafruit published an excellent guide
[here.](https://learn.adafruit.com/adafruit-guide-excellent-soldering)

In the included picture, the two 3.5mm terminal blocks used to connect the sensor to the board came soldered on. If your board looks like this
![complete](/intel_arduino_blog/images/complete_v2.jpg)

Note, I have the 3 wire version of the PT100 so my soldering is done accordingly.
(You can also see how I originally messed up the "24 3" pad, but after some cutting and resoldering, it works!)

# Wires

Why have the extra wire or two? Improved accuracy. In order to obtain very precise readings of low resistance, the resistance of the wire connecting the piece of platinum to the board must also be accounted for, since it could be a couple ohms, resulting in up to a degree celsius change. By having that extra wire, we can measure the resistance of the leads and subtract that from our total measured resistance. If you want more information Omega has a good explanation [here.](https://www.omega.com/en-us/resources/rtd-2-3-4-wire-connections)

![wire resistance](/intel_arduino_blog/images/three_wires.jpg)

You can see the difference in resistances between the wires in the 3-wire PT100 configuration.
For example, if the total measrued resistance is 104 ohms, the 2 ohms would be subtracted to get that the real resistance of the platinum is 102 ohms.

### Wiring it Up!

**4 Wires**
Typically, the wire that connect together have matching colors, but to double check connect a multimeter to see which ones are actually _pairs_.
Then, insert one pair of matching wires into one terminal and the same with the other pair. It doesn't matter which wire of the _pair_ goes on the inside or outside. Use a 2.4 mm flathead screwdriver to

![complete four wire](/intel_arduino_blog/images/four_wired.jpg)

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

![complete wiring](/intel_arduino_blog/images/final_product.jpg)

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
