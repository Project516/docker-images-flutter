FROM ghcr.io/cirruslabs/android-sdk:36

LABEL org.opencontainers.image.source=https://github.com/cirruslabs/docker-images-flutter
USER root

ARG flutter_version

ENV FLUTTER_HOME=${HOME}/sdks/flutter \
    FLUTTER_VERSION=$flutter_version
ENV FLUTTER_ROOT=$FLUTTER_HOME

ENV PATH ${PATH}:${FLUTTER_HOME}/bin:${FLUTTER_HOME}/bin/cache/dart-sdk/bin

RUN git clone --depth 1 --branch ${FLUTTER_VERSION} https://github.com/flutter/flutter.git ${FLUTTER_HOME}

RUN yes | flutter doctor --android-licenses \
    && flutter doctor

RUN flutter precache --android

# Make the SDK writable by any user and mark it as a safe git directory.
# This allows running the container with --user "$(id -u):$(id -g)" so bind-mounted
# worktrees stay owned by the host user instead of filling up with root-owned build output.
# See https://github.com/Project516/docker-images-flutter/issues/6
RUN chmod -R a+rwX "$FLUTTER_ROOT" \
 && git config --system --add safe.directory "$FLUTTER_ROOT"
