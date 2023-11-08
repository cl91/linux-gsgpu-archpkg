# Maintainer: AndyRTR <andyrtr@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>

pkgbase=xorg-server
pkgname=('xorg-server' 'xorg-server-xephyr' 'xorg-server-xvfb' 'xorg-server-xnest'
         'xorg-server-common' 'xorg-server-devel')
pkgver=21.1.8
pkgrel=2
arch=('loong64' 'x86_64')
license=('custom')
groups=('xorg')
url="https://xorg.freedesktop.org"
makedepends=('xorgproto' 'pixman' 'libx11' 'mesa' 'mesa-libgl' 'xtrans'
             'libxkbfile' 'libxfont2' 'libpciaccess' 'libxv' 'libxcvt'
             'libxmu' 'libxrender' 'libxi' 'libxaw' 'libxtst' 'libxres'
             'xorg-xkbcomp' 'xorg-util-macros' 'xorg-font-util' 'libepoxy'
             'xcb-util' 'xcb-util-image' 'xcb-util-renderutil' 'xcb-util-wm' 'xcb-util-keysyms'
             'libxshmfence' 'libunwind' 'systemd' 'meson' 'git')
#source=(${pkgbase}-${pkgver}::git+https://gitlab.freedesktop.org/xorg/xserver.git#commit=27a0ee32ccef8d621aaa758c804fc9a5ceeb5a56
source=(https://xorg.freedesktop.org/releases/individual/xserver/${pkgbase}-${pkgver}.tar.xz{,.sig}
        xvfb-run # with updates from FC master
        xvfb-run.1
        10-modeset.conf
        xephyr_Dont_check_for_SeatId_anymore.patch
        0001-modesetting-match-against-Multimedia-Video-Controlle.patch
        0001-present-Send-a-PresentConfigureNotify-event-for-dest.patch
)
validpgpkeys=('3C2C43D9447D5938EF4551EBE23B7E70B467F0BF'  # Peter Hutterer (Who-T) <office@who-t.net>
              '67DC86F2623FC5FD4BB5225D14706DBE1E4B4540'  # Olivier Fourdan <fourdan@xfce.org>
              'FD0004A26EADFE43A4C3F249C6F7AE200374452D') # Povilas Kanapickas <povilas@radix.lt>
sha512sums=('6104b3620ed2e1e27d9a8e963388bbe8785a764585b1bc03dbf5d719a92894773dda580d377ca18ceeab353e65a5d23cc947bab84a4012f9dd1eca31cac36937'
            'SKIP'
            '672375cb5028ba9cda286e317d17bd8c9a9039483e7f79c21f223fd08ba07655729e9f59a082f4b8f5d8de45a77a9e9affce1002fb8c6657e26ef1a490654e49'
            'de5e2cb3c6825e6cf1f07ca0d52423e17f34d70ec7935e9dd24be5fb9883bf1e03b50ff584931bd3b41095c510ab2aa44d2573fd5feaebdcb59363b65607ff22'
            '1aa711f4948cd1557d77bd47a64ea92be55cf737b8204214c6c3ae2ecd00dc6928fd7a789d1aa5faaff5d6162f895a41efd332e8f1710d0a7c9b33326b057ec5'
            '34de52147054535256f35143d321e4d5e189baae502afca2bd3291094946dbead0829b1f196ae2a4d23bd6d0e1e04b65a387dee43f12dee55d247e37aec419d7'
            '0e55cc994dd8f1309c6ea40cb1f5b2d763d850a466c9d30bd8619fd68ce369b19f098d28ab684db1bd2758a6402654cf93188651e863f0e1be4eed8cf9b40e14'
            'bc7d054bef2b4550d067b3abb14eb7c534e5022ba0b0e59e14687886fe204914757b014c31150bce705baed5d8cb1be87e8624da43cb1fe874b138ced00ee18b')

prepare() {
  cd ${pkgbase}-$pkgver

  # FS#73274
  patch -Np1 -i ../xephyr_Dont_check_for_SeatId_anymore.patch
  # upstream fix (merged)
  patch -Np1 -i ../0001-present-Send-a-PresentConfigureNotify-event-for-dest.patch
  # fix modesetting driver for loongson and gsgpu
  patch -Np1 -i ../0001-modesetting-match-against-Multimedia-Video-Controlle.patch
}

build() {
  # Since pacman 5.0.2-2, hardened flags are now enabled in makepkg.conf
  # With them, module fail to load with undefined symbol.
  # See https://bugs.archlinux.org/task/55102 / https://bugs.archlinux.org/task/54845
  export CFLAGS=${CFLAGS/-fno-plt}
  export CXXFLAGS=${CXXFLAGS/-fno-plt}
  export LDFLAGS=${LDFLAGS/,-z,now}

  arch-meson ${pkgbase}-$pkgver build \
    -D ipv6=true \
    -D xvfb=true \
    -D xnest=true \
    -D xcsecurity=true \
    -D xorg=true \
    -D xephyr=true \
    -D glamor=true \
    -D udev=true \
    -D udev_kms=true \
    -D dtrace=false \
    -D systemd_logind=true \
    -D suid_wrapper=true \
    -D xkb_dir=/usr/share/X11/xkb \
    -D xkb_output_dir=/var/lib/xkb \
    -D libunwind=true

  # Print config
  meson configure build
  ninja -C build

  # fake installation to be seperated into packages
  DESTDIR="${srcdir}/fakeinstall" ninja -C build install
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    # use copy so a new file is created and fakeroot can track properties such as setuid
    cp -av "${src}" "${dir}/"
    rm -rf "${src}"
  done
}

package_xorg-server-common() {
  pkgdesc="Xorg server common files"
  depends=(xkeyboard-config xorg-xkbcomp xorg-setxkbmap)

  _install fakeinstall/usr/lib/xorg/protocol.txt
  _install fakeinstall/usr/share/man/man1/Xserver.1

  install -m644 -Dt "${pkgdir}/var/lib/xkb/" "${pkgbase}-${pkgver}"/xkb/README.compiled
  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING
}

package_xorg-server() {
  pkgdesc="Xorg X server"
  depends=(libepoxy libxfont2 pixman xorg-server-common libunwind
           dbus libgl xf86-input-libinput nettle
           libpciaccess libdrm libxshmfence libxcvt) # FS#52949
  # see xorg-server-*/hw/xfree86/common/xf86Module.h for ABI versions - we provide major numbers that drivers can depend on
  # and /usr/lib/pkgconfig/xorg-server.pc in xorg-server-devel pkg
  provides=('X-ABI-VIDEODRV_VERSION=25.2' 'X-ABI-XINPUT_VERSION=24.4' 'X-ABI-EXTENSION_VERSION=10.0' 'x-server')
  conflicts=('nvidia-utils<=331.20' 'glamor-egl' 'xf86-video-modesetting')
  replaces=('glamor-egl' 'xf86-video-modesetting')
  install=xorg-server.install

  _install fakeinstall/usr/bin/{X,Xorg,gtf}
  _install fakeinstall/usr/lib/Xorg{,.wrap}
  _install fakeinstall/usr/lib/xorg/modules/*
  _install fakeinstall/usr/share/X11/xorg.conf.d/10-quirks.conf
  _install fakeinstall/usr/share/man/man1/{Xorg,Xorg.wrap,gtf}.1
  _install fakeinstall/usr/share/man/man4/{exa,fbdevhw,inputtestdrv,modesetting}.4
  _install fakeinstall/usr/share/man/man5/{Xwrapper.config,xorg.conf,xorg.conf.d}.5

  # distro specific files must be installed in /usr/share/X11/xorg.conf.d
  install -m755 -d "${pkgdir}/etc/X11/xorg.conf.d"
  install -m644 -Dt "${pkgdir}/usr/share/X11/xorg.conf.d" 10-modeset.conf

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING
}

package_xorg-server-xephyr() {
  pkgdesc="A nested X server that runs as an X application"
  depends=(libxfont2 libgl libepoxy libunwind systemd-libs libxv pixman xorg-server-common
           xcb-util-image xcb-util-renderutil xcb-util-wm xcb-util-keysyms
           nettle libtirpc)

  _install fakeinstall/usr/bin/Xephyr
  _install fakeinstall/usr/share/man/man1/Xephyr.1

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING
}

package_xorg-server-xvfb() {
  pkgdesc="Virtual framebuffer X server"
  depends=(libxfont2 libunwind pixman xorg-server-common xorg-xauth 
           libgl nettle libtirpc systemd-libs)

  _install fakeinstall/usr/bin/Xvfb
  _install fakeinstall/usr/share/man/man1/Xvfb.1

  install -m755 "${srcdir}/xvfb-run" "${pkgdir}/usr/bin/"
  install -m644 "${srcdir}/xvfb-run.1" "${pkgdir}/usr/share/man/man1/" # outda

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING
}

package_xorg-server-xnest() {
  pkgdesc="A nested X server that runs as an X application"
  depends=(libxfont2 libunwind libxext pixman xorg-server-common nettle libtirpc systemd-libs)

  _install fakeinstall/usr/bin/Xnest
  _install fakeinstall/usr/share/man/man1/Xnest.1

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING
}

package_xorg-server-devel() {
  pkgdesc="Development files for the X.Org X server"
  depends=('xorgproto' 'mesa' 'libpciaccess'
           # not technically required but almost every Xorg pkg needs it to build
           'xorg-util-macros')

  _install fakeinstall/usr/include/xorg/*
  _install fakeinstall/usr/lib/pkgconfig/xorg-server.pc
  _install fakeinstall/usr/share/aclocal/xorg-server.m4

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${pkgbase}-${pkgver}"/COPYING

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir
}
