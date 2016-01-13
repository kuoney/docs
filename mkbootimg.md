This is about how to create a bootimg for Android using bits and pieces.

My current working directory is /home/oney/tmp/6sx

Tools:

Download
	mkbootimg tools: https://android-serialport-api.googlecode.com/files/android_bootimg_tools.tar.gz
	unmkbootimg: http://forum.xda-developers.com/attachment.php?attachmentid=1731547&d=1360931188

Both are needed.

First use unmkbootimg to separate the dtb from the rest:

	./unmkbootimg boot-imx6sx-lcd.img

	unmkbootimg version 1.2 - Mikael Q Kuisma <kuisma@ping.se>
	Kernel size 6675904
	Kernel address 0x80808000
	Ramdisk size 500217
	Ramdisk address 0x81800000
	Secondary size 45748
	Secondary address 0x81700000
	Kernel tags address 0x80800100
	Flash page size 2048
	Board name is ""
	Command line "console=ttymxc0,115200 init=/init androidboot.console=ttymxc0
	consoleblank=0 androidboot.hardware=freescale cma=384M"
	This image is built using standard mkbootimg
	Extracting kernel to file zImage ...
	Extracting root filesystem to file initramfs.cpio.gz ...
	Extracting second to file second.gz ...
	All done.
	---------------
	To recompile this image, use:
	  mkbootimg --kernel zImage --ramdisk initramfs.cpio.gz --base 0x80800000
	--cmdline 'console=ttymxc0,115200 init=/init androidboot.console=ttymxc0
	consoleblank=0 androidboot.hardware=freescale cma=384M' -o new_boot.img
	---------------

second.gz is an UNCOMPRESSED image that contains the dtb. Replace the dtb from
your build tree. I used arch/arm/boot/dts/imx6sx-sdb-lcdif1.dtb

Then use the mkimgboot tool in the first download like shown in the example
above.

	./mkbootimg --kernel unp/boot-imx6sx-lcd.img-zImage --ramdisk \
	unp/boot-imx6sx-lcd.img-ramdisk.gz --second new.dtb --cmdline \
	'console=ttymxc0,115200 init=/init androidboot.console=ttymxc0 \
	consoleblank=0 androidboot.hardware=freescale cma=384M' --base \
	0x80800000 --pagesize 2048 -o new3.img
