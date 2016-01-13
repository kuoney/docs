Taken from:
https://groups.google.com/forum/#%21topic/wandboard/PHv-yXyhEWM

In case anyone is following in my footsteps, here are some tips on how I managed to get the Vivante samples to build and run:

1.  Dora requires the X11 and Wayland packages to be included the first time you build it, but the Vivante samples will not run unless X11 and Wayland are removed, even though the Vivante samples are part of the Dora release.   This is rather counter-intuitive, to say the least.

2.  So, after your first build, then build Dora again with X11 and Wayland removed to get a linux platform that will allow OpenGL ES applications to run in framebuffer (full-screen) mode.  Add these lines to your local.conf file:

    DISTRO_FEATURES_remove = "x11 wayland"
    IMAGE_ROOTFS_EXTRA_SPACE = "3145728"          (For a 4GB card)
  
3.  The sample apps must be built for hard-float to run on Dora.  There are pre-built Vivante hard-float sample apps which can be run in the Dora image at:

     /opt/viv_samples

4.  To build a toolchain that will build the Vivante samples for hard-float and Dora, use:

    cd  ~/Yocto/fsl-community-bsp
    MACHINE=wandboard-quad
    source setup-environment  build
    bitbake  meta-toolchain
    
5. Install the new built toolchain with this generated script:

    cd ~/Yocto/fsl-community-bsp/build/tmp/deploy/sdk
    ./poky-eglibc-i686-meta-toolchain-cortexa9hf-vfp-neon-toolchain-1.5.sh

    The default target directory for the new SDK is:  /opt/poky/1.5
    
6. Run this script on your workstation to extract all the libraries and headers required to build OpenGL ES apps:

     cd Yocto/fsl-community-bsp/downloads
     ./gpu-viv-bin-mx6q-3.10.9-1.0.0-hfp.bin

7.  Then, copy all the Vivante libraries and headers to your new sysroot:

    sudo  cp  -r  gpu-viv-bin-mx6q-3.10.9-1.0.0-hfp/usr/include  /opt/poky/1.5/sysroots/cortexa9hf-vfp-neon-poky-linux-gnueabi/usr/.
    sudo  cp  -r  gpu-viv-bin-mx6q-3.10.9-1.0.0-hfp/usr/lib  /opt/poky/1.5/sysroots/cortexa9hf-vfp-neon-poky-linux-gnueabi/usr/.

8.  Get the source code to the Vivante sample apps from the gpu_sdk_v1.00.tar.gz package, which you can download from:

     http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6DL&fpsp=1&tab=Design_Tools_Tab

9.  Initialize the environment variables to use the new toolchain:

    source /opt/poky/1.5/environment-setup-cortexa9hf-vfp-neon-poky-linux-gnueabi

10. Edit these lines in each of the Makefile.fbdev files for the samples you want to build, so that the new toolchain and hard-float will be used:

    # Make command to use for dependencies
    # CC = $(CROSS_COMPILE)gcc
    # AR = $(CROSS_COMPILE)ar
    ROOTFS = $(OECORE_TARGET_SYSROOT)

    # CFLAGS = -DDEBUG -D_DEBUG -D_GNU_SOURCE -mfloat-abi=softfp -mfpu=neon -fPIC -O3 -fno-strict-aliasing -fno-optimize-sibling-calls -Wall -g

11: Run make to build a sample:

      cd  gpu_sdk_v1.00\Samples\GLES1.1\01_SimpleTriangle
      make Makefile.fbdev

12.  Finally, copy the built executables to your Dora card to run them on your WandBoard!

