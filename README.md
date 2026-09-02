# Siv3D Linux Builds

Unofficial prebuilt OpenSiv3D SDK archives for Linux.

This repository builds OpenSiv3D with its official CMake configuration and publishes the resulting headers, static library, and runtime resources as versioned GitHub Release assets.

This project is not affiliated with or endorsed by the OpenSiv3D project.

## Supported environment

- OpenSiv3D 0.6.16
- Ubuntu 22.04
- x86_64
- GCC 11 and the system libstdc++ ABI
- CMake `Release` configuration

The SDK is intended for native Ubuntu 22.04 builds.
Compatibility with other distributions or Ubuntu releases is not guaranteed because Siv3D uses C++ system libraries such as OpenCV.

## Archive layout

Each release contains a gzip-compressed tar archive with the following layout:

```text
include/
  Siv3D/
    Siv3D.hpp
    HamFramework.hpp
    Siv3D/
    ThirdParty/
lib/
  libSiv3D.a
  cmake/Siv3D/
share/
  Siv3D/resources/engine/
licenses/
  OpenSiv3D-LICENSE
```

The archive contains OpenSiv3D itself, but not its Ubuntu system dependencies.
Applications must use the system libstdc++ ABI and link the system dependencies listed below.

## System dependencies

Install the development packages required by OpenSiv3D before linking an application:

```sh
sudo apt-get update
sudo apt-get install -y \
  libasound2-dev libavcodec-dev libavformat-dev libavutil-dev libboost-dev \
  libcurl4-openssl-dev libfreetype6-dev libgif-dev libgl1-mesa-dev \
  libglib2.0-dev libglu1-mesa-dev libgtk-3-dev libharfbuzz-dev libmpg123-dev \
  libogg-dev libopencv-dev libopus-dev libopusfile-dev libpng-dev \
  libsoundtouch-dev libswresample-dev libtiff-dev libturbojpeg0-dev \
  libvorbis-dev libwebp-dev libxft-dev pkg-config uuid-dev xorg-dev zlib1g-dev
```

## Build locally

Docker Buildx can reproduce the SDK without modifying the host system:

```sh
docker buildx build \
  --file docker/sdk/Dockerfile \
  --target artifact \
  --platform linux/amd64 \
  --output type=local,dest=dist/sdk \
  .
```

The exported SDK will be written to `dist/sdk`.

## Publishing

The `Release Linux SDK` workflow builds the SDK, checks its layout, creates a normalized tar.gz
archive and SHA-256 checksum, and publishes both files to GitHub Releases. It runs for the expected
release tag or can be started manually. Published assets are immutable; increment `SDK_REVISION`
when the SDK contents need to change.

The release naming scheme separates the upstream Siv3D version, target environment, and packaging
revision:

```text
v0.6.16-r1
siv3d-v0.6.16-ubuntu-22.04-x86_64.tar.gz
```

When changing the Siv3D version, update the pinned tag and commit in the Dockerfile and workflow
together. The build checks that the tag resolves to the expected commit before compiling.

## Acknowledgements

OpenSiv3D is developed by the [OpenSiv3D project](https://github.com/Siv3D/OpenSiv3D) and distributed under the MIT License.
