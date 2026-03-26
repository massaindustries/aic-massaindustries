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

Flag issues as warnings or errors with file:line references.
