# ISEC6000 Jenkins Infrastructure

Docker Compose setup for a Jenkins CI environment using Docker-in-Docker (DinD), built for ISEC6000 Secure DevOps Assessment 2.

## Structure

- `docker-compose.yml`: Jenkins and DinD services, network, and volumes
- `jenkins/Dockerfile`: custom Jenkins image with the Docker CLI and plugins

## Usage

    docker compose up -d --build
    docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

Open http://localhost:8080 and complete the setup wizard.

## Verify

    docker compose exec jenkins docker info                            # Jenkins can reach DinD over TLS
    docker compose exec docker docker info | grep -i "storage driver"  # should be overlay2

## Security Notes

- Jenkins runs as the non-root `jenkins` user
- Jenkins talks to DinD over TLS on 2376, which is not published to the host
- The UI is bound to localhost only, and agent port 50000 is not exposed
