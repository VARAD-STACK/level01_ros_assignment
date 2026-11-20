# Testbed Navigation — ROS 2 Humble

This package implements the Testbed-T1.0.0 navigation workflow with individual Nav2 lifecycle nodes and plugin servers. It intentionally does **not** use the `nav2_bringup` launch files.

## Target environment

- Ubuntu 22.04
- ROS 2 Humble
- Gazebo Classic 11
- RViz2
- Nav2 packages for Humble

## Package layout

```
testbed_navigation/
├── config/
│   ├── amcl_params.yaml
│   └── nav2_params.yaml
├── launch/
│   ├── map_loader.launch.py
│   ├── localization.launch.py
│   └── navigation.launch.py
├── CMakeLists.txt
└── package.xml
```

## Build

From the assignment workspace:

```bash
cd ~/assignment_ws
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

## Run in separate terminals

### Terminal 1 — simulation

```bash
source /opt/ros/humble/setup.bash
source ~/assignment_ws/install/setup.bash
ros2 launch testbed_bringup testbed_full_bringup.launch.py
```

Wait until Gazebo has loaded the world and the Testbed robot has spawned.

### Terminal 2 — localization

```bash
source /opt/ros/humble/setup.bash
source ~/assignment_ws/install/setup.bash
ros2 launch testbed_navigation localization.launch.py
```

In RViz2, set the Fixed Frame to `map`, add the Map display if it is not already present, then use **2D Pose Estimate** to initialize the robot's pose. The robot is spawned at approximately x=0, y=5 in the supplied world; use the actual robot position visible in Gazebo/RViz rather than blindly accepting this estimate.

For an isolated map-server check, stop localization first, then run:

```bash
ros2 launch testbed_navigation map_loader.launch.py
```

Do not run `map_loader.launch.py` and `localization.launch.py` at the same time: both create a node named `map_server`.

### Terminal 3 — navigation servers

After the map and AMCL are active and the initial pose is set:

```bash
source /opt/ros/humble/setup.bash
source ~/assignment_ws/install/setup.bash
ros2 launch testbed_navigation navigation.launch.py
```

In RViz2, use **2D Nav Goal** to send a goal in free space. The robot should plan around obstacles and drive toward the goal.

## Important interfaces

- Map: `/map`
- Laser scan: `/scan`
- Odometry: `/odom`
- Localization estimate: `/amcl_pose`
- Main navigation action: `/navigate_to_pose`
- TF chain: `map -> odom -> base_footprint -> base_link`

## Diagnostic commands

```bash
ros2 node list
ros2 lifecycle nodes
ros2 topic list
ros2 topic echo /amcl_pose
ros2 topic echo /scan --once
ros2 topic echo /odom --once
ros2 action list
ros2 run tf2_tools view_frames
```

Expected navigation actions include `/navigate_to_pose`, `/compute_path_to_pose`, and `/follow_path`.

## Tuning notes

The supplied simulated LiDAR publishes a LaserScan on `/scan` with a configured range of 0.10–1.50 m. The obstacle-layer ranges and AMCL laser range are therefore kept within that sensor range. The initial DWB velocity limits are deliberately conservative for first simulation tests.

These parameters are a starting configuration, not a claim of verified performance. Confirm scan rate, odometry, TF, map alignment, costmaps, and safe goal locations on the target computer. Adjust the robot radius and planner/controller settings only after inspecting the simulation.

## Testing record

Record the actual Ubuntu/ROS 2 versions, build output, launch results, goal coordinates, and any errors encountered during the friend-computer test. Add genuine screenshots or a short video after successful localization and navigation. No simulation result is claimed until those tests have been run.
