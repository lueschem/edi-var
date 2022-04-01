# edi Project Configuration for Variscite Boards

This [edi](https://www.get-edi.io) project configuration currently supports the
[Variscite var-som-mx8m-nano](https://www.variscite.com/product/system-on-module-som/cortex-a53-krait/var-som-mx8m-nano-nxp-i-mx-8m-nano/)
on the [Symphony-Board](https://www.variscite.com/product/single-board-computers/symphony-board/).

<img alt="Compulab iot-gate-imx8" src=https://www.get-edi.io/assets/images/blog/VAR-SOM-MX8M-NANO.PNG width="75%"/>


## Introduction

The edi configuration contained in this repository can be used to
generate the following artifacts:

* A **minimal** (e.g. no display, no sound) Debian bullseye arm64 (64bit) image suitable for the Variscite var-som-mx8m-nano.
* A matching Mender update artifacts for the above configuration.
* An amd64/arm64 based LXD container with a pre-installed
cross development toolchain for C and C++.

## Important Note

Please note that image generation operations require superuser privileges
and therefore you can easily break your host operating system. Therefore
make sure that you have a backup copy of your data.

## Basic Usage

### Preparation

Prior to using this edi project configuration you have to install
[edi](https://www.get-edi.io) according to
[this instructions](https://docs.get-edi.io/en/latest/getting_started.html).
Please take a careful look at the "Setting up ssh Keys" section since you
will need a proper ssh key setup in order to access the container or
the target device using ssh.

The image post processing commands require some additional tools. On
Ubuntu 20.04 those tools can be installed as follows:

``` bash
sudo apt install e2fsprogs bmap-tools mtools parted rsync zerofree python3-sphinx mender-artifact
```

### Creating a Target Image

A target image can be created using the following command:

``` bash
sudo edi -v image create var-som-mx8m-nano-bullseye-arm64.yml
```

The resulting image can be copied to a SD card (here /dev/mmcblk0)
using the following command
(**Please note that everything on the SD card will be erased!**):

``` bash
sudo bmaptool copy artifacts/var-som-mx8m-nano-bullseye-arm64.img /dev/mmcblk0
```

If the command fails, unmount the SD card and repeat the above command.

Once you have booted the device using this SD card you can
access it using ssh (the access should be granted thanks to to your
ssh keys):

``` bash
ssh variscite@IP_ADDRESS
```

The password for the user _variscite_ is _variscite_ (just in case you want to
execute a command using `sudo` or login via a local terminal).

### Flashing the Image to the eMMC

The same image that has been used for the SD card can also be flashed to the builtin eMMC as follows:

Copy the image to the device that has been booted from the SD card:

``` bash
scp artifacts/var-som-mx8m-nano-bullseye-arm64.img variscite@IP_ADDRESS:
```

Access the device:

``` bash
ssh variscite@IP_ADDRESS
```

Flash the image to the eMMC (**Everything on mmcblk2 will be erased!**):

``` bash
sudo dd if=var-som-mx8m-nano-bullseye-arm64.img of=/dev/mmcblk2 bs=1M
```

Now you can switch the device off and toggle the "BOOT SELECT" switch from "SD" to "INT".
When powering up the device again, it should boot the new image from the eMMC storage device.

### Connecting to Mender

To enable over the air (OTA) updates, the generated images are configured
to connect to [https://hosted.mender.io/](https://hosted.mender.io/).
In order to connect to your Mender tenant you have to provide your tenant token prior to building the images.
The tenant token can be added to `configuration/mender/mender.yml`. If you do not want to
add the tenant token to the version control system you can also copy `configuration/mender/mender.yml` to
`configuration/mender/mender_custom.yml` and add the tenant token there.

### Creating a Cross Development LXD Container

A cross development container can be created using the
following command:

``` bash
sudo edi -v lxc configure var-som-mx8m-nano-bullseye-arm64-cross-dev var-som-mx8m-nano-bullseye-arm64-cross-dev.yml
```

The container can be accessed as follows (the password is _ChangeMe!_):

``` bash
lxc exec var-som-mx8m-nano-bullseye-arm64-cross-dev -- login ${USER}
```

Or with ssh (Hint: retrieve IP_OF_CONTAINER with `lxc list`):

``` bash
ssh IP_OF_CONTAINER
```

You can directly start to cross compile applications:


``` bash
aarch64-linux-gnu-g++ ...
```

For your convenience, the LXD container shares the folder _edi-workspace_
with the host operating system.

## Documenting an Artifact

During the image build the documentation gets rendered to artifacts/CONFIGNAME_documentation
as reStructuredText. The text files can be transformed into a nice pdf file with some
additional tools that need to be installed first:

``` bash
sudo apt install texlive-latex-recommended texlive-pictures texlive-latex-extra texlive-xetex latexmk
```

Then the pdf can be generated using the following commands:

``` bash
cd artifacts/CONFIGNAME_documentation
make PDFLATEX=xelatex latexpdf
make PDFLATEX=xelatex latexpdf
```

### More Information

For more information about the Variscite hardware please take a look at the
[official documentation](https://variwiki.com/index.php?title=Debian_Build_Release&release=RELEASE_BULLSEYE_V1.0_VAR-SOM-MX8M-NANO).

For more information about this setup please read the [edi documentation](https://docs.get-edi.io) and
[this blog post](https://www.get-edi.io/A-new-Approach-to-Operating-System-Image-Generation/).

For details about the Mender based robust update integration please refer to this
[blog post](https://www.get-edi.io/Updating-a-Debian-Based-IoT-Fleet/).

If you are curious about the U-Boot bootloader setup please take a look at this
[blog post](https://www.get-edi.io/Booting-Debian-with-U-Boot/).

For the kernel build instructions please check the [docs folder of this project](docs/kernel_build.md).
