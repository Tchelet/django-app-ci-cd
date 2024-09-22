FROM jenkins/inbound-agent:latest

USER root

# Install Docker from Debian repositories
RUN apt-get update && \
    apt-get install -y docker.io

# Add Jenkins user to the docker group
RUN usermod -aG docker jenkins

USER jenkins

# Set the entrypoint to start the Jenkins agent
ENTRYPOINT ["jenkins-agent"]
