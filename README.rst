=============================
 U-boot / ATF build manifest
=============================

The various branches available here will configure the repo build for the
appropriate branches in each repository and clone them into the working
directory.  As of now, there is no build script yet, and the required
environment settings are generally unique to each board.  If this is not
the main branch, see below for the manual build commands.

See the `arm64-multiplatform`_ repo for the mainline kernel branches tested,
and the pine64 page on the `Sunxi wiki`_ for board info.

.. _Sunxi wiki: https://linux-sunxi.org/Pine64
.. _arm64-multiplatform: https://github.com/sarnold/arm64-multiplatform

Select the build branch for your board using the github branch button above,
which will select the correct branches and repos for the board/SoC family.

Follow the steps in the readme for the branch you select, then use the same branch
name with the "repo init" command.  This will checkout the
corresponding branch HEAD commits on the configured branch for each repository.

You can get status and more using the ``repo`` command in the top level directory
once you've run the ``repo init`` and ``repo sync`` commands.  Try::

  $ repo info
  $ repo show
  $ repo status

See the default.xml file for repo and branch details.

Install the build requirements for Allwinner A64/H5/H6 arm64 SoCs,
including pine64-plus.  On Ubuntu focal, it would be something like this:

* aarch64 cross toolchain: ``apt-get install gcc-8-aarch64-linux-gnu flex bison build-essential``
* arm cortex-M cross toolchain: ``apt-get install gcc-arm-none-eabi``
* or1k cross-toolchain: ``wget https://musl.cc/or1k-linux-musl-cross.tgz``

On Gentoo, install crossdev and setup the crossdev overlay, then build all
three cross toolchains::

  # crossdev --ov-output /usr/local/crossdev -t aarch64-unknown-linux-gnu
  # crossdev --ov-output /usr/local/crossdev -t arm-none-eabi
  # crossdev --ov-output /usr/local/crossdev -t or1k-linux-musl


Install the repo utility
------------------------

::

  $ mkdir ~/bin
  $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
  $ chmod a+x ~/bin/repo

Download the firmware source
----------------------------

::

  $ PATH=${PATH}:~/bin
  $ mkdir allwinner-build
  $ cd allwinner-build
  $ repo init -u https://github.com/sarnold/u-boot-ATF-manifest.git -b allwinner-a64
  $ repo sync

If you need to download the ``ork1`` cross toolchain above, then from
the same directory, also do::

  $ wget https://musl.cc/or1k-linux-musl-cross.tgz
  $ tar xf or1k-linux-musl-cross.tgz
  $ export PATH=$PWD/or1k-linux-musl-cross/bin:$PATH

If you used crossdev, then skip the above.


Pine64 Example
--------------

Build a full bootloader binary to boot pine64/pine64-plus from sdcard.


From the same directory after ``repo sync`` above::

  $ make -C arm-trusted-firmware M0_CROSS_COMPILE=armv7m-hardened-eabi- CROSS_COMPILE=aarch64-unknown-linux-gnu- PLAT=sun50i_a64
  $ make -C crust CROSS_COMPILE=or1k-linux-musl- pine64_plus_defconfig
  $ make -C crust CROSS_COMPILE=or1k-linux-musl- scp
  $ make -C u-boot CROSS_COMPILE=aarch64-linux-gnu- pine64_plus_defconfig
  $ make -C u-boot BL31=../arm-trusted-firmware/build/sun50i_a64/release/bl31.bin SCP=../crust/build/scp/scp.bin CROSS_COMPILE=aarch64-unknown-linux-gnu-

.. note:: Substitute your own CROSS_COMPILE triplets in the above commands.
          The exact names will vary depending on your build host and where
          your toolchains come from.

The result of a successful build can be found when complete::

  $ ls -l u-boot/u-boot-sunxi-with-spl.*
  -rw-r--r-- 1 user user 776655 Mar 23 21:40 u-boot/u-boot-sunxi-with-spl.bin

There are several u-boot build products, but the above file is the combined
``u-boot.itb`` plus SPL/SCP firmware that's required for sdcard/emmc booting.

The u-boot build/deployment steps in general are specific for each SoC/board;
for pine64 they are included in the u-boot board-specific doc `README.sunxi64`_.
For this example, use the above ``u-boot-sunxi-with-spl.bin`` file and ``dd``
it to a blank sdcard with the args ``bs=8k seek=1``.

.. warning:: Be sure to use the correct disk target for the ``dd`` command!
             See the following example.

To check your sdcard device, run the following command from a console prompt::

  $ lsblk 
  NAME                                       MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
  sda                                          8:0    0 232.9G  0 disk  
  ├─sda1                                       8:1    0   512M  0 part  
  └─sda2                                       8:2    0 232.4G  0 part  
    └─root_c5c14ddd-1afd-40ec-bb34-b9b19d8a47bb-vg1-root
                                             253:0    0 232.4G  0 crypt 
      ├─vg1-swap                             253:1    0    10G  0 lvm   [SWAP]
      ├─vg1-root                             253:2    0    50G  0 lvm   /
      └─vg1-home                             253:3    0 163.8G  0 lvm   /home
  sdb                                          8:16   1  14.8G  0 disk  
  └─sdb1                                       8:17   1   1.3G  0 part  

In the above output there is one hard disk and one USB sdcard reader, so
the correct disk target for ``dd`` would be ``/dev/sdb``.

The general (manual) process for building/deploying u-boot for Rock-pi 4
onto an sdcard is documented on the `Linuxonarm wiki`_.  You can still
follow the sdcard creation steps documented there and use the u-boot
binary you built above.


.. _README.sunxi64: https://source.denx.de/u-boot/u-boot/blob/master/board/sunxi/README.sunxi64
.. _Linuxonarm wiki: https://www.digikey.com/eewiki/display/linuxonarm/ROCK+Pi+4

