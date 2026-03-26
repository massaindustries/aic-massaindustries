Help debug a cable insertion policy using proven community techniques.

Steps to diagnose insertion failures:

## 1. Verify Basic Connectivity
```bash
pixi run ros2 topic list | grep -E "observation|insert_cable|scoring"
pixi run ros2 lifecycle get /aic_model
```

## 2. Monitor Real-Time Scoring
```bash
# Subscribe to insertion events (hidden but powerful!)
pixi run ros2 topic echo /scoring/insertion_event
```

## 3. Check F/T Sensor (Gravity Bias)
The F/T sensor reads ~20N constantly due to gripper weight. Verify tare compensation:
```python
raw = observation.wrist_wrench.wrench
tare = observation.controller_state.fts_tare_offset.wrench
net_fz = raw.force.z - tare.force.z  # This should be ~0 when not in contact
```

## 4. Verify Pose Tracking
```bash
pixi run ros2 topic echo /aic_controller/controller_state --field tcp_error
```
If tcp_error is large, the robot hasn't reached its target. Don't proceed to insertion.

## 5. Log Episode Data (Best Practice)
Build structured episode logs with snapshots at each phase:
- initial_snapshot (baseline state)
- hover_snapshot (above target)
- align_snapshot (fine-tuned position)
- insert_snapshot (during descent)
- final_snapshot (outcome)

Track: requested pose, actual pose, tolerance reached, XYZ error, force magnitude, duration.

## 6. Common Failure Modes
- **Robot doesn't move**: Check lifecycle state is ACTIVE, check mode (Cartesian vs Joint)
- **Misses port**: Verify plug-tip tracking (not TCP), check alignment phase duration
- **Force penalty**: Raw wrench not tare-compensated, threshold too high
- **Timeout**: Approach phase too slow, too many retries
- **Wrong cable name**: aic_engine uses YAML key naming (cable_1 for Trial 3)

Analyze the user's specific issue and suggest targeted fixes.
