---
layout: post
title:  "Auburn REU"
date:   2018-06-05 18:46:09 -0400
categories: jekyll update
---

![Auburn University](/assets/camp.jpg){: .center width="1000px"}

## Introduction: ##

This post is going to be an on-going page that will detail the experience I have had researching at Auburn University in the field of Smart UAV's. Our research topic originated as making a homing system for a drone that has lost GPS signal. After much reading and discussion, the topic turned to using LiDAR and computer vision for object avoidance. The current focus is the detection and avoidance of dynamic objects.

My partner for this research is [Jack J. Amend](http://jackjamend.me), feel free to visit his webpage and view his insights on the project.
Currently we are on week 5 of our research. Much has been done since the initial few weeks and I am excited to be sharing this with everyone!

## Research: ##

### LiDAR:

The first major advancement I want to talk about is our use of LiDAR. We are using both the Lidar Lite V3 and RPLidar A2M8.

![](/assets/rplidar.jpeg)

The RPLidar A2M8 is a product by SLAMTEC that gives us a 360 degree view of our surroundings in a 2D plane. The scanner also gives the distance from itself of everything that it detects. We plan to use this LiDAR for control tests of a LiDAR only collision avoidance system using the following code and the rplidar python extension by SkoltechRobotics.

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

The Lidar Lite V3 resulted in a few more problems, but was a great learning experience. The LiDAR Light V3 looks like so and comes with a six wire output.

![](/assets/lidar-lite-v3.jpeg)

 Only four wires are needed for our usage though. The wires are as follows:

* Red: Power
* Green: I2C SCA connection
* Blue: I2C SDA connection
* Black: Ground

I2C stands for Inter-integrated Connection and it is what the LiDAR uses to send and receive information. We initially tried to connect the LiDAR to a raspberry pi 3 Model B+. When connected to the raspberry pi, we ran a program in python3 to try to see a constant output of distances from the LiDAR. When the program was run, the raspberry pi gave us outputs of 0 for everything. After much troubleshooting with the I2C we were able to figure out that the problem was the version of Raspbian that we were using. Installations after 2016 use the I2C connections in such a way that the LiDAR Lite V3 does not work. After switching to a November 2016 installation of Raspbian, the LiDAR works as expected.

The LiDAR was then used to determine the distance of objects directly in front of the drone and whether or not to start object avoidance protocol. 

### Optical flow:

Optical flow is being used in this project to detect what things are moving in the picture and what needs to be avoided by the drone if something gets too close. It uses Shi Tomase algorithm for good features to track and the Lucas Kanade method for optical flow. We graphed an 8x8 grid on the image and counted how many features were in each block. Then we decided which direction was the safest place to go based on which side of the picture had the least amount of features present.

My partner Jack worked a lot on this part of the research.   

### Experiment:

The experiment is based on the idea that we do not want the drone to have a waypoint or location that it is trying to get to. The only thing that it needs to do is avoid any sort of collision. We encountered problems with the AR Parrot 2.0 that made flying the drone difficult and hazardous to the hardware that we were using (ie. it would randomly fly into the ceiling or take off in directions that we know for certain we were not telling it to). So the data was found while holding the drone and moving a large object in and out of the frame. Everything in the program worked pretty well and the program gave the correct output for moving the drone in different directions and out of the way of detected harm.

There are many things that could be improved upon and we hope to do some more independent work on the program and in a different simulation software to see if we could get some more substantial data. The tracking algorithm that we used could also be changed. It has difficulties tracking any points that are moving because of the blur of the picture. This could be helped by a different camera that still has very clear images of moving objects or by a different algorithm that specifically detects moving objects.

To get a more in-depth understanding of the research, feel free to look at the following work-in-progress paper.    

### Technical paper link:
[LiDAR and Optical Flow Based Collision avoidance for UAVs](/assets/collision-avoidance-system.pdf)

Updated 7/24/2018

## Important references to the research:

_Azevedo, F., Oliveira, A., Dias, A., Almeida, J., Moreira, M., Santos, T., Ferreira, A.,
Martins, A., and Silva, E. (2017). Collision avoidance for safe structure inspection with multirotor uav.
In Mobile Robots (ECMR), 2017 European Conference on, pages 1–7. IEEE._

_Chang, R., Ding, R., and Lin, M. (2017). Optical flow based obstacle avoidance for
multi-rotor aerial vehicles. In 2017 IEEE 29th International Conference on Tools with Artificial Intelligence
(ICTAI), pages 574–578._

_He, F., Du, Z., Liu, X., and Ta, Y. (2010). Laser range finder based moving object tracking
and avoidance in dynamic environment. In Information and Automation (ICIA), 2010 IEEE International
Conference on, pages 2357–2362. IEEE._

_Horvat, D., Kobale, D., Zorec, J., Mongus, D., and Zalik, B. (2016). Assessing the us- ˇ
ability of lidar processing methods on uav data. In Computational Science and Computational Intelligence
(CSCI), 2016 International Conference on, pages 665–670. IEEE._

_Hrabar, S. (2012). An evaluation of stereo and laser-based range sensing for rotorcraft unmanned
aerial vehicle obstacle avoidance. Journal of Field Robotics, 29(2):215–239._

_Lai, Y.-K., Huang, Y.-H., and Hwang, C.-M. (2016). Front moving object detection for car
collision avoidance applications. In Consumer Electronics (ICCE), 2016 IEEE International Conference
on, pages 367–368. IEEE._

_Li, R., Liu, J., Zhang, L., and Hang, Y. (2014). Lidar/mems imu integrated navigation
(slam) method for a small uav in indoor environments. In Inertial Sensors and Systems Symposium (ISS),
2014 DGON, pages 1–15. IEEE._

_Li, X., Li, Z., Fu, B., Wu, B., and Liu, Y. (2017). A mini consumer grade unmanned aerial
vehicle (uav) for small scale terrace detection. In Geoscience and Remote Sensing Symposium (IGARSS),
2017 IEEE International, pages 3349–3352. IEEE._

_Merz, T. and Kendoul, F. (2011). Beyond visual range obstacle avoidance and
infrastructure inspection by an autonomous helicopter. In Intelligent Robots and Systems (IROS), 2011
IEEE/RSJ International Conference on, pages 4953–4960. IEEE._

## Acknowledgements ##

The research was hosted by Auburn University and was funded by the National Science Foundation and the Department of Defense.

![]({{ "/assets/NSF.jpg" | absolute_url }}){: .center} ![](/assets/AU.png){: width = "500px"}
