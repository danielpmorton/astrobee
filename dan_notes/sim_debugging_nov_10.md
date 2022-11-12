Simulation notes 11/10

When starting the simulation with their default command
`roslaunch astrobee sim.launch dds:=false robot:=sim_pub rviz:=true`
(after running `source devel/setup.bash` in the shell)
we run into:
```
log file: /home/dan/.ros/log/f5638fce-6193-11ed-b32e-94c691a89e82/spawn_astrobee-6*.log
[ERROR] [1668152372.005909198, 0.816000000] : (ros.graph_localizer) graph_localizer_intializer.cc:113: ResetBiasesFromFile: Failed to read imu bias file.

[imu_calibration-53] process has finished cleanly
log file: /home/dan/.ros/log/f5638fce-6193-11ed-b32e-94c691a89e82/imu_calibration-53*.log
[ERROR] [1668152378.164918281, 6.960000000] : (ros.imu_integration) imu_integrator.cc:59: IntegrateImuMeasurements: End time occurs before first measurement.

[ERROR] [1668152378.164964228, 6.960000000] : (ros.imu_integration) imu_integrator.cc:128: IntegratedPim: Failed to integrate imu measurments.

[ERROR] [1668152378.164990220, 6.960000000] : (ros.graph_localizer) graph_localizer.cc:192: GetCombinedNavState: Failed to create integrated pim.
```

From their sim debugging page, this may be an issue with limited compute power - so we have to run the following two commands in two separate terminal windows (each having sourced `devel/setup.bash`)
`roslaunch astrobee sim.launch default_robot:=false rviz:=true`
`roslaunch astrobee spawn.launch dds:=false robot:=sim_pub`

The first command seems to execute ok, but the second one encounters the following:
```
[ERROR] [1668152711.410356897, 36.856000000] : (ros.states) Failiure occurred when trying to configure streaming LEDs
[spawn_astrobee-2] process has finished cleanly
log file: /home/dan/.ros/log/ac45b4b0-6194-11ed-b32e-94c691a89e82/spawn_astrobee-2*.log
[ERROR] [1668152720.328994727, 43.800000000] : (ros.imu_integration) imu_integrator.cc:59: IntegrateImuMeasurements: End time occurs before first measurement.

[ERROR] [1668152720.329038112, 43.800000000] : (ros.imu_integration) imu_integrator.cc:128: IntegratedPim: Failed to integrate imu measurments.

[ERROR] [1668152720.329063985, 43.800000000] : (ros.graph_localizer) graph_localizer.cc:192: GetCombinedNavState: Failed to create integrated pim.
```
After running these commands, the robot is not visible, but all of the frames are. To make astrobee visible, un-check and then re-check the Debug → / checkbox under the Rviz Displays section in the bottom left corner

At this point, Astrobee seems to be just randomly floating around the ISS at this point – it’s not docked. Alternatively, if we just used the single-line command (the one that caused the low processing power issue), we do see the Astrobee show up on the dock

Tried:
- Move commands
    - Gave error: `simpleMove6DOF command failed! Move goal failed with response: Keep out zone violation`
    - This happened no matter where in the ISS Astrobee was located, or how small the movement was (relative or absolute)
    - Also: simpleMove6DOF command failed! Cannot move when in ready and not stopped.
        - This only happened after I tried resetting the bias but it seems to be in a weird state?
- Resetting bias
    - Tried finding some information on how to initialize the bias, but couldn’t find this
- Resetting EKF
    - This did not seem to do anything to improve the situation


After running `rosrun executive teleop_tool -get_state` this gave
```
Operating State: Ready
Mobility State: Perched
```

So the robot thinks it is perched but it’s floating around the ISS

So, I retried the one-liner and ignored the errors that showed up

`roslaunch astrobee sim.launch dds:=false robot:=sim_pub rviz:=true`

Then, I retried some of the teleop commands
```
rosrun executive teleop_tool -undock
rosrun executive teleop_tool -get_pose
rosrun executive teleop_tool -get_state
rosrun executive teleop_tool -move -relative -pos "1 2 0.5"
```

These were all successful!


Notes 11/11

Running the simulator with Gazebo active (setting the flag `sviz:=true` when executing the main launch command) appears to work well. The world in Gazebo seems to be upside-down, but this is not a significant issue

`gds:=true` failed, because the Ground Data System is not set up. I've cloned the repo but if we want to use this, **we'll need to get the RTI libraries / communication nodes from our Astrobee point of contact**. They do recommend this as the preferred means of teleoperating the robot

Starting the gnc visualizer: Can either use `gviz:=true` when launching the program, or separately run `python src/tools/gnc_visualizer/scripts/visualizer.py  --comm ros` 
Note that the gnc visualizer crashed a few times


All flags from the Running the Sim page. If not specified, enable with `flag:=true`
- pose: Starting pose. ex) `pose:="x y z qx qy qz qw"`
- gds: Starts the Ground Data System
- rviz: Starts RVIZ
- sviz: Starts Gazebo
- gviz: Starts the GNC visualizer
- dds: Starts communication nodes
- speed: Simulation speed multiplier (1 = real time)
- ns: Namespace (for using multiple robots)
- robot: Which robot config file to use (leave this as sim_pub for now)
- default_robot: If you want to launch the world without a robot, set this `false`
- perch: Starts astrobee in a perch-ready position
- world: "iss" (default) or "granite"
- debug: node name to debug, ex) `"executive"`

I've spent a while trying to figure out how they're loading their configs into ROS/Rviz/Gazebo but having a tough time. 
It seems the `simulation.config` file is used to enable the images from the `nav_cam` and `dock_cam`, and you can change these values in the config fairly easily, but figuring out how ROS actually reads and responds to these changes is confusing
- Understanding this will be helpful for figuring out if we can write our own configs in the future with different physics models
- Config files appear to be in Lua format
- Look into the ConfigReader class and where this is used

Reference links:
https://nasa.github.io/astrobee/html/running-the-sim.html
https://nasa.github.io/astrobee/html/sim-issues.html
https://nasa.github.io/astrobee/html/teleop.html


Still need to check out
https://nasa.github.io/astrobee/html/advanced-sim-info.html
https://nasa.github.io/astrobee/html/astrobee-code-style.html
https://nasa.github.io/astrobee/html/conventions.html
https://nasa.github.io/astrobee/html/new_robot.html
https://nasa.github.io/astrobee/html/doc.html
https://nasa.github.io/astrobee/html/release.html
https://nasa.github.io/astrobee/html/managing_debians.html
https://nasa.github.io/astrobee/html/adding_a_command.html
https://nasa.github.io/astrobee/html/using_telemetry.html
https://nasa.github.io/astrobee/html/maintaining_telemetry.html
