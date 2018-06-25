---
layout: post
title:  "Auburn REU"
date:   2018-06-05 18:46:09 -0400
categories: jekyll update
---

![Auburn University](/assets/camp.jpg){: .center width="1000px"}

This post is going to be an on-going page that will detail the experience I have had researching at Auburn University in the field of Smart UAV's. Our research topic originated as making a homing system for a drone that has lost GPS signal. After much reading and discussion, the topic turned to using LiDAR and computer vision for object avoidance. The current focus is the detection and avoidance of dynamic objects.

My partner for this research is [Jack J. Amend](jackjamend.me), feel free to visit his webpage and view his insights on the project.
Currently we are on week 5 of our research. Much has been done since the initial few weeks and I am excited to be sharing this with everyone!

The first major advancement I want to talk about is our use of LiDAR. We are using both the Lidar Lite V3 and RPLidar A2M8. The RPLidar A2M8 is a product by SLAMTEC that gives us a 360 degree view of our surroundings in a 2D plane. The scanner also gives the distance from itself of everything that it detects. We plan to use this LiDAR for control tests of a LiDAR only collision avoidance system using the following code and the rplidar python extension by SkoltechRobotics.

{% highlight python %}
from rplidar import RPLidar
import msvcrt
import time
import math

lidar = RPLidar("\\\\.\\com4")
#finds the com port the lidar is plugged into on a Windows machine
time.sleep(1)
#makes sure the RPLidar has enough time to successfully connect

info = lidar.get_info()
print(info)

health = lidar.get_health()
print(health)
#gets the info and health of the unit we are using, makes sure nothing is wrong

for i, scan in enumerate(lidar.iter_scans()):
    print('%d: Got %d measurements' % (i,len(scan)))
    ar = [0]\*6
    wFlag = False
    for j in scan:
        k = math.floor(j[1]/60)
        if (j[2]<1000):
            #print("Warning: object at %f degrees is too close" % (j[1]))
            ar[k]+= -math.inf
            wFlag =True
        else:
            ar[k]+=j[2]

    if(wFlag):
        print(ar)
        print("Object(s) are too close...")
        fre = max(ar)
        if(fre<1000):
            print("There is nowhere safe to venture, immediate landing to avoid damage...")
            break
        evac = (ar.index(fre)+1)\*60 - 30
        print("Evacuate in the direction of %d degrees" % (evac))

    if(msvcrt.kbhit() and msvcrt.getch() == chr(27).encode()):
        break
#Separates 360 degree view into 6 equivalent sections, if an object is within 1 meter, evade.
#Evade in the direction of the section that has the greatest sum of distances in a 60 degree angle.
#If all there is nowhere to evade, land immediately to avoid damage.

#stops scanning, motor, and connection of the RPLidar so that it safely stops running
lidar.stop()
lidar.stop_motor()
lidar.disconnect()
{% endhighlight %}

The Lidar Lite V3 resulted in a few more problems, but was a great learning experience about different connections and wirings on a breadboard and raspberry pi.
Talk about configuring, troubleshooting with the dog thing, then using the arduino to finally get it... Then we have no real other info about it

Optical flow:

Technical paper link:

Important references to the research:

![]({{ "/assets/NSF.jpg" | absolute_url }}){: .center}

[jack]: www.jackjamend.me
