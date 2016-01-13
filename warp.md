# WarpBoard Instructions using YOCTO

mkdir community-bsp
cd community-bsp/
repo init -u https://github.com/Freescale/fsl-community-bsp-platform -b fido
repo sync	# [ about 2 min ]

mkdir warp-build
export MACHINE=imx6sl-warp
source setup-environment warp-build

vi conf/local.conf	# [ 12 processes ]
bitbake core-image-base	# [ about 4 hrs ]

...

# let's install a toolchain
# METHOD 1
bitbake meta-toolchain	# [ 1 to 2 hours ]

./tmp/deploy/sdk/poky-glibc-x86_64-meta-toolchain-cortexa9hf-vfp-neon-toolchain-1.8.sh
Enter target directory for SDK (default: /opt/poky/1.8):
You are about to install the SDK to "/opt/poky/1.8". Proceed[Y/n]?y
[sudo] password for oney:

# better yet, METHOD 2
# This method has significant advantages over the previous method because it
# results in a toolchain installer that contains the sysroot that matches your
# target root filesystem

# NOTE: This toolchain produces junk u-boot with 1.8?
bitbake core-image-base -c populate_sdk	# [ a long time ]
./tmp/deploy/sdk/poky-glibc-x86_64-core-image-base-cortexa9hf-vfp-neon-toolchain-1.8.sh
Enter target directory for SDK (default: /opt/poky/1.8):
You are about to install the SDK to "/opt/poky/1.8". Proceed[Y/n]?y
[sudo] password for oney:

# Let's compile u-boot and linux for the warpboard
. /opt/poky/1.8/environment-setup-cortexa9hf-vfp-neon-poky-linux-gnueabi

cd ~/repos/u-boot/
make warp_config
make

cd ~/repos/linux/
make imx_v6_v7_defconfig
make -j6 all
mkdir ~/tmp/warp-modules
export INSTALL_MOD_PATH=~/tmp/warp-modules
make modules_install

# imx_usb_loader
cd $HOME/repos/warp/
git clone https://github.com/warpboard/imx_usb_loader.git -b warp/master
cd imx_usb_loader
sudo apt-get install libusb-1.0
make

# Boot the board in serial download mode and see it using
lsusb
	Bus 002 Device 027: ID 15a2:0063 Freescale Semiconductor, Inc.

sudo ./imx_usb /home/oney/community-bsp/warp-build/tmp/deploy/images/imx6sl-warp/u-boot-imx6sl-warp-v2015.04+gitAUTOINC+5d9ffd2214-r0.imx

# the board boots. Stop the boot process. Meanwhile on the laptop:
sudo apt-get install dfu-util

# On warpboard:
env default -f -a
saveenv
mmc rescan
ums 0 mmc 0

# make sure there are no errors by checking /var/log/syslog on the host
# then run this on the host:
sudo dd
if=/home/oney/community-bsp/warp-build/tmp/deploy/images/imx6sl-warp/core-image-base-imx6sl-warp-20150917224911.rootfs.sdcard of=/dev/sdb

