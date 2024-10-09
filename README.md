# nixOS-setup

This repository is meant to showcase my NixOS configuration, all with documentation and scripts that will follow my progress.

# How to install NixOS on your system

1) Download the [Minimal ISO image](https://nixos.org/download.html). You can find the download link to the ISO best suited for your hardware specifications at the bottom of the page.

3) Create a bootable media from the ISO. I personally used [Etcher](https://github.com/balena-io/etcher) for this step, but any other image writer available works fine as well.

5) Restart your PC and boot into the device you have flashed the image onto. You can usually achieve this by repeatedly pressing the `F9-F12` keys on your keyboard while the system boots.

7) Now that you have booted into the NixOS minimal installer, you need to establish an internet connection. Unfortunately, being a minimal installation, Network Manager is not available, so you will have to use WPA Supplicant to do the job. Fear not, it is not that hard. What you need to do first is activate the `wpa_supplicant` service. But, before doing that, make sure you are executing the following commands as root.

## Switch to root user

```bash
	sudo -i
```

## Enable the wireless device and the network service

```bash
    rfkill unblock wifi
	systemctl start wpa_supplicant
```

## Connecting to a Wi-Fi network

```shell
	 wpa_cli
	     > add_network
	     > set_network 0 ssid "<ssid>"
	     > set_network 0 psk "<password>"
	     > set_network 0 key_mgmt "<WPA-PSK or WPA-EAP>"
	     > enable_network 0
	     > quit
```

## Clone this repository

> You can skip this part if you do not wish to use my configuration.

```bash
	# Install and use git for this step
	nix-shell -p git
	git clone https://github.com/stefanRoberto/nixOS-setup

	# Switch back to root user
	exit 
```
## Disk Partitioning

```bash
	# Show block devices
	lsblk # you can also use blkid or fdisk -l

	# Start partitioning the desired disk
	gdisk </dev/device> # example: gdisk /dev/nvme1n1
		: d # use this to delete partitions 
		: n # use this to make new partitions
		: w # use this to write modifications
		: q # use this to quit gdisk
```

### ❗ <u>TODO</u> ❗
>
 - [ ] Continue working on **Disk Partitioning** by describing the type of partitions needed to be created (EFI partition,  Main partition, Swap partition)
 >
## Formatting the created partitions

 > Make sure to check your newly partitioned disk so that you know which partitions should be formatted. You can use `lsblk`, `fdisk -l` or `blkid`
 
```bash	
	# Format the EFI partition 
	mkfs.fat -F 32 -n boot /dev/<EFI-partition> # example: /dev/nvme1n1p1 

	# Format the main partition (use ext4, btrfs, or any other desired format)
	mkfs.ext4 -L nixos /dev/<main-partition> # example : /dev/nvme1n1p2

	# Format the swap partition
	mkswap -L swap /dev/<swap-partition> # example : /dev/nvme1n1p3
	
	# You can also enable swap directly after formatting
	swapon /dev/<swap-partition> # example : /dev/nvme1n1p3
```

## Mount the boot and main partitions

```bash
	# Create a new directory in /mnt/ for the boot partition
	mkdir -p /mnt/boot
	
	mount /dev/<EFI-partition> /mnt/boot
	mount /dev/<main-partition> /mnt/
```
## Generate the nixos config files

```bash
	nixos-generate-config
```


## Replace the configuration file

> Skip this step if you did not clone this repository to use my configuration.

```bash
	cp -f ~/nixOS-setup/configuration.nix /mnt/etc/nixos/configuration.nix
```

## Edit the configuration.nix file

> This is up to you to modify however you wish.
> You will find this file in `/mnt/etc/nixos/`, along with `hardware-configuration.nix`

```bash
	# You can use Vim or Nano, as they are both available to you
	vim /mnt/etc/nixos/configuration.nix
```

## Install NixOS

```bash
	nixos-install
```

> Now, you will be prompted at the end of the installation to set the root password.
> After you do this, reboot and remove the installation media.

This is it, you have successfully installed NixOS minimally. Congratulations!
