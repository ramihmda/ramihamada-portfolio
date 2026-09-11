---
title: Colonoscopy Robot
permalink: /projects/colonoscopy/
order: 3
summary: >-
  A 4-axis robotic colonoscopy platform built to take physical effort out of
  scope navigation. I built the teleoperation, the tip tracking, and the
  operator's live view.
stack: ROS, Jetson Nano, C++, Python, Dynamixel, NDI Aurora, Tailscale
role: Software, ARTS Lab
status: Completed
---

Driving a colonoscope by hand is physically demanding, and the operator is doing
it while also reading what is on the screen. The idea behind this platform is to
motorize the scope's degrees of freedom so that navigation becomes an input
problem rather than a strength problem.

<!-- Figure from the published paper. Add a credit line to the caption
     before this goes in front of anyone who might recognise it. -->
<figure>
  <img src="{{ '/assets/images/colonoscope_system.jpg' | relative_url }}"
       alt="Schematic of the robotic colonoscope: control handle gripping mechanism, feeder mechanism, Xbox controller, and operator screen."
       loading="lazy" width="1280" height="720">
  <figcaption>The platform. A commercial colonoscope keeps its own optics and
  channels. A gripping mechanism on the control handle steers the tip, a feeder
  drives insertion and retraction, and both map to an Xbox controller.</figcaption>
</figure>

## Control and teleoperation

Everything runs on a Jetson Nano. Four motorized degrees of freedom are driven
through a Dynamixel stack, and the operator holds an Xbox controller. I mapped
the controller inputs into the motor commands and used ROS to tie actuation,
sensing, and visualization together.

I also set up Tailscale on the robot and drove it over a remote connection to see
how teleoperation held up outside the lab network. Latency is the thing that
degrades first, and it shows up in how you steer long before it shows up in any
log.

<figure>
  <img src="{{ '/assets/images/colon_setup.jpg' | relative_url }}"
       alt="The colonoscopy robot test setup, with the actuation system, phantom colon, NDI Aurora field generator, and tip camera."
       loading="lazy" width="816" height="460">
  <figcaption>The full bench setup: actuation, the test environment, the Aurora
  field generator, and the scope's tip camera.</figcaption>
</figure>

## Tip tracking

An NDI Aurora electromagnetic tracker reports the position and orientation of the
scope tip in real time, which matters because the tip is inside something opaque
and the camera view alone does not tell you where you are.

Tracking data is logged alongside the experiment data and the video on the same
timeline, so a run can be replayed afterward and the scope's actual path through
the phantom reconstructed rather than guessed at.

## The operator's view

A miniature camera at the scope tip gives the live view during navigation. I
pulled that stream into the same interface as the controls, so the operator is
not switching attention between two screens while steering.

The tumor-detection model is another researcher's work, not mine. My part was the
plumbing: getting the camera feed and the detections into one view the operator
is already looking at.
