---
title: Colonoscopy Robot
permalink: /projects/colonoscopy/
order: 3
summary: >-
  A 4-axis robotic colonoscopy platform with teleoperation, electromagnetic
  tip tracking, and an integrated operator interface.
stack: ROS, Jetson Nano, C++, Python, Dynamixel, NDI Aurora, Tailscale
role: Software and controls, ARTS Lab
status: Completed
---

I developed the control and sensing software for a robotic colonoscopy platform that motorizes the primary motions used during scope navigation.

The system uses Dynamixel actuators controlled by a Jetson Nano, with ROS coordinating teleoperation, electromagnetic tip tracking, remote operation, and the live operator interface.

<figure>
  <img src="{{ '/assets/images/colonoscope_system.jpg' | relative_url }}"
       alt="Schematic of the robotic colonoscope, including the control-handle mechanism, feeder mechanism, Xbox controller, and operator display."
       loading="lazy" width="1280" height="720">
  <figcaption>
    Robotic colonoscope system. Figure from M. R. Javazm et al.,
    “Analytical Design and Development of a Modular and Intuitive Framework for
    Robotizing and Enhancing the Existing Colonoscopy Procedures,”
    <em>Journal of Medical Devices</em>, 2026.
  </figcaption>
</figure>

## Control and Teleoperation

The robot uses four motorized axes driven through Dynamixel actuators. I mapped Xbox controller inputs to robot motion commands and integrated actuation and system state through ROS.

This gave the operator direct control over the robot's steering and scope motion from a handheld controller rather than manually manipulating the colonoscope handle.

I also configured remote access using Tailscale and tested teleoperation outside the local lab network. This allowed remote operation while preserving access to the live camera feed and robot state.

<figure>
  <img src="{{ '/assets/images/colon_setup.jpg' | relative_url }}"
       alt="Colonoscopy robot test setup with the robotic actuation system, phantom colon, NDI Aurora field generator, and tip camera."
       loading="lazy" width="816" height="460">
  <figcaption>Experimental setup with robotic actuation, the colon phantom, electromagnetic tracking, and the live scope camera.</figcaption>
</figure>

## Tip Tracking

I integrated an NDI Aurora electromagnetic tracking system to measure the position and orientation of the colonoscope tip during navigation.

The tracker provides an external estimate of the scope's motion inside the phantom, where the camera image alone does not provide the tip's global position. Tracking data can be recorded alongside the experimental data to reconstruct and analyze the path taken during a trial.

## Operator Interface

I integrated the live camera feed and robot controls into a common operator view so that navigation and system monitoring could be handled from the same interface.

The project also included a tumor-detection model developed by another researcher. My contribution was integrating its detection output with the live endoscopic video so the results could be displayed directly in the operator interface during experiments.
