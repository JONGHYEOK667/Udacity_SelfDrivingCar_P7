[//]: # (Image References)

[image1]: ./capture/capture_Finish.JPG "capture_Finish"
[image2]: ./capture/capture_Normal.JPG "capture_Normal"
[image3]: ./capture/capture_Follow.JPG "capture_Follow"
[image4]: ./capture/capture_laneChangeLeft.JPG "capture_laneChangeLeft"
[image5]: ./capture/capture_laneChangeRight.JPG "capture_laneChangeRight"

# Highway driving

## Overview
---
In this project, we need to implement a path planning algorithms to drive a car on a highway on a simulator provided by Udacity. The simulator sends car telemetry information (car's position and velocity) and sensor fusion information about the rest of the cars in the highway (Ex. car id, velocity, position). It expects a set of points spaced in time at 0.02 seconds representing the car's trajectory. The communication between the simulator and the path planner is done using WebSocket. The path planner uses the uWebSockets WebSocket implementation to handle this communication. Udacity provides a seed project to start from on this project .  


## Rubric points
---  

## Compilation
### The code compiles correctly.  
- Some files are added   
spline.h : It is the Cubbic Spline interpolation implementation for trajectory planning.   
fsm.h    : It it the Finite State Machine implementation for longitudinal / lateral control mode management. 


## Valid Trajectories
### The car is able to drive at least 4.32 miles without incident..
- simulation run more than 4.32 miles without incedint  

![alt text][image1]

### The car drives according to the speed limit.
- clear  

### Max Acceleration and Jerk are not Exceeded.
- clear  

### Car does not have collisions.
- clear   

### The car stays in its lane, except for the time between changing lanes.  
-  clear  

### The car is able to change lanes
- Lane change is performed when the speed of the vehicle ahead is slow and the lane change trajectory is safe.  




## Reflection   
The code is consisted of three parts : 

### Prediction (line 196 - 310)
In this part, the risk of collision with each target is determined. This is used to determine whether each lane is safe.

- `car_ahead` : flag that foward area targets in driving lane is exist.  
- `Tgt_vel_ego` : speed of the target in front of the driving lane (Cruise control target speed)  
- `car_left` : flag that foward area targets in left lane is exist.  
- `car_left_back` : flag that backward area targets in left lane is exist.  
- `car_right` : flag that foward area targets in right lane is exist.  
- `car_right_back` : flag that backward area targets in right lane is exist.



### Mode Manager (line 330 - 357)
In this part, the control mode transition of longitudinal / lateral control using FSM (Finite State Machine) is determined.

1. longitudinal control mode    

| from state | to state | trigger condition | action |
|:--------:|:------:|:---------------:|:----:|    
| Normal | Follow | `car_ahead`  | Des_vel = `Tgt_vel_ego`|
| Follow | Normal | `!car_ahead` | Des_vel = MAX_VEL|
| Normal | Normal | `!car_ahead`  | Des_vel = MAX_VEL|
| Follow | Follow | `car_ahead`  | Des_vel = `Tgt_vel_ego`|


### Normal state
![alt text][image2]
### Follow state
![alt text][image3]

2. lateral control mode    

| from state | to state | trigger condition | action |
|:--------:|:------:|:---------------:|:----:|    
| laneFollow | laneChangeLeft |`car_ahead` && `!car_left` && `!car_left_back` && `!car_lane_change` && Ego_Lane!= LEFT_LANE| lane--, car_lane_change = true |
| laneFollow | laneChangeRight |`car_ahead` && `!car_right` && `!car_right_back` && `!car_lane_change` && Ego_Lane != RIGHT_LANE| lane++, car_lane_change = true|
| laneChangeLeft | laneFollow | car_lane_change && (lane == Ego_Lane) | car_lane_change = false|
| laneChangeRight | laneFollow | car_lane_change && (lane == Ego_Lane) | car_lane_change = false|

### laneChangeLeft state
![alt text][image4]
### laneChangeRight state
![alt text][image5]
### Controller (line 382 - 541)
In the part, The ideas and concepts are taken from the Q&A video.
The trajectories are calculated with the help of splines based on the speed and lane output from the behavior, car coordinates and past path points.
And to make the path smoother, the step of previous path and waypoint for caculating spline was tunned.




 






























# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).  

To run the simulator on Mac/Linux, first make the binary file executable with the following command:
```shell
sudo chmod u+x {simulator_file_name}
```

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

#### The map of the highway is in data/highway_map.txt
Each waypoint in the list contains  [x,y,s,dx,dy] values. x and y are the waypoint's map coordinate position, the s value is the distance along the road to get to that waypoint in meters, the dx and dy values define the unit normal vector pointing outward of the highway loop.

The highway's waypoints loop around so the frenet s value, distance along the road, goes from 0 to 6945.554.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

Here is the data provided from the Simulator to the C++ Program

#### Main car's localization Data (No Noise)

["x"] The car's x position in map coordinates

["y"] The car's y position in map coordinates

["s"] The car's s position in frenet coordinates

["d"] The car's d position in frenet coordinates

["yaw"] The car's yaw angle in the map

["speed"] The car's speed in MPH

#### Previous path data given to the Planner

//Note: Return the previous list but with processed points removed, can be a nice tool to show how far along
the path has processed since last time. 

["previous_path_x"] The previous list of x points previously given to the simulator

["previous_path_y"] The previous list of y points previously given to the simulator

#### Previous path's end s and d values 

["end_path_s"] The previous list's last point's frenet s value

["end_path_d"] The previous list's last point's frenet d value

#### Sensor Fusion Data, a list of all other car's attributes on the same side of the road. (No Noise)

["sensor_fusion"] A 2d vector of cars and then that car's [car's unique ID, car's x position in map coordinates, car's y position in map coordinates, car's x velocity in m/s, car's y velocity in m/s, car's s position in frenet coordinates, car's d position in frenet coordinates. 

## Details

1. The car uses a perfect controller and will visit every (x,y) point it recieves in the list every .02 seconds. The units for the (x,y) points are in meters and the spacing of the points determines the speed of the car. The vector going from a point to the next point in the list dictates the angle of the car. Acceleration both in the tangential and normal directions is measured along with the jerk, the rate of change of total Acceleration. The (x,y) point paths that the planner recieves should not have a total acceleration that goes over 10 m/s^2, also the jerk should not go over 50 m/s^3. (NOTE: As this is BETA, these requirements might change. Also currently jerk is over a .02 second interval, it would probably be better to average total acceleration over 1 second and measure jerk from that.

2. There will be some latency between the simulator running and the path planner returning a path, with optimized code usually its not very long maybe just 1-3 time steps. During this delay the simulator will continue using points that it was last given, because of this its a good idea to store the last points you have used so you can have a smooth transition. previous_path_x, and previous_path_y can be helpful for this transition since they show the last points given to the simulator controller with the processed points already removed. You would either return a path that extends this previous path or make sure to create a new path that has a smooth transition with this last path.

## Tips

A really helpful resource for doing this project and creating smooth trajectories was using http://kluge.in-chemnitz.de/opensource/spline/, the spline function is in a single hearder file is really easy to use.

---

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!


## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

