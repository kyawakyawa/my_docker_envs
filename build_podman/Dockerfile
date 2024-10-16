FROM ubuntu:24.04

ENV GOPATH=/go

RUN apt-get update && apt-get -y install \
  btrfs-progs \
  crun \
  git \
  golang-go \
  go-md2man \
  iptables \
  libassuan-dev \
  libbtrfs-dev \
  libc6-dev \
  libdevmapper-dev \
  libglib2.0-dev \
  libgpgme-dev \
  libgpg-error-dev \
  libprotobuf-dev \
  libprotobuf-c-dev \
  libseccomp-dev \
  libselinux1-dev \
  libsystemd-dev \
  make \
  netavark \
  pkg-config \
  uidmap && \
  git clone https://go.googlesource.com/go $GOPATH && \
  cd $GOPATH && \
  git checkout go1.22.8 && \
  cd src && \
  ./all.bash && \
  export PATH=$GOPATH/bin:$PATH && \
  cd / && \
  git clone https://github.com/containers/conmon && \
  cd conmon && \
  git checkout v2.1.12 && \
  export GOCACHE="$(mktemp -d)" && \
  make -j16 && \
  make podman && \
  cd / && rm -rf /conmon && \
  git clone https://github.com/containers/podman/ && \
  cd podman && git checkout v5.2.4 && \
  make -j16 BUILDTAGS="selinux seccomp cni" PREFIX=/usr && \
  make install PREFIX=/usr && \
  cd / && rm -rf /podman && \
  rm -rf /$GOPATH && \
  apt-get -y purge \
  btrfs-progs \
  crun \
  git \
  golang-go \
  go-md2man \
  iptables \
  libassuan-dev \
  libbtrfs-dev \
  libc6-dev \
  libdevmapper-dev \
  libglib2.0-dev \
  libgpgme-dev \
  libgpg-error-dev \
  libprotobuf-dev \
  libprotobuf-c-dev \
  libseccomp-dev \
  libselinux1-dev \
  libsystemd-dev \
  make \
  netavark \
  pkg-config \
  uidmap && \
  apt-get clean

CMD sh -c "cp /usr/bin/podman /mnt && cp /usr/local/libexec/podman/conmon /mnt"
