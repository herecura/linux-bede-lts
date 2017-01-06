# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-bede-lts
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.4
_patchver=40
pkgrel=1
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
    #'sysctl-linux-bede.conf'
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

sha256sums=('401d7c8fef594999a460d10c72c5a94e9c2e1022f16795ec51746b0d165418b2'
            'SKIP'
            '71b2267ec82d68e832b77623ffd0e4cd9adeef8127808c497b593cf955db97a2'
            'c57cab431d070299e9fd34d920e1d746a8e8f58c48a5f694ba88576f7d129a5c'
            'd5bb4aabbd556f8a3452198ac42cad6ecfae020b124bcfea0aa7344de2aec3b5'
            '52702085e848078f52ac0f6bd1f35d33632dfbd5bc327522f4f67a889d8e208c'
            '76c5b865f8c1d74f582f26ee68ad389baaf99e18604a85eee046e5ccc8de5469'
            '945cd7a2e7d16f2383b9fcd32eba46e873e8b7d3c5c0a855af7c84ab577fb00a'
            '0dc651926071934baeffd6bb0678f549af3af35f06e6484311de955e3202519f'
            'SKIP'
            'bb8af32880059e681396a250d8e78f600f248da8ad4f0e76d7923badb5ee8b42'
            '4d4a622733c2ba742256f369c32a1e98fc216966589f260c7457d299dbb55971'
            '09189eb269a9fd16898cf90a477df23306236fb897791e8d04e5a75d5007bbff')


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

    # copy System.map and bzImage
    install -m644 System.map "$pkgdir/boot/System.map$_kernelname"
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
    #install -Dm644 "$srcdir/sysctl-linux-bede.conf" "$pkgdir/usr/lib/sysctl.d/60-linux-bede-lts.conf"
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
