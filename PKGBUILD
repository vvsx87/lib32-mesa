# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

pkgbase=lib32-mesa
pkgname=('lib32-vulkan-intel' 'lib32-vulkan-radeon' 'lib32-libva-mesa-driver' 'lib32-mesa-vdpau' 'lib32-mesa')
pkgver=19.2.0
pkgrel=3
arch=('x86_64')
makedepends=('python-mako' 'lib32-libxml2' 'lib32-expat' 'lib32-libx11' 'glproto' 'lib32-libdrm' 'dri2proto' 'dri3proto' 'presentproto'
             'lib32-libxshmfence' 'lib32-libxxf86vm' 'lib32-libxdamage' 'gcc-multilib' 'lib32-libelf' 'lib32-llvm' 'lib32-libvdpau'
             'lib32-libva' 'lib32-wayland' 'wayland-protocols' 'lib32-libglvnd' 'lib32-lm_sensors' 'lib32-libxrandr' 'meson')
url="http://mesa3d.sourceforge.net"
license=('custom')
source=(https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz{,.sig}
        LICENSE
        glvnd.patch
	intel-topology-query-fix-old-gens.patch)
sha512sums=('7278bbfba9c29fe91d1959ff1a48422e917db85287460523d12ae8c6d7f49f76e9636bf4c0d8d7d89e5569b3c67135f1b23b8f6c9d52d39413d8ec22e3bb40f0'
            'SKIP'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7'
            '3e5746dcd493bff3f04b26de6168b15d0f161de62c1c6657106b61cbb1ad4925cbf3a691d5055491e759f88dbe0362dc909e7d726f87528980662f26ceb6dcbc'
            'a5e2ccef20edc81859255c66cb838c5244774d9d6c56dcfce2e462b6ddaa66ef7847242b050402305621c9c9e706629af30dd27c8466b6bd32d1be40cb3e53a0')
validpgpkeys=('8703B6700E7EE06D7A39B8D6EDAE37B02CEB490D'  # Emil Velikov <emil.l.velikov@gmail.com>
              '946D09B5E4C9845E63075FF1D961C596A7203456'  # Andres Gomez <tanty@igalia.com>
              'E3E8F480C52ADD73B278EE78E1ECBE07D7D70895'  # Juan Antonio Suárez Romero (Igalia, S.L.) <jasuarez@igalia.com>"
              'A5CC9FEC93F2F837CB044912336909B6B25FADFA'  # Juan A. Suarez Romero <jasuarez@igalia.com>
              '71C4B75620BC75708B4BDB254C95FAAB3EB073EC') # Dylan Baker <dylan@pnwbakers.com>
  
prepare() {
  cd mesa-${pkgver}

  # libglvnd-1.2.0 support
  patch -Np1 -i ${srcdir}/glvnd.patch
  # Fix FS#63945
  patch -Np1 -i ${srcdir}/intel-topology-query-fix-old-gens.patch
}

build() {
  export CC="gcc -m32"
  export CXX="g++ -m32"
  export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  export LLVM_CONFIG="/usr/bin/llvm-config32"
  
  arch-meson mesa-$pkgver build \
    --libdir=/usr/lib32 \
    -D b_lto=false \
    -D b_ndebug=true \
    -D platforms=x11,wayland,drm,surfaceless \
    -D dri-drivers=i915,i965,r100,r200,nouveau \
    -D gallium-drivers=r300,r600,radeonsi,nouveau,virgl,svga,swrast,iris \
    -D vulkan-drivers=amd,intel \
    -D swr-arches=avx,avx2 \
    -D dri3=true \
    -D egl=true \
    -D gallium-extra-hud=true \
    -D gallium-nine=true \
    -D gallium-omx=disabled \
    -D gallium-opencl=disabled \
    -D gallium-va=true \
    -D gallium-vdpau=true \
    -D gallium-xa=true \
    -D gallium-xvmc=false \
    -D gbm=true \
    -D gles1=false \
    -D gles2=true \
    -D glvnd=true \
    -D glx=dri \
    -D libunwind=false \
    -D llvm=true \
    -D lmsensors=true \
    -D osmesa=gallium \
    -D shared-glapi=true \
    -D valgrind=false

  # Print config
  meson configure build

  ninja -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="${srcdir}/fakeinstall" ninja -C build install
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

package_lib32-vulkan-intel() {
  pkgdesc="Intel's Vulkan mesa driver (32-bit)"
  depends=('lib32-wayland' 'lib32-libx11' 'lib32-libdrm' 'lib32-libxshmfence')
  provides=('lib32-vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/intel_icd*.json
  _install fakeinstall/usr/lib32/libvulkan_intel.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-vulkan-radeon() {
  pkgdesc="Radeon's Vulkan mesa driver (32-bit)"
  depends=('lib32-wayland' 'lib32-libx11' 'lib32-llvm-libs' 'lib32-libdrm' 'lib32-libelf' 'lib32-libxshmfence')
  provides=('lib32-vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/radeon_icd*.json
  _install fakeinstall/usr/lib32/libvulkan_radeon.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-libva-mesa-driver() {
  pkgdesc="VA-API implementation for gallium (32-bit)"
  depends=('lib32-libdrm' 'lib32-libx11' 'lib32-expat' 'lib32-llvm-libs' 'lib32-libelf' 'lib32-libxshmfence')

  _install fakeinstall/usr/lib32/dri/*_drv_video.so
   
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-mesa-vdpau() {
  pkgdesc="Mesa VDPAU drivers (32-bit)"
  depends=('lib32-libdrm' 'lib32-libx11' 'lib32-expat' 'lib32-llvm-libs' 'lib32-libelf' 'lib32-libxshmfence')

  _install fakeinstall/usr/lib32/vdpau
   
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_lib32-mesa() {
  pkgdesc="An open-source implementation of the OpenGL specification (32-bit)"
  depends=('lib32-libdrm' 'lib32-libxxf86vm' 'lib32-libxdamage' 'lib32-libxshmfence' 'lib32-lm_sensors'
           'lib32-libelf' 'lib32-llvm-libs' 'lib32-wayland' 'lib32-libglvnd' 'mesa')
  optdepends=('opengl-man-pages: for the OpenGL API man pages'
              'lib32-mesa-vdpau: for accelerated video playback')
  provides=('lib32-ati-dri' 'lib32-intel-dri' 'lib32-nouveau-dri' 'lib32-mesa-dri' 'lib32-mesa-libgl' 'lib32-opengl-driver')
  conflicts=('lib32-ati-dri' 'lib32-intel-dri' 'lib32-nouveau-dri' 'lib32-mesa-dri' 'lib32-mesa-libgl')
  replaces=('lib32-ati-dri' 'lib32-intel-dri' 'lib32-nouveau-dri' 'lib32-mesa-dri' 'lib32-mesa-libgl')

  # ati-dri, nouveau-dri, intel-dri, svga-dri, swrast, swr
  _install fakeinstall/usr/lib32/dri/*_dri.so
   
  _install fakeinstall/usr/lib32/d3d
  _install fakeinstall/usr/lib32/lib{gbm,glapi}.so*
  _install fakeinstall/usr/lib32/libOSMesa.so*
  _install fakeinstall/usr/lib32/libxatracker.so*
  _install fakeinstall/usr/lib32/pkgconfig

  # libglvnd support
  _install fakeinstall/usr/lib32/libGLX_mesa.so*
  _install fakeinstall/usr/lib32/libEGL_mesa.so*

  # indirect rendering
  ln -s /usr/lib32/libGLX_mesa.so.0 "${pkgdir}/usr/lib32/libGLX_indirect.so.0"
  
  rm -rv fakeinstall/usr/share/drirc.d
  rm -rv fakeinstall/usr/include
  rm -rv fakeinstall/usr/share

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}
