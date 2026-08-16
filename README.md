# Docker images for Flutter

Maintained fork of [cirruslabs/docker-images-flutter][upstream], which
[stopped publishing new images on 1 May 2026][notice] when Cirrus Labs wound
down after an acquisition. Its `stable` tag is pinned to Flutter 3.44.0 and
will not move again.

This fork keeps the images coming, and does it without anybody having to
remember to.

```bash
docker run --rm -it -v "$PWD":/build --workdir /build \
  ghcr.io/project516/flutter:stable flutter test
```

## Tags

| Tag | What it is |
|---|---|
| `stable`, `latest` | the current Flutter stable |
| `beta` | the current Flutter beta |
| `3.47.0`, `3.44.4`, … | one exact version, never moved |

Pin the exact version in CI. `stable` is convenient locally and a moving
target everywhere else, which is how a green format gate locally and a red one
in CI stop being contradictory.

## How it stays current

Upstream kept the version in a matrix in `.cirrus.yml`, so every Flutter
release needed a human to edit a file. That is the maintenance this fork
removes.

`.github/workflows/build.yml` runs daily and asks Flutter's own release
manifest what the current stable and beta are. A channel already published is
skipped, so an ordinary day costs a few seconds and builds nothing. When a new
version lands, it builds and publishes itself.

Each architecture builds on its own native runner (`ubuntu-latest` for amd64,
`ubuntu-24.04-arm` for arm64) rather than under QEMU, then a merge job stitches
the two into one manifest list. Emulating `flutter precache` and the Android
SDK layer turns a ten-minute build into an hours-long one, and the tags only go
on the manifest, so a half-finished build never leaves `stable` pointing at one
architecture.

To build a version by hand, run the workflow from the Actions tab. It takes a
channel and a force flag for rebuilding something already published.

## Publishing somewhere else

GHCR needs no configuration: the built-in `GITHUB_TOKEN` is enough.

Docker Hub is opt-in. Set the repository variable `DOCKERHUB_REPO` (for example
`project516/flutter`) and the secrets `DOCKERHUB_USERNAME` and
`DOCKERHUB_TOKEN`, and each published manifest is copied there too. Leave them
unset and that job is skipped, so a fork works with no setup at all.

## Known dependency

`sdk/Dockerfile` builds on `ghcr.io/cirruslabs/android-sdk:36`, which is from
the same wound-down project and is also frozen. It works today; if the Android
SDK level needs to move, that base image is the thing to replace, and it is the
one piece of upstream this fork still depends on.

## Licence

Apache 2.0, as upstream. See [LICENSE](LICENSE).

[upstream]: https://github.com/cirruslabs/docker-images-flutter
[notice]: https://github.com/cirruslabs/docker-images-flutter/blob/master/README.md
