FROM eclipse-temurin:21-jdk-noble@sha256:07d9692c2269232ebd4d45ed6b2bd520e04eb38a31ab5d3253c38ef0e90b40c5

USER root

# Pinned versions. Bumping any of these is a one-line change.
ARG flutter_version
ARG flutter_commit
ARG ANDROID_CMDLINE_TOOLS_VERSION=13114758
ARG ANDROID_PLATFORM=android-36
ARG ANDROID_BUILD_TOOLS=36.0.0

ENV ANDROID_HOME=/opt/android-sdk
ENV FLUTTER_HOME=/opt/flutter
ENV FLUTTER_ROOT=${FLUTTER_HOME}
ENV FLUTTER_VERSION=${flutter_version}
ENV PUB_CACHE=/opt/pub-cache
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
 && echo "7ec965280a073311c339e571cd5de778b9975026cfcbe79f2b1cdcb1e15317ee  cmdline-tools.zip" | sha256sum -c - \
 && unzip -q cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools \
 && mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
 && rm cmdline-tools.zip

# Install Android SDK components.
# Pinning through build args means a future API level bump is one line.
# platform-tools is downloaded directly from Google with a checksum because
# sdkmanager installs the latest revision with no way to pin.
ARG ANDROID_PLATFORM_TOOLS_VERSION=37.0.1
RUN mkdir -p ${ANDROID_HOME}/platform-tools \
 && cd /tmp \
 && wget -q "https://dl.google.com/android/repository/platform-tools_r${ANDROID_PLATFORM_TOOLS_VERSION}-linux.zip" -O platform-tools.zip \
 && echo "d230f13842f60f782a8645f9c813f8f845bf36089ea7289f28c48f17979313f1  platform-tools.zip" | sha256sum -c - \
 && unzip -q platform-tools.zip -d ${ANDROID_HOME} \
 && rm platform-tools.zip

RUN yes | sdkmanager --licenses > /dev/null \
 && sdkmanager \
        "platforms;${ANDROID_PLATFORM}" \
        "build-tools;${ANDROID_BUILD_TOOLS}"

# Validate the commit hash before fetching to fail closed on missing or
# malformed values. A shallow fetch of an explicit SHA is safe regardless
# of the default refspec.
RUN test -n "${flutter_commit}" \
 && echo "${flutter_commit}" | grep -qxE '[0-9a-f]{40}' \
 && git init ${FLUTTER_HOME} \
 && cd ${FLUTTER_HOME} \
 && git remote add origin https://github.com/flutter/flutter.git \
 && git fetch --depth 1 origin "${flutter_commit}" \
 && git checkout FETCH_HEAD

# Accept licenses and run doctor. Fail loudly if precache fails: no `|| true`
# masking a broken artifact cache.
RUN yes | flutter doctor --android-licenses > /dev/null \
 && flutter doctor \
 && flutter precache --android

# Make only the download caches writable for non-root users.
# Toolchain binaries (flutter_tools.snapshot, dart-sdk) and .git metadata
# stay read-only to prevent untrusted modification of build tools.
# See Project516/docker-images-flutter#6.
RUN mkdir -p "${PUB_CACHE}" \
 && chmod -R a+rwX "${FLUTTER_HOME}/bin/cache/artifacts" "${PUB_CACHE}" \
 && git config --system --add safe.directory "${FLUTTER_ROOT}"
