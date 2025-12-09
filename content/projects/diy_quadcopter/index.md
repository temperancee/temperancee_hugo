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
We seek a way to measure the roll and pitch angles. We ignore the yaw, as we just want the drone to hover, and don't particularly care which way it faces (and from a more practical standpoint, measuring yaw requires a [magnetometer](https://en.wikipedia.org/wiki/Magnetometer), which are tricky to use, and make the drone more expensive than I need it to be). The sensor we use for this is the Intertial Measurement Unit (IMU), which is a combination of a gyroscope and an accelerometer (more expensive IMUs also include magnetometers, in which case they are called 9-axis IMUs, whereas a gyroscope accelerometer combo is a 6-axis IMU). The gyroscope measures angular _velocity_, and the accelerometer measures acceleration in the x, y, and z directions. These are of course, not what we want, but with a bit of maths we can turn both of these measurements into measurements of roll and pitch.

### Deriving roll and pitch from the accelerometer and gyroscope

Angular velocity is just the rate of change of our Euler angles, so by integrating the angular velocity of roll and pitch, we get our desired angles. We use Euler integration for this, leading us to the expression $$\phi_t = \phi_{t-1} + \phi^\prime_t\Delta t$$ where $\phi_t$ is the roll at time $t$, $\phi^\prime_t$ is the rate of change of the roll at time $t$ (this is what the gyroscope outputs), and $\Delta t$ is the time between each iteration (more on this later). We have the same equation for our pitch, $\theta$.

The accelerometer measurements ...

### The problem


### Hidden Markov Models and the Bayes Filter

Hidden Markov Models (HMM) can be used to model partially observable processes that evolve over time. By this we mean there are some unobserved random variables $X_t$ which we are interested in, but cannot observe, called the _latent_ variables, and observable variables $Y_t$ which depend on $X_t$ called measurement variables. We wish to make inferences about $X_t$ but only have access to $Y_t$, which may be, for example, measurements of $X_t$ corrupted by noise. A key property of the HMM is that $$\mathbb{P}(X_t = x_t | X_{t-1} = x_{t-1}, ..., X_0 = x_0) = \mathbb{P}(X_t = x_t | X_{t-1} = x_{t-1})$$ that is, the probability of being in a particular state at time $t$ depends only on the previous state.

In our case, the $x_t = \begin{pmatrix} \phi & \theta \end{pmatrix}^T$ are our _state_, the variables that characterise the state of our quadcopter (i.e., we only care about roll and pitch, so that is all that is included in the state, but if we cared about yaw, or x, y, z coordinates, those too would feature). We will discuss what exactly we model $y_t$ as in our situation a bit further down the line.

For Hidden Markov Models that model sequential processes, we can predict the next state given our previous state estimate and the current measurement, $y_t$, using the [Bayes Filter](https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation). I am not going to go into details of the derivation, which can be found in Probabilistic Robotics by Sebastian Thrun. The gist is that using a model of how our previous state $x_{t-1} and a control chosen at time $t$, $u_t$, affect $x_t$, we update our beliefs about $x_t$. The control here, $u_t$ is a way that we can influence the state, for example, by increasing the speed of our drones propellers, we can make it move around. The second step in the Bayes Filter is to incorporate the measurement information $y_t$ into our beliefs. Through this process, we can estimate the next state, in our case, our roll and pitch angles.

### The Kalman Filter

What model should we use for how our measurment affects state and how previous state and control affects state? There are many answers to this, but one of the simplest ones is the Kalman Filter. The Kalman Filter assumes a linear relationship between the next state and the previous state and the control, with Gaussian noise. It also assumes a linear relationship between the state and measurment, again with Gaussian noise. This allows us to derive an analytical solution for the update rules given by the Bayes Filter, whereas in almost all other cases, the integrals we have to compute are intractable. 

We specify that $$x_t = A_t x_{t-1} + B_t u_t + \epsilon_t$$

Another point of note is that the state is not something we can observe directly, we can only attempt to measure it via the accelerometer and gyroscope, which are *noisy*, i.e., there is some random error associated with the measurements they give us. In this way, the Kalman Filter is an analogue of a Hidden Markov Model, except our latent variables (the hidden ones, in this case, the roll and pitch) are on a continuous state space, rather than a discrete one as is the case with a HMM (I think). 

Formally, we assume that the

In this context, our state is the roll and pitch of the quadcopter. These two variables sit together in a 2D vector in our equations. We cannot model the yaw of the quadcopter, because this would require a magnetometer, but fortunately, we can make do with just pitch and yaw for the purposes of making the quadcopter hover and move through the air (which was the original goal for this project).


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
