---
title: Quadcopter project log
date: 2022-11-02 19:30:00 -0600
categories: [Projects]
tags: [quadcopter, diy, build, project log]     # TAG names should always be lowercase
---

# The project
As a fun side project I decided to take up building my own quadcopter out of some spare parts I had received.
The build centers around the F450 frame and is intended to be a platform to test in the future advanced control techniques (Adaptive controllers, Neurocontrollers, MPCs, etc.) and autonomy techniques (path planning, SLAM, ...).

![Drone first complete build](/assets/img/drone_log-first_build.jpg)
# BOM
- DJI F450 frame
- Omnibus F4 SD
- 3300mAh 3S LiPo battery
- 30A BLDC ESC [PDF manual](/assets/30A_BLDC_ESC_Product_Manual%202022-03-09%2015_25_48.pdf)

# Joystick setup

As we're not going to use an RC controller but rather control the drone through the WiFi connection of the ESP-8266 (which in turn communicates with the Flight Controller through the MAVLink protocol) we need to disable the check for the presence of the RC controller, which prevents the drone from taking off in absence of it.

- [COM\_RC\_IN\_MODE=1](https://docs.px4.io/master/en/advanced_config/parameter_reference.html#COM_RC_IN_MODE) - Joystick/No RC Checks

## ESC Configuration

The center pin of the ESC

PX4 parameters modified

- **PWM\_MAIN\_RATE**  – 50Hz pwm output frequency for the ESC (ESC-30a are not very fast), default was 400Hz, too much

# Serial configs

In order to use the ESP WiFi MAVlink bridge we need to setup the Serial communication first and then to setup the MAVlink on this serial port. In this build the port UART6 (TS6/RX6 on the Omnibus F4 SD) is used.

PX4 parameters modified:

- GPS\_1\_CONFIG – 0 this disables the default GPS assignment on UART6, in order to use it for the MAVlink serial connection to the ESP
- MAV\_0\_CONFIG – UART 6 , this sets the MAV\_0 link on the UART 6 port
- SER\_URT6\_BAUD – 921600 8N1, sets the correct baud rate on the UART6 port
- MAV\_0\_RATE – 92160, this sets the MAV\_0 link datarate, as the connection is an 8N1 type this is set as 921600/10=92160 (this is a rule given by the documentation)

# ESP 8266 firmware flash

Since we're using off-the-shelf esp-8266 we are going to need to flash a special firmware on it to enable the communication through the MAVLink protocol

**Describe how to flash**

# ESP 8266 wiring

# ESCs

Invalid Minimum Value

Some ESCs need to see a special low value pulse before switching on (to protect users who have the throttle stick in the middle position on power-up).

PX4 sends a value of [PWM\_MAIN\_DISARM](https://docs.px4.io/master/en/advanced_config/parameter_reference.html#PWM_MAIN_DISARM) pulse when the vehicle is disarmed, which silences the ESCs when they are disarmed and ensures that ESCs initialise correctly.

This value should be set correctly for the ESC (correct values vary between roughly 1200 and 900 us).

- TRY TO CHANGE IT UNTIL THE ESCs work correctly as soon as powered up

**ESCs update 09/03/2022:**

I suspect the motors are "cogging". When spun up cold sometimes they stutter but when a slight load is applied they start spinning correctly. This is a weird behaviour, looking into how to improve it.

I tested this by running the motors out of an Arduino. Since the problem kept showing up I disassembled them and checked for a short circuit of some of the windings or of the windings with the screw that mount the BLDC's stator on the frame and this was not the case.

[Brushless Motor Cogging Explained – May Not be What You'd Think! - YouTube](https://www.youtube.com/watch?v=i33uD18YEAU)

[(23) How to Improve Cogging or Start Up Hesitation in a Brushless Motor (Occurs in a Sensorless Motor) - YouTube](https://www.youtube.com/watch?v=M2k7QloXVNo)

I've finally figured out the problem. The motors were being tested without the propellers and this was causing troubles with the ESCs. More in detail the Hobbypower ESCs are so-called "sensorless BLDC"; this means that they detect if the rotor is actually spinning and at what speed by measuring the back-EMF; however when there are no propellers installed there is basically no torque load (i.e. the rotor spins freely as there is no drag from the blades generating a torque opposite to the spinning direction). This results in the BLDC stuttering and the ESC shutting down the power after 2 seconds as a safety measure, in order to avoid frying the MOSFET.

**ESC startup problem**

bug with the ESCs at startup: when the autopilot is first powered on through usb and then the lipo is connected the ESCs work fine, however when I power up the ESCs and the FC simultaneously the ESCs do not enter in the correct mode and Cannot be armed.
![ESC cogging problem](/assets/img/drone_log-esc_startup_problems.png)

Try to change **PWM\_MAIN\_DISARM**
# First test flight 28-03-22
The first test flight managed to get the drone up in the air... and crash it a few seconds later! There was probably an eccessive rate of climb resulting in a divergence of the control action as PID tuning had not yet been performed. 
This resulted in a broken arm and blade, but it was swiftly replaced to get it back to action!

The video is shot from a DJI drone with my father as the filmmaker!

<iframe width="560" height="315" src="https://www.youtube.com/embed/yYXmbW054NU?start=13" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**GPS setup**

Using an Arduino the GPS module has been tested and verified to work.

**GPS Baud rate: 9600baud**
The default baud rate of the GPS module has been verified to be 9600baud
![GPS baud rate test](/assets/img/drone_log-gps_baud_rate.png)