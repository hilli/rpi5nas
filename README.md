# Raspberry Pi5 NAS with an NVMe drive

Notes and files from [currently an attempt] at creating a NAS on a Raspberry Pi 5 with an NVMe drive attached.

## Parts used

- [Raspberry Pi 5, 4GB](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [Pineberry Pi HatDrive! Bottom](https://pineberrypi.com/products/hatdrive-bottom-2230-2242-2280-for-rpi5)
- Transcend MTE245S - PCIe SSD 245S, M.2 2280 drives, 2TB and 4TB respectively.

## Tech details

Links from mainly @geerlingguy and Raspberry Pi folks that have been useful:

- https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5
- https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html#raspberry-pi-connector-for-pcie
- https://pipci.jeffgeerling.com//hats/pineberry-pi-hatdrive-top.html - PCIe Gen 3 + Boot from NVMe
- https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/559
- https://www.youtube.com/watch?v=EXWu4SUsaY8

“It works best if you flash Pi OS directly to the NVMe SSD, using the Pi with a USB adapter, Pi Imager direct, or on another computer.”

## OS

- Rasbian (Raspberry Pi OS) Lite
- 

### Eeprom update

sudo rpi-eeprom-update -d -a

### Using the NVMe

`boot.txt`

```inifile
[all]
dtparam=pciex1
dtparam=pciex1_gen=3
```

### Stop rasbian to resize to the hole disk

Turns out that the Raspberry Pi Images can customize someof the above and creates a `firstrun.sh` script instead. So ignore the part below.

In `/boot/cmdline.txt` remove:

```shell
init=/usr/lib/raspi-config/init_resize.sh
```
To stop auto resizing the disk at first boot.

Instead do (something like):

```shell
sudo -i
echo "-,16Gib" | sfdisk --no-reread -N2 /dev/sda
partprobe
resize2fs /dev/sda2
reboot
```

