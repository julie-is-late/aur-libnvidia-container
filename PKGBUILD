# Maintainer: Jakub Klinkovský <lahwaacz at archlinux dot org>
# Contributor: Mark Wagie <mark dot wagie at proton dot me>
# Contributor: Julie Shapiro <jshapiro at nvidia dot com>
# Contributor: Kien Dang <mail at kien dot ai>

pkgname=libnvidia-container
pkgver=1.14.6
pkgrel=2
_nvmodver=550.54.14  # check the VERSION in libnvidia-container/mk/nvidia-modprobe.mk
pkgdesc="NVIDIA container runtime library"
arch=(x86_64)
url="https://github.com/NVIDIA/libnvidia-container"
license=(Apache-2.0 GPL-3.0-or-later LGPL-3.0-or-later GPL-2.0-only)
depends=(
  glibc
  libcap
  libelf
  libseccomp
  libtirpc
)
makedepends=(
  go
  rpcsvc-proto
)
provides=(libnvidia-container-tools=$pkgver)
replaces=(libnvidia-container-tools)
# we cannot use LTO as otherwise we do not get reproducible package with full RELRO (libnvidia-container-go.so)
options=('!lto')
source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/v$pkgver.tar.gz"
        "$pkgname-nvidia-modprobe-$_nvmodver.tar.gz::https://github.com/NVIDIA/nvidia-modprobe/archive/$_nvmodver.tar.gz"
        fix-makefile.patch)
b2sums=('d3c526d7b04ac9cbc6b6bb63f25d4c5b17571169a6cb1a6ab9f7c1cc322a27e3a853373551682b535146914fd2eca809d02391acb458a874a7e9e5c0fc8bf459'
        '7b334877d98d0c75d5750192dea868436938852443ced14e74e59076ed4d8be9e361cdefbe48295d87bb91ac4565152ec3f3233479b3da19bb8baf8e7ef53cd6'
        '4938bdd72116a8f9f77e5a13a209e51332611ceb84d7e5e4155658023b6cd17c1a5e96a00c6809417092568c3558f9466c0a39c623cf31ba66351aafd724b5e5')

_common_make_flags=(
  prefix=/usr
  # the docdir contains only licenses
  docdir=/usr/share/licenses
  # override version variables that take defaults from git
  GIT_TAG=$pkgver
  REVISION=$pkgver
)

prepare() {
  cd $pkgname-$pkgver

  # nvidia-modprobe patching based on libnvidia-container/mk/nvidia-modprobe.mk
  mkdir -p deps/src/nvidia-modprobe-$_nvmodver
  cp -r ../nvidia-modprobe-$_nvmodver/modprobe-utils/ deps/src/nvidia-modprobe-$_nvmodver/
  touch "deps/src/nvidia-modprobe-${_nvmodver}/.download_stamp"
  patch -d "deps/src/nvidia-modprobe-${_nvmodver}" -p1 < mk/nvidia-modprobe.patch

  # the Makefile needs patching due to several deficiencies:
  # - avoid manual debuginfo files
  # - avoid downloading external libraries (libelf and libtirpc; the WITH_TIRPC and WITH_LIBELF flags do not work)
  # - allow to set documentation install path per Arch packaging guidelines
  # - respect system $CGO_CFLAGS and $CGO_LDFLAGS for nvcgo, pass custom -ldflags to go build
  # - avoid rebuilding nvcgo in the install step
  patch -Np1 -i ../fix-makefile.patch
}

build() {
  cd $pkgname-$pkgver
  export GOPATH="$srcdir"
  export GOFLAGS="-mod=vendor"
  make "${_common_make_flags[@]}" CGO_CFLAGS="$CFLAGS" CGO_LDFLAGS="$LDFLAGS" GO_LDFLAGS="-compressdwarf=false -linkmode=external"
}

package() {
  cd $pkgname-$pkgver
  make "${_common_make_flags[@]}" DESTDIR="$pkgdir" install
}
