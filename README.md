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

- **Non-root Jenkins:** the controller runs as the `jenkins` user; root is used only during the image build to install the Docker CLI.
- **Network exposure:** the UI is bound to `127.0.0.1` only, agent port 50000 is not published, and DinD's TLS port 2376 is reachable only from the internal `jenkins` network.
- **TLS to DinD:** Jenkins authenticates to the daemon with client certificates; the certs volume is mounted read-only in Jenkins.
- **Access control:** sign-up is disabled, matrix-based authorization is enabled, and anonymous users have no access.
- **Scoped builds:** pipelines run as a non-admin user via the Authorize Project plugin, with only the permissions they need (Agent/Build, Credentials/UseItem).
- **`JAVA_OPTS`:** enables the Credentials/UseItem permission, so the build user can use stored credentials without needing Job/Configure.
- **Secrets:** the Docker Hub access token lives in the Jenkins credential store, never in this repo.
- **Privileged DinD trade-off:** dockerd requires privileged mode, which is near-root on the host. Exposure is limited (no published ports, TLS-only access, trusted builds only) rather than the privilege being contained. Rootless alternatives (rootless DinD, Sysbox, Kaniko) would remove it.
- **Pinned versions:** the Jenkins base image and DinD are pinned to specific versions, and unused/deprecated plugins were removed.
