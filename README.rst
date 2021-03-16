=============================
 U-boot / ATF build manifest
=============================

The various branches available here will configure the repo build for the
appropriate branches in each repository and clone them into the working
directory.  As of now, there is no build script yet, and the required
environment settings are generally unique to each board.  If this is not
the main branch, see below for the manual build commands.

See the `arm64-multiplatform`_ repo for the mainline kernel branches tested,
and the espessobin page on the `Gentoo wiki`_ for board info.

.. _Gentoo wiki: https://wiki.gentoo.org/wiki/ESPRESSOBin
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

Install the build requirements for Marvell armada 3xxx/7xxx/8xxx SoCs,
including espressobin.  On Ubuntu focal, it would be something like this:

* aarch64 cross toolchain: ``apt-get install gcc-8-aarch64-linux-gnu flex bison build-essential``
* arm cortex-M cross toolchain: ``apt-get install gcc-arm-none-eabi``
* crypto libs/headers: ``apt-get install libssl-dev``


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
  $ mkdir marvell-armada-build
  $ cd marvell-armada-build
  $ repo init -u https://github.com/sarnold/u-boot-ATF-manifest.git -b marvell-armada
  $ repo sync


**Example**: build a bootloader flash image for ``espressobin-v5-1GB-ddr3``


From the same directory after ``repo sync`` above::

  $ make -C u-boot CROSS_COMPILE=aarch64-linux-gnu- mvebu_espressobin-88f3720_defconfig u-boot.bin -j3
  $ make -C trusted-firmware-a CROSS_COMPILE=aarch64-linux-gnu- CROSS_CM3=arm-none-eabi- PLAT=a3700 CLOCKSPRESET=CPU_1000_DDR_800 DDR_TOPOLOGY=2 MV_DDR_PATH=$PWD/mv-ddr-marvell/ WTP=$PWD/a3700-utils-marvell/ CRYPTOPP_PATH=$PWD/cryptopp/ BL33=$PWD/u-boot/u-boot.bin mrvl_flash -j3

The result of a successful build will be displayed when complete::

  Built /home/user/atf-build/trusted-firmware-a/build/a3700/release/flash-image.bin successfully

See the `espressobin wiki`_ for how to flash the above image.


.. _espressobin wiki: http://wiki.espressobin.net/tiki-index.php?page=Update+the+Bootloader

Works with mainline u-boot:

* Marvell Armada SoCs
* Rockchip ROCK-Pi-4

