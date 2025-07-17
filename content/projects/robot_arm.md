+++
date = '2025-07-12T17:06:42+01:00'
draft = true
title = 'Robot Arm'
+++

This page documents my work on building a robot arm. At a glance this project covers:
- Designing the arm in FreeCAD and 3D printing it
- Using a Raspberry Pi Pico W to control the servo motors and communicate with the web server
- Creating the web interface to control the robot. Currently, you control angles - in the future I would like to add inverse kinematics and a camera for motion planning
- Using threading to make use of the Pico's dual cores, and threading on the web server to allow for simultaneous commuication with the Pico and web interface

# The implementation details

This project can be split into four parts:
- The CAD
- The electronics
- The Pico program
- The web interface

## The CAD

The robot was designed using FreeCAD. I designed all the parts myself, except the claw (currently). While it certainly works, there are a lot of things I would have done differently if I were to start over.

## The electronics 

The electronics in this project are so simple they're not really worth mentioning - the servo motors are connected directly to PWM pins on the Pico, and the whole project is powered by a 5V power supply. All the wiring lives on a breadboard.

## The pico program

The Pico code is written in Micropython. Both cores on the Pico are utilised, courtesy of the [_thread](https://docs.micropython.org/en/latest/library/_thread.html) library. Servo control is handled by core0 and networking is handled by core1. Servo control is implemented as a callback that activates every time servo angles are sent from the webserver. Long polling is used: The Pico sends a HTTP GET request for angles to the server, and the server responds when they are updated. 

## The web interface

The front end of the web interface is an exceedingly basic HTML file, containing only 6 sliders (for angle control) and a few buttons. Maybe I'll add some CSS in the future.

The HTTP server is written in Python, making use of the `ThreadingHTTPServer` class from the [http.server](https://docs.python.org/3/library/http.server.html) library. As mentioned in the last section, long polling is used to send data to the Pico; when the server receives a GET request, it checks for updates to the `angles.json` file using timestamps, and sends angles only when the JSON has been updated.



# The story

Back in September 2024, I began creating my own robot arm. I learnt a lot about CAD, Micropython, gears (although, there aren't any in this robot, bar those within servos), and forward/inverse kinematics.
By the end of December, I had a "working" robot arm (I couldn't get the inverese kinematics to work!). I didn't really have a way to control it though, bar writing motor movements into the code, then editing and reuploading the code. At this point I put the arm aside to work on my quadcopter (**post on that coming soon!!**).

Recently I returned to finish off the project. I was losing interest in my quadcopter and had just seen the trailer for KBot by K-Scale Labs. My interest in robotics has always been tied to the idea of getting robots to do menial house chores for me lol. Seeing how much development was going on in the area of humanoid robotics made me want to finish my arm project.
I have since created a web interface to control the robot, and am currently redesigning the wrist and claw. Following that, I plan to take another crack at inverse kinematics, and maybe add a camera to the arm, then learn ROS2 to implement some motion planning algorithms.

