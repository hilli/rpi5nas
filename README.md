# Raspberry Pi5 NAS with an NVMe drive

Notes and files from creating a NAS on a Raspberry Pi 5 with an NVMe drive attached.

## Parts used

- [Raspberry Pi 5, 4GB](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [Pineberry Pi HatDrive! Bottom](https://pineberrypi.com/products/hatdrive-bottom-2230-2242-2280-for-rpi5)
- Transcend MTE245S - PCIe SSD 245S, M.2 2280 drives, 2TB and 4TB respectively.


## Modus Operandi

### Overview

Details below.

- Flash both an micro-SD and the NVMe drive with Raspberry Pi Imager. Why not just `dd` you may ask? Since `dd` does not add a configuration to you image, so it just jumps on you WiFi, has you user and ssh-key etc.
- Add nvme details to `config.txt` on both SD and NVMe. They are available on a FAT partition after making the image. You may have to eject and attach to SD/NVMe to you computer again to see it mount.
- Boot the SD card in the Pi
- Change eeprom config and update it
- Remove SD and reboot to NVMe
- If you are using a 4TB NVMe there is a bit more to do to use it all. But you should probably still fix your `fstab` anyhow.


“It works best if you flash Pi OS directly to the NVMe SSD, using the Pi with a USB adapter, Pi Imager direct, or on another computer.” - @geerlingguy

### OS

- Rasbian (Raspberry Pi OS) Lite

### Set NVMe early in the boot order

To make to Raspberry Pi 5 boot off the NVMe disk you first have to tell to do so from booting it on a SD card.

#### Edit the EEPROM on the Raspberry Pi 5.

`sudo rpi-eeprom-config --edit`

#### Change the BOOT_ORDER line to the following:

`BOOT_ORDER=0xf416`

(Thats adding a `6` at the end to enable NVMe boot to the prioritized list).

Follow the output after the file is saved to update the EEPROM (Thats just a `reboot`).

#### Eeprom update

```shell
# Update eeprom installer
sudo apt update && sudo apt upgrade
# Update the actual eeprom
sudo rpi-eeprom-update -d -a
```

### Using the NVMe

Details for `config.txt`

```inifile
[all]
dtparam=pciex1
# Optional: Set PCIe speed to gen3 (beta, but should work - If not drop it to a 2)
dtparam=pciex1_gen=3
```

## 4TB disks

DOS (MBR) partition layout does not support more than 2TB disks.

```shell
$ sudo fdisk -l /dev/nvme0n1
Disk /dev/nvme0n1: 3.64 TiB, 4000787030016 bytes, 7814037168 sectors
Disk model: TS4TMTE245S                             
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc35c5597

Device         Boot   Start        End    Sectors  Size Id Type
/dev/nvme0n1p1         8192    1056767    1048576  512M  c W95 FAT32 (LBA)
/dev/nvme0n1p2      1056768 4294967295 4293910528    2T 83 Linux
```

So if you are using something larger, and you want to have the entire diskspace in the OS partition, then convert the partition table to GPT:

```shell
$ sudo gdisk /dev/nvme0n1
GPT fdisk (gdisk) version 1.0.9

Caution: invalid main GPT header, but valid backup; regenerating main header
from backup!

Warning: Invalid CRC on main header data; loaded backup partition table.
Warning! Main and backup partition tables differ! Use the 'c' and 'e' options
on the recovery & transformation menu to examine the two tables.

Warning! Main partition table CRC mismatch! Loaded backup partition table
instead of main partition table!

Warning! One or more CRCs don't match. You should repair the disk!
Main header: ERROR
Backup header: OK
Main partition table: ERROR
Backup partition table: OK

Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: damaged

Found valid MBR and corrupt GPT. Which do you want to use? (Using the
GPT MAY permit recovery of GPT data.)
 1 - MBR
 2 - GPT
 3 - Create blank GPT

Your answer: 1

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/nvme0n1.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.
```

### Resize the partition to use the full disk

```shell
$ sudo gdisk /dev/nvme0n1
GPT fdisk (gdisk) version 1.0.9

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/nvme0n1: 7814037168 sectors, 3.6 TiB
Model: TS4TMTE245S                             
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 00FC0F3C-5D99-4E4C-9573-E42A3792EDD5
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 7814037134
Partitions will be aligned on 2048-sector boundaries
Total free space is 3519077997 sectors (1.6 TiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            8192         1056767   512.0 MiB   0700  Microsoft basic data
   2         1056768      4294967295   2.0 TiB     8300  Linux filesystem

Command (? for help): d
Partition number (1-2): 2

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-7814037134, default = 1056768) or {+-}size{KMGTP}: 
Last sector (1056768-7814037134, default = 7814035455) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/nvme0n1: 7814037168 sectors, 3.6 TiB
Model: TS4TMTE245S                             
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 00FC0F3C-5D99-4E4C-9573-E42A3792EDD5
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 7814037134
Partitions will be aligned on 2048-sector boundaries
Total free space is 9837 sectors (4.8 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            8192         1056767   512.0 MiB   0700  Microsoft basic data
   2         1056768      7814035455   3.6 TiB     8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/nvme0n1.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.
```

Re-read the partition table and resize the partition:

```shell
$ sudo partprobe
$ sudo resize2fs /dev/nvme0n1p2
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/nvme0n1p2 is mounted on /; on-line resizing required
old_desc_blocks = 128, new_desc_blocks = 233
The filesystem on /dev/nvme0n1p2 is now 976622336 (4k) blocks long.

$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2  3.6T  1.8G  3.5T   1% /
```

BUUUUT then you can't boot from it without changing the bootloader to point to the correct partition ID and `/etc/fstab` is also pretty off. 

Fix with:

```shell
sudo su -
# Fix /boot/firmware
perl -p -i -e  "s/PARTUUID=.*?-01 /PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p1) /g" /etc/fstab
# Fix /
perl -p -i -e  "s/PARTUUID=.*?-02 /PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p2) /g" /etc/fstab
# Reload systemd and mount /boot/firmware
systemctl daemon-reload
mount /boot/firmware
# Fix bootloader args
perl -p -i -e  "s/ root=PARTUUID=.*? / root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p2) /g" /boot/firmware/cmdline.txt
```

Add this to the /boot/firmware/config.txt

```yaml
[all]
kernel=vmlinuz
cmdline=cmdline.txt
initramfs initrd.img followkernel
# Config settings specific to arm64
arm_64bit=1
dtoverlay=dwc2
dtparam=pciex1
# Optional: Set PCIe speed to gen3 (beta, but should work - If not drop it to a 2)
dtparam=pciex1_gen=3
```

Time to reboot

```shell
sudo reboot
```

This *should* result in a Pi5 that boots *and* has a root partition of 3.6TB:

```shell
$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       3.6T  1.8G  3.5T   1% /
```

## Limiting the OS partition

Rasbian (and other OSs for Raspberry Pi for that sakes matter) will resize the OS partition to the full extend of the disk (modulo MBR limitations). This is a bit annoying when we want to handle some of the diskspace ourselves.

Raspberry Pi Images uses a bit of a different system at boot, creating a `firstrun.sh` script in `bootfs` with no way to stop the resize process before boot.

It will however stop if it sees a 3rd partition. So lets create one.

I am on macOS, and can list the disks with `diskutil list`. It gives me, on this occation, the following (partial) output for the NVMe disk with Rasbian on it:

```text
/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *4.0 TB     disk4
   1:             Windows_FAT_32 bootfs                  536.9 MB   disk4s1
   2:                      Linux                         2.2 GB     disk4s2
                    (free space)                         1.8 TB     -
```

Notice the 4TB disk size, less than 3GB allocated and only 1.8TB free.

So lets add another partition to (in this case) `/dev/disk4. You can install `gdisk` with `brew install gdisk` if it's missing. I want 50GB available for the system after installation, so first deleting partition 2 and then recreating from the same point, but at 50G. Then adding another partition for the remainder of the disk.

```shell
$ sudo gdisk /dev/disk4
GPT fdisk (gdisk) version 1.0.9

Warning: Devices opened with shared lock will not have their
partition table automatically reloaded!
Caution: invalid main GPT header, but valid backup; regenerating main header
from backup!

Warning: Invalid CRC on main header data; loaded backup partition table.
Warning! Main and backup partition tables differ! Use the 'c' and 'e' options
on the recovery & transformation menu to examine the two tables.

Warning! Main partition table CRC mismatch! Loaded backup partition table
instead of main partition table!

Warning! One or more CRCs don't match. You should repair the disk!
Main header: ERROR
Backup header: OK
Main partition table: ERROR
Backup partition table: OK

Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: damaged

Found valid MBR and corrupt GPT. Which do you want to use? (Using the
GPT MAY permit recovery of GPT data.)
 1 - MBR
 2 - GPT
 3 - Create blank GPT

Your answer: 1

Command (? for help): p
Disk /dev/disk4: 7814037168 sectors, 3.6 TiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 289BB9BB-917A-48D8-B5AF-28726370FD4F
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 7814037134
Partitions will be aligned on 2048-sector boundaries
Total free space is 7808695917 sectors (3.6 TiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            8192         1056767   512.0 MiB   0700  Microsoft basic data
   2         1056768         5349375   2.0 GiB     8300  Linux filesystem

Command (? for help): d
Partition number (1-2): 2

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-7814037134, default = 1056768) or {+-}size{KMGTP}:
Last sector (1056768-7814037134, default = 7814035455) or {+-}size{KMGTP}: +50G
Current type is AF00 (Apple HFS/HFS+)
Hex code or GUID (L to show codes, Enter = AF00): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): n
Partition number (3-128, default 3): 3
First sector (34-7814037134, default = 105914368) or {+-}size{KMGTP}:
Last sector (105914368-7814037134, default = 7814035455) or {+-}size{KMGTP}:
Current type is AF00 (Apple HFS/HFS+)
Hex code or GUID (L to show codes, Enter = AF00): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/disk4.
Warning: Devices opened with shared lock will not have their
partition table automatically reloaded!
Warning: The kernel may continue to use old or deleted partitions.
You should reboot or remove the drive.
The operation has completed successfully.

```

macOS is definately confuesed now,

```text
/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *4.0 TB     disk4
   1:                       0xEE                         2.2 TB     disk4s3
   2:             Windows_FAT_32 bootfs                  536.9 MB   disk4s1
```

so eject the disk and reattach it:

```text
/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *4.0 TB     disk4
   1:       Microsoft Basic Data bootfs                  536.9 MB   disk4s1
   2:           Linux Filesystem                         53.7 GB    disk4s2
   3:           Linux Filesystem                         3.9 TB     disk4s3
```

Thats better. Lets fix the boot params. First we need the UUID of the disk OS partition:

```shell
$ diskutil info /dev/disk4s2 | grep UUID | awk '{print $5}'
7BC0760C-6776-42CF-AF2E-B602FA7F350E
```

The `boofs` partition is mounted at `/Volumes/bootfs` so we can fix the kernel params this way:


```shell
perl -p -i -e  "s/ root=PARTUUID=.*? / root=PARTUUID=$(diskutil info /dev/disk4s2 | grep UUID | awk '{print $5}') /g" /Volumes/bootfs/cmdline.txt
```

The OS fs still need to be resized to the 50GB extend, but let see if this works first.
Transfer the NVMe to the Pi and boot.



## Timing results

Pretty decent for a Raspberry Pi

```shell
root@pinas5:~# apt install -y hdparm
root@pinas5:~# hdparm -t --direct /dev/nvme0n1

/dev/nvme0n1:
 Timing O_DIRECT disk reads: 2448 MB in  3.00 seconds = 815.97 MB/sec
```

## References used

Links from mainly @geerlingguy and Raspberry Pi folks that have been useful:

- https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5
- https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html#raspberry-pi-connector-for-pcie
- https://pipci.jeffgeerling.com//hats/pineberry-pi-hatdrive-top.html - PCIe Gen 3 + Boot from NVMe
- https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/559
- https://www.youtube.com/watch?v=EXWu4SUsaY8
- https://forums.raspberrypi.com/viewtopic.php?t=344892
