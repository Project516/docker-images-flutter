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
RUN yes | sdkmanager --licenses > /dev/null \
 && sdkmanager \
        "platform-tools" \
        "platforms;${ANDROID_PLATFORM}" \
        "build-tools;${ANDROID_BUILD_TOOLS}"

# Clone Flutter at the verified commit from the release manifest.
RUN git init ${FLUTTER_HOME} \
 && cd ${FLUTTER_HOME} \
 && git remote add origin https://github.com/flutter/flutter.git \
 && git fetch --depth 1 origin ${flutter_commit} \
 && git checkout FETCH_HEAD

# Accept licenses and run doctor. Fail loudly if precache fails: no `|| true`
# masking a broken artifact cache.
RUN yes | flutter doctor --android-licenses > /dev/null \
 && flutter doctor \
 && flutter precache --android

# Make only the Flutter cache writable for non-root users.
# Toolchain binaries, SDK packages, and .git metadata stay read-only
# to prevent untrusted modification of build tools.
# See Project516/docker-images-flutter#6.
RUN chmod -R a+rwX "${FLUTTER_HOME}/bin/cache" \
 && git config --system --add safe.directory "${FLUTTER_ROOT}"
