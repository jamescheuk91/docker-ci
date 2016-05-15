FROM debian:jessie
MAINTAINER jccf091

# Now create the Jenkins user in this container by adding:
RUN useradd -d "/var/jenkins_home" -u 1000 -m -s /bin/bash jenkins

# We need to recreate the Jenkins log directory in this image because it'll be the new foundation, so add the following lines:
RUN mkdir -p /var/log/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins

# Now we can add the Docker volume magic. Let’s make the log folder a mount:
VOLUME ["/var/log/jenkins", "/var/jenkins_home"]
# Let’s set the user of this container to “jenkins” for consistency, so add:
USER jenkins
# Lastly, while these images don’t actually “run” anything, I like to have them output a message when they start as a reminder to their purpose, so toss in the final line:
CMD ["echo", "Data container for Jenkins"]
