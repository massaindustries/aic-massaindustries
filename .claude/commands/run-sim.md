Provide instructions and commands to run the AIC simulation environment.

Based on the user's request, provide the appropriate commands:

## Full Simulation Launch
```bash
# Terminal 1: Start simulation
pixi run ros2 launch aic_bringup aic_gz_bringup.launch.py use_sim_time:=true

# Terminal 2: Run policy
pixi run ros2 run aic_model aic_model --ros-args -p use_sim_time:=true -p policy:=<policy_module>
```

## With Ground Truth (Training Mode)
```bash
pixi run ros2 launch aic_bringup aic_gz_bringup.launch.py ground_truth:=true use_sim_time:=true
```

## Docker Mode
```bash
docker compose -f docker/docker-compose.yaml build model
docker compose -f docker/docker-compose.yaml up
```

## Debugging Tools
```bash
# Monitor topics
pixi run ros2 topic list
pixi run ros2 topic echo /aic_controller/controller_state

# Check node lifecycle
pixi run ros2 lifecycle get /aic_model

# View TF tree
pixi run ros2 run tf2_tools view_frames
```

Help the user start the appropriate configuration for their use case (training, testing, debugging, Docker validation).
