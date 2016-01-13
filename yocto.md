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
