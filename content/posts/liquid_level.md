+++
title = 'Liquid Level Controller'
date = 2024-07-15T15:33:16-07:00
draft = false
math = true
tags = ["Arduino", "C++"]
showTags = true
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

All arduino programs (called sketches) have two functions: a `setup() `, which runs once to _setup_ variables and and a `loop()`, which does precisely what its name suggests, and _loops_ consecutively. Code outside those two functions will be just like any other C/C++ program, but to keep it simple we just used it to include libraries, initalize global variables and instantiate classes. Check out more infomation on basic Arduino sketches and load other of information [here](https://docs.arduino.cc/learn/programming/sketches/).

Let's get started! We'll work from top to bottom to avoid jumping around.

```cpp
    #include <PID_v1.h>

    double sensorValue;
    double output;
    String input;
    int outputPin = 3; //Digital output PMW pin connection
    int sensorPin = A0; //Analog sensor pin connection
```

First we include the PID library, which will do all the PID related calculations for us. Then, we need to declare some variables: two [doubles](https://www.programiz.com/cpp-programming/float-double) to store the reading the level sensor outputs and what the PWM should be set to, a string to hold any input to the serial monitor, and two integers to label the physical pins in a descriptive way.

```cpp
    double Kp = 50; //Proportional Gain
    double Ki = 1; //Integral Constant
    double Kd = 0; //Derivative Constant
    double level = 6; //Set Point (3-12 cm)

    double setPoint = 19.5 \* (level) + 462; //Internal setPoint
    PID myPID(&sensorValue, &output, &setPoint, Kp, Ki, Kd, DIRECT);
```

Next we declare and initalize the 3 PID parameters: proptional gain (Kp), integral constant (Ki), and derivative constant (Kd). We also need to store the level we want the water to remain, which requires a conversion to express the desired water level to units that match the level sensor’s output. Last we instantiate (create) an object of the PID class and include the variables we created as arguments, making sure to pass the address of the `sensorValue`, `output` and `setPoint`.

```cpp
    void setup (){
    Serial.begin(9600);
    pinMode(outputPin, OUTPUT);
    pinMode(sensorPin, INPUT);
    myPID.SetMode(AUTOMATIC);
    myPID.SetOutputLimits(128,255);
    }
```

Our setup function only requires a few things. We need to start the serial monitor so we can visualize data, such as the current water level, and input our desired level. Then we can use the standard pinMode function to tell the Arduino whether the pin should be reading data or writing data.Using the variables from earlier, we set the `outputPin` as output and the `sensorPin` as input. We also want to tell the PID class to adjust our output automatically (line 5). The limits of the PID's output need to correspond with a PWM signal, which ranges from 0 to 255. Here, the lower bound is 128 not 0 because the pump motor won’t turn on under that value, and since water is constantly leaving the cup, the pump should always be on. If you use a different pump, runs some simple tests to see the lower signal your pump can recieve.

Now to the bulk of the program, the **`loop()`** function.

```cpp
    void loop (){
    /*---------------PID Control---------------*/
    sensorValue = analogRead(sensorPin); //Read the value from the sensor
    myPID.Compute(); //PID computes output PWM
    analogWrite(outputPin, output); //Update PWM output value
    level = (sensorValue - 462) / 19.5; //Convert sensor value to liquid level
```

At the start of the loop, we first read the value from the level sensor; the standard `analogRead()` accomplishes this. Then, the PID class computes the desired output; remember since we passed certain [arguments by reference](https://www.geeksforgeeks.org/cpp-functions-pass-by-reference/) and set the mode to automatic, we don’t need to pass them into the compute function, their values will automatically be changed by the class. We can then use yet another standard function `analogWrite()` to send the PID’s output as a PWN singal to the pump. Last, we convert the level of the water from the level sensor scale back into units of cm, so we can print it out.

```cpp
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

Next, we need to read any value that might be in the serial monitor, store it in the variable as a string, and trim off any extra spaces. Then, if the input is a float and within the height of the sensor/container, the `setPoint` is changed to the new level (while converting it into the level sensor's units). Notice how the level _only_ gets changed if the input is a number within the range, this way any other garbage in the serial monitor doesn't mess with the system.

```cpp
  // Print time ms
  Serial.print(millis());
  Serial.print(", ");

  // Print liquid level in cm
  Serial.print(level, 2);
  Serial.print(", ");

  // Print PWM value
  Serial.println(output);

  delay(1000);
}
```

The last thing we need to implement is printing the data to the serial monitor, so that we can see how the water level and PWN signal are changing with respect to time. Adding things to the serial monitor is made quite convenient with the standard Serial library. We can use
`Serial.print()` to print the time, a comma, followed by the water level. Here we used commas to seperate the data, but spaces or any other special character work just as well. When printing the final data point, in our case the `output` variable, using `Serial.println()` will add a newline character to the end for clean formatting.
The final line of code (besides the closing bracket) adds a delay of 1000 milliseconds (1 second) in between each loop run because we don't that much data and accuracy for our system to work.
