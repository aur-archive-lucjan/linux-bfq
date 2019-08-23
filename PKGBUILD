# Maintainer: Piotr Gorski <lucjan.lucjanov@gmail.com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

### BUILD OPTIONS
# Set these variables to ANYTHING that is not null to enable them

### Tweak kernel options prior to a build via nconfig
_makenconfig=

### Tweak kernel options prior to a build via menuconfig
_makemenuconfig=

### Tweak kernel options prior to a build via xconfig
_makexconfig=

### Tweak kernel options prior to a build via gconfig
_makegconfig=

# NUMA is optimized for multi-socket motherboards.
# A single multi-core CPU actually runs slower with NUMA enabled.
# See, https://bugs.archlinux.org/task/31187
_NUMAdisable=y

# Compile ONLY probed modules
# Build in only the modules that you currently have probed in your system VASTLY
# reducing the number of modules built and the build time.
#
# WARNING - ALL modules must be probed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD will call it directly to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed-db
_localmodcfg=

# Use the current kernel's .config file
# Enabling this option will use the .config of the RUNNING kernel rather than
# the ARCH defaults. Useful when the package gets updated and you already went
# through the trouble of customizing your config options.  NOT recommended when
# a new kernel is released, but again, convenient for package bumps.
_use_current=

### Running with a 1000 HZ tick rate
_1k_HZ_ticks=

### Do not edit below this line unless you know what you're doing

pkgbase=linux-bfq
# pkgname=('linux-bfq' 'linux-bfq-headers' 'linux-bfq-docs')
_major=5.2
_minor=9
pkgver=${_major}.${_minor}
_srcname=linux-${pkgver}
pkgrel=4
arch=('x86_64')
url="https://github.com/sirlucjan/bfq-mq-lucjan"
license=('GPL2')
options=('!strip')
makedepends=('kmod' 'inetutils' 'bc' 'libelf' 'python-sphinx' 'python-sphinx_rtd_theme'
             'graphviz' 'imagemagick')
#_lucjanpath="https://raw.githubusercontent.com/sirlucjan/kernel-patches/master/${_major}"
_lucjanpath="https://gitlab.com/sirlucjan/kernel-patches/raw/master/${_major}"
_bfq_path_1="bfq-paolo-dev-lucjan-for-5.2.3"
_bfq_patch_1="0001-block-bfq-dev-lucjan.patch"
_bfq_path_2="bfq-paolo-dev-lucjan-v2"
_bfq_patch_2="0002-block-bfq-dev-lucjan.patch"
_bfq_path_3="bfq-paolo-dev-lucjan-v3"
_bfq_patch_3="0003-block-bfq-dev-lucjan.patch"
_bfq_path_4="bfq-paolo-dev-lucjan-v4"
_bfq_patch_4="0004-block-bfq-dev-lucjan.patch"
_bfq_path_5="bfq-paolo-dev-lucjan-v5r2"
_bfq_patch_5="0005-block-bfq-dev-lucjan.patch"
_bfq_path_6="bfq-paolo-dev-lucjan-v6"
_bfq_patch_6="0006-block-bfq-dev-lucjan.patch"
_gcc_path="https://raw.githubusercontent.com/graysky2/kernel_gcc_patch/master"
_gcc_patch="enable_additional_cpu_optimizations_for_gcc_v9.1+_kernel_v4.13+.patch"

source=("https://www.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar.xz"
        "https://www.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar.sign"
        "${_gcc_path}/${_gcc_patch}"
        "${_lucjanpath}/${_bfq_path_1}/${_bfq_patch_1}"
        "${_lucjanpath}/${_bfq_path_2}/${_bfq_patch_2}"
        "${_lucjanpath}/${_bfq_path_3}/${_bfq_patch_3}"
        "${_lucjanpath}/${_bfq_path_4}/${_bfq_patch_4}"
        "${_lucjanpath}/${_bfq_path_5}/${_bfq_patch_5}"
        "${_lucjanpath}/${_bfq_path_6}/${_bfq_patch_6}"
        "${_lucjanpath}/arch-patches/0001-add-sysctl-to-disallow-unprivileged-CLONE_NEWUSER-by.patch"
        "${_lucjanpath}/arch-patches/0002-ZEN-Add-CONFIG-for-unprivileged_userns_clone.patch"
        "${_lucjanpath}/arch-patches/0003-iwlwifi-mvm-disable-TX-AMSDU-on-older-NICs.patch"
        "${_lucjanpath}/arch-patches/0004-iwlwifi-Add-support-for-SAR-South-Korea-limitation.patch"
         # the main kernel config files
        'config'
         # pacman hook for depmod
        '60-linux.hook'
         # pacman hook for initramfs regeneration
        '90-linux.hook'
         # pacman hook for remove initramfs
        '99-linux.hook'
         # standard config files for mkinitcpio ramdisk
        'linux.preset')

_kernelname=${pkgbase#linux}
: ${_kernelname:=-bfq}

prepare() {
    cd ${_srcname}

    ### Setting version
        msg2 "Setting version..."
        sed -e "/^EXTRAVERSION =/s/=.*/=/" -i Makefile
        scripts/setlocalversion --save-scmversion
        echo "-$pkgrel" > localversion.10-pkgrel
        echo "$_kernelname" > localversion.20-pkgname

    ### Patching sources
        local src
        for src in "${source[@]}"; do
            src="${src%%::*}"
            src="${src##*/}"
            [[ $src = *.patch ]] || continue
        msg2 "Applying patch $src..."
        patch -Np1 < "../$src"
        done

    ### Setting config
        msg2 "Setting config..."
        cp ../config .config
        make olddefconfig

    ### Prepared version
        make -s kernelrelease > ../version
        msg2 "Prepared %s version %s" "$pkgbase" "$(<../version)"

    ### Optionally use running kernel's config
	# code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
	if [ -n "$_use_current" ]; then
		if [[ -s /proc/config.gz ]]; then
			msg2 "Extracting config from /proc/config.gz..."
			# modprobe configs
			zcat /proc/config.gz > ./.config
		else
			warning "Your kernel was not compiled with IKCONFIG_PROC!"
			warning "You cannot read the current config!"
			warning "Aborting!"
			exit
		fi
	fi

	
    ### Optionally set tickrate to 1000
	if [ -n "$_1k_HZ_ticks" ]; then
		msg2 "Setting tick rate to 1k..."
		sed -i -e 's/^CONFIG_HZ_300=y/# CONFIG_HZ_300 is not set/' \
                    -i -e 's/^# CONFIG_HZ_1000 is not set/CONFIG_HZ_1000=y/' \
                    -i -e 's/^CONFIG_HZ=300/CONFIG_HZ=1000/' ./.config
	fi
	
    ### Optionally disable NUMA for 64-bit kernels only
        # (x86 kernels do not support NUMA)
        if [ -n "$_NUMAdisable" ]; then
            msg2 "Disabling NUMA from kernel config..."
            sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
                -i -e '/CONFIG_AMD_NUMA=y/d' \
                -i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
                -i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
                -i -e '/# CONFIG_NUMA_EMU is not set/d' \
                -i -e '/CONFIG_NODES_SHIFT=6/d' \
                -i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
                -i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
                -i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
                -i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
        fi

    ### Optionally load needed modules for the make localmodconfig
        # See https://aur.archlinux.org/packages/modprobed-db
        if [ -n "$_localmodcfg" ]; then
        msg2 "If you have modprobed-db installed, running it in recall mode now"
            if [ -e /usr/bin/modprobed-db ]; then
            [[ -x /usr/bin/sudo ]] || {
            echo "Cannot call modprobe with sudo. Install sudo and configure it to work with this user."
            exit 1; }
            sudo /usr/bin/modprobed-db recall
            make localmodconfig
            fi
        fi

    ### Running make nconfig
	
	[[ -z "$_makenconfig" ]] ||  make nconfig
	
    ### Running make menuconfig
	
	[[ -z "$_makemenuconfig" ]] || make menuconfig
	
    ### Running make xconfig
	
	[[ -z "$_makexconfig" ]] || make xconfig
	
    ### Running make gconfig
	
	[[ -z "$_makegconfig" ]] || make gconfig

    ### Save configuration for later reuse
	cat .config > "${startdir}/config.last"
}

build() {
  cd ${_srcname}

  make bzImage modules htmldocs
}

_package() {
    pkgdesc="The ${pkgbase/linux/Linux} kernel and modules with the BFQ-dev"
    depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
    optdepends=('crda: to set the correct wireless channels of your country' 'modprobed-db: Keeps track of EVERY kernel module that has ever been probed - useful for those of us who make localmodconfig')
    backup=("etc/mkinitcpio.d/$pkgbase.preset")
    install=linux.install

  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  cd $_srcname

  msg2 "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"
  install -Dm644 "$modulesdir/vmlinuz" "$pkgdir/boot/vmlinuz-$pkgbase"

  msg2 "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" modules_install

  # a place for external modules,
  # with version file for building modules and running depmod from hook
  local extramodules="extramodules$_kernelname"
  local extradir="$pkgdir/usr/lib/modules/$extramodules"
  install -Dt "$extradir" -m644 ../version
  ln -sr "$extradir" "$modulesdir/extramodules"

  # remove build and source links
  rm "$modulesdir"/{source,build}

  msg2 "Installing hooks..."

  # sed expression for following substitutions
  local subst="
    s|%PKGBASE%|$pkgbase|g
    s|%KERNVER%|$kernver|g
    s|%EXTRAMODULES%|$extramodules|g
  "

  # hack to allow specifying an initially nonexisting install file
  sed "$subst" "$startdir/$install" > "$startdir/$install.pkg"
  true && install=$install.pkg

  # fill in mkinitcpio preset and pacman hooks
  sed "$subst" ../linux.preset | install -Dm644 /dev/stdin \
    "$pkgdir/etc/mkinitcpio.d/$pkgbase.preset"
  sed "$subst" ../60-linux.hook | install -Dm644 /dev/stdin \
    "$pkgdir/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "$subst" ../90-linux.hook | install -Dm644 /dev/stdin \
    "$pkgdir/usr/share/libalpm/hooks/90-${pkgbase}.hook"
  sed "$subst" ../99-linux.hook | install -Dm644 /dev/stdin \
    "$pkgdir/usr/share/libalpm/hooks/99-${pkgbase}.hook"

  msg2 "Fixing permissions..."
  chmod -Rc u=rwX,go=rX "$pkgdir"
}


_package-headers() {
   pkgdesc="Header files and scripts for building modules for ${pkgbase/linux/Linux} kernel"
   depends=('linux-bfq')

  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  cd $_srcname

  msg2 "Installing build files..."
  install -Dt "$builddir" -m644 Makefile .config Module.symvers System.map vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  # add objtool for external module building and enabled VALIDATION_STACK option
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # add xfs and shmem for bfq building
  mkdir -p "$builddir"/{fs/xfs,mm}

  # ???
  mkdir "$builddir/.tmp_versions"

  msg2 "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  msg2 "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  msg2 "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  msg2 "Removing documentation..."
  rm -r "$builddir/Documentation"

  msg2 "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  msg2 "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  msg2 "Stripping build tools..."
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
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  msg2 "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase-$pkgver"

  msg2 "Fixing permissions..."
  chmod -Rc u=rwX,go=rX "$pkgdir"
}


_package-docs() {
    pkgdesc="Kernel hackers manual - HTML documentation that comes with the ${pkgbase/linux/Linux} kernel"
    depends=('linux-bfq')

  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  cd $_srcname

  msg2 "Installing documentation..."
  mkdir -p "$builddir"
  cp -t "$builddir" -a Documentation

  msg2 "Removing doctrees..."
  rm -r "$builddir/Documentation/output/.doctrees"

  msg2 "Moving HTML docs..."
  local src dst
  while read -rd '' src; do
    dst="$builddir/Documentation/${src#$builddir/Documentation/output/}"
    mkdir -p "${dst%/*}"
    mv "$src" "$dst"
    rmdir -p --ignore-fail-on-non-empty "${src%/*}"
  done < <(find "$builddir/Documentation/output" -type f -print0)
  
  msg2 "Adding symlink..."
  mkdir -p "$pkgdir/usr/share/doc"
  ln -sr "$builddir/Documentation" "$pkgdir/usr/share/doc/$pkgbase"

  msg2 "Fixing permissions..."
  chmod -Rc u=rwX,go=rX "$pkgdir"
}


pkgname=("$pkgbase" "$pkgbase-headers" "$pkgbase-docs")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

sha512sums=('2e3fad96cce66e1ef0a8f82c221970e6535d2d20b5ed8dcc3f368cd51576ff57bd31f5b5b09f7b19acee4f73c7ef5f1a0aa8ecb5176c431c2d4cb5c770432f4b'
            'SKIP'
            '2eb574fbfac6e334d3b06e52e466dbf8e88034515729b6571990b10f75a0fe2a52f188615405c5a695b5820669e595deead44d7961a97c5872359be3435fdf63'
            'd996a89d7831c4d49980b1b5b73ccd21da419302e86ea6faaf7d8f5a686bdbfe688a75c34d3e9583cbb11a011d2adcb4cb3aef66aba677e0d810f3eb1fb5be4b'
            '8ce2e2c3da37435090abdf0d1875aa0b2870d9cb9ce148cdc30855c1beeffc87cd7a8003c3c351f8f31753c1a4678df6cb3c02c39797b361632c9e5abbe2173f'
            '960028ff15108a88e67caf7769b21e9989aa5c56120fa417baa5a721ef09fb86e73d2ba9f1def27b0f98ac935cc670799b3fbb2b78b98b26516a9063b4c84e5f'
            '831337f1b6042d97dc79cc5768a31f97773248a41af4704742deff4e3224aa0c57e7645d939b9fab9140fc24f0c879fabef85ed58255844ac507f9e971840ee1'
            '2c8721cdc9a975955aaf8ef95d0917a1c39841edb036d4115a62e8a00998d63147d011453bd3c18d7a4732aed8acf41f376548c9c5d5e791fa58005dad6b9fef'
            'd0480c545bacf3a47f99065e29e7854e840de1cd91d682663d8cb4d23f2d000f48c2de5ab648b85f8748ddb8e2984ce2fd788076fe5e807274187920bd630fe7'
            '8a158ae5660426f0fb9fa74f37c093f325b5a23392b95e50c36825205f42311cbf05f818f0275ebda27ce9fad40652ba6dc2397f19f45addb3f68f8f4476196f'
            '9d472377c50ddd9ddab5d4ab3092f3d487eeee20a4d4a04d3cefeef5bbf40daa3eb609b15741ca4a8cd32bf2b6d0781de961cbf55e2f80fd40260f282bf022f4'
            '166653c6d7d76b052aa26f6a0782a74db75d2611c3d5d770219a00fed4da7d2317b35e39aaadec25da1fc91fb7f18385247303d531d15e3998886bb65f8cd89f'
            '1e422fc04acad01380b96dad92b43bbc2ddb26e3f109c519534fe6635e6fc1decebeff22b6f31be773330f1d4504883264e719889a4961a03c31670dbc2d9859'
            '513453f463fa0ba6afc91e65f4b23828e69ccc899ed86bf136efa2b6077ce950eb5fb8605d4508ef7726970a2140ab76d2c3a02e2f1334a85143bc625f2f0c74'
            '7ad5be75ee422dda3b80edd2eb614d8a9181e2c8228cd68b3881e2fb95953bf2dea6cbe7900ce1013c9de89b2802574b7b24869fc5d7a95d3cc3112c4d27063a'
            '2718b58dbbb15063bacb2bde6489e5b3c59afac4c0e0435b97fe720d42c711b6bcba926f67a8687878bd51373c9cf3adb1915a11666d79ccb220bf36e0788ab7'
            '8742e2eed421e2f29850e18616f435536c12036ff793f5682a3a8c980cf5dbfc88d17fd9539c87de15d9e4663dc3190f964f18a4722940465437927b6052abbf'
            '2dc6b0ba8f7dbf19d2446c5c5f1823587de89f4e28e9595937dd51a87755099656f2acec50e3e2546ea633ad1bfd1c722e0c2b91eef1d609103d8abdc0a7cbaf')

validpgpkeys=(
              '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
             )
