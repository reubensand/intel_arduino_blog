+++
title = 'Liquid Level Controller'
date = 2024-07-15T15:33:16-07:00
draft = false
+++

## Abstract

Create a simple water level control system designed to maintain a consistent water level in a tank using a comibation of hardware and software.
Water leaves the tank through a hole at the bottom, and a pump with variable flow rate replenishs it. Other disturbances to the level, such as
water being poured in, should also be handled. To do this a liquied level sensor constantly monitors the water level. The reading it gives is
sent to an **Arduino**, which runs a **PID** (Proportional-Integral-Derivative) control algorithm. The PID controller calculates the difference
between the desired water level (setpoint) and the actual water level; then it determines the appropriate flow rate the submersible pump should
be at to minimize the error. A pulse-width modulation (PWN) signal is then sent to the pump, which increases the flow rate if the level is
below the setpoint and decreases the flow rate if it is above.

//insert demo picture here

## Parameters

Now that you have a decent idea of the system we can discuss the many parameters that affect how quickly the system is able to return to the desired water level.

## Setting up the experiement

Wiring up the experiment may seem daunting but we'll walk through the process step by step.

## The code

All arduino programs (called sketches) has two functions: a `setup() `, which runs once to _setup_ variables and and a `loop()`, which does precisely what its name suggests, and _loops_ consecutively. Code outside those two functions will be just like any other C/C++ program, but to keep it simple we just used it to include libraries, initalize global variables and instantiate classes. Check out more infomation on basic Arduino sketches and load other of information [here](https://docs.arduino.cc/learn/programming/sketches/).

Let's get started! We'll work from top to bottom to avoid confusion.

```cpp
    #include <PID_v1.h>

    double sensorValue;
    double output;
    String input;
    int outputPin = 3; //Digital output PMW pin connection
    int sensorPin = A0; //Analog sensor pin connection
```

First we include the PID library, which will do all the PID related calculations for us. Then, we need to declare some variables: two decimals
(double) to store the reading the level sensor gives and what the PWM should be set to, a string to hold whatever gets entered into the serial
monitor, and two numbers that describe what pin is doing what on the physical board.
```cpp
    double Kp = 50; //Proportional Gain
    double Ki = 1; //Integral Constant
    double Kd = 0; //Derivative Constant
    double level = 6; //Set Point (3-12 cm)
```
double setPoint = 19.5 \* (level) + 462; //Internal setPoint
PID myPID(&sensorValue, &output, &setPoint, Kp, Ki, Kd, DIRECT);

Next we have the 3 PID variables, as well as the level we want the water to remain, all stored as decimal numbers. Underneath those lines, we do
a conversion to express the desired water level we set earlier to units that match the level sensor’s output. Last we instantiate (create) a
version of the PID class and include the addresses of all the variables we created as arguments.
```cpp
    void setup (){
    Serial.begin(9600);
    pinMode(outputPin, OUTPUT);
    pinMode(sensorPin, INPUT);  
    myPID.SetMode(AUTOMATIC);
    myPID.SetOutputLimits(128,255);
    }
```
Our setup function only requires a few things. We need to start the serial monitor so we can visualize whatever data we need and input our desired level. Then we can use the built in pinMode function to tell the Arduino whether the pin should be reading data or writing data, using our variables from earlier. We also want to tell the PID class to adjust our output automatically, and set the limits of the output to correspond with a PWM signal, which ranges from 128 to 255. The lower bound is 128 not 0 because the pump motor won’t turn on under that value, and since water is constantly leaving the cup, the pump should always be on.

Now to the meat of the program, the loop function.
```cpp
    void loop (){
    /*---------------PID Control---------------*/
    sensorValue = analogRead(sensorPin); //Read the value from the sensor
    myPID.Compute(); //PID computes output PWM
    analogWrite(outputPin, output); //Update PWM output value
    level = (sensorValue - 462) / 19.5; //Convert sensor value to liquid level

        /*---------------Input Setpoint---------------*/
        //read and trim from serial input
        input = Serial.readString();
        input.trim();


        //set value for setpoint
        if(input.toFloat()){
            if(input.toFloat() >= 3 && input.toFloat() <= 12){
                setPoint = 19.5 * (input.toFloat()) + 462;
            }
        }
```
The first part of the loop all enables the feature to set the desired level through the serial monitor. There are quite a few built in functions to help us with this. We read the input as a string and trim all the extra space before/after. Then,
