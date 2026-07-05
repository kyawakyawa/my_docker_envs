# Slurm Runtime Builder for Ubuntu 22.04

Ubuntu 22.04 コンテナ上で Slurm をビルドし、成果物をホスト側の `/mnt` マウント先へ出力するための Dockerfile です。

この版では、Slurm の source tarball 取得元として SchedMD 公式の download directory を使います。

```text id="odblmn"
https://download.schedmd.com/slurm/
```

そのため、通常は `SLURM_VERSION` だけを指定すれば、対応する tarball を自動的に取得できます。

例:

```text id="d81qho"
SLURM_VERSION=22.05.11
-> https://download.schedmd.com/slurm/slurm-22.05.11.tar.bz2
```

SchedMD の download directory には、`slurm-22.05.11.tar.bz2` や `slurm-22.05-latest.tar.bz2` のような tarball が置かれています。

## 特徴

この Dockerfile の特徴は以下です。

* ベースイメージは Ubuntu 22.04
* Docker image build 時には Slurm のビルド依存のみをインストール
* Slurm 本体のダウンロードとビルドは `docker run` 時に実行
* `SLURM_VERSION` だけで SchedMD 公式 tarball を取得可能
* `slurmrestd` / JWT 関連 plugin のビルドに必要な依存を含む
* デフォルトでは Slurm 一式を `/opt/slurm-${SLURM_VERSION}` 以下にまとめる
* `sysconfdir` と `localstatedir` もデフォルトで prefix 以下に配置
* ビルド成果物は tarball と展開済み install tree の両方で出力
* SchedMD の `SHA256` ファイルを使った検証に対応

## 基本方針

デフォルトでは、以下のような構成で configure します。

```text id="wtesf7"
prefix         = /opt/slurm-${SLURM_VERSION}
sysconfdir     = ${prefix}/etc/slurm
localstatedir  = ${prefix}/var
```

例えば `SLURM_VERSION=22.05.11` の場合は以下になります。

```text id="v91ero"
prefix         = /opt/slurm-22.05.11
sysconfdir     = /opt/slurm-22.05.11/etc/slurm
localstatedir  = /opt/slurm-22.05.11/var
```

これにより、Slurm 関連ファイルを `/opt/slurm-22.05.11` 以下にまとめて扱えます。

## Docker image のビルド

Dockerfile があるディレクトリで以下を実行します。

```bash id="wgl5qc"
docker build -t slurm-builder:ubuntu22.04 .
```

## デフォルト設定で Slurm 22.05.11 をビルド

```bash id="3kupux"
mkdir -p out

docker run --rm \
  -v "$PWD/out:/mnt" \
  slurm-builder:ubuntu22.04
```

デフォルトでは以下の tarball を取得します。

```text id="8i93tg"
https://download.schedmd.com/slurm/slurm-22.05.11.tar.bz2
```

## バージョンを指定してビルド

`SLURM_VERSION` を変えるだけで、別バージョンをビルドできます。

```bash id="a5mak7"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=23.02.8 \
  slurm-builder:ubuntu22.04
```

この場合、以下の URL から取得します。

```text id="u895ys"
https://download.schedmd.com/slurm/slurm-23.02.8.tar.bz2
```

## latest tarball を使う

SchedMD の download directory には、系列ごとの `latest` tarball もあります。

例えば Slurm 22.05 系の latest を使う場合は以下です。

```bash id="dzvvqp"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05-latest \
  slurm-builder:ubuntu22.04
```

この場合、以下の URL から取得します。

```text id="3mnynv"
https://download.schedmd.com/slurm/slurm-22.05-latest.tar.bz2
```

ただし、再現性を重視する場合は `22.05-latest` ではなく、`22.05.11` のように明示的なバージョンを指定する方が安全です。

## tarball 名を明示する

通常は `SLURM_VERSION` だけ指定すれば十分です。

ただし、特殊な tarball 名を使いたい場合は `SLURM_ARCHIVE` を指定できます。

```bash id="7raur7"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e SLURM_ARCHIVE=slurm-22.05.11.tar.bz2 \
  slurm-builder:ubuntu22.04
```

`SLURM_ARCHIVE` が空の場合は、実行時に以下の形式で自動生成されます。

```text id="6bmhlu"
slurm-${SLURM_VERSION}.tar.bz2
```

## URL を直接指定する

SchedMD 公式 URL 以外から取得したい場合は、`SLURM_URL` を指定します。

```bash id="w4a6b1"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e SLURM_URL=https://example.local/mirror/slurm-22.05.11.tar.bz2 \
  slurm-builder:ubuntu22.04
```

この場合、`SLURM_URL` が優先されます。

## SHA256 検証

デフォルトでは、`VERIFY_SHA256=1` です。

SchedMD 公式 download directory から取得する場合、以下のファイルを使って tarball の SHA256 を検証します。

```text id="49xd0d"
https://download.schedmd.com/slurm/SHA256
```

検証を無効化したい場合は、以下のようにします。

```bash id="erntk9"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e VERIFY_SHA256=0 \
  slurm-builder:ubuntu22.04
```

`SLURM_URL` に SchedMD 公式 download directory 以外の URL を指定した場合、`VERIFY_SHA256=1` でも SHA256 検証は自動的にスキップされます。

## 出力される成果物

デフォルトでは、ホスト側の `./out` に以下のような成果物が作成されます。

```text id="2k0cnb"
out/
├── slurm-22.05.11-ubuntu22.04-x86_64.tar.gz
├── slurm-22.05.11-ubuntu22.04-x86_64/
│   ├── bin/
│   ├── sbin/
│   ├── lib/
│   ├── include/
│   ├── share/
│   ├── etc/
│   │   └── slurm/
│   └── var/
└── slurm-22.05.11-ubuntu22.04-x86_64.env
```

### tarball

```text id="yeo9yc"
slurm-22.05.11-ubuntu22.04-x86_64.tar.gz
```

`make install DESTDIR=...` した内容をまとめた tarball です。

ホスト側に展開して利用できます。

### 展開済み install tree

```text id="qczhrp"
slurm-22.05.11-ubuntu22.04-x86_64/
```

prefix 以下だけを取り出したディレクトリです。

この中に `bin/`, `sbin/`, `lib/`, `etc/slurm/`, `var/` などが入ります。

### metadata

```text id="v28913"
slurm-22.05.11-ubuntu22.04-x86_64.env
```

ビルド時の設定を記録したファイルです。

例:

```text id="zbr95s"
SLURM_VERSION=22.05.11
SLURM_BASE_URL=https://download.schedmd.com/slurm
SLURM_ARCHIVE=slurm-22.05.11.tar.bz2
SLURM_URL=https://download.schedmd.com/slurm/slurm-22.05.11.tar.bz2
VERIFY_SHA256=1
PREFIX=/opt/slurm-22.05.11
SYSCONFDIR=/opt/slurm-22.05.11/etc/slurm
LOCALSTATEDIR=/opt/slurm-22.05.11/var
CONFIGURE_FLAGS=
```

## prefix を変更する

デフォルトでは `/opt/slurm-${SLURM_VERSION}` にインストールされる前提でビルドされます。

prefix を変更したい場合は `PREFIX` を指定します。

```bash id="kr148z"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e PREFIX=/usr/local/slurm \
  slurm-builder:ubuntu22.04
```

この場合、configure には以下のように渡されます。

```bash id="u385gh"
./configure \
  --prefix=/usr/local/slurm \
  --sysconfdir=/usr/local/slurm/etc/slurm \
  --localstatedir=/usr/local/slurm/var
```

## sysconfdir / localstatedir を変更する

デフォルトでは、設定ファイルと状態ディレクトリは prefix 以下に置かれます。

```text id="8i3cv9"
SYSCONFDIR=${PREFIX}/etc/slurm
LOCALSTATEDIR=${PREFIX}/var
```

既存の Slurm 構成に合わせて `/etc/slurm` や `/var` を使いたい場合は、以下のように指定します。

```bash id="qobswh"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e SYSCONFDIR=/etc/slurm \
  -e LOCALSTATEDIR=/var \
  slurm-builder:ubuntu22.04
```

この場合、configure には以下のように渡されます。

```bash id="ihpyd6"
./configure \
  --prefix=/opt/slurm-22.05.11 \
  --sysconfdir=/etc/slurm \
  --localstatedir=/var
```

## configure option を追加する

`CONFIGURE_FLAGS` で任意の configure option を追加できます。

```bash id="z8k5im"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e CONFIGURE_FLAGS="--disable-debug" \
  slurm-builder:ubuntu22.04
```

複数指定も可能です。

```bash id="3f7lfk"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e CONFIGURE_FLAGS="--disable-debug --without-rpath" \
  slurm-builder:ubuntu22.04
```

## make の並列数を指定する

デフォルトでは、コンテナ内の CPU 数に応じて `make -j $(nproc)` が実行されます。

並列数を明示したい場合は `MAKE_JOBS` を指定します。

```bash id="7dv1t7"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e MAKE_JOBS=8 \
  slurm-builder:ubuntu22.04
```

## slurmrestd / JWT 対応

この Dockerfile では、`slurmrestd` と JWT 認証 plugin のビルドに必要な依存を含めています。

関連する主なパッケージは以下です。

```text id="4w0lcl"
libjwt-dev
libjson-c-dev
libhttp-parser-dev
libyaml-dev
```

ビルド後、以下のようなファイルが出力されていれば、`slurmrestd` や JWT 関連 plugin がビルドされています。

```text id="6ochg7"
sbin/slurmrestd
lib/slurm/auth_jwt.so
lib/slurm/openapi_*.so
lib/slurm/data_parser_*.so
lib/slurm/serializer_*.so
```

確認例:

```bash id="l7zixt"
find out/slurm-22.05.11-ubuntu22.04-x86_64 \
  \( -name 'slurmrestd' \
     -o -name 'auth_jwt.so' \
     -o -name '*openapi*.so' \
     -o -name '*data_parser*.so' \
     -o -name '*serializer*.so' \) \
  -print | sort
```

JWT 認証を使うには、plugin が存在するだけでなく、実運用時に `slurm.conf` や `slurmdbd.conf` 側の設定が必要です。

例:

```conf id="pp5eia"
AuthType=auth/munge
AuthAltTypes=auth/jwt
AuthAltParameters=jwt_key=/opt/slurm-22.05.11/etc/slurm/jwt_hs256.key
```

JWT key の作成例です。

```bash id="m4oi9w"
install -d -m 0755 /opt/slurm-22.05.11/etc/slurm

dd if=/dev/urandom \
  of=/opt/slurm-22.05.11/etc/slurm/jwt_hs256.key \
  bs=32 count=1

chown slurm:slurm /opt/slurm-22.05.11/etc/slurm/jwt_hs256.key
chmod 0600 /opt/slurm-22.05.11/etc/slurm/jwt_hs256.key
```

`slurmrestd` の起動例です。

```bash id="y4s351"
export SLURM_CONF=/opt/slurm-22.05.11/etc/slurm/slurm.conf
export LD_LIBRARY_PATH=/opt/slurm-22.05.11/lib:${LD_LIBRARY_PATH:-}
export SLURM_JWT=invalid

/opt/slurm-22.05.11/sbin/slurmrestd 0.0.0.0:6820
```

JWT token の発行例です。

```bash id="vve3ke"
export SLURM_CONF=/opt/slurm-22.05.11/etc/slurm/slurm.conf
export LD_LIBRARY_PATH=/opt/slurm-22.05.11/lib:${LD_LIBRARY_PATH:-}

scontrol token username="$USER" lifespan=3600
```

API 呼び出し例です。

```bash id="4kx0wh"
TOKEN="$(scontrol token username="$USER" lifespan=3600 | sed 's/^SLURM_JWT=//')"

curl -s \
  -H "X-SLURM-USER-NAME: $USER" \
  -H "X-SLURM-USER-TOKEN: $TOKEN" \
  http://slurm-controller.example:6820/slurm/v0.0.38/ping
```

## ホストへ配置する例

デフォルト prefix の `/opt/slurm-22.05.11` に配置する例です。

tarball から展開する場合:

```bash id="1c8cpo"
sudo tar -C / -xzf out/slurm-22.05.11-ubuntu22.04-x86_64.tar.gz
```

展開済み install tree からコピーする場合:

```bash id="b3gnai"
sudo mkdir -p /opt/slurm-22.05.11

sudo rsync -a \
  out/slurm-22.05.11-ubuntu22.04-x86_64/ \
  /opt/slurm-22.05.11/
```

実行時には、必要に応じて以下を設定します。

```bash id="q48eci"
export PATH=/opt/slurm-22.05.11/bin:/opt/slurm-22.05.11/sbin:$PATH
export LD_LIBRARY_PATH=/opt/slurm-22.05.11/lib:${LD_LIBRARY_PATH:-}
export SLURM_CONF=/opt/slurm-22.05.11/etc/slurm/slurm.conf
```

## よく使うコマンド

### image をビルド

```bash id="g79izc"
docker build -t slurm-builder:ubuntu22.04 .
```

### Slurm 22.05.11 をビルド

```bash id="k7ifnf"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  slurm-builder:ubuntu22.04
```

### Slurm 22.05 系 latest をビルド

```bash id="42sdek"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05-latest \
  slurm-builder:ubuntu22.04
```

### Slurm 23.02.8 をビルド

```bash id="hgeilb"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=23.02.8 \
  slurm-builder:ubuntu22.04
```

### SHA256 検証を無効化

```bash id="x4770s"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e VERIFY_SHA256=0 \
  slurm-builder:ubuntu22.04
```

### prefix を指定

```bash id="xyyzz2"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e PREFIX=/opt/slurm-22.05.11 \
  slurm-builder:ubuntu22.04
```

### `/etc/slurm` と `/var` を使う

```bash id="6es2gw"
docker run --rm \
  -v "$PWD/out:/mnt" \
  -e SLURM_VERSION=22.05.11 \
  -e SYSCONFDIR=/etc/slurm \
  -e LOCALSTATEDIR=/var \
  slurm-builder:ubuntu22.04
```

## 注意点

### ビルド時の prefix と実配置先は合わせる

Slurm は plugin directory や library path の都合があるため、ビルド時の `--prefix` と実際の配置先はできるだけ一致させるのが安全です。

デフォルトでは `/opt/slurm-${SLURM_VERSION}` prefix でビルドされるため、ホスト側でも同じ場所に配置することを推奨します。

### latest tarball は再現性に注意

`SLURM_VERSION=22.05-latest` のような指定は便利ですが、後から中身が変わる可能性があります。

再現性を重視する場合は、以下のように明示的なバージョンを指定してください。

```text id="0m8qmr"
SLURM_VERSION=22.05.11
```

### 実行時にネットワークが必要

この Dockerfile は `docker run` 時に Slurm source tarball をダウンロードします。

そのため、実行時に `https://download.schedmd.com/slurm/` へアクセスできる必要があります。

オフライン環境で使いたい場合は、あらかじめ tarball をダウンロードしておき、`SLURM_URL` に file URL や内部 mirror URL を指定する構成に変更してください。

## まとめ

この Dockerfile は、SchedMD 公式 download directory から Slurm source tarball を取得し、Ubuntu 22.04 上で Slurm をビルドするための runtime builder です。

通常は `SLURM_VERSION` だけを変えれば別バージョンをビルドできます。

デフォルトでは `/opt/slurm-${SLURM_VERSION}` 以下に Slurm 関連ファイルをまとめるため、deb パッケージ化が難しいバージョンでも、tarball / install tree として扱いやすい成果物を作れます。

