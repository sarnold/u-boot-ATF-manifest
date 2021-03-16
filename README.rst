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


Note about DDR_TOPOLOGY
-----------------------

For Armada37x0 only, the default DDR topology map index/name is 0.  You
will need to check your board for the version stencil and RAM size/chip
select.  For example, if your board stencil says ``ESPRESSOBin_V5_0_1``
and you have 2 RAM chips (opposite each other, one on each side of the
board) then you would use "2" for ``DDR3 2CS 1GB (EspressoBin V3-V5)``.

Supported Options:

* 0 - DDR3 1CS 512MB (DB-88F3720-DDR3-Modular, EspressoBin V3-V5)
* 1 - DDR4 1CS 512MB (DB-88F3720-DDR4-Modular)
* 2 - DDR3 2CS 1GB (EspressoBin V3-V5)
* 3 - DDR4 2CS 4GB (DB-88F3720-DDR4-Modular)
* 4 - DDR3 1CS 1GB (DB-88F3720-DDR3-Modular, EspressoBin V3-V5)
* 5 - DDR4 1CS 1GB (EspressoBin V7, EspressoBin-Ultra)
* 6 - DDR4 2CS 2GB (EspressoBin V7)
* 7 - DDR3 2CS 2GB (EspressoBin V3-V5)
* CUST - CUSTOMER BOARD (Customer board settings)


Mainline u-boot (EspressoBin V5) output:

::

    TIM-1.0
    mv_ddr-devel-g7c35173 DDR3 16b 1GB 2CS
    WTMI-devel-18.12.1-c444aeb
    WTMI: system early-init
    SVC REV: 3, CPU VDD voltage: 1.143V
    Setting clocks: CPU 1000 MHz, DDR 800 MHz
    NOTICE:  Booting Trusted Firmware
    NOTICE:  BL1: v2.4(release):v2.4-445-ga8fb76e59 (Marvell-devel-18.12.2)
    NOTICE:  BL1: Built : 20:30:32, Mar 14 2021
    NOTICE:  BL1: Booting BL2
    NOTICE:  BL2: v2.4(release):v2.4-445-ga8fb76e59 (Marvell-devel-18.12.2)
    NOTICE:  BL2: Built : 20:30:32, Mar 14 2021
    NOTICE:  BL1: Booting BL31
    NOTICE:  BL31: v2.4(release):v2.4-445-ga8fb76e59 (Marvell-devel-18.12.2)
    NOTICE:  BL31: Built : 20:30:32, Mar 14 2021


    U-Boot 2021.04-rc3-00166-gad7e1c7c6e (Mar 14 2021 - 19:58:47 -0700)

    DRAM:  1 GiB
    Comphy-0: USB3_HOST0    5 Gbps
    Comphy-1: PEX0          2.5 Gbps
    Comphy-2: SATA0         5 Gbps
    SATA link 0 timeout.
    AHCI 0001.0300 32 slots 1 ports 6 Gbps 0x1 impl SATA mode
    flags: ncq led only pmp fbss pio slum part sxs
    PCIE-0: Link down
    MMC:   sdhci@d0000: 0, sdhci@d8000: 1
    Loading Environment from SPIFlash... SF: Detected w25q32dw with page size 256 Bytes, erase size 4 KiB, total 4 MiB
    OK
    Model: Globalscale Marvell ESPRESSOBin Board
    Card did not respond to voltage select! : -110
    Net:   eth0: neta@30000
    Hit any key to stop autoboot:  0
    =>
