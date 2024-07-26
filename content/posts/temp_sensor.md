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

**Power Pins**

-   Vin: The chips need 3V DC to operate, but an onboard voltage regulator allows up to 5V to allow common 5V microcontrollers (Arduinos) to connected smoothly.
-   GND: The common ground pin
-   3V3: This ouputs 3.3V and up to 100mA of current to be drawn. Not needed for basic temperature sensing.

**SPI Logic Pins**

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
