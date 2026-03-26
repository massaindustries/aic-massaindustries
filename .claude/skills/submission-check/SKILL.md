---
description: Validates Docker submission before pushing to ECR
globs:
  - "docker/aic_model/Dockerfile"
  - "docker/docker-compose.yaml"
  - "pixi.toml"
---

# Submission Validation Skill

When Docker or build files are modified, validate:

1. **Dockerfile syntax**: Valid multi-stage build, correct base image
2. **Base image**: Should be `ros:kilted-ros-core` or compatible
3. **Policy reference**: CMD/ENTRYPOINT includes correct policy module path
4. **Dependencies**: All pip/pixi dependencies declared
5. **No secrets**: No AWS credentials, passwords, or API keys in Dockerfile
6. **Size optimization**: Multi-stage build, no unnecessary dev dependencies
7. **docker-compose.yaml**: Model service correctly configured
8. **Network**: No external network access configured for model service
9. **pixi.toml**: All workspace dependencies resolved

Report any issues that would prevent successful evaluation.
