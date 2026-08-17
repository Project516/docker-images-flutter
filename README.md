# Docker images for Flutter

Container images with the Flutter SDK and the Android SDK, published for every
Flutter stable and beta release.

Started as a fork of [cirruslabs/docker-images-flutter][upstream], which
[stopped publishing on 1 May 2026][notice] when Cirrus Labs wound down after an
acquisition. Its `stable` tag is frozen at Flutter 3.44.0 and will not move
again. This is not a patch queue waiting to go upstream: there is no upstream
left to send it to, so the images are maintained here.

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

To build a version by hand, run the workflow from the Actions tab. It takes a
channel and a force flag for rebuilding something already published.

## Publishing somewhere else

GHCR needs no configuration: the built-in `GITHUB_TOKEN` is enough.

Docker Hub is opt-in. Set the repository variable `DOCKERHUB_REPO` (for example
`project516/flutter`) and the secrets `DOCKERHUB_USERNAME` and
`DOCKERHUB_TOKEN`, and each published manifest is copied there too. Leave them
unset and that job is skipped, so a fork of this works with no setup at all.

## Known rough edges

`sdk/Dockerfile` builds on `ghcr.io/cirruslabs/android-sdk:36`, which is from
the same wound-down project and is also frozen. It works today; if the Android
SDK level needs to move, that base image is the thing to replace, and it is the
one piece of upstream still in the chain.

On arm64 that base image ships an amd64 `adb`, so `flutter doctor` reports it
cannot run `adb`. The build succeeds and the image is fine for compiling and
testing, which is what it is for. Worth knowing before someone reads the doctor
output and assumes the image is broken.

## Issues

Open one. Bugs, a Flutter version that did not publish, or a request for
another base image are all fair.

## Licence

MIT. See [LICENSE](LICENSE).

[upstream]: https://github.com/cirruslabs/docker-images-flutter
[notice]: https://github.com/cirruslabs/docker-images-flutter/blob/master/README.md
