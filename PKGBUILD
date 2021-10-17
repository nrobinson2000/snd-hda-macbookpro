# Maintainer: Nathan Robinson <nrobinson2000 at me dot com>

pkgname=snd_hda_macbookpro
pkgver=r29.dbbd7fe
pkgrel=1
pkgdesc="Kernel audio driver for Macs with 8409 HDA chip + MAX98706/SSM3515 amps"
arch=("x86_64")
url="https://github.com/davidjo/snd_hda_macbookpro"
license=("custom")
source=("git+${url}")
md5sums=('SKIP')
depends=("make" "gcc")

pkgver() {
    cd "$pkgname"
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

package() {

kernel_version=$(uname -r | cut -d '-' -f1)  #ie 5.2.7
major_version=$(echo $kernel_version | cut -d '.' -f1)
minor_version=$(echo $kernel_version | cut -d '.' -f2)
major_minor=${major_version}${minor_version}

revision=$(uname -r | cut -d '.' -f3)
revpart1=$(echo $revision | cut -d '-' -f1)
revpart2=$(echo $revision | cut -d '-' -f2)
revpart3=$(echo $revision | cut -d '-' -f3)

build_dir='build'
update_dir="$pkgdir/usr/lib/modules/$(uname -r)/updates"
patch_dir="$srcdir/$pkgname/patch_cirrus"
hda_dir="$build_dir/hda-$kernel_version"

[[ ! -d $update_dir ]] && mkdir -p $update_dir
[[ ! -d $build_dir ]] && mkdir -p $build_dir
[[ -d $hda_dir ]] && rm -rf $hda_dir


# so for 5.4.0-48 version we need to use the patched Ubuntu source
# and not the mainline kernel source
# for the moment assume will be required for any further versions
# what about 5.5, 5.6, 5.8 etc???
if [ $major_version -eq 5 -a $minor_version -eq 4 -a $revpart2 -ge 48  -a `uname -v | grep -c Ubuntu` -eq 1 ]; then

	# supposedly this should download to here
	# well that fails - get the useless metadata package
	# (remember this script should be run as root)
	#apt source linux-source

	if [ ! -d /usr/src/linux-source-$kernel_version ]; then

		echo "linux kernel source not found in /usr/src: /usr/src/linux-source-$kernel_version"
		echo "assuming the linux kernel source package is not installed"
		echo "please install the linux kernel source package:"
		echo "sudo apt install linux-source"

		exit 1
	
	fi

	tar --strip-components=3 -xvf /usr/src/linux-source-$kernel_version.tar.bz2 linux-source-$kernel_version/sound/pci/hda

else

	# attempt to download linux-x.x.x.tar.xz kernel
	wget -c https://cdn.kernel.org/pub/linux/kernel/v$major_version.x/linux-$kernel_version.tar.xz -P $build_dir

	if [[ $? -ne 0 ]]; then
   		# if first attempt fails, attempt to download linux-x.x.tar.xz kernel
   		kernel_version=$major_version.$minor_version
   		wget -c https://cdn.kernel.org/pub/linux/kernel/v$major_version.x/linux-$kernel_version.tar.xz -P $build_dir
	fi

	[[ $? -ne 0 ]] && echo "kernel could not be downloaded...exiting" && exit

	tar --strip-components=3 -xvf $build_dir/linux-$kernel_version.tar.xz linux-$kernel_version/sound/pci/hda --directory=build/

fi

mv hda $hda_dir
mv $hda_dir/Makefile $hda_dir/Makefile.orig
mv $hda_dir/patch_cirrus.c $hda_dir/patch_cirrus.c.orig
cp $patch_dir/Makefile $patch_dir/patch_cirrus* $hda_dir/

# if kernel version is >= 5.5 then change
# event = snd_hda_jack_tbl_get_from_tag(codec, tag);
# to
# event = snd_hda_jack_tbl_get_from_tag(codec, tag, 0);
[[ $major_version -eq 5 && $minor_version -ge 5 ]] && sed -i 's/event = snd_hda_jack_tbl_get_from_tag(codec, tag);/event = snd_hda_jack_tbl_get_from_tag(codec, tag, 0);/' $hda_dir/patch_cirrus.c

# if kernel version >= 5.6 then
# change timespec to timespec64
# change getnstimeofday to ktime_get_real_ts64
if [ $major_version -eq 5 -a $minor_version -ge 6 ]; then
   sed -i 's/timespec/timespec64/' $hda_dir/patch_cirrus.c
   sed -i 's/timespec/timespec64/' $hda_dir/patch_cirrus_new84.h
   sed -i 's/getnstimeofday/ktime_get_real_ts64/' $hda_dir/patch_cirrus.c
   sed -i 's/getnstimeofday/ktime_get_real_ts64/' $hda_dir/patch_cirrus_new84.h
fi

cd $hda_dir

make 

KDIR="$pkgdir/usr/lib/modules/$(uname -r)"
make install KDIR="$KDIR"

}
