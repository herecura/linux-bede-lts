# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-bede-lts
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.9
_patchver=37
pkgrel=2
arch=('i686' 'x86_64')
license=('GPL2')
makedepends=('bc' 'kmod')
url="http://www.kernel.org"
options=(!strip)

validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'
    '647F28654894E3BD457199BE38DBBDC86092693E'
)

source=(
    "https://www.kernel.org/pub/linux/kernel/v4.x/linux-${_basekernel}.tar.xz"
    "https://www.kernel.org/pub/linux/kernel/v4.x/linux-${_basekernel}.tar.sign"
    # the main kernel config files
    "config-desktop.i686"
    "config-desktop.x86_64"
    # standard config files for mkinitcpio ramdisk
    "linux$_kernelname.preset"
    # pacman hooks
    'linux-bede-lts-01-depmod.hook'
    'linux-bede-lts-02-initcpio.hook'
    'linux-bede-lts-remove.hook'
    # sysctl tweaks
    'sysctl-linux-bede.conf'
)

# revision patches
if [ ${_patchver} -ne 0 ]; then
    pkgver=$_basekernel.$_patchver
    _patchname="patch-$pkgver"
    source=( "${source[@]}"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_patchname}.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_patchname}.sign"
    )
else
    pkgver=$_basekernel
fi

# extra patches
_extrapatches=(
    'apple-gmux.patch'
    'macbook-suspend.patch'
    'poweroff-quirk-workaround.patch'
)
if [ ${#_extrapatches[@]} -ne 0 ]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('bf67ff812cc3cb7e5059e82cc5db0d9a7c5637f7ed9a42e4730c715bf7047c81ed3a571225f92a33ef0b6d65f35595bc32d773356646df2627da55e9bc7f1f1a'
            'SKIP'
            '6d24b5e4ed0cb85d0e2c620a1ab184b60224ebeaae1515531b4b9971e5a6a3d3175488fa51cd038176ba35a0b08945483d8133f1bf5a01417d5369a0776e9944'
            '60e45212dd37b828a15bdd41f38e2842a4e93c9e2debb77861ba8d855034f62488183277a1ca30c5fccc2443f28f6c3216c74da61817f9ffa013d68873944e30'
            '501627d920b5482b99045b17436110b90f7167d0ed33fe3b4c78753cb7f97e7f976d44e2dae1383eae79963055ef74b704446e147df808cdcb9b634fd406e757'
            '6db48a1d85c6010ad1b0bf5208ba8da39a03f2b67d6e5da80bc054a8730c40e99bcc050f6e559ff813a7bfc561a3257f933474a55c59e5b0705248534bbba7af'
            '8f97c57bf456e9d5a696f93ee86b61411634f39c52dd3307a94eeb79d4d5951b69299001bf086fee32df4d2442fbc8977ac07afb25bc62f01d3f205353f851ae'
            '105e5c4eeb4431170154a719be8c5b6e49ba11abcd11e51d5f70a9d7af7f1da753b28bb9e378e068c37ac799f77907380fb9ea2ff6af3c25aaaf5a4c979993c1'
            'cc249aa48d362a570ec7e16fa9760552fd5fcc3615a29c154b2ee97e51c3c1c1c7449efd031bca59a7b65c473a2afaff075a043dbcb0fbf4a600c83cc9cb8f83'
            '3beddda62cca46e66c48b3dcec5292b9a3b8c068c29d7382ed1e27b4acf57a1ab45a966faccb2d23ecd114c0df9e662d852bde800337963be8d7cb061c185892'
            'SKIP'
            '7ec816bfb2e56016eb79614d1619a4921f46a55940b1a4e44d9490375bb63c15c6b61d6275354378d4edc1c88f93afbc08d193c269bcc57a350f9da095e91e10'
            'f27b52dbf081cb6402b651b02744c99d340eac886cc3deff95ad426246976f37c8c0acae5b5dc80c8b0a642d66882d3dddc841810ce08ae1519a3d0e8c8ce423'
            '5bf7e9487d3b31c0207a797b7abfd89794249f1dd16689423203722b201a7d1e40735ed957596ffb10b1dacb87d16b99d4560ff87aed7b24322c257c979d5acc')

prepare() {
    cd "$srcdir/linux-$_basekernel"

    # Add revision patches
    if [ $_patchver -ne 0 ]; then
        msg2 "apply $_patchname"
        patch -Np1 -i "$srcdir/$_patchname"
    fi

    # extra patches
    for patch in ${_extrapatches[@]}; do
        patch="$(basename "$patch" | sed -e 's/\.\(gz\|bz2\|xz\)//')"
        pext=${patch##*.}
        if [[ "$pext" == 'patch' ]] || [[ "$pext" == 'diff' ]]; then
            msg2 "apply $patch"
            patch -Np1 -i "$srcdir/$patch"
        fi
    done

    # set configuration
    msg2 "copy configuration"
    if [[ "$CARCH" = "x86_64" ]]; then
        cat "$srcdir/config-desktop.x86_64" >./.config
    else
        cat "$srcdir/config-desktop.i686" >./.config
    fi
    if [[ "$_kernelname" != "" ]]; then
        sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"\U$_kernelname\"|g" ./.config
    fi

    # set extraversion to pkgrel
    msg2 "set extraversion to $pkgrel"
    sed -ri "s|^(EXTRAVERSION =).*|\1 -$pkgrel|" Makefile

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh

    # hack to prevent output kernel from being marked as dirty or git
    msg2 "apply hack to prevent kernel tree being marked dirty"
    echo "" > "$srcdir/linux-$_basekernel/.scmversion"

}

build() {
    cd "$srcdir/linux-$_basekernel"

    msg2 "prepare"
    make prepare
    # load configuration
    # Configure the kernel. Replace the line below with one of your choice.
    #make menuconfig # CLI menu for configuration
    #make xconfig # X-based configuration
    #make oldconfig # using old config from previous kernel version
    # ... or manually edit .config
    ####################
    # stop here
    # this is useful to configure the kernel
    #msg "Stopping build"
    #return 1
    ####################
    # yes "" | make config
    # build!
    msg2 "build"
    make $MAKEFLAGS bzImage modules
}

package_linux-bede-lts() {
    pkgdesc="The Linux Kernel and modules, BlackEagle Desktop Edition"
    provides=('linux')
    backup=(
        "etc/mkinitcpio.d/$pkgname.preset"
    )
    depends=('coreutils' 'kmod>=10' 'mkinitcpio>=0.9')
    optdepends=(
        'crda: to set the correct wireless channels of your country'
        'linux-firmware: when having some hardware needing special firmware'
    )

    install=$pkgname.install

    KARCH=x86
    cd "$srcdir/linux-$_basekernel"

    mkdir -p "$pkgdir"/{lib/modules,lib/firmware,boot,usr}

    # get kernel version
    _kernver=$(make kernelrelease)

    # install modules
    make INSTALL_MOD_STRIP=1 INSTALL_MOD_PATH="$pkgdir" modules_install

    # install bzImage
    install -m644 arch/$KARCH/boot/bzImage "$pkgdir/boot/vmlinuz$_kernelname"

    # install fallback mkinitcpio.conf file and preset file for kernel
    install -m644 -D "$srcdir/$pkgname.preset" "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    # set correct depmod command for install
    sed \
        -e  "s/KERNEL_NAME=.*/KERNEL_NAME=$_kernelname/g" \
        -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=$_kernver/g" \
        -i "$startdir/$pkgname.install"
    sed \
        -e "s|source .*|source /etc/mkinitcpio.d/$pkgname.kver|g" \
        -e "s|default_image=.*|default_image=\"/boot/initramfs$_kernelname.img\"|g" \
        -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs$_kernelname-fallback.img\"|g" \
        -i "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    echo -e "# DO NOT EDIT THIS FILE\nALL_kver='$_kernver'" > "$pkgdir/etc/mkinitcpio.d/$pkgname.kver"

    # remove build and source links
    rm -f "$pkgdir/lib/modules/$_kernver"/{source,build}

    # remove the firmware
    rm -rf "$pkgdir/lib/firmware"

    _fldkernelname=$(echo $_kernelname | tr "[:lower:]" "[:upper:]")
    # make room for external modules
    ln -s "../${_basekernel}$_fldkernelname-external" "$pkgdir/lib/modules/$_kernver/external"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "$pkgdir/lib/modules/$_basekernel$_fldkernelname-external"
    echo "$_kernver" > "$pkgdir/lib/modules/${_basekernel}$_fldkernelname-external/version"

    # gzip all modules
    find "$pkgdir" -name '*.ko' -exec gzip -9 {}  \;

    # Now we call depmod...
    depmod -b "$pkgdir" -F System.map "$_kernver"

    # move module tree /lib -> /usr/lib
    mv "$pkgdir/lib" "$pkgdir/usr/"

    # install pacman hooks
    install -Dm644 "$srcdir/linux-bede-lts-01-depmod.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-lts-01-depmod.hook"
    install -Dm644 "$srcdir/linux-bede-lts-02-initcpio.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-lts-02-initcpio.hook"
    install -Dm644 "$srcdir/linux-bede-lts-remove.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-lts-remove.hook"

    # install sysctl tweaks
    install -Dm644 "$srcdir/sysctl-linux-bede.conf" "$pkgdir/usr/lib/sysctl.d/60-linux-bede-lts.conf"
}

package_linux-bede-lts-headers() {
    pkgdesc="Header files and scripts for building modules for linux$_kernelname"
    provides=('linux-headers')
    install -dm755 "$pkgdir/usr/lib/modules/$_kernver"
    cd "$pkgdir/usr/lib/modules/$_kernver"
    ln -sf ../../../src/linux-$_kernver build
    cd "$srcdir/linux-$_basekernel"
    install -D -m644 Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Makefile"
    install -D -m644 kernel/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/kernel/Makefile"
    install -D -m644 .config \
        "$pkgdir/usr/src/linux-$_kernver/.config"

    find . -path './include/*' -prune -o -path './scripts/*' -prune \
        -o -type f \( -name 'Makefile*' -o -name 'Kconfig*' \
        -o -name 'Kbuild*' -o -name '*.sh' -o -name '*.pl' \
        -o -name '*.lds' \) | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    cp -a scripts include "$pkgdir/usr/src/linux-$_kernver"
    find $(find arch/$KARCH -name include -type d -print) -type f \
        | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    install -Dm644 Module.symvers "$pkgdir/usr/src/linux-$_kernver"

    # add docbook makefile
    install -D -m644 Documentation/DocBook/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Documentation/DocBook/Makefile"

    # strip scripts directory
    find "$pkgdir/usr/src/linux-$_kernver/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
        case "$(file -bi "$binary")" in
            *application/x-sharedlib*) # Libraries (.so)
                /usr/bin/strip $STRIP_SHARED "$binary"
                ;;
            *application/x-archive*) # Libraries (.a)
                /usr/bin/strip $STRIP_STATIC "$binary"
                ;;
            *application/x-executable*) # Binaries
                /usr/bin/strip $STRIP_BINARIES "$binary"
                ;;
        esac
    done

    chown -R root:root "$pkgdir/usr/src/linux-$_kernver"
    find "$pkgdir/usr/src/linux-$_kernver" -type d -exec chmod 755 {} \;
}
