---
description: Automatically reviews policy code changes for AIC competition compliance
globs:
  - "aic_example_policies/**/*.py"
  - "aic_model/**/*.py"
---

# AIC Policy Code Review

When policy files are modified, automatically check:

1. **Import correctness**: All ROS 2 message types imported from correct packages
2. **Policy base class**: Must extend `aic_model.policy.Policy`
3. **insert_cable signature**: `(self, task, get_observation, move_robot, send_feedback) -> bool`
4. **No banned patterns**:
   - No `lookup_transform` with ground truth frame names (e.g., frames containing "port", "plug", "cable" from TF)
   - No `subprocess` or `os.system` calls
   - No network/socket operations
   - No file writes outside /tmp
5. **Timeout respect**: Policy should check `task.time_limit`
6. **Return value**: Must return bool (True = success, False = failure)
7. **Command safety**: Stiffness values should be reasonable (not exceeding 5000 N/m for translation, 500 Nm/rad for rotation)

## Critical Community-Discovered Checks

8. **F/T Tare Compensation**: If using force/torque data, verify tare offset is subtracted from raw wrench.
   The sensor has ~20N gravity bias. Look for `fts_tare_offset` usage when wrench values are read.
   ```python
   # CORRECT - compensated
   net_force = raw_wrench.force.z - tare_offset.wrench.force.z
   # WRONG - raw (includes ~20N gravity)
   net_force = raw_wrench.force.z
   ```

9. **Pose Verification**: After commanding a pose via `move_robot()`, check that the policy verifies
   the robot actually reached it (via `controller_state.tcp_error` or position readback).
   "Command and pray" is a known failure pattern.

10. **Delta-Time Scaling**: Any velocity-based movement must use proper time deltas, NOT assume
    fixed frame rate. Frame-rate dependent speeds cause 60x errors.

11. **Force Threshold**: If force thresholds are used, verify they're set below 19.5N (penalty at 20N
    sustained >1s). Common mistake: using 20N or higher without margin.

12. **Plug-Tip vs TCP**: If distance calculations are used, verify they track the plug tip position,
    not the TCP (tool center point at gripper flange).

13. **SC vs SFP Awareness**: If handling both connector types, check for different tuning parameters.
    SC connectors are round with spring-loaded latches; SFP are rectangular.

Flag issues as warnings or errors with file:line references.
