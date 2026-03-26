# Testing Rules

## Running Tests
- Use `pixi run` prefix for all ROS 2 commands in pixi-managed environments
- Test scripts are in `aic_model/test/` (manual scripts, not pytest)
- Integration testing requires running simulation via `aic_gz_bringup.launch.py`

## Test Workflow
1. Start simulation: `pixi run ros2 launch aic_bringup aic_gz_bringup.launch.py`
2. In separate terminal: `pixi run ros2 run aic_model aic_model --ros-args -p use_sim_time:=true -p policy:=<your_policy>`
3. Monitor with: `pixi run ros2 topic echo /aic_controller/controller_state`

## Docker Testing
```bash
docker compose -f docker/docker-compose.yaml build model
docker compose -f docker/docker-compose.yaml up
```
- eval container starts Gazebo + engine + controller
- model container runs your policy
- Connected via Zenoh middleware on internal Docker network

## Validation Checks Before Submission
- [ ] Policy loads without errors (Tier 1)
- [ ] Model responds to `/insert_cable` action
- [ ] Robot moves and sends valid commands
- [ ] No ground truth TF usage in submitted code
- [ ] Docker container builds and runs
- [ ] Lifecycle transitions complete within 60s each
- [ ] Task completes within time_limit

## Style Checks
```bash
pixi run black --check .
pixi run isort --check --diff .
pixi run pyright
```

## CI Checks
- `build.yml`: colcon build with ROS 2 Kilted
- `style.yml`: clang-format (C++) + black (Python)
- `pixi.yml`: isort + pyright on Python packages
