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

  # emerge --ask --update dev-vcs/git sys-apps/dtc dev-lang/swig sys-devel/crossdev dev-python/tox
  # crossdev --target aarch64-unknown-linux-gnu


.. note:: If not using the Tox_ workflow, you must also install `pyelfutils`
          using your OS package manager, eg, `apt` or `emerge`, something
          like the following:
          
           `emerge dev-python/pyelftools`


Repo tool method
================

Uses the repo manifest file: ``default.xml``.

Install the repo utility
------------------------

::

  $ mkdir ~/bin
  $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
  $ chmod a+x ~/bin/repo

Download the firmware source
----------------------------

Use the branch name selected previously in the `repo init` command below:

::

  $ PATH=${PATH}:~/bin
  $ mkdir rockchip-rk3328-build
  $ cd rockchip-rk3328-build
  $ repo init -u https://github.com/sarnold/u-boot-ATF-manifest.git -b rockchip-libre
  $ repo sync


**Example**: build a (rockchip) bootloader image for the Libre ``roc-rk3328-cc``


From the same directory after ``repo sync`` above::

  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- PLAT=rk3328 -C ext/tfa
  $ export BL31=`pwd`/ext/tfa/build/rk3328/release/bl31/bl31.elf
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- -C ext/u-boot/ roc-cc-rk3328_defconfig
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- -C ext/u-boot/ -j8

The result of a successful build will leave a combined boot image in the
u-boot source directory::

  $ ls -l u-boot/u-boot-rockchip.bin
  -rw-r--r-- 1 user user 9239552 Dec 29 21:08 u-boot/u-boot-rockchip.bin


Repolite method
===============

Uses the YAML config: ``.repolite.yml`` and the ``tox.ini`` file. We also
follow the latest configuration in the `TFA platform`_ and `u-boot` docs
in the steps below.

.. _TFA platform: https://trustedfirmware-a.readthedocs.io/en/v2.10/plat/rockchip.html#
.. _u-boot: https://docs.u-boot.org/en/v2023.10/board/rockchip/rockchip.html#building

Tox
---

As long as you have git and at least Python 3.6, then you can install and
use `tox`_.  After cloning the repository, you can run ``repolite --sync``
via the ``tox`` command.  It will build a python virtual environment with
all the python dependencies and run the specified commands, eg:

::

  $ git clone https://github.com/sarnold/u-boot-ATF-manifest
  $ cd u-boot-ATF-manifest/
  $ git checkout rockchip-libre
  $ tox -e sync       # install deps, clone/checkout repositories
  $ ls ext/
  tfa  u-boot  rkbin

To replicate a v2.10.0 build, run the following tox commmands::

  $ tox -e update
  $ tox -e tfa,uboot

The above build command has CPUS=4 as the default, and can be changed via
an environment variable::

  $ CPUS=8 tox -e tfa,uboot

Eventually, the console display should look something like this:

::

  ...
    LD      tpl/u-boot-tpl
    OBJCOPY tpl/u-boot-tpl-nodtb.bin
    SYM     tpl/u-boot-tpl.sym
    COPY    tpl/u-boot-tpl.bin
    BINMAN  .binman_stamp
  Image 'simple-bin' is missing optional external blobs but is still functional: tee-os

  /binman/simple-bin/fit/images/@tee-SEQ/tee-os (tee-os):
     See the documentation for your board. You may need to build Open Portable
     Trusted Execution Environment (OP-TEE) and build with TEE=/path/to/tee.bin

    OFCHK   .config
  make: Leaving directory '/home/nerdboy/my_stuff/home/hardware/u-boot-ATF-manifest/ext/u-boot'
  uboot: commands[2]> bash -c 'cp -v ext/u-boot/u-boot-rockchip.bin ../'
  'ext/u-boot/u-boot-rockchip.bin' -> 'u-boot-rockchip.bin'
    uboot: OK (6.39=setup[1.74]+cmd[0.28,4.36,0.01] seconds)
    congratulations :) (6.44 seconds)


Manual build
------------

The manual commands should be run from the top-level repo directory, while
substituting for appropriate platform and machine/model config. The commands
given below are for the typical roc-rk3328-cc with 2GB of DDR4 RAM chips and no
eMMC flash.  Note that all of the rk3328 variants should use ``PLAT=rk3328`` for
TFA.

::

  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- distclean -C ext/tfa
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- PLAT=rk3328 -C ext/tfa
  $ export BL31=`pwd`/ext/tfa/build/rk3328/release/bl31/bl31.elf
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- -C ext/u-boot/ distclean
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- -C ext/u-boot/ roc-cc-rk3328_defconfig
  $ make CROSS_COMPILE=aarch64-unknown-linux-gnu- -C ext/u-boot/ -j8
  $ cp -v ext/u-boot/u-boot-rockchip.bin .

.. _tox: https://github.com/tox-dev/tox
.. _repolite: https://sarnold.github.io/repolite/

Miscellaneous
=============

Mainly platform-specific notes for Rockchip/Firefly/Libre Computer.

* 

Note about u-boot and UEFI boot
-------------------------------

As of at least the v2022.10 release, u-boot can boot the latest arm64 installers from
major Linux distros, eg, the Debian arm64 mini.iso or the Gentoo arm64 minimal
installer ISO, as long as the target board has current distroboot support.

Most devices will need to boot from USB or TFTP; both options are supported
by Debian, Gentoo, Arch, etc, however, "generic" installers assume a "normal"
single/default ethernet device.

That said, at least some Rockchip boards have a very minimal default environment
in (mainline) u-boot to support the vendor-style flash storage layout with
7 different partitions. Out-of-the-box support seems limited to that or the
legacy extlinux bootflow, as it gets confused by ESP plus a root partition.

On the flip side, *some* Rockchip boards have upstream support for EDK2, so
can be installed with fully functional UEFI firmware support.  There is also
at least one github project that adds EDK2 support to some of the pine64
boards, eg, Quartz64. See the `quartz64_uefi project repo`_ for more details.

.. _quartz64_uefi project repo: https://github.com/jaredmcneill/quartz64_uefi

Mainline u-boot (roc-rk3328-cc) output:

::

  U-Boot TPL 2024.01-rc5-00011-g0a0ceea226 (Dec 28 2023 - 22:53:45)
  DDR4, 333MHz
  BW=32 Col=10 Bk=4 BG=2 CS0 Row=16 CS=1 Die BW=16 Size=2048MB
  Trying to boot from BOOTROM
  Returning to boot ROM...

  U-Boot SPL 2024.01-rc5-00011-g0a0ceea226 (Dec 28 2023 - 22:53:45 -0800)
  Trying to boot from MMC1
  NOTICE:  BL31: v2.10.0  (release):v2.10.0-136-g555126491
  NOTICE:  BL31: Built : 22:40:23, Dec 28 2023
  NOTICE:  BL31:Rockchip release version: v1.2


  U-Boot 2024.01-rc5-00011-g0a0ceea226 (Dec 28 2023 - 22:53:45 -0800)

  Model: Firefly roc-rk3328-cc
  DRAM:  2 GiB
  PMIC:  RK8050 (on=0x40, off=0x00)
  Core:  233 devices, 24 uclasses, devicetree: separate
  MMC:   mmc@ff500000: 1, mmc@ff520000: 0
  Loading Environment from MMC... *** Warning - bad CRC, using default environment

  In:    serial@ff130000
  Out:   serial@ff130000
  Err:   serial@ff130000
  Model: Firefly roc-rk3328-cc
  Net:   eth0: ethernet@ff540000
  Hit any key to stop autoboot:  0
