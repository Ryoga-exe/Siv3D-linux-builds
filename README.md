# Siv3D Linux Builds

Unofficial prebuilt OpenSiv3D SDK archives for Linux.

This repository builds OpenSiv3D with its official CMake configuration and publishes the resulting headers, static libraries, and runtime resources as versioned GitHub Release assets.

This project is not affiliated with or endorsed by the OpenSiv3D project.

## Compatibility

- OpenSiv3D 0.6.16
- Bundled OpenCV 4.5.1
- Bundled FFmpeg 4.4.8 shared libraries
- Bundled SoundTouch 2.3.1 shared library
- x86_64 Linux with glibc and libstdc++
- Built and tested on Ubuntu 22.04 with GCC 11
- CMake `Release` configuration

Other glibc-based distributions, including Arch Linux, may work but are not currently tested.
The SDK does not target musl-based distributions such as Alpine Linux. Applications are linked
against most system libraries of the target distribution. OpenCV, FFmpeg, and SoundTouch are pinned
and bundled with the SDK instead of being provided by the target system.

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
  libSoundTouch.so -> libSoundTouch.so.2
  libSoundTouch.so.2 -> libSoundTouch.so.2.3.1
  libSoundTouch.so.2.3.1
  libavcodec.so -> libavcodec.so.58
  libavcodec.so.58 -> libavcodec.so.58.134.100
  libavcodec.so.58.134.100
  libavformat.so -> libavformat.so.58
  libavformat.so.58 -> libavformat.so.58.76.100
  libavformat.so.58.76.100
  libavutil.so -> libavutil.so.56
  libavutil.so.56 -> libavutil.so.56.70.100
  libavutil.so.56.70.100
  libswresample.so -> libswresample.so.3
  libswresample.so.3 -> libswresample.so.3.9.100
  libswresample.so.3.9.100
  libswscale.so -> libswscale.so.5
  libswscale.so.5 -> libswscale.so.5.9.100
  libswscale.so.5.9.100
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
  SoundTouch-COPYING.txt
  FFmpeg-LICENSE.md
  FFmpeg-COPYING.LGPLv2.1
  FFmpeg-BUILD.txt
sources/
  SoundTouch-2.3.1.tar.gz
  FFmpeg-4.4.8.tar.gz
```

The archive contains OpenSiv3D, a pinned static OpenCV build, and pinned shared FFmpeg and SoundTouch
builds. The FFmpeg build disables GPL, version 3, nonfree, and automatically detected external
components; its only explicitly enabled external dependency is zlib. Corresponding FFmpeg and
SoundTouch sources, licenses, and the FFmpeg configure options are included to make LGPL compliance
straightforward. The remaining Linux dependencies must be provided by the target distribution, and
applications must use the system libstdc++ ABI. Applications also need to deploy the bundled shared
libraries next to the executable or otherwise make the SDK `lib/` directory available to the dynamic
loader.

## System dependencies on Ubuntu

On Ubuntu 22.04, install the development packages required by OpenSiv3D before linking an
application. On other distributions, install the equivalent packages using the system package
manager.

```sh
sudo apt-get update
sudo apt-get install -y \
  libasound2-dev libboost-dev libcurl4-openssl-dev libfreetype6-dev \
  libgif-dev libgl1-mesa-dev \
  libglib2.0-dev libglu1-mesa-dev libgtk-3-dev libharfbuzz-dev libjpeg-dev \
  libmpg123-dev libogg-dev libopus-dev libopusfile-dev libpng-dev \
  libtiff-dev libturbojpeg0-dev libvorbis-dev libwebp-dev libxft-dev \
  pkg-config uuid-dev xorg-dev zlib1g-dev
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
v0.6.16-r3
siv3d-v0.6.16-linux-gnu-x86_64.tar.gz
```

When changing the Siv3D, OpenCV, FFmpeg, or SoundTouch version, update its pinned tag and commit in
the Dockerfile and workflow together. Also update `docker/sdk/FFmpeg-BUILD.txt` when changing FFmpeg.
The build checks that each tag resolves to the expected commit before compiling.

## Acknowledgements

OpenSiv3D is developed by the [OpenSiv3D project](https://github.com/Siv3D/OpenSiv3D) and distributed under the MIT License.
OpenCV is developed by the [OpenCV project](https://github.com/opencv/opencv) and distributed under
the Apache License 2.0.
SoundTouch is developed by the [SoundTouch project](https://codeberg.org/soundtouch/soundtouch) and
distributed under the GNU Lesser General Public License 2.1.
FFmpeg is developed by the [FFmpeg project](https://ffmpeg.org/) and this repository configures it
for distribution under the GNU Lesser General Public License 2.1 or later.
