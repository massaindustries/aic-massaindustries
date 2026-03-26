# AIC API Conventions & Interface Reference

## ROS 2 Topics (Subscribe - Sensor Data)

| Topic | Type | Rate | Description |
|-------|------|------|-------------|
| `/left_camera/image` | sensor_msgs/Image | 20Hz | Left wrist camera (1152x1024 RGB) |
| `/center_camera/image` | sensor_msgs/Image | 20Hz | Center wrist camera |
| `/right_camera/image` | sensor_msgs/Image | 20Hz | Right wrist camera |
| `/fts_broadcaster/wrench` | geometry_msgs/WrenchStamped | 20Hz | 6D force/torque at TCP |
| `/joint_states` | sensor_msgs/JointState | 20Hz | 7 joint angles |
| `/gripper_state` | sensor_msgs/JointState | 20Hz | Gripper finger position |
| `/aic_controller/controller_state` | ControllerState | 20Hz | TCP pose, velocity, error |
| `/tf`, `/tf_static` | tf2_msgs | varies | Transform frames |

## ROS 2 Topics (Publish - Commands)

| Topic | Type | Rate | Description |
|-------|------|------|-------------|
| `/aic_controller/pose_commands` | MotionUpdate | 10-30Hz | Cartesian pose/velocity targets |
| `/aic_controller/joint_commands` | JointMotionUpdate | 10-30Hz | Joint position/velocity targets |

## ROS 2 Services

| Service | Type | Description |
|---------|------|-------------|
| `/aic_controller/change_target_mode` | ChangeTargetMode | Switch Cartesian(1)/Joint(2) |
| `/aic_controller/tare_force_torque_sensor` | (disabled in eval) | Reset F/T baseline |

## ROS 2 Action Servers (Implement)

| Action | Type | Description |
|--------|------|-------------|
| `/insert_cable` | InsertCable | Main task - receives Task, returns success |

## Key Message Structures

### Task (goal)
- `id`: Unique task identifier
- `cable_type`, `cable_name`: Cable specification
- `plug_type`, `plug_name`: Plug on cable end
- `port_type`, `port_name`: Target port
- `target_module_name`: Module containing target port
- `time_limit`: Max seconds for completion

### MotionUpdate (Cartesian commands)
- `header.frame_id`: "base_link" or "gripper/tcp"
- `pose`: Target position + orientation (MODE_POSITION)
- `velocity`: Target twist (MODE_VELOCITY)
- `target_stiffness`: 6x6 matrix (row-major float64[36])
- `target_damping`: 6x6 matrix (row-major float64[36])
- `trajectory_generation_mode`: MODE_POSITION(2) or MODE_VELOCITY(1)

### Observation (sensor snapshot)
- `left_image`, `center_image`, `right_image`: RGB camera images
- `left_camera_info`, etc.: Camera calibration
- `wrist_wrench`: WrenchStamped (force xyz + torque xyz)
- `joint_states`: JointState (positions, velocities, efforts)
- `controller_state`: ControllerState (TCP pose/vel/error)

### Default Stiffness/Damping Values
```python
# From Policy.set_pose_target() defaults:
stiffness = [2000.0, 2000.0, 2000.0, 200.0, 200.0, 200.0]  # [x,y,z,rx,ry,rz]
damping = [100.0, 100.0, 100.0, 20.0, 20.0, 20.0]
```

## Control Architecture
```
Commands (10-30Hz) -> Clamping -> Interpolation -> Impedance Control (500Hz) -> Robot
```

- Cartesian: tau = J^T [K_p(x_des - x) + K_d(xdot_des - xdot) + W_f] + tau_null
- Joint: tau = K_p(q_des - q) + K_d(qdot_des - qdot) + tau_f

## Connector Types
- **SFP_MODULE** plugs into **SFP_PORT** (on NIC cards, Zone 1)
- **SC_PLUG** plugs into **SC_PORT** (Zone 2)
- **LC_PLUG** plugs into **LC_PORT** (future phases)

## Task Board Layout
- Zone 1: NIC cards with SFP ports (5 rails, cards translate 0-62mm, rotate +/-10deg)
- Zone 2: SC optical ports (2 rails, translate 0-115mm)
- Zone 3: SFP module pick fixtures (translate 0-188mm, rotate +/-60deg)
- Zone 4: SC plug pick fixtures (translate 0-188mm, rotate +/-60deg)
