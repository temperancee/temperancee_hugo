+++
date = '2025-08-15T9:08:42+01:00'
draft = true
title = 'Robot Arm'
+++

This page documents my work on building a robot arm. At a glance this project covers:
- Designing the arm in FreeCAD and 3D printing it
- Using a Raspberry Pi Pico W to control the servo motors and communicate with the web server
- Creating the web interface to control the robot. Currently, you control angles - I am currently working on adding inverse kinematics and a camera for object detection and motion planning
- Using threading to make use of the Pico's dual cores, and threading on the web server to allow for simultaneous commuication with the Pico and web interface

{{< single_figure path=robot1.jpg width=600 alt="A photo of the robot arm" >}}

## The implementation details

This project can be split into four parts:
- The CAD
- The electronics
- The Pico program
- The web interface

### The CAD

The robot was designed using FreeCAD. I designed all the parts myself. While it certainly works, there are a lot of things I would have done differently if I were to start over:
- Figuring out a better way to manage spreadsheets - I often have to reuse certain values like the measurements of the motors, which means these values get copied over to mutliple sheets
- Similarly, figuring out a more logical way to manage projects - when should I make a new file vs just a new body?

{{< pair_figure path=cad_base.png path2=cad_wrist.png width=400 alt="CAD model of the robot base" alt2="CAD model of the robot wrist" >}}


### The electronics 

The electronics in this project are so simple they're not really worth mentioning - the servo motors are connected directly to PWM pins on a Raspberry Pi Pico W, and the whole project is powered by a 5V wall power supply. All the wiring lives on a breadboard.

### The Pico program

The Pico code is written in Micropython. Both cores on the Pico are utilised, courtesy of the [_thread](https://docs.micropython.org/en/latest/library/_thread.html) library. Servo control is handled by core0 and networking is handled by core1. Servo control is implemented as a callback that activates every time servo angles are sent from the webserver. Long polling is used: The Pico sends a HTTP GET request for angles to the server, and the server responds when they are updated. A better way to do this would to be use Websockets, but I don't have experience with them, and chose to simplify the project for myself with long polling.

### The web interface

The front end of the web interface is an exceedingly basic HTML file, containing only 6 sliders (for angle and claw control) and a few buttons. Maybe I'll add some CSS in the future.

The HTTP server is written in Python, making use of the `ThreadingHTTPServer` class from the [http.server](https://docs.python.org/3/library/http.server.html) library. As mentioned in the last section, long polling is used to send data to the Pico; when the server receives a GET request, it checks for updates to the `angles.json` file using timestamps, and sends angles only when the JSON has been updated.



## The story

Back in September 2024, I began creating my own robot arm. I learnt a lot about CAD, Micropython, gears (although, there aren't any in this robot, bar those within servos), and forward/inverse kinematics.
By the end of December, I had a "working" robot arm (I couldn't get the inverese kinematics to work!). I didn't really have a way to control it though, bar writing motor movements into the code, then editing and reuploading the code. At this point I put the arm aside to work on my quadcopter (**post on that coming soon!!**).

Recently I returned to finish off the project. I was losing interest in my quadcopter and had just seen the trailer for KBot by K-Scale Labs. My interest in robotics has always been tied to the idea of getting robots to do menial house chores for me, lol. Seeing how much development was going on in the area of humanoid robotics made me want to finish my arm project.
I have since created a web interface to control the robot, redesigned the wrist to use a bigger motor, and redesigned the claw to be able to pick up lego figures (a suitable test subject which I happen to have in abundance).

The next step is to finish relearning forward/inverse kinematics. I have revised my knowledge of rotation matrices and homogenous transformations, and am slowly making my way through *Robot Modelling and Control* by Spong, Hutchinson, and Vidyasagar. Following this, I plan to train an object detection model to identify lego figures, and combine the inverse kinematics and computer vision algorithms together, allowing the robot to pick up lego figures and move them around.

