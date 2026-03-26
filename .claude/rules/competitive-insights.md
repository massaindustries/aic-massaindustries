# Competitive Intelligence & Community Insights

Compiled from GitHub issues, forum threads, participant repos, and community discussions.

## Critical Technical Discoveries

### 1. Force/Torque Sensor Gravity Bias (MUST FIX)
The F/T sensor reads ~20N constantly due to gripper weight (~2kg * 9.81 m/s^2). Raw wrench data is USELESS without tare compensation:
```python
# ALWAYS subtract tare offset from raw wrench
raw_wrench = observation.wrist_wrench.wrench
tare = observation.controller_state.fts_tare_offset.wrench
net_force_x = raw_wrench.force.x - tare.force.x
net_force_y = raw_wrench.force.y - tare.force.y
net_force_z = raw_wrench.force.z - tare.force.z
# Same for torques
```
Without this, force thresholds for contact detection will be completely wrong.

### 2. Plug-Tip Distance > TCP Distance
Monitoring the actual plug connector-to-port gap is MORE valuable than tracking TCP position. The TCP (tool center point) is at the gripper flange, not the plug tip. Plug-tip position is what determines insertion success.

### 3. Insertion Events Topic (Hidden Feature)
Subscribe to `/scoring/insertion_event` (String msg type) to get REAL-TIME notification when a successful insertion occurs. This publishes the port identifier string. **Partial insertion ≠ insertion event** - the scoring system can award partial credit without triggering this signal.

### 4. "Command ≠ Execution" Problem
Sending a pose command does NOT guarantee the robot reached it. Always verify:
- Read back `controller_state.tcp_pose` after commanding
- Check `controller_state.tcp_error` for tracking error
- Measure actual XYZ error between commanded and achieved pose
- Track movement duration (did it settle?)

### 5. TF Frame Naming Discrepancy
- Automatic spawning (aic_engine) names cables via YAML keys: `cable_1` for Trial 3
- Manual spawning hardcodes `cable_0`
- **Always use `cable_0` when manually spawning custom scenes**
- Set `spin_thread=True` in TF listener to receive transforms correctly

### 6. Isaac Lab vs Gazebo Coordinate Mismatch (Bug #424)
There is a known coordinate frame mismatch between Isaac Lab training and Gazebo evaluation:
- Port entrance position computed differently
- Insertion direction differs
- Plug reference frame misaligned
- **Policies trained in Isaac Lab may fail in Gazebo due to systematic misalignment**
- If training in Isaac Lab, validate thoroughly in Gazebo before submission

### 7. NIC Card Translation Limits - Docs Are Wrong
- Docs say `[0, 0.062]` meters but actual config allows `±0.084` meters
- Limits represent min/max deltas from rail center (bidirectional, not unidirectional)
- **Always reference sample_config.yaml rather than docs for accurate constraints**

### 8. Descent Speed Frame-Rate Bug
Descent velocity can be frame-rate dependent, making it ~60x faster than intended (0.6 m/s instead of 0.01 m/s). **Always use proper delta-time scaling for velocity-based movements.**

## Winning Strategies from Participants

### State Machine Approach (Rocky - Score 216/300, 3 successful trials)
Best documented approach uses clean state machine:
1. **INIT**: TF frame validation
2. **APPROACH**: PI velocity control to 20cm hover above target
3. **ALIGN**: Fine-tune XY position (<0.5mm) and angular alignment for 2+ seconds
4. **INSERT**: Constant 12mm/s descent with compliance gains for lateral correction
5. **DONE**: Terminal state

**Key parameters**:
```
kp_linear: 1.2
ki_linear: 0.2
kp_angular: 2.0 (reduced to 25% during insertion)
max_linear_vel: 0.08 m/s
force_safety_threshold: 19.5N (just under 20N penalty threshold)
```

### Data Collection Strategy
- Use CheatCode (ground truth) policy to generate CLEAN training demonstrations
- Record via LeRobot pipeline for synchronized camera + action data
- Target 50+ first-try demonstrations per trial type
- First-try demos > failure-recovery demos (cleaner training signal)
- Record: gripper tip velocity, multi-camera feeds, bias-compensated F/T data

### Hover-Then-Descend Pattern
Multiple participants converge on:
1. Hover verification above target
2. Camera confirmation as validation step
3. Gradual descent (not aggressive insertion)
4. Force profile monitoring during insertion

### SC vs SFP Tuning
- SC connectors have spring-loaded latches and round geometry
- SFP ports are rectangular
- Different tuning needed for each connector type
- Consider dual configurations in your policy

## Common Pitfalls

### Docker/Submission
- Container must work with internal network ONLY (no internet)
- Model image tags are IMMUTABLE - always increment version
- GPU may not be used in evaluator - verify your policy works without GPU rendering
- MuJoCo is for development only - evaluation runs EXCLUSIVELY in Gazebo

### Policy Development
- "Spam and pray" recovery (lift and retry) has very low success (~12%)
- Magic numbers accumulate and breed fragility - use configurable parameters
- Over-reliance on hard-coded thresholds creates brittle policies
- Build reliable movement infrastructure BEFORE attempting insertion
- Verify robot actually reached commanded pose before proceeding

### Training
- Force monitoring below 10N during insertion is the safe target (penalty starts at 20N sustained for >1s)
- 19.5N is the practical safety threshold (leaves margin before 20N penalty)
- Clean demonstrations beat recovery demonstrations for training
- Simulation physics affect training data quality (gravity bias, tool mass, sensor compensation)

## Scoring Optimization Tips

### Priority Order (from community data)
1. **Get insertion success** (75 pts) - this dominates everything
2. **Stay under 20N force** - penalty is -12 pts, very costly
3. **Avoid collisions** - penalty is -24 pts, devastating
4. **Be fast** (12 pts) - aim for <15s task duration
5. **Smooth trajectory** (6 pts) - use interpolation, avoid jerky motion
6. **Short path** (6 pts) - direct approach to target

### Observation Processing Tips
- Use the aggregated `/observations` topic (has everything in one message)
- Monitor `controller_state.tcp_error` for real-time tracking accuracy
- Use force feedback for contact detection (after tare compensation)
- Camera images are raw RGB 1152x1024 - resize for ML models

## Competitor Landscape (as of March 2026)
- ~157 forks of official repo, most are unmodified
- Rocky (Ohio State freshman) is the most documented open-source participant - score 216
- "Team Autoencoder" forked but no visible custom code
- "No Quarter Robotics" sharing debugging insights on Discourse
- Most teams keeping strategies private
- Community converging on: CheatCode for data collection → ACT/LeRobot training → clean state machine policy
