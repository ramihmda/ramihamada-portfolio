---
title: Drone Autonomy Platform
permalink: /projects/drone/
order: 1
summary: >-
  An autonomous quadrotor with onboard visual-inertial localization,
  RGB-D mapping, real-time perception, and PX4 flight-controller integration.
stack: ROS 2, PX4, MAVLink, Jetson Orin Nano, OAK-D W, RTAB-Map, TensorRT, C++, Python
role: Independent project, from part selection through flight testing
status: In progress
---

I designed and built this quadrotor from component selection and mechanical integration through the onboard autonomy software. The platform is based on a Holybro X500 frame with a Pixhawk 6C running PX4, a Jetson Orin Nano companion computer, and an OAK-D W stereo camera.

The system combines onboard visual-inertial localization, RGB-D mapping, real-time object detection, and a low-latency FPV link to QGroundControl. I first brought the aircraft up as a manually flown platform before integrating the autonomy stack, and validated more than 30 minutes of flight time.

<figure>
  <img src="{{ '/assets/images/drone_picture.jpg' | relative_url }}"
       alt="The assembled quadrotor: Holybro X500 frame with the stereo camera mounted below the front and the Jetson enclosure between the plates."
       loading="lazy" width="1600" height="1038">
</figure>

## System Architecture

PX4 handles the low-level flight controls while the Jetson runs the higher-level autonomy stack in ROS 2. I separated vehicle communication, sensing, localization, mapping, perception, and system monitoring into independent packages with hardware-specific interfaces kept behind adapter layers.

The Pixhawk communicates with the Jetson over MAVLink, while the OAK-D W provides stereo imagery, depth, IMU data, visual-inertial odometry, and a dedicated hardware-encoded FPV stream. This keeps the mapping and perception software independent of the underlying camera and flight-controller interfaces.

## Hardware

I selected and integrated the flight controller, companion computer, stereo camera, RC system, telemetry hardware, propulsion, and power system. I also designed the Jetson enclosure and camera mounting hardware in Fusion 360 and fabricated the parts with FDM printing.

<div class="figure-pair">
  <figure>
    <img src="{{ '/assets/images/jetson_case.png' | relative_url }}"
         alt="Printed enclosure for the Jetson Orin Nano, with a lattice shell and a vented top plate."
         loading="lazy" width="1268" height="758">
  </figure>
  <figure>
    <img src="{{ '/assets/images/camera_mount.png' | relative_url }}"
         alt="Printed bracket that clamps the stereo camera to the frame's forward tubes."
         loading="lazy" width="990" height="550">
  </figure>
</div>

## GPS-independent localization

For localization without GPS, I integrated stereo-IMU visual-inertial odometry from the OAK-D W and converted the estimate into the coordinate frame and message format required by PX4. The external vision estimate is fused with PX4's EKF2 estimator to provide the flight controller with GPS-independent position and orientation information.

A major issue during integration was the companion-computer link itself. Sending external odometry at the required rate saturated the original 57,600-baud UART connection, producing packet loss and high MAVLink latency. Increasing the TELEM3 link to 921,600 baud eliminated the receive loss and reduced round-trip latency to a few milliseconds.

After integrating the VIO estimate with EKF2, a 10 m closed-loop test finished with less than 10 cm of endpoint error.

## Mapping and synchronization

RTAB-Map runs onboard using RGB-D data from the OAK-D W and the visual-inertial estimate for local motion tracking.

During testing, I found that the VIO odometry remained current while its corresponding ROS transform could fall behind by as much as 2.45 seconds under mapping load. That timing error caused RTAB-Map to reject otherwise valid sensor data because the image, odometry, and transform timestamps no longer agreed.

I isolated the problem to the TF publication path and replaced it with a ROS 2 bridge that publishes each transform directly from the corresponding odometry sample using the same timestamp. Worst-case observed lag dropped from 2.45 seconds to under 90 milliseconds, and the following test completed a clean 44-node SLAM run without the previous transform extrapolation failures.

## Onboard perception and FPV

The OAK-D W produces a dedicated 1920×1440, 30 fps H.264 FPV stream using its hardware encoder. The Jetson forwards the encoded stream to QGroundControl without decoding and re-encoding it, leaving its GPU available for perception and autonomy workloads.

Object detection runs onboard with an FP16 TensorRT model at 640×480. I also modified QGroundControl to receive the detection output and draw the results directly over the live FPV feed, keeping perception output in the same interface used for vehicle telemetry and flight monitoring.
