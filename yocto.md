# A Handy Little Yocto Cheat Sheet

_How to find the working dir and source dir of a recipe:_

	export recipe=busybox
	bitbake -e $recipe | grep "^WORKDIR="
	bitbake -e $recipe | grep "^S="

- - -

If you run into this issue while using the SDK:

	arm-poky-linux-gnueabi-ld.bfd: cannot find -lgcc
	make[2]: *** [examples/standalone/hello_world] Error 1
	make[1]: *** [examples/standalone] Error 2
	make: *** [examples] Error 2

The solution is:

	make LDFLAGS="" CC="$CC"
- - -

Standalone compile:

	.  /opt/poky/1.5.1/environment-setup-cortexa9hf-vfp-neon-poky-linux-gnueabi
	unset LDFLAGS

	# Note: Install uboot-mkimage if not installed
	sudo aptitude install u-boot-tools
	# export PATH=$PATH:/home/oney/repos/fsl-u-boot/tools/mkimage

	cd ~/repos/fsl-linux

	# only use the company line because everything else is broken
	make ARCH=arm imx_v7_defconfig
	make -j6 uImage LOADADDR=0x80008000
	make -j6 modules

	# make the dtb
	make imx6sx-sdb.dtb
- - -

To start a build:

	source ./setup-environment <directory>
	bitbake <image-name>

IMAGES:
tmp/deploy/images
SDcard: sudo dd if=core-image-minimal-imx6qsabrelite.sdcard of=/dev/sdb

Local configuration for boards:
conf/local.conf

New Development:
* Sync the source and create a branch

	cd ~/yocto
	repo sync
	repo start <branch> --all

* Setup environment

	source ./setup-environment build

* The previous command will create the build directory but it's better to name
* it relevant like machine-name-tag-build

* List machines available

	ls -l ~/yocto/sources/meta-fsl-arm/conf/machine/*.conf
	ls -l ~/yocto/sources/meta-fsl-arm-extra/conf/machine/*.conf

* List the image types available:

	find ~/yocto/sources -name '*image*'

	* core-image-minimal: Small image just to boot
	* core-image-base: console-only with all target hw support
	* core-image-sato: Sato is a mobile env & GUI for embedded. X11.
	* fsl-image-test: FSL multimedia + test apps
	* fsl-image-gui: core-image-sato + FSL multimedia + HW-accel X11

* Build with bitbake <image>

*** Directory structure:

	* build-<arch>-<iteration> : Build name for the machine arch like mx6
		|---- conf : configuration for this build
		|---- sstate-cache: pre-built cache for each package
		|---- tmp
	* downloads : It's like /opt/freescale/pkgs. All the sources.
	* sources : recipes etc.

* The tmp/ folder under the build has all the images and build results.

	* tmp
		|-- deploy : final images
		|-- buildstats: when each image was last built etc.
		|-- log : build logs. Logs have the PID of the task, i.e. no overwriting.
		|-- work : sources, patches, logs etc. for the last bitbake.

* work/ folder

	* work
		|-- <toolchain>/<package>/<version>/temp/log.* : Output from command
		|-- <toolchain>/<package>/<version>/temp/run.* : Script to execute the command

## To run a task,

	bitbake -c <task> <module> where task could be:
		clean, cleanall, package, install, configure, compile, **listtasks** etc.

* images folder

	* deploy
		|-- images/<board> : ext3, sdcard, tar.bz2 rootfs; uImage, u-boot.

** NFS Setup:

	sudo ~/bin/setup_nfs.sh mx6 sabresd <project-name>
	# populate the newly created directory
	sudo tar jxvf tmp/deploy/images/<dir>/<file>.tar.bz2 -C /export/<newdir>
	# copy uImage
	cp tmp/deploy/images/<dir>/uImage /tftpboot/uImage.<cpu>.<board>
	# copy dtb
	cp tmp/deploy/images/<dir>/uImage--3.10.17-r0-imx6q-sabresd-<date>.dtb /tftpboot/dtb.<project>

	# restart the service
	sudo service nfs-kernel-server restart
	# also add on linksys interface:
		Services->static leases

	# u-boot settings to modify:
	setenv uimage uImage.<cpu>.<board>.<project>
	setenv fdt_file dtb.<cpu>.<board>.<project>
	setenv nfsroot /export/<cpu>.<board>.<project>
	saveenv

** Toolchain build / setup

	. ./setup_environment <build-dir>
	bitbake meta-toolchain

	# the previous command will create an SDK that's installable.
	cd ~/yocto/<build-dir>/tmp/deploy/sdk
	./<toolchain-version>.sh
	# the default installation location is /opt/poky/<version>, /opt/poky/1.5.1
	# in our case.
	# To start using them, simply set up the environment:
	. /opt/poky/1.5.1/environment-setup-cortexa9hf-vfp-neon-poky-linux-gnueabi
	# don't forget to unset LDFLAGS if you want to build the kernel/u-boot
	unset LDFLAGS

	# this is not needed anymore with mkimage being available on Ubuntu
	export PATH=$PATH:/home/oney/repos/fsl-u-boot/tools/mkimage

** Building and debugging individual packages.

	# For the 3.10.17 ga paclage
	# Use the current directory we already set up
	MACHINE=imx6qdlsolo source ./fsl-setup-release.sh -b 317-ga-mx6q -e x11

	# Let's see the available tasks for imx-test, u-boot, and linux:
	bitbake -c listtasks imx-test
	bitbake -c listtasks u-boot-imx
	bitbake -c listtasks linux-imx

	bitbake -c fetch linux-imx

	# output:
	/home/oney/yocto/317-ga-mx6q/tmp/work/imx6qdlsolo-poky-linux-gnueabi/linux-imx/3.10.17-r0/git

	bitbake -c menuconfig linux-imx

	bitbake -c cleanall u-boot-imx # also removes the package
	bitbake -c build u-boot-imx

	# Making changes to the kernel and building it:

	export LINUX=/home/oney/yocto/317-ga-mx6q/tmp/work/imx6qdlsolo-poky-linux-gnueabi/linux-imx/3.10.17-r0/git
	cd $LINUX

	# edit the files, create a new branch and all that jazz
	# then use the -f flag to tell yocto to recompile. It's stupid and
	# doesn't know you changed files...
	bitbake -c compile linux-imx -f
	cp $LINUX/arch/arm/boot/uImage /tftpboot/<your-image-name>

	# to build all kernel (including the dtb files)
	bitbake linux-imx -c install -f -v

	# the image is at:
	export IMAGE=/home/oney/yocto/317-ga-mx6q/tmp/work/imx6qdlsolo-poky-linux-gnueabi/linux-imx/3.10.17-r0/image/boot/
- - -

# here is the setup:

cd ~/yocto2
repo init -u https://github.com/Freescale/fsl-community-bsp-platform -b daisy
repo sync

# now add imx-faes repo:
cd sources/
git clone https://bitbucket.org/fae_east_sw/meta-fsl-imx-faes

. ./setup-environment ima-faes-demos
vi /home/oney/yocto2/ima-faes-demos/conf/bblayers.conf

# add this line:

  ${BSPDIR}/sources/meta-fsl-imx-faes \

# verify:
bitbake-layers show-layers
	layer                 path                                      priority
	==========================================================================
	meta                  /home/oney/yocto2/sources/poky/meta       5
	meta-yocto            /home/oney/yocto2/sources/poky/meta-yocto  5
	meta-oe               /home/oney/yocto2/sources/meta-openembedded/meta-oe  6
	meta-fsl-arm          /home/oney/yocto2/sources/meta-fsl-arm    5
	meta-fsl-arm-extra    /home/oney/yocto2/sources/meta-fsl-arm-extra  4
	meta-fsl-demos        /home/oney/yocto2/sources/meta-fsl-demos  4
	meta-fsl-imx-faes     /home/oney/yocto2/sources/meta-fsl-imx-faes  7

# had to install some crap - might be an Ubuntu 14.04 thing
sudo apt-get install libsdl1.2-dev

# build imx-faes image
bitbake -k fsl-image-test-faes
- - -

* Error, TMPDIR has changed location. You need to either move it back to <dir> or rebuild
Do not EVER change the build directory. Perl @INC fails, the
location of quilt fails. Things just go crazy.

This happens when you move the build directory after a build. If you moved the
whole thing, then you can edit this file:

	new-build-dir/tmp/saved_tmpdir

Change the path. Also modify conf/local.conf to make sure the build is properly
aligned.

* unrecognized option '-Wl,-O1' when building the kernel/u-boot using the SDK

unset LDFLAGS as described here:
http://www.denx.de/wiki/view/ELDK-5/FrequentlyAskedQuestionsAndAnswers#Compiling_U_Boot_or_Linux_fails
- - -
