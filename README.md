# U-Boot-Dock

This github repository contains a collection of u-boot macros which make
the u-boot usage more beneficial. Almost every SBC vendor introduces his
very own way how u-boot is going to be used. This project tries to consolidate
these different approaches with a new consistent, flexibel and clean structure.

# Warning

Installing this collection of u-boot macros will change the stored u-boot
environment on your device. This might render your SBC unusable. Especially
when the u-boot environment is saved on a non-removable storage device
(e.g. the Utilite) you might brick your device.

# Motivation

When I started development on my different SBC, I was quite unhappy with
the existing boot mechanisms I've found. I missed the ability to choose
between different configurations during boot. For example I wanted to test
a newly build Linux kernel without overwriting the existing one. Or I
wanted to boot various Linux flavours stored on different partitions on the
same storage device instead of swapping a bunch of µSD cards.

# Lifting Constraints

Many SBC vendors deployed boot images require a two partition layout:

* Boot: a small vat partition carrying the Linux kernel image
* Root: containing the Linux root filesystem

In the very first days this partitioning was necessary due to u-boot's
limitation being able to read files from fat filesystems only. Nowadays
u-boot supports reading files from other filesystems (e.g. ext4, btrfs)
no matter which partition they reside.

A `single partition layout` can lift old burdons.

Because `u-boot` does not force the usage of vfat (unlike UEFI based
approaches) we can lift burdons using other filesystems:

* take advantage of a relocatable, self-contained root filesystem.
* implement multiboot environments, without side effects outside their root
* take advantage of symbolic links inside the `/boot` structure

# Restructure `/boot` directory layout

The design goal of a simple, thus intuitive directory layout for `U-Boot`
should isolate object-structures.

* kernels can be coupled with different inital ram disks
* different SBC boards need their corresponding device tree blobs
* distros may rely on given metadata. For compatibility reasons a
`config-$(make kernelrelease)` might be still present in `/boot` (needed
for tools like `update-initramfs`).
* each configuration may have its own overlay for the default U-Boot
environment(`uEnv.txt`)
* a pre-compiled U-Boot script (`boot.scr`, optional).

This leads to following flat design proposal:

* /boot
  * ./conf.d
	* ./conf.d/default
		* ./conf.d/default/uImage
		* ./conf.d/default/uInitrd
		* ./conf.d/default/board.dtb
		* ./conf.d/default/uEnv.txt
	* ./conf.d/test
		* ./conf.d/test/vmlinuz-amlogic-5.4
		* ./conf.d/test/initramfs-amlogic-5.4-.img
		* ./conf.d/test/meson-sm1-khadas-vim3l.dtb
		* ./conf.d/default/uEnv.txt

Or a complex implementation using symbolic links:

* /boot
  * ./conf.d
	* ./conf.d/default -> ../../default
	* ./conf.d/rescue -> ../../rescue
	* ./conf.d/arch-5.5 -> ../../arch-5.5
	* ./conf.d/vim3l -> ../../vim3l
	* ./conf.d/cubieboard4 -> ../../cubieboard4
	* ./conf.d/arch-mainline -> ../../arch-5.4
	* ./conf.d/android-pie -> ../../kernel.d/android
  * ./default
	* ./default/uImage -> ../../kernel.d/ubuntu/vmlinuz
	* ./default/uInitrd -> ../../kernel.d/ubuntu/Initrd
	* ./default/board.dtb -> ../../kernel.d/ubuntu/board.dtb
	* ./default/uEnv.txt
  * ./rescue
	* ./rescue/uImage -> ../../kernel.d/ubuntu/vmlinuz
	* ./rescue/uInitrd -> ../../kernel.d/ubuntu/Initrd-failsave
	* ./rescue/board.dtb -> ../../kernel.d/ubuntu/board.dtb
	* ./rescue/uEnv.txt
  * ./arch-5.4
	* ./arch-5.4/vmlinuz-amlogic-5.4 -> ../../kernel.d/arch/vmlinuz-amlogic-5.4
	* ./arch-5.4/initramfs-amlogic-5.4.img -> ../../kernel.d/arch/initramfs-amlogic-5.4.img
	* ./arch-5.4/meson-sm1-khadas-vim3l.dtb -> ../../kernel.d/arch/meson-sm1-khadas-vim3l.dtb
	* ./arch-5.4/uEnv.txt
  * ./arch-5.5
	* ./arch-5.5/vmlinuz-amlogic-5.5-rc1 -> ../../kernel.d/arch/vmlinuz-amlogic-5.5-rc1
	* ./arch-5.5/initramfs-amlogic-5.5-rc1.img -> ../../kernel.d/arch/initramfs-amlogic-5.5-rc1.img
	* ./arch-5.5/meson-sm1-khadas-vim3l.dtb -> ../../kernel.d/arch/meson-sm1-khadas-vim3l.dtb
	* ./arch-5.5/uEnv.txt -> ../../kernel.d/arch/uEnv.txt
  * ./arch-5.5-vim3
	* ./arch-5.5-vim3/vmlinuz-amlogic-5.5-rc1 -> ../../../kernel.d/arch/vmlinuz-amlogic-5.5-rc1
	* ./arch-5.5-vim3/initramfs-amlogic-5.5-rc1.img -> ../../kernel.d/arch/initramfs-amlogic-5.5-rc1.img
	* ./arch-5.5-vim3/meson-sm1-khadas-vim3l.dtb -> ../../kernel.d/arch/meson-sm1-khadas-vim3l.dtb
	* ./arch-5.5-vim3/uEnv.txt -> ../../kernel.d/arch/uEnv.txt
  * ./cubieboard4
	* ./ubuntu/vmlinuz -> ../../../kernel.d/ubuntu/vmlinuz-amlogic-5.5-rc1
	* ./ubuntu/Initrd.img -> ../../kernel.d/ubuntu/initramfs-amlogic-5.5-rc1.img
	* ./ubuntu/cubieboard4.dtb -> ../../kernel.d/ubuntu/cubieboard4.dtb
	* ./ubuntu/uEnv.txt
  * ./vim3l
	* ./vim3l/vmlinuz-amlogic-5.4 -> ../../kernel.d/arch/vmlinuz-amlogic-5.4
	* ./vim3l/initramfs-amlogic-5.4.img -> ../../kernel.d/arch/ubuntu/initramfs-amlogic-5.4.img
	* ./vim3l/meson-sm1-khadas-vim3l.dtb -> ../../kernel.d/arch/meson-sm1-khadas-vim3l.dtb
	* ./vim3l/uEnv.txt
  * ./kernel.d
	* ./dtfs
		* ./dtbs/amlogic
			* ./dtbs/amlogic/meson-g12b-odroid-n2.dtb
			* ./dtbs/amlogic/meson-g12b-a311d-khadas-vim3.dtb
			* ./dtbs/amlogic/meson-sm1-khadas-vim3l.dtb
		* ./dtbs/sunxi
			* ./dtbs/sunxi/sun7i-a20-bananapro.dtb
			* ./dtbs/sunxi/sun9i-a80-cubieboard4.dtb
  * ./kernel.d/ubuntu
	* ./kernel.d/ubuntu/vmlinuz
		* ./kernel.d/ubuntu/Initrd
		* ./kernel.d/ubuntu/Initrd-failsave
		* ./kernel.d/ubuntu/bananapro.dtb -> ../dtbs/sunxi/sun7i-a20-bananapro.dtb
		* ./kernel.d/ubuntu/cubieboard4.dtb -> ../dtbs/sunxi/sun9i-a80-cubieboard4.dtb
	* ./kernel.d/arch
		* ./kernel.d/arch/vmlinuz-amlogic-5.4
		* ./kernel.d/arch/vmlinuz-amlogic-5.5-rc1
		* ./kernel.d/arch/initramfs-amlogic-5.5-rc1.img
		* ./kernel.d/arch/initramfs-amlogic-5.4-fallback.img
		* ./kernel.d/arch/initramfs-amlogic-5.4.img
		* ./dtfs/amlogic/meson-g12b-odroid-n2.dtb -> ../../dtfs/amlogic/meson-g12b-odroid-n2.dtb
		* ./dtfs/amlogic/meson-g12b-a311d-khadas-vim3.dtb -> ../../dtfs/amlogic/meson-g12b-a311d-khadas-vim3.dtb
		* ./kernel.d/arch/meson-sm1-khadas-vim3l.dtb -> ../../dtfs/amlogic/meson-sm1-khadas-vim3l.dtb
		* ./kernel.d/arch/uEnv.txt
	* ./kernel.d/android
		* ./kernel.d/android/kernel.img
		* ./kernel.d/android/SYSTEM
		* ./kernel.d/android/dtb.img
		* ./kernel.d/android/boot.ini
		* ./kernel.d/android/aml_autoscript

## Substructure `/boot/kernel.d/<arch>`

Create as subdirectory for any supported kernel architecture. Inside the
subdirectory save the vendor supported blobs

  * kernel image binary (`[z|u]Image`)
  * device tree binary (`*.dtb`)
  * initial ramdisk image (`[u]Initrd`)
  * additions like `System.map` or `config` (not needed for boot)

## Substructure `/boot/conf.d/<config>`

This directory will store different, selectable `U-Boot` configurations.
You are free to save the needed binary blobs itself or create symbolic links to
the structured vendor/system blobs.

	* symlinks to saved in the kernel.d structure
	* symlinks to saved arch structure referencing the kernel.d

Usualy you will implement

* `/boot/conf.d/default/`, the default U-Boot configuration
* `/boot/conf.d/test/`, for testing a new kernel or uEnv.txt or whatever
* `/boot/conf.d/failsafe/`, the fail safe configuration which is supposed
	to boot under any circumstances

 The configuration directory itself can be a
symbolic link, so resetting `/boot/conf.d/default/` will change the default
U-Boot configuration.


# Custom U-Boot environment (`uEnv.txt`)

U-Boot loads it's persistent environment during startup. There is a
particular initial default environment for each board which already
contains all necessary settings to boot up the default configuration,
i.e. the board will boot even if the `uEnv.txt` is not present or has been
accidentally deleted. If this file is present in the configuration
directory, it's contents will be merged on the fly into the actual U-Boot
environment (but not be written back to the persistent U-Boot environment).

# Some conventions about U-Boot variables

`U-Boot` can significantly gain much more flexiblilty while using
macros and environment variables.

There are two different ways to define a variable in the U-Boot shell / monitor:

*  `setenv foo bar`: adds the variable `foo` to the U-Boot environment
*  `foo=bar`: sets the shell-only variable `foo` which is not part of the
	  U-Boot environment (might be helpful if the environment size is
	  rather small), but cannot be used for calling macros.

This proposel will structure the variable names pepending following
constants to the variable name:


* `p_<var-name>`: Shell-only variables passing parameters to macros
* `v_<var-name>`: local variables inside the shell
* `m_<var-name>`: all U-Boot variables containing macros. This helps to
	distinguish code parts from 'real' U-Boot variables.
*  `k_<var-name`: the macro `m_set_bootargs` will uses this class of
	variables to compose the kernel command line (aka `bootargs`).  As an
	example, instead of altering `bootargs` directly, you might only want
	to change the variable `k_rootfs`. The macro-call of `m_set_bootargs`
	will take care to include your choosen value to the `bootargs`
	environment variable.

# Default U-Boot variables

The following U-Boot variables are defined in the initial default
environment and can be overwritten (ether via macros or when defined via
uEnv.txt):

| variable | semantics | typical value |
|:---------|:----------|:--------------|
|bootcmd   | default command to be executed | `run m_autoboot;`|
|bootenv   | file containing custom U-Boot environment | `uEnv.txt`|
|bootdelay | time to interrupt non-interactive booting | `5` |
|bootscr   | compiled U-Boot macro to be executed automatically | `boot.scr` |
|dbglevel  | turn on/off debugging messages| `0` |
|errlevel  | turn on/off error messages| `0` |
|fdt       | file containing the device tree binary | board specific |
|ramdisk   | file containing initial ramdisk content | `uInitrd` |
|kernel    | file containing kernel image | `zImage` |
|mmcdev    | MMC device:partion to boot from | `0:1` |
|prefix    | directory containing U-Boot configurations | `/boot/conf.d` |
|run_pre_boot| execute the `m_pre_boot` macro immediately before `boot?` is executed | `0` |
|satadev   | SATA device:partition to boot from | `0:1` |
|usbdev    | USB device:partition to boot from | `0:1` |


# Boot sequence

## Typical boot sequence

### [`m_autoboot`](https://github.com/umiddelb/u-571/blob/master/macro/m_autoboot.uboot)
The macro `m_autoboot` implements the boot priority. You can tell U-Boot to
try to boot from USB first (`run m_autoboot_usb`) then to try from SATA
(`run m_autoboot_sata`) and finally to boot from MMC (`run
m_autoboot_usb`).

### [`m_autoboot_*`](https://github.com/umiddelb/u-571/blob/master/macro/m_autoboot_mmc.uboot)
This macro initializes the boot device and calls `m_boot_conf`.

### [`m_boot_conf`](https://github.com/umiddelb/u-571/blob/master/macro/m_boot_conf.uboot)
This is the main loop which boots a certain configuration (`p_conf`) from a specific device.

#### [`m_load_env`](https://github.com/umiddelb/u-571/blob/master/macro/m_load_env.uboot)
This macro looks for a text file called `uEnv.txt` inside of the
configuration directory (`$prefix/$p_conf`), loads its contents and merges
them with the existing U-Boot environment by overriding values of existing
variables.

#### [`m_run_bootscr`](https://github.com/umiddelb/u-571/blob/master/macro/m_run_bootscr.uboot)
This macro looks for a file called `boot.scr` inside of the configuration
directory, load the file and execute its contents (usually without
returning back). `boot.scr` contains compiled U-Boot commands in a binary
format. This step is optional. Please refere to U-Boot's utility `mkimage`
to create a `boot.scr`.

#### [`m_set_bootargs`](https://github.com/umiddelb/u-571/blob/master/macro/m_run_bootscr.uboot)
This macro composes the kernel command line by setting the U-Boot variable
`bootargs`.

#### [`m_load_kernel`](https://github.com/umiddelb/u-571/blob/master/macro/m_load_kernel.uboot)
This macro loads the kernel image from `$prefix/$p_conf/kernel/$kernel`.

#### [`m_load_ramdisk`](https://github.com/umiddelb/u-571/blob/master/macro/m_load_ramdisk.uboot)
This macro loads the initial ramdisk archive from
`$prefix/$p_conf/$ramdisk`.  The archive is created and updated by using
the `update-initramfs` utility, which is usually done when a new kernel
image has been installed. Although the kernel will boot without an initial
ramdisk, most up-to-date Linux distributions will benefit from it.

#### [`m_load_fdt`](https://github.com/umiddelb/u-571/blob/master/macro/m_load_fdt.uboot)
This macro loads the device tree binary from `$prefix/$p_conf/$fdt`.
The Linux kernel on ARM needs a low level device description (device tree)
in binary format, either appended to the kernel image or as a separate
file. Many platforms prefer loading the device tree binary as a separate
file, which offers more flexibility, allowing distribution of a single
installation image for different platforms. It also allows tweaking of the device tree
for different use cases (see `m_pre_boot`).  The variable `ftd` may contain
a [list of
names](https://github.com/umiddelb/u-571/blob/master/board/odroid-xu4/board_init.env#L11)
(delimited by a white space), which will be probed subsequently.

#### [`m_pre_boot`](https://github.com/umiddelb/u-571/blob/master/board/odroid-c1/uEnv.txt#L11)
In some case you want to modify the device tree after being loaded into
memory.  This macro is called immediately before the `boot*` command is
issued.  Some use cases require a modification of the already loaded device
tree. Using this macro will let you do so.


#### Depending on the kernel format (denoted by the actual file name), the kernel image will be booted via `booti`, `bootz` or `bootm`.


# Use Cases

## Non Permanent Changes
Running non permanent changes require access to the serial console in order
to interrupt the automated U-Boot boot procedure and to issue own U-Boot
shell commands.

### Boot a different kernel

Booting a freshly compiled Linux kernel without bricking my device was one
of the reasons why I've build U-Boot-Dock. This can be achieved by creating
another configuration directory, e.g. `/boot/conf.d/test`. Then create a
`uEnv.txt` file and a `kernel` link (e.g. pointing to /boot/kernel.d/test,
assuming that your test kernel image has been copied there).

Lets assume, you have prepared a functional `boot-structure` and conneted
the SBC to your workstation with a USB serial connection, you are ready
to issue a reboot and enter an interactive U-Boot shell. Issue

	p_conf=test; run m_autoboot_mmc;

and U-Boot will start from MMC using your `test` environmet instead of the
system `default` configuration.

### Boot a rescue/fail save configuration


### Boot from (by U-Boot) unsupported storage devices

Create a particular configuration on the first internal storage device
accessible by U-Boot and modify the `k_rootfs` variable inside the `uEnv.txt`
file. Let it point to the actual device name (eg. `/dev/sda1`) or define
its unique identifier (eg. `UUID=<uuid-value>`).

Suggestion

- /Boot/conf.d/u0p1/ for USB0 Partiton 1
- /boot/conf.d/s0p2/ for SATA0 Partition 2
- on the first partition , usually eMMC or µSD


## Permanent Changes (affecting the default configuration)

### boot a different kernel

Let `/boot/conf.d/default/kernel` point to a different directory within
`/boot/kernel.d/`

### Boot with different kernel settings

Modify `/boot/conf.d/default/uEnv.txt`.

### Boot from the 2nd partition

Modify the U-Boot environment variable `mmcdev`, `satadev` or `usbdev` from
`0:1` to `0:2` and make sure that `/boot/...` is populated accordingly on
`0:2`.

## boot from USB device first
Modify the U-Boot environment variable `m_autoboot` in a way that `... run
m_autoboot_usb ...` is called first.

# Target installation

## Toolset

To ease the use of `U-Boot-Dock` a small toolset is provided.

[Makefile](https://github.com/umiddelb/u-571/blob/master/macro/Makefile)
targets are called to resolve your presets, which are board specific. This
board specific definitions are structured in a the `board\<board-name>`
subdirectory.

U-Boot will handle macros as one large character string. This makes editing
not very intuitive, since you can't use line feeds, indentation, nor
comments. `U-Boot-Dock's` defines its macro definitions in sources files
(`*.uboot`) inside the subdirectory `macros`. When Makefile targets are
executed, they take care to remove comments, line feeds and identation. All
macro resultions be will combine combine and the final U-Boot complient
environment will be saved.

Please adapt following files inside the board subdirectory:

* board_init.env (board defaults)
* uEnv.txt (runtime adaptions)

## Userland access to the U-Boot environment

The U-Boot environment is stored on dedicated space outside the
filesystem. The U-Boot-tools package contains the tool

* fw_setenv
* fw_printenv

to access the U-Boot environment from the userland. You need to configure
this tools according to your specific board. If the tools can't access your
target device the macro call from the Toolset (target `install`) will
fail.

Please refer to [this
article](https://github.com/umiddelb/armhf/wiki/Get-more-out-of-%22Das-U-Boot%22)
for more details.

## Flashing your U-Boot environment

First you need to select your SBC for the given board subirectory. You are
welcome to create an entry for a non listed board and expand the given selection.

Please choose your [board](https://github.com/umiddelb/u-571/tree/master/board)

As a second step, open a terminal and change into the choosen board config.
Adapt your `board_init.env`. When ready, flash the config to your boot
media by calling

`make install`

You may create a `bundle.Env` file, that will reflact the U-Boot
environment that the previoas flash call will store on your target device.

## License

<!-- License source -->
[Logo-CC_BY]: https://i.creativecommons.org/l/by/4.0/88x31.png "Creative Common Logo"
[License-CC_BY]: https://creativecommons.org/licenses/by/4.0/legalcode "Creative Common License"

This work is licensed under a [Creative Common License 4.0][License-CC_BY]

![Creative Common Logo][Logo-CC_BY]

© 2016, 2019  Uli Middelberg;
© 2019 Ralf Zerres
