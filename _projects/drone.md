---
title: Drone Autonomy Platform
permalink: /projects/drone/
order: 1
summary: >-
  A quadrotor that localizes and maps without GPS. Stereo visual-inertial
  odometry runs onboard and feeds the flight controller as external vision.
stack: ROS 2, PX4, MAVLink, Jetson Orin Nano, OAK-D W, RTAB-Map, TensorRT, C++, Python
role: Solo build, from part selection through flight test
status: In progress
---

This one is mine start to finish. I chose the parts, designed the mounts,
assembled the aircraft, and wrote the software.

The airframe is a Holybro X500 carrying a Pixhawk 6C on PX4, a Jetson Orin Nano
as the companion computer, and an OAK-D W stereo camera. PX4 owns attitude and
rate control. Everything above that, meaning sensing, localization, mapping, and
the link back down to the flight controller, runs in ROS 2 on the Jetson.

Most of the hard problems so far have been transport and timing, not algorithms.
That surprised me less than it probably should have.

## Architecture

The onboard stack is split into separate ROS 2 components for sensing,
localization, mapping, vehicle communication, and health monitoring. The split is
practical rather than architectural: when mapping falls over mid-test, I can
restart that one node instead of landing and rebooting the whole aircraft.

The Jetson and the Pixhawk talk over MAVLink on a serial link. The OAK-D W
carries its own inference chip and IMU, so it hands up stereo depth, RGB, raw
IMU, visual-inertial odometry, and a separately encoded FPV stream without
loading the Jetson's CPU.

## Hardware

Part selection covered the flight controller, companion computer, stereo camera,
RC receiver, telemetry radio, propulsion, and power. The Jetson and camera mounts
are my own, modeled in Fusion 360 and printed.

I flew it manually and tuned it before adding any autonomy, which turned out to
matter. Getting comfortable with the aircraft as an aircraft made the later
debugging much less nerve-racking. It holds more than 30 minutes on a charge.

## Visual-inertial odometry

For motion estimation without GPS, I put together a DepthAI pipeline that fuses
the OAK-D W's stereo pair with its IMU. The same pipeline also publishes depth,
RGB, and a hardware-encoded video stream.

Before trusting any of it, I checked the coordinate convention by hand: pick up
the drone, move it forward, then left, then up, then yaw it, and watch which
numbers change and in which direction. The axes matched the ROS body frame as
expected. The tests also caught something I would not have found otherwise. The
DepthAI velocity output sat at exactly zero while the aircraft was physically
moving.

So I publish the pose and leave the velocity fields unset rather than filling
them with zeros. An unset field tells the estimator downstream that it has no
measurement. A zero tells it the drone is stationary, which is a much worse lie.
Stable output runs around 25 Hz.

## Getting the estimate into PX4

I wrote the ROS 2 and MAVSDK interface that converts each VIO sample into the
frame and message format PX4 expects, then ships it to the Pixhawk as external
vision odometry.

That integration is where the transport bug showed up. PX4 was reporting packet
loss and round-trip latency in the hundreds of milliseconds, even though the VIO
source upstream was healthy. The odometry stream was saturating the Jetson to
Pixhawk UART, which was still at its default 57,600 baud.

Moving TELEM3 to 921,600 baud fixed it. PX4 started receiving external odometry
at roughly 25 Hz with zero receive loss, and MAVLink ping latency dropped to a
few milliseconds.

### EKF2 fusion

PX4's EKF2 estimator can fuse this external pose alongside its own inertial
sensors. The transport, the frame conversion, and the physical axis checks are
all validated. What is left is camera lever-arm calibration and the fusion
testing itself, which I am working through now. I am not using the estimate for
autonomous position control until that is done and flown.

## SLAM, and a 2.45 second timestamp

Mapping runs on RTAB-Map. I keep the smooth local VIO separate from map-level
loop-closure corrections so that a correction never yanks the control estimate
sideways.

RTAB-Map immediately surfaced a timing problem I had not noticed. The transform
coming out of the VIO pipeline could trail its own odometry message by as much as
2.45 seconds. RTAB-Map would ask for a transform at the odometry timestamp, find
nothing that recent, and throw an extrapolation error.

The fix was a small ROS 2 bridge that republishes the VIO pose stamped with the
odometry's timestamp instead of its own. Worst-case lag went from 2.45 seconds to
under 90 milliseconds, and the next run completed a clean 44-node map with no
extrapolation failures.

## Video and perception

The OAK-D W encodes a 1920x1440, 30 fps H.264 FPV stream on its own silicon, so
the Jetson never has to spend cycles re-encoding video just to get it to the
ground station.

I am currently wiring TensorRT detections into QGroundControl so they draw over
that live view next to the normal vehicle telemetry, rather than in a separate
window nobody is looking at during a flight.

## Where it stands

Next up is finishing EKF2 external-vision fusion and validating GPS-independent
position hold in controlled flight tests.
