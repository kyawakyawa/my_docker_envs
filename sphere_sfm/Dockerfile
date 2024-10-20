FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

RUN apt-get update && apt-get install -y \
     git \
     cmake \
     ninja-build \
     build-essential \
     libboost-program-options-dev \
     libboost-filesystem-dev \
     libboost-graph-dev \
     libboost-system-dev \
     libeigen3-dev \
     libflann-dev \
     libfreeimage-dev \
     libmetis-dev \
     libgoogle-glog-dev \
     libgtest-dev \
     libgmock-dev \
     libsqlite3-dev \
     libglew-dev \
     qtbase5-dev \
     libqt5opengl5-dev \
     libcgal-dev \
     libceres-dev

RUN mkdir -p /src && cd /src \
 && git clone https://github.com/json87/SphereSfM.git \
 && cd SphereSfM && git checkout 24d63adc0410c38f18608715d2ad5d969d8050aa && cd .. \
 && git clone https://github.com/colmap/colmap.git \
 && cd colmap && git checkout 3.10 && cd .. \
 && cp SphereSfM/lib/SiftGPU/CMakeLists.txt . \
 && rm SphereSfM/lib/SiftGPU/* \
 && cp colmap/src/thirdparty/SiftGPU/* SphereSfM/lib/SiftGPU/ \
 && cp CMakeLists.txt SphereSfM/lib/SiftGPU/ \
 && cd SphereSfM && mkdir build && cd build \
 && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -GNinja -DCMAKE_CUDA_ARCHITECTURES=61 .. \
 && ninja && ninja install

WORKDIR /mnt
CMD ["/usr/local/bin/colmap", "--help"]
