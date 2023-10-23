# Terrain Particle Filter
This project implements a particle filter to localize a mobile robot in an environment with known terrain. 

The robot is equipped with an Inertial Measurement Unit (IMU) that measures acceleration and angular velocity, and additionally produces a 3D orientation estimate. 

### Prediction Step
Each time a sensor measurement is received, the acceleration and angular velocity measurements are first integrated using the robot kinematics to obtain new pose predictions for each particle. 

### Update Step
Next, virtual orientation measurements for each particle are constructed using the predicted heading and the terrain normal vector. 

Finally, the particle weights are updated by comparing the virtual orientation measurements to the actual orientation measurement received from the IMU. The particle weights are then normalized and resampled to obtain a new set of particles for the next iteration.

### Simulation and Infrastructure
This project is implemented as a ROS package, with simulation in Gazebo. Much of the infrastructure is adapted from the [mrover-ros](https://github.com/umrover/mrover-ros) repository.

This project is a work in progress.