# Maintainer: Ivan Puntiy <ivan.puntiy-at-gmail>
# Contributor: Schala

pkgname=mingw-w64-wxmsw3.1
epoch=1
pkgver=3.1.1
pkgrel=1
pkgdesc="Win32 implementation of wxWidgets API for GUI (development branch, mingw-w64)"
arch=(any)
url="http://wxwidgets.org"
license=("custom:wxWindows")
makedepends=(mingw-w64-cmake)
depends=(mingw-w64-crt mingw-w64-expat mingw-w64-libpng mingw-w64-libjpeg-turbo mingw-w64-libtiff)
options=(staticlibs !strip !buildflags)
source=("https://github.com/wxWidgets/wxWidgets/releases/download/v${pkgver}/wxWidgets-${pkgver}.tar.bz2"
        "fix-linker.patch")
sha256sums=('c925dfe17e8f8b09eb7ea9bfdcfcc13696a3e14e92750effd839f5e10726159e'
            'fdd5e555e21cffe8d341a50086399f18d640b42e8e126a56fb98c966ffd4c05e')

_architectures="i686-w64-mingw32 x86_64-w64-mingw32"

prepare() {
  cd "${srcdir}/wxWidgets-${pkgver}"
  patch -Np1 -i "${srcdir}/fix-linker.patch"
}

build() {
  local _build_flags="\
        -DwxBUILD_PRECOMP=OFF \
        -DwxUSE_OPENGL=ON \
        -DwxUSE_UNICODE=ON \
        -DwxUSE_GRAPHICS_CONTEXT=ON \
        -DwxUSE_WEBVIEW=ON \
        -DwxUSE_MEDIACTRL=ON \
        -DwxUSE_XPM=ON \
        -DwxUSE_REGEX=builtin \
        -DwxUSE_LIBPNG=sys \
        -DwxUSE_EXPAT=sys \
        -DwxUSE_LIBJPEG=sys \
        -DwxUSE_LIBTIFF=sys \
        -DwxUSE_ZLIB=sys"

  # Fix for current libuuid.a issues
  # see: https://github.com/Alexpux/MINGW-packages/issues/1761
  # looks like this was fixed, uncomment if needed
  # _build_flags="${_build_flags} LDFLAGS=-Wl,--allow-multiple-definition"

  cd "${srcdir}/wxWidgets-${pkgver}"
  for _arch in ${_architectures}; do
    # shared build
    mkdir -p build-shared-${_arch} && pushd build-shared-${_arch}
    # can't build monolithic - some dependencies are not applied?
    # ${_arch}-cmake ${_build_flags} -DwxBUILD_MONOLITHIC=ON ..
    # but compilation still errs here:
    ${_arch}-cmake ${_build_flags} ..
    make #VERBOSE=1
    popd

    # static build
    mkdir -p build-static-${_arch} && pushd build-static-${_arch}
    ${_arch}-cmake ${_build_flags} -DwxBUILD_SHARED=OFF ..
    make
    popd
  done
}

package() {
  mkdir -p "${pkgdir}/usr/bin"
  for _arch in ${_architectures}; do
    for _build in "shared" "static"; do
      cd "${srcdir}/wxWidgets-${pkgver}/build-${_build}-${_arch}"
      make DESTDIR="${pkgdir}" install
    done

    mv "${pkgdir}/usr/${_arch}/lib/"*.dll "${pkgdir}/usr/${_arch}/bin"
    find "${pkgdir}/usr/${_arch}" -name '*.exe' | xargs -rtl1 rm
    find "${pkgdir}/usr/${_arch}" -name '*.dll' | xargs -rtl1 ${_arch}-strip --strip-unneeded
    find "${pkgdir}/usr/${_arch}" -name '*.a' -o -name '*.dll' | xargs -rtl1 ${_arch}-strip -g

    ln -s "/usr/${_arch}/lib/wx/config/${_arch}-msw-unicode-${pkgver%.*}" "${pkgdir}/usr/bin/${_arch}-wx-config-3.1"

    # conflicts with stable package
    mv "${pkgdir}/usr/${_arch}/bin/wx-config"{,-3.1}
    rm -r "${pkgdir}/usr/${_arch}/share"

    # rm "${pkgdir}/usr/${_arch}/bin/wxrc-3.1"
  done
}
