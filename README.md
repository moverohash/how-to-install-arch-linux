
## Notes
This guide assumes that you have already downloaded the ISO and created a bootable USB stick or set up a virtual machine using VMDK or QEMU.

## 1. Connect to the Internet

- Use **iwctl** for WiFi; Ethernet should be automatically detected:

    ```
    [iwd]# device list
    ```

- If the device or its corresponding adapter is turned off, turn it on:

    ```
    [iwd]# device name set-property Powered on
    ```
    ```
    [iwd]# adapter adapter set-property Powered on
    ```

- To initiate a scan for networks (note that this command will not output anything):

    ```
    [iwd]# station name scan
    ```

- List all available networks:

    ```
    [iwd]# station name get-networks
    ```

- Connect to a network:

    ```
    [iwd]# station name connect SSID
    ```

## 2. Partition the Disk

- Show partition layout and select your desired location:

    ```
    lsblk
    ```

- Create a new partition layout using `fdisk`:

    ```
    fdisk /dev/sdX
    ```

    **Inside `fdisk`, create 2 partitions:**
    - Type `n` to create a new partition.
    - Choose `p` for primary partition.
    - Specify the partition number (e.g., `1` or `2`).
    - Define the starting and ending sectors:
      - **Partition 1:**
        - Start sector: `2048` (or the default suggested sector)
        - End sector: `+1G` (to specify 1GB)
      - **Partition 2:**
        - Start sector: `2048` (or the default suggested sector)
        - End sector: Just press Enter (to use all remaining space)
    - Type `w` to write changes.

- Format the new partitions using `mkfs`:

    **Partition 1:**
    ```
    mkfs.fat -F32 /dev/sdX1
    ```

    **Partition 2:**
    ```
    mkfs.ext4 /dev/sdX2
    ```

## 3. Install the Base System

- Mount the root partition:

    ```
    mount /dev/sdX2 /mnt
    ```

- Install the base system:

    ```
    pacstrap /mnt base linux linux-firmware
    ```

- Generate the filesystem table:

    ```
    genfstab -U /mnt >> /mnt/etc/fstab
    ```

## 4. Enter the New System and Install Important Packages

- Enter the new system environment:

    ```
    arch-chroot /mnt
    ```

- Set up a root password:

    ```
    passwd
    ```

- Create a new user and add to the `wheel` group:

    ```
    useradd -m -G wheel username
    passwd username
    ```

- Install essential packages:

    ```
    pacman -S grub efibootmgr networkmanager
    ```

- Enable NetworkManager:

    ```
    systemctl enable NetworkManager
    ```

- Create necessary directories for GRUB:

    ```
    mkdir -p /boot/EFI
    ```

- Mount the EFI partition:

    ```
    mount /dev/sdX1 /boot/EFI
    ```

- Install GRUB:

    ```
    grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=grub
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

---

