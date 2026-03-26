# Architecture & System Design

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              EVALUATION CONTAINER (aic_eval)             │
│  Gazebo Sim + aic_bringup + aic_engine + aic_controller │
│  + aic_adapter + aic_scoring + Zenoh Router (TCP 7447)  │
└──────────────────────┬──────────────────────────────────┘
                       │ Zenoh (TCP 7447)
┌──────────────────────┴──────────────────────────────────┐
│              MODEL CONTAINER (aic_model)                 │
│        ROS 2 Lifecycle Node + Your Policy               │
└─────────────────────────────────────────────────────────┘
```

## Data Flow
1. Engine sends `/insert_cable` action goal with Task details
2. Model accepts goal, starts policy execution
3. Policy loop: observe -> infer -> command -> repeat
4. Adapter publishes Observation at 20Hz (cameras, F/T, joints, controller state)
5. Policy sends MotionUpdate/JointMotionUpdate commands at 10-30Hz
6. Controller executes impedance control at 500Hz
7. Scoring evaluates performance metrics continuously
8. Policy returns success/failure, engine scores trial

## Package Dependency Graph
```
aic_task_interfaces ──┐
aic_model_interfaces ─┤── aic_model ── aic_example_policies
aic_control_interfaces┘        │
                               ├── Policy (abstract)
                               └── ActionServer(/insert_cable)

aic_engine_interfaces ── aic_engine ── aic_scoring
aic_control_interfaces ── aic_controller
aic_model_interfaces ── aic_adapter
```

## Robot Configuration
- **Robot**: Universal Robots UR5e (6-DOF) + Robotiq Hand-E gripper
- **Sensors**: ATI AXIA80-M20 F/T sensor, 3x Basler acA2440-20gc cameras
- **Controller**: Custom impedance controller (Cartesian + Joint modes)
- **Workspace**: Task board with NIC cards (SFP ports) and SC optical ports

## ROS 2 Lifecycle State Machine
```
                    on_configure (60s max)
  unconfigured ──────────────────────> configured
       ^                                   │
       │ on_cleanup (60s)      on_activate │ (60s max)
       │                                   v
  configured <──────────────────────── active
                on_deactivate (60s)     │
                                        │ (accept tasks here)
                                        v
                                    on_shutdown (60s)
                                        │
                                        v
                                     shutdown
```

## Key Launch Parameters
```bash
# Simulation launch (aic_gz_bringup.launch.py)
ground_truth:=true      # Enable TF frames for training
rviz:=true              # Show RViz visualization
use_sim_time:=true      # Use simulation clock
config_file:=<path>     # Custom trial configuration
```

## Container Communication
- **Middleware**: rmw_zenoh_cpp (not DDS)
- **Router**: Zenoh router in eval container (port 7447)
- **Network**: Docker internal network (no external access)
- **Auth**: Optional ACL with username/password (AIC_EVAL_PASSWD, AIC_MODEL_PASSWD)
