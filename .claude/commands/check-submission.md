Perform a comprehensive pre-submission validation check.

Run through these checks:

1. **Docker Build**: Verify `docker/aic_model/Dockerfile` builds successfully
2. **Policy Loading**: Confirm the target policy module is importable
3. **Lifecycle Compliance**: Check lifecycle transitions complete within 60s
4. **Interface Compliance**: Verify all required topics/services/actions are used correctly
5. **No Cheating**: Scan for ground truth TF usage, hardcoded sensor data, or backend manipulation
6. **Dependencies**: Ensure all Python/system dependencies are in pixi.toml or Dockerfile
7. **Style Checks**: Run `black --check`, `isort --check`, `pyright`
8. **Container Size**: Check for unnecessary files that bloat the image
9. **Network Isolation**: Verify no external network calls in policy code

Report pass/fail for each check with details on any failures.
