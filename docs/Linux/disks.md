title: Disks

# **Disks**


--------------------------------------------------------------------------------------------------------------
## **Format a disk**

* **gpt partitions**:

    ```bash linenums="1"
    sudo parted /dev/sda mklabel gpt
    sudo parted -a opt /dev/sda mkpart primary 0% 100%
    sudo mkfs.ext4 -L TempStorage /dev/sda1
    ```

* **mbr/dos**

    ```bash
    sudo parted /dev/sdb mklabel msdos
    parted -a opt /dev/sdb mkpart primary 0% 100%
    sudo mkfs.vfat /dev/sdb1
    ```

    You might need to change partition type to HPFS//NFFS/exFAT
    (you can check it in the output of `sudo fdisk -l /dev/sdb`):

    ```bash
    set dev /dev/sdb
    sudo dd if=/dev/zero of=$dev bs=4M status=progress
    sudo sync
    sudo parted --script $dev mklabel msdos
    sudo parted --script -a opt $dev mkpart primary 0% 100%
    echo -e "t\n7\nw" | sudo fdisk $dev
    sudo mkfs.exfat -L ASDF $dev"1"
    sudo sync
    ```



--------------------------------------------------------------------------------------------------------------
## **Move image to a drive**

```
dd if=./src-filename of=/dev/sdb status=progress conv=fsync
```


--------------------------------------------------------------------------------------------------------------
## **Get the list of physical disks**

```title="lsblk -d -o NAME,TYPE,SIZE -e 7,11"
NAME    TYPE   SIZE
sda     disk 223.6G
nvme0n1 disk   3.6T
```

```title="sudo lshw -class disk"
  *-namespace:2
       description: NVMe disk
       physical id: 1
       bus info: nvme@0:1
       logical name: /dev/nvme0n1
       size: 3726GiB (4TB)
  *-disk
       description: ATA Disk
       product: INTEL SSDSC2
       physical id: 0.0.0
       bus info: scsi@1:0.0.0
       logical name: /dev/sda
```

`lsblk --fs` --- shows all block devices along with uuids.



--------------------------------------------------------------------------------------------------------------
## **Erase disk**


#### **HDD & SD cards**

* `sudo dd if=/dev/zero of=/dev/sdX bs=1M status=progress`
* `sudo shred -v -n 3 /dev/sdX` (might need to install `coreutils`)


#### **NVME**

* `sudo nvme format /dev/nvme0n1 --ses=1  # or --ses=2` (might need to install `nvme-cli`)



## **Resize a GPT with a ext4**

1. Run parted: `parted /dev/sdX`
2. Change display unit to sectors: `unit s`.
3. Print current partition table and note the start sector for your partition: `p`
4. Delete your partition (won't delete the data or filesystem): `rm <number>`
5. Recreate the partition with the starting sector from above: `mkpart primary <start> <end>`
6. Exit parted: `quit`
7. Check the filesystem: `sudo e2fsck -f /dev/sdXX`
8. Resize filesystem: `sudo resize2fs /dev/sdXX`




--------------------------------------------------------------------------------------------------------------
## **Add encrypted disk**

```
sudo cryptsetup luksFormat /dev/nvme0n1
sudo cryptsetup open /dev/nvme0n1 nvme0_crypt
sudo mkfs.ext4 -L VortexHome /dev/mapper/nvme0_crypt
```

then add it in `/etc/crypttab`:

```title="sudo nano /etc/crypttab"
nvme0_crypt UUID=dd3a78d4-1085-422f-abd4-e42a1bf50073 none luks,discard

# Or
# nvme0_crypt UUID=c652c9c4-54a0-440f-824a-ec1d8e0d2d50 none luks,keyscript=/usr/share/yubikey-luks/ykluks-keyscript,discard
```

Update initramfs: `sudo update-initramfs -u`

and finally in fstab:

```title="sudo nano /etc/fstab"
/dev/mapper/nvme0_crypt    /home2          ext4    errors=remount-ro 0       1
```



--------------------------------------------------------------------------------------------------------------
## **Disk health checks**

#### **Non-destructive disk health check (with `smartctl`)**

* Start test: `sudo smartctl -t long /dev/sdf # or sudo smartctl -t short /dev/sdf`
* Check results: `sudo smartctl -l selftest /dev/sdf or sudo smartctl -a /dev/sdf`

Note. If `smartctl` test says that disk is healthy, it does not necessarily mean so. See the section below.

#### **Destructive disk health check (with `badblocks`)**

Docs: [Source1](https://superuser.com/questions/66820/full-physical-hd-check),
[Source2](https://calomel.org/badblocks_wipe.html)

```bash linenums="1"
sudo badblocks -c 32768 -p 3 -b 4096 -wvs /dev/sdX # (1)!
Reading and comparing: done
Testing with pattern 0x55: done
Reading and comparing: done
Testing with pattern 0xff: done
Reading and comparing: done
Testing with pattern 0x00: done
Reading and comparing: 1506379640ne, 51:25:53 elapsed. (0/0/0 errors)
1506379641ne, 51:25:56 elapsed. (1/0/0 errors)
1506379642ne, 51:25:59 elapsed. (2/0/0 errors)
1506379643ne, 51:26:02 elapsed. (3/0/0 errors)
```

1.  * `-b` -- block size.

    * `-w` --- use **write-mode test**.

    * `-c` --- is the number of blocks which are tested at a time.

    * `-v` ---verbose mode.

    * `-s` --- show the progress.

    * `-p` --- number of passes.

Later, you can verify unreadability of particular sector with `hdparm` command:

```bash linenums="1"
sudo hdparm --read-sector 3012759284 /dev/sdf # (1)!
/dev/sdf:
reading sector 3012759284: FAILED: Input/output error
```

1.  Note that **if do not set `-b` option properly `badblocks`, you have to adjust sector here**.

    The default block size in badblocks is 1024 bytes (from `badblocks`
    man: `-b block-size Specify the size of blocks in bytes. The default is 1024.`), whereas Logical
    block size is 512 bytes (`sudo hdparm -I /dev/sdf: Logical  Sector size: 512 bytes`), hence 3012759284 (= 1506379642 * 2).




--------------------------------------------------------------------------------------------------------------
## **mdadm & RAID**

#### **Replacing a faulty/failing HDD in an mdadm RAID**
Docs: [Source1](https://www.tjansson.dk/2013/12/replacing-a-failed-disk-in-a-mdadm-raid/),
[Source2](https://georgebohnisch.com/replace-failing-drive-raid6-array-mdadm/).

Determine removed by mdadm disk:

* [How to determine removed by mdadm disk](https://serverfault.com/questions/841115/how-do-i-determine-the-failed-removed-hdd-in-mdadm-raid).
* Light up the led of the failed HDD (assuming you know the failed disk, see the link above): `cat /dev/sdz >/dev/null`
