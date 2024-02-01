=============================
 U-boot / ATF build manifest
=============================

The various branches available here will configure the repo build for the
appropriate branches in each repository and clone them into the working
directory.  As of now, there is no build script yet, and the required
environment settings are generally unique to each board.  If this is not
the main branch, see below for the manual build commands.

See the `arm64-multiplatform`_ repo for some of the mainline kernel branches
tested.

.. _arm64-multiplatform: https://github.com/sarnold/arm64-multiplatform

Select the build branch for your board using the github branch button above,
which will select the correct branches and repos for the board/SoC family.

Install the base build requirements for your board.  On Ubuntu focal, it
would be something like this:

* aarch64 cross toolchain: ``apt-get install gcc-8-aarch64-linux-gnu flex bison build-essential``
* arm cortex-M cross toolchain: ``apt-get install gcc-arm-none-eabi``
* <additional board-specific dependencies>

If this branch has a manifest file (default.xml)
================================================

Follow the steps in the readme for the branch you select, then use the same branch
name with the "repo init" command.  This will checkout the corresponding
branch HEAD commits on the configured branch for each repository.

You can get status and more using the ``repo`` command in the top level directory
once you've run the ``repo init`` and ``repo sync`` commands.  Try::

  $ repo info
  $ repo show
  $ repo status

See the default.xml file for repo and branch details.

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
  $ mkdir <your_board_name>-build
  $ cd <your_board_name>-build
  $ repo init -u https://github.com/sarnold/u-boot-ATF-manifest.git -b <board_branch>
  $ repo sync


If this branch has a Tox file
=============================

Uses the YAML config ``.repolite.yml`` to manage git repos and the
``tox.ini`` file to setup the env and execute commands. We also follow
the latest configuration in the `TFA platform`_ and `u-boot`_ docs in
the steps below.

.. note:: The TFA support is not yet integrated with this build.

.. _TFA platform: https://trustedfirmware-a.readthedocs.io/en/v2.10/plat/rockchip.html#
.. _u-boot: https://docs.u-boot.org/en/v2023.10/board/rockchip/rockchip.html#building

As long as you have git and at least Python 3.6, then you can install and
use `tox`_.  After cloning the repository, you can run ``repolite --sync``
via the ``tox`` command.  It will build a python virtual environment with
all the python dependencies and run the specified commands, eg:

::

  $ git clone https://github.com/sarnold/u-boot-ATF-manifest
  $ cd u-boot-ATF-manifest/
  $ git checkout rpi64
  $ tox -e sync       # install deps, clone/checkout repositories
  $ ls ext/
  u-boot  firmware

To build u-boot for 64bit rpi variants, run the following tox commmand::

  $ tox -e uboot

The above build command has CPUS=4 as the default, and can be changed via
an environment variable::

  $ CPUS=8 tox -e uboot

Eventually, the console display should look something like this:

::

    COPY    u-boot.bin
    DTC     arch/arm/dts/bcm2835-rpi-a.dtb
    DTC     arch/arm/dts/bcm2835-rpi-a-plus.dtb
    DTC     arch/arm/dts/bcm2835-rpi-b.dtb
    DTC     arch/arm/dts/bcm2835-rpi-b-plus.dtb
    DTC     arch/arm/dts/bcm2835-rpi-b-rev2.dtb
    DTC     arch/arm/dts/bcm2835-rpi-cm1-io1.dtb
    DTC     arch/arm/dts/bcm2835-rpi-zero.dtb
    DTC     arch/arm/dts/bcm2835-rpi-zero-w.dtb
    DTC     arch/arm/dts/bcm2836-rpi-2-b.dtb
    DTC     arch/arm/dts/bcm2837-rpi-3-a-plus.dtb
    DTC     arch/arm/dts/bcm2837-rpi-3-b.dtb
    DTC     arch/arm/dts/bcm2837-rpi-3-b-plus.dtb
    DTC     arch/arm/dts/bcm2837-rpi-cm3-io3.dtb
    DTC     arch/arm/dts/bcm2711-rpi-4-b.dtb
    SHIPPED dts/dt.dtb
    OFCHK   .config
  make: Leaving directory '/home/nerdboy/my_stuff/home/hardware/u-boot-ATF-manifest/ext/u-boot'
  uboot: commands[2]> bash -c 'cp -v ext/u-boot/u-boot.bin .'
  'ext/u-boot/u-boot.bin' -> './u-boot.bin'
    uboot: OK (20.54=setup[0.03]+cmd[2.28,18.22,0.00] seconds)
    congratulations :) (20.59 seconds)


Works with mainline u-boot
--------------------------

* Marvell Armada SoCs (espressobin variants tested)
* Allwinner A64/H5/H6 SoCs (pine64 variants tested)
* Rockchip ROCK-Pi-4, nanopi-r5c, roc-rk3328-cc
* Raspberrypi 64-bit models


References
----------

* `ARM Trusted Firmware docs`_
* `U-Boot docs`_
* `Rpi U-boot`_


.. _ARM Trusted Firmware docs: https://trustedfirmware-a.readthedocs.io/en/latest/
.. _U-Boot docs: https://u-boot.readthedocs.io/en/latest/
.. _Rpi U-boot: https://elinux.org/RPi_U-Boot
