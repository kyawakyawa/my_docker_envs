FROM archlinux:latest

# タイムゾーン
ENV TZ Asia/Tokyo

RUN echo Server = http://ftp.jaist.ac.jp/pub/Linux/ArchLinux/\$repo/os/\$arch  > etc/pacman.d/mirrorlist \
    && pacman -Syu --noconfirm \
    && pacman -S --noconfirm base-devel vim neovim clang openmp gcc cmake make lldb git ninja openssh tmux fish sudo unzip \
    && pacman -S --noconfirm glfw-x11 spirv-tools vulkan-headers vulkan-icd-loader vulkan-intel vulkan-tools vulkan-validation-layers \
    && pacman -S --noconfirm intel-mkl ffmpeg protobuf openssl \
    && pacman -Sc --noconfirm \
    && useradd cpp -m && echo cpp:passward | chpasswd \
    && echo -e "cpp ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN mkdir -p /root/install && cd /root/install \
    && rm -rf eigen && git clone https://gitlab.com/libeigen/eigen.git \
    && cd eigen && git checkout 3.4.0 \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf opencv && git clone https://github.com/opencv/opencv.git \
    && cd opencv && git checkout 4.5.3 \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -DENABLE_CXX11=ON \
                            -DBUILD_DOCS=OFF \
                            -DBUILD_EXAMPLES=OFF \
                            -DBUILD_JASPER=OFF \
                            -DBUILD_OPENEXR=OFF \
                            -DBUILD_PERF_TESTS=OFF \
                            -DBUILD_TESTS=OFF \
                            -DWITH_EIGEN=ON \
                            -DWITH_FFMPEG=ON \
                            -DWITH_OPENMP=ON \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf spdlog && git clone https://github.com/gabime/spdlog.git \
    && cd spdlog && git checkout v1.9.2 \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf googletest && git clone https://github.com/google/googletest.git \
    && cd googletest && git checkout release-1.11.0  \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf gflags && git clone https://github.com/gflags/gflags.git \
    && cd gflags && git checkout v2.2.2  \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -DBUILD_SHARED_LIBS=ON \
                            -DBUILD_STATIC_LIBS=OFF \
                            -DINSTALL_SHARED_LIBS=ON \
                            -DINSTALL_STATIC_LIBS=OFF \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf glog && git clone https://github.com/google/glog.git \
    && cd glog && git checkout v0.5.0  \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -DBUILD_SHARED_LIBS=ON \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf ceres-solver && git clone https://github.com/ceres-solver/ceres-solver.git \
    && cd ceres-solver && git checkout 2.0.0  \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -DSUITESPARSE=OFF \
                            -DCXSPARSE=OFF \
                            -DBUILD_SHARED_LIBS=ON \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf DBoW2 && git clone https://github.com/shinsumicco/DBoW2.git \
    && cd DBoW2 \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && rm -rf socket.io-client-cpp && git clone https://github.com/shinsumicco/socket.io-client-cpp.git \
    && cd socket.io-client-cpp && git submodule init && git submodule update \
    && CC=clang CXX=clang++ cmake \
                            -DCMAKE_BUILD_TYPE=Release \
                            -DCMAKE_INSTALL_PREFIX=/usr/local \
                            -DBUILD_UNIT_TESTS=OFF \
                            -Bbuild -H. -GNinja \
    && cd build && ninja && ninja install && cd ../../ \
    && cd /root && rm -r install

USER 1000:1000

RUN cd /tmp/ && git clone https://aur.archlinux.org/yay-bin.git yay-bin && cd yay-bin \
    && makepkg -si --noconfirm && cd .. && rm -rf yay-bin && cd / \
    && curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish \
    && fish -c "fisher install oh-my-fish/theme-bobthefish" \
    && cd /tmp/ && curl https://raw.githubusercontent.com/Shougo/dein.vim/master/bin/installer.sh > installer.sh \
    && sh ./installer.sh ~/.cache/dein && rm ./installer.sh \
    && cd ~/ && git clone -b server https://github.com/kyawakyawa/dotfiles.git && cd dotfiles && ./dotfilesLink.sh \
    && curl -fsSL https://deno.land/x/install/install.sh | sh \
    && echo -e "export DENO_INSTALL=\"\$HOME/.deno\"" >> ~/.bashrc && echo -e "export PATH=\"\$DENO_INSTALL/bin:\$PATH\"" >> ~/.bashrc \
    && cd ~/ && mkdir bin && echo -e "export PATH=\"\$PATH\":\"\$HOME/bin\"" >> ~/.bashrc

COPY ide /home/cpp/bin/

