# Custom Jenkins controller image.
# Notes: Official Jenkins LTS on JDK 21, pinned to an exact version so builds are reproducible and upgrades are deliberate.
FROM jenkins/jenkins:2.568.3-jdk21

# Temporarily switch to root to install OS Packages
USER root

# Install the Docker CLI only (not the engine). 
# Jenkins sends commands to the separate DinD daemon over TLS, so no daemon runs in this container.
# Steps: 
# - install prerequisites, 
# - add Docker's official GPG key and apt repo (signed-by restricts that key to this repo only), 
# - install docker-ce-cli,
# - then clean the apt cache to keep the image small.
RUN apt-get update && apt-get install -y lsb-release ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
    https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
    | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && apt-get install -y docker-ce-cli && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Drop back to the normal jenkins user.
USER jenkins

# Pre-install plugins at build time:
#   blueocean       - modern pipeline visualisation UI
#   docker-workflow - Docker steps and "agent { docker { ... } }" in Jenkinsfiles
#   json-path-api   - JSON parsing library used by other plugins
# Versions are unpinned for latest compatibility;
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"
