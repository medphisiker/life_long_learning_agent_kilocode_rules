# Deployment Rules

- **DEV**: `docker-compose-dev.yml`. Build from local `Dockerfile`. For dev/testing.
- **PROD**: `docker-compose-prod.yml`. Use `ghcr.io` images. For stable deployment.
- **Network**: PROD services must use external networks expected by `agent_service`.
- **Ports**: PROD ports must strictly match `agent_service` environment variables.
- **Test**: After push to GHCR, always test PROD by deleting local images and running `docker-compose-prod.yml`.