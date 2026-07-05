# syntax=docker/dockerfile:1.4

FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    git \
    xz-utils \
    bzip2 \
    gzip \
    tar \
    file \
    make \
    gcc \
    g++ \
    build-essential \
    pkg-config \
    autoconf \
    automake \
    libtool \
    perl \
    python3 \

    libmunge-dev \
    munge \
    libpam0g-dev \
    libhwloc-dev \
    libnuma-dev \
    libreadline-dev \
    libssl-dev \
    libjson-c-dev \
    libhttp-parser-dev \
    libyaml-dev \
    libjwt-dev \
    libdbus-1-dev \
    libbpf-dev \
    zlib1g-dev \
    liblz4-dev \
    libmariadb-dev \
    && rm -rf /var/lib/apt/lists/*

# SchedMD official download default.
#
# 通常は SLURM_VERSION だけ変えればよい。
# 例:
#   SLURM_VERSION=22.05.11
#   -> https://download.schedmd.com/slurm/slurm-22.05.11.tar.bz2
#
#   SLURM_VERSION=22.05-latest
#   -> https://download.schedmd.com/slurm/slurm-22.05-latest.tar.bz2
ENV SLURM_VERSION=22.05.11
ENV SLURM_BASE_URL=https://download.schedmd.com/slurm

# 空なら実行時に slurm-${SLURM_VERSION}.tar.bz2 を使う
ENV SLURM_ARCHIVE=

# 空なら実行時に ${SLURM_BASE_URL}/${SLURM_ARCHIVE} を使う
# mirror や GitHub archive を使いたい場合だけ上書きする
ENV SLURM_URL=

# 1 の場合、SchedMD の SHA256 ファイルを使って検証する
ENV VERIFY_SHA256=1

# PREFIX が空なら実行時に /opt/slurm-${SLURM_VERSION} を使う
ENV PREFIX=
ENV PREFIX_BASE=/opt

# 空なら実行時に ${PREFIX}/etc/slurm, ${PREFIX}/var を使う
ENV SYSCONFDIR=
ENV LOCALSTATEDIR=

ENV OUTPUT_DIR=/mnt
ENV MAKE_JOBS=0
ENV CONFIGURE_FLAGS=""

RUN cat > /usr/local/bin/build-slurm <<'EOF' && chmod +x /usr/local/bin/build-slurm
#!/usr/bin/env bash
set -euo pipefail

if [ "${MAKE_JOBS:-0}" = "0" ]; then
  MAKE_JOBS="$(nproc)"
fi

SLURM_ARCHIVE="${SLURM_ARCHIVE:-slurm-${SLURM_VERSION}.tar.bz2}"
SLURM_URL="${SLURM_URL:-${SLURM_BASE_URL}/${SLURM_ARCHIVE}}"

PREFIX="${PREFIX:-${PREFIX_BASE}/slurm-${SLURM_VERSION}}"
SYSCONFDIR="${SYSCONFDIR:-${PREFIX}/etc/slurm}"
LOCALSTATEDIR="${LOCALSTATEDIR:-${PREFIX}/var}"

mkdir -p "${OUTPUT_DIR}"

WORKDIR="$(mktemp -d /tmp/slurm-build.XXXXXX)"
trap 'rm -rf "${WORKDIR}"' EXIT

cd "${WORKDIR}"

echo "==> Build settings"
echo "    SLURM_VERSION=${SLURM_VERSION}"
echo "    SLURM_BASE_URL=${SLURM_BASE_URL}"
echo "    SLURM_ARCHIVE=${SLURM_ARCHIVE}"
echo "    SLURM_URL=${SLURM_URL}"
echo "    VERIFY_SHA256=${VERIFY_SHA256}"
echo "    PREFIX=${PREFIX}"
echo "    SYSCONFDIR=${SYSCONFDIR}"
echo "    LOCALSTATEDIR=${LOCALSTATEDIR}"
echo "    OUTPUT_DIR=${OUTPUT_DIR}"
echo "    MAKE_JOBS=${MAKE_JOBS}"
echo "    CONFIGURE_FLAGS=${CONFIGURE_FLAGS}"

echo
echo "==> Downloading Slurm source"
curl -fL "${SLURM_URL}" -o "${SLURM_ARCHIVE}"

if [ "${VERIFY_SHA256}" = "1" ] && [[ "${SLURM_URL}" == "${SLURM_BASE_URL}/"* ]]; then
  echo
  echo "==> Verifying SHA256"
  curl -fL "${SLURM_BASE_URL}/SHA256" -o SHA256

  if ! grep -E "[[:space:]]${SLURM_ARCHIVE}$" SHA256 > SHA256.selected; then
    echo "ERROR: ${SLURM_ARCHIVE} not found in ${SLURM_BASE_URL}/SHA256" >&2
    exit 1
  fi

  sha256sum -c SHA256.selected
elif [ "${VERIFY_SHA256}" = "1" ]; then
  echo
  echo "==> Skipping SHA256 verification because SLURM_URL is outside SLURM_BASE_URL"
fi

echo
echo "==> Extracting"
mkdir src
tar -xaf "${SLURM_ARCHIVE}" -C src --strip-components=1

cd src

if [ ! -x ./configure ]; then
  echo
  echo "==> ./configure not found; running autoreconf"
  autoreconf -fi
fi

DESTDIR="${WORKDIR}/install-root"

echo
echo "==> Configuring"
./configure \
  --prefix="${PREFIX}" \
  --sysconfdir="${SYSCONFDIR}" \
  --localstatedir="${LOCALSTATEDIR}" \
  ${CONFIGURE_FLAGS}

echo
echo "==> Building"
make -j "${MAKE_JOBS}"

echo
echo "==> Installing into staged root"
make install DESTDIR="${DESTDIR}"

ARCH="$(uname -m)"
OUT_STEM="slurm-${SLURM_VERSION}-ubuntu22.04-${ARCH}"
OUT_TAR="${OUTPUT_DIR}/${OUT_STEM}.tar.gz"
OUT_DIR="${OUTPUT_DIR}/${OUT_STEM}"

echo
echo "==> Creating full staged tarball"
tar -C "${DESTDIR}" -czf "${OUT_TAR}" .

echo
echo "==> Creating extracted prefix tree"
rm -rf "${OUT_DIR}"
mkdir -p "${OUT_DIR}"

PREFIX_IN_DESTDIR="${DESTDIR}${PREFIX}"

if [ ! -d "${PREFIX_IN_DESTDIR}" ]; then
  echo "ERROR: installed prefix directory not found: ${PREFIX_IN_DESTDIR}" >&2
  exit 1
fi

cp -a "${PREFIX_IN_DESTDIR}/." "${OUT_DIR}/"

cat > "${OUTPUT_DIR}/${OUT_STEM}.env" <<ENVEOF
SLURM_VERSION=${SLURM_VERSION}
SLURM_BASE_URL=${SLURM_BASE_URL}
SLURM_ARCHIVE=${SLURM_ARCHIVE}
SLURM_URL=${SLURM_URL}
VERIFY_SHA256=${VERIFY_SHA256}
PREFIX=${PREFIX}
SYSCONFDIR=${SYSCONFDIR}
LOCALSTATEDIR=${LOCALSTATEDIR}
TARBALL=${OUT_TAR}
EXTRACTED_DIR=${OUT_DIR}
CONFIGURE_FLAGS=${CONFIGURE_FLAGS}
ENVEOF

echo
echo "==> REST/JWT related files"
find "${OUT_DIR}" \
  \( -name 'slurmrestd' \
     -o -name 'auth_jwt.so' \
     -o -name '*openapi*.so' \
     -o -name '*data_parser*.so' \
     -o -name '*serializer*.so' \) \
  -print | sort || true

echo
echo "==> Binaries"
find "${OUT_DIR}/bin" "${OUT_DIR}/sbin" -maxdepth 1 -type f 2>/dev/null | sort || true

echo
echo "==> Done"
echo "    tarball:       ${OUT_TAR}"
echo "    extracted dir: ${OUT_DIR}"
EOF

ENTRYPOINT ["/usr/local/bin/build-slurm"]
