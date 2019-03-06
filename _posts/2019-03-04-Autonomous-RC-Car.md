---
layout: post
title:  "Making an Autonomous RC Car"
date:   2019-03-04 12:45:09 -0400
categories: jekyll update
---

For my Senior Project at DePauw I decided to extend some of the research that I had done previously at Auburn University. There were a lot of ideas that I had still wanted to try out, but on things that were a bit easier to control, like an RC car. In this post I will walk through the materials that I used and the different steps it took to make an Autonomous RC car. Find the previous research [here](http://www.masonseeger.me/jekyll/update/2018/06/05/Auburn-REU.html).

The basis of this project was using LiDAR and multiprocessing on a Raspberry Pi to make an autonomous RC car. You can find all of the code in my [autocar repository](https://github.com/masonseeger/autoCar/blob/master/keyController.py) on github. Feel free to DM me any question or improvements!

##1. Materials ##

The following is a detailed list of materials that I used for this project. In total everything costs around $550-$600.

* Exceed RC 1/10 2.4Ghz Electric Infinitive EP RTR Off Road Truck TT Yellow RC Remote Control RC Car - used as the RC car.
* Anker Astro E1 Candy-Bar Sized Ultra Compact Portable Charger, External Battery Power Bank, with High-Speed Charging PowerIQ Technology - Used as an external battery to power the raspberry pi.
* ELEMENT Element14 Raspberry Pi 3 B+ Motherboard - used as controller and computer for driving.
* Sandisk Extreme 32GB Micro SDHC UHS-I Card with Adapter [Old Version] - SDSQXVF-032G-GN6MA - essentially the hard drive for the Raspberry Pi
* 16 Channel PWM/Servo Driver IIC interface-PCA9685 for arduino or Raspberry pi shield module servo shield - helps communicate with multiple servo and DC motors.
* SainSmart HC-SR04 Ranging Detector Mod Distance Sensor (Blue) - used for ledge detection.
* Arducam 5 Megapixels 1080p Sensor OV5647 Mini Camera Video Module for Raspberry Pi Model A/B/B+, Pi 2 and Raspberry Pi 3,3B+ - Used to stream live footage of what the RC car sees.
* RPLIDAR A2M8 360° Laser Range Scanner - a LiDAR sensor used for the autonomous driving function.
* 120pcs 20cm Multicolor Jumper Wires for Arduino / Raspberry Pi, 40 Male to Male, 40 Male to Female, 40 Female to Female by Corpco - used to connect various parts to the Raspberry Pi GPIO pins.

##2 Setup##
The setup is simple, but I advise using caution and easily recognizable wiring conventions (ie red = power, black = ground, etc.) because it is very easy to end up frying different components. If you want to be even more cautious you can use resisters, but I encountered no problems without them.

Connect the car using the Servo Driver and the Raspberry pi. I used [this tutorial](www.youtube.com/watch?v=byRLYkZkJZE) and found that it covers this subject very well. Make sure that you remember which cable is connected to which part of the servo driver. The rest of the parts are pretty self explanatory except for the SainSmart range detector which will be discussed later.

##3 Connect to the Raspberry Pi##
This is fairly simple, while you are on the same network as the Pi, simply ssh into the Pi using its IP address. You can find this a bunch of different ways, I just used an app called Fing to scan all of the IP addresses and names currently on my network.
##4 Installing the required packages and making a driving function##
To make a driving function on your raspberry pi using python we need a few different libraries. Those can be found below:
{% highlight python %}
import time
import numpy as np
import math
from statistics import *
from rplidar import RPLidar
import RPi.GPIO as GPIO
import board
import busio
import adafruit_pca9685
from adafruit_motor import servo # comes from adafruit-circuitpython-motor library
import curses
from gpiozero import DistanceSensor
from multiprocessing import Process
{% endhighlight %}

To start the driving funciton you will need to connect to motor and the servo on your RC car. Using the very helpful adafruit-circuitpython-motor and adafruit_pca9685 libraries you set the channels you are connected to on the servo driver and tell them what they are controlling using the following code.
{% highlight python %}
  i2c = busio.I2C(board.SCL, board.SDA) # makes sure the GPIO pins are on the correct setting
  pca = adafruit_pca9685.PCA9685(i2c)

  speed_channel = pca.channels[4]
  steer_channel = pca.channels[5]
  pca.frequency = 60

  steer = servo.Servo(steer_channel, min_pulse = 1000, max_pulse = 2000)
  speed = servo.ContinuousServo(speed_channel, min_pulse = 1000, max_pulse = 2000)

  steer.angle = aligned
  speed.throttle = 0
{% endhighlight %}
  The steer.angle and speed.throttle are variables of the motors that control the angle you are turning and the speed you are at. The steer.angle is set in degrees can be a value usually somewhere between 0-180, but it can be a little different for each car. I found my aligned variable to be about 115 and it could turn 45-50 to either side. The speed is from -1 to 1 where -1 is fully reverse, 0 is stop, and 1 is full forward. Note that the car will not go directly from forward motion to backward motion, **you must set speed to a negative value, then to 0, then to the negative value you wish it to go in order to reverse.** This is thanks to the ESC protecting the motor from damage should you try to go full forward into full reverse.
Note that there is no built-in key function for the Pi (input() doesn't work on a Pi) so you will need to use the **curses** library to take key input. I recommend using a simple WASD key set-up for convention. You will also need to connec See the following code for an example.
{% highlight python %}
stdscr = curses.initscr()
curses.noecho()
curses.cbreak()
stdscr.nodelay(True) #no need to press enter to continue

while True:
    press = stdscr.getch() #gets a new singular keypress converted to ASCII
    if press == 27: #ESC
        #stop program
    elif press == 97: #a
        #turn left
        if steer.angle<170:
            steer.angle += 5
    elif press == 100: #d
        #turn right
        if steer.angle>55:
            steer.angle -= 5
    elif press == 115: #s
        #decrease speed
        if speed.throttle >= -0.5:
            speed.throttle -=.05
    elif press == 119: #w
        #increase speed
        if speed.throttle <= 0.5:
            speed.throttle +=.05
    elif press == 32: #space
        #stop motion
        stdscr.addstr(cPos,0,"Stopping motor and realigning wheels")
            speed.throttle = 0
{% endhighlight %}

You should now have an a keyboard controlled RC car!

The next bit of this will be explained at a very high level.

Now for the autonomous feature. I connected up the LiDAR unit and used the rplidar library to control it. I separated the surrounding of the car into six sections, each 60 degrees in size. The 0th sector was from 330 to 30 degrees where 0 degrees is directly in front of the car. I calculated safe spaces around the RC car and based on how much space was left on either side of the car and set the car to turn an amount based off of the distance on either side of the car. This is the basis of the autonomous driving function.

I also used the range detector by placing it on the front of the car and it it detected a large spike in distance, it would stop the car before it fell off of a ledge. I then used multiprocessing to place the auto-driving function and ledge detection into separate processes so that they could run and report information at the same time, increasing response times.

That's basically the whole project. I used a camera and the following terminal commands to display video to a local IP. Then I used VLC media player and its stream function to watch the video at http//LOCAL_IP:8160. [This video](www.youtube.com/watch?v=JjPsW-7FUng.) provides a more in depth solution.
`code(
  raspivid -o - -t 0 -hf -vf -w 800 -h 400 -fps 24 |cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux=ts,dst=:8160}' :demux=h264
  )`

 The project was of mixed success. It did well in scenarios where there were many defined walls around it, but failed in scenarios with small obstacles like chair and table legs.

Updated: 3/6/2019
