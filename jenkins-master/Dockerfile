FROM debian:jessie
MAINTAINER jccf091

# Install all dependencies
RUN echo 'deb http://httpredir.debian.org/debian jessie-backports main' > /etc/apt/sources.list.d/jessie-backports.list
RUN apt-get update
RUN apt-get install -y curl wget locales vim && \
  echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
  locale-gen

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN echo "deb http://packages.erlang-solutions.com/debian trusty contrib" >> /etc/apt/sources.list && \
    apt-key adv --fetch-keys http://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc && \
    apt-get -qq update && apt-get install -y \
    esl-erlang \
    git \
    unzip \
    build-essential \
    wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Download and Install Specific Version of Elixir
ENV ELIXIR_VERSION v1.2.5

WORKDIR /elixir
RUN wget -q https://github.com/elixir-lang/elixir/releases/download/$ELIXIR_VERSION/Precompiled.zip && \
    unzip Precompiled.zip && \
    rm -f Precompiled.zip && \
    ln -s /elixir/bin/elixirc /usr/local/bin/elixirc && \
    ln -s /elixir/bin/elixir /usr/local/bin/elixir && \
    ln -s /elixir/bin/mix /usr/local/bin/mix && \
    ln -s /elixir/bin/iex /usr/local/bin/iex

# Install local Elixir hex and rebar
RUN /usr/local/bin/mix local.hex --force && \
    /usr/local/bin/mix local.rebar --force

WORKDIR /


ENV PHOENIX_VERSION 1.1.4

# install the Phoenix Mix archive
RUN mix archive.install --force https://github.com/phoenixframework/archives/raw/master/phoenix_new-$PHOENIX_VERSION.ez


ENV JAVA_VERSION 8u66
ENV JAVA_DEBIAN_VERSION 8u72-b15-1~bpo8+1

# see https://bugs.debian.org/775775
# and https://github.com/docker-library/java/issues/19#issuecomment-70546872
ENV CA_CERTIFICATES_JAVA_VERSION 20140324

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    wget \
    curl \
    ca-certificates \
    zip \
    openssh-client \
    unzip \
    openjdk-8-jdk="$JAVA_DEBIAN_VERSION" \
    ca-certificates-java="$CA_CERTIFICATES_JAVA_VERSION" \
    && rm -rf /var/lib/apt/lists/*

RUN /var/lib/dpkg/info/ca-certificates-java.postinst configure

# Install Tini
ENV TINI_SHA 066ad710107dc7ee05d3aa6e4974f01dc98f3888

# Use tini as subreaper in Docker container to adopt zombie processes
RUN curl -fL https://github.com/krallin/tini/releases/download/v0.5.0/tini-static -o /bin/tini && chmod +x /bin/tini \
  && echo "$TINI_SHA /bin/tini" | sha1sum -c -

# SET Jenkins Environment Variables
ENV JENKINS_HOME /var/jenkins_home
ENV JENKINS_SLAVE_AGENT_PORT 50000
ENV JENKINS_VERSION 1.651.2
ENV JENKINS_SHA f61b8b604acba5076a93dcde28c0be2561d17bde
ENV JENKINS_UC https://updates.jenkins-ci.org
ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log
ENV JAVA_OPTS="-Xmx8192m"
ENV JENKINS_OPTS="--handlerCountStartup=100 --handlerCountMax=300 --logfile=/var/log/jenkins/jenkins.log  --webroot=/var/cache/jenkins/war"

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN useradd -d "$JENKINS_HOME" -u 1000 -m -s /bin/bash jenkins

# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME /var/jenkins_home

# `/usr/share/jenkins/ref/` contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

# Install Jenkins
RUN curl -fL http://mirrors.jenkins-ci.org/war-stable/$JENKINS_VERSION/jenkins.war -o /usr/share/jenkins/jenkins.war \
  && echo "$JENKINS_SHA /usr/share/jenkins/jenkins.war" | sha1sum -c -

# Prep Jenkins Directories
RUN chown -R jenkins "$JENKINS_HOME" /usr/share/jenkins/ref
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins

# Expose Ports for web and slave agents
EXPOSE 8080
EXPOSE 50000

# Copy in local config filesfiles
COPY init.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy
COPY jenkins.sh /usr/local/bin/jenkins.sh
COPY plugins.sh /usr/local/bin/plugins.sh
RUN chmod +x /usr/local/bin/plugins.sh
RUN chmod +x /usr/local/bin/jenkins.sh

# Switch to the jenkins user
USER jenkins

# Tini as the entry point to manage zombie processes
ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/jenkins.sh"]
