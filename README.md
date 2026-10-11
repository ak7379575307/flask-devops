# Flask DevOps Project: Docker + CI/CD with GitHub Actions

A small Flask application, containerized with Docker and shipped through an automated CI/CD pipeline. Every push to `main` runs the tests and, only if they pass, builds a Docker image and pushes it to Docker Hub.

## Architecture

```
Developer -> git push -> GitHub -> GitHub Actions
                                      |
                                      |-- Job 1: test (pytest)
                                      |-- Job 2: build image and push to Docker Hub
                                                 (runs only if tests pass)
```

## Tech stack
- Python 3.12, Flask, Gunicorn
- Docker (multi-stage build, non-root user)
- GitHub Actions (CI/CD)
- Docker Hub (image registry)
- AWS EC2 (Amazon Linux 2023) for development and testing

## API endpoints
| Endpoint | Description |
|---|---|
| `/` | Hello message and container hostname |
| `/health` | Health check, returns `{"status":"ok"}` |
| `/version` | App version |

## Run locally

```bash
docker pull ashv1234/flask-devops:latest
docker run -d -p 5000:5000 --name flaskapp ashv1234/flask-devops:latest
curl localhost:5000/health
```

## CI/CD pipeline
- **Trigger:** push or pull request to `main`
- **test job:** installs dependencies and runs `pytest`
- **build-and-push job:** needs `test` to pass, logs in to Docker Hub using GitHub Secrets, builds the image and pushes two tags:
  - `latest`
  - the commit SHA (for traceability and rollback)

## Key points
- **Multi-stage Dockerfile** keeps the final image small, with no build tools in it
- **Non-root user** inside the container for better security
- **Quality gate:** a failing test stops the pipeline, so a broken build is never pushed
- **Secrets** (Docker Hub credentials) are stored in GitHub Secrets, never in code

## Challenges and fixes
- `COPY requirements.txt` failed during build: the file was missing from the build context, fixed by creating it in the project root
- Push rejected for the workflow file: the access token needed the `workflow` scope
- Pipeline failed at Docker Hub login: the secrets were missing, fixed by adding `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` as repository secrets

## Next steps
- Deploy the image to AWS EKS (Deployment, Service, probes)
- Add continuous deployment to the pipeline
- Add monitoring with Prometheus and Grafana
