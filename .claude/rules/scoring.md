# Scoring System Reference

## Tier 1 - Model Validity (1 point)
- Model loads and transitions through lifecycle correctly
- Responds to InsertCable action server
- Sends at least one valid command to robot controller
- Minimum topic publishing requirements met

## Tier 2 - Performance Metrics (up to 24 points, penalties possible)

### Trajectory Smoothness (0-6 points)
- Score = 6 * max(0, 1 - jerk/50)
- Jerk measured in m/s^3 (linear end-effector jerk)
- Only scored if task progresses (plug moves toward port)
- 0 jerk = 6 points, >=50 jerk = 0 points

### Task Duration (0-12 points)
- Score = 12 * max(0, 1 - (duration - 5) / 55)
- Duration <= 5s = 12 points
- Duration >= 60s = 0 points
- Measured from action goal acceptance to completion

### Trajectory Efficiency (0-6 points)
- Score = 6 * max(0, 1 - (path_length - initial_distance) / 1.0)
- Path length = total end-effector travel distance
- Shortest possible = initial distance to port
- Path >= 1m + initial_distance = 0 points

### Insertion Force Penalty (0 to -12 points)
- Penalizes sustained force > 20N for > 1 second
- Measured at force/torque sensor

### Off-Limit Contact Penalty (0 to -24 points)
- Collision with task board enclosure or non-target areas
- Severe penalty for unsafe behavior

## Tier 3 - Task Success (up to 75 points)

### Full Insertion: 75 points
- Plug fully seated in correct port
- Wrong port: -12 points

### Partial Insertion: 38-50 points
- Plug inside port bounding box
- Score proportional to insertion depth

### Proximity: 0-25 points
- Distance from plug to port entrance
- Closer = more points
- Max acceptable distance determines 0-point threshold

## Total Score Per Trial: Max 100 points
- Tier 1: 1
- Tier 2: up to 24 (minus penalties)
- Tier 3: up to 75

## Qualification Trials
- Trial 1 (SFP): Convergence test with NIC card position randomization
- Trial 2 (SFP): Same as Trial 1 with different randomization seed
- Trial 3 (SC): Generalization test with SC plug/port type

## Optimization Priorities
1. Get full insertion (75 pts) - biggest reward
2. Minimize task duration (12 pts) - be fast
3. Smooth trajectory (6 pts) - avoid jerky motion
4. Efficient path (6 pts) - direct route
5. Avoid force penalties (-12 pts) - gentle contact
6. Avoid collisions (-24 pts) - never hit the enclosure
