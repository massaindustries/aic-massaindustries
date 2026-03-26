# AI for Industry Challenge (AIC) - Repository Guide

## Project Overview

This is the **AI for Industry Challenge (AIC)** repository - a robotics competition for autonomous cable insertion using a UR5e robot arm in simulation. Participants develop AI policies that control a robot to insert fiber optic cables (SFP, SC, LC) into network hardware ports on a task board.

**Competition Timeline**: March 2 - September 8, 2026
**Prize Pool**: $180,000 for top 5 teams

## Repository Architecture

```
aic-massaindustries/
├── aic_model/              # YOUR CODE: ROS 2 Lifecycle node + Policy framework
├── aic_example_policies/   # Reference policies (WaveArm, CheatCode, RunACT)
├── aic_interfaces/         # ROS 2 message/service/action definitions
│   ├── aic_control_interfaces/   # MotionUpdate, JointMotionUpdate, ControllerState
│   ├── aic_model_interfaces/     # Observation message
│   ├── aic_task_interfaces/      # InsertCable action, Task message
│   └── aic_engine_interfaces/    # ResetJoints service
├── aic_engine/             # Trial orchestrator (C++ - provided, do not modify)
├── aic_controller/         # Impedance controller (C++ - provided)
├── aic_adapter/            # Sensor fusion adapter (C++ - provided)
├── aic_scoring/            # Scoring system (C++ - provided)
├── aic_bringup/            # Launch files for simulation
├── aic_gazebo/             # Gazebo plugins
├── aic_description/        # URDF/SDF robot descriptions
├── aic_assets/             # 3D models and simulation assets
├── aic_utils/              # Teleoperation, MuJoCo, LeRobot, Isaac utilities
├── docker/                 # Dockerfile for eval and model containers
└── docs/                   # Comprehensive documentation (18 files)
```

## Tech Stack

- **Languages**: Python (policies, utils), C++ (engine, controller, scoring)
- **Framework**: ROS 2 Kilted Kaiju (Ubuntu 24.04)
- **Simulation**: Gazebo (primary), MuJoCo (alternative), Isaac Lab (alternative)
- **ML**: PyTorch, LeRobot (ACT policies), HuggingFace Hub
- **Build**: Pixi (conda), colcon, ament_cmake, ament_python
- **Middleware**: Zenoh (rmw_zenoh_cpp) for cross-container communication
- **Containers**: Docker + docker-compose for eval/model separation

## Key Concepts

### Policy Development (What We Implement)
- Subclass `aic_model.policy.Policy`
- Implement `insert_cable(task, get_observation, move_robot, send_feedback) -> bool`
- Use `move_robot()` for Cartesian/Joint commands
- Use `get_observation()` for sensor data (cameras, F/T, joint states)
- Return `True` for successful insertion

### ROS 2 Lifecycle
unconfigured -> configured -> active (accept tasks) -> deactivate -> cleanup -> shutdown

### Control Modes
- **Cartesian (Pose/Velocity)**: MotionUpdate msg to `/aic_controller/pose_commands`
- **Joint**: JointMotionUpdate msg to `/aic_controller/joint_commands`
- Switch via `/aic_controller/change_target_mode` service

### Observations (20 Hz)
- 3x wrist camera images (1152x1024 RGB)
- 6D force/torque wrench
- Joint states (7 joints)
- Controller state (TCP pose, velocity, error, reference)

### Scoring (100 points max per trial)
- Tier 1 (1pt): Model validity - loads and responds
- Tier 2 (24pts): Smoothness (6), Duration (12), Efficiency (6), minus Force penalty (-12), Collision penalty (-24)
- Tier 3 (75pts): Insertion success (75), Partial (38-50), Proximity (0-25)

## Build & Run Commands

```bash
# Install dependencies
pixi install

# Run simulation
pixi run ros2 launch aic_bringup aic_gz_bringup.launch.py

# Run model with example policy
pixi run ros2 run aic_model aic_model --ros-args -p use_sim_time:=true -p policy:=aic_example_policies.ros.WaveArm

# Run with ground truth (for training/debugging)
pixi run ros2 launch aic_bringup aic_gz_bringup.launch.py ground_truth:=true

# Docker build and test
docker compose -f docker/docker-compose.yaml build model
docker compose -f docker/docker-compose.yaml up

# Style checks
pixi run isort --check --diff .
pixi run black --check .
pixi run pyright
```

## Code Style

- **Python**: black formatter, isort (profile=black), pyright (basic mode)
- **C++**: clang-format v19 (Google style)
- **Commits**: Descriptive, lowercase, imperative mood
- **No hardcoded paths** - use ROS 2 parameters and relative paths
- **No ground truth exploitation** in submitted policies (CheatCode is for training only)

## CI/CD

- `build.yml`: ROS 2 colcon build + test
- `style.yml`: clang-format + black checks
- `pixi.yml`: isort + pyright validation

## Critical Rules

1. **NEVER modify** aic_engine, aic_controller, aic_adapter, aic_scoring, aic_gazebo, aic_description, aic_bringup during submission
2. **ONLY modify** aic_model/, aic_example_policies/, docker/aic_model/Dockerfile, and your custom policy packages
3. **No ground truth** in submitted policies - TF frames with `ground_truth:=true` are for training only
4. **Container must be OCI-compliant** and pushed to AWS ECR
5. **1 submission per day** per team
6. **Time limits** enforced per task (from Task.time_limit field)
7. **Lifecycle timeouts**: configure/activate/deactivate/cleanup/shutdown each have 60s max

## Toolkit Gotchas (from official bugs/PRs)

- **F/T sensor gravity bias**: Raw wrench includes ~20N from gripper weight. ALWAYS subtract `controller_state.fts_tare_offset`
- **Force penalty**: Triggers at 20N sustained >1s (-12pts). Leave safety margin
- **Observation can be None**: First call to `get_observation()` may return None. ALWAYS guard with `if obs is None` (Issue #339)
- **set_cartesian_mode() stalls ~30%**: Add retry logic when switching control modes (Issue #209)
- **set_pose_target()**: Now accepts custom stiffness/damping parameters (PR #430)
- **Phantom collision plane**: Multi-axis Cartesian moves near task board can stall. Decompose into single-axis moves (Issue #444)
- **NIC collision box wider than visual**: Actual collision extends beyond visible geometry (Issue #278)
- **Task board TFs**: Now published as static (PR #405) - lookups reliable
- **Off-limit contacts**: Test with `gz topic -e -t /aic/gazebo/contacts/off_limit` (PR #431)
- **Jerk scoring varies with RTF**: Machine-dependent Real-Time Factor affects jerk calculation (Issue #303)
- **Isaac Lab ↔ Gazebo**: Known coordinate frame mismatch (Bug #424) and 2x gravity compensation difference (Bug #434). Always validate in Gazebo
- **MuJoCo ↔ Gazebo**: Apply dynamics tuning from PR #419 (damping, armature, friction) for transfer
- **NIC card limits**: Check sample_config.yaml for actual ranges (docs may not match config)
- **Zenoh mandatory**: `RMW_IMPLEMENTATION=rmw_zenoh_cpp` always (PR #416). Docker: don't overwrite `ZENOH_CONFIG_OVERRIDE` (PR #432)
- **POSIX SHM crash**: Most common Docker issue. Set `transport/shared_memory/enabled=false` in Zenoh config (Issue #377)
- **MuJoCo is dev only**: Evaluation runs exclusively in Gazebo

## Intelligence Files

- `.claude/intel/competitor-approaches.md` - Approcci osservati da altri partecipanti (riferimento, non vincoli)
- `.claude/intel/official-bugs-and-fixes.md` - Bug e fix confermati dal repo ufficiale
- `.claude/intel/discourse-threads.md` - Indice thread forum Open Robotics
