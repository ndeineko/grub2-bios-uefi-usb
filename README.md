# How to create a 3-in-1 bootable usb drive on Linux.

A usb drive with only 1 partition to load grub2 on usb-bootable machines with `Legacy BIOS`, `64bit UEFI` or `32bit UEFI`.

*Note: due to the maximum size of a file inside an EFI system partition, files (such as ISO disk images) of 4 GiB or larger must be placed on another partition. That second partition can be of type ext4, for instance.*

## Partition the drive and install grub2

**Warning: the usb drive will be formatted, save your data before proceeding!**

First of all, on you current installation, check if the folder `/usr/lib/grub/` exists and is not empty.
If it is empty or does not exist, make sure the package grub-common (or equivalent for your distribution) version 2 or higher is installed.
Depending on the system, `/usr/lib/grub/` will contain one or more of the following folders: `x86_64-efi`, `x86_64-efi-signed`, `i386-pc`, `i386-efi`, ...

The `x86_64-efi`, `i386-pc` and `i386-efi` folders need to be present in order to install the corresponding bootloader on the usb drive.

###### Install them using the package manager, for instance on Ubuntu :

`sudo apt install --no-install-recommends --assume-yes grub-pc-bin grub-efi-ia32-bin grub-efi-amd64-bin`

Now, find the device file for your usb drive. Here, the file is `/dev/sdX`. Replace `X` with the appropriate lower case letter(s) in the commands.

###### Make sure it's the right drive! (check the capacity and the partitions) :

`sudo fdisk -l /dev/sdX`

###### Open fdisk :

`sudo fdisk /dev/sdX`

###### Press the following keys (THIS WILL ERASE ALL DATA FROM THE SELECTED DRIVE!) :

`o` `<enter>` # Create a new empty DOS partition table

`n` `<enter>` # Create a new partition

`p` `<enter>` # Select primary partition type

`1` `<enter>` # Set partition number to 1

`<enter>` # Start partition at the first possible sector (default)

`<enter>` # Set partition end to the last possible sector (default)

*Note: if fdisk (newer versions only) asks whether the partition signature should be deleted, then answer yes.*

`t` `<enter>` # Change partition type

`e` `f` `<enter>` # Set partition type to EFI (FAT-12/16/32)

`a` `<enter>` # Enable the bootable flag on partition 1

`w` `<enter>` # Write the partition table

###### Create a fresh filesystem in the newly created partition :

`sudo mkfs.fat -F32 /dev/sdX1`

###### Mount the filesystem :

`sudo mount -o umask=000 /dev/sdX1 /mnt`

###### Write the MBR and install the grub files required for legacy BIOS boot on the drive :

`sudo grub-install --no-floppy --boot-directory=/mnt/boot --target=i386-pc /dev/sdX`

###### Install `/EFI/BOOT/BOOTX64.EFI` and other grub files required to load grub from a 64-bit UEFI firmware :

`sudo grub-install --removable --boot-directory=/mnt/boot --efi-directory=/mnt --target=x86_64-efi /dev/sdX`

###### Install `/EFI/BOOT/BOOTIA32.EFI` and other grub files required for 32-bit UEFI :

`sudo grub-install --removable --boot-directory=/mnt/boot --efi-directory=/mnt --target=i386-efi /dev/sdX`

###### Create a grub.cfg file :

`touch /mnt/boot/grub/grub.cfg`

## Example grub.cfg with Xubuntu 22.04 Live
*Notes :*
* *Skip this part if you already have a working grub.cfg for the usb drive.*
* *Other examples can be found in this repository's grub.cfg file.*

###### Create a folder for cd images :

`mkdir /mnt/isos`

###### Download an Ubuntu CD image (for example: [Xubuntu 22.04 64-bit](http://cdimages.ubuntu.com/xubuntu/releases/jammy/release/xubuntu-22.04.4-desktop-amd64.iso)) :

*Note: make sure there is enough space on the usb drive.*

`wget --directory-prefix=/mnt/isos http://cdimages.ubuntu.com/xubuntu/releases/jammy/release/xubuntu-22.04.4-desktop-amd64.iso`

###### Edit grub.cfg :

`nano /mnt/boot/grub/grub.cfg`

###### Write or paste something like this :

````
menuentry 'Xubuntu 22.04 amd64'{
	#rmmod tpm #uncomment if grub version is >=2.04 in UEFI mode (see https://bugs.launchpad.net/ubuntu/+source/grub2/+bug/1851311)
	set isofile="/isos/xubuntu-22.04.4-desktop-amd64.iso"
	#search --set=root --file $isofile #uncomment if the bootloader and OS files are on different partitions
	loopback isoloop $isofile
	linux (isoloop)/casper/vmlinuz locale=fr_FR console-setup/layoutcode=fr boot=casper iso-scan/filename=$isofile quiet --
	initrd (isoloop)/casper/initrd
}
````

*Notes :*
* *Remove or change the value of the `locale` parameter to set the language of the live system.*
* *Remove or change the value of the `console-setup/layoutcode` parameter to change the keyboard layout.*

###### Save grub.cfg (in nano) and exit :

`CTRL+O`

`<enter>`

`CTRL+X`

## Finish

###### Unmount the filesystem :

`sudo umount /mnt`
