# AMR Robot ROS 2 Jazzy - Project Context

## Project
This is a ROS 2 Jazzy AMR simulation project running on an NVIDIA Jetson.

Workspace:
~/ros2_rey

Main package:
~/ros2_rey/src/amr_robot

The goal is to build and debug a differential-drive AMR simulation as preparation for a future real robot.

## Environment

- OS: NVIDIA Jetson Linux
- Architecture: aarch64
- ROS 2: Jazzy
- Gazebo Sim: 8.15.0
- gz_ros2_control: 1.2.20
- ros_gz_sim installed
- ros_gz_bridge installed

## Robot

Robot name:
amr_robot

Base:
- base_link
- dimensions: 0.6 x 0.4 x 0.2 m

Wheels:
- left_wheel_joint
- right_wheel_joint
- wheel radius: 0.10 m
- wheel separation: 0.44 m

The robot uses differential drive.

## ROS 2 Control

The Xacro contains:

ros2_control name:
GazeboSystem

Hardware plugin:
gz_ros2_control/GazeboSimSystem

Both wheels expose:

Command:
- velocity

State:
- position
- velocity

## Controller

Controller:
diff_drive_controller/DiffDriveController

Current controller configuration:

- left_wheel_names: left_wheel_joint
- right_wheel_names: right_wheel_joint
- wheel_separation: 0.44
- wheel_radius: 0.10
- publish_rate: 50
- base_frame_id: base_link
- odom_frame_id: odom
- enable_odom_tf: true
- cmd_vel_timeout: 0.5

Important:
The installed Jazzy diff_drive_controller does NOT expose
use_stamped_vel according to:

ros2 param list /diff_drive_controller

Do not assume unsupported parameters exist.

## Current Problem

Gazebo successfully loads gz_ros2_control.

Controller manager successfully initializes:

GazeboSystem
left_wheel_joint
right_wheel_joint

Hardware activation succeeds.

Current hardware interfaces:

command:
left_wheel_joint/velocity [available] [unclaimed]
right_wheel_joint/velocity [available] [unclaimed]

state:
left_wheel_joint/position
left_wheel_joint/velocity
right_wheel_joint/position
right_wheel_joint/velocity

diff_drive_controller can be loaded.

Controller can be configured to inactive.

Controller activation fails:

Switch controller timed out after 5 seconds.

Direct service call with 10 seconds also fails:

Switch controller timed out after 10 seconds.

Current controller state:
inactive

## Important Discovery

Gazebo world statistics currently show:

paused: true

The output from:

gz topic -e -t /world/amr_world/stats

repeatedly showed:

paused: true

sim_time was empty/not advancing.

Gazebo launch log also contains:

[gz_ros_control]: Desired controller update period (0.02 s) is slower than the gazebo simulation period (0 s).

This strongly suggests the Gazebo simulation is paused or not stepping.

## Previous Checks

ROS /clock exists and is bridged:

/clock

ros2 topic hz /clock previously showed messages.

Controller manager node exists:

/controller_manager

diff_drive_controller node exists:

/diff_drive_controller

Controller manager services exist, including:

/controller_manager/list_controllers
/controller_manager/switch_controller

open_loop:
false

position_feedback:
true

## Important Debugging Rules

Do NOT rewrite the project.

Do NOT change multiple configuration files at once.

Do NOT randomly modify controller parameters.

First verify whether Gazebo simulation is paused.

Then unpause Gazebo and verify:

gz topic -e -t /world/amr_world/stats

shows:

paused: false

and simulation time advances.

Only after simulation is confirmed running should controller activation be tested again.

When debugging, explain the reason for each change and make one change at a time.

Prefer inspecting the existing files before editing them.

Do not assume Humble behavior is identical to Jazzy.

## Current Launch

Launch command:

ros2 launch amr_robot gazebo.launch.py

The launch starts:

- Gazebo
- robot_state_publisher
- ros_gz_sim create
- ros_gz_bridge

The bridge currently includes:

/clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock

## User Preference

The user wants step-by-step debugging.

Do not give a large list of commands at once.

After each diagnostic step, wait for the result before proceeding.

Explain what the output means.
