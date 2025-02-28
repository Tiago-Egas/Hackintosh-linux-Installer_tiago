# Making the installer in Linux

* Supported version: 0.9.5

While you don't need a fresh install of macOS to use OpenCore, some users prefer having a fresh slate with their boot manager upgrades.

To start you'll need the following:

* 4GB USB Stick
* [macrecovery.py](https://github.com/acidanthera/OpenCorePkg/releases)

## Downloading macOS

Now to start, first cd into [macrecovery's folder](https://github.com/acidanthera/OpenCorePkg/releases) and run one of the following commands:


```sh
# Adjust below command to the correct folder
cd /Path/To/Downloaded/Folder/OpenCore-0.9.5-REALEASE/Utilities/macrecovery/
```

Next, run one of the following commands depending on the OS you'd like to boot:

```sh
# Lion(10.7):
python3 ./macrecovery.py -b Mac-2E6FAB96566FE58C -m 00000000000F25Y00 download
python3 ./macrecovery.py -b Mac-C3EC7CD22292981F -m 00000000000F0HM00 download

# Mountain Lion(10.8):
python3 ./macrecovery.py -b Mac-7DF2A3B5E5D671ED -m 00000000000F65100 download

# Mavericks(10.9):
python3 ./macrecovery.py -b Mac-F60DEB81FF30ACF6 -m 00000000000FNN100 download

# Yosemite(10.10):
python3 ./macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000GDVW00 download

# El Capitan(10.11):
python3 ./macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000GQRX00 download

# Sierra(10.12):
python3 ./macrecovery.py -b Mac-77F17D7DA9285301 -m 00000000000J0DX00 download

# High Sierra(10.13)
python3 ./macrecovery.py -b Mac-7BA5B2D9E42DDD94 -m 00000000000J80300 download
python3 ./macrecovery.py -b Mac-BE088AF8C5EB4FA2 -m 00000000000J80300 download

# Mojave(10.14)
python3 ./macrecovery.py -b Mac-7BA5B2DFE22DDD8C -m 00000000000KXPG00 download

# Catalina(10.15)
python3 ./macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download

# Big Sur(11)
python3 ./macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download

# Monterey (12)
python3 ./macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download

# Latest version
# ie. Ventura (13)
python3 ./macrecovery.py -b Mac-B4831CEBD52A0C4C -m 00000000000000000 -os latest download
```

From here, run one of those commands in terminal and once finished you'll get an output similar to this:

![alt text](/images/broly10.png)

## Making the installer

This script automatically flash the recovery and OpenCore to the USB-drive, the Opencore partition will be mounted at`/mnt`  
if you prefer to do the entire process manually skip to the manual installation.

![alt text](/images/broly0.png)

   1. run 
   ```
   curl -o ocflashdrive.sh https://raw.githubusercontent.com/Tiago-Egas/ocflashdrive_tiago/main/ocflashdrive.sh && chmod +x ocflashdrive.sh && ./ocflashdrive.sh
   ```  
   or manually download it a paste it inside `/macrecovery/` directory and type `./ocflashdrive.sh`
   
   2. type in your root password and wait for the script to do its job.

![lsblk](/images/broly12.png)

 ### Manual Instalation
  
For manual instalation this section will target making the necessary partitions in the USB device. You can use your favorite program be it `sgdisk` `gdisk` `fdisk` `parted` `gparted` or `gnome-disks`. This guide will focus on `sgdisk` as it's fast and simple.

### Method 1

In terminal:

   1. run `lsblk` and determine your USB device block
   ![lsblk](/images/broly1.png)
   2. run ``sudo umount /dev/xxx?*`` replace `/xxx` with your USB block

   3. run `sudo sgdisk --zap-all /dev/xxx && partprobe` to remove all partitions on the drive  

   4. run `sudo sgdisk /dev/xxx -o` to clear the partition table and make a new GPT one  
  
   5. run `sudo sgdisk /dev/xxx --new=0:0: -t 0:0700 && partprobe` to create a Microsoft basic data partition type

   6. run `sudo mkfs.vfat -F 32 -n "OPENCORE" /dev/xxx1` to format your USB to FAT32 and named OPENCORE

   7. Use `lsblk` to determine your partition's identifiers

   8. mount your USB partition with `udisksctl` (`udisksctl mount -b /dev/xxx1`, no sudo required in most cases)  
 or with `mount` (`sudo mount /dev/xxx1 /where/your/mount/stuff`, sudo is required)
   9. `cd` to your USB drive and `mkdir com.apple.recovery.boot` in the root of your FAT32 USB partition
   10. now `cp` or `rsync` both `BaseSystem.dmg` and `BaseSystem.chunklist` into `com.apple.recovery.boot` folder.
   ![lsblk](/images/broly3.png)

### Method 2 (in case 1 didn't work)

In terminal:

   1. run `lsblk` and determine your USB device block
   ![lsblk](/images/broly1.png)

   2. run ``sudo umount /dev/xxx?*`` to umount the USB device

   3. run `sudo sgdisk --zap-all /dev/xxx && partprobe` to remove all partitions on the drive

   4. run `sudo sgdisk /dev/xxx -o` to clear the partition table and make a new GPT one

   5. run `sudo sgdisk /dev/xxx --new=0:0:+300MiB -t 0:ef00 && partprobe` to create a 300MB partition that will be named later on OPENCORE

   6. run `sudo sgdisk -e /dev/xxx --new=0:0: -t 0:af00 && partprobe` for Apple HFS/HFS+ partition type

   7. Use `lsblk` again to determine the 300MB drive and the other partition
   ![alt text](/images/broly6.png)

   8. run `sudo mkfs.vfat -F 32 -n "OPENCORE" /dev/xxx1` to format the 300MB partition to FAT32, named OPENCORE

   9. then `cd` to `/OpenCore/Utilities/macrecovery/` and you should get to a `.dmg` and `.chunklist` files
   ![lsblk](/images/broly5.png)

   10. download `dmg2img` (available on most distros)

   11. run `dmg2img -l BaseSystem.dmg` and determine which partition has `disk image` property
   ![alt text](/images/broly8.png)

   12. run `dmg2img -p <the partition number> -i BaseSystem.dmg -o <your HFS+ partition block>`
 to extract and write the recovery image to the partition disk
       ![lsblk](/images/broly9.png)
      * It will take some time. A LOT if you're using a slow USB (took me about less than 5 minutes with a fast USB2.0 drive).
   13. mount the FAT32 partition `udisksctl` (`udisksctl mount -b /dev/xxx1`, no sudo required in most cases)  
 or with `mount` (`sudo mount /dev/xxx1 /where/your/mount/stuff`, sudo is required) this is where you will drop your OC EFI folder.

## Now with all this done, head to https://dortania.github.io/OpenCore-Install-Guide/ktext.html to finish up your work
credits to Dortania for the original guide.
