title: Disks

# **Disks**


## **Format a disk**

```bash linenums="1"
sudo parted /dev/sda mklabel gpt
sudo parted -a opt /dev/sda mkpart primary 0% 100%
sudo mkfs.ext4 -L TempStorage /dev/sda1
```


## **Resize a GPT with a ext4**

1. Run parted: `parted /dev/sdX`
2. Change display unit to sectors: `unit s`.
3. Print current partition table and note the start sector for your partition: `p`
4. Delete your partition (won't delete the data or filesystem): `rm <number>`
5. Recreate the partition with the starting sector from above: `mkpart primary <start> <end>`
6. Exit parted: `quit`
7. Check the filesystem: `sudo e2fsck -f /dev/sdXX`
8. Resize filesystem: `sudo resize2fs /dev/sdXX`



## **Disk health checks**

### Non-destructive disk health check (with `smartctl`)

* Start test: `sudo smartctl -t long /dev/sdf # or sudo smartctl -t short /dev/sdf`
* Check results: `sudo smartctl -l selftest /dev/sdf or sudo smartctl -a /dev/sdf`

Note. If `smartctl` test says that disk is healthy, it does not necessarily mean so. See the section below.

### Destructive disk health check (with `badblocks`)
Docs: [Source1](https://superuser.com/questions/66820/full-physical-hd-check),
[Source2](https://calomel.org/badblocks_wipe.html)

```bash linenums="1"
sudo badblocks -c 32768 -p 3 -b 512 -wvs /dev/sdX # (1)!
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

1.  * `-b` -- block size. The default is 1024. **Set it to the output of `#! bash sudo hdparm -I /dev/sdb | grep "Sector size"`**

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




## **mdadm & RAID**

### Replacing a faulty/failing HDD in an mdadm RAID
Docs: [Source1](https://www.tjansson.dk/2013/12/replacing-a-failed-disk-in-a-mdadm-raid/),
[Source2](https://georgebohnisch.com/replace-failing-drive-raid6-array-mdadm/).

##### Determine removed by mdadm disk

[How to determine removed by mdadm disk](https://serverfault.com/questions/841115/how-do-i-determine-the-failed-removed-hdd-in-mdadm-raid).

Light up the led of the failed HDD (assuming you know the failed disk, see the link above): `cat /dev/sdz >/dev/null`

