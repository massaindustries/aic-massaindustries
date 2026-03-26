---
description: Reviews policy implementations for scoring optimization
globs:
  - "aic_example_policies/aic_example_policies/ros/*.py"
---

# Policy Scoring Optimization Review

When a policy is modified, analyze it against the scoring criteria:

## Tier 2 Checks (24 points possible)
- **Smoothness (6pts)**: Look for sudden velocity changes, missing interpolation, or high-frequency oscillation
- **Duration (12pts)**: Check if the approach is direct or has unnecessary movements
- **Efficiency (6pts)**: Measure if the path takes detours vs going straight to target
- **Force safety (-12pts penalty)**: Check for missing force monitoring, contact detection, or force limits
- **Collision avoidance (-24pts penalty)**: Verify workspace bounds are respected

## Tier 3 Checks (75 points possible)
- **Insertion strategy**: Does the policy have a clear insertion phase?
- **Alignment**: Is there visual servoing or position correction before insertion?
- **Contact handling**: Does it use force feedback during insertion?

## Recommendations
Suggest concrete code improvements to maximize the score across all tiers.
