# Explanation begins with # and commands begin with $

# Enter a live environment of your choice
# Open terminal switch to root user
# Partition the disk

$ cfdisk

# 1G for boot
# 2x RAM for swap
# rest for root

# format the disk
$ mkfs.ext4 /dev/sda3
$ mkfs.fat -F 32 /dev/sda1
$ mkswap /dev/sda2

# Make required paths to mount if not already available
$ mkdir --parents /mnt/gentoo/efi
$ mkdir --parents /mnt/gentoo

# Mount partitions
$ mount /dev/sda3 /mnt/gentoo
$ mount /dev/sda1 /mnt/gentoo/efi
$ swapon /dev/sda2

$ cd /mnt/gentoo

# Set time if wrong
$ chronyd -q OR date MMDDhhmmYYYY

# Download tarball using wget or browser
$ wget <PASTED_STAGE_FILE_URL>

# Untar the tarball
$ tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner

# Edit make.conf for the first time
$ nano /mnt/gentoo/etc/portage/make.conf # Add lines "MAKEOPTS="-jn" 'n' hear is Memory/2

# Gentoo eubuild repository
$ mkdir --parents /mnt/gentoo/etc/portage/repos.conf
$ cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

# Copy DNS info
$ cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

# Mounting the necessary filesystems
$ mount --types proc /proc /mnt/gentoo/proc
$ mount --rbind /sys /mnt/gentoo/sys
$ mount --make-rslave /mnt/gentoo/sys
$ mount --rbind /dev /mnt/gentoo/dev
$ mount --make-rslave /mnt/gentoo/dev
$ mount --bind /run /mnt/gentoo/run
$ mount --make-slave /mnt/gentoo/run

# For non-gentoo live environments
$ test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
$ mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
$ chmod 1777 /dev/shm /run/shm

# Entering the new environment
$ chroot /mnt/gentoo /bin/bash
$ source /etc/profile
$ export PS1="(chroot) ${PS1}"

# Prepare bootloader
$ mount /dev/sda1 /efi

# Installing a Gentoo ebuild repository snapshot from the web
$ emerge-webrsync

# Updating the Gentoo ebuild repository
$ emerge --sync

# Read the news
$ eselect news list
$ eselect news read

# select the desktop profile
$ eselect profile list
$ eselect profile set 5

# Update @world set
$ emerege --ask --verbose --update --deep --newuse @world

# Enable USEFLAGS by adding following line to make.conf
$ USE="X -gnome -kde alsa pulseaudio elogind dbus" 

# Remove obselete packages
$ emerge --depclean

# CPU_FLAGS_ 
$ emerge --ask --oneshot app-portage/cpuid2cpuflags
$ cpuid2cpuflags
$ echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags

# VIDEO_CARDS | Add following line to make.conf
$ VIDEO_CARDS="intel"

# Accept the licences by adding following line in make.conf
$ ACCEPT_LICENSE="*"

# Set timezone 
$ echo "Asia/Kolkata" > /etc/timezone

# Generate locale by uncommenting en_US.UTF-8 UTF-8 in /etc/locale.gen
$ locale-gen
$ eselect locale list
$ eselect locale set <NUMBER>

# Download kernel
$ emerge --ask sys-kernel/linux-firmware
$ emerge --ask sys-kernel/gentoo-kernel-bin
$ emerge --depclean

# select kernel
$ eselect kernel list
$ eselect kernel 1

# Edit /etc/fstab by adding following lines
$ /dev/sda1   /efi        vfat    defaults             0 2
$ /dev/sda2   none        swap    sw                   0 0
$ /dev/sda3   /           ext4    defaults             0 1

# Set hostname
$ echo hostname > /etc/hostname

# Setup dhcpcd
$ emerge --ask net-misc/dhcpcd
$ rc-update add dhcpcd default
$ rc-service dhcpcd start

# Setup netifrc
$ emerge --ask --noreplace net-misc/netifrc
$ nano /etc/conf.d/net
$ config_eth0="dhcp"
$ cd /etc/init.d
$ ln -s net.lo net.eth0
$ rc-update add net.eth0 default

# edit the host file by adding hostname before "localhost" and after 127.0.0.1
$ nano /etc/hosts

# setup root password
$ passwd

# setup hardware clock
$ nano /etc/conf.d/hwclock
$ clock="local"

# setup sysklogd
$ emerge --ask app-admin/sysklogd
$ rc-update add sysklogd default

# setup mlocate
$ emerge --ask sys-apps/mlocate
$ updatedb

# setup chrony 
$ emerge -av net-misc/chrony
$ rc-update add chronyd default

# setup filesystem tools
$ emerge -av sys-fs/e2fsprogs
$ emerge -av sys-fs/dosfstools

# setup grub
$ emerge --ask --verbose sys-boot/grub
$ echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
$ grub-install --target=x86_64-efi --efi-directory=/efi
$ grub-mkconfig -o /boot/grub/grub.cfg

# EXIT & REBOOT
$ exit
$ cd
$ umount -l /mnt/gentoo/dev{/shm,/pts,}
$ umount -R /mnt/gentoo
$ reboot
