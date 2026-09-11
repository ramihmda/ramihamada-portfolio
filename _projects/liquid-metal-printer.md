---
title: Liquid Metal 3D Printer
permalink: /projects/3d-printing/
order: 2
summary: >-
  A modified FDM printer that draws liquid gallium into silicone to make
  stretchable strain sensors. I wrote the control stack, the firmware changes,
  and the automated test rig.
stack: ROS 2, Python, wxPython, C/C++ firmware, Prusa i3 MK3S+, Nordson Ultimus V, Dynamixel
role: Software and controls
status: Ongoing research, ARTS Lab
---

The lab makes stretchable strain sensors by putting a conductive liquid metal
trace inside a soft silicone body. The old way to do that was injection molding:
build a mold, cure silicone around it, inject gallium. Changing the sensor
geometry meant machining new tooling every time.

Printing the trace directly removes that step. The printer lays gallium onto a
cured silicone substrate, then a second silicone layer seals it in. New geometry
is now a new toolpath instead of a new mold.

I own the software side: the ROS 2 architecture, printer and dispenser
synchronization, the operator interface, the firmware modifications, calibration,
and the characterization workflow.

<figure>
  <img src="{{ '/assets/images/system_diagram.jpg' | relative_url }}"
       alt="The physical liquid-metal printing system, showing the printer, syringe toolhead, and pneumatic dispenser."
       loading="lazy" width="1102" height="533">
</figure>

## Control architecture

Motion, pneumatic extrusion, and the interface are three independent ROS 2
nodes. The reason is timing. A print is a continuous G-code stream, and if a
slow dispenser handshake or a UI redraw can block that stream, you get a blob or
a gap in the trace exactly where the stall happened.

- The printer node streams G-code to the Prusa, applies the mold alignment
  transform, and reads serial responses on its own thread so writes never block.
- The dispenser node speaks the Nordson Ultimus V serial protocol, including
  framing, checksums, and the command handshake.
- The GUI node handles jogging, registration, parameter tuning, and emergency
  stop without touching print execution.

<figure>
  <img src="{{ '/assets/images/ros_diagram.png' | relative_url }}"
       alt="ROS 2 node graph separating printer motion, pneumatic extrusion, and the operator interface."
       loading="lazy" width="810" height="454">
  <figcaption>Motion, extrusion, and the interface run as separate nodes so that
  a slow dispenser command cannot stall the G-code stream.</figcaption>
</figure>

Dispenser lag was the first real problem. Serial commands to the Ultimus V
arrived late enough relative to motion that deposition started after the nozzle
had already moved. Rather than chase it in software, I wired a hardware voltage
trigger through the Prusa's fan output and drove it inline from the G-code, so
extrusion starts on the same event as the move.

## Interface and registration

The operator interface is wxPython. It jogs and homes the printer, sends raw
G-code, drives the dispenser, adjusts pressure and vacuum, and previews a
transformed toolpath before anything is committed to real silicone.

Registration used to be the slow part of every session. The workflow now records
four physical reference points, solves for the transform that aligns the
generated path to the mold, and saves it as a profile. Repeat setups reuse the
profile instead of starting over, which cut setup time by about 70%.

<figure>
  <img src="{{ '/assets/images/gui.png' | relative_url }}"
       alt="The wxPython control interface for the liquid metal printer."
       loading="lazy" width="1420" height="818">
  <figcaption>Jog and homing controls, dispenser settings, and a preview of the
  transformed toolpath before anything is committed to silicone.</figcaption>
</figure>

## Firmware and process calibration

Stock Prusa firmware assumes a hot end melting plastic. This toolhead is a heavy
syringe with an external gallium heater, so the C/C++ firmware needed real
changes: new thermal limits and fault handling for a heater that is not a hot
end, lower acceleration and jerk for the added mass, and recalibrated thermistor
tables.

Getting a repeatable trace out of it is a four-way tuning problem between motion
speed, dispense pressure, nozzle height, and pneumatic timing. Push too hard and
gallium pools. Back off too far and the trace breaks. I ran pressure sweeps and
calibration patterns to find the window, and reworked the multilayer toolpaths to
cut out repositioning moves that were leaving start and stop artifacts.

<figure>
  <img src="{{ '/assets/images/sensor_samples.jpg' | relative_url }}"
       alt="A pressure sweep test print showing gallium trace continuity at different dispense pressures."
       loading="lazy" width="1164" height="1455">
  <figcaption>Pressure sweep at fixed feed rate and nozzle height. Low pressure
  breaks the trace, high pressure pools it.</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/images/completed_sensor.jpg' | relative_url }}"
       alt="A finished stretchable liquid metal strain sensor."
       loading="lazy" width="1024" height="1024">
  <figcaption>A finished sensor, printed and encapsulated.</figcaption>
</figure>

## Characterizing the sensors

Once the process was repeatable enough to trust, the bottleneck moved to
measurement. Characterization had been manual, which made it slow and made trials
hard to compare.

Python scripts now drive a Dynamixel test rig through repeatable displacement
profiles and log force and resistance together on a common clock.
Post-processing aligns repeated trials so that a change in electrical resistance
can be read directly against the mechanical deformation that caused it.

<figure>
  <img src="{{ '/assets/images/experimental_setup.jpg' | relative_url }}"
       alt="Test rig used to stretch sensors while logging force and resistance."
       loading="lazy" width="1024" height="1024">
  <figcaption>The characterization rig. A Dynamixel actuator applies a controlled
  stretch while force and resistance are logged in sync.</figcaption>
</figure>

## Where the sensors end up

The fabrication work exists to serve an application. These soft strain sensors go
into inflatable tactile sensing balloons that ride along a colonoscope. Inflated,
a balloon presses against the colon wall, and the strain response registers a
change in stiffness where a polyp sits. Colonoscopy is otherwise a purely visual
procedure, so this is about adding a sense of touch to it.

<!-- Figure from the published paper. Add a credit line to the caption before
     this goes in front of anyone who might recognise it. -->
<figure>
  <img src="{{ '/assets/images/tactile_balloon.jpg' | relative_url }}"
       alt="A colonoscope fitted with two inflatable strain-sensing balloons, one inflated and one deflated, inside a colon with polyps."
       loading="lazy" width="660" height="295">
  <figcaption>Two sensing balloons on a colonoscope, one inflated and one
  deflated. The controllable distal end steers; the balloons do the feeling.</figcaption>
</figure>

That application is also what sets the requirements I was tuning against. Balloon
geometry changes between designs, which is the argument against cutting mold
tooling every time. And a trace has to stay continuous through repeated inflation,
which is what the pressure sweeps and the characterization rig were really
checking for.
