mx6-sdb, LVDS display:
Two LVDS:
	video=video=mxcfb0:dev=ldb,LDB-XGA,if=RGB24 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB24 ldb=sep0

Using the CLAA(?) (i.e. mx53 qsb) LVDS:

	video=mxcfb1:dev=ldb,LDB-XGA,if=RGB24 ldb=sin0

Note: I changed to LVDS1 (from LVDS2) which means the u-boot arguments need to
be adjusted.
