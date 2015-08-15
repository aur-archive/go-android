# $Id$
# Maintainer: Vesa Kaihlavirta <vegai@iki.fi>
# Maintainer: Alexander Rødseth <rodseth@gmail.com>
# Contributor: Rémy Oudompheng  <remy@archlinux.org>
# Contributor: Andres Perera <andres87p gmail>
# Contributor: Matthew Bauer <mjbauer95@gmail.com>
# Contributor: Christian Himpel <chressie@gmail.com>
# Contributor: Mike Rosset <mike.rosset@gmail.com>
# Contributor: Daniel YC Lin <dlin.tw@gmail.com>
# Contributor: John Luebs <jkluebs@gmail.com>
# Contributor: Stanislav Seletskiy <s.seletskiy@gmail.com>

pkgname=go-android
_pkgname=go
epoch=2
pkgver=1.4
pkgrel=1
pkgdesc='Compiler and tools for the Go programming language from Google (Android support)'
arch=('x86_64' 'i686')
url='http://golang.org/'
license=('BSD')
depends=('perl' 'gawk' 'android-ndk')
makedepends=('inetutils' 'mercurial' 'git')
options=('!strip' 'staticlibs')
optdepends=('mercurial: for fetching sources from mercurial repositories'
            'git: for fetching sources from git repositories'
            'bzr: for fetching sources from bazaar repositories'
            'subversion: for fetching sources from subversion repositories')
install="$pkgname.install"
source=("https://storage.googleapis.com/golang/go1.4.src.tar.gz")
sha1sums=('6a7d9bd90550ae1e164d7803b3e945dc8309252b')

prepare() {
  if [ ! $CC_FOR_TARGET ]; then
    echo 'makepkg for this package should be run with $CC_FOR_TARGET:'
    echo '  CC_FOR_TARGET=<path-to-ndk-cc> makepkg'
    echo
    echo '* <path-to-ndk-cc> should point to the toolchain gcc for the specific'
    echo '  Android version. Toolchain can be bootstrapped like this:'
    echo '    $NDK/build/tools/make-standalone-toolchain.sh \'
    echo '      --platform=android-15 \
    echo '      --arch=arm \'
    echo '      --install-dir=<path-to-toolchain>'
    exit 1
  fi
}

build() {
  cd "$_pkgname/src"

  export GOROOT="$srcdir/$_pkgname"
  export GOBIN="$GOROOT/bin"
  export GOPATH="$srcdir/"
  export GOROOT_FINAL=/usr/lib/go-android

  export GOOS=android
  export GOARCH=arm

  export GOARM=7

  bash make.bash

  export GOOS=linux
  export GOARCH=amd64

  #CC_FOR_TARGET="" bash make.bash --no-clean

  $GOROOT/bin/go get -d golang.org/x/tools/cmd/godoc
  $GOROOT/bin/go build -o $srcdir/godoc golang.org/x/tools/cmd/godoc
  for tool in vet cover; do
    $GOROOT/bin/go get -d golang.org/x/tools/cmd/${tool}
    $GOROOT/bin/go build -o $GOROOT/pkg/tool/${GOOS}_${GOARCH}/${tool} golang.org/x/tools/cmd/${tool}
  done

  $GOROOT/bin/go get -d golang.org/x/mobile || :
  $GOROOT/bin/go build -o $GOROOT/pkg/tool/${GOOS}_${GOARCH}/gobind golang.org/x/mobile/cmd/gobind
}

package() {
  cd "$_pkgname"

  export GOROOT="$srcdir/$_pkgname"
  export GOBIN="$GOROOT/bin"

  mkdir -p "$pkgdir/usr/lib/go-android/bin"

  install -Dm755 "$srcdir/godoc" "$pkgdir/usr/lib/go-android/bin/godoc"

  install -Dm644 LICENSE \
    "$pkgdir/usr/share/licenses/go-android/LICENSE"

  mkdir -p \
    "$pkgdir/"{etc/profile.d,usr/{share/go-android,lib/go-android,lib/go-android/src,lib/go-android/site/src}}

  cp -r doc misc -t "$pkgdir/usr/share/go-android"
  mv "$srcdir/src/golang.org" "$pkgdir/usr/share/go-android/misc/golang.org"
  cp -a "$srcdir/src" "$pkgdir/usr/lib/go-android/"
  ln -s /usr/share/go/doc "$pkgdir/usr/lib/go-android/doc"
  cp -a bin "$pkgdir/usr/lib/go-android"
  cp -a pkg "$pkgdir/usr/lib/go-android"
  cp -a "$GOROOT/src" "$pkgdir/usr/lib/go-android/"
  cp -a "$GOROOT/src/cmd" "$pkgdir/usr/lib/go-android/src/cmd"
  cp -a "$GOROOT/src/lib9" "$pkgdir/usr/lib/go-android/src/"
  cp -a "$GOROOT/lib" "$pkgdir/usr/lib/go-android/"
  cp -a "$GOROOT/include" "$pkgdir/usr/lib/go-android/"

  install -Dm644 src/Make.* "$pkgdir/usr/lib/go-android/src"

  # Remove object files from target src dir
  find "$pkgdir/usr/lib/go-android/src/" -type f -name '*.[ao]' -delete

  # Fix for FS#32813
  find "$pkgdir" -type f -name sql.go -exec chmod -x {} \;

  # Headers for C modules
  install -Dm644 src/runtime/runtime.h \
    "$pkgdir/usr/lib/go-android/src/runtime/runtime.h"
  install -Dm644 src/runtime/cgocall.h \
    "$pkgdir/usr/lib/go-android/src/runtime/cgocall.h"

  # For FS#42660 / FS#42661 / gox
  install -Dm755 src/make.bash "$pkgdir/usr/lib/go-android/src/make.bash"
  install -Dm755 src/run.bash "$pkgdir/usr/lib/go-android/src/run.bash"
  cp -r misc/ "$pkgdir/usr/lib/go-android/"

  # For godoc
  install -Dm644 favicon.ico "$pkgdir/usr/lib/go-android/favicon.ico"

  rm -f "$pkgdir/usr/share/go-android/doc/articles/wiki/get.bin"

  install -Dm644 VERSION "$pkgdir/usr/lib/go-android/VERSION"

  find "$pkgdir/usr/lib/go-android/"{pkg,bin} -type f -exec touch '{}' +
}

# vim:set ts=2 sw=2 et:
