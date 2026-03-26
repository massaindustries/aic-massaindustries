Review the policy implementation for AIC competition compliance and quality.

Check the following for the policy file provided as argument (or scan aic_example_policies/ for custom policies):

1. **API Compliance**: Verify it subclasses `aic_model.policy.Policy` and implements `insert_cable()`
2. **No Ground Truth**: Ensure no TF lookup for ground truth frames (disallowed in submission)
3. **Timeout Handling**: Check that the policy respects `task.time_limit`
4. **Cancellation**: Verify the policy can be interrupted gracefully
5. **Command Rate**: Ensure commands are sent at 10-30Hz (not too fast, not too slow)
6. **Observation Usage**: Check that sensor data is properly consumed
7. **Safety**: Look for potential force spikes or unsafe motions
8. **Style**: Verify black/isort/pyright compliance
9. **Scoring Optimization**: Suggest improvements for trajectory smoothness, efficiency, and speed

Provide specific line-by-line feedback and an overall score estimate based on the scoring system.
