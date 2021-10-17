# Maintainer: Nathan Robinson <nrobinson2000 at me dot com>

pkgname=snd-hda-macbookpro
_gitname=snd_hda_macbookpro
pkgver=r29.dbbd7fe.5.14.12
pkgrel=1
pkgdesc="Kernel audio driver for Macs with 8409 HDA chip + MAX98706/SSM3515 amps"
arch=("x86_64")
url="https://github.com/davidjo/snd_hda_macbookpro"
license=("custom")
source=("git+${url}"
        "cirrus-makefile.patch")
sha256sums=('SKIP'
            '97c20c210ff812f7428646215e365f1928b838e78e8292aaaf0cdba0bb1b8236')
makedepends=("wget" "make" "gcc")

pkgver() {
    cd "$pkgname"
    printf "r%s.%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)" "$(uname -r | cut -d '-' -f1)"
}

prepare() {
    kernel_version=$(uname -r | cut -d '-' -f1)
    major_version=$(echo $kernel_version | cut -d '.' -f1)
    wget -c https://cdn.kernel.org/pub/linux/kernel/v$major_version.x/linux-$kernel_version.tar.xz -P $srcdir
    tar --strip-components=3 -xvf $srcdir/linux-$kernel_version.tar.xz linux-$kernel_version/sound/pci/hda --directory=build/
}

package() {
    update_dir="$pkgdir/usr/lib/modules/$(uname -r)/updates"
    patch_dir="$srcdir/$pkgname/patch_cirrus"
    hda_dir="$srcdir/hda"
    mkdir -p $update_dir

    # Apply patches
    cp $patch_dir/Makefile $patch_dir/patch_cirrus* $hda_dir
    patch $hda_dir/Makefile $srcdir/cirrus-makefile.patch

    # if kernel version is >= 5.5 then change
    # event = snd_hda_jack_tbl_get_from_tag(codec, tag);
    # to
    # event = snd_hda_jack_tbl_get_from_tag(codec, tag, 0);
    sed -i 's/event = snd_hda_jack_tbl_get_from_tag(codec, tag);/event = snd_hda_jack_tbl_get_from_tag(codec, tag, 0);/' $hda_dir/patch_cirrus.c

    # if kernel version >= 5.6 then
    # change timespec to timespec64
    # change getnstimeofday to ktime_get_real_ts64
    sed -i 's/timespec/timespec64/' $hda_dir/patch_cirrus.c
    sed -i 's/timespec/timespec64/' $hda_dir/patch_cirrus_new84.h
    sed -i 's/getnstimeofday/ktime_get_real_ts64/' $hda_dir/patch_cirrus.c
    sed -i 's/getnstimeofday/ktime_get_real_ts64/' $hda_dir/patch_cirrus_new84.h

    cd $hda_dir
    make
    make install KDIR="$pkgdir/usr/lib/modules/$(uname -r)"
}
