# Starter Repository Bug Report

Repository: `VARAD-STACK/level01_ros_assignment`  
Target: ROS 2 Humble / Gazebo Classic 11

This report records defects identified by source inspection. Runtime verification is still required on Ubuntu 22.04 with the assignment simulation.

| File | Defect | Correction |
|---|---|---|
| `testbed_bringup/maps/testbed_world.yaml` | The image field referenced `wrong_path_testbed_world.pgm`, which does not match the supplied map image filename. | Point the image field to `testbed_world.pgm`. |
| `testbed_description/urdf/testbed.gazebo` | The IMU plugin's `initial_orientation_as_reference` element had an extra trailing `>`, making the XML malformed. | Remove the extra character. |
| `testbed_gazebo/launch/spawn_playground.launch.py` | The launch file declared a `world` argument but did not forward it to `gazebo.launch.py`. Gazebo could therefore start without the intended testbed world. | Forward `world` through `launch_arguments`. |
| `testbed_bringup/CMakeLists.txt` | The `maps` directory was not installed with the package. The map YAML and image would be unavailable from the installed package share directory. | Install both `launch` and `maps`. |
| `testbed_gazebo/CMakeLists.txt` | The `models` directory was used to build `GAZEBO_MODEL_PATH` but was not installed. | Install `models` with `launch` and `worlds`. |
| `testbed_description/urdf/testbed.gazebo` | The robot declared the ROS 1 `libgazebo_ros_control.so` plugin although the simulation uses the Gazebo ROS 2 differential-drive plugin directly and no ros2_control controller setup is provided. | Remove the unused ROS 1 control plugin to avoid a missing/incompatible plugin load. |

## Verification status

Static source inspection only. Build, XML/xacro processing, Gazebo startup, map loading, AMCL localization, and Nav2 goal execution must be tested on the target ROS 2 Humble computer. Add any additional runtime-discovered defects here with the observed error and the exact correction.
