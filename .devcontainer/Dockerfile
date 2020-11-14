FROM ubuntu:focal

USER root

# android args
ARG ANDROID_ARCHITECTURE="x86_64"
ARG ANDROID_VERSION="29"
ARG ANDROID_BUILD_TOOLS_VERSION="29.0.3"
# comes from https://developer.android.com/studio/#command-tools
ARG ANDROID_SDK_TOOLS_VERSION="6858069"
# flutter args
ARG FLUTTER_REF=stable

# vscode args
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="false"
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV FLUTTER_SDK_ROOT=/sdks/flutter \
    ANDROID_SDK_ROOT=/opt/android-sdk

ENV PATH=${PATH}:${ANDROID_SDK_ROOT}/cmdline-tools/latest/bin:${ANDROID_SDK_ROOT}/platform-tools:${FLUTTER_SDK_ROOT}/bin:${FLUTTER_SDK_ROOT}/bin/cache/dart-sdk/bin

COPY .devcontainer/library-scripts/*.sh /tmp/library-scripts/
RUN set -e; \
    apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

# install dependencies
# sources:
#  - https://flutter.dev/docs/get-started/install/linux
#  - https://github.com/cirruslabs/docker-images-android/blob/master/sdk/tools/Dockerfile
#  - https://github.com/matsp/docker-flutter/blob/master/stable/Dockerfile
RUN set -ex; \
    apt-get update \
    && apt-get install --yes --no-install-recommends \
        bash \
        curl \
        git \
        libasound2 \
        libatk-bridge2.0-0 \
        libgdk-pixbuf2.0-0 \
        libgl1-mesa-glx \
        libglu1-mesa \
        libglvnd0 \
        libgtk-3-0 \
        libnspr4 \
        libnss3-dev \
        libpulse0 \
        libxcomposite1 \
        libxss1 \
        libxtst6 \
        openjdk-8-jdk \
        sed \
        ssh \
        unzip \
        wget \
        x11-xserver-utils \
        xauth \
        xz-utils \
        zip \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# android sdk
RUN set -ex; \
    wget -q https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_TOOLS_VERSION}_latest.zip -O android-sdk-tools.zip \
    && mkdir -p ${ANDROID_SDK_ROOT}/cmdline-tools \
    && unzip -q android-sdk-tools.zip \
    && mv cmdline-tools/ ${ANDROID_SDK_ROOT}/cmdline-tools/latest \
    && chown -R root:root ${ANDROID_SDK_ROOT} \
    && rm android-sdk-tools.zip \
    && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers \
    && yes | sdkmanager --licenses \
    && mkdir -p /root/.android \
    && touch /root/.android/repositories.cfg \
    && sdkmanager "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" \
    && sdkmanager "platforms;android-${ANDROID_VERSION}" \
    && sdkmanager "sources;android-${ANDROID_VERSION}" \
    && sdkmanager "platform-tools" \
    && sdkmanager "emulator" \
    && sdkmanager "system-images;android-${ANDROID_VERSION};google_apis_playstore;${ANDROID_ARCHITECTURE}"

# flutter
RUN set -ex; \
    git clone --branch ${FLUTTER_VERSION} https://github.com/flutter/flutter.git ${FLUTTER_SDK_ROOT} \
    && flutter config --no-analytics \
    && flutter precache \
    && yes | flutter doctor --android-licenses \
    && flutter doctor \
    && flutter emulators --create \
    && flutter update-packages

USER ${USERNAME}

