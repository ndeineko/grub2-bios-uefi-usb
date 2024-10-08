# How to create a 3-in-1 bootable USB drive on Linux

A USB drive with only 1 partition* to load GRUB2 on USB-bootable machines with `Legacy BIOS`, `64-bit UEFI` or `32-bit UEFI`.

\**Due to the maximum size of a file inside an EFI system partition, files of 4 GiB or larger (such as some ISO images) must be placed on another partition. That second partition can be of type `ext4`, for instance.*

## Partition the drive and install GRUB2

**Warning : the USB drive will be formatted, save your data before proceeding!**

First of all, on your current installation, check if the folder `/usr/lib/grub/` exists and is not empty.
If it is empty or does not exist, make sure the package grub-common (or equivalent for your distribution) version 2 or higher is installed.
Depending on the system, `/usr/lib/grub/` will contain one or more of the following folders : `x86_64-efi`, `x86_64-efi-signed`, `i386-pc`, `i386-efi`, ...

The `x86_64-efi`, `i386-pc` and `i386-efi` folders need to be present in order to install the corresponding bootloader on the USB drive.

###### Install them using the package manager, for instance on Ubuntu :

`sudo apt install --no-install-recommends --yes grub-pc-bin grub-efi-ia32-bin grub-efi-amd64-bin`

Now, find the device file for your USB drive. Here, the file is `/dev/sdX`. Replace `X` with the appropriate lower case letter(s) in the commands.

###### Make sure it's the right drive (check the capacity and the partitions) :

`sudo fdisk -l /dev/sdX`

###### Open fdisk :

`sudo fdisk /dev/sdX`

###### Press the following keys (THIS WILL ERASE ALL DATA FROM THE SELECTED DRIVE) :

`o` `<enter>` # Create a new empty DOS partition table

`n` `<enter>` # Create a new partition

`p` `<enter>` # Select primary partition type

`1` `<enter>` # Set partition number to 1

`<enter>` # Start partition at the first possible sector (default)

`<enter>` # Set partition end to the last possible sector (default)

*Note : if you are asked whether the partition signature should be deleted, then answer yes.*

`t` `<enter>` # Change partition type

`e` `f` `<enter>` # Set partition type to EFI (FAT-12/16/32)

`a` `<enter>` # Enable the bootable flag on partition 1

`w` `<enter>` # Write the partition table

###### Create a fresh filesystem in the newly created partition :

`sudo mkfs.fat -F32 /dev/sdX1`

###### Mount the filesystem :

`sudo mount -o umask=000 /dev/sdX1 /mnt`

###### Write the MBR and install the GRUB files required for legacy BIOS boot on the drive :

`sudo grub-install --no-floppy --boot-directory=/mnt/boot --target=i386-pc /dev/sdX`

###### Install `/EFI/BOOT/BOOTX64.EFI` and other GRUB files required to load GRUB from a 64-bit UEFI firmware :

`sudo grub-install --removable --boot-directory=/mnt/boot --efi-directory=/mnt --target=x86_64-efi /dev/sdX`

###### Install `/EFI/BOOT/BOOTIA32.EFI` and other GRUB files required for a 32-bit UEFI :

`sudo grub-install --removable --boot-directory=/mnt/boot --efi-directory=/mnt --target=i386-efi /dev/sdX`

###### Create a grub.cfg file :

`touch /mnt/boot/grub/grub.cfg`

## Example grub.cfg with Xubuntu 22.04 Live
*Notes :*
* *Skip this part if you already have a working grub.cfg for the USB drive.*
* *Other examples can be found in this repository's [grub.cfg](grub.cfg) file.*

###### Create a folder for ISO images :

`mkdir /mnt/isos`

###### Download a Xubuntu ISO image (for example : [Xubuntu 22.04 64-bit](http://cdimages.ubuntu.com/xubuntu/releases/jammy/release/xubuntu-22.04.5-desktop-amd64.iso)) :

*Note : make sure there is enough space on the USB drive.*

`wget --directory-prefix=/mnt/isos http://cdimages.ubuntu.com/xubuntu/releases/jammy/release/xubuntu-22.04.5-desktop-amd64.iso`

###### Edit grub.cfg :

`nano /mnt/boot/grub/grub.cfg`

###### Paste (and ajust) this simple GRUB config file :

````
if [ "${grub_platform}" = "efi" ]; then rmmod tpm; fi # See https://bugs.launchpad.net/ubuntu/+source/grub2/+bug/1851311
menuentry 'Xubuntu 22.04 amd64' {
	set isofile="/isos/xubuntu-22.04.5-desktop-amd64.iso"
	#search --set=root --file $isofile # Uncomment if the bootloader and OS files are on different partitions
	loopback isoloop $isofile
	linux (isoloop)/casper/vmlinuz locale=en_US console-setup/layoutcode=us boot=casper iso-scan/filename=$isofile quiet --
	initrd (isoloop)/casper/initrd
}
````

*Notes about kernel boot parameters :*
* *Boot parameters used in the above example are specific to Ubuntu and its variants.*
* *`locale=` sets the language of the live system. Valid values include `en_US`, `pt_BR`, `zh_CN`, `fr_FR`, ...*
* *`console-setup/layoutcode=` sets the keyboard layout. Some possible values are `us`, `br`, `cn`, `fr`, ...*

###### Press the following keys :

`CTRL+O` `<enter>` # Save grub.cfg

`CTRL+X` # Exit nano

## Finish

###### Unmount the filesystem :

`sudo umount /mnt`
