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

## Sensor Fusion and the Kalman Filter

The goal of this section is simply to stabilise the quadcopter in the air, so it can hover. We can think about this as fixing the roll and pitch angles of the quadcopter to 0. Roll, pitch, and yaw angles tell us how much a body in 3D space has been rotated around each of the x, y, and z axes, as in the diagram below:
<!-- diagram -->
We will refer to these angles as Euler angles throughout this article, although technically they are not necessarily the same - see [Euler Angles](https://en.wikipedia.org/wiki/Euler_angles) for more details.
We seek a way to measure the roll and pitch angles. We ignore the yaw, as we just want the drone to hover, and don't particularly care which way it faces (and from a more practical standpoint, measuring yaw requires a [magnetometer](https://en.wikipedia.org/wiki/Magnetometer), which are tricky to use, and make the drone more expensive than I need it to be). The sensor we use for this is the Intertial Measurement Unit (IMU), which is a combination of a gyroscope and an accelerometer (more expensive IMUs also include magnetometers, in which case they are called 9-axis IMUs, whereas a gyroscope and accelerometer combo is a 6-axis IMU). The gyroscope measures angular _velocity_, and the accelerometer measures acceleration in the x, y, and z directions. These are of course, not what we want, but with a bit of maths we can turn both of these measurements into measurements of roll and pitch.

### Deriving roll and pitch from the accelerometer and gyroscope

Angular velocity is just the rate of change of our Euler angles, so by integrating the angular velocity of roll and pitch, we get our desired angles. We use Euler integration for this, leading us to the expression $$\phi_t = \phi_{t-1} + \phi^\prime_t\Delta t$$ where $\phi_t$ is the roll at time $t$, $\phi^\prime_t$ is the rate of change of the roll at time $t$ (this is what the gyroscope outputs), and $\Delta t$ is the time between each iteration (more on this later). We have the same equation for our pitch, $\theta$.

For the accelerometer measurements, we can use trigonometry. The derivations are detailed [here](https://mwrona.com/posts/accel-roll-pitch/). One thing to note about the derivation is that since we don't care about yaw, we essentially treat all yaws of a given orientation as equal, which allows us to simplify the Euler angle rotation matrix formulae and arrive at the form used in the linked article.

### The Kalman Filter

I will not go into detail about how the Kalman Filter works here, but I do have an article about it: [Kalman Filter]( {{< relref "blog/kalman_filter.md" >}}).

In this context, our state is the roll and pitch of the quadcopter. These two variables sit together in a 2D vector in our equations.

Applying the Kalman Filter to the problem of sensor fusion requires a seemingly odd interpretation of the control and measurement aspects of the Kalman Filter. We treat the angular velocity measurements from the gyroscope as controls, and the accelerometer-derived angle estimates as measurements. The idea here is that changing the motor speeds (what we would intuitively call the control in this situation) directly affects the angular velocity, and by setting $A_t = I, B_t = \Delta t I$, the Kalman Filter prediction formula becomes the Euler integration formula, which is why using Euler integration makes so much sense here.

The measurement step proceeds as we would expect, our measurement, $z_t$ is a vector containing the roll and pitch estimates derived from our acceleromenter measurements.

With the theory out of the way, we can turn to practical aspects. I initialised the variance matrices to values I found other people using for quadcopter state estimation and they seemed to work fine. The accelerometer produces measurements more infrequently than the gyroscope, so the prediction step of the Kalman Filter runs every 2ms, and the measurement step every 10ms. These steps are implemented as FreeRTOS tasks and are executed using timers.



## The PCB design

I designed the PCB for the flight computer in KiCAD. Learning PCB design took a long time. I found the videos by Phil's Lab on YouTube to be an invaluable source of knowledge. I learnt about the importance of trace widths, layers, stitching vias, (just list all the somewhat advanced things you had to learn).

I also revisited some basic electronic knowledge when trying to understand the unidirectional motor circuits I used [picture]. This involved learning about MOSFETs and some basics on inductors (including flyback diodes).

## The driver

The IMU I chose to use for this project did not have a readily available Arduino driver written for it, so I endeavoured to read the data sheet and write a very minimal one myself. This was my first time writing a driver, and I found working at such a low-level of programming to be quite cool. The driver simply implements read and write functions for the IMU's registers, an initialisation function which writes to some configuration registers on start-up, and a FreeRTOS task to read sensor data from the IMU's accelerometer and gyroscope registers, transforming the acceleration and Euler angle rate of change units and updating the IMU struct variable with the new data (via a mutex, to protect the object from being read by another task).

## FreeRTOS

In order to have the Kalman Filter and the planned PID loop run at the same time, I decided to use a real time operating system (RTOS) to handle scheduling the different tasks. I chose to use FreeRTOS for its simplicity, as this use case did not require a large suite of libraries like those offered by Zephyr for example. I learnt about mutexes, semaphores, queues, and timers - mutexes and timers especially were key to making sure the angle state was handled correctly, and that the Kalman Filter steps ran at the right time. This series of [videos by Shawn Hymel](https://www.youtube.com/playlist?list=PLUWXFeSM9LNTBX4GoapiTk_Kxs8FOPv0W) made the topic very clear and provided exercises to make sure I understood the material. I also learnt about notifications in FreeRTOS, which I used to replace the binary semaphores that were previously in my code.

# Conclusion

While I abandoned this project in the end it was very fun to work on and taught me more than any previous project I had attempted. I found that I enjoyed working with FreeRTOS and writing low-level drivers, even if it made the programming side of things more complicated. Finally getting the opportunity to learn PCB design properly was great, and I reckon it will serve me well in the future, although I will try my best to avoid needing a bespoke PCB, since it did take a lot of time and effort, and it isn't exactly cheap to get boards produced!

When my next term of university begins at the start of 2026 I'll be working on a self-balancing two-wheeled robot with another student from the Robotics Society at my university. It will require me to use my knowledge of Kalman Filters again, and will give me another opportunity to  implement a PID loop (I promise I'll actually stick with it this time). Expect a post about that a few months from now.

Thanks for reading!
