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
- The PCB design
- The (very simple) driver
- The Kalman filter
- FreeRTOS considerations

## The Kalman Filter

Something something $x_n = A_n x_{n+1} + B_n u_n + \epsilon$

The setup is as follows: We want to get an estimate of the roll and pitch of our quadcopter at time step $n$. This will allow us to stabilise the drone in the air using a PID loop (which I never got around to implementing...), which is just a way of making precise and thoughtful changes to our motor outputs to achieve the stability we desire. The problem arises in that the gyroscope in our IMU gives us measurements of the rate of change of each euler angle at a given moment. We can integrate these using Euler integration to get estimates for the angles themselves, but this estimate drifts further and further away from the true angle values over time, due to the Euler integration method summing all of our previous errors. Our accelerometer provides measurements of the accleration in the x, y, and z directions, which we can resolve into euler angles via trigonometry, but these estimates take a while to update... or something. The solution is to fuse the two estimates together in order to produce a more accurate one.

We seek a [Bayes Filter](https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation), a method by which we can integrate new data (from our sensors) into our prior knowledge (about the state) in order to calculate our posterior knowledge about the state. Anybody familiar with Bayesian statistics will recognise that Bayes' rule would be perfect for this. We also construct a hidden Markov model - the actual state is hidden to us, but we can observe it through noisy measurements (from our sensors).

In this context, our state is the roll and pitch of the quadcopter. These two variables sit together in a 2D vector in our equations. We cannot model the yaw of the quadcopter, because this would require a magnetometer, but fortunately, we can make do with just pitch and yaw for the purposes of making the quadcopter hover and move through the air (which was the original goal for this project).

On the hill where nothing grows




