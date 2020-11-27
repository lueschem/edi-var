# Kernel Build Instructions

The following instructions are derived from the
[Variscite setup](https://variwiki.com/index.php?title=Debian_Build_Release&release=RELEASE_BULLSEYE_V1.0_VAR-SOM-MX8M-NANO)
and adapted so that a Debian package gets built.


First we enter the cross development LXD container (for more details please check the README.md of this project):

``` bash
ssh IP_OF_CROSS_DEV_CONTAINER
```

For the kernel build we install some additional Debian packages within the development container:

``` bash
sudo apt install build-essential bc kmod cpio flex cpio libncurses5-dev bison libssl-dev wget lzop rsync
```

Then we clone the Variscite linux-imx kernel:

``` bash
git clone https://github.com/varigit/linux-imx.git
cd linux-imx
git checkout imx_5.4.24_2.1.0_var01
```

Finally we configure and build the kernel:

``` bash
export DTB="freescale/imx8mn-var-som.dtb freescale/imx8mn-var-som-m7.dtb freescale/imx8mn-var-som-rev10.dtb freescale/imx8mn-var-som-rev10-m7.dtb"
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- imx8_var_defconfig
make -j $(nproc) KBUILD_IMAGE=arch/arm64/boot/Image ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- ${DTB} deb-pkg
```

The resulting Debian package got uploaded to [this packagecloud repository](https://packagecloud.io/get-edi/debian/).
