# Maintainer: Nathan Robinson <nrobinson2000 at me dot com>

pkgname=snd-hda-macbookpro
_gitname=snd_hda_macbookpro
pkgver=r29.dbbd7fe.5.10.83
pkgrel=1
pkgdesc="Kernel audio driver for Macs with 8409 HDA chip + MAX98706/SSM3515 amps"
arch=("x86_64")
url="https://github.com/davidjo/snd_hda_macbookpro"
license=("custom")
source=("git+${url}"
        "cirrus-makefile.patch"
        "linux-5.6.patch")

sha256sums=('SKIP'
            '9f94d088191e67a9d45b552b8ebd9ffef6e23592f1f2dfec0d1f04b2292eff65'
            '1b8bc2c9441c2a4cff0d22d4921a5199ea15a0a76d2764b96552bde436a11d6d')
makedepends=("wget" "make" "gcc")

pkgver() {
    cd "$_gitname"
    printf "r%s.%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)" "$(uname -r | cut -d '-' -f1)"
}

prepare() {
    kernel_version=$(uname -r | cut -d '-' -f1)
    major_version=$(echo $kernel_version | cut -d '.' -f1)
    wget -c https://cdn.kernel.org/pub/linux/kernel/v$major_version.x/linux-$kernel_version.tar.xz -P $srcdir
    tar --strip-components=3 -xf $srcdir/linux-$kernel_version.tar.xz linux-$kernel_version/sound/pci/hda --directory=build/
}

package() {
    update_dir="$pkgdir/usr/lib/modules/$(uname -r)/updates"
    patch_dir="$srcdir/$_gitname/patch_cirrus"
    hda_dir="$srcdir/hda"

    # Copy updated module sources
    cp $patch_dir/Makefile $patch_dir/patch_cirrus* $hda_dir

    # Apply patches
    patch -d $hda_dir -i $srcdir/cirrus-makefile.patch -p1
    patch -d $hda_dir -i $srcdir/linux-5.6.patch -p1

    # Build module
    mkdir -p $update_dir
    make -C $hda_dir
    make -C $hda_dir install KDIR="$pkgdir/usr/lib/modules/$(uname -r)"

    # Compress module
    xz $update_dir/snd-hda-codec-cirrus.ko
}
