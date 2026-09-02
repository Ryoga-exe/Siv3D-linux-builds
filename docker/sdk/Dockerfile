# syntax=docker/dockerfile:1.7

ARG UBUNTU_VERSION=22.04
FROM ubuntu:${UBUNTU_VERSION} AS build

ARG DEBIAN_FRONTEND=noninteractive
ARG SIV3D_REF=v0.6.16
ARG SIV3D_COMMIT=fdc5662048a27974c989c3dac1fc38d41074e62b

ENV CCACHE_DIR=/var/cache/ccache \
    CMAKE_C_COMPILER_LAUNCHER=ccache \
    CMAKE_CXX_COMPILER_LAUNCHER=ccache

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        ccache \
        cmake \
        g++ \
        gcc \
        git \
        libasound2-dev \
        libavcodec-dev \
        libavformat-dev \
        libavutil-dev \
        libboost-dev \
        libcurl4-openssl-dev \
        libfreetype6-dev \
        libgif-dev \
        libgl1-mesa-dev \
        libglib2.0-dev \
        libglu1-mesa-dev \
        libgtk-3-dev \
        libharfbuzz-dev \
        libmpg123-dev \
        libogg-dev \
        libopencv-dev \
        libopus-dev \
        libopusfile-dev \
        libpng-dev \
        libsoundtouch-dev \
        libswresample-dev \
        libtiff-dev \
        libturbojpeg0-dev \
        libvorbis-dev \
        libwebp-dev \
        libxft-dev \
        ninja-build \
        pkg-config \
        uuid-dev \
        xorg-dev \
        zlib1g-dev && \
    rm -rf /var/lib/apt/lists/*

RUN test "$(dpkg --print-architecture)" = "amd64"

RUN git init /tmp/OpenSiv3D && \
    git -C /tmp/OpenSiv3D remote add origin https://github.com/Siv3D/OpenSiv3D.git && \
    git -C /tmp/OpenSiv3D fetch --depth 1 origin "refs/tags/${SIV3D_REF}" && \
    git -C /tmp/OpenSiv3D checkout --detach FETCH_HEAD && \
    test "$(git -C /tmp/OpenSiv3D rev-parse HEAD)" = "${SIV3D_COMMIT}"

RUN cmake \
        -S /tmp/OpenSiv3D/Linux \
        -B /tmp/OpenSiv3D/Linux/build \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/opt/siv3d-sdk

RUN --mount=type=cache,target=/var/cache/ccache \
    cmake --build /tmp/OpenSiv3D/Linux/build --parallel

RUN cmake --install /tmp/OpenSiv3D/Linux/build && \
    mkdir -p /opt/siv3d-sdk/share/Siv3D /opt/siv3d-sdk/licenses && \
    cp -a /tmp/OpenSiv3D/Linux/App/resources /opt/siv3d-sdk/share/Siv3D/resources && \
    install -m 0644 /tmp/OpenSiv3D/LICENSE /opt/siv3d-sdk/licenses/OpenSiv3D-LICENSE

RUN cmake \
        -S /tmp/OpenSiv3D/Linux/App \
        -B /tmp/OpenSiv3D/Linux/App/sdk-test \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_PREFIX_PATH=/opt/siv3d-sdk \
        -DBUILD_TESTING=ON && \
    cmake --build /tmp/OpenSiv3D/Linux/App/sdk-test --parallel && \
    ctest \
        --test-dir /tmp/OpenSiv3D/Linux/App/sdk-test \
        --output-on-failure

FROM scratch AS artifact

COPY --from=build /opt/siv3d-sdk/ /
