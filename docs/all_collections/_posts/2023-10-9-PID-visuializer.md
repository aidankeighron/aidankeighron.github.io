---
layout: post
title: PID Visuializer
date: 2023-7-29 12:00:00
categories: [java, FRC]
---

# Project Overview

I use PID's a lot in robotics, so I created a visualizer of how a PID works to make it easier to explain. Ill go over what PID's are, how they work, and show the code that I used to visualize them. As A bonus at the end I will talk about enhancing your PID's using a trapezoidal motion profile.

The code shown here has been modified to make it easier to explain. If you would like to check it out, the full code is available on GitHub: [PID Visualizer](https://github.com/aidankeighron/PID-Visualizer){:target="\_blank"}

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#what-is-a-pid-and-where-is-it-used">What is a PID and Where it is Used</a></li>
<li><a href="#how-pid-works">How PID Works</a></li>
<li><a href="#pid-visualizer">PID Visualizer</a></li>
<li><a href="#bonus-taking-pid-to-the-next-level">BONUS Taking PID to the Next Level</a></li>
</ul>
</details>

# What is a PID and Where it is Used

PID or Proportional Integral Derivative is a algorithm for controlling and output. It is a way to realistically get value A to become value B accounting for acceleration and inertia. Looking at a graph of a PID output we can see this in action:

<img src="/assets/icons/pid-visuial/not-enough-Derivative.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 60%;">

The blue line is the target and the red line is the output of the PID algorithm. You can see that the PID output starts out slow and then accelerates to mimic what happens in real world environments. At then end you see the PID gradually slow down to account for inertia. PID's are extremely customizable you can tune a PID to get to the target as quickly as possible, as accurately as possible, or a combination of the two.

PID's a used everywhere from getting your house to your car. They are used in homes to keep a temperature at a certain level. They are used in your car to keep you moving a a specific speed with cruise control. PID's are all around you controlling everything you use.

# How PID Works

First lets go over some terminology:
    Actual - Where your output currently is EX: position of a robotic arm, speed of a car, temperature of a room, etc.
    Setpoint - Where you want your output to be
    Error - `Setpoint - Actual` Difference between where you are and where you want to be

Next lets look at the equation for a PID and break each part down:
$$output = P + I + D$$

### Proportional Term (P):

$$P = kP * error$$

The proportional term is the error multiplied by a constant kP. kP is found by essentially starting with a random value (usually a factor of 10 less that 1) and moving it around until you get a desired response. There are some mathematical ways of tuning a PID but it is usually better to tune it by hand because of the wide range of possible "solutions". The proportional term has the largest overall impact on the output of the PID.

<img src="/assets/icons/pid-visuial/Derivative-too-low.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 60%;">
* Probably too low of a kP term

### Integral Term (I):

$$I = kI * \int_0^t error(T) \,dT$$

The integral term is found be integrating over all of the errors since the PID has started and multiplying it by a constant kI. kI is usually a very small number as a high kI can result is oscillation very high outputs. Because of these problems the integral term is usually capped with a maximum value. The integral term is an extra push to get the actual to reach the setpoint when the error is small enough that the proportional term does not have as much of an effect.

<img src="/assets/icons/pid-visuial/Derivative-too-low.png" style="border:5px solid black;display: block;margin-left: auto;margin-right: auto;width: 60%;">
* Here you can see oscillation within a PID, probably too high of a kI

### Derivative Term (D):
$$\frac{\mathrm{d} error}{\mathrm{d}t}$$

The Derivative term can be found by taking the derivative of the error with respect to time multiplied by kD. This term can be helpful when trying to account for external factors. Because it is taking the derivative it helps dampen large sudden changes in error. The derivative term can cancel out things like a car with cruise control active suddenly going down a hill, or a block of ice being added to a room that is trying to keep a constant temperature.

### Tuning a PID

I talked about this briefly earlier, but there is not a lot of science in tuning a PID. Because each application of a PID has different values they best way to tune one is to mess around with the constants until you get an output you like. I recommend starting with the kP term, then tuning kI, then kD. You might find that you don't even need the D or I terms. Some applications only need P, PI, or PD. I have never seen it used before, but you could probably even find a use for ID.

# PID Visualizer

# BONUS Taking PID to the Next Level

Full code: [PID Visualizer](https://github.com/aidankeighron/PID-Visualizer){:target="\_blank"}