# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

pkgbase=lib32-mesa
pkgname=('lib32-vulkan-mesa-layers' 'lib32-opencl-mesa' 'lib32-vulkan-intel' 'lib32-vulkan-radeon' 'lib32-vulkan-virtio' 'lib32-libva-mesa-driver' 'lib32-mesa-vdpau' 'lib32-mesa')
pkgdesc="An open-source implementation of the OpenGL specification (32-bit)"
pkgver=22.3.2
pkgrel=5
arch=('x86_64')
makedepends=('python-mako' 'lib32-libxml2' 'lib32-expat' 'lib32-libx11' 'xorgproto' 'lib32-libdrm'
             'lib32-libxshmfence' 'lib32-libxxf86vm' 'lib32-libxdamage' 'lib32-libvdpau'
             'lib32-libva' 'lib32-wayland' 'wayland-protocols' 'lib32-zstd' 'lib32-libelf'
             'lib32-llvm' 'libclc' 'clang' 'lib32-clang' 'lib32-libglvnd' 'lib32-libunwind'
             'lib32-lm_sensors' 'lib32-libxrandr' 'lib32-vulkan-icd-loader' 'lib32-systemd'
             'glslang' 'cmake' 'meson')
url="https://www.mesa3d.org/"
license=('custom')
options=('!debug' '!lto')
source=(https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz{,.sig}
        0001-anv-force-MEDIA_INTERFACE_DESCRIPTOR_LOAD-reemit-aft.patch
        https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/20472.patch
        LICENSE)
sha512sums=('32934dd23cfcd6165c365597d9a469da0b806b72ea98a200f499344c3b47815db3bf78875b4ea766d2d28d9c70b50c1615d2d3fcbfd4769447fe0a9d3b32951f'
            'SKIP'
            'd02f3fd44cf95b7dbfd607a58b764bd79d02b8b8586acd37bd4b2340aea171410b2b5eda7eab5c5d2c87bbf512e2322d5468f95aab0bfedeabc5367ebdee3b1d'
            '9826cc08faa051248eb45c2e975f3c9a61250f1cf38df421392185dfa7bdb41bfa840df2e8a956b961ea9c4bcd62ff34f0460e7f5a99ef9298cdd7946b19237c'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7')
validpgpkeys=('8703B6700E7EE06D7A39B8D6EDAE37B02CEB490D'  # Emil Velikov <emil.l.velikov@gmail.com>
              '946D09B5E4C9845E63075FF1D961C596A7203456'  # Andres Gomez <tanty@igalia.com>
              'E3E8F480C52ADD73B278EE78E1ECBE07D7D70895'  # Juan Antonio Suárez Romero (Igalia, S.L.) <jasuarez@igalia.com>
              'A5CC9FEC93F2F837CB044912336909B6B25FADFA'  # Juan A. Suarez Romero <jasuarez@igalia.com>
              '71C4B75620BC75708B4BDB254C95FAAB3EB073EC'  # Dylan Baker <dylan@pnwbakers.com>
              '57551DE15B968F6341C248F68D8E31AFC32428A6') # Eric Engestrom <eric@engestrom.ch>

prepare() {
  cd mesa-$pkgver

  # https://gitlab.freedesktop.org/mesa/mesa/-/issues/7111
  # https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/17247
  # https://github.com/HansKristian-Work/vkd3d-proton/issues/1200
  patch -Np1 -i ../0001-anv-force-MEDIA_INTERFACE_DESCRIPTOR_LOAD-reemit-aft.patch
  
  # https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/20472
  patch -Np1 -i ../20472.patch
}

build() {
  # Build only minimal debug info to reduce size
  CFLAGS+=' -g1'
  CXXFLAGS+=' -g1'

  export CC="gcc -m32"
  export CXX="g++ -m32"
  export PKG_CONFIG="i686-pc-linux-gnu-pkg-config"
  cat >crossfile.ini <<END
[binaries]
llvm-config = '/usr/bin/llvm-config32'
END

  # swr driver is broken with some cpu see FS#66972

  arch-meson mesa-$pkgver build \
    --native-file crossfile.ini \
    --libdir=/usr/lib32 \
    -D b_ndebug=false \
    -D b_lto=false \
    -D platforms=x11,wayland \
    -D gallium-drivers=r300,r600,radeonsi,nouveau,virgl,svga,swrast,i915,iris,crocus,zink \
    -D vulkan-drivers=amd,intel,intel_hasvk,virtio-experimental \
    -D vulkan-layers=device-select,intel-nullhw,overlay \
    -D dri3=enabled \
    -D egl=enabled \
    -D gallium-extra-hud=true \
    -D gallium-nine=true \
    -D gallium-omx=disabled \
    -D gallium-opencl=icd \
    -D gallium-va=enabled \
    -D gallium-vdpau=enabled \
    -D gallium-xa=enabled \
    -D gbm=enabled \
    -D gles1=disabled \
    -D gles2=enabled \
    -D glvnd=true \
    -D glx=dri \
    -D libunwind=enabled \
    -D llvm=enabled \
    -D lmsensors=enabled \
    -D osmesa=true \
    -D shared-glapi=enabled \
    -D microsoft-clc=disabled \
    -D valgrind=disabled

  # Print config
  meson configure --no-pager build

  ninja -C build
  meson compile -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="${srcdir}/fakeinstall" meson install -C build
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    mv -v "${src}" "${dir}/"
  done
}

package_lib32-vulkan-mesa-layers() {
  pkgdesc="Mesa's Vulkan layers (32-bit)"
  depends=('lib32-libdrm' 'lib32-libxcb' 'lib32-wayland')
  depends+=('vulkan-mesa-layers')
  conflicts=('lib32-vulkan-mesa-layer')
  replaces=('lib32-vulkan-mesa-layer')

  rm -rv fakeinstall/usr/share/vulkan/explicit_layer.d
  rm -rv fakeinstall/usr/share/vulkan/implicit_layer.d
  rm -rv fakeinstall/usr/bin/mesa-overlay-control.py

  _install fakeinstall/usr/lib32/libVkLayer_*.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-opencl-mesa() {
  pkgdesc="OpenCL support for AMD/ATI Radeon mesa drivers (32-bit)"
  depends=('lib32-expat' 'lib32-libdrm' 'lib32-libelf' 'lib32-clang' 'lib32-zstd')
  depends+=('opencl-mesa')
  optdepends=('opencl-headers: headers necessary for OpenCL development')
  provides=('lib32-opencl-driver')

  rm -rv fakeinstall/etc/OpenCL
  _install fakeinstall/usr/lib32/lib*OpenCL*
  _install fakeinstall/usr/lib32/gallium-pipe

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-vulkan-intel() {
  pkgdesc="Intel's Vulkan mesa driver (32-bit)"
  depends=('lib32-wayland' 'lib32-libx11' 'lib32-libxshmfence' 'lib32-libdrm' 'lib32-zstd'
           'lib32-systemd')
  optdepends=('lib32-vulkan-mesa-layers: additional vulkan layers')
  provides=('lib32-vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/intel_*.json
  _install fakeinstall/usr/lib32/libvulkan_intel*.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-vulkan-radeon() {
  pkgdesc="Radeon's Vulkan mesa driver (32-bit)"
  depends=('lib32-wayland' 'lib32-libx11' 'lib32-libxshmfence' 'lib32-libelf' 'lib32-libdrm'
           'lib32-zstd' 'lib32-llvm-libs' 'lib32-systemd')
  depends+=('vulkan-radeon')
  optdepends=('lib32-vulkan-mesa-layers: additional vulkan layers')
  provides=('lib32-vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/radeon_icd*.json
  _install fakeinstall/usr/lib32/libvulkan_radeon.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-vulkan-virtio() {
  pkgdesc="Venus Vulkan mesa driver (32-bit)"
  depends=('lib32-wayland' 'lib32-libx11' 'lib32-libxshmfence' 'lib32-libdrm' 'lib32-zstd'
           'lib32-systemd')
  optdepends=('lib32-vulkan-mesa-layers: additional vulkan layers')
  provides=('lib32-vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/virtio_icd*.json
  _install fakeinstall/usr/lib32/libvulkan_virtio.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-libva-mesa-driver() {
  pkgdesc="VA-API implementation for gallium (32-bit)"
  depends=('lib32-libdrm' 'lib32-libx11' 'lib32-llvm-libs' 'lib32-expat' 'lib32-libelf'
           'lib32-libxshmfence' 'lib32-zstd')
  provides=('lib32-libva-driver')

  _install fakeinstall/usr/lib32/dri/*_drv_video.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-mesa-vdpau() {
  pkgdesc="Mesa VDPAU drivers (32-bit)"
  depends=('lib32-libdrm' 'lib32-libx11' 'lib32-llvm-libs' 'lib32-expat' 'lib32-libelf'
           'lib32-libxshmfence' 'lib32-zstd')
  provides=('lib32-vdpau-driver')

  _install fakeinstall/usr/lib32/vdpau

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-mesa() {
  depends=('lib32-libdrm' 'lib32-wayland' 'lib32-libxxf86vm' 'lib32-libxdamage' 'lib32-libxshmfence'
           'lib32-libelf' 'lib32-libunwind' 'lib32-llvm-libs' 'lib32-lm_sensors' 'lib32-libglvnd'
           'lib32-zstd' 'lib32-vulkan-icd-loader')
  depends+=('libsensors.so')
  depends+=('mesa')
  optdepends=('opengl-man-pages: for the OpenGL API man pages'
              'lib32-mesa-vdpau: for accelerated video playback'
              'lib32-libva-mesa-driver: for accelerated video playback')
  provides=('lib32-mesa-libgl' 'lib32-opengl-driver')
  conflicts=('lib32-mesa-libgl')
  replaces=('lib32-mesa-libgl')

  rm -rv fakeinstall/usr/share/drirc.d/00-mesa-defaults.conf
  rm -rv fakeinstall/usr/share/drirc.d/00-radv-defaults.conf
  rm -rv fakeinstall/usr/share/glvnd/egl_vendor.d/50_mesa.json

  # ati-dri, nouveau-dri, intel-dri, svga-dri, swrast, swr
  _install fakeinstall/usr/lib32/dri/*_dri.so

  #_install fakeinstall/usr/lib32/bellagio
  _install fakeinstall/usr/lib32/d3d
  _install fakeinstall/usr/lib32/lib{gbm,glapi}.so*
  _install fakeinstall/usr/lib32/libOSMesa.so*
  _install fakeinstall/usr/lib32/libxatracker.so*
  #_install fakeinstall/usr/lib32/libswrAVX*.so*

  rm -rv fakeinstall/usr/include
  _install fakeinstall/usr/lib32/pkgconfig

  # libglvnd support
  _install fakeinstall/usr/lib32/libGLX_mesa.so*
  _install fakeinstall/usr/lib32/libEGL_mesa.so*

  # indirect rendering
  ln -s /usr/lib32/libGLX_mesa.so.0 "${pkgdir}/usr/lib32/libGLX_indirect.so.0"

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}
