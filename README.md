# ar4_ros2_setup_notes

Notes

## Overview
This document outlines the full-day debug and setup process for getting the Annin Robotics AR4 MK3 robot up and running in ROS 2 using the `annin_ar4_driver` package. It includes diagnostic steps, reasoning, workarounds, and configuration edits — all structured to help future users (and my future self) repeat or extend the process efficiently.

---

## ✅ Initial System Details
- **Robot:** AR4 MK3  
- **Platform:** ROS 2 Humble on Ubuntu 22.04  
- **Microcontroller:** Teensy (connected over `/dev/ttyACM0`)  
- **Gripper Control:** Arduino Nano (expected at `/dev/ttyUSB0`)  

---

## ✅ Major Accomplishments

### 1. Confirmed Working Launch Files
- Found `driver.launch.py` as the main launch file.
- Verified location:  
  `/home/ken/ar4_ws/install/annin_ar4_driver/share/annin_ar4_driver/launch/driver.launch.py`

### 2. Adjusted `driver.launch.py` to Add Debugging
Added this line to improve visibility into node startup issues:

```python
arguments=['--ros-args', '--log-level', 'debug']

3. Verified Plugin Loading

Plugin path successfully located and loaded:

/home/ken/ar4_ws/install/annin_ar4_driver/lib/libannin_ar4_driver.so


4. Identified Missing State/Command Interfaces

Log showed:

missing state interfaces:
'joint_1/position' ... 'gripper_jaw2_joint/velocity'

missing command interfaces:
'joint_1/position' ... 'gripper_jaw1_joint/position'

➡️ Led to investigation of URDF and interface configuration issues.

5. Traced URDF/XACRO File Structure

Confirmed proper macro use in:

ar.urdf.xacro

ar.ros2_control.xacro


Joint interfaces were being dynamically loaded from a YAML config file.

6. Discovered YAML Custom Tag Issue

YAML included lines like:

j1_limit_min: !degrees -170

Running:

yaml.safe_load("j1_limit_min: !degrees -170")

Produced:

ConstructorError: could not determine a constructor for the tag '!degrees'

➡️ Broke parsing and caused joint interfaces to fail silently.

7. Verified Hardware Connection

Teensy correctly detected via:

dmesg | grep ttyACM

Confirmed /dev/ttyACM0 was being created and destroyed with USB plug/unplug

Used screen /dev/ttyACM0 115200 to verify connection responded



---

⚠️ Remaining Issues

1. Serial Port Connection Failure

Teensy: Failed to connect to serial port /dev/ttyACM0

Gripper: Failed to connect to serial port /dev/ttyUSB0

Likely requires correct firmware to be flashed


2. Joint Interface Registration Failed

ros2_control_node fails to initialize joint controllers

Cause: !degrees tags not being parsed → invalid or missing joint limits → controller manager can't attach to any interfaces



---

🔜 Next Steps

1. Replace !degrees YAML Tags

Convert to radians manually to ensure compatibility with xacro.load_yaml


2. Confirm or Flash ROS 2-Compatible Firmware to Teensy

Check if provided annin_ar4_driver repo includes a firmware/ directory or .ino file


3. Rebuild Workspace

cd ~/ar4_ws
colcon build --symlink-install
source install/setup.bash

4. Reconnect Serial and Monitor

Use sudo dmesg | grep ttyACM to confirm connection

Use screen /dev/ttyACM0 115200 to test firmware response


5. Retry Launch

ros2 launch annin_ar4_driver driver.launch.py

Confirm no missing interfaces

Confirm controller manager starts and connects
