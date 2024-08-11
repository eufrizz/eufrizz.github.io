---
layout: post
title:  "So you've got your robot arm - what now?"
date:   2024-08-11 21:27:24 +10
categories: robotics arm mujoco manipulation inverse kinematics trajectory
comments: true
excerpt: How to start with a robot arm - from first motions in simulation to the real world. Inverse kinematics, trajectories, position control, velocity control, in MuJoCo and on an Ufactory Lite 6.
assets: /assets/images/mujoco-2-real/
preview-image: "/assets/images/mujoco-2-real/robot_arm_mujoco.png"
---

If you find yourself in the fortunate position of having just set up a robot arm in your living room, as I did a couple of months ago, I've created a Jupyter Notebook to get you started with the software side of things.

<div style="text-align: center;">
<video width="75%" autoplay loop playsinline>
  <source src="{{page.assets}}/spin.mp4" type="video/mp4">
</video>
</div>
<div style="text-align: center;" class="caption"> Your first step to an autonomous merry go round</div>



This notebook is designed to get you running some first movements on your robot arm. It will introduce you to the various modes of control in the arm's API, including cartesian and joint control, position and velocity control, inverse kinematics, simulation in MuJoCo, and screw trajectories. This is a condensed version of how I got up to speed on the fundmentals of robot arms, specifically on my Ufactory Lite 6.

__Github repo link:__ [mujoco_2_real](https://github.com/eufrizz/mujoco_2_real/tree/main)

Happy spinning :)


*Any feedback is welcome! Feel free to shoot me an email using the button at the bottom of the page.*
