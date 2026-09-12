# Docker images for Flutter

Container images with the Flutter SDK and the Android SDK, published for every
Flutter stable and beta release.

Originally started as a fork of [cirruslabs/docker-images-flutter](https://github.com/cirruslabs/docker-images-flutter).

```bash
podman run --rm -it -v "$PWD":/build:z --workdir /build \
  ghcr.io/project516/flutter:stable flutter test
```

Docker works the same way, without the `:z`:

```bash
docker run --rm -it -v "$PWD":/build --workdir /build \
  ghcr.io/project516/flutter:stable flutter test
```

These are OCI images in a standard registry, so podman, Docker, nerdctl,
Kubernetes and anything else that speaks OCI can all pull them. Nothing about
the build is Docker-specific.

## Tags

| Tag | What it is |
|---|---|
| `stable`, `latest` | the current Flutter stable |
| `beta` | the current Flutter beta |
| `3.47.0`, `3.46.0`, … | one exact version, never moved |

`linux/amd64` and `linux/arm64` under every tag.

**Pin the exact version in CI.** `stable` is convenient locally and a moving
target everywhere else, which is how a green format gate on a laptop and a red
one in CI stop being contradictory: `dart format` output changes between Dart
minor versions.

## Using it with podman

Two things bite on SELinux systems (Fedora, RHEL) and neither is specific to
these images:

- **Label the mounts with `:z`.** Without it the container cannot read the
  bind mount. Use `:z` (shared) rather than `:Z` (exclusive) if more than one
  container touches the same directory, or a second container silently loses
  access to it.
- **Set `PUB_CACHE` to a mounted path** if you want the pub cache to survive
  between runs, otherwise every run re-downloads:

```bash
podman run --rm \
  -v "$PWD":/repo:z \
  -v "$HOME/.pub-cache-container":/pubcache:z \
  -w /repo -e PUB_CACHE=/pubcache \
  ghcr.io/project516/flutter:stable \
  bash -lc 'flutter pub get && flutter test'
```

Rootless podman runs as root inside the container, so `flutter` prints a
warning about running as a superuser. It is noise here, not a problem.

## How it stays current

Upstream kept the Flutter version in a matrix in `.cirrus.yml`, so every
release needed a human to edit a file. When the humans left, the images
stopped. That is the failure mode this repo is built to avoid.

`.github/workflows/build.yml` runs daily and asks Flutter's own release
manifest what the current stable and beta are. A channel already published is
skipped, so an ordinary day builds nothing and costs seconds. When a new
version lands, it builds and publishes itself.

Each architecture builds on its own native runner (`ubuntu-latest` for amd64,
`ubuntu-24.04-arm` for arm64) rather than under QEMU, then a merge job stitches
the two into one manifest list. Emulating `flutter precache` and the Android
SDK layer turns a ten-minute build into an hours-long one. Tags go only on the
manifest, so a half-finished build never leaves `stable` pointing at one
architecture.

## Publishing somewhere else

GHCR needs no configuration: the built-in `GITHUB_TOKEN` is enough.

Docker Hub is opt-in. Set the repository variable `DOCKERHUB_REPO` (for example
`project516/docker-images-flutter`) and the secrets `DOCKERHUB_USERNAME` and
`DOCKERHUB_TOKEN`, and each published manifest is copied there too. Leave them
unset and that job is skipped, so a fork of this works with no setup at all.

`DOCKERHUB_TOKEN` is a Docker Hub personal access token with Read & Write on
the target repository, not an account password.

The mirror runs as part of a release. To put a tag on Docker Hub that published
before the mirror was configured, run the "Mirror to Docker Hub" workflow from
the Actions tab with that version tag. It copies the existing manifest list out
of GHCR, so nothing is rebuilt.

## Known rough edges

`sdk/Dockerfile` builds on `eclipse-temurin:21-jdk-noble` (pinned to an
immutable digest) and installs the Android SDK directly via Google's
command-line tools. No Cirrus Labs Android SDK image dependency remains.

On arm64, Google does not publish a native Linux `platform-tools` package, so
`adb` is x86_64-only. `flutter doctor` will report it cannot run `adb`, but the
build succeeds and the image is fine for compiling and testing. Worth knowing
before someone reads the doctor output and assumes the image is broken.

## Licence

MIT. See [LICENSE](LICENSE).
