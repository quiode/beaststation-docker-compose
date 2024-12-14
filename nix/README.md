# NixOS Configuration

## Installation

This is a step-by-step guide to install nixos (with mirrored zfs) on a (virtual) machine.

### Setup

- Enable EFI support
- 2 \* 25G of (empty) Disks
- 8G of Memory
- 4 Cores
- set Network to Bridged Adapter

### Guide

Use the minimal edition of the nixos installer. This is tested for nixos version 24.11.

#### Host Setup

1. Become root: `sudo su`
1. Enable german keys: `loadkeys de`
1. Set password: `passwd`
1. Enable ssh: `systemctl restart sshd`
1. Get the ip of the server: `ip address`

Now connect with the system using the host ssh: `ssh root@ip`.

#### Partitioning

1. Save the disks `DISK1=/dev/disk/by-id/...`, `DISK2=/dev/disk/by-id/...`
1. create the partitions:

```bash
sgdisk -o $DISK1 # optional, clear the disk
sgdisk -n1:1M:+1G -t1:EF00 $DISK1 # efi/boot
sgdisk -n2:0:+4G -t2:8200 $DISK1 # set to a custom size of needed, swap
sgdisk -n3:0:0 -t3:8300 $DISK1 # root/rest

sgdisk -o $DISK2 # clear the second disk
sfdisk --dump $DISK1 | sfdisk $DISK2 # copy the partitions to the second disk
```

3. check the partitions `fdisk -l`

3. create zfs pool

```bash
zpool create \
    -o ashift=12 \
    -o autotrim=on \
    -O acltype=posixacl \
    -O canmount=off \
    -O dnodesize=auto \
    -O normalization=formD \
    -O relatime=on \
    -O xattr=sa \
    -O mountpoint=none \
    rpool \
    mirror \
    $DISK1-part3 $DISK2-part3
```

5. create datasets and mount them

```bash
zfs create rpool/root
zfs create rpool/nix
zfs create rpool/var
zfs create rpool/home

mkdir -p /mnt
mount -t zfs rpool/root /mnt -o zfsutil
mkdir /mnt/nix /mnt/var /mnt/home

mount -t zfs rpool/nix /mnt/nix -o zfsutil
mount -t zfs rpool/var /mnt/var -o zfsutil
mount -t zfs rpool/home /mnt/home -o zfsutil
```

6. format boot

```bash
mkfs.fat -F 32 -n boot $DISK1-part1
mkfs.fat -F 32 -n boot $DISK2-part1
```

7. format swap

```bash
mkswap -L swap $DISK1-part2
swapon $DISK1-part2

mkswap -L swap $DISK2-part2
swapon $DISK2-part2
```

#### Install NixOS

1. mount boot

```bash
mkdir -p /mnt/boot
mount $DISK1-part1 /mnt/boot

mkdir -p /mnt/boot-fallback
mount $DISK2-part1 /mnt/boot-fallback

# Generate the nixos config
nixos-generate-config --root /mnt
```

#### Configure NixOs

1. go into the nixos directory: `cd /mnt/etc/nixos`
1. update bootloader config and network host (get is using `head -c4 /dev/urandom | od -A none -t x4`) in `configuration.nix`

```nix
{
  networking.hostId = "6b0b03b3";
  boot.loader = {
    efi.canTouchEfiVariables = true;
    grub = {
      enable = true;
      efiSupport = true;
      device = "nodev";
      configurationLimit = 5;
      mirroredBoots = [
        {
          devices = [ "/dev/disk/by-id/ata-VBOX_HARDDISK_VBcd418fbb-dbdaf4e2-part1" ];
          path = "/boot-fallback";
        }
      ];
    };
  };
}
```

3. update the hardware configuration `hardware-configuration.nix`

*(Now check the hardware-configuration.nix in /mnt/etc/nixos/hardware-configuration.nix and add whats missing e.g. options = [ "zfsutil" ] for all filesystems except boot and randomEncryption = true; for the swap partition. Also change the generated swap device to the partition we created e.g. /dev/disk/by-id/nvme-SKHynix_HFS512GDE9X081N_FNB6N634510106K5O-part2 in this case and /dev/disk/by-id/nvme-SKHynix_HFS512GDE9X081N_FNB6N634510106K5O-part1 for boot.)* - <https://wiki.nixos.org/wiki/ZFS>

```nix
{
  # ...

  fileSystems."/" =
    { device = "rpool/root";
      fsType = "zfs";
      options = ["zfsutil"];
    };

  fileSystems."/nix" =
    { device = "rpool/nix";
      fsType = "zfs";
      options = ["zfsutil"];
    };

  fileSystems."/var" =
    { device = "rpool/var";
      fsType = "zfs";
      options = ["zfsutil"];
    };

  fileSystems."/home" =
    { device = "rpool/home";
      fsType = "zfs";
      options = ["zfsutil"];
    };

  fileSystems."/boot" =
    { device = "/dev/disk/by-id/ata-VBOX_HARDDISK_VBdb4578c1-95c7dacd-part1";
      fsType = "vfat";
      options = [ "fmask=0022" "dmask=0022" ];
    };

  fileSystems."/boot-fallback" =
    { device = "/dev/disk/by-id/ata-VBOX_HARDDISK_VBcd418fbb-dbdaf4e2-part1";
      fsType = "vfat";
      options = [ "fmask=0022" "dmask=0022" ];
    };

  swapDevices =
    [ { 
         device = "/dev/disk/by-id/ata-VBOX_HARDDISK_VBdb4578c1-95c7dacd-part2"; 
         randomEncryption = true;
      }
      { 
         device = "/dev/disk/by-id/ata-VBOX_HARDDISK_VBcd418fbb-dbdaf4e2-part2"; 
         randomEncryption = true;
      }
    ];

  # ...
}
```

4. install nixos `nixos-install --root /mnt`