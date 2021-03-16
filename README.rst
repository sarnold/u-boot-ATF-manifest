=============================
 U-boot / ATF build manifest
=============================

The various branches available here will configure the repo build for the
appropriate branches in each repository and clone them into the working
directory.  As of now, there is no build script yet, and the required
environment settings are generally unique to each board.  If this is not
the main branch, see below for the manual build commands.

See the `arm64-multiplatform`_ repo for the mainline kernel branches tested.

.. _arm64-multiplatform: https://github.com/sarnold/arm64-multiplatform

Select the build branch for your board using the github branch button above,
which will select the correct branches and repos for the board/SoC family.

Follow the steps in the readme for the branch you select, then use the same branch
name with the "repo init" command.  This will checkout the corresponding
branch HEAD commits on the configured branch for each repository.

You can get status and more using the ``repo`` command in the top level directory
once you've run the ``repo init`` and ``repo sync`` commands.  Try::

  $ repo info
  $ repo show
  $ repo status

See the default.xml file for repo and branch details.

Install the base build requirements for your board.  On Ubuntu focal, it
would be something like this:

* aarch64 cross toolchain: ``apt-get install gcc-8-aarch64-linux-gnu flex bison build-essential``
* arm cortex-M cross toolchain: ``apt-get install gcc-arm-none-eabi``
* <additional board-specific dependencies>


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



Works with mainline u-boot
--------------------------

* Marvell Armada SoCs
* Rockchip ROCK-Pi-4


References
----------

**`ARM Trusted Firmware docs`_**
**`U-Boot docs`_**



.. _ARM Trusted Firmware docs: https://trustedfirmware-a.readthedocs.io/en/latest/
.. _U-Boot docs: https://u-boot.readthedocs.io/en/latest/
