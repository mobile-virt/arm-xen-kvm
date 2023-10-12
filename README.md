# arm-legacy-kvm
This repository is for enabling support for kvm on older devices and is inspired by the work of @Marietto2008 to improve support for virtualization on arm chromebooks and related arm devices. This repository is specifically for supporting kvm on arm 32-bit chromebooks such as the chromebook snow (SAMSUNG XE303C12-A01US Samsung Series 3 Chromebook XE303C12 - Exynos 5 1.7 GHz).

Thanks to @Marietto2008 for testing and debugging work that has resulted in discovering important Linux kernel build parameters such as the correct settings in the config file, the correct setting of the uImage LOADADDR, the kernel and u-boot versions that have kvm support, and many other helpful hints.

This repository is for building the Linux kernel to support kvm on arm 32-bit devices and currently has released builds of Linux kernel version 5.4.257 with kvm support for armv7 using source from this location:

https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/snapshot/linux-5.4.257.tar.gz

The 5.4.y kernel series is chosen since it is the latest LTS kernel series to still have support for kvm on armhf (32-bit) devices which was removed upstream in kernel version 5.7:

[Goodbye KVM/arm](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=463050599742a89e0508355e626e032e8d0dab8d)

The two experimental builds are compiled with support for kvm for testing on the Samsung chromebook snow . See the latest release page for more details about these builds. As noted on the release page, they are not tested yet.
## Using the scripts to build a kernel
The kernels are built in a chroot using scripts. Only 5.4 armv7 kernels are supported and they can be cross-compiled on an x86_64 host. The scripts should automatically detect if it needs to install the cross compilers. In fact, so far, only cross compiling armv7 kernels on an x86_64 host has been tested. Root privileges are required to enter the chroot. The preferred way to run the scripts as root is by using sudo, and this will cause the build directory to be mounted in the chroot and in the host file system under the sudo user's home directory. Otherwise, the build directory will be mounted in the chroot and in the host file system in root's home directory, which is presumed to be /root in the scripts. Currently the scripts use bash and have not been tested with other shells.
### Create a chroot: $ sudo scripts/create-chroot name
Requires debootstrap is installed. The name is whatever the user chooses it to be. It should not have any spaces or special characters. This will create the chroot at /srv/chroot/name. The create-chroot script uses debootstrap to install a minimal Debian 12 system and all the build dependencies needed to build a version 5.4.y kernel. It also creates a cache for the downloaded packages at /srv/chroot/name-pkg-cache and configures appropriate bind mounts for /sys, /proc, /dev, and /dev/pts and creates a user account for the sudo user if sudo was used to gain root privileges. At that point, it is necessary to answer some questions that adduser asks before it creates the account. If the script ends before dropping into a bash shell in the chroot, take note of any messages, and before trying to create a chroot of the same name, try to destroy the failed attempt to create the chroot using the destroy-chroot script.
### Enter a chroot: $ sudo scripts/enter-chroot name
The chroot name needs to have been created previously by create-chroot. This will simply setup the chroot so one can run commands such as setting the time zone, add locales, etc. Note that when entering the chroot using sudo, the user will not have root privileges in the chroot initially, but when exiting the shell, the script will ask if the user needs to make changes that require root privileges, and the user will then have root privileges in the chroot as the root user.

The enter-chroot script will also create the build directory at $HOME/kernelbuild (or /root/kernelbuild if sudo was not used) in both the chroot and the host system the first time the chroot is entered if those directories do not already exist, otherwise, the scripts will use the pre-existing directory at $HOME/kernelbuild or /root/kernelbuild. If extra space is needed for the build, the $HOME/kernelbuild (or /root/kernelbuild when sudo is not used) directory can be mounted on a different filesystem on the host and it will always be bind mounted in the chroot. So the build directory will always be accessible in the host filesystem. It is possible to do most of the building in the chroot without root privileges, the  exception is when creating an initrd.img which depends on installing the kernel and modules, and that requires root privileges. The easiest way to get started building the kernel is by using the build-kernel script.
### Build a kernel: $ sudo scripts/build-kernel name config-file
The two kernels in the initial release have corresponding config files in the configs directory. After creating the chroot "name" one can attempt to build one of those kernels by passing the relative path to the config file as the second parameter on the command line of the build-kernel script. One can also try to build another kernel using the script, but it may be necessary to answer some questions if the kernel build system decides it needs to run make menuconfig or similar. It is possible to run 'make nconfig' using the enter-chroot script since the ncurses packages are installed by the create-chroot script.

The result of the build is a single tarball which contains the kernel image, depending on the config some dtb device tree files, a uImage file for booting with u-boot, modules if the kernel was configured to use modules, initrd.img, config-<version> and System.map-<version>. These results are what is uploaded as a release.
### Destroy a chroot: $ sudo scripts/destroy-chroot name
This script is a helper to at least check that none of the bind mounts that are done by the other scripts are still active which would indicate the chroot is in use by someone and in that case the script will refuse to destroy the chroot. When the script thinks it is safe, then it will ask once before destroying the chroot which uses rm -rf /src/chroot/name to destroy it. **Use this script carefully to avoid removing something important!**
### Future plans
Some testing has shown it is possible to use the enter-chroot script and enter the commands to make deb and rpm packages. The create-chroot script intalls the Debian 12 rpm package to support the rpm-pkg make targets. The build-kernel script will hopefully in the future be able to accept the option to build a deb or rpm package with a command line option. These deb and rpm packages include kernel header packages useful for developing apps that depend on the kernel headers and for building out-of-tree modules for these kernels. As an example, the build-kernel-deb script creates the Debian packages.
