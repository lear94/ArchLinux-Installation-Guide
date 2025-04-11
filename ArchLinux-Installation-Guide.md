# Arch Linux Installation Guide

This guide will help you install Arch Linux step by step. Make sure to follow each section in the given order.


## **[Section 1] Connect Wi-Fi and Set Date and Time**

### **1.1 Connect Wi-Fi**

```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <SSID>
```

It’s important to ensure that the date and time are correctly synchronized before starting the installation.

```
timedatectl set-ntp true
```

## **[Section 2] Partition Formatting**

Replace `/dev/deviceX` with the actual devices for your installation.

### **2.1 Formatting Partitions**

1.  Format the EFI partition:

    ```
    mkfs.fat -F32 /dev/device1
    ```

2.  Format the partition:

    ```
    mkfs.btrfs /dev/device2
    ```

### **2.2 ConfigureSubvolumes**

1.  Mount the partition:
	```
	mount -v /dev/device2 /mnt
	```

2.  Create the subvolumes:

    ```
    btrfs subvolume create /mnt/@
    btrfs subvolume create /mnt/@home
    btrfs subvolume create /mnt/@log
    btrfs subvolume create /mnt/@cache
    btrfs subvolume create /mnt/@snapshots
    ```

3.  Unmount the partition:

    ```
    umount /mnt
    ```


## **[Section 3] Mount Partitions**

### **3.1 Mounting Partitions**

1.  Mount the root partition with options:

    ```
    mount -o noatime,compress=zstd,space_cache=v2,subvol=@ /dev/device2 /mnt
    ```

2.  Create and mount the subvolumes:

    ```
    mkdir -pv /mnt/{boot,home,var/log,var/cache,.snapshots}
    mount -o noatime,compress=zstd,space_cache=v2,subvol=@home /dev/device2 /mnt/home
    mount -o noatime,compress=zstd,space_cache=v2,subvol=@log /dev/device2 /mnt/var/log
    mount -o noatime,compress=zstd,space_cache=v2,subvol=@cache /dev/device2 /mnt/var/cache
    mount -o noatime,compress=zstd,space_cache=v2,subvol=@snapshots /dev/device2 /mnt/.snapshots
    ```

3.  Mount the EFI partition to /boot:

    ```
    mount -v /dev/device1 /mnt/boot
    ```
    

## **[Section 4] Configure Repositories and Install the Base System**

1.  Update the mirror list with the fastest mirrors:

    ```
    reflector --verbose --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
    ```

2.  Install the base system:

     ```
     pacstrap /mnt base linux linux-firmware btrfs-progs
     ```


## **[Section 5] Generate fstab**

Generate the `fstab` file to ensure the system recognizes the mounted partitions on boot.

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## **[Section 6] Basic Configuration in chroot**

1.  Enter chroot:

    ```
    arch-chroot /mnt
    ```

2.  Set time zone:

    ```
    ln -sf /usr/share/zoneinfo/<REGION>/<CITY> /etc/localtime
    hwclock --systohc
    ```

3.  Configure language and locales:

    ```
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
    locale-gen
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    ```

4.  Configure the keyboard layout:

    ```
    echo "KEYMAP=la-latin1" > /etc/vconsole.conf
    echo "FONT=lat9w-16" > /etc/vconsole.conf
    ```

5.  Set the hostname:

    ```
    echo "hostname" > /etc/hostname
    ```

6.  Install essential packages:

    ```
    pacman -S grub efibootmgr networkmanager sudo nano
    ```

7.  Install GRUB:

    ```
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

8.  Set root password:

    ```
    passwd
    ```

9.  Create a user:

    ```
	useradd -m user
	passwd user
	usermod -aG wheel,audio,video,optical,storage user
    ``` 

10.  Exit chroot, unmount, and reboot:
    
	    ```
	    exit
	    umount -v /mnt/{boot,home,var/log,var/cache,.snapshots}
	    umount -v /mnt
	    reboot
	    ```

## **[Section 7] After Reboot**

1.  Connect to Wi-Fi:

    ```
    nmcli device wifi connect <SSID> --ask
    ```

2.  Update the system and set up a graphical environment (e.g., Plasma and SDDM and Konsole):

    ```
    sudo pacman -Syu
    sudo pacman -S plasma sddm konsole
    sudo systemctl enable sddm
    ```
    
3. Create swap:

    ```
    btrfs filesystem mkswapfile --size <SIZE> /swapfile
    ```
4. Add swap entry in fstab:
   ```
   /swapfile none swap defaults 0 0
    ```
   
This guide should help you install Arch Linux and set up an initial graphical environment.
