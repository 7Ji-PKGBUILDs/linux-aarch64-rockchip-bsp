# Maintainer: Jat-faan Wong

_desc="almost mainline rkbsp but with some patches, for Rockchip platforms"

pkgbase=linux-rkbsp
pkgname=(
  "${pkgbase}"
  "${pkgbase}-headers"
)
pkgver='6.1.43'
pkgrel=2
arch=('aarch64')
url="https://github.com/JeffyCN/mirrors"
_patchurl="https://github.com/wyf9661/linux-rkbsp"
license=('GPL2')
makedepends=( # Since we don't build the doc, most of the makedeps for other linux packages are not needed here
  'kmod' 'bc' 'dtc'
)
options=(!strip)
_srcname="kernel-6.1-2024_01_02"
_dirname="mirrors-${_srcname}"

source=(
  "${url}/archive/refs/tags/${_srcname}.tar.gz"
  "${_patchurl}/releases/download/${_srcname}/01-rockchip.patch"
  "${_patchurl}/releases/download/${_srcname}/02-downgrade-mali.patch"
  "${_patchurl}/releases/download/${_srcname}/03-add-usb-wifi-driver.patch" 
  '01-gcc-wrapper.patch'
  '02-gcc-ignore-stringop-overread-warnings.patch'
  'linux.preset'
)
sha512sums=(
  '954babdc708b17133b09e13c0be612ccf9c53a3753db8b99415960c20162f16c4fedea4a7ce64fdea3d82347949cfa886d6757e864c29660757b0ba8c1aaed76'
  '19c2d96e159e750d0374e053cc6308391d2cb102bbf2b302bb73282252445964f8f2146943369a4721d8dad7fab5ef3c9aba35e3736ff9582ce1e30fd059843d'
  'f5b2ccb31481db83e4b6caaee9488225875783dd73be5a356d95ee2d7de14efc1ba60015c888e23dd250662bc754167fc91fd1ed2516833617df9177466843ca'
  '2dc8ca376b2791707b1d82a07b5c66afa344d9f41007bb7a785f12d3efa1284f0a28b6bc4de906d2918af79f8d2ec8021a74bf60ff00331ef4b6e2859a9154f6'
  'a2daf21e3df0a0a50b0e81f4a163754acc08fb1104b875560a984123ccb83c31bd6fd47951e666faaa73723a400766cf9350b13d4ec0d566183f81cff03a68d8'
  '0cd82b847a501a09d47a81ae358528eb0e0605aa5e70f7c5f911ef58bca940fc61eb3bb7715a1eabd727528b4e50ef390c5935865fda870463adbdfe00b00ee0'
  '60d8c983976d37e218b17511586a316353a8ef14e08477c6d3b5b712d53886617a374b5ea9d2321e1a94c461cf979e6d94cf2c26c3df0da314e53a9223c8329f'
)

prepare() {
  cd "${_dirname}"

  echo "Patching kernel ..."
  for p in ../*.patch; do
    patch -p1 -N -i $p || true
  done

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  # Prepare the configuration file
  echo "Preparing config..."
  cat arch/arm64/configs/rockchip_defconfig > .config
  make olddefconfig prepare

  make -s kernelrelease > version
  echo "Prepared for $(<version)"
}

build() {
  cd "${_dirname}"
  # Host LDFLAGS or other LDFLAGS set by makepkg/user is not helpful for building kernel: it should links nothing outside of itself
  unset LDFLAGS
  # Only need normal Image, as most Amlogic devices does not need/support Image.gz
  # Image and modules are built in the same run to make sure they're compatible with each other
  # -@ enables symbols in dtbs, so overlay is possible
  make ${MAKEFLAGS} DTC_FLAGS="-@" Image modules dtbs
}
_package() {
  pkgdesc="The ${_srcname} of rkbsp, ${_desc}"
  depends=('coreutils' 'kmod' 'initramfs')
  optdepends=('wireless-regdb: to set the correct wireless channels of your country')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")

  cd "${_dirname}"
  
  # install dtbs
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs/${pkgbase}" dtbs_install

  # install modules
  make INSTALL_MOD_PATH="${pkgdir}/usr" INSTALL_MOD_STRIP=1 modules_install

  # copy kernel
  local _dir_module="${pkgdir}/usr/lib/modules/$(<version)"
  install -Dm644 arch/arm64/boot/Image "${_dir_module}/vmlinuz"

  # remove reference to build host
  rm -f "${_dir_module}/"{build,source}

  # used by mkinitcpio to name the kernel
  echo "${pkgbase}" | install -D -m 644 /dev/stdin "${_dir_module}/pkgbase"

  # install mkinitcpio preset file
  sed "s|%PKGBASE%|${pkgbase}|g" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the ${_srcname} of rkbsp, ${_desc}"
  depends=("python")

  cd "${_dirname}"
  local builddir="${pkgdir}/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map version
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  # add xfs and shmem for aufs building
  mkdir -p "$builddir"/{fs/xfs,mm}

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#${pkgbase}}
  }"
done
