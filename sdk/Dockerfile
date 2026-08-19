FROM eclipse-temurin:21-jdk-noble

USER root

# Pinned versions. Bumping any of these is a one-line change.
ARG flutter_version
ARG ANDROID_CMDLINE_TOOLS_VERSION=13114758
ARG ANDROID_PLATFORM=android-36
ARG ANDROID_BUILD_TOOLS=36.0.0

ENV ANDROID_HOME=/opt/android-sdk
ENV FLUTTER_HOME=/opt/flutter
ENV FLUTTER_ROOT=${FLUTTER_HOME}
ENV FLUTTER_VERSION=${flutter_version}
ENV PATH=${PATH}:${FLUTTER_HOME}/bin:${FLUTTER_HOME}/bin/cache/dart-sdk/bin:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools

# Install only the tools the SDK install needs. The base already has JDK 21.
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        unzip \
        wget \
 && rm -rf /var/lib/apt/lists/*

# Android command-line tools from Google.
RUN mkdir -p ${ANDROID_HOME}/cmdline-tools \
 && cd /tmp \
 && wget -q "https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_CMDLINE_TOOLS_VERSION}_latest.zip" -O cmdline-tools.zip \
 && unzip -q cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools \
 && mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
 && rm cmdline-tools.zip

# Install Android SDK components.
# Pinning through build args means a future API level bump is one line.
RUN yes | sdkmanager --licenses > /dev/null \
 && sdkmanager \
        "platform-tools" \
        "platforms;${ANDROID_PLATFORM}" \
        "build-tools;${ANDROID_BUILD_TOOLS}"

# Clone Flutter at the requested tag.
RUN git clone --depth 1 --branch ${FLUTTER_VERSION} \
        https://github.com/flutter/flutter.git ${FLUTTER_HOME}

# Accept licenses and run doctor. Fail loudly if precache fails: no `|| true`
# masking a broken artifact cache.
RUN yes | flutter doctor --android-licenses > /dev/null \
 && flutter doctor \
 && flutter precache --android

# Make the SDK writable by any user and mark it as a safe git directory.
# This allows `docker run --user "$(id -u):$(id -g)"` so bind-mounted worktrees
# stay owned by the host user. See Project516/docker-images-flutter#6.
RUN chmod -R a+rwX "${FLUTTER_ROOT}" "${ANDROID_HOME}" \
 && git config --system --add safe.directory "${FLUTTER_ROOT}"
