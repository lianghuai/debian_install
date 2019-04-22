
# resize logical volume 

### resize physical pve volume : 
- option 1 : using **GParted** or **parted** resize physical volumes
- option 2: using [fdisk](https://comcmd.com/tutorials/resize-linux-partition-without-losing-data): 
  * delete the partition 
  * create a new partition from the **same start sector**, with more spaces
  * `resize2fs` the new partition

### show logical volumes info
```
lvm>lvdisplay
```
### resizing examples 
```
lvm> lvextend -L +2G /etc/pve/swap
lvm> lvextend -L +10G /etc/pve/root
lvm> lvextend -L +50G pve/data
```
### using extended spaces
```
#df -h
#resize2fs /dev/mapper/pve-root
```
------

# Add Physical Drive to VM

### install lshw
```
apt install lshw
```

### show drive id 
```
ls -l /dev/disk/by-id/
```

### add to vm :
command : 
```
qm set <vm-id> --<hd-type-and-number> /dev/disk/by-id/<hd-id>
#example :
qm set 101 --sata2 /dev/disk/by-id/ata-ST3250824AS_3ND18NLT
```

---------

# Sleep hard driver if idle 
### turn off pvestated : 
```
pvestatd stop
```
test spin down the HDD : 
```
hdparm -y /dev/sdb
```


