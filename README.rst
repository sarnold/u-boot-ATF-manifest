=============================
 U-boot / ATF build manifest
=============================

The various branches available here will configure the repo build for the
appropriate branches in each repository and clone them into the working
directory.  As of now, there is no build script yet, and the required
environment settings are generally unique to each board.  If this is not
the main branch, see below for the manual build commands.

See the `arm64-multiplatform`_ repo for the mainline kernel branches tested,
and the espessobin page on the `readthedocs pages`_ for board info.

.. _readthedocs pages: https://roc-rk3328-cc.readthedocs.io/en/latest/index.html
.. _arm64-multiplatform: https://github.com/sarnold/arm64-multiplatform

Select the build branch for your board using the github branch button above,
which will select the correct branches and repos for the board/SoC family.

Install the minimum build requirements for Rockchip rk33xx/rk35xx SoCs,
including roc-rk3328-cc.  On Ubuntu focal, it would be something like this:

* aarch64 cross toolchain: ``apt-get install gcc-8-aarch64-linux-gnu flex bison git build-essential``

For Debian bullseye/bookworm install the same packages as above but with
a newer version of gcc.

For Gentoo, install the tool deps and then build with crossdev::

  # emerge --ask --update dev-vcs/git sys-apps/dtc dev-lang/swig dev-python/pyelftools sys-devel/crossdev dev-python/tox
  # crossdev --target aarch64-unknown-linux-gnu


Repolite method
===============

Uses the YAML config: ``.repolite.yml`` and the ``tox.ini`` file. We also
follow the latest configuration in the `TFA platform`_ and `u-boot` docs
in the steps below.

.. _TFA platform: https://trustedfirmware-a.readthedocs.io/en/v2.10/plat/rockchip.html#
.. _: https://docs.u-boot.org/en/v2023.10/board/rockchip/rockchip.html#building

Tox
---

As long as you have git and at least Python 3.6, then you can install and
use `tox`_.  After cloning the repository, you can run ``repolite --sync``
via the ``tox`` command.  It will build a python virtual environment with
all the dependencies and run the specified commands, eg:

::

  $ git clone https://github.com/sarnold/u-boot-ATF-manifest
  $ cd u-boot-ATF-manifest/
  $ git checkout marvell-armada
  $ tox -e sync       # install deps, clone/checkout repositories
  $ ls ext/
  tfa  u-boot

To replicate the v2.10.0 build shown below, run the following tox commmands::

  $ tox -e update
  $ tox -e build

The above build command has CPUS=4 as the default, and can be changed via
an environment variable::

  $ CPUS=8 tox -e build

Eventually, the console display should look something like this:

::

    Success: NTIM Processing has completed successfully!

  Finish time: 10/30/23 19:08:11

  TBB Exiting...!
  No input file for TIMN is supplied
  Total number of images to process in file[0] - 3
  0 Image at offset 00000000 is TIM_ATF.bin
  1 Image at offset 00004000 is wtmi.bin
  2 Image at offset 00015000 is boot-image.bin
  Total number of images 3

  Built /home/user/u-boot-ATF-manifest/ext/tfa/build/a3700/release/flash-image.bin successfully

  make: Leaving directory '/home/user/u-boot-ATF-manifest/ext/tfa'
  build: commands[3] /home/user/u-boot-ATF-manifest/ext> bash -c 'cp -v tfa/build/a3700/release/flash-image.bin ../'
  'tfa/build/a3700/release/flash-image.bin' -> '../flash-image.bin'
    update: OK (0.23=setup[0.03]+cmd[0.20] seconds)
    build: OK (262.54=setup[0.01]+cmd[62.42,1.62,198.49,0.00] seconds)
    congratulations :) (262.83 seconds)


Manual build
------------

The manual commands should be run from the repolite_ "vendor" directory, by default
named ``ext/``, while substituting for appropriate DDR and model type. The commands
given below are for the original roc-rk3328-cc-v5 with iGB of DDR3 RAM chips and no
emmc flash => ``DDR3 2CS 1GB`` as shown in the u-boot console output below.  Note
that all of the espressobin variants should use ``PLAT=a3700`` for TFA.

::

  $ cd ext/
  $ make -C u-boot CROSS_COMPILE=aarch64-unknown-linux-gnu- mvebu_espressobin-88f3720_defconfig u-boot.bin
  $ make -C mox-boot CROSS_CM3=arm-none-eabi- wtmi_app.bin
  $ make -C tfa CROSS_COMPILE=aarch64-unknown-linux-gnu- CROSS_CM3=arm-none-eabi- USE_COHERENT_MEM=0 PLAT=a3700 CLOCKSPRESET=CPU_1000_DDR_800 DDR_TOPOLOGY=2 MV_DDR_PATH=$PWD/mv-ddr WTP=$PWD/mv-utils CRYPTOPP_PATH=$PWD/cryptopp/ BL33=$PWD/u-boot/u-boot.bin WTMI_IMG=$PWD/mox-boot/wtmi_app.bin FIP_ALIGN=0x100 mrvl_flash -j8
  $ cp tfa/build/a3700/release/flash-image.bin ../flash-image-1g-2cs.bin
  $ cd -

.. _tox: https://github.com/tox-dev/tox
.. _repolite: https://sarnold.github.io/repolite/

Miscellaneous
=============

Mainly platform-specific notes.

Note about u-boot and UEFI boot
-------------------------------

The latest 2022 release of u-boot can boot the latest arm64 installers from
major Linux distros, eg, the Debian arm64 mini.iso or the Gentoo arm64 minimal
installer ISO, as long as the target board has current distroboot support.

Most devices will need to boot from USB or TFTP; both options are supported
by Debian, Gentoo, Arch, etc, however, "generic" installers assume a "normal"
single/default ethernet device.

For now, see ebbr-on-espressobin_ for more info.

.. _ebbr-on-espressobin: https://marcin.juszkiewicz.com.pl/2021/02/15/ebbr-on-espressobin/


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
    mv_ddr-devel-g2b37d92 DDR3 16b 1GB 2CS
    WTMI-devel-18.12.1-a3e1c67
    WTMI: system early-init
    CPU VDD voltage default value: 1.155V
    Setting clocks: CPU 1000 MHz, DDR 800 MHz
    CZ.NIC's Armada 3720 Secure Firmware v2022.06.11 (Oct 24 2023 20:33:51)
    Running on ESPRESSObin
    NOTICE:  Booting Trusted Firmware
    NOTICE:  BL1: lts-v2.8.9(release):lts-v2.8.9
    NOTICE:  BL1: Built : 20:35:08, Oct 24 2023
    NOTICE:  BL1: Booting BL2
    NOTICE:  BL2: lts-v2.8.9(release):lts-v2.8.9
    NOTICE:  BL2: Built : 20:35:08, Oct 24 2023
    NOTICE:  BL1: Booting BL31
    NOTICE:  BL31: lts-v2.8.9(release):lts-v2.8.9
    NOTICE:  BL31: Built : 20:35:08, Oct 24 2023


    U-Boot 2022.10 (Oct 24 2023 - 20:32:43 -0700)

    DRAM:  1 GiB
    Core:  47 devices, 24 uclasses, devicetree: separate
    WDT:   Not starting watchdog@8300
    Comphy chip #0:
    Comphy-0: USB3_HOST0    5 Gbps
    Comphy-1: PEX0          5 Gbps
    Comphy-2: SATA0         6 Gbps
    SATA link 0 timeout.
    AHCI 0001.0300 32 slots 1 ports 6 Gbps 0x1 impl SATA mode
    flags: ncq led only pmp fbss pio slum part sxs
    PCIe: Link down
    MMC:   sdhci@d0000: 0, sdhci@d8000: 1
    Loading Environment from SPIFlash... SF: Detected w25q32dw with page size 256 Bytes, erase size 4 KiB, total 4 MiB
    OK
    Model: Globalscale Marvell ESPRESSOBin Board
    Net:   eth0: ethernet@30000
    Hit any key to stop autoboot:  0
    =>
