# Maintainer: Daniel Eklöf <daniel at ekloef dot se>
pkgname=('foot' 'foot-terminfo')
pkgver=1.6.2  # Don’t forget to update CHANGELOG.md
pkgrel=1
arch=('x86_64' 'aarch64')
url=https://codeberg.org/dnkl/foot
license=(mit)
depends=('libxkbcommon' 'wayland' 'pixman' 'fontconfig' 'fcft')
makedepends=('meson' 'ninja' 'scdoc' 'python' 'ncurses' 'wayland-protocols')
checkdepends=('check')
source=(https://codeberg.org/dnkl/foot/archive/${pkgver}.tar.gz
        https://codeberg.org/dnkl/tllist/archive/1.0.4.tar.gz)
sha256sums=('dee10d8153a3ed57e21f5d8c0ed4121ddf83eeb33655728e5b8c46dfa3566a45'
            'a135934d4955902d67f75f3c542ace3bfb7be3be9c44796852e76ea9e1d82b33')

build() {
  cd foot

  mkdir -p subprojects
  pushd subprojects
  ln -sf ../../tllist .
  popd

  # makepkg uses -O2 by default, but we *really* want -O3
  # -Wno-missing-profile since we're not exercising everything when doing PGO builds
  export CFLAGS+=" -O3 -Wno-missing-profile"

  meson --prefix=/usr --buildtype=release -Db_lto=true . build

  find -name "*.gcda" -delete
  meson configure -Db_pgo=generate build
  ninja -C build

  local script_options="--scroll --scroll-region --colors-regular --colors-bright --colors-256 --colors-rgb --attr-bold --attr-italic --attr-underline"

  local tmp_file=$(mktemp)

  if [[ -v WAYLAND_DISPLAY ]]; then
    build/foot \
      --config /dev/null \
      --term=xterm \
      sh -c "./scripts/generate-alt-random-writes.py ${script_options} ${tmp_file} && cat ${tmp_file}"
  else
    ./scripts/generate-alt-random-writes.py \
      --rows=67 \
      --cols=135 \
      ${script_options} \
      ${tmp_file}
    build/pgo ${tmp_file} ${tmp_file} ${tmp_file}
  fi

  rm "${tmp_file}"

  meson configure -Db_pgo=use build
  ninja -C build
}

check() {
  cd foot
  ninja -C build test
}

package_foot() {
  pkgdesc="Wayland terminal emulator - fast, lightweight and minimalistic"
  changelog=CHANGELOG.md
  depends+=('foot-terminfo')

  cd foot
  DESTDIR="${pkgdir}/" ninja -C build install
  rm -rf "${pkgdir}/usr/share/terminfo"
}

package_foot-terminfo() {
  pkgdesc="Terminfo files for the foot terminal emulator"
  depends=('ncurses')

  cd foot
  install -dm 755 "${pkgdir}/usr/share/terminfo/f/"
  cp build/f/* "${pkgdir}/usr/share/terminfo/f/"
}
