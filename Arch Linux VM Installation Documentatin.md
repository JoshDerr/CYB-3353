# Arch Linux VM Installation Documentation 
### By Josh Derr

## Step 1: Installing the installation image
1) Visit the [download page](https://archlinux.org/download/) listed on the wiki and select a site available to the United States from the HTTP Direct Downloads subheading.
2) Download the "archlinux-x86_64.iso" file from the site.

## Step 2: Creating a New Virtual Machine
1) Open VMware Workstation Pro.
2) Select the "Create a New Virtual Machine" option from the listed categories.
3) When prompted for "Installer disc image file (iso)" select the iso file downloaded in the previous step.
4) When prompted to select a guest OS, choose "Linux" as the guest OS and "Other Linux 5.x kernel 64-bit" as the version.
5) Name your new virtual machine, I reccomend "Baby's First VM".
6) Select the location of your VM. 
7) Set the maximum disk space (in GB) that you want your VM to compose (at least 8 GB). 
8) Verify the information on the "Ready to Create a Virtual Machine" is correct, if so select "Finish"!
9) Go into the settings of your newly created VM, on the hardware tab ensure that there is at least 1 GB memory allocated and 1 processing core. 
10) While in the settings menu, navigate to the advanced setting via the options tab and ensure that the firmware type is set to UEFI.

## Step 3: Starting the VM
1) Once the Wizard has finished and you have made your additional changes, select the "Power on this virtual machine" button to start the VM.
2) Your VMware Workstation should have now booted into the iso file.
3) Wait for the guest operating system to finish its initial launch processes and then click the "I Finished Installing" button on the bottom of the VMware Workstation page.
4) To ensure that it booted correctly in UEFI mode run the command "# cat /sys/firmware/efi/fw_platform_size". Returned should either be 64 or 32. 
5) If 64 or 32 are not returned, go into the system settings and change boot from BIOS to UEFI.

## Step 4: Partition the Disks
1) Run the command "fdisk -l" to identify the disks assigned to block devices. 
2) Run the command "fdisk /dev/sda" to start fdisk on the hard drive.
3) Run the command "g" to initially create a partition table.
4) Run the command "n" to create each new partition (in this case we need one to be 300MiB, >512MiB, and the last being the remainder).
5) When creating the partitions leave the partition number and first sector as default. The last sector is variable depending on how big you initialized the hard drive to be and the size you want for the partition.
6) Make sure to use the command "w" when finished to write/save and exit fdisk.

## Step 5: Formatting the Partitions
1) Run the command "mkfs.ext4 /dev/sda3" to format sda3 an ext4 partition.
2) Run the command "mkswap /dev/sda2" to format sda2 as a swap partition.
3) Run the command "mkfs.fat -F32 /dev/sda1" to format the sda1 partition as FAT32.

## Step 6: Mounting the Partitions
1) Run the command "mount /dev/sda3 /mnt" to mount the third partition.
2) Run the command "swapon /dev/sda2" to mount the second partition.
3) Finally, run the command "mount --mkdir /dev/sda1 /mnt/boot" to mount the first partition.

## Step 7: Installation and Configuration of the System
1) Utilize the command "pacstrap -k /mnt base linux linux-firmware nano man-db man-pages texinfo iproute netctl util-linux grub efibootmgr" to install the base package, linux kernel, firmware, and all other packages needed for installation.
2) Generate a fstab file with the command "genfstab -U /mnt >> /mnt/etc/fstab".
3) Change root into the new system with the command "arch-chroot /mnt". 
4) Set the clock of your system with the command "ln -sf /usr/share/zoneinfo/US/Central /etc/localtime". 
5) Generate "/etc/adjtime" by using the command "hwclock --systohc".
6) Generate UTF-8 locales with the command "locale-gen".
7) Run the command "nano /etc/locale.gen" and uncomment the command "en_US.UTF-8 UTF-8".
8) Run the command "nano /etc/locale.conf" to create the locale.conf file and once inside paste the command "LANG=en_US.UTF-8" and save.
9) Use the command "nano /etc/hostname" to create the hostname file and name the file inside whatever you like.
10) Run the command "systemctl enable systemd-networkd".
11) Run the command "systemctl enable systemd-resolved".
12) Use the ip addr command to find the name of the interface (ens33).
13) Create the file "nano etc/systemd/network/20-wired.network".
14) Enter the following: 
"[Match]
Name=ens33

[Network]
DHCP=yes"
15) Set the root password with the command "passwd", my password is private, wouldn't you like to know it weather boy?
16) Run the command "grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB" to intall GRUB on the disk.
17) To make the configuration file, use the command "grub-mkconfig -o /boot/grub/grub.cfg".
18) After the previous steps, exit from chroot and then use command "umount -R /mnt" to unmount.
19) Finally, use the reboot command and log into arch!

## Step 8: Adding a GUI (Gnome)
1) Login to arch with the username root and the password previously created with the passwd command.
2) Install Xorg with the command "pacman -S xorg xorg-server".
3) Install the Gnome desktop environment by running the command "pacman -S gnome".
4) To start the gdm service and enable the Gnome GUI on start-up, use the command "systemctl enable gdm.service".

## Step 9: Adding users
1) In the terminal on the GUI, use the command "useradd -m codi" to add the user cofi.
2) To give the user codi sudo permissions, first make sure the sudo package is installed with the command "pacman -S sudo".
3) Run the command "EDITOR=nano visudo" to get into the sudo file.
4) Once in the sudo file, uncomment the command addressing the sudo group.
5) Now add sudo as a group to the system with the "groupadd sudo" command.
6) Add the user codi to the sudo group with the command "usermod -aG sudo codi".
7) Change the password of the user codi to be whatever you like with the command "passwd codi".
8) Set the password to be changed upon first login with the command "passwd -e codi".

## Step 10: Adding ZSH
1) Run the command "pacman -S zsh" to install the zsh package.

## Step 11: Color Coding the Terminal and Adding Aliases
1) To add color to the terminal, open the terminal as the user you would like and run the command "nano ~/.bashrc".
2) Once inside this file, comment out the existing PS1 variable and replace it with "PS1='\[\e[95;1m\]\u\[\e[0;38;5;51m\]@\[\e[93m\]\h\[\e[0m\]'" in a user profile.
3) To add aliases, use the command "alias (insert name)='(command to aliased)'".
4) Write and quit this file.