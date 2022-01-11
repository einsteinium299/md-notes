[[OS Installation]]

# Arch Linux install, system configuration & reference material.

Whenever I refer to an X value, you should fill in your own value like the drive you want to install Arch Linux to. Some commands in this guide can be very dangerous and may wipe all your files on the drive you're installing, be careful and know what each command does. For more detail, read the [ArchWiki](https://wiki.archlinux.org/title/Installation_guide). In this guide you will find the instructions for intalling the i3wm, if you do not want this, the start of this guide will still be relevant.

---
**PREINSTALLTION**
---
+ Check the validity of your ISO, it may have downloaded incorrectly or it has been tampered with. Download the PGP signature file here https://archlinux.org/download/ under the section 'Checksums'.
	
	```
	$ gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
	```

+ After this you may also check the md5sum and sha1sum from which the checksums can be found on the same page.

+ Burn an ISO to a USB drive with the command:
	```
	$ sudo dd bs=4M if=/home/username/herearchfile.iso of=/dev/sdX status="progress" conv=fsync oflag=direct 
	```

**INSTALLATION**

+ To set a larger font in terminal
	```
	$ setfont ter-132n
	```

+ Check if you're running UEFI with:
	```
	$ ls /sys/firmware/efi/efivars
	```
	If you do have uefi, follow the uefi steps. Most of the installation is the same.

+ Check internet connection with:
	```
	$ ping einsteinium.ga	
	```

+ Set the clock:
	```
	$ timedatectl set-ntp true
	```
	
+  Mirrorlist
	```
	$ pacman -Syyy

	$ pacman -S reflector

	$ reflector -c Netherlands -a 6 --sort rate --save /etc/pacman.d/mirrorlist

	$ pacman -Syyy
	```

+ Partitioning drive with fdisk:
	```
	fdisk /dev/sdX
	```

	** fdisk commands:**	
	- p = print partition table
	- d = delete partion
	- n = new partition
	- w = write
	- g = GPT lable
	- t = partition type
	
	Delete all partitions, make new partitions with "n" all the primary type.

	1. First partition is boot or uefi: +200M
		- for uefi boot partition type 't' and '1' to make an EFI System.
	2. Second is swap: +?G (150% of ram size)
	3. Third is root: +20G (25G to 30G for heavy installation)
	4. Last is home: the leftover space
	5. Use w to write and you're done with this step, this will delete all the data that was previously on the drive!


+ For UEFI make filesystem with:
	```
	$ mkfs.fat -F32 /dev/sdX1
	```
+ Making filesystems for each partition exluding swap with:
	```
	$ mkfs.ext4 /dev/sdX3
	$ mkfs.ext4 /dev/sdX4
	```
+ Make swap partition:
	```
	$ mkswap /dev/sdX2
	$ swapon /dev/sdX2
	```
	
	
+ Mounting partitions:
	First mount root, otherwise it will do weird things!
+ Root:
	$ mount /dev/sdX3 /mnt
+ Making directories for boot and home
	$ mkdir /mnt/home /mnt/boot

+ Mount the home and boot partitions
	```
	$ mount /dev/sdX1 /mnt/boot 
	*UEFI parition also mounted to boot partition.*	
	$ mount /dev/sdX4 /mnt/home
	```
+ Install kernel and other things with pacstrap:

	**Default kernel installation**
	$ pacstrap /mnt base base-devel linux linux-headers linux-firmware vim

	**Zen kernel installation**	
	$ pacstrap /mnt base base-devel linux-zen linux-firmware linux-zen-headers vim

	**hardend kernel for security**
	$ pacstrap /mnt base base-devel linux-hardened linux-firmware linux-hardened-headers vim

+ Create fstab file:
	$ genfstab -U /mnt >> /mnt/etc/fstab

+ Change root to the the new system
	$ arch-chroot /mnt

+ Install networkmanager and grub
	$ pacman -S networkmanager grub network-manager-applet

+ Enable network manager
	$ systemctl enable NetworkManager

+ Install grub
	**Legacy  bios**
	```
	$ grub-install --target=i386-pc /dev/sdX
	```
	
	**UEFI**
	```
	$ pacman -S efibootmgr
	$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
	```
	
+ Make grub configuration file:
	$ grub-mkconfig -o /boot/grub/grub.cfg

+ Set root password:
	$ passwd

+ Set up locale:
	$ vim /etc/locale.gen
	Uncomment
	en_US.UTF-8 UTF-8
	en_US ISO-8859-1
	$ locale-gen

+ Another locale:
	$ echo 'LANG=en_US.UTF-8' > /etc/locale.conf
	
+ Set timezone:
	$ ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

+ hwclock
	$ hwclock --systohc

+ Set hostname:
	$ vim /etc/hostname

+ exit from the system:
	$ exit

+ Unmount all partitions from the new system
	$ umount -R /mnt
	
+ Reboot
	$ reboot

---

**POSTINSTALLATION**
---

+ Add your user account:
	$ useradd -m -g wheel username

+ Give your user a password:
	$ passwd username
	
+ Change shell
	+ List available shells with: chsh -l
	+ Change shell with chsh -s /path/to/shell

+ Edit sudoers file so the new user can use sudo commands
	$ vim /etc/sudoers
	*Uncomment*
	% wheel ALL=(ALL) ALL

+ Install X.org
	$ sudo pacman -S xorg-server xorg-xinit

+ Install i3:
	$ sudo pacman -S i3-gaps i3status dmenu

+ Change .xinitrc:
	exec i3

+ Installing graphic drivers
		check what graphics card you're using with:
		$ lspci -k | grep -A 2 -E "(VGA|3D)"
	+ Install nvidia drivers:
		$ sudo pacman -S nvidia nvidia-utils nvidia-settings nvtop
	+ Install AMD driver:
		$ sudo pacman -S xf86-video-amdgpu
	+ Install intel driver:
		$ sudo pacman -S xf86-video-intel

+ Enable trim for ssd longevity
  $ sudo systemctl enable fstrim.timer
  $ sudo systemctl status fstrim.timer
  
+ Change /etc/fstab to auto mount drive or external server.
	find UUID by 
	$ lsblk -f

	add line:
	$ sudo vim /etc/fstab
	
	UUID=hereUUID	/path/to/mountpoint 	ext4	rw.relatime	0 2
	
+ My programs I like to use.
	sudo pacman -S htop ranger mpv firefox boinc neofetch sshfs pcmanfm zathura zathura-pdf-mupdf zathura-djvu git feh yt-dlp wget mlocate ntfs-3g cmake rkhunter ueberzug pavucontrol signal-desktop dino newsboat alacritty veracrypt picom openssh ufw nload zsh abook tmux

+ Installing YAY for aur packages.
	git clone https://aur.archlinux.org/yay.git
	makepkg -si PKGBUILD

+ Without aur helper
	$ git clone the aur package
	in the dir with PKGBUILD: makepkg -csi

+ AUR packages I like to use:
	yay -S xmrig-donateless lbry-app-bin anki-bin obsidian brave-bin vscodium-bin

+ Changing configs:
	Copy your config files for programs to their locations. I've got a github repo for my [dotfiles](https://github.com/einsteinium299/dotfiles), in case you need some inspiration. The best is to slowly learn them yourself so you understand your system.

+ Setting up firewall with UFW
  $ sudo ufw enable
  $ sudo ufw status
 
 + Setting up /etc/hosts
	127.0.0.1       localhost
	::1             	localhost
	127.0.1.1       hostname.localdomain     hostname

+ Firefox settings about:config

	>**Disable Pocket**
	browser.newtabpage.activity-stream.feeds.discoverystreamfeed to false
	browser.newtabpage.activity-stream.feeds.section.topstories to false
	browser.newtabpage.activity-stream.section.highlights.includePocket to false
	browser.newtabpage.activity-stream.showSponsored to false
	extensions.pocket.enabled to false
	**Disable prefetching**
	network.dns.disablePrefetch to true
	network.prefetch-next to false
	**Disable JavaScript in PDF**
	pdfjs.enableScripting to false
	**Harden SSL preferences**
	security.ssl3.rsa_des_ede3_sha to false //if not found, skip
	security.ssl.require_safe_negotiation to true
	**Disable Firefox account features**
	identity.fxaccounts.enabled

	Must have addons: uBlock, privacyredirect, bitwarden, xbrowsersync, darkreader, vimium

	Nice themes: C021, Firefox Alpenglow

+  Ssh keys
	Copy ssh key for my website to .ssh

+ Connect to wifi for laptop
	nmtui (easier)
	iwctl

+ To start and stop openssh
	sudo systemctl start sshd
	sudo systemctl stop sshd


	





