ARG VARIANT=11-bullseye
FROM mcr.microsoft.com/devcontainers/java:1-${VARIANT}

SHELL ["/bin/bash", "-euxo", "pipefail", "-c"]

ARG APP_NAME
ENV APP_NAME=${APP_NAME:-myapp}

WORKDIR /workspaces/${APP_NAME}

ARG SERVERS_DIR=/home/vscode/.rsp/redhat-server-connector/runtimes/installations
ARG WILDFLY_VER=23.0.2.Final
ARG RSP_SERVERS_DIR=/home/vscode/.rsp/redhat-server-connector/servers
ARG PG_VERSION=17

ENV LANG=C.UTF-8 \
    TZ=Asia/Tokyo

# Install packages
RUN install -d /usr/share/postgresql-common/pgdg && \
    curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc && \
    sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && \
    apt-get update -qq && export DEBIAN_FRONTEND=noninteractive && \
    apt-get install --no-install-recommends -y \
    postgresql-client-${PG_VERSION} && \
    apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Install Wildfly
USER vscode
RUN mkdir -p ${SERVERS_DIR} && \
    curl -sL https://download.jboss.org/wildfly/${WILDFLY_VER}/wildfly-${WILDFLY_VER}.tar.gz | tar zx -C ${SERVERS_DIR}

# Add Management User(myadmin/P@ssw0rd)
RUN echo "myadmin=ccf3a3beccc44c0369f1612a9e849695" >> ${SERVERS_DIR}/wildfly-${WILDFLY_VER}/standalone/configuration/mgmt-users.properties && \
    echo -e "\nmyadmin=" >> ${SERVERS_DIR}/wildfly-${WILDFLY_VER}/standalone/configuration/mgmt-groups.properties

RUN mkdir -p ${RSP_SERVERS_DIR}
COPY --chown=vscode .devcontainer/wildfly-${WILDFLY_VER} ${RSP_SERVERS_DIR}

# Create Maven repository
# RUN sudo -u vscode sh -c 'mkdir -p /home/vscode/.m2'
USER vscode
RUN mkdir -p /home/vscode/.m2 && \
    cat <<EOF > /home/vscode/.m2/settings.xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>${PWD}/.m2/repository</localRepository>
</settings>
EOF

COPY . .
