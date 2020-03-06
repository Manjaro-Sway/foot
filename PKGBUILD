# Maintainer: Daniel Eklöf <daniel at ekloef dot se>
pkgname=('foot' 'foot-terminfo')
pkgver=1.2.1
pkgrel=1
arch=('x86_64')
url=https://codeberg.org/dnkl/foot
license=(mit)
makedepends=('meson' 'ninja' 'scdoc' 'python' 'ncurses' 'wayland-protocols')
depends=('libxkbcommon' 'wayland' 'pixman')
source=(https://codeberg.org/dnkl/foot/archive/1.2.1.tar.gz
        https://codeberg.org/dnkl/tllist/archive/1.0.0.tar.gz
        https://codeberg.org/dnkl/fcft/archive/1.1.3.tar.gz)
sha256sums=('1ad30a6bfdd1bcefaca5be389f80e39929a2ec3d26a467d32557c84f474a538b'
            '115f067f6b7029959fea39e45ec64697806e901fe4ca2de67c7eb2af4b7d2ae2'
            '94be711e256b54fed3b57230389466de00c9597dcbb10db1d6fc10bb851146c2')

build() {
  cd foot

  mkdir -p subprojects
  pushd subprojects
  ln -sf ../../tllist .
  ln -sf ../../fcft .
  popd

  meson --prefix=/usr --buildtype=release --wrap-mode=forcefallback -Db_lto=true -Dc_args="-Wno-missing-profile" . build

  if [[ -v WAYLAND_DISPLAY ]]; then
    meson configure -Db_pgo=generate build
    find -name "*.gcda" -delete
    ninja -C build

    tmp_file=$(mktemp)
    build/foot --config /dev/null --term=xterm -- sh -c "./scripts/generate-alt-random-writes.py --scroll --scroll-region --colors-regular --colors-bright --colors-rgb ${tmp_file} && cat ${tmp_file}"
    rm "${tmp_file}"

    meson configure -Db_pgo=use build
  fi

  ninja -C build
}

check() {
  cd foot
  ninja -C build test
}

package_foot() {
  pkgdesc="A wayland native terminal emulator"
  optdepends=('foot-terminfo: terminfo for foot')

  cd foot
  DESTDIR="${pkgdir}/" ninja -C build install
  rm -rf "${pkgdir}/usr/share/terminfo"
}

package_foot-terminfo() {
  pkgdesc="Terminfo files for the foot terminal emulator"
  depends=('ncurses')

  install -dm 755 "${pkgdir}/usr/share/terminfo/f/"
  cp build/f/* "${pkgdir}/usr/share/terminfo/f/"
}
