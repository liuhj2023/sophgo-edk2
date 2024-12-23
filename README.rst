SOPHGO EDK2
###########

Build
=====

1. Install dependencies.

.. highlights::

    .. code:: sh

        sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build uuid-dev gcc-riscv64-unknown-elf

2. Clone repository (maybe you have done this).

.. highlights::

    .. code:: sh

        git clone https://github.com/sophgo/sophgo-edk2.git

3. Enter repository.

.. highlights::

    .. code:: sh

        cd sophgo-edk2

4. Init submodules.

.. highlights::

    .. code:: sh

        git submodule sync
        git submodule update --init --recursive

5. Setup environments.

.. highlights::

    .. code:: sh

        export WORKSPACE=`pwd`
        export PACKAGES_PATH=$WORKSPACE/edk2:$WORKSPACE/edk2-platforms:$WORKSPACE/edk2-non-osi
        export GCC5_RISCV64_PREFIX=riscv64-unknown-elf-
        source edk2/edksetup.sh

6. Build base tools.

.. highlights::

    .. code:: sh

        make -C edk2/BaseTools

7. Build binaries for SG2042.

   Final output is ``$WORKSPACE/Build/SG2042_EVB/DEBUG_GCC5/FV/SG2042.fd``.

.. highlights::

    .. code:: sh

        build -a RISCV64 -t GCC5 -p Platform/Sophgo/SG2042_EVB_Board/SG2042.dsc

8. Enable ``MultiArchUefiPkg`` for graphics cards supporting UEFI.

   Final output is ``$WORKSPACE/Build/SG2042_EVB/RELEASE_GCC5/FV/SG2042.fd``.

.. highlights::

    .. code:: sh

        build -a RISCV64 -t GCC5 -b RELEASE -D X64EMU_ENABLE -p Platform/Sophgo/SG2042_EVB_Board/SG2042.dsc


.. note::

 - ``-D X64EMU_ENABLE`` is an option to enable MultiArchUefiPkg, an emulator for x64 OpRoms, or not.
   Please see the link https://github.com/intel/MultiArchUefiPkg for more details.
 - If you use ``-b DEBUG`` to replace ``-b RELEASE`` to build, MultiArchUefiPkg maybe not work.

9. Enable ACPI for SG2042.

   Final output is ``$WORKSPACE/Build/SG2042_EVB/RELEASE_GCC5/FV/SG2042.fd``.

.. highlights::

    .. code:: sh

        build -a RISCV64 -t GCC5 -b RELEASE -D ACPI_ENABLE -p Platform/Sophgo/SG2042_EVB_Board/SG2042.dsc


.. note::

 - ``-D ACPI_ENABLE`` is an option to enable ACPI. However, ACPI support is currently only provided for SG2042 x4 EVB.
 - To enable the ACPI feature in the linux kernel on the SG2042 platform, see the link https://github.com/sophgo/linux-riscv/tree/sg2042-dev-6.6-acpi.

Deploy
======

1. Mount your SD card (assume your SD card is ``/dev/sdc`` on your PC).

.. highlights::

    .. code:: sh

        sudo mount /dev/sdc1 /mnt

2. Copy ``SG2042.fd`` to your SD card in the ``0:riscv64/`` directory.

- Option 1: Replace ``riscv64_Image`` directly.

.. highlights::

    .. code:: sh

        sudo cp SG2042.fd /mnt/riscv64/riscv64_Image


- Option 2: Write ``conf.ini`` in the ``0:riscv64/`` directory.

  The example is as follows:

.. highlights::

    .. code:: ini

        [sophgo-config]

        [devicetree]
        name = mango-sophgo-x8evb.dtb
        ; name = mango-sophgo-x4evb.dtb
        ; name = mango-milkv-pioneer.dtb
        ; name = mango-sophgo-pisces.dtb

        [kernel]
        name = SG2042.fd

        [eof]

3. Build GRUB 2. Please see `How to build and config GRUB2 <https://github.com/sophgo/sophgo-doc/blob/main/SG2042/HowTo/How%20to%20build%20and%20config%20grub2.rst>`_.

- Option 1. Place ``grubriscv64.efi`` in the EFI partition of your SSD.

   * Ubuntu: place the ``grub.cfg`` in the EFI partition.
   * Fedora: modify ``grub.cfg`` in the BOOT partition of your SSD.

- Option 2. Automatically identify GRUB 2 during startup.

  * Ubuntu:
.. highlights::

    .. code:: sh

        sudo cp grubriscv64/ubuntu-rootfs/grubriscv64.efi /mnt/EFI/BOOT/BOOTRISCV64.EFI
        sudo cp grubriscv64/ubuntu-rootfs/grubriscv64.efi /mnt/EFI/ubuntu/
        sudo cp grubriscv64/ubuntu.cfg /mnt/EFI/ubuntu/grub.cfg


Create ``grub.cfg``. Put it to ``boot/grub/grub.cfg``.

.. highlights::

    .. code:: sh

        kernel_version=`ls /home/ubuntu/bsp-debs/linux-image-[0-9]*.deb | cut -d '-' -f 4 | cut -d '.' -f 1-3`
        set default=0
        set timeout_style=menu
        set timeout=2

        set term="vt100"

        menuentry 'Ubuntu on SG2042' {
                linux /boot/vmlinuz-$kernel_version  console=ttyS0,115200 root=LABEL=cloudimg-rootfs rootfstype=ext4 rootwait rw earlycon selinux=0 LANG=en_US.UTF-8
                initrd /boot/initrd.img-$kernel_version
        }

Put it to ``boot/grub/grub.cfg``.

.. highlights::

    .. code:: sh

        sudo mount /dev/sdc2 /media
        sudo mkdir /media/boot/grub
        sudo cp grub.cfg /media/boot/grub/


Run
===

1. Connect your serial port to RISC-V debug port (UART0).

2. Power on your board, wait untill entering the UEFI shell.

3. Boot Linux kernel using GRUB2. Type commands in the UEFI Shell as follows or put a ``startup.nsh`` file in the EFI partition of your SSD.

.. highlights::

    .. code:: sh

        fs0:
        grubriscv64.efi

