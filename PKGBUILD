# Maintainer: GalaxySnail <me+aur@glxys.nl>
# Contributor: Tobias Kunze <r@rixx.de>
# Maintained at https://github.com/rixx/pkgbuilds, feel free to submit patches

shopt -s extglob

pkgname=python312
pkgver=3.12.11
pkgrel=2
_pybasever=${pkgver%.*}
_pymajver=${_pybasever%.*}
pkgdesc="The Python programming language (3.12)"
arch=('x86_64')
license=('PSF-2.0')
url="https://www.python.org/"
depends=(
  'bzip2'
  'expat'
  'gdbm'
  'libffi'
  'libnsl'
  'libxcrypt'
  'openssl'
  'zlib'
  'tzdata'
  'mpdecimal'
)
makedepends=(
  'bluez-libs'
  'cosign'
  'gdb'
  'llvm'
  'llvm-bolt'
  'mpdecimal'
  'sqlite'
  'tk'
)
source=(
  "https://www.python.org/ftp/python/${pkgver%rc*}/Python-${pkgver}.tar.xz"{,.sigstore}
  EXTERNALLY-MANAGED)
md5sums=('9613d56b90d0d0cfd19980c7e2956a06'
         '85ab669635ea9257c139232b6db7c4c0'
         '7d2680a8ab9c9fa233deb71378d5a654')

verify() {
  cosign verify-blob \
    --new-bundle-format \
    --certificate-oidc-issuer 'https://accounts.google.com' \
    --certificate-identity 'thomas@python.org' \
    --bundle ./Python-${pkgver}.tar.xz.sigstore \
    ./Python-${pkgver}.tar.xz
}

prepare() {
  cd "${srcdir}/Python-${pkgver}" || exit

  # FS#23997
  sed -i -e "s|^#.* /usr/local/bin/python|#!/usr/bin/python|" Lib/cgi.py

  # Ensure that we are using the system copy of various libraries (expat, zlib and libffi),
  # rather than copies shipped in the tarball
  rm -rf Modules/expat
  rm -rf Modules/_decimal/libmpdec
}

build() {
  cd "${srcdir}/Python-${pkgver}" || exit

  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3} -ffat-lto-objects"

  export CFLAGS+=" -fno-semantic-interposition -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer"
  export CXXLAGS+=" -fno-semantic-interposition -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer"

  # Disable bundled pip & setuptools
  # BOLT is disabled due LLVM or upstream issue
  # https://github.com/python/cpython/issues/124948
  ./configure \
    --prefix=/usr \
    --enable-ipv6 \
    --enable-loadable-sqlite-extensions \
    --enable-optimizations \
    --enable-shared \
    --with-computed-gotos \
    --with-dbmliborder=gdbm:ndbm \
    --with-lto \
    --with-system-expat \
    --with-system-libmpdec \
    --with-tzpath=/usr/share/zoneinfo \
    --without-ensurepip

  make EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd "${srcdir}/Python-${pkgver}" || exit 1
  # altinstall: /usr/bin/pythonX.Y but not /usr/bin/python or /usr/bin/pythonX
  make DESTDIR="${pkgdir}" altinstall maninstall

  # Split tests
  rm -rf "$pkgdir"/usr/lib/python*/{test,ctypes/test,distutils/tests,idlelib/idle_test,lib2to3/tests,tkinter/test,unittest/test}

  # Avoid conflicts with the main 'python' package.
  rm -f "${pkgdir}/usr/lib/libpython${_pymajver}.so"
  rm -f "${pkgdir}/usr/share/man/man1/python${_pymajver}.1"

  # Clean-up reference to build directory
  sed -i "s|$srcdir/Python-${pkgver}:||" "$pkgdir/usr/lib/python${_pybasever}/config-${_pybasever}-${CARCH}-linux-gnu/Makefile"

  # Add useful scripts FS#46146
  install -dm755 "${pkgdir}"/usr/lib/python"${_pybasever}"/Tools/{i18n,scripts}
  install -m755 Tools/i18n/{msgfmt,pygettext}.py "${pkgdir}"/usr/lib/python"${_pybasever}"/Tools/i18n/
  install -m755 Tools/scripts/{README,*py} "${pkgdir}"/usr/lib/python"${_pybasever}"/Tools/scripts/

  # PEP668
  install -Dm644 "${srcdir}/EXTERNALLY-MANAGED" -t "${pkgdir}/usr/lib/python${_pybasever}/"


  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
