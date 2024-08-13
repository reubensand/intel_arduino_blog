+++
title = 'How to Measure Temperature Using An Arduino and RTD'
date = 2024-07-25T10:58:28-07:00
draft = false
author = "Reuben De Souza"
+++

// Insert Top level image

# What is an RTD

// insert RTD picture

A _Resistance Temperature Dectector_ (RTD) is a sensor whose resistance changes as its temperature changes. The RTD we will be using today is the [PT100](https://www.adafruit.com/product/3290), from hardware company, _Adafruit_. The **PT** indicates that the "resistor" inside is just a small strip of platinum, and the **100** indicates at 0 degrees celsius the platinum will have a resistance of 100 Ohms. Platinum is used in the resistor because it is stable, provides repeatable results, and boasts a broad temperature range. Therefore, to obtain a tempereature reading, one must measure the resistance of the platinum, and perform a calculation to map the resistance to celsius.

The method to measure the resistance of the platinum is slightly more complicated than applying a voltage, measuring the corresponding current, then using Ohm's Law `R = V/I` to solve for `R`. Instead, a second reference resistor (RREF) is placed in series with the RTD. A voltage is applied to the pair, and the voltage of each individual resistor is measured. The ratio of these two voltages, which also respresents the ratio of resistance due to the constant current, can be multiplied by the known RREF to obtain the RTD's resistance. This process allows for accurate readings at low resistances, and recognizes small changes in resistance, improving the accuracy significantly. To accomplish this, a separate chip is necessary: Adafruit's [MAX31865 Amplifier](https://www.adafruit.com/product/3328). This amplifier includes the required chip to do the above measurement process, and the peripherals to communicate its reading to the Arduino using Serial Peripheral Interface [(SPI)](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi/all). SPI is a standardized way to use the [Serial](https://learn.sparkfun.com/tutorials/serial-communication/all) communication method, which makes talking to different external chips simple and customary. To find out more about the amplifier consult the [datasheet](https://learn.adafruit.com/adafruit-max31865-rtd-pt100-amplifier/downloads).

One last thing to mention about the RTD is the number of wires. Two wires are necessary to connect the platinum to the amplifier's measuring circuit, so that the process mentioned above can happen. However, since the distance between the amplifier and the platinum is long, we need to account for the resistance (and consequenatial voltage drop) of the wires two. By adding an extra wire to one side, we can measure the voltage drop across the wire and subtract that from the voltage reading we get from the platinum resistance. This explains why the Adafruit's PT100, contains 3 wires for the 1 meter cable.


# Pinout
![pinout](/intel_arduino_blog/images/pinout.jpg)
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

There are two other pins on the amplifier _3V3_ and _ROY_ aren't necessary for the basic operation of the PT100 tempereature sensor.

# Configuring the board

By default the board is set up for the 4 wire sensor. Since the PT100 only has 3 wires, a little soldering is necessary to configure thecircuits correctly. For those unfamiliar with soldering, its purpose is to make electrical connections that are a little more permanat than twisting two wires together. I'd recomend looking at a tutorial and practicing on some spare parts before attempting the real board. Here's a guide that might [help](https://learn.adafruit.com/adafruit-guide-excellent-soldering)
![jumpers](/intel_arduino_blog/images/top_down_board.jpg)

There are three jump pads that could be configured, we need to do the following:
-   {{<color color="red">}}Solder closed the jumper labeled 2/3 Wire {{</color>}}
-   {{<color color="green">}}Leave jumper labeled 2 Wire untouched {{</color>}}
-   {{<color color="purple">}}Cut the small tab just to the right of the _4_ on the pad labeled "24 3", then solder the rigthmost and middle pads closed.{{</color>}}

Tips for {{<color color="purple">}} "24 3" jump pad{{</color>}}:
-   Cutting the tab can be difficult and does scratch the surrounding area slightly, but the circuit should still work fine
-   Use a small amount of solder when connecting the middle and right pads and have a solder sucker ready in case the left pad accidentely gets connected
-   My first time trying to get the solder to lay correctly took quite a bit of trial and error, but just be careful to not overheat the chip

# Board Assembly
First, let's solder on the header pin, which when placed into a breadboard, will allow us to connect to the Arduino using jumper cables, consistently. 
![connector pins](/intel_arduino_blog/images/header-pin.jpg)
(this picture is just to display the header pin)

You'll want to solder this on to have consistent electrical contacts. I find doing this in a breadboard itself is easiest.
Place the **long** end into the breadboard holes, then slide the amplifier on top so its resting on the black bumper. Apply a small amount of solder to the top of each pin, and it should be good to go! Here's a picture of mine to reference. 

![connector pins done](/intel_arduino_blog/images/complete_v2.jpg)
If you zoom in, you see the small amount of solder on the pins, and the _correct_ jumper pads 

### Attaching the RTD
Attaching the sensor to the amplifier is simple. There should be two connector pins of matching color (mine are red), which also have a colored wire, and one other colored connector (mine was blue) attached to a metallic colored wire. Loosen the terminal's screws using a 2.4 mm screwdriver or anything you can make work. The connectors are U-shaped and you'll want to have one end of the U inserted into the terminal and the other rest on the bottom of the board so it looks like the connector is straddling the amplifier.
![sensor side angle](/intel_arduino_blog/images/sensor_side_angle.jpg)

The **matching pair** should go in the right terminal of the amplifier with the **F+** label in between the terminal and the punch hole, it doesn't matter which of the pair goes where. The solo wire should be placed in the other terminal, in either slot. Make sure to tighten screws so that the connection is secure. 
![sensor attached](/intel_arduino_blog/images/sensor_attached.jpg)

# Arduino Connection
Now let's connect the amplifier to the Arduino. I have an Arduino Uno R4 Minima, which is a basic board, but has all the required pins (4 digital + 5V + GND) Since we soldered the header pin in, we can use a breadboard and male to male [jumper wires](https://www.amazon.com/jumper-wires-male/s?k=jumper+wires+male+to+male) to make the connections easy to take disassemble/reassemble. 

Make the following connections:
-   Connect **Vin** to the 5V pin in the power section of the Arduino.
-   Connect **GND** to a common ground pin, also labeled **GND**
-   Connect **CLK** to **Digital #13**
-   Connect **SDO** to **Digital #12**
-   Connect **SDI** to **Digital #11**
-   Connect **CS** to **Digital #10**

Here is my finished product:

// take a good picture

### Download Adafruit_MAX31865 Library
Now that the hardware is all sorted, lets set up the software!

On the left bar in the Arduino IDE, click on the Library Manager (looks like books):
![library manager](/intel_arduino_blog/images/library_manager.png)

Type MAX31865 in the search bar:
![search bar](/intel_arduino_blog/images/search_libraries.png)

Click **Install**! (If a pop-up window prompts you, click Install All)

If you want to use Platformio instead, add the following line under the `[env:]` section of the `platformio.ini` file:
```
lib_deps = adafruit/Adafruit MAX31865 library@^1.6.2
```
This will autmatically install the library for you!

### Example Run
The Adafruit library comes with a basic example so let run and explain that!

To access it in the Arduino IDE, open the **File** menu, hover over **Examples** and scroll till you reach the **Adafruit MAX31865 library**, then click on **max31865**. A new window with the example program should pop up.   

![load demo](/intel_arduino_blog/images/get_example.png)

The example can also be accessed in Platformio under the following filepath:
**/.pio/libdeps/board_name/Adafruit MAX31865 library/examples/max31865/max31865.ino**
where board_name represents the board you set up the project with (mine is uno_r4_minima).

![load demo](/intel_arduino_blog/images/example_platformio.png)

You can also download this file [here](/intel_arduino_blog/files/max31865.ino).
Now you can upload your code, open the serial monitor, and hopefully your temperature readings will be displayed every second!

To understand the software better, and to be able to change things based on your use case lets take a look at the code:

```cpp
#include <Adafruit_MAX31865.h>

// Use software SPI: CS, DI, DO, CLK
Adafruit_MAX31865 thermo = Adafruit_MAX31865(10, 11, 12, 13);
// use hardware SPI, just pass in the CS pin
//Adafruit_MAX31865 thermo = Adafruit_MAX31865(10);
```
Just under the the license text at the top of the program sits the include statement for the library header file, if you have worked with Arduino's before you'll know this is how the library's code gets included. Then, the object that represents the amplifier and RTD, and its called `thermo`. You'll notice for "10, 11, 12, 13" pins from eariler, these pins can be changed, as long as the changes are represented in the code and on the board (which pin is which is written in the above commment). The amplifier class also has the ability to deal with SPI built in hardware (mentioned in the comments), but that is not necessary for our current setup.   

```cpp
// The value of the Rref resistor. Use 430.0 for PT100 and 4300.0 for PT1000
#define RREF 430.0
// The 'nominal' 0-degrees-C resistance of the sensor
// 100.0 for PT100, 1000.0 for PT1000
#define RNOMINAL 100.0
```
Next, two constants have to be defined. For the Adafruit MAX31865, the reference resistor (RREF) mentioned at the beginning is 430.0 Ohms, reflecting  the actual resistor on the amplifier board. Also, since the PT100 is designed to have a resistance of 100 Ohms at 0-degrees, we set our RNOMIAL to 100.0.

```cpp
void setup()
{
    Serial.begin(115200);
    Serial.println("Adafruit MAX31865 PT100 Sensor Test!");

    thermo.begin(MAX31865_3WIRE); // set to 2WIRE or 4WIRE as necessary
}
```
In our setup function, only two things need to happen: the serial monitor needs to be initalized to see the temperature readings, and the `thermo` object needs to begin the SPI communication. The `println()` is just to represent a start for outputs. 

```cpp
void loop()
{
    uint16_t rtd = thermo.readRTD();

    Serial.print("RTD value: ");
    Serial.println(rtd);
    float ratio = rtd;
    ratio /= 32768;
    Serial.print("Ratio = ");
    Serial.println(ratio, 8);
    Serial.print("Resistance = ");
    Serial.println(RREF * ratio, 8);
    Serial.print("Temperature = ");
    Serial.println(thermo.temperature(RNOMINAL, RREF));
```
Here is the main part of the loop! If you remember the explaination of how the resistance is actual measured/calculated, you should recognize the different numbers being printed. First, the ratio of resistance between the platinum and _RREF_ is being read from the amplifier by the `thermo` object's function `readRTD()`. That unsigned integer ratio is then normalized to a float ratio between 0 and 1 by dividing by 32768. Then, the resistance of the platinum in Ohms is found by multiplying the normalized ratio by _RREF_. Last, the calculation between resistance and temperature is done to yield the end goal, tempereature in celsius. All of these values are printed to the serial, right after they are calculated. Notice how for the final temperature calculation, a special `thermo.temperature()` is called with only two arguments. This means we
don't actually need to call the `readrtd()` and do all the calculations because the function already does it internally to get the temperature. 
```cpp
 // Check and print any faults
uint8_t fault = thermo.readFault();
if (fault)
{
    Serial.print("Fault 0x");
    Serial.println(fault, HEX);
    if (fault & MAX31865_FAULT_HIGHTHRESH)
    {
        Serial.println("RTD High Threshold");
    }
    ...
    if (fault & MAX31865_FAULT_OVUV)
    {
        Serial.println("Under/Over voltage");
    }
    thermo.clearFault();
}
```
This next part just makes use of the faults that the amplifier board is capable of detecting. These are not important unless fatal errors are occuring, so I won't cover them too in depth. Again reference the datasheet for all the information.
```cpp
    Serial.println();
    delay(1000);
}
```
To finish off the loop we can print an empty line for clean distinctions between data points, and add a delay to efficiently use the Arduino. 
I would also recommend checking out the [github](https://github.com/adafruit/Adafruit_MAX31865/) for the library to see how some of the functions are implemented.

If you read through this brief explanation thought a lot of this example code isn't super necessary, you are correct. Here's a file with the bare minimum to get a temperature reading every second:
```cpp 
#include <Adafruit_MAX31865.h>

Adafruit_MAX31865 thermo = Adafruit_MAX31865(10, 11, 12, 13);

#define RREF 430.0
#define RNOMINAL 100.0

void setup()
{
    Serial.begin(115200);
    thermo.begin(MAX31865_3WIRE); 
}

void loop()
{
    Serial.print("Temperature = ");
    Serial.println(thermo.temperature(RNOMINAL, RREF));
    Serial.println();
    delay(1000);
}
```
You can use this as the base for a more complicated project involving the PT100.

That brings us to the end of this tutorial. I hope you found it insightful!