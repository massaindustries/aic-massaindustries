# Submission Guidelines

## Container Requirements
- OCI-compliant Docker image
- Based on `ros:kilted-ros-core` (or compatible)
- Must include pixi environment with all dependencies
- Entry point: `pixi run ros2 run aic_model aic_model`
- Set policy via `-p policy:=<your_module.YourPolicy>`

## Build & Test Locally
```bash
# Build model container
docker compose -f docker/docker-compose.yaml build model

# Test full system
docker compose -f docker/docker-compose.yaml up

# View logs
docker compose -f docker/docker-compose.yaml logs -f model
```

## Push to AWS ECR
```bash
# Authenticate (credentials from competition email)
aws ecr get-login-password --region us-east-1 --profile aic | docker login --username AWS --password-stdin 973918476471.dkr.ecr.us-east-1.amazonaws.com

# Tag
docker tag localhost/my-solution:v1 973918476471.dkr.ecr.us-east-1.amazonaws.com/aic-team/<team_name>:v1

# Push
docker push 973918476471.dkr.ecr.us-east-1.amazonaws.com/aic-team/<team_name>:v1
```

## Submission Checklist
- [ ] No ground truth TF usage in policy code
- [ ] No direct state manipulation or backend interference
- [ ] Lifecycle timeouts respected (60s each)
- [ ] Task time_limit honored
- [ ] Docker container builds cleanly
- [ ] Container runs with internal network only (no internet)
- [ ] Policy responds to /insert_cable action
- [ ] Robot moves and sends valid commands (Tier 1)
- [ ] Model image tagged with incremented version (tags are immutable)

## Limits
- 1 submission per day per team
- Tag immutability: cannot overwrite existing tags
- Cloud eval specs: 64 vCPU, 256 GiB RAM, NVIDIA L4 (24GB VRAM)
