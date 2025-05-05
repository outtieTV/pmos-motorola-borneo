# pmos-motorola-borneo
PostMarketOS on the Motorola Borneo<br />
<br />
<br />
Installation instructions:<br />
1. Install Ubuntu in a Virtual Machine or Natively. Do not use Ubuntu WSL
2. Open the terminal and grant sudo priveleges to your user.
3. Upgrade your operating system and install some new packages.<br />
$ sudo apt update && sudo apt upgrade -y && sudo apt install -y python-is-python3 kpartx git gh nano
4. Create a pmbootstrap command.<br />
$ sudo nano /usr/local/bin/pmbootstrap
```
#!/bin/bash
python $HOME/pmbootstrap/pmbootstrap.py "$@"
```
5. Make it executable and run it.<br />
$ sudo chmod +x /usr/local/bin/pmbootstrap<br />
$ pmbootstrap init
```
vendor = motorola
codename = borneo
Device architecture = aarch64
Manufacturer = Motorola
Name = Moto G Power
Year = 2021
Chassis = handset
sdcard: y
flash method: fastboot
boot.img: /home/$(whoami)/boot.img
user: user
Provider: wpa_supplicant
User interface: plasma-desktop
Install systemd? never
Change additional options? n
extra packages: none
Timezone: America/New_York
Locale: en_US
Device hostname: motorola-borneo
Build outdated packages: no
pass: 147147
```
6. Get checksums.<br />
$ pmbootstrap checksum device-motorola-borneo<br />
$ pmbootstrap shutdown<br />
$ pmbootstrap checksum linux-motorola-borneo<br />
<br />
$ pmbootstrap install<br />
#fails (but still creates /mnt directory)<br />
<br />
7. cd to linux-motorola-borneo folder and cp APKBUILD to home folder ~

$ cp ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/linux-motorola-borneo/APKBUILD ~/APKBUILD.bak
$ cp ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/linux-motorola-borneo/APKBUILD ~/APKBUILD<br />
8. Edit apkbuild file<br />
$ nano ~/APKBUILD
```
# Reference: <https://postmarketos.org/vendorkernel>
# Kernel config based on: arch/arm64/configs/borneo_defconfig

pkgname=linux-motorola-borneo
pkgver=3.2.0
pkgrel=0
pkgdesc="Motorola Moto G Power kernel fork"
arch="aarch64"
_carch="arm64"
_flavor="motorola-borneo"
url="https://kernel.org"
license="GPL-2.0-only"
options="!strip !check !tracedeps pmb:cross-native"
makedepends="
    bash
    bc
    bison
    devicepkg-dev
    findutils
    flex
    openssl-dev
    perl
    mkbootimg
    musl-dev
    clang
    lld
    llvm
    compiler-rt
    linux-headers
    xz
    xz-doc
"

# Source
_repository="android_kernel_motorola_sm6225"
_commit="f33c57c6ca93bb064c8cbc9a9c6cac08de1ae5e5"
_config="config-$_flavor.$arch"
source="
    $pkgname-$_commit.tar.gz::https://github.com/LineageOS/$_repository/archive/$_commit.tar.gz
    $_config
    baseMakefile.patch
"
builddir="$srcdir/$_repository-$_commit"
_outdir="out"

prepare() {
    export CFLAGS="-w -Wno-error -Wno-implicit-function-declaration -Wno-everything"
    export CXXFLAGS="-w -Wall -fcommon -fno-strict-aliasing -Wno-everything"
    export KCFLAGS="-w -Wno-error -Wno-implicit-function-declaration -Wno-everything"
    export LDFLAGS="-fcommon -Wl,-allow-multiple-definition"
    export REPLACE_GCCH=0
    default_prepare
    . downstreamkernel_prepare
}

build() {
    unset LDFLAGS
    LLVM=1 make O="$_outdir" ARCH="$_carch" CC=clang CLANG_TRIPLE=aarch64-linux-gnu- \
        CROSS_COMPILE=aarch64-linux-android- CROSS_COMPILE_ARM32=arm-linux-androidebi- \
        KBUILD_BUILD_VERSION="$((pkgrel + 1 ))-postmarketOS"
}

package() {
    downstreamkernel_package "$builddir" "$pkgdir" "$_carch" \
        "$_flavor" "$_outdir"

    LLVM=1 make dtbs_install O="$_outdir" ARCH="$_carch" CC=clang CLANG_TRIPLE=aarch64-linux-gnu- \
        CROSS_COMPILE=aarch64-linux-android- CROSS_COMPILE_ARM32=arm-linux-androidebi- \
        INSTALL_DTBS_PATH="$pkgdir"/boot/dtbs
}

sha512sums="(run 'pmbootstrap checksum linux-motorola-borneo' to fill)"
```
9. Copy apkbuild file.<br />
$ sudo cp APKBUILD ~/.local/var/pmbootstrap/chroot_native/home/pmos/build/APKBUILD<br />
$ sudo cp APKBUILD ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/linux-motorola-borneo/APKBUILD
10. Remove patch files.<br />
$ cd ~/.local/var/pmbootstrap/chroot_native/mnt/pmaports/device/testing/linux-motorola-borneo<br />
#or cd ~/.local/var/pmbootstrap/chroot_native/mnt/pmbootstrap/git/pmaports/device/testing/linux-motorola-borneo<br />
$ rm gcc7-give-up-on-ilog2-const-optimizations.patch<br />
$ rm gcc8-fix-put-user.patch<br />
$ rm gcc10-extern_YYLOC_global_declaration.patch<br />
$ rm kernel-use-the-gnu89-standard-explicitly.patch<br />
11. Create patch file.<br />
$ nano baseMakefile.patch
```
--- a/Makefile
+++ b/Makefile
@@ -517,7 +517,7 @@
 CLANG_FLAGS    += --gcc-toolchain=$(GCC_TOOLCHAIN)
 endif
 ifneq ($(LLVM_IAS),1)
-CLANG_FLAGS    += -no-integrated-as
+# CLANG_FLAGS  += -no-integrated-as
 endif
 CLANG_FLAGS    += -Werror=unknown-warning-option
 CLANG_FLAGS    += $(call cc-option, -Wno-unsequenced)
```
#optional patch for baseMakefile.patch<br />
$ sed -i 's/^    /\t/' baseMakefile.patch #replace 4 spaces with \t<br />
12. Get ready for installation:<br />
$ pmbootstrap kconfig edit linux-motorola-borneo<br />
$ pmbootstrap build device-motorola-borneo<br />
$ pmbootstrap build linux-motorola-borneo --arch=aarch64<br />
$ cd ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/device-motorola-borneo<br />
$ nano APKBUILD
```
depends="
linux-postmarketos-qcom-sm6115
mkbootimg
postmarketos-base
"
```
$ pmbootstrap install --android-recovery-zip<br />
$ pmbootstrap export<br />
$ cp -rL /tmp/postmarketOS-export /mnt/d/pmos #or whichever folder you want<br />
