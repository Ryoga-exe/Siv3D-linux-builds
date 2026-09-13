# Siv3D Linux Builds

Unofficial prebuilt OpenSiv3D SDK archives for Linux.

This repository builds OpenSiv3D with its official CMake configuration and publishes the resulting headers, static libraries, and runtime resources as versioned GitHub Release assets.

This project is not affiliated with or endorsed by the OpenSiv3D project.

## Compatibility

- OpenSiv3D 0.6.16
- Bundled OpenCV 4.5.1
- x86_64 Linux with glibc and libstdc++
- Built and tested on Ubuntu 22.04 with GCC 11
- CMake `Release` configuration

Other glibc-based distributions, including Arch Linux, may work but are not currently tested.
The SDK does not target musl-based distributions such as Alpine Linux. Applications are linked
against the system libraries of the target distribution, so compatibility also depends on the
available FFmpeg and other library versions. OpenCV is pinned and bundled with the SDK instead of
being provided by the target system.

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
  libopencv_world.a
  cmake/Siv3D/
  opencv4/3rdparty/
share/
  Siv3D/resources/engine/
licenses/
  OpenSiv3D-LICENSE
  OpenSiv3D-THIRD-PARTY-NOTICES.hpp
  OpenCV-LICENSE
  OpenCV-COPYRIGHT
  OpenCV-LICENSE-CHANGE-NOTICE
  OpenCV-quirc-LICENSE
```

The archive contains OpenSiv3D and a pinned static OpenCV build, but not the remaining Linux system
dependencies. Applications must use the system libstdc++ ABI and link the corresponding
dependencies from the target distribution.

## System dependencies on Ubuntu

On Ubuntu 22.04, install the development packages required by OpenSiv3D before linking an
application. On other distributions, install the equivalent packages using the system package
manager.

```sh
sudo apt-get update
sudo apt-get install -y \
  libasound2-dev libavcodec-dev libavformat-dev libavutil-dev libboost-dev \
  libcurl4-openssl-dev libfreetype6-dev libgif-dev libgl1-mesa-dev \
  libglib2.0-dev libglu1-mesa-dev libgtk-3-dev libharfbuzz-dev libjpeg-dev \
  libmpg123-dev libogg-dev libopus-dev libopusfile-dev libpng-dev \
  libsoundtouch-dev libswresample-dev libswscale-dev libtiff-dev libturbojpeg0-dev \
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
v0.6.16-r2
siv3d-v0.6.16-linux-gnu-x86_64.tar.gz
```

When changing the Siv3D or OpenCV version, update its pinned tag and commit in the Dockerfile and
workflow together. The build checks that each tag resolves to the expected commit before compiling.

## Acknowledgements

OpenSiv3D is developed by the [OpenSiv3D project](https://github.com/Siv3D/OpenSiv3D) and distributed under the MIT License.
OpenCV is developed by the [OpenCV project](https://github.com/opencv/opencv) and distributed under
the Apache License 2.0.
