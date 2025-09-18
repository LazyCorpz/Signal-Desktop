ARG BASE_IMAGE=ubuntu:focal-20240530@sha256:fa17826afb526a9fc7250e0fbcbfd18d03fe7a54849472f86879d8bf562c629e

FROM ${BASE_IMAGE} AS base

COPY [ "./docker/apt.conf", "/etc/apt/" ]

FROM base AS build

# Due to some issues with NVM reading .nvmrc, we define the version
# as an environment variable and use that instead.
ARG NODE_VERSION
ENV SIGNAL_ENV=production

RUN \
    apt update -qq \
    && apt install -yqq \
        ca-certificates \
        curl \
        g++ \
        g++-10 \
        gcc \
        gcc-10 \
        git \
        make \
        python3 \
        tar \
        xz-utils \
    && apt-get clean -qq \
    # On Ubuntu 20 GCC 9 is the default but we need 10 to build
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100 \
        --slave /usr/bin/g++ g++ /usr/bin/g++-10

# Installing nvm
ARG NVM_VERSION=0.40.0
ENV NVM_DIR=/usr/local/nvm

ADD --chmod="+x" [ \
    "https://raw.githubusercontent.com/nvm-sh/nvm/v${NVM_VERSION}/install.sh", \
    "/tmp/" \
]

RUN \
    mkdir -p "${NVM_DIR}" \
    && '/tmp/install.sh' \
    && rm '/tmp/install.sh' \
    && . "${NVM_DIR}/nvm.sh" \
    && nvm install "${NODE_VERSION}" \
    && nvm alias "${NODE_VERSION}" \
    && nvm use "${NODE_VERSION}"

ENV NODE_PATH="${NVM_DIR}/v${NODE_VERSION}/lib/node_modules"
ENV PATH="${NVM_DIR}/versions/node/v${NODE_VERSION}/bin:${PATH}"
ENV CI=true

RUN \
    npm install -g pnpm@10.3.0 \
    && git config --global --add safe.directory /project

COPY --chmod="+x" [ "docker-entrypoint.sh", "/usr/local/bin/" ]

# For electron builder cache.
# It's tricky to synchronize user/group permissions between the host and container without creating
# host-specific users at container build time, which would break reproducibility.
# Instead of doing that we will fix permissions on required directories.
RUN \
    mkdir "/.cache" \
    && chmod -R 777 "/.cache"

ENTRYPOINT [ "docker-entrypoint.sh" ]

# Allows package builders like FPM (used for creating the .deb package
# on linux) to make their build timestamps determistic. Otherwise, a fresh
# UNIX timestamp will be generated at the time of the build, and is non-deterministic.
#
# Read https://reproducible-builds.org/specs/source-date-epoch/ for more info
ENV SOURCE_DATE_EPOCH=1

# Specify build type using CMD, which affects the app version and name of the resulting package.
# For more information see build.sh and docker-entrypoint.sh
CMD [ "dev" ]
