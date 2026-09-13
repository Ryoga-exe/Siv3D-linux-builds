# syntax=docker/dockerfile:1.7

ARG UBUNTU_VERSION=22.04
FROM ubuntu:${UBUNTU_VERSION} AS toolchain

ARG DEBIAN_FRONTEND=noninteractive

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
        libjpeg-dev \
        libmpg123-dev \
        libogg-dev \
        libopus-dev \
        libopusfile-dev \
        libpng-dev \
        libsoundtouch-dev \
        libswresample-dev \
        libswscale-dev \
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

FROM toolchain AS opencv-build

ARG OPENCV_REF=4.5.1
ARG OPENCV_COMMIT=1363496c1106606684d40447f5d1149b2c66a9f8

RUN git init /tmp/OpenCV && \
    git -C /tmp/OpenCV remote add origin https://github.com/opencv/opencv.git && \
    git -C /tmp/OpenCV fetch --depth 1 origin "refs/tags/${OPENCV_REF}" && \
    git -C /tmp/OpenCV checkout --detach FETCH_HEAD && \
    test "$(git -C /tmp/OpenCV rev-parse HEAD)" = "${OPENCV_COMMIT}"

RUN cmake \
        -S /tmp/OpenCV \
        -B /tmp/OpenCV/build \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/opt/opencv \
        -DCMAKE_INSTALL_LIBDIR=lib \
        -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
        -DBUILD_SHARED_LIBS=OFF \
        -DBUILD_LIST=core,imgproc,imgcodecs,objdetect,photo,videoio,world \
        -DBUILD_opencv_world=ON \
        -DBUILD_EXAMPLES=OFF \
        -DBUILD_JAVA=OFF \
        -DBUILD_opencv_apps=OFF \
        -DBUILD_opencv_highgui=OFF \
        -DBUILD_opencv_python3=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DBUILD_TESTS=OFF \
        -DBUILD_JPEG=OFF \
        -DBUILD_PNG=OFF \
        -DBUILD_TIFF=OFF \
        -DBUILD_WEBP=OFF \
        -DBUILD_ZLIB=OFF \
        -DWITH_CUDA=OFF \
        -DWITH_EIGEN=OFF \
        -DWITH_FFMPEG=ON \
        -DWITH_GSTREAMER=OFF \
        -DWITH_GTK=OFF \
        -DWITH_IPP=OFF \
        -DWITH_ITT=OFF \
        -DWITH_JASPER=OFF \
        -DWITH_LAPACK=OFF \
        -DWITH_OPENCL=OFF \
        -DWITH_OPENEXR=OFF \
        -DWITH_OPENJPEG=OFF \
        -DWITH_QT=OFF \
        -DWITH_TBB=OFF \
        -DWITH_VTK=OFF \
        -DOPENCV_GENERATE_PKGCONFIG=ON

RUN --mount=type=cache,id=opencv-ccache,target=/var/cache/ccache \
    cmake --build /tmp/OpenCV/build --parallel && \
    cmake --install /tmp/OpenCV/build && \
    mkdir -p /opt/opencv/licenses && \
    install -m 0644 /tmp/OpenCV/LICENSE /opt/opencv/licenses/OpenCV-LICENSE && \
    install -m 0644 /tmp/OpenCV/COPYRIGHT /opt/opencv/licenses/OpenCV-COPYRIGHT && \
    install -m 0644 \
        /tmp/OpenCV/doc/LICENSE_CHANGE_NOTICE.txt \
        /opt/opencv/licenses/OpenCV-LICENSE-CHANGE-NOTICE && \
    install -m 0644 \
        /tmp/OpenCV/3rdparty/quirc/LICENSE \
        /opt/opencv/licenses/OpenCV-quirc-LICENSE

FROM toolchain AS siv3d-build

ARG SIV3D_REF=v0.6.16
ARG SIV3D_COMMIT=fdc5662048a27974c989c3dac1fc38d41074e62b

ENV PKG_CONFIG_PATH=/opt/opencv/lib/pkgconfig

COPY --from=opencv-build /opt/opencv /opt/opencv

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

RUN --mount=type=cache,id=siv3d-ccache,target=/var/cache/ccache \
    cmake --build /tmp/OpenSiv3D/Linux/build --parallel

COPY docker/sdk/Siv3DConfig.cmake /tmp/Siv3DConfig.cmake

RUN cmake --install /tmp/OpenSiv3D/Linux/build && \
    mkdir -p \
        /opt/siv3d-sdk/include \
        /opt/siv3d-sdk/lib/cmake \
        /opt/siv3d-sdk/licenses \
        /opt/siv3d-sdk/share/Siv3D && \
    mv \
        /opt/siv3d-sdk/lib/cmake/Siv3D/Siv3DConfig.cmake \
        /opt/siv3d-sdk/lib/cmake/Siv3D/Siv3DTargets.cmake && \
    install -m 0644 \
        /tmp/Siv3DConfig.cmake \
        /opt/siv3d-sdk/lib/cmake/Siv3D/Siv3DConfig.cmake && \
    cp -a /opt/opencv/include/opencv4 /opt/siv3d-sdk/include/opencv4 && \
    install -m 0644 /opt/opencv/lib/libopencv_world.a /opt/siv3d-sdk/lib/libopencv_world.a && \
    if [ -d /opt/opencv/lib/opencv4/3rdparty ]; then \
        mkdir -p /opt/siv3d-sdk/lib/opencv4 && \
        cp -a /opt/opencv/lib/opencv4/3rdparty /opt/siv3d-sdk/lib/opencv4/3rdparty; \
    fi && \
    cp -a /tmp/OpenSiv3D/Linux/App/resources /opt/siv3d-sdk/share/Siv3D/resources && \
    install -m 0644 /tmp/OpenSiv3D/LICENSE /opt/siv3d-sdk/licenses/OpenSiv3D-LICENSE && \
    install -m 0644 \
        /tmp/OpenSiv3D/Siv3D/src/Siv3D/LicenseManager/LicenseList.hpp \
        /opt/siv3d-sdk/licenses/OpenSiv3D-THIRD-PARTY-NOTICES.hpp && \
    cp -a /opt/opencv/licenses/. /opt/siv3d-sdk/licenses/

FROM siv3d-build AS test

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

COPY --from=test /opt/siv3d-sdk/ /
