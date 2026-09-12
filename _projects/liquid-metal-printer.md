---
title: Liquid Metal 3D Printer
permalink: /projects/3d-printing/
order: 2
summary: >-
  A modified FDM platform for direct-write fabrication of liquid-metal
  strain sensors, with custom ROS 2 controls, registration, and automated testing.
stack: ROS 2, Python, wxPython, C/C++ firmware, Prusa i3 MK3S+, Nordson Ultimus V, Dynamixel
role: Software, controls, and test automation
status: Research project, ARTS Lab
---

I developed the software and control stack for a modified Prusa i3 MK3S+ used to fabricate stretchable liquid-metal strain sensors. The system deposits conductive gallium traces directly onto silicone, allowing sensor geometries to be changed through software rather than requiring new mold tooling for each design.

My work included the ROS 2 control architecture, printer and pneumatic dispenser integration, operator interface, mold registration, firmware modifications, process calibration, and automated sensor characterization.

<figure>
  <img src="{{ '/assets/images/system_diagram.jpg' | relative_url }}"
       alt="The liquid-metal printing system, including the modified Prusa printer, syringe toolhead, and pneumatic dispenser."
       loading="lazy" width="1102" height="533">
</figure>

## Control Architecture

I designed a ROS 2 architecture that separates printer motion, pneumatic dispensing, and user control into independent nodes. This keeps hardware communication and timing out of the GUI and allows each subsystem to run without blocking the others.

The printer node owns the Prusa serial connection, executes G-code, applies registration transforms, and handles printer responses through a dedicated reader thread. The dispenser node communicates with the Nordson Ultimus V and manages pressure, vacuum, and dispensing commands. A wxPython interface communicates with both through ROS topics and services.

<figure>
  <img src="{{ '/assets/images/ros_diagram.png' | relative_url }}"
       alt="ROS 2 architecture separating printer motion, pneumatic dispensing, and the operator interface."
       loading="lazy" width="810" height="454">
  <figcaption>Printer motion, pneumatic dispensing, and user control run as separate ROS 2 nodes.</figcaption>
</figure>

One challenge was synchronizing material flow with printer motion. Serial commands to the pneumatic dispenser introduced enough delay that extrusion could begin after the nozzle had already started moving. I moved the timing-critical start and stop control onto the printer itself using a hardware trigger driven directly from the G-code, allowing dispensing to stay synchronized with the motion sequence.

## Registration and Operator Interface

I built a wxPython interface for jogging and homing the printer, controlling the dispenser, adjusting process parameters, loading toolpaths, handling emergency stops, and previewing prints before execution.

I also developed a registration workflow for aligning generated toolpaths with the physical silicone mold. The operator records four reference points and a starting position, and the software computes the coordinate transformation used during printing. Registration profiles can be saved and reused between setups.

This reduced setup time by approximately 70%.

<figure>
  <img src="{{ '/assets/images/gui.png' | relative_url }}"
       alt="The wxPython operator interface for the liquid-metal 3D printer."
       loading="lazy" width="1420" height="818">
  <figcaption>The interface combines printer controls, process settings, registration, and transformed toolpath preview.</figcaption>
</figure>

## Firmware and Process Calibration

The original Prusa firmware was designed for thermoplastic extrusion, so I modified the Marlin C/C++ firmware and machine configuration for syringe-based gallium printing. This included integrating the custom hardware, adapting temperature control and safety behavior, and tuning motion parameters for the heavier toolhead.

Print quality depends on feed rate, pneumatic pressure, nozzle height, and dispensing timing. I developed calibration patterns and parameter sweeps to identify operating conditions that produced continuous traces without excessive pooling or breaks.

I also adjusted the printing sequence and motion profiles to reduce artifacts at starts, stops, and layer transitions.

<figure>
  <img src="{{ '/assets/images/sensor_samples.jpg' | relative_url }}"
       alt="Liquid-metal traces produced during a pneumatic pressure sweep."
       loading="lazy" width="1164" height="1455">
  <figcaption>Pressure sweeps were used to identify the operating range for continuous liquid-metal deposition.</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/images/completed_sensor.jpg' | relative_url }}"
       alt="A completed stretchable liquid-metal strain sensor."
       loading="lazy" width="1024" height="1024">
  <figcaption>A completed liquid-metal strain sensor after printing and encapsulation.</figcaption>
</figure>

## Automated Sensor Characterization

I developed Python tooling to automate testing of the fabricated sensors. A Dynamixel-driven test rig applies repeatable displacement profiles while force and electrical resistance are recorded together.

The automated workflow replaces manual testing and makes repeated trials easier to compare by synchronizing the mechanical and electrical measurements on a common timeline.

<figure>
  <img src="{{ '/assets/images/experimental_setup.jpg' | relative_url }}"
       alt="Automated test rig used to stretch liquid-metal strain sensors while recording force and resistance."
       loading="lazy" width="1024" height="1024">
  <figcaption>A Dynamixel actuator applies controlled displacement while force and resistance are recorded together.</figcaption>
</figure>

## Application

The printed strain sensors are used in soft tactile sensing devices for robotic surgery research. One application integrates the sensors into inflatable silicone structures mounted on a colonoscope, where deformation of the liquid-metal traces can be used to measure mechanical response as the device contacts tissue.

Direct-write fabrication makes it easier to iterate on sensor geometry because changes can be made in the toolpath instead of requiring new fabrication tooling for every design.

<figure>
  <img src="{{ '/assets/images/sensor_diagram.jpg' | relative_url }}"
       alt="Concept showing inflatable tactile sensing structures integrated with a colonoscope."
       loading="lazy" width="660" height="295">
  <figcaption>Example application of the fabricated strain sensors in a soft tactile sensing system.</figcaption>
</figure>
