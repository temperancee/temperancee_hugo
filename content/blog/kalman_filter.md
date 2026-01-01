+++
date = '2026-01-01T16:20:26Z'
draft = true
title = 'Kalman Filter'
+++

# Hidden Markov Models and the Bayes Filter

Hidden Markov Models (HMM) can be used to model partially observable processes that evolve over time. By this we mean there are some unobserved random variables $X_t$ which we are interested in, but cannot observe, called the _latent_ variables, and observable variables $Y_t$ which depend on $X_t$ called measurement variables. We wish to make inferences about $X_t$ but only have access to $Y_t$, which may be, for example, measurements of $X_t$ corrupted by noise. A key property of the HMM is that $$\mathbb{P}(X_t = x_t | X_{t-1} = x_{t-1}, ..., X_0 = x_0) = \mathbb{P}(X_t = x_t | X_{t-1} = x_{t-1})$$ that is, the probability of being in a particular state at time $t$ depends only on the previous state.

In our case, the $x_t = \begin{pmatrix} \phi & \theta \end{pmatrix}^T$ are our _state_, the variables that characterise the state of our quadcopter (i.e., we only care about roll and pitch, so that is all that is included in the state, but if we cared about yaw, or x, y, z coordinates, those too would feature). We will discuss what exactly we model $y_t$ as in our situation a bit further down the line.

For Hidden Markov Models that model sequential processes, we can predict the next state given our previous state estimate and the current measurement, $y_t$, using the [Bayes Filter](https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation). I am not going to go into details of the derivation, which can be found in Probabilistic Robotics by Sebastian Thrun. The gist is that using a model of how our previous state $x_{t-1}$ and a control chosen at time $t$, $u_t$, affect $x_t$, we update our beliefs about $x_t$. The control here, $u_t$ is a way that we can influence the state, for example, if we were controlling a quadcopter, we could increase the speed of the propellers to make it move around. The second step in the Bayes Filter is to incorporate the measurement information $y_t$ into our beliefs. Through this process, we can estimate the next state, in our case, our roll and pitch angles.

# The Kalman Filter

What model should we use for how our measurment affects state and how previous state and control affects state? There are many answers to this, but one of the simplest ones is the Kalman Filter. The Kalman Filter assumes a linear relationship between the next state and the previous state and the control, with Gaussian noise. It also assumes a linear relationship between the state and measurment, again with Gaussian noise. This allows us to derive an analytical solution for the update rules given by the Bayes Filter, whereas in almost all other cases, the integrals we have to compute are intractable. 

We specify that $$x_t = A_t x_{t-1} + B_t u_t + \epsilon_t$$

Another point of note is that the state is not something we can observe directly, we can only attempt to measure it via the accelerometer and gyroscope, which are *noisy*, i.e., there is some random error associated with the measurements they give us. In this way, the Kalman Filter is an analogue of a Hidden Markov Model, except our latent variables (the hidden ones, in this case, the roll and pitch) are on a continuous state space, rather than a discrete one as is the case with a HMM (I think). 

Formally, we assume that the
