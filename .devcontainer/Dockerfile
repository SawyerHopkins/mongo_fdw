# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.245.2/containers/cpp/.devcontainer/base.Dockerfile

# [Choice] Debian / Ubuntu version (use Debian 11, Ubuntu 18.04/22.04 on local arm64/Apple Silicon): debian-11, debian-10, ubuntu-22.04, ubuntu-20.04, ubuntu-18.04
ARG VARIANT="bullseye"
FROM mcr.microsoft.com/vscode/devcontainers/cpp:0-${VARIANT}
ARG PG_VERSION="13.8"

# Add PG to path
ENV PATH=/usr/local/pgsql/bin:$PATH

# [Optional] Install CMake version different from what base image has already installed. 
# CMake reinstall choices: none, 3.21.5, 3.22.2, or versions from https://cmake.org/download/
ARG REINSTALL_CMAKE_VERSION_FROM_SOURCE="none"

# Optionally install the cmake for vcpkg
COPY ./reinstall-cmake.sh /tmp/
RUN if [ "${REINSTALL_CMAKE_VERSION_FROM_SOURCE}" != "none" ]; then \
        chmod +x /tmp/reinstall-cmake.sh && /tmp/reinstall-cmake.sh ${REINSTALL_CMAKE_VERSION_FROM_SOURCE}; \
    fi \
    && rm -f /tmp/reinstall-cmake.sh

# [Optional] Uncomment this section to install additional vcpkg ports.
# RUN su vscode -c "${VCPKG_ROOT}/vcpkg install <your-port-name-here>"

# Setup PostgreSQL and Mongo apt repository
RUN echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list \
    && wget --quiet -O - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

# [Optional] Uncomment this section to install additional packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
      libssl-dev \
      mongodb-org \
      libreadline-dev \
      zlib1g-dev

# Install postgres
RUN curl https://ftp.postgresql.org/pub/source/v${PG_VERSION}/postgresql-${PG_VERSION}.tar.bz2 | bzip2 -d | tar -x \
    && cd postgresql-${PG_VERSION} \
    && ./configure --enable-cassert --enable-debug CFLAGS="-ggdb" \
    && make \
    && sudo make install \
    && sudo chown vscode /usr/local/pgsql \
    && su - vscode -c '/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data' 

# Additional setup
RUN sudo mkdir /opt/drivers \
    # Give container user ownership of an opt directory for 3rd party lib install
    && chown vscode /opt/drivers \
    # Register driver library
    && sudo echo "/opt/drivers/mongo-c/lib" >> /etc/ld.so.conf.d/libc.conf \
    # Add pg_config to bin for root make install
    && sudo ln -s /usr/local/pgsql/bin/pg_config /usr/bin/pg_config
