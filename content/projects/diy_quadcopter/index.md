+++
date = '2025-10-22T16:37:02+01:00'
draft = true
title = 'Diy Quadcopter'
+++

This page documents my attempt at building a quadcopter from scratch. This includes:
- Designing my own PCB in KiCAD
- Programming the ESP32S3 microcontroller on the PCB to interact with the IMU
- Writing a driver for said IMU in C using FreeRTOS
- Implementing a Kalman Filter in C using FreeRTOS to fuse the accelerometer and gyroscope data from the IMU into usable Euler angles

Unfortunately, this project is not complete, as I lost interest in it and switched over to working on my [robot arm]( {{< relref "projects/robot_arm/index.md" >}}). I did however enjoy working on the PCB design, learning about the Kalman Filter, and using FreeRTOS, so I wanted to document my work here.

---
Repository: [Wayfinder](https://github.com/temperancee/wayfinder)

---

Like my robot arm, this project comprises many parts:
- The Kalman filter
- The PCB design
- The (very simple) driver
- FreeRTOS considerations

## The Kalman Filter

Something something $x_n = A_n x_{n+1} + B_n u_n + \epsilon$

The setup is as follows: We want to get an estimate of the roll and pitch of our quadcopter at time step $n$. This will allow us to stabilise the drone in the air using a PID loop (which I never got around to implementing...), which is just a way of making precise and thoughtful changes to our motor outputs to achieve the stability we desire. The problem arises in that the gyroscope in our IMU gives us measurements of the rate of change of each euler angle at a given moment. We can integrate these using Euler integration to get estimates for the angles themselves, but this estimate drifts further and further away from the true angle values over time, due to the Euler integration method summing all of our previous errors. Our accelerometer provides measurements of the accleration in the x, y, and z directions, which we can resolve into euler angles via trigonometry, but these estimates take a while to update... or something. The solution is to fuse the two estimates together in order to produce a more accurate one.

We seek a [Bayes Filter](https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation), a method by which we can integrate new data (from our sensors) into our prior knowledge (what was the state at the previous timestep) in order to calculate our posterior knowledge about the state. Anybody familiar with Bayesian statistics will recognise that Bayes' rule would be perfect for this.

Another point of note is that the state is not something we can observe directly, we can only attempt to measure it via the accelerometer and gyroscope, which are *noisy*, i.e., there is some random error associated with the measurements they give us. In this way, the Kalman Filter is an analogue of a Hidden Markov Model, except our latent variables (the hidden ones, in this case, the roll and pitch) are on a continuous state space, rather than a discrete one as is the case with a HMM (I think). 

Formally, we assume that the 

In this context, our state is the roll and pitch of the quadcopter. These two variables sit together in a 2D vector in our equations. We cannot model the yaw of the quadcopter, because this would require a magnetometer, but fortunately, we can make do with just pitch and yaw for the purposes of making the quadcopter hover and move through the air (which was the original goal for this project).

**Feed you to the soil on the hill where nothing groooooowwwwws**
# HAUNTED MOUND, WHERE I COME FROM


## The PCB design

I designed the PCB for the flight computer in KiCAD. Learning PCB design took a long time. I found the videos by Phil's Lab on YouTube to be an invaluable source of knowledge. I learnt about the importance of trace widths, layers, stitching vias, (just list all the somewhat advanced things you had to learn).

I also revisited some basic electronic knowledge when trying to understand the unidirectional motor circuits I used [picture]. This involved learning about MOSFETs and some basics on inductors (including flyback diodes).

## The driver

The IMU I chose to use for this project did not have a readily available Arduino driver written for it, so I endeavoured to read the data sheet and write a very minimal one myself. This was my first time writing a driver, and I found working at such a low-level of programming to be quite cool. The driver simply implements read and write functions for the IMU's registers, an initialisation function which writes to some configuration registers on start-up, and a FreeRTOS task to read sensor data from the IMU's accelerometer and gyroscope registers, transforming the acceleration and Euler angle rate of change units and updating the IMU struct variable with the new data (via a mutex, to protect the object from being read by another task).

## FreeRTOS

Mutexes, semaphores, notifications, concurrancy so breezy
