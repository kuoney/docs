# How to setup and compile kobs-ng:

Clone this repo:

	git://sw-git01-tx30.am.freescale.net/linux-kobs.git

# Setup the cross build environment:

	. /opt/poky/1.5.1/environment-setup-cortexa9hf-vfp-neon-poky-linux-gnueabi
	unset LDFLAGS

# The commands are in INSTALL file

	aclocal
	autoheader
	automake --foreign --copy --add-missing
	autoconf
	./configure --build=x86_64-linux --host=arm-poky-linux-gnueabi		\
		--target=arm-poky-linux-gnueabi  --sysconfdir=/etc		\
		--sharedstatedir=/com --localstatedir=/var --libdir=/usr/lib	\
		--includedir=/usr/include --oldincludedir=/usr/include		\
		--infodir=/usr/share/info --mandir=/usr/share/man		\
		--disable-silent-rules --disable-dependency-tracking		\
		--with-libtool-sysroot=/home/oney/yocto2/ima-faes-demos/tmp/sysroots/imx6qsabresd
	make

# The binary is src/kobs-ng
	file src/kobs-ng
	./src/kobs-ng: ELF 32-bit LSB  executable, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.16, BuildID[sha1]=13c4356011caf8eb5bdc508e70bb4e18b8f6e652, not stripped
